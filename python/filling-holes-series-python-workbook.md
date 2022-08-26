---
abbrlink: 599780751
alias: 2016/12/13/filling-holes-series-python-workbook/index.html
categories:
- python
date: '2016-12-13T22:01:08'
description: ''
tags:
- practice
- 习题
- Python
title: 【填坑系列】Python习题集
---








## 求100万以内的所有素数个数

**第一种方法**

**思路**：当前数为i，则遍历比int(sqrt(i))+1小的所有数是是否都不可以整除，是，则是素数

**理论**：如果遍历所有比i小的除数j并且当除数j>int(sqrt(i))时，如果j能整出i，那么必定存在一个小于int(sqrt(i))的数能整出i，因此我们只需只需遍历int(sqrt(i))+1以内的数即可

**代码**

```python
cnt = 0
for i in range(2,1000000):
    for j in range(2, int(i ** 0.5) + 1):
        if i % j == 0:
            break
    else:
       cnt += 1 
print(cnt)
```

输出结果如下

```
78498
```

**第二种方法**

**理论**

- 假如一个整数是合数，则一定存在一个小于它的素数作为其因数。比如9是一个合数，而素数3就是它的一个因数。
- 假如我们知道了小于一个数的所有素数，则只需确定该数能不能被这些素数整除即可。如果不能被整除，则这个数一定是个素数。反之，则不是。
- 也就是说当我们获得一个素数时，可以将它所有的倍数都标记为非素数，这样当我们遍历到一个数时，他没有被任何小于它的素数标记为非素数，则可以确定该数是个素数。
- 比如：从2开始，在初始化时2就是素数。3是类似。遍历到4时，4已经被素数2给标记了，直接跳过

**思路**

- 初始化一个大范围内的列表，初始时所有数都为素数，遍历时按照以上理论将所有的非素数标记出来即可

**代码**

```python
cnt = 0
is_prime = [True]*1000000
is_prime[0] = False
is_prime[1] = False
for i in range(2, 1000000):
    if is_prime[i] is False:
        continue
    cnt += 1
    k = i
    while k * i < 1000000:
        is_prime[k * i] = False
        k += 1
print(cnt)
```

输出结果

```
78498
```

<!--more-->

## 打印九九乘法表

**思路**：每一次内层循环j只要循环到外层循环i即可

**代码**

```python
#!/usr/bin/env python 
#coding=utf-8

def multiplicationTable():
    for i in range(1,10):
        for j in range(1,i+1):
            r=i*j
            print("%d*%d=%-3d"%(i,j,r),sep='',end=' ')
        print('\n')
if __name__=='__main__':
    multiplicationTable()
```

输出结果如下

```
1*1=1   

2*1=2   2*2=4   

3*1=3   3*2=6   3*3=9   

4*1=4   4*2=8   4*3=12  4*4=16  

5*1=5   5*2=10  5*3=15  5*4=20  5*5=25  

6*1=6   6*2=12  6*3=18  6*4=24  6*5=30  6*6=36  

7*1=7   7*2=14  7*3=21  7*4=28  7*5=35  7*6=42  7*7=49  

8*1=8   8*2=16  8*3=24  8*4=32  8*5=40  8*6=48  8*7=56  8*8=64  

9*1=9   9*2=18  9*3=27  9*4=36  9*5=45  9*6=54  9*7=63  9*8=72  9*9=81  
```

## 求几何级数的第N项

**思路**

- 几何级数的形式是：`a+a*q+a*q^2+a*q^3+...+a*q^n`
- 需要输入的项是：a,q,n

**代码**

```python
# !/user/bin/env python
# coding = utf-8

a=int(input('请输入几何级数的a: '))
q=int(input('请输入几何级数的q: '))
n=int(input('请输入几何级数的n: '))
sum=0
for i in range(0,n+1):
    sum += a * q  ** i
print(sum)
```

输入输出结果如下

```
请输入几何级数的a: 3
请输入几何级数的q: 2
请输入几何级数的n: 4
93
```

## 求菲波那切数列的第101位

先写出递推公式再来写实现，递推公式如下

**思路**

- fib[0]=1    当i=0
- fib[1]=1    当i=1
- fib[i]=fib[i-1]+fib[i-2]    当i>1
- fib的第101位也就是fib[100]=fib[99]+fib[98]

**代码**

```python
a = 0
b = 0
for i in range(0,101):
    if i == 0:
        a = 1
    elif i ==1:
        b = 1
    else:
        c = a + b
        a = b
        b = c
print(b)
```

输出结果如下

```
573147844013817084101
```

## 求杨辉三角第n行第k列的值

```python
#!/usr/bin/env python  
# encoding: utf-8  

"""
Created on 2/26/17 5:03 PM
@author: Flowsnow
@file: YangHuiTriangle.py
@function: 求杨辉三角第n行第k列的值
"""


def yang_hui_triangle(n, k):
    lst = []
    for i in range(n+1):
        row = [1]
        lst.append(row)
        if i == 0:
            continue
        for j in range(1, i):
            row.append(lst[i-1][j-1] + lst[i-1][j])
        row.append(1)
    print(lst[n][k])


if __name__ == '__main__':
    yang_hui_triangle(6, 4)

```

## 字符串转化为数值

描述：把字符串形式的整数或浮点数转化为int或float， 不使用int和float函数

```python
#!/usr/bin/env python  
# encoding: utf-8  

"""
Created on 2/26/17 5:52 PM
@author: Flowsnow
@file: str2num.py 
@function: 把字符串形式的整数或浮点数转化为int或float， 不使用int和float函数
"""


def str2num(s: str):
    mapping = {str(x): x for x in range(10)}
    i, _, f = s.partition('.')
    # print(i, f)
    ret = 0
    for idx, x in enumerate((i+f)[::-1]):
        ret += mapping[x] * 10 ** idx
    return ret / 10 ** len(f)

if __name__ == '__main__':
    s = '1230.0541640'
    print(str2num(s))

```

## 移除一个列表中的重复元素，并保持列表原来的顺序

```
#!/usr/bin/env python  
# encoding: utf-8  

"""
Created on 2/26/17 6:30 PM
@author: Flowsnow
@file: rm_elements.py 
@function: 移除一个列表中的重复元素，并保持列表原来的顺序
"""

def remove_elements(lst: str):
    ret = []
    for x in lst:
        if x not in ret:
            ret.append(x)
    return ret

if __name__ == '__main__':
    lst = [1, 2, 4, 5, 2, 3, 1, 2, 6, 7, 8, 3, 2, 3, 4]
    print(remove_elements(lst))

```

## 扁平化字典

**例如：** 

* {'a': {'b': 1}} 扁平化之后是 {'a.b': 1}
* {'a': {'b': {'c': 1, 'd': 2}, 'x': 2}}扁平化之后是{'a.x': 2, 'a.b.c': 1, 'a.b.d': 2}

**初始字典的特点**：

- 字典的每个key都是可hash的，因此不会是字典
- 初始字典不为空字典
- 字典的value深度可以无限嵌套

**思路**：使用递归，每次递归深度都会变化，也就是说路径会变化，可以使用一个path变量记录路径

1. 如果嵌套的v不是字典时，直接加入新元素: desDict['{}.{}'.format(path, k).lstrip('.')] = v
2. 如果嵌套的v为空字典时，直接用空字符串代替: desDict['{}.{}'.format(path, k).lstrip('.')] = ''
3. 如果嵌套的v不为空字典时，直接增长path，并将v进行下一次递归: flatten_dict(v, desDict, path)
4. 每一次递归返回时，就说明当前深度的字典已经遍历完毕，需要减短path
5. rstrip函数都不是原地修改，返回的都是副本

**判断变量是否是字典**

- type()
- isInstance()

**代码**

```python
#!/usr/bin/env python  
# encoding: utf-8  

"""
Created on 2/26/17 8:26 PM
@author: Flowsnow
@file: flatten_dict.py 
@function: 扁平化字典
"""


def flatten_dict(srcDict: dict, desDict: dict,  path: str):
    for k, v in srcDict.items():
        if not isinstance(v, dict):
            desDict['{}.{}'.format(path, k).lstrip('.')] = v
        else:
            if v == {}:
                desDict['{}.{}'.format(path, k).lstrip('.')] = ''
            else:
                path = '{}.{}'.format(path, k).lstrip('.')
                flatten_dict(v, desDict, path)
                path = path.rstrip('.{}'.format(k))


if __name__ == '__main__':
    srcDict = {'a': {'b': {'c': 1, 'd': 2}, 'x': 2}}
    # srcDict = {'a': {'b': 1}}
    desDict = {}
    flatten_dict(srcDict, desDict, '')
    print(desDict)
```

## 实现base64编码解码算法

**Base64编码的思想**

- 采用64个基本的ASCII码字符对数据进行重新编码。它将需要编码的数据拆分成字节数组。以3个字节为一组。按顺序排列24 位数据，再把这24位数据分成4组，即每组6位。再在每组的的最高位前补两个0凑足一个字节。这样就把一个3字节为一组的数据重新编码成了4个字节。当所要编码的数据的字节数不是3的整倍数，也就是说在分组时最后一组不够3个字节。这时在最后一组填充1到2个0字节。并在最后编码完成后在结尾添加1到2个 “=”。

**base64编码示例**： 将对ABC进行BASE64编码

1. 首先取ABC对应的ASCII码值。A（65）B（66）C（67）；
2. 再取二进制值A（01000001）B（01000010）C（01000011）；
3. 然后把这三个字节的二进制码接起来（010000010100001001000011）；
4. 再以6位为单位分成4个数据块,并在最高位填充两个0后形成4个字节的编码后的值，（00010000）（00010100）（00001001）（00000011）
5. 再把这四个字节数据转化成10进制数得（16）（20）（9）（3）；
6. 最后根据BASE64给出的64个基本字符表，查出对应的ASCII码字符（Q）（U）（J）（D），这里的值实际就是数据在字符表中的索引。

**base64字符表**： 最多六个字节，因此最范围是0~63，所以总共64个字符

- ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/

**字符和ascii码之间的转换**：单个字符

- ord 字符转换成ascii
- chr ascii转换成字符

**字符串和ascii码之间的转换**：字符串

- map(ord, "a test String: 123456")

**加密示例**

- **CBdaF3FV**的编码结果是**Q0JkYUYzRlY=**
- **CBdaF34FV**的编码结果是**Q0JkYUYzNEZW**
- **CdaF3FV**的编码结果是**Q2RhRjNGVg==**
- **ABC**的编码结果是**QUJD**

**作用**

- 主要用做把二进制转换成字符串

**代码1：作为字符串处理**

```python
# base64编码
def base64Encode(s):
    base64StrList = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
    ret = ''
    bList = list(map(ord, s))
    bStr = ''
    #print(bList)
    for x in bList:
        tmpS = str(bin(x))
        bStr += '0' * (10 - len(tmpS)) + tmpS.lstrip('0b')
    print(bStr)
    i = 0
    while i + 6 < len(bStr):
        tmpX = bStr[i: i+6]
        #print(tmpX)
        ret += base64StrList[int(tmpX, 2)]
        i += 6
    rest = bStr[i:]
    if len(rest) == 2:
        ret += base64StrList[int(rest + '0000', 2)]
        ret += '=='
    elif len(rest) == 4:
        ret += base64StrList[int(rest + '00', 2)]
        ret += '='
    else:
        ret += base64StrList[int(rest, 2)] # 在while部分处理之后剩下一个完整的6位
    print(ret)


# base64解码
def base64Decode(s):
    base64StrList = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
    bStr = ''
    ret = ''
    for x in s:
        if x == '=':
            continue
        i = base64StrList.find(x)
        s1 = str(bin(i))
        bStr += '0'*(10-len(s1)-2) + s1.lstrip('0b')

    cnt = s.count('=')
    bStr = bStr[: -2 * cnt + len(bStr)]

    i = 0
    while i < len(bStr):
        b = bStr[i: i + 8]
        ret += chr(int(bStr[i: i + 8], 2))
        i += 8

    print(ret)
```

**代码2：作为字节数组处理**

优点：字节数组+位运算，提高处理速度，减少内存占用。

```python
#!/usr/bin/env python  
# encoding: utf-8  

"""
Created on 2/26/17 9:11 PM
@author: Flowsnow
@file: base64.py 
@function: 实现base64编码解码算法:字节数组+位运算
"""


# base64编码
def b64encode(data: bytes) -> str:
    table = b'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
    encoded = bytearray()
    c = 0
    for x in range(3, len(data)+1, 3):
        i = int.from_bytes(data[c: x], 'big')   # bytes to int
        for j in range(1, 5):
            encoded.append(table[i >> (24 - j*6) & 0x3f])
        c += 3
    r = len(data) - c
    if r > 0:
        i = int.from_bytes(data[c:], 'big') << (3-r) * 8
        for j in range(1, 5-(3-r)):
            encoded.append(table[i >> (24 - j*6) & 0x3f])
        for _ in range(3-r):
            encoded.append(int.from_bytes(b'=', 'big'))
    return encoded.decode()


# base64解码
def b64decode(data:str) -> bytes:
    table = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
    decoded = bytearray()
    s = 0
    for e in range(4, len(data)+1, 4):
        tmp = 0
        for i, c in enumerate(data[s:e]):
            if c != '=':
                tmp += table.index(c) << 24 - (i+1) * 6
            else:
                tmp += 0 << 24 - (i+1) * 6
        decoded.extend(tmp.to_bytes(3, 'big'))
        s += 4
    return bytes(decoded.rstrip(b'\x00'))


if __name__ == '__main__':
    print(b64encode(b'abc'))
    print(b64decode('YWJj'))

```

## 实现计数器，可以指定基数和步长

生成器和匿名函数的使用

```python
#!/usr/bin/env python  
# encoding: utf-8  

"""
Created on 2/27/17 9:30 AM
@author: Flowsnow
@file: make_inc.py 
@function: 实现计数器，可以指定基数和步长
"""


def make_inc(base, step):
    def counter():
        nonlocal base
        nonlocal step
        x = base
        while True:
            x += step
            yield x
    c = counter()
    return lambda : next(c)

if __name__ == '__main__':
    # inc这个函数封装了一个生成器c，并且每次调用inc的时候都是在执行next(c)
    inc = make_inc(2, 3)
    print(inc())
    print(inc())
    print(inc())
    print(inc())

```

## 查找两个字符串的最长公共子串

- 暴力法： 找出两个字符串各自所有的子串，然后一一比较，更新最长的值
- 动态规划：
  1. 两个字符串分别为s1和s2
  2. s1[i]和s2[j]分别表示其第i和第j个字符(字符顺序从0开始)
  3. 令L[i, j]表示以s1[i]和s2[j]为结尾的相同子串的最大长度。
  4. L[i, j] = L[i-1, j-1] + 1 如果s1[i] == s2[j]
  5. L[i, j] = 0 如果s1[i] != s2[j]

```python
#!/usr/bin/env python  
# encoding: utf-8  

"""
Created on 2/27/17 9:48 AM
@author: Flowsnow
@file: longest_common_substring.py 
@function: 
"""


def longest_common_substring(s1: str,s2:str):
    s = ''
    dp = []
    maxL = 0    # 记录子串的最长长度
    maxI = 0    # 记录子串最长的下标
    for i, x in enumerate(s1):
        dp.append([])
        for j, y in enumerate(s2):
            if x == y:
                if i > 0 and j > 0:
                    dp[i].append(dp[i - 1][j - 1] + 1)
                else:
                    dp[i].append(1)
                if dp[i][j] > maxL:
                    maxI = i
                    maxL = dp[i][j]
            else:
                dp[i].append(0)
    s = s1[maxI + 1 - maxL: maxI + 1]    # maxI是下标
    for i in dp:
        print(i)
    return s
if __name__ == '__main__':
    s1 = 'I-love-Python'
    s2 = 'snow-love-other'
    s = longest_common_substring(s1, s2)
    print(s)

```

输出结果

```python
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
[0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0]
[0, 0, 0, 0, 0, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0]
[0, 0, 1, 0, 0, 0, 3, 0, 0, 0, 1, 0, 0, 0, 0]
[0, 0, 0, 0, 0, 0, 0, 4, 0, 0, 0, 0, 0, 0, 0]
[0, 0, 0, 0, 0, 0, 0, 0, 5, 0, 0, 0, 0, 1, 0]
[0, 0, 0, 0, 1, 0, 0, 0, 0, 6, 0, 0, 0, 0, 0]
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0]
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2, 0, 0]
[0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0]
[0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
'-love-'
```

## 实现命令分发器

实现函数可带任意参数(可变参数除外)，解析参数并要求用户输入

## 实现ls命令

实现 -l -a -h 选项

## 实现find命令

实现 -name -type -ctime -mtime -cnewer -executable -newer -gid -uid 测试

## 实现cp命令

实现 -r -p选项

## 实现LinkedList

函数实现

类实现

## 实现优先队列

函数实现

类实现

## 实现字典

函数实现

类实现