---
title: Python文件打包与包内数据读写
date: 2018-07-17
categories:
- Python
tags:
- Road-mark
---

​	Python 写起来是很方便，不过需要发布的时候却总是有很多的麻烦事。

<!--more-->

## 添加数据文件到包内

### 项目结构

> 在开始制作分发包之前，需要先对文件结构有一个清晰的认识，这里我用一个`funny_joke`的示例来说明一下。

```shell
E:.
│  MANIFEST.in
│  setup.py
│  
└─funny
    │  main.py
    │  __init__.py
    │  
    └─data
            funny_joke.txt
```

这个项目的树状图如上所示，我们的主程序放在了`funny\main.py`内，而我们需要从`funny\data`中导入数据文件`funny_joke.txt`。在打包的时候我们也希望能够**保持**这个结构，也就是说在.egg包内数据文件的路径也是`funny\data\funny_joke.txt`。

### MANIFEST.in

`MANIFEST.in`文件包含Python在打包时读取的需要打包的附属文件的相对路径，当没有`MANIFEST.in`时，Python会依据`setup.py`中给出的打包文件进行打包，这种方法暂且不提。

根据文件结构，我们需要的文件在`funny\data`中。在`MANIFEST.in`中，用`include`表明需要打包的文件。

```shell
include funny\data\*
```

`*`表示导入所有文件，在文件路径和文件名之间有空格。

### setup.py

`setup.py`中只需要一行`include_package_data=True`就可以了，完整的代码如下：

```python
from setuptools import setup, find_packages

setup(
    name='Tell a Joke',
    version='1.0.0',
    packages=find_packages(),
    include_package_data=True,
)
```

`setup`函数还有很多其他的设置，可以[查看文档](https://docs.python.org/3.7/distutils/setupscript.html)了解更多的配置，这里就不赘述了。

## 读取包内文件

由于安装包时文件位置的不确定，所以需要先获得当前文件的路径，再进行文件的读写。获取当前路径的方式有两种，一是通过`__file__`获取，二是通过`import pkg_resources`获取。

### \__file__

```python
import os
this_dir, this_filename = os.path.split(__file__)
DATA_PATH = os.path.join(this_dir, "data", "funny_joke.txt")
```

通过`os.path.spilt(__file__)`函数可以获得该文件的文件名和文件路径，这里我们不需要文件名，所以只关注文件路径就行了。获取了文件路径之后，通过`os.path.join(this_dir, "path_to_the_file")`取得数据文件的路径，其中`this_dir`表示的是当前运行的py文件的路径，后面的`"path_to_the_file"`是**数据文件相对于主程序的相对路径**。对于示例的情况来说，代码如上。

> 这个方法仅适用于数据文件的路径可以被表示的情况，在分发包（如egg）里的数据文件就无法通过这样的方式读取，其次`py2exe`打包的文件也无法读取，因为数据文件是以zip形式储存的。

### pkg_resources

```python
import pkg_resources
DATA_PATH = pkg_resources.resource_filename('funny.main', 'data/funny_joke.txt')
```

`pke_resources.resource_filename(package_or_requirement, resource_name)`中，`package_or_requirement`填写的是库的名称，即`funny.main`。`resource_name`处填写数据文件相对于库文件所在路径的相对路径。

当然，`package_or_requirement`处填`funny`和`funny.main`都是一样的效果，`funny`指向的是`funny\__init__.py`，而`funny.main`指向的是`funny\main.py`，实际上都在同一个文件夹，所以对后面的`resource_name`没有影响。如果你的项目比较复杂，需要根据`package_or_requirement`填写的文件路径补充相应的相对路径。

看起来很复杂？**实际上并不是!**`pkg_resources.resource_filename()`函数的处理方式只要理解了，就能很轻易地掌握这个方法：

1. 首先在包中寻找`funny\main.py`所在的文件夹。如果这个文件包含在文件夹内，那么就直接获得路径；如果这个文件被压缩成egg，那么它会解压egg文件到`...\Python-Eggs\Cache`内。这里我们的文件包含在egg文件里。

   ```shell
   C:.
   └─Cache
       └─tell_a_joke-1.0.0-py3.5.egg-tmp
           └─funny
   # 先获得了这样的一个直达funny的路径
   ```

2. 之后它将`resource_name`，也就是数据文件**关于`funny`的相对路径**，代入到1中得到的文件夹中，如果能够找到文件（或文件夹）的话，就返回整个路径。

   ```shell
   C:.
   └─Cache
       └─tell_a_joke-1.0.0-py3.5.egg-tmp
           └─funny
               └─data
                       funny_joke.txt
   # 然后查找.\data\funny_joke.txt是否存在，存在则获得其路径
   ```


### 对比

这两种方法各有优劣，`__file__`无法从egg或zip中获得路径，而`pkg_resources`虽然能够从egg中获取文件路径，但是当对文件进行更改的时候，只会对复制在`Cache`下的数据文件进行更改，而egg中的源文件不变。如果不小心清理了临时文件，更改过的数据文件就没了。所以说除非是**特别小**的数据量，或者是**对数据写入没要求**的程序，最好还是放在文件夹内，方便保存。

最后以一张对比表格结束战斗。

|          |             \__file__              |         pkg_resources         |
| :------: | :--------------------------------: | :---------------------------: |
| 适用范围 |              文件夹内              |      文件夹内，压缩包内       |
| 方便程度 | 写起来不是那么费劲，不用写绝对路径 |  实际上获得的是一个绝对路径   |
| 文件读写 |        读，写（仅限文件夹）        | 读（全方位），写(仅限文件夹） |

