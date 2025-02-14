---
layout: blog
title: 暴力破解压缩包密码--John the Ripper/HashCat
categories: 瞎折腾
date: 2023-03-24 08:48:15
---

## 起因

某实验课程，四节课要啰嗦大概两节课的时间，且实验指导书设置了"Cst.\*\*\*\*\*\*"（\*为0-9/a-z/A-Z）的密码，网上随便下一个密码破解工具会很难破解（显示时间>一年），所以想寻求一些高效的破解方法。

## 原理

- John the Ripper（以下简称JtR）：此工具有John和file2john两种子工具
  - file2john会识别文件特征，提取出一串哈希值，如：运行`zip2john test.zip`会得到类似`$pkzip2$1*1*...d*$/pkzip2$`，虽然是哈希值，但是这玩意可以很大（比如我提取出来6M的哈希值文件），具体原理未知
  - John主程序会对上面的哈希值进行密码爆破，计算密码的哈希与其比对
- HashCat：此工具类似JtR的John，可以爆破密码哈希值与输入的哈希值比对，但是不能单独用于密码破解，需要用file2john提取特征哈希以后才可以破解

两种工具都有尝试，且成功配好，但是都有其应用上的限制，建议两个都准备好，以备不时之需）

## HashCat

配置环境为：

- Ubuntu 20.04 
- NVIDIA Tesla A100-32GB with CUDA

### 安装

参考：[hashcat/hashcat: World's fastest and most advanced password recovery utility (github.com)](https://github.com/hashcat/hashcat)

需要已经安装好NVIDIA Driver和CUDA Toolkit，注意这玩意有点讲究，有详细的版本对应表，可以上网查一下

直接在上面的hashcat GitHub repo中下载最新release的7z包，`7z x`解压

#### 简单测试

安装好以后需要测试hashcat能否使用：

``` shell
hashcat -a 0 -m 500 example500.hash example.dict
```

除了要注意是否可以跑通外，还需要注意输出的Device Info里面有没有GPU

### 使用

#### 准备哈希文件

之前说过，Hashcat爆破之前要准备好hash文件，还是要用到JtR。JtR可以在Kali虚拟机上面直接使用（只不过无法调用GPU），如果没有kali的话，可以参考下一节JtR的内容在Windows安装并编译JtR

安装好以后，使用命令

``` shell
zip2john test.zip > hash.txt
```

此处需要注意：

- hash.txt一定要是**UTF-8**格式存储的，如果不是，请用vscode等工具将其用UTF-8编码保存
- hash.txt的格式是 `test.zip:test.pdf:$pkzip2$1*1...d*$/pkzip2$:test.pdf:test.zip`，请掐头去尾，只保留`$pkzip2$1*1...d*$/pkzip2$`的内容
- hash.txt不能有**空格、不可见字符**等

#### 设置密码范围

我们使用mask_attack模式`-a 3`来自定义密码，具体如下：

以下是mask_attack的字符集：

> - ?l = abcdefghijklmnopqrstuvwxyz
> - ?u = ABCDEFGHIJKLMNOPQRSTUVWXYZ
> - ?d = 0123456789
> - ?h = 0123456789abcdef
> - ?H = 0123456789ABCDEF
> - ?s = «space»!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
> - ?a = ?l?u?d?s
> - ?b = 0x00 - 0xff

那么假设我要爆破的密码范围是"boulebard"+3位数字，就可以用`-a 3 boulevard?d?d?d` 来规定

如果是自定义字符集呢？可以用`-1 字符集1内容, -2 字符集2内容`自定义字符集，然后用`?1?2`使用字符集

例如，对于"Cst.\*\*\*\*\*\*"（\*为0-9/a-z/A-Z），需要定义数字+大小写字母的字符集：`-1 ?l?u?d`

然后，密码则表达为`Cst.?1?1?1?1?1?1`

#### 设置爆破模式

对于不同文件类型有不同的爆破模式`-m mode_num`，需要根据被爆破文件的格式和hash.txt的内容来在[example_hashes \[hashcat wiki\\]](https://hashcat.net/wiki/doku.php?id=example_hashes)中查询到模式码，如：

![image-20230324093110247](/images/john/image-20230324093110247.png)

对于上面的hash.txt，就只需要尝试`-m 17200/17210/17220/17225/17230`

所以，最后的调用代码为：

``` shell
./hashcat.bin -a 3 -m 17200 -1 ?l?u?d hash.txt Cst.?1?1?1?1?1?1
```

### 问题&缺点

我遇到的两个问题都在[Signature unmatched No hashed loaded. (hashcat.net)](https://hashcat.net/forum/thread-11358-post-57870.html#pid57870)提出

- 第一个问题是之前提到的，hash.txt没有用UTF-8保存，注意以下即可解决
- 第二个问题就要看运气了，某些文件提取出的hash很大，导致hashcat无法处理，这种情况就只能去用JtR破解了

## John the Ripper

配置环境为

- Windows11 22H2
- RTX2070-8GB

### 安装

参考：[john/INSTALL-WINDOWS at bleeding-jumbo · openwall/john (github.com)](https://github.com/openwall/john/blob/bleeding-jumbo/doc/INSTALL-WINDOWS)

在Windows下，需要用到CryWin安装，到[Cygwin Installation](https://cygwin.com/install.html)下载`setup-x86_64.exe`，放到`C:/crywin64/`目录下，然后在Windows PowerShell执行

``` shell
C:\cygwin64\setup-x86_64.exe -q -P gcc-core -P libgcc1 -P make -P perl
C:\cygwin64\setup-x86_64.exe -q -P libssl-devel -P libbz2-devel
C:\cygwin64\setup-x86_64.exe -q -P libgmp-devel -P zlib-devel
C:\cygwin64\setup-x86_64.exe -q -P libOpenCL-devel -P libcrypt-devel
```

输入命令以后安装的过程基本上是自动的，只不过第一次安装的时候会要你选源，选一个国内的即可

然后下载[https://github.com/openwall/john/archive/bleeding-jumbo.zip](https://github.com/openwall/john/archive/bleeding-jumbo.zip)，解压放到`C:/crywin64/home/[用户名]`目录

打开crywin64 terminal（点击图标即可），进入上面解压的目录，开始编译：

``` shell
./configure && make -s clean && make -sj4
make windows-package
```

编译好以后，测试：

``` shell
..\run\john --test=0
```

然后查看你的OpenCL设备

``` shell
..\run\john --list=opencl-devices                                               
```

但是有时候，很不幸，你可能**只能看到CPU而看不到GPU**

这时候，参考[Comprehensive Guide to John the Ripper. Part 1: Introducing and Installing John the Ripper - Ethical hacking and penetration testing (miloserdov.org)](https://miloserdov.org/?p=4961#131)中的方法：

1. 在`C:\cygwin64\bin\`中找到`cygOpenCL-1.dll`，重命名为`cygOpenCL-1.dll.bac`
2. 把`C:\Windows\System32\OpenCL.dll`复制到`C:\cygwin64\bin\`，重命名为`cygOpenCL-1.dll`

这时再次查看设备，就可以查看到显卡了

### 使用

同样的，我们使用mask模式来爆破密码，其语法参考[john/MASK at bleeding-jumbo · openwall/john (github.com)](https://github.com/openwall/john/blob/bleeding-jumbo/doc/MASK)

语法和hashcat类似，自定义字符集可以用`-1=?l?u`表示，对于Cst.那个密码，爆破命令如下：

``` shell
./run/john/ -1=?d?l?u --mask=Cst.?1?1?1?1?1?1 hast.txt
```

但是需要注意的是，即使查看到了GPU，JtR也会默认使用CPU，需要用`--format=wpapsk-opencl -dev=1`来设置使用OpenCL与GPU，但是，很遗憾，根据[Issue #4070 · openwall/john (github.com)](https://github.com/openwall/john/issues/4070)****的内容，zip文件并不支持OpenCL GPU破解，不过亲测对于上面的6位大小写+数字密码，即使是i5-10400也只需要20分钟就可以成功装好



