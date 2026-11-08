## 一、MD5 码是什么

- **全称**：MD5（Message-Digest Algorithm），即 **MD5 信息摘要算法**。
- **性质**：一种广泛使用的**密码散列函数**。
- **输出**：生成 **128 位（16 个字节）** 的散列值。
- **用途**：确保信息的**完整一致性**。
- **固定长度**：无论传入的数据有多大，经过 MD5 计算后都会生成**固定长度（128 位 / 16 字节）** 的信息摘要值。
- **确定性**：相同的数据，每次经过 MD5 算法计算后的结果**完全一样**。
- **敏感性**：数据一旦变化，MD5 码就会发生变化。
- **结论**：可以把 MD5 码作为**文件的唯一标识**，通过它判断文件内容是否发生变化。

---

## 二、MD5 码在热更新资源对比文件中的作用

- 仅通过**资源名**或**资源大小**，无法判断资源是否更新（比如内容改了但大小没变）。
- 因此需要利用 MD5 的**唯一性**来判断资源是否更新：
  - 对比新旧文件的 MD5 码，不一致 → 说明资源有更新，需要下载。

---

## 三、C# 中获取文件的 MD5 码

### 关键类

| 类型 | 说明 |
|------|------|
| `MD5` | MD5 类（抽象基类） |
| `MD5CryptoServiceProvider` | MD5 加密服务提供商类（具体实现） |
| `FileStream` | 文件流类（数据持久化四部曲，二进制部分讲过） |
| `StringBuilder` | 字符串拼接类（C# 进阶讲过） |

### 流程（三步）

1. 根据文件路径，获取文件的**流信息**（`FileStream`）。
2. 利用 `md5` 对象根据流信息，计算出 **MD5 码（字节数组形式）**（16 字节）。
3. 将字节数组形式的 MD5 码转为 **16 进制字符串**（缩短长度、便于比较）。

### 核心代码

```csharp
private string GetMD5(string filePath)
{
    using (FileStream file = new FileStream(filePath, FileMode.Open))
    {
        // 声明 MD5 对象用于生成 MD5 码
        MD5 md5 = new MD5CryptoServiceProvider();
        // 根据流信息计算 MD5 码（16 字节数组）
        byte[] md5Bytes = md5.ComputeHash(file);

        file.Close();

        // 字节数组 → 16 进制字符串
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < md5Bytes.Length; i++)
        {
            sb.Append(md5Bytes[i].ToString("x2"));   // "x2" = 两位小写十六进制
        }
        return sb.ToString();
    }
}
```

### 调用示例

```csharp
print(GetMD5(Application.dataPath + "/ArtRes/AB/PC/lua"));
print(GetMD5(Application.dataPath + "/ArtRes/AB/PC/model"));
```

---

## 四、一句话总结

> **MD5 是固定 128 位（16 字节）的摘要算法**，相同数据结果不变、数据变化结果必变，因此适合作为**文件唯一标识**；热更新中通过对比文件的 MD5 码判断资源是否更新；C# 中用 `FileStream` + `MD5CryptoServiceProvider.ComputeHash` 得到 16 字节数组，再用 `StringBuilder` 按 `"x2"` 逐字节转成 16 进制字符串即可。