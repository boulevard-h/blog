---
layout: blog
title: 使用自动脚本领取LOL国际服观赛礼包
categories: 瞎折腾
date: 2023-06-30 10:08:50
---

众所周知，除了~~没~~马服以外，拳头的自营服会在观赛直播间送出**观赛引擎**，而观赛引擎开多了就会出现很多重复的头像/表情，可以分解为橙色精粹。

于是小黄鱼上就出现了很多代挂，低至5元/月，高到30/月。

由于小黄鱼的卖家要么不太靠谱，要么收费太贵，于是决定根据他们的截图顺藤摸瓜去找找有没有开源的代挂，果不其然找到了两个：

- [LeagueOfPoro/CapsuleFarmerEvolved](https://github.com/LeagueOfPoro/CapsuleFarmerEvolved)
- [Yudaotor/EsportsHelper](https://github.com/Yudaotor/EsportsHelper)

第一个是一个老外做的，但是在2023年3月的时候被Riot律师函了，所以现在已经停止维护了。

第二个是国人做的，截至目前（2023-7）还在维护，而且可用性也相对较好，所以用这个。

## 服务器选择

首先考虑的肯定是在宿舍的笔记本上配置，Github上可以直接从releases下载exe包，到config里面配置一下就可以跑，很方便。只不过是外服的观赛，所以需要先挂个梯子。

但是一天下来就发现了不对劲，直接跑了3G流量，VPN+校园网流量的双重收费实在是顶不住这么造。

所以考虑不如直接租一个海外的高带宽服务器。

于是在tx云上买了一个 2核4G 30m带宽每月1000G流量的服务器，地址在新加坡，耗费42元/月。先观察一下这个月的情况，再决定是否要修改配置。

## 配置环境

这个库是经典的selenium + chrome_driver的web脚本，所以需要先配置好环境，作者自己写的教程：

[如何在linux环境运行（run in linux） · Yudaotor/EsportsHelper Wiki (github.com)](https://github.com/Yudaotor/EsportsHelper/wiki/如何在linux环境运行（run-in-linux）)

大差不差，但是这个教程用的是pipenv虚拟环境，而这个环境在ubuntu上真的是巨容易报错，所以稍微修改和完善了流程，用conda。

### 安装chrome

服务器的Ubuntu是纯命令行，但这并不代表其不能安装浏览器，chrome支持headless模式，不在前端运行。

`wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb`下载最新的chrome包，此时运行`sudo dpkg -i google-chrome-stable_current_amd64.deb`会有依赖报错，下面是解决方法：

先`sudo apt -f -y install`安装依赖，然后再次`sudo dpkg -i google-chrome-stable_current_amd64.deb `即可安装完成。

完成以后输入`google-chrome --version`，应该是可以看到的。此时Chrome就安装好了。

### 配置python环境

万恶的pipenv全是bug，推荐使用conda，这里建议安装miniconda即可，没必要用那么臃肿的anaconda。安装方法自行百度，非常多。

然后创建一个新的python虚拟环境，需要指定python版本，3.6版本太低依赖装不上，目前用的是3.9：

``` shell
conda ceate -n esHelper python=3.9
```

安装好以后，activate进入环境，克隆Github库并进入，然后安装python依赖：

``` shell
pip install -r requirements.txt
```

## 配置config

config文件如下

``` yaml
delay: 600                # 每次检查的时间间隔，单位为秒(默认为600秒)(每次检测时间会在你设置的时延0.8-1.5倍之间随机波动) / Time interval for each check in seconds (600 seconds by default). Each check time will randomly range between 0.8 and 1.5 times the time delay you set
headless: True           # 设置为True时，程序会在后台运行，否则会打开浏览器窗口(默认为False) / # When set to True, the program will run in the background; otherwise it will open a browser window (set False by default)
username: "账号"     # 必填，账号 / Required field, username
password: "密码"          # 必填，密码 / Required field, passward
nickName: ""              # 绰号, 不填即为用户名. / NickName. default is username.
maxStream: 3              # 最大同时观看比赛数.默认为3 / Maximum number of matches to be watched simultaneously. Default is 3.
language: "zh_CN"         # 语言选择，zh_CN是简体中文，en_US是英文, zh_TW是繁体中文 / Language selection, zh_CN for Chinese (Simplified), en_US for English,zh_TW for Chinese (Traditional)
autoSleep: False          # 是否自动休眠,即没比赛时自动休眠，有比赛时自动唤醒(默认为False) / Whether to sleep automatically, that is, to sleep automatically when there is no match, and to wake up automatically when there is a match (default is False)
onlyWatchMatches: ["lcs","lla","lpl","lck","ljl-japan","lco","lec","cblol-brazil","pcs","tft_esports"]   # 只观看的赛区名称,小写. / # The name of the league to be watched only, lowercase.
platForm: "linux"
connectorDropsUrl: ""
```

主要需要修改如下几项：

1. headless：设置为True，也就是chrome不显示前端运行，一方面是服务器纯命令行没法展示前端，另一方面是减少CPU占用。关于这个模式会不会更容易被封号，在issue中提问作者说问题不大
2. username、password
3. platForm：默认是windows，改为linux
4. connectorDropsUrl：这个是掉落提醒的链接，钉钉、Discord等软件中都有机器人，将机器人的Webhook填入以后，只要掉落了，机器人就会提醒。

配置好以后`python main.py`运行即可

## 设置webhook

在discord和钉钉群聊中都可以添加机器人，然后复制其Webhook填入到上面的config就可以了。

要注意的是钉钉的机器人需要配置安全认证，这里配置按照IP地址认证，填入服务器公网IP即可

## tmux后台

服务器上运行这种后台长时间待机任务都要用tmux，使用apt安装以后，创建一个tmux回话：

``` shell
tmux new -s esHelper
```

到会话中激活conda环境，运行main.py，然后就可以挂起到后台运行了：首先按一下`ctrl + b`，然后按一下`d`即可。

如果想要重新进去看看情况，可以用`tmux attach -t esHelper`进入查看和操作。
