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

