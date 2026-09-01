#### 下标从1开始
#### 注释
``--[[]]  多行注释  
``--单行注释

## 运算符
#### 算术运算符
![](assets/Pasted%20image%2020260831204240.png)

#### 条件运算符
![](assets/Pasted%20image%2020260831204519.png)

#### 逻辑运算符
and逻辑与  
or 逻辑或
not 取反  
遵循逻辑运算的“短路”  
#### 位运算符
不支持位运算  
#### 三目运算符
不支持  
## 条件分支语句
#### 单条分支
```
if 布尔值 then 

end 
```

#### 双分支
```
if 布尔值 then 
	语句
else
	语句
end
``` 
#### 多分支
```
if 布尔值 then 
	语句
elseif 布尔 then
	语句
elseif 布尔 then
	语句
else
	语句
end
```

#### lua没有switch

## 循环
#### while
```
while 条件 do
	代码
end
```

#### do while
```
repeat
	代码
until 满足条件跳出条件（结束时的条件）
```
#### for
```lua
for i=1,5 do    --默认递增
	print(i)
end
--逐行输出1,2,3,4,5

for i=1,5,2 do    --设定步长
	print(i)
end
--1，3，5
```

## 函数
#### 无参无返回
```lua
--function 函数名（）
--end 
	--F1()不能在上面调用
function F1()
	print("F1函数")
end
F1()
--a = function()
--end
--类似c#的委托与事件
F2 = function()
	print()
end
F2()

```
#### 有参
```lua
function F1(a)
	print(a)
end
F1(1)
F1("123")
F1(true)
F1() --传nil
F1(1,2,3) --个数不匹配，补充nil或丢弃。输出1
```
#### 有返回
```lua
--单返回值
function F1(a)
	return a
end
temp = F1("123")
print(temp)
--123

--多返回值
function F2(a)
	return a,"123",true
	
end
temp1,temp2,temp3 = F2(1)

```
#### 函数类型
function
#### 函数重载
不支持重载，重名，默认调用最后一个声明的函数

#### 变长参数
```lua
function F1(...)
	--变长参数使用一个表存起来
	arg = {...}
	for i=1,#arg do
		print(arg[i])
	end
end
F1(1,"123",true,4,5,6)


```
#### 函数嵌套

```lua
function F8()
	return function()
		print("123")
	end
end
f9 = F8()
f9()
--等价于
function F8()
	F9 = function()
		print("123")
	end
	return F9
end
f9 = F8()
f9()
```
##### 闭包
```lua
function F9(x)
	--改变传入参数的生命周期
	return function(y)
		return x+y
	end
end
f10 = F9(2)
print(f10(3))
-- 5
```


## 表 table

所有基础的复杂类型都是table比如数组之类
索引从1开始
#### 数组
``a = {1,2,3,4,"123",true,nil}
``print(a[1])  --1
print(#a)   --6,忽略nil以及之后的长度
#### 数组遍历
for i =1,#a do
	print(a[i])
end
#### 二维数组
``a = {{1,2,3},{4,5,6}}
``print(a[1][1]) --打印第一个
#### 二维数组遍历
```
for i=1,#a do
	b = a[i]
	for j=1,#b do
		print(b[j])
	end
end
```
#### 自定义索引

``aa = {[0] = 1,2,3,[-1]=4,5}
``print(aa[1]) --1
``print(aa[-1]) --4
``print(#aa) --3 从1开始计数
未定义索引挨个排号，其实自定义索引跳过一格或连续跳过相同个数？，继续计数，

## 迭代器
#### 迭代器遍历
主要用于遍历表，#得到长度并不准确
``a = {[0] = 1,2,[-1]=3,4,5，[5] = 6}
可以直接遍历键
#### ipairs
从1往后开始遍历，且只能找到连续索引的键
```lua
for i,k in ipairs(a) do
	print(i.."_"..k)
end
--1_2
--2_4
--3_5
```
#### pairs
找到所有键，通过键得到值
```lua
for i,v in pairs(a) do
	print(i.."_"..v)
end
--输出
1_2
2_4
3_5
0_1
5_6
-1_3
```

## table实现字典
#### 字典声明
```lua
-- 声明
a = {["name"] = "XC", ["age"] = 18, ["sex"] = "male"}
print(a["name"])
print(a.name)  --可以通过.成员变量访问，但不能时数字
print(a["1"])

--改值
a["name"] = "Xingchen"
print(a.name) --xingchen

--新增

a["声明"] = "这是一个声明"
print(a["声明"]) --这是一个声明

--删除

a["声明"] = nil
print(a["声明"]) --nil
```
#### 遍历
```lua
--遍历
--模拟字典，遍历一定要用pairs
for k,v in pairs(a) do
    print(k.."_"..v)
end

--遍历键
for k in pairs(a) do
    print(k)
end

--遍历值（实际上还是遍历了键）
for _,v in pairs(a) do
    print(v)
end
```


## table实现类
Lua中默认没有面向对象，需要自己实现
```Lua
--成员变量，成员函数

Student =

{

    name = "张三",

    age = 18,

    sex = "男",

    Up = function ()

        --想要在函数中访问成员变量，必须使用表名.成员变量

        print(Student.age)

        print("我成长了")

    end,

    Learn = function (t)

        print(t.name)

        print("我学习了")

    end

}

--C#要是使用类，实例化对象new，静态直接点

--Lua中更像是一个类中有很多静态变量和函数

print(Student.name)
Student.Up()

--Lua中点与冒号区别
--点（.）用于访问表的成员，通常用于静态方法或变量
--冒号（:）会自动将表作为第一个参数传递
Student.Learn(Student) --传入表名
Student:Learn()

Student.Speak = function(Student)
    print(Student.name.."我说话了")
end
Student:Speak()
--函数声明的第三种方式
function Student:Eat()
    --self表示默认传入的第一个参数
    print(self.name.."吃饭了")
end

Student:Eat()
```

## table的公共操作
```Lua
t1 = {{a=1,b=2},{a=3,b=4}}

t2 = {a=-1,b=-2}

table.insert(t1,t2)

--移除最后一个索引的内容

table.remove(t1)

--移除指定索引的内容

table.remove(t1,1)

  

t3 = {5,3,4,1,2}

--排序,默认是升序

table.sort(t3)

for _,v in ipairs(t3) do

    print(v)

end

--传入一个函数自定义规则，改例子为降序

table.sort(t3,function(a,b)

    return a>b

end)

  

--连接函数，用于拼接表中元素，返回字符串

tb = {"a","b","c"}

print(table.concat(tb,"_")) --a_b_c
```

## 多脚本执行

#### 全局变量和本地变量
```lua
--全局变量

a= 1

b = 123

for i =1,2 do

    c = 1

    local d = 2

end

print(c) --1

print(d) --nil

  

fun = function()

    tt = 1

end

print(tt) --nil

fun() --调用函数

print(tt) --1

  

local e = 1

print(e) --1
```

#### 多脚本执行
```lua
--多交本执行

--require("脚本名")
--require 执行一个脚本时，可以在脚本末尾返回一个值，require会将这个值返回给调用者
local testLA = require("Require_Text")
print(testA) --123
print(testLocalA) --nil
print(testLA) --456
require("Require_Text") --不会重复加载

--脚本卸载
print(package.loaded["Require_Text"]) --true
package.loaded["Require_Text"] = nil
print(package.loaded["Require_Text"]) --nil

```
#### 大G表
```lua
--大G表
--_G是一个全局表，里面存放了所有的全局变量
for k,v in pairs(_G) do
    print(k,v)
end
--加了local的变量不会被放入_G表中
```