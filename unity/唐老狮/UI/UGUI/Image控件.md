- **Source Image**: 图片来源（图片类型必须是”精灵“类型）
- **Color**: 图像的颜色
- **Material**: 图像的材质（一般不修改，会使用UI的默认材质）
- **Raycast Target**: 是否作为射线检测的目标（如果不勾选将不会响应射线检测）
- **Maskable**: 是否能被遮罩（之后结合遮罩相关知识点进行讲解）

- **Image Type**: 图片类型
  - **Simple**: 普通模式，均匀缩放整个图片
  - **Sliced**: 切片模式，9宫格拉伸，只拉伸中央十字区域
  - **Tiled**: 平铺模式，重复平铺中央部分
  - **Filled**: 填充模式
	  - 水平填充
	  - 垂直填充
	  - 旋转(90,180,360)

- **Use Sprite Mesh**: 使用精灵网格，勾选的话Unity会帮我们生成图片网格
- **Preserve Aspect**: 确保图像保持其现有尺寸
- **Set Native Size**: 设置为图片资源的原始大小
## 代码控制
	Image img = this.GetComponent<Image>() //获取自身image组件
可以通过img.sprite来设置图片   
