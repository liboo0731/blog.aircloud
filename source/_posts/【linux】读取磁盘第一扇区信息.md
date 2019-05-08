---
title: 【linux】读取磁盘第一扇区信息
date: 2019-05-08 13:08:14
tags:
- linux
- disk
---

### 查看磁盘第一个扇区的信息

### 1. 使用dd命令

```shell
[root@host-172-16-2-221 test]# dd if=/dev/vda of=part.dump bs=512 count=1
1+0 records in
1+0 records out
512 bytes (512 B) copied, 0.000240905 s, 2.1 MB/s
[root@host-172-16-2-221 test]# vi part.dump

```

> vim中把文件转换为16进制来显示：
>
> :%!xxd
>
> 返回正常显示：
>
> :%!xxd -r



### 2. 使用c语言输出

```c
#include <stdio.h>

void print(char c){
    int m = c;
    int high = 0x000000f0, low = 0x0000000f;
    high &= m;
    high >>= 4;
    low &= m;
    printf("%1X%1X ", high, low);
}

int main(){
    FILE* fd = fopen("/dev/vda", "rb+");
    char buffer[512] = {0};
    fseek(fd, 0, SEEK_SET);
    char buffer2[512] = {0};
    fread(buffer2, 512, 1, fd);
    int i = 1;
    for (; i <= 512; i++){
        print(buffer2[i-1]);
        if (i % 16 == 0){
            printf("\n");
        }
        else if (i % 8 == 0){
            printf("    ");
        }
    }
    fclose(fd);
    return 0;
}

```



### 3. 使用python输出 

```python
#!/usr/bin/env python
#coding=utf8

with open('/dev/vda', 'rb') as fp:
	hex_list = ["{:02x}".format(ord(c)) for c in fp.read(512)]
	print(hex_list)
    
```

> ord("a")
>
> 它以一个字符（长度为1的字符串）作为参数，返回对应的 ASCII 数值，或者 Unicode 数值
>
> "{:02X}".format(i)
>
> 这个输出是将i以16进制输出，当i是15，输出结果是0F；
>
> {:X}16进制标准输出形式，02是2位对齐，左补0形式。

*参考：[MBR-extractor](https://github.com/shubham0d/MBR-extractor)*

