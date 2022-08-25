---
categories:
  - 随笔
date: '2018-02-01T14:55:27'
description: ''
tags:
  - hmac
  - sha
  - crypto
title: HmacSHA1和原生SHA1的比较
---




首先来看一段HmacSHA1加密和SHA1加密的代码

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
Created on 1/31/18 10:03 AM
@author: Chen Liang
@function: HmacSHA1 vs SHA1
"""

import sys

reload(sys)
sys.setdefaultencoding('utf-8')
import hashlib
import hmac


def sha1(msg):
    """
    sha1加密
    :param msg:
    :return: 长度40位的摘要信息
    """
    sha = hashlib.sha1()
    sha.update(msg)
    return sha.hexdigest()


def hmac_sha1(key, msg):
    """
    hmac sha1加密
    :param key: 密钥
    :param msg: 待加密消息
    :return: 长度40位的摘要信息
    """
    m = hmac.new(key, msg, hashlib.sha1)
    return m.hexdigest()


print hmac_sha1('FKEwTiz9Te0FWlqkS4g8hEdqAsPZfdR4', 'me')
print sha1('me')
```

输出结果为

```
1db0e9132a8dff51e3a4d47497e29a500087da9a
b1c1d8736f20db3fb6c1c66bb1455ed43909f0d8
```

从结果中可以发现，HmacSHA1算法和SHA1算法都可以为任意长的消息生成一个20字节(160bit)的固定大小的输出，那么他们的区别在哪儿呢？

其实答案是很简单的。

在[HMAC vs. raw SHA-1](http://dev.ionous.net/2009/03/hmac-vs-raw-sha-1.html)这篇文章中有一个有趣的解释，翻译过来大意如下

<!--more-->

写在前面，请不要在意具体的摘要计算结果，重点在于解释的趣味性和简洁明了

> 假设你想向你喜欢的人表白，你很想拿出一首美丽的14行诗，但是最后你决定只说一句简单的`"i love you"`。
>
> 你要表白的信息能够完好无损的传达给喜欢的妹子，但是你又不想其他的人知道，那么可以了解一些关于密码哈希的知识，使用SHA-1算法从消息中生成一个摘要。
>
> `"i love you"`对应的SHA-1摘要是：`bb7b1901d99e8b26bb91d2debdb7d7f24b3158cf`
>
> 你喜欢的妹子接收到消息后，使用SHA-1算法重新计算出摘要和你发送的摘要进行比较。如果匹配就表示消息正确。
>
> 但是总有那么些刁民打算拦截你的信息，然后用另一个消息`"don't call me anymore"`替代掉，然后生成一个全新的摘要：`e267e18f05cb6ea3b10b761bbac21a0f92bb8d0d`。你喜欢的妹子收到消息之后摘要信息无法匹配得上，都有些难以置信了。
>
> 事情看起来很严峻，但是你向妹子解释了一番，保证以后再也不会发生这样类似的事情，你和妹子约定在计算hash摘要信息时在消息前面加上文本`"our secret key."`，也就是新的完整的信息是`"our secret key.i love you"`。就这样相同的消息就会产生下面这样的摘要信息：
>
> `e0759e9b59bdd6d864d29ce3a502adb6257f7615`， 原文的这个值计算有错，评论中有提出。
>
> 这时候如果那些刁民只是简单的替换摘要信息就不生效了。因为你妹子使用key+msg的方式得到的结果和替换之后的摘要信息匹配不上。这样只要别人不知道你的密钥就没有办法产生虚假的消息。
>
> 但是还有一个问题，问题在于SHA-1和HMAC之间的区别。
>
> SHA-1是使用迭代算法进行计算的，首先一个接一个地将消息分成64个字节的块，然后把这些块组合在一起来产生20个字节的摘要信息。 但是，由于你的消息可以是任意长度的，并且由于SHA通过其迭代性质在64字节的块之后继续计算块，这时候问题就出现了。
>
> 那些刁民打算再次改变你的信息，他们可能只是将更多的数据添加你的消息里面，由于你的密钥在前面的块中已经经过了计算，这时候添加在后面的消息不会受到你的密钥影响。
>
> 如果在消息后面简单的添加上"but please don't call me anymore"，计算新的摘要并发送给你喜欢的妹子，妹子会以为整段消息就是你的意思。（此处具体计算方法需要参照sha1算法的实现）
>
> 就这样一个大写的GG刻在了你的脸上（欲哭无泪）！！！
>
> 但是也不用慌，我们还有HMAC，HMAC解决了这个问题，HMAC在整个hash过程中能有效的密封消息隐藏密钥，并且不能在尾部追加数据。具体的解决办法参见HMAC的实现。
>
> 根据维基百科，没有发现任何已知的HMAC消息扩展攻击。
>
> 恭喜你，妹子到手了，请开始你的性福之旅吧（啊呸，傻逼输入法，是幸福）！！！

到这里其实HmacSHA1加密和SHA1加密的区别就很明显了，希望大家也都能理解。

---

参考：

1. [HMAC vs. raw SHA-1](http://dev.ionous.net/2009/03/hmac-vs-raw-sha-1.html)
2. [极客学院-hmac](https://wiki.jikexueyuan.com/project/explore-python/Standard-Modules/hmac.html)
3. [极客学院-hashlib](https://wiki.jikexueyuan.com/project/explore-python/Standard-Modules/hashlib.html)