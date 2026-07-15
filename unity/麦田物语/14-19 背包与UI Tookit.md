## 背包数据初始化
用*Details*作为数据的统一命名
同时使用例如10XX的四位数命名，避免与其他的数据冲突
创建ScriptableObject类，用于存放物品信息列表的数据
## MVC模式
将程序分为模型，视图，控制器三块
逻辑清晰，减少耦合

## UI Tookit制作物理编辑器
首先先创建UItookit的EditorWindow
打开Builder窗口后，拖拽组件的方式编辑GUI界面

## 创建ItemTemplate

先创建UI Document文件，打开后就是UI Builder
**数据类型为VisualTreeAsset**

## 生成ListView列表
AssetDatabase.FindAssets("t:ItemDataList_SO")寻炸数据实例返回GUID（字符串类型）数组

AssetDatabase.GUIDToAssetPath(dataArray[0]);根据GUID找资源路径
AssetDatabase.LoadAssetAtPath(path, typeof(ItemDataList_SO)) as ItemDataList_SO;根据路径返回实例
EditorUtility.SetDirty(dataBase);//必须标记才会保存数据

#### List View模板
```csharp
func<VisualElement>makeItem，每次数据绘制到窗口时按数据的个数逐一增加一个项目的时候调用。
Action<Element,int>bingItem，决定实际每一项显示的内容，e就是实际的Element

var listView = container.Q<ListView>();

IntegerField模板

// Get a reference to the field from UXML and assign a value to it.
var uxmlField = container.Q<IntegerField>("the-uxml-field");
uxmlField.value = 42;

// Create a new field, disable it, and give it a style class.
var csharpField = new IntegerField("C# Field");
csharpField.SetEnabled(false);
csharpField.AddToClassList("some-styled-field");
csharpField.value = uxmlField.value;
container.Add(csharpField);

// Mirror the value of the UXML field into the C# field.
uxmlField.RegisterCallback<ChangeEvent<int>>((evt) =>
{
    csharpField.value = evt.newValue;
});
```
物品信息编辑功能
``itemListView.selectionChanged += OnListSelectionChange;//注册选择事件

