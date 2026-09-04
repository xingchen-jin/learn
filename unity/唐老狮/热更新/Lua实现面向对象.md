## 封装

```lua
--面向对象

print('**********封装***********')

Object = {}

Object.id = 1

function Object:getId()

    return self.id

end

function Object:new()

    local obj = {}

    setmetatable(obj, self)

    --自身找不到变量时，就找元表的index指向的表中去找

    self.__index = self

    return obj

end

  

local myObj = Object:new()

print(myObj.id) -- 1

myObj:getId() -- 1

  

--对空表中声明一个新属性

myObj.id = 2

print(myObj.id) -- 2
```

