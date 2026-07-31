# Unity游戏音频系统搭建与场景音效切换实现-AI纪要

# 麦田物语 Unity 2D 游戏音频系统学习笔记

> **课程来源**：麦扣《麦田物语》Unity 教程  
> **核心目标**：实现基于场景切换的动态背景音乐与环境音效管理系统  
> **技术栈**：Unity 2021+、C#、ScriptableObject、AudioSource、Coroutine、Audio Mixer（预告）

---

## 一、音频资源准备与合规使用

- 所有音效与背景音乐均来自 [freesound.org](https://freesound.org)，采用 **CC0 许可证**，允许在商业项目中免费使用（[Unity 官方文档 - Audio Licensing](https://docs.unity3d.com/Manual/AudioFiles.html) 强调需确认音频授权类型）。
- 音频文件按功能分类存放于 `Assets/Sounds/` 目录下：
  - 脚步声（Footstep）
  - 工具使用（ToolUse）
  - 采摘/种植（Harvest, Plant）
  - 环境音（Ambient: Countryside, Room 等）
  - 背景音乐（GameMusic01–06）

> ⚠️ **开发提示**：测试时确保 Unity Editor 的 **Game 视图未勾选 Mute Audio**，否则无声音输出。

---

## 二、Unity 音频基础架构

### 2.1 Audio Listener 与 AudioSource
- Unity 场景中必须存在 **且仅有一个 `AudioListener`**，通常由主摄像机自动添加（[官方文档 - Audio Listener](https://docs.unity3d.com/Manual/class-AudioListener.html)）。
- 每个可发声对象需挂载 `AudioSource` 组件：
  - **2D 音频**：适用于背景音乐，不受空间位置影响（`Spatial Blend = 0`）。
  - **3D 音频**：适用于环境音效，支持距离衰减（`Spatial Blend = 1`）。

### 2.2 AudioManager 层级结构
在场景中创建空 GameObject `AudioManager`，其下挂载两个子对象：
- `GameMusic` → 挂载 `AudioSource`（2D，用于播放背景音乐）
- `AmbientMusic` → 挂载 `AudioSource`（3D，用于播放环境音）

---

## 三、基于 ScriptableObject 的音频数据管理

### 3.1 定义 SoundName 枚举
避免硬编码字符串，使用枚举统一管理所有音效类型：

```csharp
public enum SoundName {
    None,
    Footstep,
    ToolUse,
    Harvest,
    Plant,
    GrassRustle,
    GrassCut,
    Countryside,
    RoomAmbient,
    GameMusic01,
    // ... 其他背景音乐
}
```

### 3.2 创建 SoundDetails ScriptableObject
封装单个音效的元数据（[官方文档 - ScriptableObject](https://docs.unity3d.com/Manual/class-ScriptableObject.html)）：

```csharp
[CreateAssetMenu(fileName = "SoundDetails", menuName = "Audio/SoundDetails")]
public class SoundDetails : ScriptableObject {
    public SoundName soundName;
    public AudioClip clip;
    public Vector2 pitchRange = new Vector2(0.8f, 1.2f);   // 随机音调范围
    public Vector2 volumeRange = new Vector2(0.8f, 1.0f);   // 随机音量范围
}
```

> ✅ **设计优势**：通过随机 `pitch` 和 `volume` 增强高频音效（如脚步、砍树）的听觉多样性，避免机械重复。

### 3.3 创建 SceneSoundList ScriptableObject
绑定场景名称与对应音频：

```csharp
[Serializable]
public class SceneSoundItem {
    public string sceneName;
    public SoundName ambientSound;
    public SoundName gameMusic;
}

[CreateAssetMenu(fileName = "SceneSoundList", menuName = "Audio/SceneSoundList")]
public class SceneSoundList : ScriptableObject {
    public List<SceneSoundItem> sceneSounds;

    public SceneSoundItem GetSoundByScene(string sceneName) {
        return sceneSounds.Find(item => item.sceneName == sceneName);
    }
}
```

示例配置：
| 场景名 | 环境音 | 背景音乐 |
|--------|--------|----------|
| Field  | Countryside | GameMusic01 |
| Home   | RoomAmbient | GameMusic02 |

---

## 四、音频播放逻辑实现

### 4.1 AudioManager 脚本核心逻辑
```csharp
public class AudioManager : MonoBehaviour {
    public AudioSource gameMusicSource;
    public AudioSource ambientMusicSource;
    public SoundDetailsList soundDetailsList;      // 包含所有 SoundDetails 的 SO
    public SceneSoundList sceneSoundList;          // 场景音频映射表

    private Coroutine soundRoutine;

    void OnEnable() {
        SceneManager.sceneLoaded += OnSceneLoaded;
    }

    void OnDisable() {
        SceneManager.sceneLoaded -= OnSceneLoaded;
    }

    void OnSceneLoaded(Scene scene, LoadSceneMode mode) {
        // 停止旧协程
        if (soundRoutine != null) StopCoroutine(soundRoutine);

        var item = sceneSoundList.GetSoundByScene(scene.name);
        if (item == null) return;

        var ambientDetails = soundDetailsList.GetSoundDetails(item.ambientSound);
        var musicDetails = soundDetailsList.GetSoundDetails(item.gameMusic);

        if (ambientDetails != null && ambientMusicSource != null) {
            PlaySound(ambientMusicSource, ambientDetails);
        }

        // 启动协程延迟播放背景音乐
        soundRoutine = StartCoroutine(PlayMusicWithDelay(musicDetails));
    }

    IEnumerator PlayMusicWithDelay(SoundDetails musicDetails) {
        if (musicDetails == null || gameMusicSource == null) yield break;

        float delay = UnityEngine.Random.Range(5f, 15f); // 随机延迟 5–15 秒
        yield return new WaitForSeconds(delay);

        PlaySound(gameMusicSource, musicDetails);
    }

    void PlaySound(AudioSource source, SoundDetails details) {
        if (!source.isActiveAndEnabled) return;

        source.clip = details.clip;
        source.pitch = UnityEngine.Random.Range(details.pitchRange.x, details.pitchRange.y);
        source.volume = UnityEngine.Random.Range(details.volumeRange.x, details.volumeRange.y);
        source.Play();
    }
}
```

> 🔗 **关键 API 参考**：
> - `SceneManager.sceneLoaded`：[官方文档 - Scene Management](https://docs.unity3d.com/ScriptReference/SceneManagement.SceneManager-sceneLoaded.html)
> - `AudioSource.Play()`：[官方文档 - AudioSource](https://docs.unity3d.com/ScriptReference/AudioSource.Play.html)

### 4.2 注意事项
- 背景音乐的 `pitchRange` 应固定为 `(1, 1)`，避免旋律失真。
- 使用 `isActiveAndEnabled` 检查 AudioSource 是否可用，防止在 Timeline 或 UI 中被禁用时报错。

---

## 五、当前局限与后续优化方向

### 5.1 当前问题
- 场景切换时背景音乐 **直接停止并立即播放新曲目**，听感突兀。

### 5.2 下一步计划：引入 Audio Mixer 实现淡入淡出
- 创建 `AudioMixer` 并定义 `Music` 和 `Ambient` 通道。
- 通过 `AudioMixer.SetFloat("MusicVolume", target)` 配合协程实现 **音量渐变**（[官方文档 - Audio Mixer](https://docs.unity3d.com/Manual/AudioMixer.html)）。
- 在切换场景时：
  1. 将当前音乐音量从 0dB 渐变至 -80dB（静音）
  2. 加载新音乐后从 -80dB 渐变回 0dB

> 💡 **目标**：实现专业级的无缝音频过渡体验。

---

## 六、总结

本节课构建了一个**数据驱动、可扩展、支持随机化参数**的 Unity 2D 游戏音频系统，核心亮点包括：
1. **合规使用 CC0 音频资源**
2. **ScriptableObject 实现音频数据解耦**
3. **枚举替代字符串提升代码健壮性**
4. **协程控制播放时序增强沉浸感**

后续将通过 **Audio Mixer** 解决音乐切换生硬问题，进一步提升音频表现力。