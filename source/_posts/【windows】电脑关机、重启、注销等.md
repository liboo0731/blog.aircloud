---
title: 【windows】电脑关机、重启、注销等
date: 2017-10-17 20:44:12
tags:
- windows
- cmd
---

如何在win cmd中进行电脑重启

- shutdown.exe -a　 取消关机

- shutdown.exe -s   关机

- shutdown.exe -f　 强行关闭应用程序

- shutdown.exe -m  \\计算机名　控制远程计算机

- shutdown.exe -i　 显示“远程关机”图形用户界面，但必须是Shutdown的第一个参数

- shutdown.exe -l　 注销当前用户

- shutdown.exe -r　 关机并重启

- shutdown.exe -s -t 时间　设置关机倒计时（shutdown -r -t 5   5秒后重启计算机）

- shutdown.exe -h   休眠