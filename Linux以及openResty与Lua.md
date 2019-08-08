

### phpstorm 使用Emmylua插件编写Lua脚本

##  Lua脚本语法

* 全局变量与局部变量
```
1. 全局变量
b=0;
print(b);

print(c);---------打印未初始化的全局变量输出：nil

b = nil --------删除全局变量

2. 局部变量
local a = 1;-----定义
```
* 判断变量类型--type
```
type(x) ----返回变量类型，返回的是string字符串

print(type(x) == nil) ----返回false

print(type(x) == 'nil') ----返回true
```
* boolean布尔类型
```
1. Lua 中 nil 和 false 为“假”，其它所有值均为“真”。比如 0 和空字符串就是“真”

```
* number数字类型
```
local count = 10
local order = 3.99
local score = 98.01
print(math.floor(order))   -->output:3
print(math.ceil(score))    -->output:99
```
* 字符串
```
1. 表示字符串

用单引号或者双引号，或者使用长括号（[[]]）来定义，整个词法分析过程将不受分行限制，不处理任何转义符

2. 转义字符

string =  hello\',\"\",\\n，\\t

3. 字符串连接使用的是 ..

print("a" .. 'b') ----输出ab

注意：很多字符串连接时，采用..性能很低，推荐使用table.concat()

4. 字符串与number类型转换

print(tonumber("10") == 10)
print(tostring(10) == "10")

5. 计算字符串的长度
    1）使用 # 来计算字符串的长度，放在字符串前面
        print(#"this is string")
    2） string.len(s) ----此函数是不推荐的，推荐使用 # 运算符来获取 Lua 字符串的长度


6. string不可修改（重点理解）

  1）在 Lua 实现中，Lua 字符串一般都会经历一个“内化”（intern）的过程，即两个完全一样的 Lua 字符串在 
  
  Lua 虚拟机中只会存储一份。每一个 Lua 字符串在创建时都会插入到 Lua 虚拟机内部的一个全局的哈希表中。
  
  2）Lua 的字符串是不可改变的值，不能像在c语言中那样直接修改字符串的某个字符，而是根据修改要求来创建一个
  
  新的字符串。Lua 也不能通过下标来访问字符串的某个字符。
  
  Lua的字符串和其它对象都是自动内存管理机制所管理的对象，不需要担心字符串的内存分配和释放提供了效率，安全

```
* 字符串的函数
* table类型
```
1. ua中没有数组和map，都是用table这个类型

2. 初始化表

mytable = {}

3. 指定值

mytable[2]= "Lua2"

mytalbe["k1"] = v1

4. 移除引用

mytable = nil ----lua垃圾回收会释放内存

5. lua类似数组的table ，索引值从1开始,而不是0

6. table 是内存地址,赋值给变量;table进行赋值给变量，其实是把内存地址给了变量，变量只是引用了内存地址

local mytable1 = {"a",key1 = "v1","b",k2="v2",k3="v3","hello","world"}

local mytable2 =  mytable1

mytable2[1] = "aa"

print(mytable2[1])

print(mytable1[1])
```
* for循环结构
```
1. for 语句有两种形式：数字 for 和范型 for。

for begin（开始的数）, finish（结束的数）, step（步长） do 
    ....
end

for i = 1, 5 do 
    print(i)
end

2. 如果不想给循环设置上限的话，可以使用常量 math.huge

for i = 1, math.huge do
    if (0.3*i^3 - 20*i^2 -500) > 0 do
        print(i)
        break
    end
end

3. for 泛型

对lua的table类型进行遍历;泛型 for 循环通过一个迭代器（iterator）函数来遍历所有值

local a = {"a", "b", "c", "d"}
for i, v in ipairs(a) do
  print("index:", i, " value:", v)
end

Lua 的基础库提供了 ipairs，这是一个用于遍历数组的迭代器函数。在每次循环中，i 会被赋予一个索引值，

同时v被赋予一个对应于该索引的数组元素值。

遍历一个 table 中所有的 key

for k in pairs(t) do
    print(k)
end

pairs是可以把数组类型和哈希类型索引值，都会迭代出来

```
* 正则表达式
```
1. . 

匹配任意字符

2. % 
  1) 转义字符，改变后一个字符的原有意思
  
  2) 当后面的接的是特殊字符时，将还原特殊字符的原意
  
  3) %和一些特定的字母组合构成了lua的预定义字符集
  
  4) 和数字1~9组合表示之前捕获的分组
      (a)n%1  匹配  ana
  
3. [...]
  1) 匹配一个包含于集合内的字符
      [a%%]na 匹配  %na,ana两种情况  //%%此时前面一个%是转义字符；%%表示匹配%
      [%a]na  匹配  wna,bna,Bna //此时%a作为一个整体匹配预定义字符集[a-zA-Z]

4. [...-...]
  1) 表示ascii码在它前一个字符到它后一个字符之间的所有字符
      [a-z]na    匹配   ana,bna
      
5. [^...]
    1) 不在...中的字符集合。

6. *
    1) 表示前一个字符出现0次或多次
    
7. +
    1) 表示前一个字符出现1次或1次以上

8. -
    1) 匹配前一字符0次或多次
    
9. ?
    1) 表示前一个字符出现0次或1次

10. 分组  (...)
    1） 表达式中用小括号包围的子字符串为一个分组，分组从左到右（以左括号的位置），组序号从1开始递增
    (%d+)%1  匹配  123123 
    
 11. ^  匹配字符串开头
 12. $  匹配字符串结尾
```
* 正则的预定义字符集
```
1. %s  空白字符
2. %p  标点符号
3. %c  控制字符 例如\n
4. %w  字母数字[a-zA-Z0-9]
5. %a  字母[a-zA-Z]
6. %l  小写字母[a-z]
7. %u  大写字母[A-Z]
8. %d  数字[0-9]
9. %x  16进制数[0-9a-fA-F]
10.%z  ascii码是0的字符
```
* Lua面向对象编程示例（涉及到Lua中的table，元表，模块，面向对象编程的类概念，继承）
```
--基类 shape

local shape = { name = ""}

function shape:new(name) --相当于初始化
    local o = {
        name = name or "shape"
    }
    setmetatable(o, {__index = self}) --设置元表，用于子类继承
    return o
end

function shape:getPerimeter() --shape的getPerimeter方法
    print("getPerimeter in shape");
    return 0
end

function shape:getArea() --shape的getArea方法
    print("getArea in shape");
    return 0
end

function shape:getVum ()
    print("getVum in shape");
    return 0
end

return shape   --返回模块


-----triangle继承类

local shape = require("shape")
local triangle = {}


function triangle:new (name,a1,a2,a3) --初始化
  local obj = shape:new(name); --实例化shape对象
  
  local super_mt = getmetatable(obj); --获取实例化shape对象中的元表即obj实例化对象的父类
  
  setmetatable(self, super_mt); --继承shape类
  
  
  obj.super = setmetatable({}, super_mt) --把父类的元表赋值super对象
  
  --3）属性赋值
  obj.a1 = a1 or 0;
  obj.a2 = a2 or 0;
  obj.a3 = a3 or 0;
  
  setmetatable(obj, { __index = self })
  
  return obj;
end


function triangle:getPerimeter() -- 方法的重写getPerimeter
  print("getPerimeter in triangle");
  return (self.a1 + self.a2 + self.a3);
end


function triangle:getHalfPerimeter() -- 方法的重写 getHalfPerimeter
  print("getHalfPerimeter in triangle");
  return (self.a1 + self.a2 + self.a3) / 2
end

return triangle

--------test测试脚本
local triangle = require("triangle")

local triangle1 = triangle:new("t1",1,2,3)
local triangle2 = triangle:new("t2",6,7,8)

print("t1 getPerimeter:",triangle1:getPerimeter())
print("t1 getHalfPerimeter:",triangle1:getHalfPerimeter())

print("t2 getPerimeter:",triangle2:getPerimeter())
print("t2 getHalfPerimeter:",triangle2:getHalfPerimeter())

---- 从shape继承的getVum方法
print("t1 getVum:",triangle1:getVum())
print("t2 getVum:",triangle2:getVum())
print("===============");
print("t2 super getPerimeter:",triangle2.super:getPerimeter())
```
