---
title: python2-dialog模块使用总结
date: 2019-3-18 15:11:42
tags: 

- python
- dialog
---

### 简单示例

```python
#!/usr/bin/env python

import locale
from dialog import Dialog

# This is almost always a good thing to do at the beginning of your programs.
locale.setlocale(locale.LC_ALL, '')

# You may want to use 'autowidgetsize=True' here (requires pythondialog >= 3.1)
d = Dialog(dialog="dialog")
# Dialog.set_background_title() requires pythondialog 2.13 or later
d.set_background_title("My little program")
# For older versions, you can use:
#   d.add_persistent_args(["--backtitle", "My little program"])

# In pythondialog 3.x, you can compare the return code to d.OK, Dialog.OK or
# "ok" (same object). In pythondialog 2.x, you have to use d.DIALOG_OK, which
# is deprecated since version 3.0.0.
if d.yesno("Are you REALLY sure you want to see this?") == d.DIALOG_OK:
    d.msgbox("You have been warned...")

    # We could put non-empty items here (not only the tag for each entry)
    code, tags = d.checklist("What sandwich toppings do you like?",
                             choices=[("Catsup", "",             False),
                                      ("Mustard", "",            False),
                                      ("Pesto", "",              False),
                                      ("Mayonnaise", "",         True),
                                      ("Horse radish","",        True),
                                      ("Sun-dried tomatoes", "", True)],
                             title="Do you prefer ham or spam?",
                             backtitle="And now, for something "
                             "completely different...")
    if code == d.DIALOG_OK:
        # 'tags' now contains a list of the toppings chosen by the user
        pass
else:
    code, tag = d.menu("OK, then you have two options:",
                       choices=[("(1)", "Leave this fascinating example"),
                                ("(2)", "Leave this fascinating example")])
    if code == d.DIALOG_OK:
        # 'tag' is now either "(1)" or "(2)"
        pass
```

### 以mixform为例的参数解释

```python
# Dialog.mixedform(text, elements, height=0, width=0, form_height=0, **kwargs)

# (label, yl, xl, item, yi, xi, field_length, input_length, attributes)
# label is a string that will be displayed at row yl, column xl. item is a string giving the initial value for the field, which will be displayed at row yi, column xi (row and column numbers starting from 1).

#attributes is an integer interpreted as a bit mask with the following meaning (bit 0 being the least significant bit):
Bit number	Meaning
0	the field should be hidden (e.g., a label)
1	the field should be read-only (e.g., a password)
# 经实践得0为label，1为password，与官网所述刚好相反

# 一个字符代表一列
d.mixedform('text',
            [('one',1,1,'item',1,10,10,20,0),
             ('passwd', 2, 1, '', 2, 10, 10, 20, 1)],
            9,      #height 表格高度，比form_height多5，下面的操作按钮刚好正常显示
            50,     #width 表格宽度
            3)      #form_height 表格行数
```

