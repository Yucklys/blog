---
title: Polybar Nord主题配置
date: 2019-08-05
categories:
- Customization
- Linux
tags:
- polybar
---

最近迷上了 [Nord](https://www.nordtheme.com/) 配色，把我的终端、vim、以及 i3wm 的配色全部换成了Nord。不过 Polybar 我并没有找到合适的 Nord 配色，所以花了点时间，重新写了一遍配置，使用了 Nord 配色并且基本支持所有的官方 module 和一些社区以及本人写的 module。

- ![i3-nord.png](https://tupp.xyz/2019/08/05/15650043555d481243502de.png)

<!--more-->

# 概况

支持的 module：

1. [alsa](#alsa)
2. [backlight](#backlight)
3. [date](#date)
4. [notify (基于dunst)](#notify)
5. [network-detail](#network-detail)
6. [network-short](#network-short)
7. [battery](#battery)
8. [keyboard](#keyboard)
9. [systray](#systray)
10. [mpd](#mpd)
11. [i3](#i3)
12. [workspace-name](#workspace-name)
13. [weather](#weather)
14. [info-hackspeed](#info-hackspeed)
15. [github](#github)
16. [daily-poem](#daily-poem)
17. [temperature](temperature)
18. [memory](#memory)
19. [cpu](#cpu)
20. [show-desktop](#show-desktop)

必需依赖：

1. [Font Awesome](https://www.archlinux.org/packages/community/any/ttf-font-awesome/) （必选）

可选依赖：

1. [Weather Icon](https://github.com/erikflowers/weather-icons)（weather 组件需要）
2. Dunst （notify 组件显示历史通知需要）
3. redshift （backlight 开关 redshift 功能）

配置放在了 [GitHub](https://github.com/Yucklys/polybar-nord-theme) 上，clone 到自己的 Polybar 配置文件夹下替换即可，建议提前做好备份。`

前排提示，配合[官方wiki](https://github.com/polybar/polybar/wiki)食用口味更佳。

# Modules

## alsa

显示当前音量，根据音量共有三种不同颜色图标。默认声卡选择为`default`，可以在`nord-config`里调整声卡选项。

| 绑定     | 事件             |
| -------- | ---------------- |
| 鼠标左击 | 静音/恢复        |
| 滚轮上   | 音量 +，每次 +5% |
| 滚轮下   | 音量 -，每次 -5% |

## backlight

调整屏幕亮度。可选左击开关 redshift，需要提前配置好 redshift 或者修改 scripts/

| 绑定   | 事件             |
| ------ | ---------------- |
| 滚轮上 | 亮度 +，每次 +5% |
| 滚轮下 | 亮度 -，每次 -5% |

## date

显示时间。

| 绑定     | 事件              |
| -------- | ----------------- |
| 鼠标左击 | 切换日期/时间显示 |

## notify

通知图标。

| 绑定     | 事件                                                         |
| -------- | ------------------------------------------------------------ |
| 鼠标左击 | 模拟 ctrl+grave，即默认的 dunst 历史快捷键，会被其他快捷键设定覆盖 |

## network-detail

显示上传及下载速度，单位为KB/s，需要在`nord-config`中`interface`字段设置自己的 interface。

## network-short

显示网络连接图标，图标颜色代表网络连接延迟程度，颜色越偏红说明延迟越高。

## battery

显示电池信息，需要在`nord-config`中设置`battery-full-at`、`battery-bat`以及`battery-adp`字段。

## keyboard

显示键盘大写锁定。因为使用的键盘没有 NumLock，所以暂时不支持，如果需要可以自行添加。

## systray

显示系统托盘。我的系统是 Manjaro i3 社区版，i3 版本 4.16.1，polybar 版本 3.4.0，按照如此配置没有产生显示错误。如果遇到问题可以到官方 issue 中查找，有图标问题的人挺多的。

## mpd

显示 mpd 信息，当 mpd 进程不存在时隐藏。总共有三种显示方式，分别为**播放中**、**暂停**以及**停止**。

- 播放状态下显示全部内容，包括*暂停*、*上一首*、*歌手 - 歌名*、*下一首*、*停止*、*进度条*、*列表循环*以及*列表随机*。
- 暂停状态下显示部分内容，包括*播放*、*上一首*、*歌手 - 歌名*、*下一首*、*停止*。
- 停止状态下仅显示*播放*。

需要在`nord-config`中设置`mpd-host`及`mpd-port`。

## i3

显示 i3 workspace 图标，图标可在`nord-top`中定义。

| 绑定     | 事件                     |
| -------- | ------------------------ |
| 鼠标左击 | i3-msg workspace <index> |
| 滚轮上   | i3-msg workspace prev    |
| 滚轮下   | i3-msg workspace next    |

## workspace-name

显示当前 workspace 名字，在 i3 的配置文件处定义，格式为 `index:name`，例如`1:Home`。

## weather

显示 Openweather 天气信息，使用的脚本来自于[polybar/polybar-scripts/openweather-fullfeatured](https://github.com/polybar/polybar-scripts/tree/master/polybar-scripts/openweathermap-fullfeatured)，需要个人API及城市代码，在脚本中设置，具体情况可查看项目 Readme。

## info-hackspeed

显示每分钟输入速度，单位默认为 cpm (char per minute)，可选 wpm (word per minute)，在脚本中设置。来自于[polybar/polybar-scripts/info-hackspeed](https://github.com/polybar/polybar-scripts/tree/master/polybar-scripts/info-hackspeed)。

## daily-poem

显示今日诗词，使用了 V友@lhx2008 分享的[今日诗词 API](https://www.jinrishici.com/)，支持显示诗词推荐、来源以及匹配标签。使用需要获取 token，可到 https://v2.jinrishici.com/token 获取 token 后替换我在`nord-down`中的 token。

| 绑定     | 事件         |
| -------- | ------------ |
| 鼠标左击 | 显示诗词内容 |
| 滚轮上   | 显示诗词来源 |
| 滚轮下   | 显示匹配标签 |
| 鼠标右击 | 更新诗词     |

## temperature

显示温度，可在`nord-config`中设置`thermal-zone`，默认为0。当温度低于`base-temperature`时切换为最低温度显示，当温度高于`warn-temperature`时切换为警戒温度显示，温度图标颜色会根据温度在`base-temperature`到`warn-temperature`之间变化，根据自身情况设置合理的值即可。

## memory

显示内存使用情况，格式为 used/total。

## cpu

显示 cpu 使用情况。

## show-desktop

显示桌面。默认是切换到`25:Desktop`，可在`nord-down`中调整。如果 i3wm 启动了 workspace back_and_forth 的话，两次左击可返回原来桌面。

| 绑定     | 事件                        |
| -------- | --------------------------- |
| 鼠标左击 | i3-msg workspace 25:Desktop |

# 结语

除了 Polybar，我还写了 rofi，i3 以及 dunst 的配置，有时间再整理整理发出来吧。干这活的确挺繁琐的，Polybar 的配置难度算是比较低的了，官方的 wiki 简单易懂，不需要什么学什么额外的东西。其实还有一些细节处可以调整一下，比如歌手名和歌名的样式分开，不过这需要声明一个环境变量，对于通用的配置来说过于复杂了，所以个人配置我自己再琢磨琢磨，给大家一份开箱即用的配置就可以了。