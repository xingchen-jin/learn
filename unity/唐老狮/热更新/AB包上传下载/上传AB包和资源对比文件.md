
## 一、这个工具做什么

在编辑器菜单新增 **`AB包工具 → 上传所有AB包和对比文件`**，一键把 `Assets/ArtRes/AB/PC/` 下所有 **AB 包（无后缀）** 和 **`ABCompareInfo.txt`（对比文件）** 通过 FTP 上传到服务器。

---

## 二、关键知识点

### 1. 编辑器菜单入口

```csharp
[MenuItem("AB包工具/上传所有AB包和对比文件")]
public static void UploadAllABFile()
```

与之前的 `CreateABCompare` 相同，`[MenuItem]` + `static` 方法即可生成菜单项，脚本需在 `Editor` 目录下。

### 2. 获取目录文件并筛选

```csharp
DirectoryInfo directory = Directory.CreateDirectory(Application.dataPath + "/ArtRes/AB/PC/");
FileInfo[] fileInfos = directory.GetFiles();

if (info.Extension == "" || info.Extension == ".txt")
{
    FtpUploadFile(info.FullName, info.Name);
}
```

筛选条件两个：

| 条件 | 含义 |
|------|------|
| `Extension == ""` | 无后缀 → AB 包 |
| `Extension == ".txt"` | 对比文件 `ABCompareInfo.txt` |

> **更安全的写法**：用文件名精确匹配，如 `info.Name == "ABCompareInfo.txt"`，避免误传其他 `.txt` 文件（代码注释里也提到了这点）。

### 3. FTP 上传四步（核心）

```csharp
// ① 创建请求：ftp://IP/服务器路径/文件名
FtpWebRequest request = (FtpWebRequest)FtpWebRequest.Create(
    new Uri("ftp://10.100.209.199/AB/PC/" + fileName));

// ② 设置凭证（有账号密码才能上传）
request.Credentials = new NetworkCredential("XC", "XC12345");

// ③ 其他设置
request.Proxy = null;                                      // 不使用代理
request.KeepAlive = false;                                 // 请求完关闭控制连接
request.Method = WebRequestMethods.Ftp.UploadFile;         // 操作命令：上传
request.UseBinary = true;                                  // 二进制传输（AB包必须）

// ④ 获取请求流，循环写入文件数据
Stream uploadStream = request.GetRequestStream();
using (FileStream fileStream = File.OpenRead(filePath))
{
    byte[] buffer = new byte[2048];
    int contentLength = fileStream.Read(buffer, 0, buffer.Length);
    while (contentLength != 0)
    {
        uploadStream.Write(buffer, 0, contentLength);
        contentLength = fileStream.Read(buffer, 0, buffer.Length);
    }
    fileStream.Close();
    uploadStream.Close();
}
```

| 设置项 | 为什么必须这样 |
|--------|---------------|
| `Credentials` | FTP 上传需要鉴权，匿名账号通常只读 |
| `Proxy = null` | 避免系统代理干扰，导致连接失败 |
| `KeepAlive = false` | 每个文件独立连接，用完即关，避免连接挂起 |
| `Method = UploadFile` | 明确是上传命令 |
| `UseBinary = true` | AB 包是二进制，**必须二进制模式**，否则会损坏文件 |
| 缓冲区 2048 | 每次读写 2KB，分块传输大文件，避免一次性加载到内存 |

### 4. 异常处理

```csharp
try { ... }
catch (Exception e)
{
    Debug.LogWarning(fileName + "上传失败" + e.Message);
}
```

上传失败（网络问题、权限不足、路径不存在等）不影响其他文件继续上传。

---

## 三、异步函数（重点段落）

### 什么是异步函数

**异步函数**是用 **`async`** 修饰、内部用 **`await`** 等待异步操作完成的方法。

```csharp
private async static void FtpUploadFile(string filePath, string fileName)
{
    await Task.Run(() => { /* 耗时操作 */ });
    // await 之后的代码，等 Task 完成后才继续执行
}
```

- `async`：告诉编译器"这个方法里会用 `await`"，编译器会把它编译成**状态机**；
- `await`：**挂起当前方法**，把控制权交还给调用方，等 Task 完成后**再回来继续执行后面的代码**；
- `await` 不会阻塞调用线程，但**异步方法内部"看起来"又是顺序执行的**。

### 如何使用

```csharp
// 方式一：把耗时操作丢到线程池
await Task.Run(() =>
{
    // 这里在线程池线程上执行，不卡主线程
    // 适合 I/O 密集、计算密集任务
});

// 方式二：等待现成的异步 API
await someAsyncMethod();   // someAsyncMethod 返回 Task / Task<T>

// 方式三：等待带返回值的
int result = await GetIntAsync();
```

**调用方**不需要 `await`，直接调用即可（异步方法会立即返回，后台继续跑）。这也是为什么 `UploadAllABFile` 里是：

```csharp
FtpUploadFile(info.FullName, info.Name);   // 不 await，异步执行
```

### 异步函数 vs 协程 vs 多线程

| 对比项 | **异步函数（async/await）** | **协程（Coroutine）** | **多线程（Thread/Task）** |
|--------|---------------------------|----------------------|--------------------------|
| **本质** | 语法糖，基于 Task 的异步编程模型 | Unity 特有的**单线程分帧**执行机制 | 操作系统级的**真正并行**执行 |
| **是否多线程** | `Task.Run` 内部用线程池，**可能多线程**；纯 `await` 现成 API 不一定 | ❌ **单线程**，同一时间只有一个协程在跑 | ✅ 真多线程 |
| **执行时机** | `await` 后面代码在 Task 完成后由**调度器**恢复 | 每帧 `yield` 后由 Unity 主循环恢复 | 独立线程，不受主线程控制 |
| **宿主依赖** | 不依赖 MonoBehaviour，任何类都能用 | **必须**挂在 MonoBehaviour 上（`StartCoroutine`） | 不依赖任何东西 |
| **能否访问 Unity API** | ⚠️ `Task.Run` 内部**不能**访问 Unity API（非主线程） | ✅ 全程主线程，随便访问 | ❌ 非主线程，**不能**访问 Unity API |
| **返回值** | `Task` / `Task<T>` | `IEnumerator`（通过 `yield return` 暂停） | 无统一返回值 |
| **异常处理** | `try-catch` 能捕获 | `try-catch` 能捕获 | 异常会直接崩线程 |
| **适用场景** | I/O、网络请求、文件读写 | 帧动画、延时、分帧加载 | CPU 密集计算、后台处理 |

### 三者关系小结

```
多线程（Task/Thread）  ← 底层执行者，真正并行
      ↑
异步函数（async/await） ← 语法层，方便地"等待"多线程任务完成
      ↑
协程（Coroutine）      ← Unity 特有，单线程"伪异步"，与多线程无关
```

- **协程 ≠ 多线程**：协程只是"分帧执行"，同一时刻还是主线程在跑，只是把一段逻辑拆到多帧；
- **异步函数 ≠ 一定多线程**：`await` 一个已经完成的 Task 不会切换线程；只有 `Task.Run` 才会丢到线程池；
- **异步函数 ≈ 协程的"进化版"**：都能"暂停-恢复"，但异步函数**不依赖 MonoBehaviour**，且可跨线程；协程**强依赖 MonoBehaviour**，全程主线程。

### ⚠️ 本代码中的特殊点

`private async static void FtpUploadFile(...)` 是 **`async void`**：

- `async void` 方法**无法被 await**，异常**无法被调用方捕获**（会直接抛到同步上下文）；
- 所以代码里用了 `try-catch` 内部兜底；
- 生产环境更推荐 **`async Task`**，让调用方可以 `await` 并统一处理异常。

---

## 四、易错点小结

| 易错点 | 说明 |
|--------|------|
| `UseBinary = false` | AB 包是二进制，ASCII 模式会损坏文件 |
| 忘了 `Credentials` | 上传需要账号密码，匿名账号一般只读 |
| `Proxy` 没设 null | 系统代理会导致 FTP 连接超时/失败 |
| `Extension == ".txt"` 太宽 | 会误传其他 txt，建议精确匹配 `ABCompareInfo.txt` |
| `async void` | 异常无法被 await 捕获，建议 `async Task` |
| `GetRequestStream()` 在 try 外 | 它本身抛异常时不会被 catch，建议也放进 try |
| 用浏览器测 FTP | 新版浏览器已不支持 `ftp://`，用资源管理器或 FileZilla |
| 非主线程调 Unity API | `Task.Run` 内部不能碰 `Debug.Log` 以外的 Unity API（`Debug.Log` 本身是线程安全的，但大部分 API 不是） |

---

## 五、一句话总结

> **遍历 AB 目录，筛出无后缀的 AB 包和 `ABCompareInfo.txt`，用 `FtpWebRequest` 走"创建请求 → 设凭证 → 配置二进制上传 → 流式读写"四步上传到 FTP；上传过程用 `async/await + Task.Run` 丢到后台，不卡编辑器主线程。异步函数是"语法糖 + 状态机"，`Task.Run` 内部才是真多线程，协程则是 Unity 单线程的伪异步，三者定位不同、不可混为一谈。**