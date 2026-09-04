## 元表

#### 概念
任何表变量都可以作为另一个表变量的元表  
任何表变量都可以由自己的元表 （父亲） 
当子表中进行一些特定操作时，会执行元表的内容

#### 设置元表
```lua
meta = {}

myTable = {}

--第一个参数 子表

--第二个参数 元表

setmetatable(myTable, meta)

print(myTable) --table: 0x0000000001e3f8c0
```
#### __tostring与__call
```lua
print("----特定操作-----")

print("__tostring")

meta2 = {

    --当子表要被当作字符串使用时，调用这个函数

    __tostring = function(t)

        return t.name

    end,

    --当子表要被当作字符串使用时，调用这个函数

    --只有实现了__call方法，子表才能像函数一样被调用

    --第一个参数是表自身，第二个参数是调用时传入的参数

    __call = function(t, ...)

        print("调用了元表的__call方法")

        for i,v in ipairs({...}) do

            print(i,v)

        end

    end

}

myTable2 = {name = "你好"}

setmetatable(myTable2, meta2)

print(myTable2) --你好，调用了元表的__tostring方法

  

myTable2("参数1", "参数2")  --调用了元表的__call方法
```
## 运算符重载
```lua
meta3 ={

    --相当于运算符重载，当子表使用+运算符时，调用这个函数

    __add = function(t1, t2)

        return t1.age + t2.age

    end,

    --运算符-

    __sub = function(t1, t2)

        return t1.age - t2.age

    end,

    --运算符*

    __mul = function(t1, t2)

        return t1.age * t2.age

    end,

    __div = function(t1, t2)

        return t1.age / t2.age

    end,

    __mod = function(t1, t2)

        return t1.age % t2.age

    end,

    __pow = function(t1, t2)

        return t1.age ^ t2.age

    end,

    __eq = function(t1, t2)

        return t1.age == t2.age

    end,

    __lt = function(t1, t2)

        return t1.age < t2.age

    end,

    __le = function(t1, t2)

        return t1.age <= t2.age

    end,

    __concat = function(t1, t2)

        return t1.age .. t2.age

    end

  

}

myTable3 = {name = "你好", age = 18}

setmetatable(myTable3, meta3)

metatable4 = {name = "你好", age = 20}

setmetatable(metatable4, meta3)

  

print(myTable3 + metatable4)  --38

print(myTable3 - metatable4)  ---2

print(myTable3 * metatable4)  --360

print(myTable3 / metatable4)  --0.9

print(myTable3 % metatable4)  --18

print(myTable3 ^ metatable4)  --18^20

  

--如果要比较两个表是否相等，必须是同一个元表才会准确调用方法

print(myTable3 == metatable4)  --false

print(myTable3 < metatable4)  --true

print(myTable3 <= metatable4)  --true

print(myTable3 .. metatable4)  --1820
```

#### __index与__newindex