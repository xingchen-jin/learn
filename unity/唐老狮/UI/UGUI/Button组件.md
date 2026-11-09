## 一、 Button 基础与结构

- **定义**：UGUI 中用于处理玩家按钮相关交互的关键组件。
    
- **创建方法**：Hierarchy 窗口右键 -> `UI` -> `Button`。
    
- **默认结构**：
    
    - **父对象 (Button)**：挂载了 `RectTransform`（定位尺寸）、`Image`（背景图）、`Button`（交互逻辑）。
        
    - **子对象 (Text)**：挂载 `Text` 组件，用于显示文字（可选，若背景图自带文字可删除）。
        
- **点击原理**：只要 Button 前面没有被其他物体（如透明遮罩、其他UI）挡住，运行游戏后即可被点击。
    

## 二、 Button 核心参数

### 1. Interactable（是否接受输入）

- **勾选**（默认）：按钮可正常交互。
    
- **取消勾选**：按钮失活，无法点击，视觉上会变为**禁用颜色**（Disabled Color）。
    

### 2. Transition（过渡效果）

响应玩家输入时的视觉反馈，共四种模式：

|模式|说明|适用场景|
|---|---|---|
|**None**|无任何状态变化反馈|极少使用|
|**Color Tint**|颜色渐变（改变 Target Graphic 的颜色）|**最常用**，开发效率高|
|**Sprite Swap**|图片切换（替换不同的 Sprite 图片）|需要美术提供多张图|
|**Animation**|动画（利用 Animator 状态机）|效果丰富但制作繁琐|

#### ① Color Tint 模式参数

- **Target Graphic**：控制变色的目标图形（默认自身 Image）。
    
- **五种状态颜色**：`Normal`（正常）、`Highlighted`（悬停高亮）、`Pressed`（按下）、`Selected`（选中）、`Disabled`（禁用）。
    
- **Color Multiplier**：颜色倍增器（一般保持默认 1）。
    
- **Fade Duration**：状态过渡时间（默认 `0.1` 秒，值越大变化越慢）。
    

#### ② Sprite Swap 模式参数

- 需要指定 `Target Graphic`。
    
- 配置 `Highlighted Sprite`、`Pressed Sprite`、`Selected Sprite`、`Disabled Sprite`。
    
- _注意_：Normal 状态直接用 Image 组件原本的 Source Image 即可，无需额外设置。
    

#### ③ Animation 模式参数

- 点击 **Auto Generate Animation** 会生成一个 `.controller` 文件，自动预设好 5 个状态及 Trigger。
    
- 打开 Animation 窗口可录制关键帧（如悬停时 Scale 从 1 变 1.2）。
    
- **⚠️ 注意**：如果只希望动画播放一次（如放大后保持），需要在 Animation 窗口中取消勾选 **Loop Time**。
    

### 3. Navigation（键盘导航模式）

决定通过键盘/手柄方向键切换焦点时的逻辑：

- **None**：禁用导航。
    
- **Horizontal / Vertical**：仅允许水平/垂直方向切换。
    
- **Automatic**：系统根据屏幕空间位置自动判断下一个目标。
    
- **Everything**：包含上述所有功能。
    
- **Explicit**：开发者手动指定上、下、左、右绑定的 GameObject。
    
- **Visualize**：点击后可在 Scene 视图中看到焦点跳转的黄色连线，方便调试。
    

---

## 三、 Button 的代码控制

- **命名空间**：必须引入 `using UnityEngine.UI;`。
    
- **获取组件**：`Button btn = this.GetComponent<Button>();`。
    
- **修改参数**：可直接点出属性进行修改，例如：
    
    - 禁用按钮：`btn.interactable = false;`
        
    - 修改过渡模式：`btn.transition = Selectable.Transition.None;`
        
- **修改背景图**：推荐直接获取 Image 组件修改（`this.GetComponent<Image>()`），而不是通过 Button 获取。
    

---

## 四、 点击事件监听（两种方式）

**点击判定标准**：必须在按钮区域内按下鼠标，**并在该区域内抬起**，才算一次有效点击。（内按外松不触发）。

### 方式一：拖脚本（Inspector 面板操作）

- **步骤**：选中 Button -> 在 Inspector 的 `On Click()` 点击 `+` -> 拖入挂载了脚本的 GameObject -> 在下拉菜单中选择要调用的函数。
    
- **限制**：**只能选择 `public`（公共）函数**。
    
- **特点**：直观简单，支持在一个按钮上挂载多个函数，按顺序执行。
    

### 方式二：代码添加监听（逻辑控制）

- **常规写法（具名函数）**：
    
    csharp
    
    // 声明私有函数
    private void ClickBtn2() { print("点击"); }
    // 在 Start 中添加
    btn.onClick.AddListener(ClickBtn2);
    
- **简化写法（Lambda 表达式）**：
    
    csharp
    
    btn.onClick.AddListener(() => { print("匿名点击"); });
    

### 移除监听

- **指定移除**：`btn.onClick.RemoveListener(ClickBtn2);` （只能移除具名函数，无法移除 Lambda）。
    
- **全部移除**：`btn.onClick.RemoveAllListeners();` （清空当前按钮上的所有监听，包括 Lambda）。
    

---

## 📝 核心考点/复习自测

1. Button 默认由哪两个对象组成？各自挂载了什么组件？
    
2. 如果我想让按钮在鼠标悬停时变成一张高亮图片，应该选择哪种 Transition 模式？
    
3. 在代码中，给 Button 添加点击事件的委托类型是什么？
    
4. 为什么用 Lambda 表达式添加的监听，无法使用 `RemoveListener` 单独移除？