## 缩放模式![](assets/Pasted%20image%2020261104202725.png)
#### Expand拓展模式
![](assets/Pasted%20image%2020261104202806.png)

- 优点：显示ui完整细节   
- 缺点：可能出现黑边   

#### Shrink收缩匹配
![](assets/Pasted%20image%2020261104204104.png)

- 优点：最大程度放大UI元素，填满屏幕  
- 缺点：可能出现裁剪  
#### Match Width Or Height宽高匹配
![](assets/Pasted%20image%2020261104204551.png)
![](assets/Pasted%20image%2020261104204627.png)

主要用于只有横屏模式或竖屏模式的游戏
![](assets/Pasted%20image%2020261104204913.png)

- 优点：适应性较强
- 缺点：  

## 恒定物理模式
![](assets/Pasted%20image%2020261104211834.png)
![](assets/Pasted%20image%2020261104212050.png)
![](assets/Pasted%20image%2020261104212104.png)
![](assets/Pasted%20image%2020261104212944.png)

会让UI控件始终保持设置的尺寸大小显示  
而且会根据设备DPI进行计算让在不同设备上的显示大小更加准确  
**一般在游戏开发中极少使用这种模式**  

## 特殊模式 ：3D模式
当canvas渲染模式切换为世界空间，强制使用3D模式。   
主要用于控制该模式下的像素密度   
了解即可，一般不做修改   
