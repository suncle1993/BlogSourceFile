---
abbrlink: 10731460
alias: 2018/03/22/use-google-translation-api/index.html
categories:
- python
date: '2018-03-22T20:39:46'
description: ''
tags:
- Google
- 翻译
title: 使用Google翻译Api
---









使用Google翻译Api

### 安装Google翻译库

```shell
pip install --upgrade google-cloud-translate
```

<!--more-->

### 设置验证

要运行客户端库，必须首先创建服务帐户并设置环境变量来设置身份验证。

1. 转到Google Cloud Platform控制台中创建服务帐户密钥页面
2. 从服务帐户下拉列表中选择新建服务帐户。
3. 在服务帐户名称字段中输入一个名称。
4. 从角色下拉列表中，选择项目>所有者。
5. 点击创建。 密钥就会下载到您的计算机的JSON文件

将环境变量`GOOGLE_APPLICATION_CREDENTIALS`设置为包含服务帐户密钥的JSON文件的文件路径。在Linux或macOS系统中设置方法如下：

```shell
export GOOGLE_APPLICATION_CREDENTIALS="/home/user/Downloads/service-account-file.json"
```

### 使用客户端库调用翻译Api

代码如下：

```python
# Imports the Google Cloud client library
from google.cloud import translate

# Instantiates a client
translate_client = translate.Client()

# The text to translate
text = u'Hello, world!'
# The target language
target = 'ru'

# Translates some text into Russian
translation = translate_client.translate(
    text,
    target_language=target)

print(u'Text: {}'.format(text))
print(u'Translation: {}'.format(translation['translatedText']))
```

要想将文件中的国家名称批量翻译并输出，可以写出下面这样的代码：

```python
#!/usr/bin/env python
#encoding: utf-8

# Imports the Google Cloud client library
from google.cloud import translate

# Instantiates a client
translate_client = translate.Client()

# The target language
target = 'en'

d = {}
with open('world_country_code.csv', 'r') as fpr:
    for line in fpr.readlines():
        country = line.strip()
        if country.endswith(':'):
            result_line = country
        elif country is '':
            result_line = country
        else:
            # Translates some text into Russian
            translation = translate_client.translate(country, target_language=target)
            result_line = translation['translatedText']
            result_line = '{},{}'.format(country, result_line)
        print result_line
```



---

参考：

- `https://cloud.google.com/translate/docs/reference/libraries#client-libraries-usage-python`

