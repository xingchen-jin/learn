需要引用using XLua
提供
- DoString 执行lua1语言
- Tick  回收垃圾，一般定时执行
- Dispose  销毁
默认的Lua脚本放在Resources，需要添加.txt后缀才能执行

```csharp
using System.Collections;

using System.Collections.Generic;

using UnityEngine;

using XLua;

  

public class Lesson1_LuaEnv : MonoBehaviour

{

   void Start()

   {

        // 创建Lua解析器

        LuaEnv env = new LuaEnv();

        // 执行Lua代码

        env.DoString("print('Hello World')");

        // 执行Lua文件

        // 默认情况下，Lua文件需要放在Resources目录下

        // 估计是因为Unity的Resources.Load()方法。只能加载txt ，bytes等

        //  Lua文件需要放在Resources目录下，且后缀名为txt

        env.DoString("require('Main')");

        // 垃圾回收

        env.Tick();

        // 销毁Lua解析器

        env.Dispose();

   }

}
```
