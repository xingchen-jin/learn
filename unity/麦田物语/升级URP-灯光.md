# 游戏昼夜灯光系统实现详解

# 麦田物语：Unity 2D 动态昼夜灯光系统学习笔记
  

> **课程来源**：麦扣《麦田物语》系列教程  

> **技术栈**：Unity 2022 LTS + Universal Render Pipeline (URP) + 2D Renderer + DOTween  

> **核心目标**：实现基于游戏内时间的平滑昼夜光照切换，支持多光源差异化控制

  

---

  

## 1. 系统架构与设计思想

  

本节灯光系统采用 **事件驱动 + 数据驱动** 的混合架构：

  

- **TimeManager**：作为时间中枢，维护游戏内24小时制时间（`GameTime`），并在分钟变更时触发 `LightShiftChangeEvent`

- **LightManager**：订阅时间事件，根据当前季节（Season）与时段（LightShift: Morning/Night）计算**时间差**（`TimeDifference`），并广播至所有灯光控制器

- **LightControl**：挂载于每个 Light2D 组件上，接收切换指令，执行基于 DOTween 的颜色/强度缓动动画

- **LightSettings ScriptableObject**：集中管理配置参数（日出/日落时间、过渡时长、各时段光照参数）

  

> 📌 **官方文档参考**：  

> - [Time in Unity](https://docs.unity3d.com/Manual/class-TimeManager.html)（注意：此处为自定义游戏时间，非 Unity 物理时间）  

> - [2D Lights (URP)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/2D-Lighting.html)

  

---

  

## 2. 核心逻辑实现详解

  

### 2.1 时间判断与 LightShift 切换

  

```csharp

// TimeManager.cs

public LightShift GetLightShift(out float timeDifference)

{

    var gameTime = CurrentGameTime; // 自定义 GameTime 结构（含 Hour, Minute）

    var morning = settings.morningTime; // 如 5 (5:00)

    var night = settings.nightTime;     // 如 19 (19:00)

  

    if (gameTime >= morning && gameTime < night)

    {

        // 白天：从日出开始计时

        timeDifference = (float)(gameTime - morning).TotalMinutes;

        return LightShift.Morning;

    }

    else

    {

        // 夜晚：跨午夜处理（如 23:00 → 04:00）

        var diff = (gameTime < morning)

            ? (gameTime + TimeSpan.FromHours(24)) - night

            : gameTime - night;

        timeDifference = Mathf.Abs((float)diff.TotalMinutes); // 修正负值

        return LightShift.Night;

    }

}

```

  

> ✅ **关键点**：使用 `TimeSpan.TotalMinutes` 获取浮点分钟差，确保精度；通过 `+24h` 处理跨天计算，并用 `Mathf.Abs()` 消除负时间差。

  

---

  

### 2.2 光照过渡动画（DOTween 集成）

  

由于 `Light2D` 不直接支持 DOTween，需手动绑定属性：

  

```csharp

// LightControl.cs

void SwitchLight(Season season, LightShift shift, float timeDiff)

{

    var target = lightData.GetLightDetails(season, shift);

    float duration = settings.lightChangeDuration; // 如 25 秒（游戏时间）

  

    if (timeDiff < duration)

    {

        // 计算当前应处的中间状态

        Color offset = (target.color - currentLight.color) * (timeDiff / duration);

        Color startColor = currentLight.color + offset;

  

        // 使用 DOTween.To 实现自定义插值

        DOTween.To(

            () => startColor,

            c => currentLight.color = c,

            target.color,

            duration - timeDiff

        );

  

        // 强度同理（忽略微小差值优化性能）

        DOTween.To(

            () => currentLight.intensity,

            i => currentLight.intensity = i,

            target.intensity,

            duration - timeDiff

        );

    }

    else

    {

        // 已完成过渡，直接赋值

        currentLight.color = target.color;

        currentLight.intensity = target.intensity;

    }

}

```

  

> 📌 **官方文档参考**：  

> - [DOTween Documentation - To()](http://dotween.demigiant.com/documentation.php#to)  

> - [Light2D API](https://docs.unity3d.com/Packages/com.unity.2d.lighting@7.0/api/UnityEngine.Experimental.Rendering.Universal.Light2D.html)

  

---

  

### 2.3 多光源差异化配置示例

  

| 光源类型       | 白天 (Morning)                     | 夜晚 (Night)                      |

|----------------|-----------------------------------|----------------------------------|

| 全局主光       | 白色，Intensity = 1.0             | 深蓝，Intensity = 0.7            |

| 门灯 (WallLight)| 黄色，Alpha=0 或 Intensity=0（关闭）| 暖黄，Intensity = 1.5（开启）     |

| 路灯           | 关闭                              | 白色，Intensity = 1.2            |

  

> 💡 **提示**：通过在不同 GameObject 上挂载 `LightControl` 并绑定不同 `LightData` 条目，实现独立控制。

  

---

  

## 3. 调试与最佳实践

  

### 3.1 初始化问题修复

- **现象**：游戏启动时灯光未正确初始化（显示错误状态）

- **解决方案**：在 `TimeManager.Start()` 中**立即调用一次** `SwitchLight()`，确保初始状态正确

  

### 3.2 时间加速兼容性

- **限制**：灯光过渡**不跟随时间加速**，仅在目标时间点启动固定时长的动画

- **原因**：真实游戏中玩家无法瞬时跳过数小时，故无需实时同步加速

  

### 3.3 场景扩展建议

- 为每栋建筑（如 `House03`、`Hotel`）挂载专属 `WallLight`

- 利用 URP 的 **2D Shadow Caster 2D** 组件自动处理灯光遮挡

- 路灯可使用单一大范围点光源或多个小光源组合

  

> 📌 **官方文档参考**：  

> - [2D Shadows](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/2D-Shadows.html)  

> - [Shadow Caster 2D](https://docs.unity3d.com/Packages/com.unity.2d.lighting@7.0/manual/ShadowCaster2D.html)

  

---

  

## 4. 总结与延伸

  

本系统成功实现了：

- 基于游戏时间的自动昼夜判定

- 平滑、可配置的光照过渡动画

- 多光源独立控制与场景化部署

  

**后续可扩展方向**：

- 支持四季光照变化（通过 Season 参数）

- 添加天气系统影响光照强度（如阴天降低全局光）

- 优化性能：对远处灯光使用 LOD 或批量处理

  

> 🌾 **麦田物语精神**：用代码编织田园诗意，让每一盏灯都讲述时间的故事。