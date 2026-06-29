# 泛型
提供使方法中的变量类型可以在方法调用中确立。

我们可以在圆括号后使用where关键字对类型变量作出一些**限制**:
where T: struct (T必须为值类型) 
where T : class (T必须为引用类型) 
where T : new() (T必须有一个无参数的构造函数) 
where T :base class name (T是继承自一个基类的) 
where T : interface name(T实现了一个接口)
# 委托 Delegate

delegate <返回值类型> <委托名>(<参数列表>);
delegate int Calculate(int x, int y);
这样，我们就定义了一个委托类型。这个类型的变量中可以存放**返回值为int**且 **接受两个int参数**的函数。
```csharp
Calculate cal; //声明委托类型的变量 
cal = AddInt; //将AddInt函数装进变量 
int sum = cal(6, 7); //使用委托
```

### Multicasting
一个委托变量中也可以存储/调用多个函数。使用+=和-=进行增加删除函数
当一个装有多个函数的委托变量被调用时，其中的函数会被依次调用，且： 
（1）如果参数中有引用类型的变量，该变量会依次受到这些函数的影响。 
（2）如果函数有返回值，则最终返回最后一个函数的返回值。 
一个委托类型中的函数们叫做调用列表（Invocation List）。我们可以通过GetInvocationList()获取它： 
``cal.GetInvocationList(); 
``cal.GetInvocationList().Length;

### 事件event
对委托的一层封装，仅支持+=和-=，禁止=直接赋值

### Action和Func
![](assets/Pasted%20image%2020260628204934.png)

# 字段和属性
属性（Property）提供了这个功能。属性可以理解为一个对字段的封装，它能给一个字段添加一些功能：比如读写限制、读 写时执行函数等等
一个属性具有两个组成部分：getter与setter。它们分别定义了读、写该属性的行为：
```cSharp
private int speed = 5; 
public int Speed{ 
	get{return speed;} 
	set{speed = value;}
}
```
属性的Getter定义了读取该属性的行为。Getter**必须有返回值**。
属性的Setter定义了写入该属性的行为。Setter有一个看不见的参数 “value”。**value**代表我们要写入的那个值。
```csharp
private double _seconds; 
public double Hours{ 
	get { return _seconds / 3600; }
	set{ 
		if (value < 0 || value > 24) throw new ArgumentOutOfRangeException(nameof(value), "The valid range is between 0 and 24."); 
		_seconds = value * 3600; SC-102 L02 
	}
}
```

**自动属性：** ``public int Speed { get; set; }``会自动生成一个字段
``public int Speed { get; protected set;}``可以给getter，setter添加访问权限。
如果只有get那么本类也不具备写入权限.

# 接口 Interface
接口（Interface）定义了一系列类的规范。接口中可以包括方法、属性、事件等成员，而这些成员**必须被其子类实现**。 比如我们想在游戏里添加“可被攻击”的一系列物体。它们是有一些共同的且必须拥有的特点的，比如受击方法、生命值等。 于是我们可以建立一个接口将它们统一起来。
```csharp
public interface IDamageable {
	float hp {get; set;} //属性：生命值 
	void OnDamage(float value);//方法：受击
}
```

# 部分类
部分类允许我们把同一个类写在多个文件里，每个文件提供类的一部分。 定义时使用partial关键字
# lambda表达式
### expresion lambda
``(参数列表) => (返回的表达式)``比如`` (x,y) => x*y``
使用lambda表达式时把它当作一个委托类型的变量即可，
``Func mul = (x,y) => x*y``

### statement lambda
``(参数列表) => { 执行的程序语句 }``
```
Action greet = name => 
{ 
	string greeting = $"Hello {name}!";
	 Console.WriteLine(greeting); 
};
greet("World");
```

# 隐式类型
隐式类型允许我们在定义变量时不直接写出其类型。使用var关键字定义隐式类型变量
### foreach循环
类似for循环，foreach循环是一种遍历集合的方式。与隐式类型搭配时，foreach循环会变得很方便
```csharp
foreach(var item in list){ 
	item += 5;
}
```
