---
layout: post
comments: true
title: '自定义我的输入法'
excerpt: ...
categories:
  - '填坑'
  - '部署'
date: '2024-01-29T16:39:04+08:00'
tocopen: true
---

这年头混迹赛博宇宙，零信任原则的重要性不言而喻。以个人输入法为例，鉴于一些厂商不太体面的行为预设[^samsung][^baidu][^sogou]，选择使用离线、开源的输入法已经成为一项紧迫的需求。

## 选型

但凡做过些调研的朋友们应该都会投入rime的怀抱。rime在不同平台上有着不同的发行版，各有各的发行版配置，但主体部分还是互通的。
在其[官网](https://rime.im/download/)就可以找到下载链接，涵盖了Windows, macOS, Linux, Android，甚至还有（不确定有没有坑的第三方？的）[iOS客户端](https://apps.apple.com/cn/app/%E4%BB%93%E8%BE%93%E5%85%A5%E6%B3%95/id6446617683)。

## 配置

出于对[折腾](../2023-wandering/#接近开源方案)的热情，我目前的主要使用平台是Arch Linux和Android，不过由于各发行版之间只有少量配置区分，这里我会按照如下三部分进行记录：

### 共通

#### 全拼

[Rime 定製指南](https://github.com/rime/home/wiki/CustomizationGuide)固然十分详尽，对于新人上手却不十分友好，我选择了[薄荷输入法](https://www.mintimate.cc/zh/)作为自己的初始化模板。这个初始化模板已经足够好用，且使用了[雾凇拼音](https://github.com/iDvel/rime-ice)的非常好用的词库，以至于在我只使用`ibus-rime`的那一段时间都并未对其本身做太多修改（尽管至今也依然如此），仅关闭了它自带的[模糊拼音](https://www.mintimate.cc/zh/guide/fuzzyPinyin.html#%E8%96%84%E8%8D%B7%E6%8B%BC%E9%9F%B3%E7%9A%84%E6%A8%A1%E7%B3%8A%E6%8B%BC%E9%9F%B3)、五笔与比划依赖。我的版本已公开在[github](https://github.com/Kuro96/my_rime_configs/blob/63d5885e2bbbd87e7eba2b9bb5c723d854a7d81c/rime_mint.schema.yaml)，同时附上我参考的[原始版本](https://github.com/Mintimate/oh-my-rime/blob/1d54a6e4a81a98ca6c21acbd4462a690bcd207ff/rime_mint.schema.yaml)。

#### 双拼

本着一步到位的原则，既然折腾了，干脆就顺便把听闻已久的双拼给安排了。听了朋友的推荐，我选用的是小鹤双拼的方案，顺便把薄荷拼音的一些特性整合了进来，大体修改同上，同样已公开在[github](https://github.com/Kuro96/my_rime_configs/blob/37c89e1e35ee042f4a6826678bd79821bd2f3043/flypy.schema.yaml)，并附[原始版本](https://github.com/rime/rime-double-pinyin/blob/6e2e2262200a98496fd85327c9d3863a56897780/double_pinyin_flypy.schema.yaml)。

### Arch Linux: `ibus-rime`

Linux上的输入法一般由`ibus`或是`fcitx`管理，参考[官方文档](https://github.com/rime/home/wiki/CustomizationGuide#小狼毫外觀設定)仅需修改少量配置即可：

```yaml
# default.custom.yaml
patch:
  "ascii_composer/good_old_caps_lock": true
  "ascii_composer/switch_key":
    Caps_Lock: commit_code
    Control_L: noop
    Control_R: noop
    Shift_L: commit_code
    Shift_R: inline_ascii
  "menu/page_size": 9
  schema_list:
    - {schema: flypy}
    - {schema: rime_mint}
  "switcher/hotkeys":
    - "Control+grave"
  "switcher/save_options":
    - full_shape
    - ascii_punct
    - simplification
    - zh_hans
    - emoji_suggestion
  "translator/enable_encoder": true
  "translator/enable_sentence": true
  "translator/enable_user_dict": true
  "translator/encode_commit_history": true
```

### Android: `trime`

安卓平台的rime发行版为[trime](https://github.com/osfans/trime)，看起来似乎是第三方基于[librime](https://github.com/rime/librime)开发的。当前该发行版似乎还在积极改进中，我在这就暂不吐槽我在修改配置过程中遇到的种种bug了，仅贴上[官方文档](https://github.com/osfans/trime/wiki/trime.yaml-%E8%A9%B3%E8%A7%A3)并提醒：**时刻记得备份自己的配置文件（们）**，以及**尽可能不要使用其GUI中的`键盘设置`**。~~不然会变得不幸~~

我是基于同文官方的[`trime.yaml`](https://github.com/osfans/trime/blob/bf04d082b22453ce3f6162b27b7a888f8384a5cd/app/src/main/assets/rime/trime.yaml)修改的配置，主要调整了少量键盘布局、增加了夜间配色、实现了少量自定义按键以及移除了大量（对我而言）没什么用的键盘、配色等，我的方案可以参看我的[gist](https://gist.github.com/Kuro96/d8502271c43c5ce194790b32b87ee047)。

#### 关于夜间配色

[官方文档](https://github.com/osfans/trime/wiki/trime.yaml-%E8%A9%B3%E8%A7%A3#%E9%85%8D%E8%89%B2%E6%96%B9%E6%A1%88)中指出如下：

> [3.2.6]`dark_scheme`：配色方案如有此参数，即视为明亮模式的配色。当系统切换为暗黑模式后，再次弹出键盘时，自动切换配色方案为`dark_scheme`指定的配色。

不过经过我的实测，这样配置确实可以自动切换到夜间模式，但是在自动切换回日间模式时，输入法的悬浮窗还会保留夜间模式的配色，这点在trime官方群讨论中得到了证实，希望日后可以得到修复（

#### 关于按键自定义

自定义按键理论上可以做很多有意思，不过另一方面，输入法做得过于喧宾夺主也并不值得推崇。
这里只举一个例子，权当抛砖引玉：

```yaml
Bili: {label: B站, command: run, option: "bilibili://search/%4$s"}
# ...
      - {click: 'r', long_click: 'Bili', swipe_up: 4}
```

这里我把长按`r`键设置为了使用bilibli app搜索当前文本框光标前的字符。其中，`bilibili://search/`的字段可以通过`adb shell`和`logcat | grep`结合app内手动操作获取到，理论上其他app也可以通过类似的方法获取，具体还请自行实践。

## DLC

### 同步方案

说点题外话，rime的发行版们其实有实现基于uuid的同步方案，不过这种方式其实就是生成了一个`sync`文件夹，具体的云端同步还需要另外结合网盘使用。加之我在配置trime时留下的不好的回忆，让我对这个方案不甚放心。

最后我选择了git作为同步方案：

- 使用`master`分支同步共通配置，并主动剔除发行版相关的配置文件，此分支开放在我的github repo供观众老爷们参考
- 各个发行版在`master`分支的基础上增加其对应配置文件并同步于其对应分支
- `master`独立更新，各发行版分支基于`master`来`rebase`并更新

这对我来说有几个好处：

1. 使用git更利于我进行版本管理（至少配置trime时不再会动不动删档重开）
2. 不需借助`syncthing/nextcloud/onedrive`等工具就能实现多端同步

但这种方案也有几个已知不足之处，观众老爷们如果有更好的方案欢迎评论区留言：

1. 由于使用了`rebase`，各发行版分支在`push/pull`时往往需要使用`-f`参数来强制推送/拉取，这不够优雅
2. `xxx.userdb`目录，即用户输入历史不方便同步，使用`git lfs`等方案也可能导致repo不必要的臃肿

[^samsung]: [三星手机窃取用户隐私：内置搜狗输入法成为“帮凶”](https://www.163.com/dy/article/IOTQL5920534B9EY.html)
[^baidu]: [百度输入法已经完全无下限了，远离百度，珍爱一生。](https://www.v2ex.com/t/1011440)
[^sogou]: [7年后国产输入法依然存在严重问题 上传用户输入内容且可被监听](https://www.bilibili.com/read/cv25654585/)
