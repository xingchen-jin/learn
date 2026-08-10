## 设置阈值（Stick Deedzone）
![](assets/Pasted%20image%2020260807155812.png)

![](assets/Pasted%20image%2020260809165625.png)
Unity 新输入系统（Input System）里预设的这个 **`UI`** Action Map，**官方设计的核心用途并不是让你写在脚本里去手动监听（比如去写 `Input.GetKey` ），而是专门配套给 UI 事件系统（EventSystem）使用的。**

根据 Unity 官方文档和标准实践，这些预设的输入动作推荐用在哪里，我帮你按功能块拆解清楚：

---

### 一、核心应用场所：`InputSystemUIInputModule` 组件

这是 Unity 官方推荐的统一接管 UI 交互的标准组件。  
**具体做法**：

1. 在场景中选中 **`EventSystem`** 游戏物体。
    
2. 在 Inspector 面板中，**删掉原有的 `Standalone Input Module`** 组件。
    
3. 点击 `Add Component`，搜索并添加 **`Input System UI Input Module`** 组件。
    
4. 把这个组件上对应的 `Navigate`、`Submit`、`Cancel`、`Point` 等槽位，拖入你截图里的那个 `PlayerInputController` 资源中的对应 Action。
    

**这样配置后，你的 UI 交互（比如鼠标点击按钮、键盘回车、手柄导航）就能直接自动工作了，不需要写任何一行代码。**

---

### 二、各项 `UI` 动作的官方推荐绑定与使用场景

|预设动作|官方推荐绑定源|功能场景|
|---|---|---|
|**Navigate**|键盘 `WASD/方向键` 或 手柄 `Left Stick`|在多个 UI 按钮之间切换焦点（例如手柄控制选单）。|
|**Submit**|键盘 `Enter` 或 手柄 `A` 键|确认当前选中的 UI 元素（类似于点击按钮）。|
|**Cancel**|键盘 `Esc` 或 手柄 `B` 键|关闭弹窗、退出当前界面、返回上一级菜单。|
|**Point**|鼠标 `Mouse/position` 或 触摸屏|**获取鼠标/触摸坐标位置**。用户截图里的 `Control Type: Vector 2` 是绝对正确的，配合 `PassThrough` 才能实现鼠标移动（不需要按住左键就能高亮按钮）。|
|**Click**|鼠标 `Mouse/leftButton` 或 触摸屏|点击 UI 按钮（触发 `OnClick`）。|
|**ScrollWheel**|鼠标 `Mouse/scroll`|在 ScrollView（滚动列表）或滑动条上滚动页面。|
|**MiddleClick**|鼠标 `Mouse/middleButton`|鼠标中键（部分软件中用于快速关闭标签页，视游戏需求可选）。|
|**RightClick**|鼠标 `Mouse/rightButton`|右键菜单（通常在 ARPG/RTS 里用来打开物品详情或子菜单）。|
|**TrackedDevicePosition**|VR/AR 设备的物理空间位置|在 XR（VR/AR）游戏中，用 VR 手柄追踪 UI 选中位置。|
|**TrackedDeviceOrientation**|VR/AR 设备的物理空间旋转|配合上方位置使用，实现 VR 里的“激光射线”点击 UI。|

---

### 三、如果你非要写脚本监听这些 UI 动作（进阶情况）

**官方不建议**直接用 `UI/Click` 去触发 `OnClick`，因为这会让你多写无用的代码。  
但有一种情况你可以用到脚本监听：**你需要在 UI 按下时，同时触发游戏世界的某个动作（比如按下 UI 按钮的同时切换玩家状态）**。

如果是这种情况，官方推荐利用 `PlayerInput` 组件上的 **`Invoke Unity Events`** 模式：

1. 把 `PlayerInput` 组件的 `Behavior` 选为 `Invoke Unity Events`。
    
2. 展开下方的 `Events`，你会看到名为 `UI` 的折叠项。
    
3. 点击 `+` 号，把游戏物体上的特定方法（比如 `SwitchToMenuState()`）拖进去。
    
4. 当玩家按下对应的 `UI` 动作（如 `Submit`）时，这个方法就会被自动调用。
    

---

### 💡 核心避坑提示（针对你的截图）

注意看你截图中选中的 **`Point`** 动作，它的 `Action Type` 是 **`PassThrough`**，`Control Type` 是 **`Vector 2`**。  
这是**绝对正确的**。官方原版模板就是用的 `PassThrough`。  
⚠️ **很多人常犯的错**：如果不小心把 `Point` 改成了 `Button`（或者 `Value`），且没有添加 `Interactions` 为 `Press`，那么鼠标**只要在 UI 上悬停、并没有点击**时，UI 是无法感应到鼠标位置的，导致你无法通过“鼠标悬停”高亮按钮。**切记保持图中默认的 PassThrough 即可！**


![](assets/Pasted%20image%2020260809165732.png)
### 1. 各项预设动作的作用与推荐写法

- **`Move`（移动）**：控制角色前后左右跑动（键盘 `WASD` 或手柄 `左摇杆`）。
    
    - _代码读取_：通常是一个 `Vector2` 值，传递给 `CharacterController` 或 `Rigidbody` 进行物理移动。
        
- **`Look`（视角/朝向）**：控制摄像机或角色转身（鼠标 `移动` 或手柄 `右摇杆`）。
    
- **`Fire`（攻击/射击）**：触发角色攻击、挥砍或交互（鼠标 `左键` 或手柄 `RT/R2` 扳机键）。
    
    - _代码读取_：通常通过 `.WasPressedThisFrame()` 来检测“按下的一瞬间”触发攻击动画。
    - 