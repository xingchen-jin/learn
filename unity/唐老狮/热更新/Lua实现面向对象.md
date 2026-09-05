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

## 继承

```lua
function Object:subClass(className)

    _G[className] = {}

    local obj = _G[className]
    --子类定义base，指向父类(多态)
    obj.base = self
    self.__index = self

    setmetatable(obj, self)

end

Object:subClass('Person')

print(Person.id) -- 1

local p1 = Person:new()

print(p1.id) -- 1

p1.id = 2

  

Object:subClass('Monster')

local m1 = Monster:new()

print(m1.id) -- 1
```

## 多态
```lua
Object:subClass('GameObject')

GameObject.posX = 0;

GameObject.posY = 0;

function GameObject:Move()

    self.posX = self.posX + 1

    self.posY = self.posY + 1

    print('GameObject Move to: ' .. self.posX .. ', ' .. self.posY)

end

GameObject:subClass('Player')

Player:Move()

function Player:Move()

    --不要使用:Move()，因为:会把base传给move，而base是父类

    --self.base:Move()

    self.base.Move(self)

    print('Player Move')  

end

local p1 = Player:new()

p1:Move()

p1:Move()

local p2 = Player:new()

p2:Move()
```
