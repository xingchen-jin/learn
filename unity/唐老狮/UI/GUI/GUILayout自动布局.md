## GUILayout静态类（自动布局，主要用于编辑器开发）

可以通过GUILayout.Button或其他组件可以获得自动布局（自动排列，自动对齐大小，适应最长）的组件  

用GUILayout.BeginHorizontal();和GUILayout.EndHorizontal();包裹组件，会变成横向排列，默认纵向排列    

用GUILayout.BeginArea(rect)可以设置位置与可见范围
或通过GUI.BeginGroup和GUI.EndGroup包裹来规定显示位置，范围

## GUILayout布局选项
可以通过诸如GUILayout.Width(width)或GUILayout.Min/MaxWidth()固定控件宽

GUILayout.ExpandWidth（bool）来控制水平拓展的关闭或开启，也就是自动对齐控制，默认开启效果为对齐最长。 


以上纵向同理



