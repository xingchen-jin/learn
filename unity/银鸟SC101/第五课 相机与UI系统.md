# 一，相机
### 相机组件

![](assets/83515fe53c0fb6858d066f734a91baa0.jpeg)![](assets/a696a4e12b378e99a7001360c6c7b7e1.jpeg)
![](assets/4e9589f153191aac054f5772fdf33896.jpeg)
相机如何跟随角色：先获取开始时相机与角色的向量差，之后更新相机的position为玩家的position加上该向量差
# 二，UI系统

### Canavas
![](assets/2b871dbcefbae36ec3661a510f7d2583.jpeg)
![](assets/9ba15e6a10846a58d817484d2e610fd7.jpeg)
### Image
![](assets/026b80f5c2f9db55bbbe08de877fc898.jpeg)
### Text
![](assets/c4b846889b5ef1108ff5d7d48df7fbe5.jpeg)
text可以采用先组件放大，再按T使用框型工具缩小使字体更清晰
### button
![](assets/Pasted%20image%2020260622160412.png)
##### 1,按钮的触发范围是button及其子物体有Raycast Target并且
有颜色（包括透明但早期不行）的地方
##### 2，视觉效果的实现
![](assets/Pasted%20image%2020260622161149.png)
Interactable：取消勾选则变暗下面效果失效
Normal：普通状态
Highlighted:经过状态
Pressed:  按下

Fade Duration：过渡时间
Transition 除了Color
还有Sprite Swap和Animation（高自定义）
##### 3，按钮的作用
![](assets/Pasted%20image%2020260622161755.png)
点击触发事件**可以执行某一个组件中的public函数**
且最多**一个参数**。

### 代码中访问UI组件
先引入包
using UnityEngine.UI;

