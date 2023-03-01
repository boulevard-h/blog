---
layout: blog
title: VOLE-UPSI
date: 2023-03-01 20:24:42
categories: 隐私计算
tags: [PSI, C++, UPSI]
---

本文主要记录了[volePSI](https://github.com/Visa-Research/volepsi)的build，以及用其构造UPSI的过程。

## 环境

部署在腾讯云上的Ubuntu22.04LTS系统，使用IDE为code-server 

## 安装

需要安装libOTe，按照之前的经验，使用比较好的vpn（如pigcha），连接美国节点，开启全局（网卡）代理，即可比较顺利的克隆libOTe。

### 安装命令

首先使用的是`python3 build.py --install`

但是这样是最小化安装，虽然可以运行单元测试，但不能够使用fileBasedPSI。

后续查看了fileBasedPSI.cpp的源码报错信息发现需要启用Coproto的BOOST组件，所以采用`python3 build.py -DCOPROTO_ENABLE_BOOST=ON -DCOPROTO_ENABLE_OPENSSL=ON`编译。

但是这样做又出现一个编译bug，提示找不到libOTe，提交[issue#20](https://github.com/Visa-Research/volepsi/issues/20)后发现是找不到OpenSSL的原因，尝试了一下安装OpenSSL并添加到PATH和G++编译路径后无果，所以干脆取消对于OpenSSL的启用，最终使用如下命令编译成功：

```shell
python3 build.py -DCOPROTO_ENABLE_BOOST=ON -DCOPROTO_ENABLE_OPENSSL=OFF
```

## 构造UPSI

与之前的[OPRF-PSI]([GitHub - peihanmiao/OPRF-PSI: Private Set Intersection in the Internet Setting From Lightweight Oblivious PRF](https://github.com/peihanmiao/OPRF-PSI))需要大量修改API来支持std::thread调用不同，VOLE-PSI有File Based模式：只要在命令行输入调用参数，即可自动读取文件求交，然后将结果输出到文件。这样的调用方式非常方便实现UPSI，根本不需要修改API。

### 实现单次调用

这里只需要使用stdlib.h自带的`system(const char *command)`即可。

使用string类完成调用命令的构造，然后用`sting.c_str()`方法转为const char *即可完成单次PSI命令行调用：

``` cpp
void callPSI(std::string role, std::string inputName, std::string port, bool *FinSig) {
    std::string callPSI = "../volepsi/out/build/linux/frontend/frontend -bin -receiverSize 8192 -senderSize 8192 -useSilver ";
    callPSI = callPSI + "-r " + role + " ";
    callPSI = callPSI + "-in " + inputName + " ";
    callPSI = callPSI + "-ip 127.0.0.1:" + port + " ";

    std::cout << callPSI << std::endl;

    system(callPSI.c_str());
    *FinSig = true;
}
```

### 多线程调用

这里使用的是`std::thread`库来完成多线程，这个库比`pthread`要新。

首先创建线程对象`std::thread tr(function, params)`，由于callPSI函数非常简单，所以没有什么好说的。之前的OPRF-PSI就不一样，要调用非常复杂的API，涉及到函数名前面需要加`&`、部分参数需要`std::ref`修饰等等。

``` cpp
std::thread t30(callPSI,"0","X3.bin","1212",&Sig30);
```

然后使用`detach()`方法调用，这个方法是**非阻塞**的。如果使用`join()`的话，子线程会阻塞父线程，那么就会导致不发同时调用多个PSI，而是按顺序执行。但是用`detach()`又会有一个问题：父线程会在子线程之前终止，导致出现**孤儿进程**。所以需要给callPSI加入一个`bool*`的参数传入，作为子线程结束的标志：

``` cpp
t30.detach();t31.detach();t32.detach();
t03.detach();t13.detach();t23.detach();

while(!(Sig30&&Sig31&&Sig32&&Sig03&&Sig13&&Sig23)){
     usleep(1);
}
```

这里还需要注意一点：Linux系统的sleep单位是秒，这里需要用usleep。

### 添加计时器

开始打算使用ctime标准库的`clock()`计算运行时间，但是不知道为什么测出来不准，所以改用[cryptoTools](https://github.com/ladnir/cryptoTools)库的Timer。

#### 安装cryptoTools库

这个比较好装，clone然后python执行build脚本，期间挂梯子保障git clone不出问题就好。

#### cmake链接cryptoTools库

头文件加入`#include<cryptoTools/Common/Timer.h>`

编写cmake文件：

``` cmake
cmake_minimum_required(VERSION 3.15)

# set the project name
project(UPSI_test)

SET(CMAKE_PREFIX_PATH "cryptoTools/")
find_package(cryptoTools REQUIRED)

# add the executable
add_executable(UPSI_test main.cpp)

target_link_libraries(UPSI_test cryptoTools)
```

这里要注意几点：

- find_package、target_link_libraries是根据cryptoTools官方说明来的

- 直接find_package会找不到，所以需要把库路径添加到CMAKE_PREFIX_PATH里面

- 如果遇到报错：

  `Cannot specify link libraries for target "XXX" which is not built by this project.`

  那么需要检查两点：

  - add_executable生成的文件需要是你的工程名
  - add_executable要在target_link_libraries前面

- 有时候cmake明明写对了，但是还是会报错。这可能是之前执行过错误的CMake导致的，删掉`CMakeCache.txt`、`CMakeFiles`再试试

#### 使用Timer计时器

这个就很简单了：

``` cpp
 osuCrypto::Timer timer;
 timer.setTimePoint("begin");
 testbench();
 timer.setTimePoint("End");
 std::cout << timer;
```

打印的结果美观又准确，不得不说osuCrypto做的这些密码学库（CryptoTools、libOTe、libPSI...）都非常强大，就是C++大工程实在是有点折磨人...

``` shell
Label    Time (ms)  diff (ms)
__________________________________
End          90.8     90.759  **********
```

