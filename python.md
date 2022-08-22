## python运算符

### python 赋值运算符

以下假设变量a为10，变量b为20：

运算符|描述|实例
-|-|-
|=|	简单的赋值运算符|	c = a + b 将 a + b 的运算结果赋值为 c|
|+=|加法赋值运算符|	c += a 等效于 c = c + a 
|-=|减法赋值运算符|	c -= a 等效于 c = c - a
|*=|	乘法赋值运算符|	c *= a 等效于 c = c * a
|/=|	除法赋值运算符|	c /= a 等效于 c = c / a
|%=|	取模赋值运算符|	c %= a 等效于 c = c % a
|**=|	幂赋值运算符|	c **= a 等效于 c = c ** a
|//=|	取整除赋值运算符|	c //= a 等效于 c = c // a

### python 位运算符

按位运算符是把数字看作二进制来进行计算的。Python中的按位运算法则如下：
下表中变量 a 为 60，b 为 13，二进制格式如下：
```
a = 0011 1100

b = 0000 1101

-----------------

a&b = 0000 1100

a|b = 0011 1101

a^b = 0011 0001

~a  = 1100 0011
```

运算符|描述|实例
-|-|-
|&|按位与运算符：参与运算的两个值,如果两个相应位都为1,则该位的结果为1,否则为0|		(a & b) 输出结果 12 ，二进制解释： 0000 1100
\||按位或运算符：只要对应的二个二进位有一个为1时，结果位就为1。|		(a \| b) 输出结果 61 ，二进制解释： 0011 1101
|^|按位异或运算符：当两对应的二进位相异时，结果为1	|	(a ^ b) 输出结果 49 ，二进制解释： 0011 0001
|~|按位取反运算符：对数据的每个二进制位取反,即把1变为0,把0变为1 。~x 类似于 -x-1|		(~a ) 输出结果 -61 ，二进制解释： 1100 0011，在一个有符号二进制数的补码形式。
|<<|左移动运算符：运算数的各二进位全部左移若干位，由 << 右边的数字指定了移动的位数，高位丢弃，低位补0。|		a << 2 输出结果 240 ，二进制解释： 1111 0000
|>>|右移动运算符：把">>"左边的运算数的各二进位全部右移若干位，>> 右边的数字指定了移动的位数|		a >> 2 输出结果 15 ，二进制解释： 0000 1111

### python逻辑运算符

Python语言支持逻辑运算符，以下假设变量 a 为 10, b为 20:

运算符|	逻辑表达式|	描述|	实例
-|-|-|-
and|	x and y|	布尔"与" - 如果 x 为 False，x and y 返回 False，否则它返回 y 的计算值。|	(a and b) 返回 20。
or|	x or y|	布尔"或" - 如果 x 是非 0，它返回 x 的计算值，否则它返回 y 的计算值。|	(a or b) 返回 10。
not|	not x|	布尔"非" - 如果 x 为 True，返回 False 。如果 x 为 False，它返回 True。|	not(a and b) 返回 False

### python成员运算符

运算符|	描述|	实例
-|-|-
in|	如果在指定的序列中找到值返回 True，否则返回 False。|	x 在 y 序列中 , 如果 x 在 y 序列中返回 True。
not in|	如果在指定的序列中没有找到值返回 True，否则返回 False。|	x 不在 y 序列中 , 如果 x 不在 y 序列中返回 True。

### python身份运算符

身份运算符用于比较两个对象的存储单元

运算符|	描述|	实例
-|-|-
is|	is 是判断两个标识符是不是引用自一个对象	|x is y, 类似 id(x) == id(y) , 如果引用的是同一个对象则返回 True，否则返回 False
is not|	is not 是判断两个标识符是不是引用自不同对象	|x is not y ， 类似 id(a) != id(b)。如果引用的不是同一个对象则返回结果 True，否则返回 False。

is 与 == 区别：
is 用于判断两个变量引用对象是否为同一个(同一块内存空间)， == 用于判断引用变量的值是否相等。

```
a = [1, 2, 3]
b = a
b is a 
True
b == a
True
b = a[:]
b is a
False
b == a
True
```

### python运算符优先级

以下表格列出了从最高到最低优先级的所有运算符：

运算符|	描述
-|-
|**|	指数 (最高优先级)
|~ + -|	按位翻转, 一元加号和减号 (最后两个的方法名为 +@ 和 -@)
|* / % //|	乘，除，取模和取整除
|+ -|	加法减法
|>> <<|	右移，左移运算符
|&|	位 'AND'
|^ \||	位运算符
|<= < > >=|	比较运算符
|<> == !=|	等于运算符
|= %= /= //= -= += *= **=|	赋值运算符
is is not|	身份运算符
in not in|	成员运算符
not and or|	逻辑运算符

## python条件语句

Python程序语言指定任何非0和非空（null）值为true，0 或者 null为false。

```
if 判断条件：
    执行语句……
else：
    执行语句……
	
	
if 判断条件1:
    执行语句1……
elif 判断条件2:
    执行语句2……
elif 判断条件3:
    执行语句3……
else:
    执行语句4……	
```

```
示例
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 例3：if语句多个条件
 
num = 9
if num >= 0 and num <= 10:    # 判断值是否在0~10之间
    print 'hello'
# 输出结果: hello
 
num = 10
if num < 0 or num > 10:    # 判断值是否在小于0或大于10
    print 'hello'
else:
    print 'undefine'
# 输出结果: undefine
 
num = 8
# 判断值是否在0~5或者10~15之间
if (num >= 0 and num <= 5) or (num >= 10 and num <= 15):    
    print 'hello'
else:
    print 'undefine'
# 输出结果: undefine
```

简单的语句组

```
var = 100 
 
if ( var  == 100 ) : print "变量 var 的值为100" 
 
print "Good bye!"
```

## python 循环语句

Python 提供了 for 循环和 while 循环（在 Python 中没有 do..while 循环）

### python while 循环

```
示例1
count = 0
while (count < 9):
   print 'The count is:', count
   count = count + 1
 
print "Good bye!"

示例2
# continue 和 break 用法
 
i = 1
while i < 10:   
    i += 1
    if i%2 > 0:     # 非双数时跳过输出
        continue
    print i         # 输出双数2、4、6、8、10
 
i = 1
while 1:            # 循环条件为1必定成立
    print i         # 输出1~10
    i += 1
    if i > 10:     # 当i大于10时跳出循环
        break
```

#### 在 python 中，while … else 在循环条件为 false 时执行 else 语句块

```
#!/usr/bin/python
 
count = 0
while count < 5:
   print count, " is  less than 5"
   count = count + 1
else:
   print count, " is not less than 5"
```

#### 简单语句组

```
#!/usr/bin/python
 
flag = 1
 
while (flag): print 'Given flag is really true!'
 
print "Good bye!"

注意：以上的无限循环你可以使用 CTRL+C 来中断循环。
```

### python for 循环语句

语法：
for循环的语法格式如下：

```
for iterating_var in sequence:
   statements(s)
   
   
示例

for letter in 'Python':     # 第一个实例
   print("当前字母: %s" % letter)
 
fruits = ['banana', 'apple',  'mango']
for fruit in fruits:        # 第二个实例
   print ('当前水果: %s'% fruit)
 
print ("Good bye!")   
```

#### 通过序列索引迭代

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
fruits = ['banana', 'apple',  'mango']
for index in range(len(fruits)):
   print ('当前水果 : %s' % fruits[index])
 
print ("Good bye!")


结果

当前水果 : banana
当前水果 : apple
当前水果 : mango
Good bye!
```
