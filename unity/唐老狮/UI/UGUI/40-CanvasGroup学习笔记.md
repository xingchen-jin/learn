# 🎨 UGUI 画布组（Canvas Group）学习笔记

> 课程：UGUI 进阶 · 第 40 讲 | 主讲：唐老师
> 整理依据：课堂文稿 + AI 总结，并对照 **Unity UGUI 官方手册（Canvas Group）** 核对术语与原理。

---

## 🎯 学习目标

学完本讲，你应能回答三个问题：

1. 想**一次性**控制一整个面板（含 N 个按钮、图片、文本）的淡入淡出 / 启用禁用，为什么用普通方法很难？
2. `Canvas Group` 组件的四个属性分别管什么？
3. 面板有父子嵌套时，多层 Canvas Group 会怎样叠加影响？

**一句话总览：** Canvas Group 是挂在一个"父物体"上的**总控开关**——调它一个，它下面所有子控件的透明度、可交互性、是否挡射线会**一起变**，无需逐个修改。

---

## 📌 一、它解决什么问题？

假设一个 `Panel` 下挂了很多 `Button`、`Image`、`Text`。现在要让它**整体淡入淡出**或**整体禁用**：

- ❌ 没有 Canvas Group 时：必须**逐个**拿到每个 `Image`、`Text` 组件，分别修改它们颜色的 Alpha 通道；禁用也得一个个 `button.interactable = false`。控件一多就非常繁琐。
- ✅ 用 Canvas Group：给面板的**父物体**加一个 Canvas Group，改它的 `Alpha` 一个值，整个面板连同所有子控件一起显影 / 淡入淡出。

> 💡 官方原话：*"can be used to control certain aspects of a whole group of UI elements from one place without needing to handle them each individually."*（在一处集中控制整组 UI，而不必逐个处理。）

---

## 🧩 二、四大属性详解

给物体添加 `Canvas Group` 组件后，Inspector 上会出现以下四个参数：

| 属性 | 作用 | 取值 / 默认 |
| --- | --- | --- |
| **Alpha**（整体透明度） | 控制该组内所有 UI 元素的不透明度，实现淡入淡出 | 0 = 完全透明，1 = 完全不透明；默认 1 |
| **Interactable**（整体可交互） | 决定整组是否接受输入；取消勾选后子按钮进入"灰显 / 禁用"状态，无法点击 | 勾选 = true（默认） |
| **Blocks Raycasts**（是否阻挡射线） | 勾选时阻挡鼠标/指针射线（按钮可被点）；取消后射线**穿透**，子控件不再响应输入 | 勾选 = true（默认） |
| **Ignore Parent Groups**（忽略父级） | 本物体若也挂 Canvas Group，勾选后**不受上层 Canvas Group 影响**，独立 Override | 默认不勾选 |

### ⚠️ 三条必须记住的严谨提示

1. **Alpha 是"相乘"，不是"覆盖"。**
   官方明确：*"the Canvas Group alpha and the alpha values of the individual UI elements are multiplied with each other."*
   → 子元素**保留自己原本的透明度**，最终显示不透明度 = 父级 Canvas Group 的 Alpha × 子元素自身的 Alpha。所以哪怕把 Canvas Group 拉到 1，一个自身 Alpha=0.5 的图也只显示一半透明度。

2. **Blocks Raycasts 只管 UGUI 的图形射线，不管物理射线。**
   官方注明：作用于挂在 Canvas 上的 **GraphicRaycaster**，*"This does not apply to Physics.Raycast."*
   → 它拦截/放行的是 UI 点击事件，与 3D 物理射线检测无关，别混淆。

3. **Interactable=false ≠ SetActive(false)。**
   取消 Interactable 只是让控件**变灰、不响应交互**，物体依然**激活、仍然可见**；这与隐藏物体（SetActive）是两回事。

---

## 🧮 三、父子层级如何叠加（Ignore Parent Groups）

本课的核心实验：在 Panel 里再套一个子 Panel（或子物体），并给它也挂一个 Canvas Group，观察 `Ignore Parent Groups` 的作用。

```
Panel  (CanvasGroup A)        ← 父级总控
└── 子物体  (CanvasGroup B)   ← 自己也挂了一个
```

- **未勾选** Ignore Parent Groups（B）：改父级 A 的 Alpha / 交互状态，会**向下传递并叠加**影响子物体 B —— 一起变。
- **勾选** Ignore Parent Groups（B）：子物体 B **独立于父级**，父级怎么调都不影响它，由 B 自己的参数说了算。

> 🔑 理解口诀：**默认"随父"，勾选"独立"。** 多层 Canvas Group 的透明度按上面第 1 条逐层相乘。

---

## ⚙️ 四、运行时控制（C# 参考）

组件在 `UnityEngine.UI` 命名空间下，四个属性与 Inspector 一一对应：`alpha / interactable / blocksRaycasts / ignoreParentGroups`。

**用协程实现面板淡入：**

```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.UI;   // CanvasGroup 在此命名空间

public class PanelFade : MonoBehaviour
{
    private CanvasGroup cg;

    void Awake()
    {
        // 挂在面板父物体上；若没有会自动补一个，便于统一控制
        cg = GetComponent<CanvasGroup>();
        if (cg == null) cg = gameObject.AddComponent<CanvasGroup>();
    }

    public void FadeIn(float duration = 0.3f)  => StartCoroutine(Fade(0f, 1f, duration));
    public void FadeOut(float duration = 0.3f) => StartCoroutine(Fade(1f, 0f, duration));

    IEnumerator Fade(float from, float to, float dur)
    {
        cg.blocksRaycasts = (to > from);   // 淡入时恢复可点，淡出时提前穿透
        cg.alpha = from;
        float t = 0f;
        while (t < dur)
        {
            cg.alpha = Mathf.Lerp(from, to, t / dur);
            t += Time.deltaTime;
            yield return null;
        }
        cg.alpha = to;
        if (to <= 0f) gameObject.SetActive(false); // 淡出后可选地真正隐藏
    }
}
```

> 淡入时把 `blocksRaycasts` 设回 true，淡出刚开始就设为 false，可避免"半透明面板还挡住下层点击"的细节 bug——这正是 Blocks Raycasts 的典型用途。

---

## ⚖️ 五、典型用途（官方列举 + 场景）

| 需求 | 做法 |
| --- | --- |
| 整个窗口淡入淡出 | 给窗口父物体加 Canvas Group，控制其 `Alpha` |
| 一批控件整体置灰 / 不可用 | 给父物体设 `Interactable = false` |
| 让某些 UI 不挡住下层鼠标事件 | 在它或其父级上设 `Blocks Raycasts = false` |
| 子面板不受父级显隐/禁用牵连 | 在子面板上勾选 `Ignore Parent Groups` |

---

## 📝 复习卡片（速记）

- **是什么**：挂在父物体上的 UI **总控组件**，一处调、整组变（影响自身 + 所有子物体）。
- **四大属性**：`Alpha` 整体透明度｜`Interactable` 整体可否交互｜`Blocks Raycasts` 是否挡射线｜`Ignore Parent Groups` 是否忽略父级。
- **关键原理①**：Alpha 与子元素自身透明度**相乘**（0~1）。
- **关键原理②**：Blocks Raycasts 只影响 **GraphicRaycaster**（UI 点击），**不影响 Physics.Raycast**。
- **关键原理③**：Interactable=false 是**置灰禁用**，不是隐藏；想消失还得配合 SetActive / Alpha。
- **层级口诀**：默认**随父叠加**，勾选 Ignore 则**子级独立**。
- **命名空间**：`UnityEngine.UI.CanvasGroup`；运行时属性 `alpha / interactable / blocksRaycasts / ignoreParentGroups`。
- **适用场景**：面板显隐动画、整体禁用、穿透点击、嵌套面板独立控制。

---

> ✅ 至此 UGUI 基础系列（含异形按钮、自动布局、Canvas Group）学习完毕。

---

内容由 AI 生成
