# 值类型和引用类型
![](assets/Pasted%20image%2020260628134906.png)
堆：较大的区域，主要存放new的实例
栈：较小的区域，较快,存放值类型

值类型：包括基本数据类型，枚举类型，结构体。通过拷贝复制，存放在栈

引用类型：存放指向该数据的地址，**引用类型本身存放在栈（stack）上，而所指向的数据存储在堆（Heap）**。所有引用类型都继承自System.Object
常见有类，字符串，数组，**委托**
### string
1，不变性，当进行修改操作会创建新的变量
2，String Interning ：C#内置了string interning机制。它很像一个“字符串池”，当我们想要引用一个字符串时，如果池中已经有内容相同的字 符串，我们可以直接访问该已存在的字符串，不需要创建新的字符串。
```Csharp
string str1 = "Hello"; string str2 = "Hello"; bool areEqual = Object.ReferenceEquals(str1, str2);//结果为true
```
### ref关键字（类似c里面的指针传递）
C#中函数的参数可以被ref关键字修饰。它允许我们在传参时使用**变量的引用**，而非**变量的值**。 （一般来说，我们向函数中传参数时都是复制了一份变量的值并传入的。这也是为什么函数内的变量通常和函数外的变量没关系。）
```Csharp
int a = 1; AddOne(ref a); print(a); // a=2
 …… 
 void AddOne(ref int a){ a++; }
```

有了ref关键字后，参数“a”是通过引用 传递的。于是函数内的“a”和函数外的“a” 是一个东西了。

### 装箱与拆箱
在值类型与引用类型之间作转换。比较**消耗性能**
装箱：值 to 引用
拆箱：引用 to 值

### GC：垃圾回收机制

C#提供了垃圾回收器（Garbage Collector）来帮助我们清理不需要的内存。

堆中的内存是分为3个**Generation**的！ 

Generation 0：主要为临时变量、刚刚被创建的（较小的）类的实例。**这里的清理频率是最高的。**

Generation 1：这是一个缓冲区，从gen0中“存活下来”的物体会被放入gen1。（这说明它们更常被使用） 

Generation 2：从gen1中“存活下来”的物体会被放入gen2。这里主要为长期需要的变量（static变量等），**清理的频率最低**

当第k个generation被GC清理时，所有**小于k**的generation也都会被一并清理。 
所以当GC清理gen2的时候可以被认为是一个 “**完整的垃圾回收**”。

#### 垃圾回收过程

![](assets/Pasted%20image%2020260628143444.png)![](assets/Pasted%20image%2020260628143650.png)
#### 减少垃圾产生
![](assets/Pasted%20image%2020260628144243.png)

# C#本质
![](assets/Pasted%20image%2020260628144705.png)
![](assets/Pasted%20image%2020260628145034.png)
