---
title: 【onda】Onda obook10 安装CentOS7 系统
date: 2019-07-01 14:53:19
tags:
- onda
---

### 问题：U盘安装长时间停留在 starting dracut initqueue hook

1. 安装引导界面按 “e” 进行编辑，修改完成执行 “Ctrl + X”

    ![2.png](2.png)

```
默认信息：
setparams 'Install CentOS Linux 7'
linuxefi /images/pxeboot/vmlinuz inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet
initrdefi /images/pxeboot/initrd.img

修改为：
setparams 'Install CentOS linux 7'
linuxefi /images/pxeboot/vmlinuz linux dd nomodeset quiet
initrdefi /images/pxeboot/initrd.img
```

2. 在显示信息中找到优盘设备，比如：sdb4，记住此设备路径

    ![3.png](3.png)

3. 重启，再次按 “e” 进入引导界面，执行编辑，并在修改完成后执行 “Ctrl + X”

```
setparams 'Install CentOS Linux 7'
linuxefi /images/pxeboot/vmlinuz inst.stage2=hd:/dev/sdb4 nomodeset quiet
initrdefi /images/pxeboot/initrd.img
```

4. 按需求开始正常安装



 