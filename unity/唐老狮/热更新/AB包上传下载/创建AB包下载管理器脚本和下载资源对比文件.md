

## 一、这个类做什么

`ABUpdateMgr` 负责热更新的第一步：**从 FTP 服务器下载资源对比文件 `ABCompareInfo.txt`，解析出每个远端 AB 包的「名字、大小、MD5」，存入字典**，供后续与本地 AB 包对比、决定哪些需要更新。

---

## 二、ABInfo：AB 包信息类（内部类）

```csharp
private class ABInfo
{
    public string name;   // AB包名字
    public long   size;   // AB包大小
    public string md5;    // AB包md5值

    public ABInfo(string name, string size, string md5)
    {
        this.name = name;
        this.size = long.Parse(size);   // 字符串 → long
        this.md5  = md5;
    }
}
```

### 运算符重载：用 MD5 判断两个 ABInfo 是否“相同”

```csharp
public static bool operator ==(ABInfo a, ABInfo b) => a.md5 == b.md5;
public static bool operator !=(ABInfo a, ABInfo b) => a.md5 != b.md5;
```

- **重载 `==` / `!=` 后，比较两个 ABInfo 就等价于比较它们的 MD5**；
- 后续对比“本地 AB 包 vs 远端 AB 包”时，直接 `localInfo == remoteInfo` 即可判断是否同一个版本；
- **MD5 相同 → 文件内容一致 → 不需要更新；MD5 不同 → 需要下载**；
- ⚠️ 重载 `==` 时通常要同时重写 `GetHashCode()`（这里没写，学习阶段可忽略，实际项目建议补上，避免在 Dictionary/HashSet 中行为异常）。

---

## 三、单例模式（挂在 GameObject 上）

```csharp
private static ABUpdateMgr _instance;
public static ABUpdateMgr Instance
{
    get
    {
        if (_instance == null)
        {
            GameObject go = new GameObject("ABUpdateMgr");
            _instance = go.AddComponent<ABUpdateMgr>();
            // 过场景不保留，一般只在加载场景时加载一次
        }
        return _instance;
    }
}

private void OnDestroy()
{
    _instance = null;   // 销毁时清空，避免野引用
}
```

| 特点 | 说明 |
|------|------|
| **懒加载** | 第一次访问 `Instance` 才创建 GameObject 并挂脚本 |
| **不跨场景** | 没加 `DontDestroyOnLoad`，过场景自动销毁（注释也说明了） |
| **OnDestroy 清空** | 对象销毁后 `_instance` 置 null，下次访问会重新创建 |

> 对比常见的 `DontDestroyOnLoad` 单例：这里**刻意不常驻**，因为热更新管理器一般只在“加载场景”里用一次。

---

## 四、核心流程：DownLoadCompareFile()

```csharp
public void DownLoadCompareFile()
{
    // ① 从 FTP 下载对比文件到 persistentDataPath
    DownLoadFile("ABCompareInfo.txt", Application.persistentDataPath + "/");

    // ② 读取下载后的文件内容
    string compareInfo = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo.txt");

    // ③ 按行拆分
    string[] lines = compareInfo.Split('\n');

    // ④ 每行再按空格拆成「名字 大小 MD5」，存入字典
    string[] infos = null;
    for (int i = 0; i < lines.Length; i++)
    {
        infos = lines[i].Split(' ');
        _abInfoDic.Add(infos[0], new ABInfo(infos[0], infos[1], infos[2]));
    }
}
```

### 数据流

```
FTP 服务器 ABCompareInfo.txt
        │ DownLoadFile
        ▼
Application.persistentDataPath/ABCompareInfo.txt
        │ File.ReadAllText
        ▼
"lua 1024 abc123...\nmodel 2048 def456...\n..."
        │ Split('\n')  按行
        ▼
["lua 1024 abc123...", "model 2048 def456...", ...]
        │ Split(' ')   按空格
        ▼
["lua","1024","abc123..."] → new ABInfo → _abInfoDic["lua"] = ABInfo
```

### 存储结构

```csharp
private Dictionary<string, ABInfo> _abInfoDic = new Dictionary<string, ABInfo>();
```

- **key**：AB 包名（如 `"lua"`、`"model"`）；
- **value**：`ABInfo`（含 size、md5）；
- 后续用本地 AB 包名去这个字典里查，就能拿到远端版本信息做对比。

### `Application.persistentDataPath` 是什么

- Unity 的**可读写持久化目录**（各平台不同，Windows 下是 `C:\Users\用户名\AppData\LocalLow\公司名\产品名`）；
- 热更新下载的文件应该放这里，**不能放 `StreamingAssets`**（那里只读，且 Android 下在压缩包内）；
- 每次启动热更新前，通常要检查这个目录下是否已有文件，避免重复下载。

---

## 五、DownLoadFile()：FTP 下载（与上传对称）

```csharp
private void DownLoadFile(string fileName, string localPath = "")
{
    // ① 创建请求：ftp://IP/服务器路径/文件名
    FtpWebRequest request = (FtpWebRequest)FtpWebRequest.Create(
        new Uri("ftp://10.100.209.199/AB/PC/" + fileName));

    // ② 凭证
    request.Credentials = new NetworkCredential("XC", "XC12345");

    // ③ 配置
    request.Proxy = null;
    request.KeepAlive = false;
    request.Method = WebRequestMethods.Ftp.DownloadFile;   // 下载命令
    request.UseBinary = true;                               // 二进制

    // ④ 获取响应流，写入本地文件
    FtpWebResponse res = (FtpWebResponse)request.GetResponse();
    Stream downLoadStream = res.GetResponseStream();

    using (FileStream fileStream = File.Create(localPath + fileName))
    {
        byte[] buffer = new byte[2048];
        int contentLength = downLoadStream.Read(buffer, 0, buffer.Length);
        while (contentLength != 0)
        {
            fileStream.Write(buffer, 0, contentLength);
            contentLength = downLoadStream.Read(buffer, 0, buffer.Length);
        }
    }
}
```

### 与「上传」的关键区别

| 步骤 | 上传 | **下载** |
|------|------|---------|
| Method | `UploadFile` | **`DownloadFile`** |
| 获取流 | `request.GetRequestStream()`（写入服务器） | **`request.GetResponse().GetResponseStream()`（从服务器读）** |
| 流方向 | `FileStream.Read` → `uploadStream.Write` | **`downLoadStream.Read` → `FileStream.Write`** |
| 本地文件 | `File.OpenRead` | **`File.Create`** |

**记忆口诀**：上传是“本地 → 请求流”，下载是“响应流 → 本地”。

### 配置项含义（与上传一致）

| 配置 | 说明 |
|------|------|
| `Credentials` | 下载鉴权。注释提到：**匿名账号可以省略，但实际开发不建议开匿名** |
| `Proxy = null` | 避免系统代理干扰 |
| `KeepAlive = false` | 每次请求独立连接 |
| `UseBinary = true` | 二进制传输，避免文件损坏 |

---

## 六、易错点小结

| 易错点 | 说明 |
|--------|------|
| 下载目录用了 `StreamingAssets` | 那里只读（Android 还在压缩包内），必须用 `persistentDataPath` |
| `Split('\n')` 未处理空行 | 对比文件末尾若有多余换行，会拆出空字符串，`infos[1]` 越界；建议用 `StringSplitOptions.RemoveEmptyEntries`（注释里也给了这种写法） |
| `_abInfoDic.Add` 重复 key | 同一文件调用两次 `DownLoadCompareFile` 会抛 `ArgumentException`，应改用 `_abInfoDic[name] = info` 或先 `Clear()` |
| `File.ReadAllText` 时机 | `DownLoadFile` 是同步的，能保证下载完再读；如果改成异步，必须等下载完成再读，否则读到空/旧文件 |
| 运算符重载不重写 `GetHashCode` | 放进 Dictionary/HashSet 时行为可能异常，实际项目应补 |
| FTP 地址硬编码 | `10.100.209.199`、账号密码写死在代码里，实际项目应做成配置 |
| `OnDestroy` 未清空 | 会留野引用，这里已处理 |
| 下载失败未清理半截文件 | `File.Create` 已创建文件，失败后可能留下空文件，下次启动会误判“已下载” |

---

## 七、一句话总结

> **`ABUpdateMgr` 用 FTP 把 `ABCompareInfo.txt` 下载到 `persistentDataPath`，`ReadAllText` 读出内容，按 `\n` 拆行、按空格拆成「名字 大小 MD5」，构建成 `Dictionary<string, ABInfo>`；`ABInfo` 重载 `==`/`!=` 用 MD5 比较，后续本地与远端对比时直接判等即可。下载与上传的区别只在 `Method`、流的获取方式（`GetResponseStream` vs `GetRequestStream`）和读写方向，其余配置完全对称。**