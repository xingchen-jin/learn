## 前期准备
在**PackageManager**搜索Asset Bundle Browser下载官方工具，或者在github上下
## 创建AB包

![](assets/Pasted%20image%2020260828211053.png)
点击一个资源文件，就可以在视图串口下方看见这个，选择一个包，Unity回自动识别并关联在对应未打包的AB包

## AB包构建与查看
在Window - AsstBundles打开AB包面板
可以在这个面板上查看AB包资源以及构建AB包，并做出一些设置

#### Build页
![](assets/Pasted%20image%2020260828211517.png)

## 基础API
#### ab包资源加载

```csharp
AsstBundle ab = AsstBundle.LoadFromFile（string path）;//同步加载
ab.LoadAsset<类别>(名字); //同步加载
//...
//协程中进行异步加载
AsstBundleCreateRequest abcr = AssetBundle.LoadFromAsync(string path2);//异步加载包
yield return abcr;
AssetBundleRequest abq = abcr.assetBundle.LoadAsync(resName,typrof(类型));//异步加载资源
yield return abq;

```

## AB包依赖
如果一个包中的资源依赖于另一个包，就必须把所依赖的包全部加载出来才能正常使用改资源。比如一个方块存放在model包里，材质依赖于material包，那么实例化这个方块就必须把material包加载出来才能正常显示，否者显示紫色方块。

#### 可以通过主包或者一个包所依赖的所有包
主包就是跟路径同名的包
通过
```csharp
	AssetBundleManifes abManifest = 	abMain.LoadAsset<AssetBundleManifest>("AssetBundleManifest");//获取依赖信息
	string[] strs = abmanifest.GetAllDependencies("需要查询依赖的包的名字"); //返回依赖
```

**这样做有个缺点，就是会把包里的所有依赖全部加载出来。其实，主包里也只存了包之间的依赖关系，而不是文件之间的关系**
