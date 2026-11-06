![](assets/Pasted%20image%2020261105195324.png)

## 组件参数
- Pivot：轴心(中心)点，取值范围0~1  (左下角为0，0)
- Anchors(相对父矩形锚点)：  
	Min是矩形锚点范围X和Y的最小值  
	Max是矩形锚点范围X和Y的最大值  
	取值范围都是0~1  
	 当锚点代表一个范围，会用left/top/right/Bottom  
	 为点时用pos  
- Pos(X,Y,Z)：轴心点(中心点)相对锚点的位置  
- Width/Height：矩形的宽高，变化方向取决于中心点位置  
- Left/Top/Right/Bottom：矩形边缘相对于锚点的位置；当锚点分离时会出现这些内容  
- Rotation：围绕轴心点旋转的角度  
- Scale：缩放大小（底部略有截断，推测为Scale）
![](assets/Pasted%20image%2020261105202358.png)
启用原始编辑模式应该时会改变位置，不改变轴心和锚点位置  

点击左上角的方框，可以快速设置面板   
**可以快速设置锚点**  
按住**Shift**点击鼠标左键可以同时设置**轴心点**（相对自身矩形）  
按住**Alt**点击鼠标左键可以同时设置**位置**   

## 代码
代码中通过  
``this.transform as RectTransform ``获取  
通过sizeDelta可以设置大小   

