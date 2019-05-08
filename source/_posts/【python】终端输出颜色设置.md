---
title: 【python】终端输出颜色设置
date: 2019-05-08 19:01:19
tags:
- python
---

### termcolor模块

#### 帮助文档

```python
FUNCTIONS
    colored(text, color=None, on_color=None, attrs=None)
    Colorize text.
    
    Available text colors:
        red, green, yellow, blue, magenta, cyan, white.
    
    Available text highlights:
        on_red, on_green, on_yellow, on_blue, on_magenta, on_cyan, on_white.
    
    Available attributes:
        bold, dark, underline, blink, reverse, concealed.
    
    Example:
        colored('Hello, World!', 'red', 'on_grey', ['blue', 'blink'])
        colored('Hello, World!', 'green')
    
    cprint(text, color=None, on_color=None, attrs=None, **kwargs)
    Print colorize text.
    
    It accepts arguments of print function.

DATA
    ATTRIBUTES = {'blink': 5, 'bold': 1, 'concealed': 8, 'dark': 2, 'rever...
    COLORS = {'blue': 34, 'cyan': 36, 'green': 32, 'grey': 30, 'magenta': ...
    HIGHLIGHTS = {'on_blue': 44, 'on_cyan': 46, 'on_green': 42, 'on_grey':...
    RESET = '\x1b[0m'
    VERSION = (1, 1, 0)
    __ALL__ = ['colored', 'cprint']
    print_function = _Feature((2, 6, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0)...
```

#### 示例代码

```python
import sys
from termcolor import colored, cprint
 
# 红色背景字符不发光
text = colored('Hello, World!', 'red', attrs=['reverse', 'blink'])
print(text)
 
# 红色背景上显示绿色字符
cprint('Hello, World!', 'green', 'on_red')
 
# 青蓝色背景上显示绿色字符
print_red_on_cyan = lambda x: cprint(x, 'red', 'on_cyan')
print_red_on_cyan('Hello, World!')
print_red_on_cyan('Hello, Universe!')
 
for i in range(10):
    cprint(i, 'magenta', end=' ')
 
# 红色字符加粗
cprint("Attention!", 'red', attrs=['bold'], file=sys.stderr)
```



*参考：[python库termcolor用法](https://www.cnblogs.com/everfight/p/python_termcolor.html)*



### 自定义颜色显示

#### 显示格式

\033[显示方式;前景色;背景色m

#### 显示方式

| 显示方式 | 意义         |
| -------- | ------------ |
| 0        | 终端默认设置 |
| 1        | 高亮显示     |
| 4        | 使用下划线   |
| 5        | 闪烁         |
| 7        | 反白显示     |
| 8        | 不可见       |

#### 前景色和背景色

| 前景色 | 背景色 | 显示颜色 |
| :----: | :----: | :------: |
|   30   |   40   |   黑色   |
|   31   |   41   |   红色   |
|   32   |   42   |   绿色   |
|   33   |   43   |   黃色   |
|   34   |   44   |   蓝色   |
|   35   |   45   |  紫红色  |
|   36   |   46   |  青蓝色  |
|   37   |   47   |   白色   |

#### ANSI控制码的说明 

|     ANSI控制码     |          说明          |
| :----------------: | :--------------------: |
|       \33[0m       |      关闭所有属性      |
|       \33[1m       |       设置高亮度       |
|       \33[4m       |         下划线         |
|       \33[5m       |          闪烁          |
|       \33[7m       |          反显          |
|       \33[8m       |          消隐          |
| \33[30m -- \33[37m |       设置前景色       |
| \33[40m -- \33[47m |       设置背景色       |
|       \33[nA       |      光标上移n行       |
|       \33[nB       |      光标下移n行       |
|       \33[nC       |      光标右移n行       |
|       \33[nD       |      光标左移n行       |
|      \33[y;xH      |      设置光标位置      |
|       \33[2J       |          清屏          |
|       \33[K        | 清除从光标到行尾的内容 |
|       \33[s        |      保存光标位置      |
|       \33[u        |      恢复光标位置      |
|      \33[?25l      |        隐藏光标        |
|      \33[?25h      |        显示光标        |

#### 自定义函数

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

def text_colors(text, fcolor=None, bcolor=None, style=None):
    '''
    自定义字体样式及颜色
    '''
    # 字体颜色
    fg={
       'black': '\033[30m',     #字体黑
       'red': '\033[31m',       #字体红
       'green': '\033[32m',     #字体绿
       'yellow': '\033[33m',    #字体黄
       'blue': '\033[34m',      #字体蓝
       'magenta': '\033[35m',   #字体紫
       'cyan': '\033[36m',      #字体青
       'white':'\033[37m',      #字体白
        'end':'\033[0m'         #默认色
    }
    # 背景颜色
    bg={
       'black': '\033[40m',     #背景黑
       'red': '\033[41m',       #背景红
       'green': '\033[42m',     #背景绿
       'yellow': '\033[43m',    #背景黄
       'blue': '\033[44m',      #背景蓝
       'magenta': '\033[45m',   #背景紫
       'cyan': '\033[46m',      #背景青
       'white':'\033[47m',      #背景白
    }
    # 内容样式
    st={
        'bold': '\033[1m',      #高亮
        'url': '\033[4m',       #下划线
        'blink': '\033[5m',     #闪烁
        'seleted': '\033[7m',   #反显
    }

    if fcolor in fg:
        text = fg[fcolor] + text + fg['end']
    if bcolor in bg:
        text = bg[bcolor] + text + fg['end']
    if style in st:
        text = st[style] + text + fg['end']
    return text

#使用方法：
print(text_colors('文本内容','字体颜色','背景颜色','字体样式'))
```



*参考：*

*[python终端颜色设置](http://www.cnblogs.com/zhanmeiliang/p/5948947.html)*

*[Python控制台输出颜色](https://blog.csdn.net/qq_33282586/article/details/80637242)*

