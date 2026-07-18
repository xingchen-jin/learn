## 制作Player动画
#### 1，先创建一个animator Controller作为模板
![](assets/Pasted%20image%2020260718161758.png)
两个tree结点，通过bool值切换。
![](assets/Pasted%20image%2020260718161853.png)
idle结点内添加四个motion Field，通过两个float值切换
![](assets/Pasted%20image%2020260718162043.png)
walk结点内比较特殊，在内部继续创建四个树用于切换行走和跑步

#### 2，通过模板创建override controller，设定不同的动画片段实现不同部位动画的设立。

#### 3，在Player代码中调整值实现状态切换。
