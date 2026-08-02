1，VirtualCinema

body：控制pos
aim：控制roation
body设置FramingTransposer缓动跟随
Aim设置POV模式


---

## 一、核心概念

1. 

**Cinemachine**：Unity的相机控制系统，用于创建高质量的摄像机镜头，支持平滑过渡、轨道运动等高级功能。

2. 

**CinemachineVirtualCamera**：Cinemachine的核心组件，控制相机行为，可添加子组件实现特定效果（如缩放、跟随）。

3. 

**CinemachineFramingTransposer**：负责相机镜头的位置和缩放调整，通过`m_CameraDistance`属性控制相机到目标的缩放距离。

## 二、代码结构与解析

### 1. 命名空间与引用

```
using System.Collections;using System.Collections.Generic;using UnityEngine;using Cinemachine;       // 引入Cinemachine命名空间using System;
```

### 2. 类定义与变量

```
public class CinemachineController : MonoBehaviour {    private CinemachineVirtualCamera virtualCamera;    // 引用Cinemachine虚拟相机组件    private CinemachineFramingTransposer framingTransposer;  // 引用FramingTransposer组件，用于缩放控制    [Header("CameraSettings")]      // 编辑器显示分组标题    [SerializeField] private float defaultDistance;      // 默认相机距离    [SerializeField] private float sensitivity;          // 缩放灵敏度    [SerializeField] private float smoothness;           // 平滑过渡速度    [SerializeField] private float minDistance;          // 最小缩放距离限制    [SerializeField] private float maxDistance;          // 最大缩放距离限制    private float targetDistance;                       // 目标缩放距离（动态计算）    private float scrollValue;                          // 鼠标滚轮滚动值    private PlayerInputMap playerInputMap;               // 自定义输入映射类（需额外定义）}
```

**关键点**：

● 

使用`SerializeField`让私有变量可在Inspector面板编辑。

● 

`targetDistance`用于平滑过渡到目标距离。

### 3. 方法解析

#### Start()方法

```
void Start() {    virtualCamera = GetComponent<CinemachineVirtualCamera>();    framingTransposer = virtualCamera.GetCinemachineComponent<CinemachineFramingTransposer>();    framingTransposer.m_CameraDistance = defaultDistance;  // 初始化缩放距离为默认值    targetDistance = defaultDistance;    Cursor.visible = false;     // 隐藏鼠标光标    Cursor.lockState = CursorLockMode.Locked;  // 锁定光标到屏幕中心    playerInputMap = new PlayerInputMap();    playerInputMap.Enable();   // 启用输入映射（需自定义类实现）}
```

**关键点**：

● 

获取组件：通过`GetComponent`获取CinemachineVirtualCamera，再通过`GetCinemachineComponent`获取FramingTransposer。

● 

光标控制：常用于游戏内锁定视角，提升沉浸感。

#### Update()方法

```
void Update() {    scrollValue = GetScroller();}
```

**关键点**：

● 

在每帧Update中获取鼠标滚轮输入值。

#### LateUpdate()方法

```
void LateUpdate() {    targetDistance -= scrollValue * sensitivity;  // 根据滚轮值更新目标距离    targetDistance = Mathf.Clamp(targetDistance, minDistance, maxDistance);  // 限制距离范围    framingTransposer.m_CameraDistance = Mathf.Lerp(framingTransposer.m_CameraDistance, targetDistance, smoothness * Time.deltaTime);}
```

**关键点**：

● 

**平滑缩放**：使用`Mathf.Lerp`实现从当前距离到目标距离的线性插值，`smoothness`控制插值速度，`Time.deltaTime`确保帧率无关。

● 

**边界控制**：`Mathf.Clamp`确保缩放距离在指定范围内，避免溢出。

#### 辅助方法：GetScroller()

```
private float GetScroller() {    return playerInputMap.Player.MouseScroll.ReadValue<Vector2>().y;}
```

**关键点**：

● 

通过自定义的`PlayerInputMap`读取鼠标滚轮垂直滚动值（正值向上滚动，负值向下滚动）。

## 三、关键知识点总结

1. 

**相机组件获取**：通过`GetComponent`和`GetCinemachineComponent`获取Cinemachine相关组件。

2. 

**平滑过渡实现**：使用`Mathf.Lerp`结合`Time.deltaTime`实现帧率无关的平滑动画。

3. 

**输入处理**：封装输入逻辑到`PlayerInputMap`类，提高代码可读性和复用性。

4. 

**属性限制**：通过`Mathf.Clamp`确保参数在安全范围内，避免异常行为。

5. 

**LateUpdate使用**：在LateUpdate中修改相机位置，确保在摄像机更新逻辑之后执行，避免抖动。

## 四、常见问题与解决方案

|   |   |   |
|---|---|---|
|问题描述|可能原因|解决方案|
|相机缩放不平滑|`smoothness`值过低或过高|调整`smoothness`，值越大越平滑但延迟越高|
|滚轮滚动无反应|输入映射未正确配置|检查`PlayerInputMap`的实现是否正确|
|距离超出预期范围|`minDistance`/`maxDistance`设置不当|调整边界值，确保符合场景需求|

## 五、优化建议

1. 

**性能优化**：若相机频繁更新导致性能问题，可减少Update频率或使用协程控制。

2. 

**输入适配**：支持多平台输入（如手柄摇杆缩放），扩展`PlayerInputMap`功能。

3. 

**曲线控制**：使用`AnimationCurve`替代固定灵敏度，实现非线性缩放效果。

## 六、待办事项

● 

研究Cinemachine其他组件（如CinemachineBrain、Blend）实现更复杂的镜头切换。

● 

测试不同`smoothness`值对性能的影响。

● 

封装输入处理为通用模块，提升代码复用性。

## 七、参考资料

● 

[Cinemachine官方文档](https://docs.unity3d.com/Packages/com.unity.cinemachine@2.8/manual/index.html)

● 

[Mathf.Lerp API参考](https://docs.unity3d.com/ScriptReference/Mathf.Lerp.html)

● 

[Unity输入系统教程](https://docs.unity3d.com/Manual/InputSystem.html)

_笔记生成时间：2026-08-01_

---

**备注**：若需进一步优化代码（如输入系统重构或性能调优），可提供更详细的输入映射类实现逻辑。

希望这份笔记能帮助你理解Cinemachine相机控制的逻辑！如果需要补充其他细节或调整格式，随时告诉我。