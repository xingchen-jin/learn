Ugui提供的控件，只有常用的事件，比如点击。但要实现双击，长按，拖拽时就要实现事件接口。

## 常用接口
|事件接口 (Interface)|对应方法 (Method)|触发时机|通俗理解|
|---|---|---|---|
|**IPointerEnterHandler**|`OnPointerEnter`|当指针进入对象时调用|鼠标进入|
|**IPointerExitHandler**|`OnPointerExit`|当指针退出对象时调用|鼠标离开|
|**IPointerDownHandler**|`OnPointerDown`|在对象上按下指针时调用|按下|
|**IPointerUpHandler**|`OnPointerUp`|松开指针时调用（在指针正在点击的游戏对象上调用）|抬起|
|**IPointerClickHandler**|`OnPointerClick`|在同一对象上按下再松开指针时调用|点击|

**拖拽相关接口：**

|事件接口 (Interface)|对应方法 (Method)|触发时机|通俗理解|
|---|---|---|---|
|**IBeginDragHandler**|`OnBeginDrag`|即将开始拖动时在拖动对象上调用|开始拖拽|
|**IDragHandler**|`OnDrag`|发生拖动时在拖动对象上调用|拖拽中|
|**IEndDragHandler**|`OnEndDrag`|拖动完成时在拖动对象上调用|结束拖拽|
### UGUI 不常用事件接口（了解即可）

|事件接口 (Interface)|对应方法 (Method)|触发时机 / 用途说明|
|---|---|---|
|**IInitializePotentialDragHandler**|`OnInitializePotentialDrag`|在找到拖动目标时调用，可用于初始化值|
|**IDropHandler**|`OnDrop`|在拖动目标对象上调用|
|**IScrollHandler**|`OnScroll`|当鼠标滚轮滚动时调用|
|**IUpdateSelectedHandler**|`OnUpdateSelected`|每次勾选时在选定对象上调用|
|**ISelectHandler**|`OnSelect`|当对象成为选定对象时调用|
|**IDeselectHandler**|`OnDeselect`|取消选择选定对象时调用|

#### 导航相关事件接口

|事件接口 (Interface)|对应方法 (Method)|触发时机 / 用途说明|
|---|---|---|
|**IMoveHandler**|`OnMove`|发生移动事件（上、下、左、右等）时调用|
|**ISubmitHandler**|`OnSubmit`|按下 Submit 按钮时调用|
|**ICancelHandler**|`OnCancel`|按下 Cancel 按钮时调用|


### PointerEventData 参数的关键内容

**父类**：`BaseEventData`

|参数 (Property)|关键内容说明|
|---|---|
|**pointerId**|鼠标左、右、中键点击鼠标的 ID。通过它可以判断是哪个键（例如右键点击）。|
|**position**|当前指针位置（屏幕坐标系）。|
|**pressPosition**|按下的时候指针的位置。|
|**delta**|指针移动增量（常用于拖拽计算）。|
|**clickCount**|连击次数（常用于判断单击/双击）。|
|**clickTime**|点击时间（上一次点击发生的时间）。|
|**pressEventCamera**|最后一个 `OnPointerPress`（按下）事件关联的摄像机。|
|**enterEventCamera**|最后一个 `OnPointerEnter`（进入）事件关联的摄像机。_(注：原图代码注释拼写为 `enterEvetnCamera`，实际 API 属性名为 `enterEventCamera`)_|
