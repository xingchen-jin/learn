抱歉，上一份笔记确实在“必备要素”上表述不够严谨。以下是严格依据 Unity 6 官方文档重新整理的 `EditorWindow` 笔记，重点厘清**什么是必须的**、**什么是可选的**。

---

# Unity EditorWindow 笔记（官方文档核准版）

## 一、EditorWindow 是什么

`EditorWindow` 是 UnityEditor 命名空间下的类，**继承自 `ScriptableObject`**。它的作用是创建可以独立浮动或作为选项卡停靠的编辑器窗口，外观和行为与 Unity 自带的 Inspector、Scene 等窗口一致。

> 脚本**必须放在名为 `Editor` 的文件夹中**，这是硬性要求，否则 `UnityEditor` 相关代码无法在打包时正常工作。

---

## 二、创建自定义窗口的官方三步

Unity 官方文档明确列出创建自定义编辑器窗口的步骤：

1. **创建一个继承自 `EditorWindow` 的脚本**。
2. **用代码触发窗口显示自身**（通常是 `MenuItem` + `GetWindow`，但这不是唯一的触发方式）。
3. **实现 GUI 代码**来绘制工具界面。

这三步中，**第 1 步是不可省略的根基**；第 2、3 步的具体实现方式有选择空间，下面分别说明。

---

## 三、继承 EditorWindow：唯一必须的要素

```csharp
using UnityEditor;
using UnityEngine;

public class MyWindow : EditorWindow
{
    // 窗口内容在这里绘制
}
```

所有自定义编辑器窗口都**必须**继承 `EditorWindow`。这是唯一没有替代方案的硬性要求。

---

## 四、打开窗口：MenuItem 不是必须的

`MenuItem` 是最常见的打开方式，但**不是唯一方式**。窗口的显示完全由代码主动调用 `GetWindow` 等 API 来触发。

### 4.1 最常见的方式：MenuItem + GetWindow

```csharp
[MenuItem("Window/My Window")]
public static void ShowWindow()
{
    EditorWindow.GetWindow(typeof(MyWindow));
}
```

`GetWindow` 的**默认行为是复用窗口**：再次点击菜单项时，如果窗口已存在，会聚焦到已有窗口，而不是创建第二个。

### 4.2 GetWindow 与 GetWindowWithRect

| API | 特点 |
|---|---|
| `GetWindow(typeof(T))` | 标准可停靠窗口，**会保存上次的位置和大小**，可加入自定义布局 |
| `GetWindowWithRect(typeof(T), rect)` | **指定初始位置和大小**，对窗口外观有更多控制权 |

两者返回的都是 `EditorWindow` 类型，需要转换回你的子类才能访问自定义成员：

```csharp
MyWindow window = EditorWindow.GetWindow<MyWindow>();   // 泛型方式，无需强转
// 或
MyWindow window = (MyWindow)EditorWindow.GetWindow(typeof(MyWindow));  // 非泛型方式，需强转
```

`Show()` 方法用于显示窗口，但在 `GetWindow` 之后通常不是严格必需的，因为 `GetWindow` 本身就会让窗口可见。

### 4.3 MenuItem 是“入口”，不是“窗口的一部分”

`MenuItem` 只是**提供一个可从菜单触发的静态方法**。你完全可以在其他编辑器脚本、自定义快捷键、甚至自动化流程中调用同一个 `ShowWindow` 方法，窗口照样能打开。`MenuItem` 的职责是“给用户一个点击入口”，而不是“让窗口成为窗口”。

---

## 五、绘制界面：OnGUI 或 CreateGUI，二选一

Unity 编辑器 UI 有**两套系统**，选择哪套决定了你实现哪个方法：

| 系统 | 实现的方法 | 官方态度 |
|---|---|---|
| **IMGUI**（即时模式） | `OnGUI()` | 仍支持，但**官方强烈推荐迁移到 UI Toolkit** |
| **UI Toolkit**（现代 UI） | `CreateGUI()` | **官方推荐**，更现代、灵活、可扩展 |

### 5.1 如果用 IMGUI：实现 OnGUI

```csharp
void OnGUI()
{
    // 使用 GUI / GUILayout 绘制控件
    if (GUILayout.Button("点击我"))
    {
        Debug.Log("按钮被点击");
    }
}
```

`OnGUI` 每帧可能被调用多次（处理渲染和 GUI 事件），因此**所有控件的返回值都必须保存到成员变量**，否则状态会丢失。

IMGUI 可用 `GUI`（绝对坐标）或 `GUILayout`（自动布局），另外还有编辑器专用的 `EditorGUI` 和 `EditorGUILayout` 提供更多控件。

### 5.2 如果用 UI Toolkit：实现 CreateGUI

```csharp
public void CreateGUI()
{
    // rootVisualElement 此时已就绪，可以添加 UI Toolkit 控件
    var button = new Button(() => Debug.Log("点击"));
    button.text = "点击我";
    rootVisualElement.Add(button);
}
```

`CreateGUI` 在窗口的 `rootVisualElement` 准备好填充时被调用。**依赖 UXML/USS 加载的代码应放在 `CreateGUI` 中**，以确保资源已就绪。如果使用 UXML，可以通过 `AssetDatabase.LoadAssetAtPath<VisualTreeAsset>` 加载后 `Instantiate` 到 `rootVisualElement`。

### 5.3 两套系统不要混用

一个窗口只应选择其中一种绘制方式。IMGUI 的 `OnGUI` 和 UI Toolkit 的 `CreateGUI` 不应当同时作为主要绘制入口，否则会出现难以排查的冲突。

---

## 六、窗口生命周期（官方顺序）

官方文档列出了 EditorWindow 的主要回调顺序：

| 回调 | 调用时机 |
|---|---|
| `OnEnable` | 脚本加载或对象启用时 |
| `CreateGUI` | 编辑器**不在更新 AssetDatabase 时**，生成 UI（UI Toolkit 专用） |
| `Update` | 窗口可见时每秒多次调用 |
| `OnGUI` | 每帧多次，用于渲染和处理 GUI 事件（IMGUI 专用） |
| `OnDisable` | 脚本禁用或对象销毁时，用于清理资源 |

---

## 七、状态保持：为什么控件返回值必须赋回成员变量

无论 IMGUI 还是 UI Toolkit，窗口的状态都**不会自动保留**。IMGUI 是无状态的，每一帧都要靠成员变量重建；UI Toolkit 虽然控件对象本身存在，但数据绑定也需要显式维护。

以 IMGUI 为例，以下写法是**错误**的：

```csharp
void OnGUI()
{
    string temp = GUI.TextField(new Rect(10, 10, 200, 20), "默认值");  // ❌ 局部变量，下一帧就丢了
}
```

正确写法：

```csharp
private string inputText = "默认值";  // 成员变量

void OnGUI()
{
    inputText = GUI.TextField(new Rect(10, 10, 200, 20), inputText);  // ✅ 返回值赋回成员变量
}
```

`GUI.Toolbar`、`GUI.TextField` 等所有有状态的控件，都必须把返回值赋回成员变量，否则用户操作不会生效。

---

## 八、总结：什么是必须的，什么是可选的

| 要素 | 是否必须 | 说明 |
|---|---|---|
| **继承 `EditorWindow`** | **必须** | 所有编辑器窗口的基类，没有替代方案 |
| **脚本放在 `Editor` 文件夹** | **必须** | 否则 `UnityEditor` 代码无法正常工作 |
| **`MenuItem`** | 非必须 | 只是打开窗口的**最常见入口**，可通过其他代码触发 `GetWindow` |
| **`GetWindow` / `GetWindowWithRect`** | **打开窗口必须** | 但调用时机可以由任意代码触发，不绑定菜单 |
| **`OnGUI`** | 非必须 | **IMGUI** 的绘制入口，官方强烈推荐迁移到 UI Toolkit |
| **`CreateGUI`** | 非必须 | **UI Toolkit** 的绘制入口，官方推荐的现代方案 |

核心结论：**“继承 `EditorWindow`”和“脚本放在 `Editor` 文件夹”是两条硬性要求；“如何打开”和“如何绘制”都有官方支持的多条路径。**
## 课程代码

```lua
using System.Collections;

using System.Collections.Generic;

using UnityEditor;

using UnityEngine;

  

public class ABTools : EditorWindow

{

    private int nowSelIndex = 0;

    private string[] targetStrs = {"PC","安卓","IOS"};

    private string serverIP = "ftp://127.0.0.1";

    [MenuItem("AB包工具/打开工具窗口")]

    private static void OpenWindow()

    {

        //获取一个ABTools 编辑器窗口对象

       ABTools windown =  EditorWindow.GetWindowWithRect(typeof(ABTools),new Rect(0, 0, 350, 220)) as ABTools;

       windown.Show();

    }

  

    private void OnGUI()

    {

        GUI.Label(new Rect(10, 10, 150, 15), "平台选择");

        //页签显示 是从数组中取出字符串内容来显示 所以 需要改变当前选中的索引

        nowSelIndex = GUI.Toolbar(new Rect(10, 30, 250, 20), nowSelIndex, targetStrs);

        //资源服务器IP地址设置

        GUI.Label(new Rect(10, 60, 150, 15), "资源服务器地址");

        serverIP = GUI.TextField(new Rect(10, 80, 150, 20), serverIP);

        //创建对比文件 按钮

        if(GUI.Button(new Rect(10, 110, 100, 40), "创建对比文件"))

        {

  

        }

        //保存默认资源到StreamingAssets 按钮

        if (GUI.Button(new Rect(115, 110, 225, 40), "保存默认资源到StreamingAssets"))

        {

  

        }

        //上传AB包和对比文件 按钮

        if (GUI.Button(new Rect(10, 160, 330, 40), "上传AB包和对比文件"))

        {

  

        }

  

    }

}
```