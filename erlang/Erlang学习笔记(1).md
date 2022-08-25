---
categories:
  - erlang
date: '2017-08-11T11:08:13'
description: ''
tags:
  - Erlang
title: Erlang学习笔记(1)
---



### 0x0 说在前面

Erlang读音`/ˈɜːrlæŋ/`。第一次见到的时候总感觉怎么读都读不对，后来在维基上看到Erlang标注了音标，才能准确的读出来，而且也没那么怪异。因为工作才有机会接触这门语言，也因此只有三天的时间可以看《Erlang程序设计》这本书。学习这门语言的时候带着一个工作目标：把一个Erlang日志收集分析统计的代码转换成Python的。而Erlang的风格是尽量不写注释，尽量在写函数名和变量名的时候表达清楚代码的含义。这样一来学习Erlang就成了必要的，很庆幸，领导给了三天时间学习，三天时间基本也足够了。除了这一片基础语法的入门篇之外，后续还有一篇或者两篇并发编程和分布式编程的，毕竟这个才是Erlang擅长的领域。话不多说，`show me your article`

<!--more-->

### 0x1 配置开发环境

依赖工具：

- Erlang版本：18.3
- IDE：IDEA

下载链接：

- Erlang：https://www.erlang.org/downloads  选择otp18.3即可。
- IDEA：https://www.jetbrains.com/idea/download  选择社区版即可。

IDEA配置Erlang插件：

- [IDEA官方文档-使用IDEA开发Erlang](https://www.jetbrains.com/help/idea/getting-started-with-erlang.html#r_Getting_started_with_Erlang.xmld271393e315)

### 0x2 基础知识

#### 注释

- % 百分比符号标明注释的开始。
- %% 两个符号通常用于注释函数。
- %%% 三个符号通常用于注释模块。

#### 变量

所有的变量都必须以大写字母开头，变量只可一次赋值，赋值之后不可在变。 f()函数释放shell绑定变量。

#### 浮点数

- 浮点数必须含有小数点且小数点后必须有一位10进制数
- 用/来除两个整数时相除结果会自动转换成浮点数
- div取整，rem取余

#### 三种标点符号

- 整个函数的定义结束时用一个句号“.”
- 函数参数，数据构建，顺序语句之间，用逗号“,”分隔
- 函数定义、`case`、`if`、`try..catch`、`receive`表达式中的模式匹配时，用分号“;”分界

#### 恒等

恒等测试符号 =:=以及不等测试符号 =/=

#### 块表达式

当程序中某处的语法要求只能使用单个表达式但是逻辑上又需要在此使用多个表达式时，就可以使用begin...end快表达式

```erlang
begin
  Expr1,
  ...
  ExprN
end
```

### 0x3 内置数据结构

#### 元组及模式匹配（解构）

- _ 代表丢弃的变量，和python相同
- 匹配时模式匹配符=左右两边的元组的结构必须相同。

```erlang
1> Point = {point, 20, 43}.
{point,20,43}
2> {point, x, y} = Point.
** exception error: no match of right hand side value {point,20,43}
3> {point, X, Y} = Point.
{point,20,43}
4> X.
20
5> Y.
43
6> Person = {person, {name, {first, joe}, {last, armstrong}}, {footsize, 42}}.
{person,{name,{first,joe},{last,armstrong}},{footsize,42}}
7> {_, {_, {_, Who}, {_, _}}, {_, Size}} = Person.
{person,{name,{first,joe},{last,armstrong}},{footsize,42}}
8> Who.
joe
9> Size.
42
```

#### 列表

- 列表元素可以是不同的类型。
- 列表头：列表的第一个元素
- 列表尾：列表除第一个元素剩下的部分
- 竖线符号|
  - 将列表的头和尾分割开来
  - [E1, E2, E4, ... , |L]：使用|向列表L的起始处加入多个元素构造成新的列表
- 列表链接操作符 ++ （中缀添加操作符）

**列表操作演示代码**

```erlang
1> L = [1+7, hello, 2-2, {cost, apple, 30-20}, 3]. 
[8,hello,0,{cost,apple,10},3]
2> L1 = [123, {oranges, 4} | L].
[123,{oranges,4},8,hello,0,{cost,apple,10},3]
3> [E1 | L2] = L1.
[123,{oranges,4},8,hello,0,{cost,apple,10},3]
4> E1.
123
5> L2.
[{oranges,4},8,hello,0,{cost,apple,10},3]
6> [E2, E3 | L3] = L2.
[{oranges,4},8,hello,0,{cost,apple,10},3]
7> E3.
8
```

**列表表达式**

形式：[F(X) || X <- L]

```erlang
1> L = [1, 2, 3, 4, 5].
[1,2,3,4,5]
2> [2 * X || X <- L].
[2,4,6,8,10]
3> [X || {a, X} <- [{a, 1}, {b, 2}, {c, 3}, {a, 4}, hello, "wow"]].
[1,4]
```

#### 字符串

Erlang的字符串是一个整数列表。整数列表的内容由每一个字符对应的ascii码构成

```erlang
1> I = $s.
115
2> [I-32, $u, $r, $p, $r, $i, $s, $e].
"Surprise"
3> $r.                                
114
4> [I-32, $u, $r, $p, 114, $i, $s, $e].
"Surprise"
```

#### 映射组(Map)

映射组是一个由多个Key-Vaule结构组成的符合数据类型，类似于Python的字典。具体使用如下

```erlang
1> M1 = #{"name" => "alicdn", "percentage" => 80}.
#{"name" => "alicdn","percentage" => 80}
2> maps:get("name", M1).
"alicdn"
3> M2 = maps:update("percentage", 50, M1).
#{"name" => "alicdn","percentage" => 50}
4> map_size(M1).
2
5> #{"name" := X, "percentage" := Y} = M2.
#{"name" => "alicdn","percentage" => 50}
6> X.
"alicdn"
7> Y.
50
```

构造映射组和模式匹配时的符号不一样，`=>`和`:=`的区别。常见的put方法参见erlang maps库的使用。

### 0x4 模块

- 一个模块存放于一个.erl文件中（模块名和文件名相同）
- 编译模块的命令：c(模块名)。编译成功之后就会加载到当前shell中
- 调用模块中的函数：模块名:函数名(参数)
- 导入模块中的函数：`-import(lists, [map/2, sum/1]).`
- 导出模块中的函数：
  - 导出指定函数`-export([start/0, area/2]).`
  - 导出全部函数`-compile(export_all).`，避免在开发阶段经常会向export中添加函数或者删除函数

```erlang
-module(learn_test).
-author("ChenLiang").

%% API
-export([area/1]).


area({rectangle, Width, Height}) -> Width * Height;
area({circle, R}) -> 3.14159 * R * R;
area({square, X}) -> X * X.
```

编译模块，调用函数

```erlang
1> c(learn_test).
{ok,learn_test}
2> learn_test:area({circle, 2.0}).
12.56636
3>
```

### 0x5 函数

#### 基本函数

同名同目（参数数量，arity）的才是同一个函数。因此函数名相同，目不相同的函数是完全不同的两个函数。同名不同目的函数通常作为辅助函数。

- 函数不会显示地返回值，函数中最后一条语句的执行结果将作为函数的返回值。


- 同一个函数中，并列的逻辑分支之间，用分号 “;” 分界；顺序语句之间，用逗号 “,” 分隔。

**示例代码：计算列表元素的和**

```erlang
-module(learn_test).
-author("ChenLiang").

%% API
-export([sum/1]).

sum(L) -> sum(L, 0).

sum([], N) -> N;
sum([H|T], N) -> sum(T, H + N).
```

#### 匿名函数

erlang中的匿名函数就是fun。fun也可以有若干个不同的字句。

```erlang
1> Z = fun(X) -> 2*X end.
#Fun<erl_eval.6.50752066>
2> Double = Z.
#Fun<erl_eval.6.50752066>
3> Double(4).
8
4> TempConvert = fun({c, C}) -> {f, 32 + C * 9 / 5};
5> ({f, F}) -> {c, (F - 32) * 5 / 9}                
6> end.                                             
#Fun<erl_eval.6.50752066>
7> TempConvert({c, 100}).                           
{f,212.0}
8> TempConvert({f, 212}).
{c,100.0}
```

#### 高阶函数

返回fun或者接受fun作为参数的函数都称为高阶函数。

**以fun为参数的函数**

常见的是lists模块中的map(Fun, List1) -> List2，filter(Pred, List1) -> List2函数。

lists模块的具体使用参见：https://www.erlang.org/doc/man/lists.html

```erlang
1> Even = fun(X) -> X rem 2 =:= 0 end.
#Fun<erl_eval.6.50752066>
2> lists:map(Even, [1, 2, 3, 4, 5, 6]).
[false,true,false,true,false,true]
3> lists:filter(Even, [1, 2, 3, 4, 5, 6]).
[2,4,6]
```

**返回fun的函数**

一般在返回的函数内部封装了一些变量和逻辑。通常情况下不写返回fun的函数。

```erlang
1> Mult = fun(Times) -> (fun(X) -> X * Times end ) end.
#Fun<erl_eval.6.50752066>
2> Triple = Mult(3).
#Fun<erl_eval.6.50752066>
3> Triple(4).
12
```

### 0x6 断言

强化模式匹配的功能，给模式匹配增加一些变量测试和比较的能力

```erlang
max(X, Y) when X > Y -> X;
max(_, Y) -> Y.
```

### 0x7 记录

记录是Erlang中基于元组的key-value数据定义，使用示例如下：

```erlang
-module(learn_test).
-author("ChenLiang").

%% API
-export([record_test1/0, record_test2/0]).

-record(person, {name, age=18, hobby=["erlang"]}).    %% record定义可以存放于hrl和erl中

record_test1() ->
  Person = #person{name="hahaha"},    %% 为record中字段赋值
  Person#person.hobby.    %%  通过.操作符访问record中字段

record_test2() ->
  Person = #person{},
  #person{name = Name} = Person,    %% 通过模式匹配获取record字段
  Name. %% 输出undefined
```

### 0x8 .hrl头文件

某些文件的扩展名为 `.hrl`。这些`.hrl`是在 `.erl` 文件中会用到的头文件，使用方法如下：

```erlang
-include("File_Name").
```

例如：

```erlang
-include("mess_interface.hrl").
```

.hrl 文件中可以包含任何合法的 Erlang 代码，但是通常里面只包含一些记录和宏的定义。

### 0x9 case / if 表达式

#### case 表达式

case语句语法

```erlang
case Experssion of
  Pattern1 [when Guard1] -> Expr_seq1;
  Pattern2 [when Guard2] -> Expr_seq2;
  ...
end
```

将Expression的结果和各个Pattern逐个匹配，匹配成功，则计算表达式序列的值，并返回。全部匹配不到，则直接报错。

case语句使用示例:

```erlang
-module(learn_test).
-author("ChenLiang").

%% API
-export([filter/2]).


filter(P, [H|T]) ->
  case P(H) of
    true -> [H|filter(P, T)];
    false -> filter(P, T)
  end
;
filter(_, []) -> [].
```

在erl shell中运行结果如下：

```
1> c(learn_test).
{ok,learn_test}
2> learn_test:filter(fun(X) -> X rem 2 =:= 0 end, [1, 2, 3, 4, 5]).
[2,4]
```

#### if 表达式

if语句使用示例

```erlang
-module(learn_test).
-author("ChenLiang").

%% API
-export([bigger/2]).


bigger(X, Y) ->
  if
    X > Y -> X;
    X < Y -> Y;
    true -> -1
  end.
```

如果没有匹配的断言，则会抛出异常。因此最后一个断言通常是true断言。

### 0xA 异常

Erlang中一切都是表达式，都有返回值，因此异常捕获语句也有返回值。

捕获所有的异常`_:_`

```erlang
-module(learn_test).
-author("ChenLiang").

%% API
-export([catch_exc1/0,catch_exc2/0]).


exception() ->
  exit({system, "123123"}).

catch_exc1() ->
  try
      exception()
  catch
      _:_  -> 111
  end.

catch_exc2() ->
  try
    exception()
  catch
    _  -> 222
  end.
```

erl shell输出结果

```erlang
1> learn_test:catch_exc1().
111
2> learn_test:catch_exc2().
** exception exit: {system,"123123"}
     in function  learn_test:exception/0 (learn_test.erl, line 17)
     in call from learn_test:catch_exc2/0 (learn_test.erl, line 28)
```

多种错误的检测可以 使用try catch风格。

参考[stackoverflow-How do I elegantly check many conditions in Erlang?](https://stackoverflow.com/questions/666111/how-do-i-elegantly-check-many-conditions-in-erlang/669075#669075)



