## 一、这个工具做什么

在编辑器菜单栏新增 **`AB包工具 → 创建对比文件`**，一键扫描 `Assets/ArtRes/AB/PC/` 目录下的所有 AssetBundle，把每个 AB 包的**文件名、大小、MD5 码**拼接成一行，最终写入 `ABCompareInfo.txt`。热更新时客户端拿这份文件和服务器对比，就能判断哪些 AB 需要更新。

---

## 二、关键知识点

### 1. 编辑器扩展入口：`[MenuItem]`

```csharp
[MenuItem("AB包工具/创建对比文件")]
public static void CreateABCompareFile()
```

- 加在 **`static` 方法**上，Unity 编辑器会在顶部菜单生成对应路径的菜单项；
- 点击菜单项即执行该方法；
- 所在脚本需放在 `Editor` 目录（或程序集）下，且 `using UnityEditor;`。

### 2. 获取文件夹信息：`Directory.CreateDirectory` + `DirectoryInfo`

```csharp
DirectoryInfo directory = Directory.CreateDirectory(Application.dataPath + "/ArtRes/AB/PC/");
FileInfo[] fileInfos = directory.GetFiles();
```

| 类型 | 作用 |
|------|------|
| `DirectoryInfo` | 目录信息对象 |
| `Directory.CreateDirectory(path)` | **目录不存在则创建，存在则直接返回**（不会报错），顺带拿到 `DirectoryInfo` |
| `GetFiles()` | 返回目录下所有文件的 `FileInfo[]`（**不含子目录内的文件**） |

### 3. 遍历文件并筛选 AB 包

```csharp
foreach (FileInfo info in fileInfos)
{
    if (info.Extension == "")   // 无后缀的才是 AB 包
    {
        ...
    }
}
```

`FileInfo` 常用属性：

| 属性 | 含义 |
|------|------|
| `info.Name` | 文件名（含后缀，如 `lua`、`ABCompareInfo.txt`） |
| `info.FullName` | 完整路径 |
| `info.Length` | 文件大小（字节） |
| `info.Extension` | 文件后缀（含点，如 `.txt`；**无后缀时为空字符串 `""`**） |

> **筛选依据**：Unity 打包出来的 AB 文件**没有后缀**，而 `ABCompareInfo.txt`、`.manifest` 等都有后缀，所以用 `Extension == ""` 精准过滤。

### 4. 拼接对比信息

```csharp
abCompareInfo += info.Name + " " + info.Length + " " + GetMD5(info.FullName);
abCompareInfo += "\n";
```

每行格式：

```
AB包名 文件大小 MD5码
```

> **MD5 计算代码见《MD5 笔记》**（`GetMD5` 方法内容相同，此处不重复）。

### 5. 去掉最后一个换行符

```csharp
abCompareInfo = abCompareInfo.Substring(0, abCompareInfo.Length - 1);
```

循环里每行末尾都加了 `\n`，最后一行会多出一个空行；用 `Substring(0, Length - 1)` 砍掉最后一个字符即可。

> ⚠️ 前提是**至少扫描到一个 AB 包**，否则 `abCompareInfo` 为空字符串，`Substring(0, -1)` 会抛异常。实际项目建议加 `if (abCompareInfo.Length > 0)` 保护。

### 6. 写入文件 + 刷新编辑器

```csharp
File.WriteAllText(Application.dataPath + "/ArtRes/AB/PC/ABCompareInfo.txt", abCompareInfo);
AssetDatabase.Refresh();
```

| API | 作用 |
|-----|------|
| `File.WriteAllText(path, content)` | 把字符串写入文件（**覆盖**写，不存在则创建） |
| `AssetDatabase.Refresh()` | 通知 Unity 重新扫描 `Assets` 目录，让新生成的文件出现在 Project 窗口里 |

> 没有 `AssetDatabase.Refresh()`，文件虽然已写进磁盘，但 Unity 编辑器界面不会立即显示。

---

## 三、整体流程

```
点击菜单 "AB包工具/创建对比文件"
  ├─ Directory.CreateDirectory(...)        // 确保目录存在，拿 DirectoryInfo
  ├─ directory.GetFiles()                  // 拿到所有 FileInfo
  ├─ foreach 遍历
  │    └─ info.Extension == "" ?           // 只处理 AB 包
  │         ├─ 记录 Name
  │         ├─ 记录 Length
  │         └─ GetMD5(FullName)            // 算 MD5（见 MD5 笔记）
  │         拼成一行 → 追加 "\n"
  ├─ Substring 去掉最后一个 "\n"
  ├─ File.WriteAllText(...)                // 写入 ABCompareInfo.txt
  └─ AssetDatabase.Refresh()               // 刷新编辑器
```

---

## 四、易错点小结

| 易错点 | 说明 |
|--------|------|
| 忘记 `using UnityEditor;` | `[MenuItem]`、`AssetDatabase` 都在 `UnityEditor` 命名空间下 |
| 脚本不在 `Editor` 目录 | 打包时会报错（`UnityEditor` 只能在编辑器下用） |
| `Extension` 判断写错 | 无后缀是 `""`，不是 `null`；`.txt` 是带点的小写字符串 |
| 空目录调用 `Substring` | 至少要有一个 AB 包，否则 `Length - 1` 越界 |
| 忘记 `AssetDatabase.Refresh` | 文件已生成但 Project 窗口看不到，容易误以为失败 |
| `GetFiles()` 不递归 | 只扫当前目录，**不含子文件夹**；AB 分散在子目录时要改用 `GetFiles(SearchOption.AllDirectories)` |

---

## 五、一句话总结

> **`[MenuItem]` 提供菜单入口，`DirectoryInfo.GetFiles()` 拿到所有文件，用 `Extension == ""` 筛出 AB 包，每行拼成「名字 大小 MD5」，`Substring` 去掉末尾换行，`File.WriteAllText` 写入 `ABCompareInfo.txt`，最后 `AssetDatabase.Refresh()` 刷新编辑器。** MD5 计算部分见《MD5 笔记》，此处复用。