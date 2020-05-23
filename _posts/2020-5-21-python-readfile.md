---
layout: post
title: Python 匹配不定文件名
tags: python
---


> 在进行数据处理的时候，可能会有数据文件名称不一致的问题，本文使用 fnmatch 进行匹配文件名称然后进行批量读取文件。

##  目录
* 目录
{:toc}
# 一、使用场景

在处理数据的时候，总会出现很多不同的文件需要一一读取，但是因为有些命名不规范导致读取很麻烦，比如说下如下所示：

![example99c2a4d0a4c971b3.png](https://pic.tyzhang.top/images/2020/05/23/example99c2a4d0a4c971b3.png)

需要读取每个文件夹中的HandPhone_keyboard的数据，但是因为命名的不规范，后缀为HTC与Sam，使得读取文件很复杂。python中的`fnmatch`插件可以很好的解决这种文件读取，采用的方法为先将文件夹中的所有的文件名称读取进来，然后使用`fnmatch`进行匹配需要的文件夹名称。

# 二、读取结果

最终读取的结果如下，可以发现，前面是比较有规律的，但是后面的可能是(Samsung_S6)，或者是(HTC_One)，那么就需要进行匹配然后读取出来文件。

```shell
./data/1/1_HandPhone_Keyboard_(Samsung_S6).csv
./data/2/2_HandPhone_Keyboard_(Samsung_S6).csv
./data/3/3_HandPhone_Keyboard_(Samsung_S6).csv
./data/4/4_HandPhone_Keyboard_(Samsung_S6).csv
./data/5/5_HandPhone_Keyboard_(Samsung_S6).csv
./data/6/6_HandPhone_Keyboard_(Samsung_S6).csv
./data/7/7_HandPhone_Keyboard_(Samsung_S6).csv
./data/8/8_HandPhone_Keyboard_(Samsung_S6).csv
./data/9/9_HandPhone_Keyboard_(HTC_One).csv
./data/10/10_HandPhone_Keyboard_(Samsung_S6).csv
./data/11/11_HandPhone_Keyboard_(Samsung_S6).csv
./data/12/12_HandPhone_Keyboard_(HTC_One).csv
./data/13/13_HandPhone_Keyboard_(Samsung_S6).csv
./data/14/14_HandPhone_Keyboard_(HTC_One).csv
./data/15/15_HandPhone_Keyboard_(HTC_One).csv
./data/16/16_HandPhone_Keyboard_(Samsung_S6).csv
./data/17/17_HandPhone_Keyboard_(HTC_One).csv
./data/18/18_HandPhone_Keyboard_(HTC_One).csv
./data/19/19_HandPhone_Keyboard_(Samsung_S6).csv
./data/20/20_HandPhone_Keyboard_(HTC_One).csv
```

# 三、实现代码

代码中主要的就是使用字符串进行拼接出来路径，然后进行读取文件。对于`fnmatch`，实现的方法为首先目录中的所有文件名使用`os.listdir`读取进来，然后使用`fnmatch.fnmatchcase`方法进行匹配，如果匹配成功则输出。其中`pattern = str(i) + '_HandPhone_Keyboard_*.csv'`是进行匹配的规则，其中\*表示后面的所有字符都进行匹配。

```python
#!/usr/bin/env python 
# -*- coding:utf-8 -*-
# author: Seven time: 2020/5/15 
# Describe : 进行读取数据。

import random
import fnmatch
import os


if __name__ == '__main__':

    base_path = "./data/"  # 设置文件路径。
    person_number = 20  # 保存有多少用户
	
    # 最终路径 ./data/20/20_HandPhone_Keyboard_(HTC_One).csv
    # 获取本地路径 ，使用fnmatchcase方法读取出来所有的需要路径。
    paths = []
    for i in range(1, person_number + 1):
        # 获取文件夹中所有的文件目录
        files = os.listdir(base_path + str(i))
        # 设置过滤条件，进行匹配需要的数据，* 表示任意匹配
        pattern = str(i) + '_HandPhone_Keyboard_*.csv'
        for name in sorted(files):
            # 因为有很多的路径，所以需要读取到的本地路径与 pattern有没有匹配，匹配成功的是需要的
            if fnmatch.fnmatchcase(name, pattern):
                # 获取到需要的数据。
                path = base_path + str(i) + "/" + name
                # 存储数据
                paths.append(path)
                print(path)
	
    # 进行读取数据。
    for i in range(1, person_number + 1):
        # 拼接成需要的数据。
        path = paths[i - 1]
        # 读取数据
        try:
            user = pd.read_csv(path)
            print("第" + str(i) + "个数据长度" + str(len(user)))
        except FileNotFoundError:
            pass

```

