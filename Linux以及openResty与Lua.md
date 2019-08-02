##  基于Linux的openResty与Lua脚本

### Lua脚本语言

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
