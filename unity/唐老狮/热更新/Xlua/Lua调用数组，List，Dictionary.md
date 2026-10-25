```lua
print("********** Lua调用数组 **********")

  

local obj = CS.Lesson3() --创建C#类对象

  

-- Lua调用数组

--长度 userdata

print(obj.arr.Length)

print(obj.arr[0]) --虽然Lua数组下标从1开始，但调用C#数组时，仍然是从0开始

  

-- 遍历

-- 最大值取得到，所以一定要长度-1

print("遍历数组")

for i = 0, obj.arr.Length - 1 do

    print(obj.arr[i])

end

  

--创建新的数组

print("创建新的数组")

local arr2 = CS.System.Array.CreateInstance(typeof(CS.System.Int32), 5)

print(arr2.Length)

for i = 0, arr2.Length - 1 do

    arr2[i] = i + 1

    print(arr2[i])

end

  

print("********** Lua调用List **********")

obj.list:Add(6)

obj.list:Add(7)

print("List长度：" .. obj.list.Count)

  

for i = 0, obj.list.Count - 1 do

    print(obj.list[i])

end

print(obj.list)

  

-- 创建List对象

--老版本

local list2 = CS.System.Collections.Generic["List`1[System.Int32]"]()

print(list2)

list2:Add(1)

print(list2[0])

-- v2.1.12以后

local List_String = CS.System.Collections.Generic.List(CS.System.String)

local list3 = List_String()

list3:Add("233")

print(list3[0])

  

print("********** Lua调用Dictionary **********")

  

obj.dic:Add("d", 4)

print(obj.dic['d'])

  

for k, v in pairs(obj.dic) do

    print(k, v)

end

  

--创建字典对象

--相当于得到一个Dictionary<string, Vector3>类型，再调用构造函数创建对象

local Dic_Int_Vector3 = CS.System.Collections.Generic.Dictionary(CS.System.Int32, CS.UnityEngine.Vector3)

local dic2 = Dic_Int_Vector3()

dic2:Add(1, CS.UnityEngine.Vector3(1, 2, 3))

print(dic2[1])

  

local Dic_String_Vector3 = CS.System.Collections.Generic.Dictionary(CS.System.String, CS.UnityEngine.Vector3)

local dic3 = Dic_String_Vector3()

dic3:Add("123", CS.UnityEngine.Vector3.right)

for k, v in pairs(dic3) do

    print(k, v)

end

print(dic3["123"]) --得到nil

print(dic3:TryGetValue("123")) --TryGetValue是字典的底层方法，和dic3["123"]等价

print(dic3:get_Item("123")) --get_Item是索引器的底层方法，和dic3["123"]等价

dic3:set_Item("123", CS.UnityEngine.Vector3.left) --set_Item是索引器的底层方法，和dic3["123"] = Vector3.left等价

print(dic3:get_Item("123"))
```

