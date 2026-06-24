# 通用知识
### Mathf类
![](assets/Pasted%20image%2020260622170153.png)
##### Lerp
![](assets/Pasted%20image%2020260622171316.png)
若参数3固定，可以实现速度慢慢减慢的效果，如刹停

### Prefab预制件
![](assets/Pasted%20image%2020260622172124.png)

# UI组件
### Slider
![](assets/Pasted%20image%2020260624122421.png)
应用：音量条，血条，经验条
![](assets/Pasted%20image%2020260624122700.png)
BackGround:空的颜色
Fill Area ： 实心颜色
Handle :拖动的把柄
![424](assets/Pasted%20image%2020260624122926.png)
Value：实心部分的比例，0-1的float
direction：进度条方向

**问题：
 在0和1的时候并没有填满**
 ![](assets/Pasted%20image%2020260624123149.png)
 **会空出来一点
 解决**：
 1，调整FillArea的边框至边缘，2，接着调整Fill的RectTransition至0

### Mask
![](assets/Pasted%20image%2020260624123937.png)
**矩形推荐RectMask2D性能更优秀**

### RectTransform
![](assets/Pasted%20image%2020260624124954.png)
![](assets/Pasted%20image%2020260624124851.png)
继承自Transform
PosX/Y/Z相对于中心点(**锚点**)的像素为单位的距离
width/height矩形长宽调整
![](assets/Pasted%20image%2020260624125500.png)
**Pivot中心点，旋转绕其转**
图像内位置（0-1）（左下-右上）
![](assets/Pasted%20image%2020260624125747.png)
Anchors可以用于不同屏幕的长宽比**适配**.
可调整长宽用。**用比例代替像素**
![](assets/Pasted%20image%2020260624130301.png)
**Alt**：pos跟随锚点
**shift**：Pivot适应调整

### UI制作动画
![](assets/Pasted%20image%2020260624130534.png)
### CanvasGroup
![](assets/Pasted%20image%2020260624130858.png)

为暂停面板加入动画
![](assets/Pasted%20image%2020260624131634.png)

控制变量进行过渡控制
![](assets/Pasted%20image%2020260624133234.png)
![](assets/Pasted%20image%2020260624132135.png)
![](assets/Pasted%20image%2020260624132213.png)

### 制作对话框
![](assets/Pasted%20image%2020260624132545.png)
![](assets/Pasted%20image%2020260624132720.png)
World Space世界内对话框
![](assets/Pasted%20image%2020260624134031.png)
将Canvos作为npc的子物体，创建号Animator编辑动画，在npc上添加**npc脚本**组件引用Animator组件给玩家控制脚本或其他脚本调用，**降低耦合性**

![](assets/Pasted%20image%2020260624134208.png)
体力条与对话框都是通过trigger触发ui动画
