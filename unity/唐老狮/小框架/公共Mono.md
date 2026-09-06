## 作用
让不继承MonoBehaviour的脚本也能
- 1.利用帧更新或定时更新处理逻辑

- 2.利用协同程序处理逻辑

- 3.可以统一执行管理帧更新或定时更新相关逻辑(不管你是否继承MonoBehaviour)

## 实现原理
- 公共mono使用继承mono的单例模式保证唯一性，以及使用mono的生命周期和协程
- 使用事件，在生命周期调用事件，只要在需要注册的脚本中注册事件，就可以利用公共mono的生命周期来实现自己的更新逻辑。
- 对于协同程序，直接是用单例模式来启动。
## 简单实现

```csharp
using System.Collections;

using System.Collections.Generic;

using UnityEngine;

using UnityEngine.Events;

  

/// <summary>

/// 公共Mono模块管理器

/// </summary>

public class MonoMgr : SingletonAutoMono<MonoMgr>

{

    private event UnityAction updateEvent;

    private event UnityAction fixedUpdateEvent;

    private event UnityAction lateUpdateEvent;

  

    /// <summary>

    /// 添加Update帧更新监听函数

    /// </summary>

    /// <param name="updateFun"></param>

    public void AddUpdateListener(UnityAction updateFun)

    {

        updateEvent += updateFun;

    }

  

    /// <summary>

    /// 移除Update帧更新监听函数

    /// </summary>

    /// <param name="updateFun"></param>

    public void RemoveUpdateListener(UnityAction updateFun)

    {

        updateEvent -= updateFun;

    }

  

    /// <summary>

    /// 添加FixedUpdate帧更新监听函数

    /// </summary>

    /// <param name="updateFun"></param>

    public void AddFixedUpdateListener(UnityAction updateFun)

    {

        fixedUpdateEvent += updateFun;

    }

    /// <summary>

    /// 移除FixedUpdate帧更新监听函数

    /// </summary>

    /// <param name="updateFun"></param>

    public void RemoveFixedUpdateListener(UnityAction updateFun)

    {

        fixedUpdateEvent -= updateFun;

    }

  

    /// <summary>

    /// 添加LateUpdate帧更新监听函数

    /// </summary>

    /// <param name="updateFun"></param>

    public void AddLateUpdateListener(UnityAction updateFun)

    {

        lateUpdateEvent += updateFun;

    }

  

    /// <summary>

    /// 移除LateUpdate帧更新监听函数

    /// </summary>

    /// <param name="updateFun"></param>

    public void RemoveLateUpdateListener(UnityAction updateFun)

    {

        lateUpdateEvent -= updateFun;

    }

  
  

    private void Update()

    {

        updateEvent?.Invoke();

    }

  

    private void FixedUpdate()

    {

        fixedUpdateEvent?.Invoke();

    }

  

    private void LateUpdate()

    {

        lateUpdateEvent?.Invoke();

    }

}

```