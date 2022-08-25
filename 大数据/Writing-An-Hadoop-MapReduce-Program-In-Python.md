---
categories:
  - 大数据
date: '2018-04-17T15:54:21'
description: ''
tags:
  - Hadoop
  - Python
  - MapReduce
title: 使用Python语言写Hadoop MapReduce程序
---




在了解到Hadoop的生态环境以及Hadoop单机模式和伪分布式模式安装配置之后，我们可以使用自己熟悉的语言来编写Hadoop MapReduce程序，进一步了解MapReduce编程模型。

本教程将使用Python语言为Hadoop编写一个简单的MapReduce程序：**单词计数**

> 尽管Hadoop框架是用Java编写的，但是为Hadoop编写的程序不必非要Java写，还可以使用其他语言开发，比如Python，Ruby，C++等

编写完成的MapReduce程序可以直接在你已经搭建好的伪分布式程序中调试运行。

<!--more-->

# MapReduce的Python代码

我们将使用[Hadoop流API](https://hadoop.apache.org/docs/r1.2.1/streaming.html#Hadoop+Streaming)通过STDIN和STDOUT在Map和Reduce代码间传递数据。我们只需要使用Python的sys.stdin读取输入数据和打印输出到sys.stdout。这就是我们需要做的，因为Hadoop流会处理好其他的一切。

## mapper.py

将下面的代码保存在文件 `/home/hadoop/workspace/mapper.py` 中。它将从STDIN读取数据，拆分为单词并输出一组映射单词和它们数量（中间值）的行到STDOUT。尽管这个Map脚本不会计算出单词出现次数的总和（中间值）。相反，它会立即输出`<word> 1`元组的形式——即使某个特定的单词可能会在输入中出现多次。在我们的例子中，我们让后续的Reduce做最终的总和计数。当然，你可以按照你的想法在你自己的脚本中修改这段代码。

需要给mapper.py文件赋予可执行权限：

```bash
chmod +x /home/hadoop/workspace/mapper.py
```

`/home/hadoop/workspace/mapper.py`代码如下

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
Created on 4/17/18 11:16 AM
@author: Chen Liang
@function:  word count mapper
"""

import sys

# 从标准输入STDIN输入
for line in sys.stdin:
    # 移除line收尾的空白字符
    line = line.strip()
    # 将line分割为单词
    words = line.split()
    # 遍历
    for word in words:
        # 将结果写到标准输出STDOUT
        # 此处的输出会作为Reduce代码的输入
        print('{}\t{}'.format(word, 1))
```

## reducer.py

将下面的代码保存在文件 `/home/hadoop/workspace/reducer.py` 中。它将从STDIN读取mapper.py的结果（因此mapper.py的输出格式和reducer.py预期的输入格式必须匹配），然后统计每个单词出现的次数，最后将结果输出到STDOUT中。

需要给reducer.py文件赋予可执行权限：

```bash
chmod +x /home/hadoop/workspace/reducer.py
```

`/home/hadoop/workspace/reducer.py`代码如下

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
Created on 4/17/18 11:16 AM
@author: Chen Liang
@function: word count reducer
"""

import sys

current_word = None
current_count = 0
word = None

for line in sys.stdin:
    # 移除line收尾的空白字符
    line = line.strip()

    # 解析我们从mapper.py得到的输入
    word, count = line.split('\t', 1)

    # 将字符串count转换为int
    try:
        count = int(count)
    except ValueError:
        # 不是数字，不做处理，跳过
        continue

    # hadoop在将kv对传递给reduce之前会进行按照key进行排序，在这里也就是word
    if current_word == word:
        current_count += count
    else:
        if current_word is not None:
            # 将结果写入STDOUT
            print('{}\t{}'.format(current_word, current_count))
        current_count = count
        current_word = word

# 最后一个单词不要忘记输出
if current_word == word:
    print('{}\t{}'.format(current_word, current_count))

```

## 代码测试

在MapReduce作业中正式使用mapper.py和reducer.py之前，最好先在本地测试mapper.py和reducer.py脚本。否则，作业可能成功完成了但没有得到作业结果数据或者得到了不是你想要的结果。

这里有一些想法，关于如何测试这个Map和Reduce脚本的功能。

使用`cat data | map | sort | reduce`这样的顺序。具体测试如下：

```bash
# 首先在本地测试 mapper.py 和 reducer.py

# 非常基本的测试
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace$ echo "foo foo quux labs foo bar quux" | /home/hadoop/workspace/mapper.py
foo	1
foo	1
quux	1
labs	1
foo	1
bar	1
quux	1
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace$ vim reducer.py 
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace$ echo "foo foo quux labs foo bar quux" | /home/hadoop/workspace/mapper.py | sort -k1,1 | /home/hadoop/workspace/reducer.py
bar	1
foo	3
labs	1
quux	2
# 使用示例文件
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace$ cat /home/hadoop/workspace/file/input1.txt | /home/hadoop/workspace/mapper.py 
Now	1
that	1
everything	1
is	1
prepared,	1
we	1
can	1
finally	1
run	1
our	1
Python	1
MapReduce	1
...
```

其中`/home/hadoop/workspace/file/input1.txt`示例输入文件的内容如下：

```
Now that everything is prepared, we can finally run our Python MapReduce job on the Hadoop cluster.
As I said above, we leverage the Hadoop Streaming API for helping us passing data between our Map and Reduce code via STDIN and STDOUT.
```

# 在Hadoop上运行Python代码

## 下载示例输入数据

对于这个示例，我们将使用的三个文本来自Gutenberg项目：

1. [The Outline of Science, Vol. 1 (of 4) by J. Arthur Thomson](https://www.gutenberg.org/etext/20417)
2. [The Notebooks of Leonardo Da Vinci](https://www.gutenberg.org/etext/5000)
3. [Ulysses by James Joyce](https://www.gutenberg.org/etext/4300)

下载对应链接下的`Plain Text UTF-8`，三个文本对应的地址分别为：

1. https://www.gutenberg.org/cache/epub/20417/pg20417.txt
2. https://www.gutenberg.org/files/5000/5000-8.txt
3. https://www.gutenberg.org/files/4300/4300-0.txt

下载每个文件为纯文本文件，以UTF-8编译并且将这些文件存储在一个临时目录中，如/tmp/gutenberg。

```
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace/file$ ll
total 3612
drwxrwxr-x 2 hadoop hadoop    4096 Apr 17 14:46 ./
drwxrwxr-x 3 hadoop hadoop    4096 Apr 17 14:32 ../
-rw-rw-r-- 1 hadoop hadoop     237 Apr 17 14:32 input1.txt
-rw-rw-r-- 1 hadoop hadoop  674570 Apr 17 14:45 pg20417.txt
-rw-rw-r-- 1 hadoop hadoop 1580890 Aug 17  2017 pg4300.txt
-rw-rw-r-- 1 hadoop hadoop 1428841 Apr  7  2015 pg5000.txt
```

## 将本地示例数据拷贝到HDFS

首先在HDFS中创建一个子目录，然后拷贝文件过来（如果input已存在先删除再创建，以免影响测试结果）。

```bash
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace$ hdfs dfs -mkdir input
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace$ hdfs dfs -ls
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2018-04-17 14:51 input
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace$ hdfs dfs -put /home/hadoop/workspace/file/pg*.txt input
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace$ hdfs dfs -ls input
Found 3 items
-rw-r--r--   1 hadoop supergroup     674570 2018-04-17 14:53 input/pg20417.txt
-rw-r--r--   1 hadoop supergroup    1580890 2018-04-17 14:53 input/pg4300.txt
-rw-r--r--   1 hadoop supergroup    1428841 2018-04-17 14:53 input/pg5000.txt
```

## 运行MapReduce作业

运行MapReduce作业，敲入如下命令：

```bash
hadoop jar /usr/local/src/hadoop-3.1.0/share/hadoop/tools/lib/hadoop-streaming-3.1.0.jar -file mapper.py -mapper mapper.py -file reducer.py -reducer reducer.py -input input/* -output output-first
```

查看`output-first`目录确保程序执行正常：

```bash
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace$ hdfs dfs -ls output-first
Found 2 items
-rw-r--r--   1 hadoop supergroup          0 2018-04-17 15:03 output-first/_SUCCESS
-rw-r--r--   1 hadoop supergroup     878847 2018-04-17 15:03 output-first/part-00000
```

将文件从HDFS中拷入到你本地文件系统中

```bash
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace$ mkdir /home/hadoop/workspace/file/output-first
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace$ hdfs dfs -get output-first/* /home/hadoop/workspace/file/output-first/
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace$ cd /home/hadoop/workspace/file/output-first/
hadoop@iZwz9367lkujh8ulgxc2cwZ:~/workspace/file/output-first$ ls
part-00000  _SUCCESS
```

一般情况下，Hadoop对每个reducer产生一个输出文件；在我们的示例中，然而它将只创建单个文件，因为输入的文件都很小。

如果你想要在运行的时候修改Hadoop参数，如增加Reduce任务的数量，你可以使用-D选项：

```bash
-D mapred.reduce.tasks=16
```

只能指定reduce的task数量不能指定map的task数量。

# 改进Mapper和Reducer代码

上面的Mapper和Reducer例子应该给你提供了一种思路，关于如何创建第一个MapReduce程序。重点是代码简洁和易于理解，特别是对于Python语言的初学者。在现实程序中，你可能想要通过[Python的迭代器和生成器](http://www.ibm.com/developerworks/library/l-pycon.html)来优化你的代码。

一般来说，迭代器和生成器有一个优点：序列中的元素在你需要它的时候才会生成。计算资源昂贵或内存紧缺的时候很有用。

注意：下面的Map和Reduce脚本只有运行在Hadoop环境中才会正常工作，即在 MapReduce任务中作为Mapper和Reducer。这表示在本地运行的测试命令"cat DATA | ./mapper.py | sort -k1,1 | ./reducer.py"不会正常工作，因为一些功能是由Hadoop来完成的。

准确地说，我们计算了一个单词出现的次数，例如("foo", 4)，只有恰巧相同的单词（foo）相继出现多次。然而，在大多数情况下，我们让Hadoop在Map和Reduce过程时自动分组(key, value)对这样的形式，因为Hadoop在这方面比我们简单的Python脚本效率更高。

## advanced_mapper.py

advanced_mapper.py是改进之后的mapper代码：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
Created on 4/17/18 3:23 PM
@author: Chen Liang
@function: 更高级的Mapper，使用Python迭代器和生成器
"""

import sys


def read_input(std_input):
    for line in std_input:
        # 将line分割成单词
        yield line.split()


def main(separator='\t'):
    # 从标准输入STDIN输入
    data = read_input(sys.stdin)
    for words in data:
        # 将结果写到标准输出，此处的输出会作为reduce的输入
        for word in words:
            print('{}{}{}'.format(word, separator, 1))

if __name__ == "__main__":
    main()

```

## advanced_reducer.py

advanced_reducer.py是改进之后的reducer代码：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
Created on 4/17/18 3:23 PM
@author: Chen Liang
@function: 更高级的Reducer，使用Python迭代器和生成器
"""

from itertools import groupby
from operator import itemgetter
import sys


def read_mapper_output(std_input, separator='\t'):
    for line in std_input:
        yield line.rstrip().split(separator, 1)


def main(separator='\t'):
    # 从STDIN输入
    data = read_mapper_output(sys.stdin, separator=separator)
    # groupby通过word对多个word-count对进行分组，并创建一个返回连续键和它们的组的迭代器：
    #  - current_word - 包含单词的字符串（键）
    #  - group - 是一个迭代器，能产生所有的["current_word", "count"]项
    # itemgetter: 用于获取对象的哪些维的数据，itemgetter(0)表示获取第0维
    for current_word, group in groupby(data, itemgetter(0)):
        try:
            total_count = sum(int(count) for current_word, count in group)
            print('{}{}{}'.format(current_word, separator, total_count))
        except ValueError:
            pass

if __name__ == '__main__':
    main()

```

代码改进结束。

---

参考：

- https://emunix.emich.edu/~sverdlik/COSC472/WritingAnHadoopMapReduceProgramInPython-MichaelG.Noll.html
- https://python.freelycode.com/contribution/detail/307
- https://hadoop.apache.org/docs/r1.2.1/streaming.html#Hadoop+Streaming
- https://wiki.apache.org/hadoop/HadoopStreaming
- https://blog.csdn.net/dongtingzhizi/article/details/12068205
- https://www.cnblogs.com/dreamer-fish/p/5522687.html