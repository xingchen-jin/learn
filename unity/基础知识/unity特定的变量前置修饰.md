以下是 Unity 中常用的“变量/字段/方法前置修饰”完整列表，即**C# 特性（Attribute）**和**特殊方法**，它们能控制序列化、Inspector 显示、组件行为等。

---

## 一、序列化与 Inspector 显示（字段前置）

|特性|说明|适用位置|
|---|---|---|
|`[SerializeField]`|强制序列化**私有字段**，使其在 Inspector 中可见、可编辑。|字段|
|`[NonSerialized]`|禁止序列化**公共字段**，使其不在 Inspector 中显示（但公有的仍可被外部访问）。|字段|
|`[HideInInspector]`|隐藏**公有字段**，使其不在 Inspector 中显示，但仍可序列化保存值。|字段|
|`[Range(min, max)]`|将数值字段显示为滑动条。|float / int 字段|
|`[Min(value)]`|限制数值最小可输入值（滑动条或输入框）。|float / int 字段|
|`[Multiline(lines)]`|将字符串显示为多行文本框。|string 字段|
|`[TextArea(minLines, maxLines)]`|可调节高度的多行文本区域。|string 字段|
|`[Tooltip("提示信息")]`|鼠标悬停在字段上时显示工具提示。|任何字段|
|`[Header("标题")]`|在 Inspector 中插入加粗的分隔标题。|任何字段|
|`[Space(height)]`|在字段上方添加垂直间距。|任何字段|
|`[ColorUsage(showAlpha, hdr)]`|控制颜色字段是否显示 Alpha 通道和 HDR 选项。|Color 字段|
|`[GradientUsage(bool hdr)]`|控制渐变字段是否用于 HDR。|Gradient 字段|
|`[SerializeReference]`|允许序列化**多态引用**（需要继承 Unity 可序列化类）。|字段|
|`[FormerlySerializedAs("旧名称")]`|字段重命名后保留原序列化数据，避免数据丢失。|字段|

---

## 二、组件与脚本行为（类/结构前置）

|特性|说明|适用位置|
|---|---|---|
|`[RequireComponent(typeof(某组件))]`|强制脚本所挂载的游戏对象必须同时拥有指定组件，添加脚本时会自动添加。|类|
|`[DisallowMultipleComponent]`|禁止在同一个游戏对象上添加多个该脚本实例。|类|
|`[AddComponentMenu("路径/脚本名")]`|自定义“Add Component”菜单中的路径。|类|
|`[ExecuteInEditMode]`（已弃用）|使脚本在编辑器模式下也运行，新版本用 `[ExecuteAlways]`。|类|
|`[ExecuteAlways]`|脚本在编辑模式和运行时都会执行。|类|
|`[SelectionBase]`|设置该游戏对象为场景视图的“选择根”，点选子物体时自动选中它。|类|
|`[HelpURL("网址")]`|在组件 Inspector 标题栏的“？”按钮链接到自定义帮助页面。|类|

---

## 三、方法相关特性

|特性|说明|适用位置|
|---|---|---|
|`[ContextMenu("菜单项名称")]`|右键点击组件或在组件齿轮菜单中，可手动调用此方法。|方法|
|`[ContextMenuItem("菜单项名称", "方法名")]`|为某个字段在 Inspector 上右键添加执行方法，用于快捷操作（如重置数据）。|字段|
|`[RuntimeInitializeOnLoadMethod]`|游戏启动时自动调用一次，类似静态构造函数，不依赖场景加载。|静态方法|
|`[InitializeOnLoadMethod]`|编辑器启动时或脚本重编译后自动调用一次（仅编辑器）。|静态方法|
|`[MenuItem("菜单路径")]`|在编辑器顶部菜单栏创建按钮，点击执行静态方法。|静态方法|

---

## 四、MonoBehaviour 生命周期方法（最常用）

这些是 Unity 自动调用的方法，无需特性，但属于“特殊前置修饰”的范畴，按执行顺序列出：

|方法|说明|
|---|---|
|`Awake()`|对象实例化时立即调用，无论脚本是否激活。用于初始化自身引用。|
|`OnEnable()`|每当对象激活时调用（包括初始激活、`SetActive(true)`）。|
|`Start()`|在首次 `Update` 之前调用，且仅在脚本启用时调用。|
|`FixedUpdate()`|固定时间间隔调用，用于物理更新。|
|`Update()`|每帧调用一次，用于常规游戏逻辑。|
|`LateUpdate()`|所有 `Update` 执行完毕后调用，常用于摄像机跟随。|
|`OnDisable()`|对象禁用时调用（`SetActive(false)` 或脚本 disabled）。|
|`OnDestroy()`|对象被销毁时调用（延迟到帧末）。|
|`OnGUI()`|每帧可能多次调用，用于绘制 IMGUI 控件。|
|`OnApplicationPause(bool)`|程序暂停/恢复时调用。|
|`OnApplicationQuit()`|应用程序退出前调用。|
|`OnBecameVisible()` / `OnBecameInvisible()`|当渲染器进入/离开任何摄像机视锥体时调用。|
|`OnCollisionEnter/Exit/Stay()`|碰撞器物理碰撞回调。|
|`OnTriggerEnter/Exit/Stay()`|触发器区域回调。|
|`OnMouseDown/Up/Enter/Exit/Over()`|鼠标与碰撞器交互回调（需 Collider）。|
|`OnDrawGizmos()`|编辑器和游戏运行时在 Scene 视图绘制辅助图标（Gizmo）。|
|`OnDrawGizmosSelected()`|仅当物体选中时绘制 Gizmo。|
|`Reset()`|在 Inspector 中点击组件齿轮菜单的“Reset”时调用，用于恢复默认值。|
|`OnValidate()`|在 Inspector 中修改脚本字段后或加载脚本时调用，用于数据验证。|

---

## 五、编辑器专用脚本特性（额外补充）

|特性|说明|适用位置|
|---|---|---|
|`[CustomEditor(typeof(脚本))]`|声明自定义 Inspector 面板类。|类|
|`[CustomPropertyDrawer(typeof(类型))]`|为可序列化类型自定义 Inspector 绘制方式。|类|
|`[CanEditMultipleObjects]`|允许自定义编辑器同时编辑多个选中物体。|类|

---

## 六、继承与序列化提醒

- Unity 序列化**不会**自动序列化属性（Property）、静态字段、索引器。
    
- `[Serializable]` 用于标记自定义类/结构体，使其能在 Inspector 中显示（需配合 `[SerializeField]` 引用）。
    
- 私有字段若没有 `[SerializeField]` 不会被保存，公共字段默认序列化（除非标记 `[NonSerialized]`）。
    

---

以上涵盖了 Unity 中绝大多数的字段/方法前置修饰特性，熟练使用它们能极大地优化 Inspector 体验和脚本架构。如果需要某个特性的详细用法，可以进一步查阅 [Unity 官方 Attribute 文档](https://docs.unity3d.com/ScriptReference/AddComponentMenu.html)。