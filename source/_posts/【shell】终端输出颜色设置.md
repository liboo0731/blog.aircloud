---
title: 【shell】终端输出颜色设置
date: 2019-05-8 10:40:49
tags:
- shell
---

### 两种方法，两种样式：

```shell
#!/bin/sh

echo
echo -e "\\033[0;31m系统颜色设置代码调试，此颜色为一号颜色 - 红！"
echo
echo -e "\\033[0;32m系统颜色设置代码调试，此颜色为二号颜色 - 绿！"
echo
echo -e "\\033[0;33m系统颜色设置代码调试，此颜色为三号颜色 - 黄！"
echo
echo -e "\\033[0;34m系统颜色设置代码调试，此颜色为四号颜色 - 蓝！"
echo
echo -e "\\033[0;35m系统颜色设置代码调试，此颜色为五号颜色 - 紫！"
echo
echo -e "\\033[0;36m系统颜色设置代码调试，此颜色为六号颜色 - 青！"
echo
echo -e "\\033[0;39m系统颜色设置代码调试，此颜色为九号颜色 - 白！"

BLACK='\E[1;30m'
RED='\E[1;31m'
GREEN='\E[1;32m'
YLW='\E[1;33m'
BLUE='\E[1;34m'
PURPLE='\E[1;35m'
CYAN='\E[1;36m'
WHITE='\E[1;39m'
RES='\E[0m'

echo
echo -e "${BLACK}系统颜色设置代码调试，此颜色为一号颜色 - 黑！${RES}"
echo
echo -e "${RED}系统颜色设置代码调试，此颜色为一号颜色 - 红！${RES}"
echo
echo -e "${GREEN}系统颜色设置代码调试，此颜色为一号颜色 - 绿！${RES}"
echo
echo -e "${YLW}系统颜色设置代码调试，此颜色为一号颜色 - 黄！${RES}"
echo
echo -e "${BLUE}系统颜色设置代码调试，此颜色为一号颜色 - 蓝！${RES}"
echo
echo -e "${PURPLE}系统颜色设置代码调试，此颜色为一号颜色 - 紫！${RES}"
echo
echo -e "${CYAN}系统颜色设置代码调试，此颜色为一号颜色 - 青！${RES}"
echo
echo -e "${WHITE}系统颜色设置代码调试，此颜色为一号颜色 - 白！${RES}"
echo
```

