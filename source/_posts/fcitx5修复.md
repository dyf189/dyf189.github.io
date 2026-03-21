---
layout: post
title: fcitx5输入法修复记录
date: 2026-02-25 17:51:37
tags:
  - linux
  - fcitx5
  - ubuntu
keywords: linux,fcitx5,ibus,输入法
author: dyf189
toc: true
---

## 背景

将电脑上的ubuntu22升到ubuntu24之后，fcitx5输入法突然不能使用了，开机后无论是键盘还是拼音输入法都不是激活状态

同时炸了的还有SDDM,之后修复后会另写一篇文章  
  
> 补：重新安装SDDM后又回来了，~~所以文章不用写了~~

## 折腾过程  

### 更换输入法框架

第一次怀疑是fcitx5的某些离谱bug导致的，在尝试修改配置，安装搜狗输入法fcitx5版无效之后我就尝试更换输入法框架

但是无论是安装ibus还是旧版的fcitx4，都没有效果，该不能用还是不能用

ibus添加sunpinyin并切换后没有效果，输入法框出不来

fcitx4我用的是讯飞输入法进行尝试，但是尽管讯飞输入法的桌面浮窗出来了，打字，输入法框还是出不来

### 重新安装桌面

~~（我敢说这是我做过最离谱的决定）~~

在询问AI还有他人后我决定把kde plasma桌面重新进行安装，然而并没有什么卵用(ーー;)

还是老样子，无法使用输入法，最终我决定换回fcitx5尝试

## 解决过程

### 了解错误原因

我从和deepseek聊的好几段对话中最终找到了通过运行   `fcitx5-diagnose` 来找到错误原因

错误输出如下

```bash
# 前端设置：
此脚本检查的环境变量仅能显示当前命令行的环境。仍有可能您的环境并没有应用于整个桌面。您可以通过使用命令对某个无法正常工作的进程使用命令 `xargs -0 -L1 /proc/$PID/environ` 检查此进程的实际的环境变量。

## Xim:
1.  `${XMODIFIERS}`:

    **环境变量 XMODIFIERS 的值被设为了“fcitx5”而不是“@im=fcitx”。请检查您是否在某个初始化文件中错误的设置了它的值。**

    **请使用您发行版提供的工具将环境变量 XMODIFIERS 设为 "@im=fcitx" 或者将 `export XMODIFIERS=@im=fcitx` 添加到您的 `~/.xprofile` 中。参见 [输入法相关的环境变量：XMODIFIERS](http://fcitx-im.org/wiki/Input_method_related_environment_variables/zh-cn#XMODIFIERS)。**
    **无法解析 XMODIFIERS: fcitx5.**

    从环境变量中获取的 Xim 服务名称为 fcitx.

2.  根窗口上的 XIM_SERVERS：

    Xim 服务的名称与环境变量中设置的相同。

## Qt:
1.  qt4 - `${QT4_IM_MODULE}`:

    **环境变量 QT_IM_MODULE 的值被设为了“fcitx5”而不是“fcitx”。请检查您是否在某个初始化文件中错误的设置了它的值。**
    **您可能会在 qt4 程序中使用 fcitx 时遇到问题.**

    **请使用您发行版提供的工具将环境变量 QT_IM_MODULE 设为 "fcitx" 或者将 `export QT_IM_MODULE=fcitx` 添加到您的 `~/.xprofile` 中。参见 [输入法相关的环境变量：QT_IM_MODULE](http://fcitx-im.org/wiki/Input_method_related_environment_variables/zh-cn#QT_IM_MODULE)。**

    **`fcitx5-qt4-immodule-probing` 未找到.**

2.  qt5 - `${QT_IM_MODULE}`:

    **环境变量 QT_IM_MODULE 的值被设为了“fcitx5”而不是“fcitx”。请检查您是否在某个初始化文件中错误的设置了它的值。**
    **您可能会在 qt5 程序中使用 fcitx 时遇到问题.**

    **请使用您发行版提供的工具将环境变量 QT_IM_MODULE 设为 "fcitx" 或者将 `export QT_IM_MODULE=fcitx` 添加到您的 `~/.xprofile` 中。参见 [输入法相关的环境变量：QT_IM_MODULE](http://fcitx-im.org/wiki/Input_method_related_environment_variables/zh-cn#QT_IM_MODULE)。**

    使用 fcitx5-qt5-immodule-probing 来检查在当前环境下将被实际使用的输入法模块：

        QT_QPA_PLATFORM=xcb
        QT_IM_MODULE=fcitx5
        IM_MODULE_CLASSNAME=fcitx::QFcitxPlatformInputContext

3.  qt6 - `${QT_IM_MODULE}`:

    **环境变量 QT_IM_MODULE 的值被设为了“fcitx5”而不是“fcitx”。请检查您是否在某个初始化文件中错误的设置了它的值。**
    **您可能会在 qt6 程序中使用 fcitx 时遇到问题.**

    **请使用您发行版提供的工具将环境变量 QT_IM_MODULE 设为 "fcitx" 或者将 `export QT_IM_MODULE=fcitx` 添加到您的 `~/.xprofile` 中。参见 [输入法相关的环境变量：QT_IM_MODULE](http://fcitx-im.org/wiki/Input_method_related_environment_variables/zh-cn#QT_IM_MODULE)。**

    使用 fcitx5-qt6-immodule-probing 来检查在当前环境下将被实际使用的输入法模块：

        QT_QPA_PLATFORM=xcb
        QT_IM_MODULE=fcitx5
        IM_MODULE_CLASSNAME=fcitx::QFcitxPlatformInputContext

4.  Qt 输入法模块文件：

    找到了 fcitx5 的 qt5 输入法模块：`/lib/x86_64-linux-gnu/qt5/plugins/platforminputcontexts/libfcitx5platforminputcontextplugin.so`。
    找到了 fcitx5 的 qt6 输入法模块：`/lib/x86_64-linux-gnu/qt6/plugins/platforminputcontexts/libfcitx5platforminputcontextplugin.so`。
    找到了 fcitx5 qt6 模块：`/lib/x86_64-linux-gnu/fcitx5/qt6/libfcitx-quickphrase-editor5.so`。

    下列错误也许并不准确，因为对路径所对应的 Qt 版本的猜测取决于发行版如何打包 Qt。如果您不使用任何对应版本的 Qt 程序，或者在 Wayland 下使用 Qt 的 text-input 支持，下列错误也不是严重问题。
    **无法找到 Qt4 的 fcitx5 输入法模块。**

## Gtk:
1.  gtk - `${GTK_IM_MODULE}`:

    **环境变量 GTK_IM_MODULE 的值被设为了“fcitx5”而不是“fcitx”。请检查您是否在某个初始化文件中错误的设置了它的值。**
    **您可能会在 gtk 程序中使用 fcitx 时遇到问题.**

    **请使用您发行版提供的工具将环境变量 GTK_IM_MODULE 设为 "fcitx" 或者将 `export GTK_IM_MODULE=fcitx` 添加到您的 `~/.xprofile` 中。参见 [输入法相关的环境变量：GTK_IM_MODULE](http://fcitx-im.org/wiki/Input_method_related_environment_variables/zh-cn#GTK_IM_MODULE)。**

    **`fcitx5-gtk2-immodule-probing` 未找到.**

    使用 fcitx5-gtk3-immodule-probing 来检查在当前环境下将被实际使用的输入法模块：

        GTK_IM_MODULE=fcitx5

    使用 fcitx5-gtk4-immodule-probing 来检查在当前环境下将被实际使用的输入法模块：

        GTK_IM_MODULE=fcitx5

2.  `gtk-query-immodules`:

    1.  gtk 2:

        **无法找到 gtk 2 的 `gtk-query-immodules`。**

        **无法找到 gtk 2 的 fcitx5 输入法模块。**

    2.  gtk 3:

        **无法找到 gtk 3 的 `gtk-query-immodules`。**

        **无法找到 gtk 3 的 fcitx5 输入法模块。**

3.  Gtk 输入法模块缓存：

    1.  gtk 2:

        在 `/lib/x86_64-linux-gnu/gtk-2.0/2.10.0/immodules.cache` 找到了 gtk `2.24.33` 的输入法模块缓存。
        版本行：

            # Created by /usr/lib/x86_64-linux-gnu/libgtk2.0-0t64/gtk-query-immodules-2.0 from gtk+-2.24.33

        **无法输入法模块缓存 `/lib/x86_64-linux-gnu/gtk-2.0/2.10.0/immodules.cache` 中找到 fcitx5**

        **无法在缓存中找到 gtk 2 的 fcitx5 输入法模块。**

    2.  gtk 3:

        在 `/lib/x86_64-linux-gnu/gtk-3.0/3.0.0/immodules.cache` 找到了 gtk `3.24.41` 的输入法模块缓存。
        版本行：

            # Created by /usr/lib/x86_64-linux-gnu/libgtk-3-0t64/gtk-query-immodules-3.0 from gtk+-3.24.41

        已找到 gtk `3.24.41` 的 fcitx5 输入法模块。

            "/usr/lib/x86_64-linux-gnu/gtk-3.0/3.0.0/immodules/im-fcitx5.so" 
            "fcitx" "Fcitx5 (Flexible Input Method Framework5)" "fcitx5" "/usr/locale" "ja:ko:zh:*" 
            "fcitx5" "Fcitx5 (Flexible Input Method Framework5)" "fcitx5" "/usr/locale" "ja:ko:zh:*"
```

从错误输出中可以看到fcitx5无法找到一堆模块的缓存，同时报错中也给出了解决方案  

```
环境变量 QT_IM_MODULE 的值被设为了“fcitx5”而不是“fcitx”。请检查您是否在某个初始化文件中错误的设置了它的值。
    您可能会在 qt6 程序中使用 fcitx 时遇到问题.

    请使用您发行版提供的工具将环境变量 QT_IM_MODULE 设为 "fcitx" 或者将 `export QT_IM_MODULE=fcitx` 添加到您的 `~/.xprofile` 中。
```

在之前询问ai过程，ai让我把环境变量设置为fcitx5,很明显正常应该设置为fcitx

### 修复

设置用户变量

```bash
nano ~/.xprofile
```

```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```

上面的操作理论来讲是行得通的，但是结果并没有成功，于是便修改系统变量

查看bash内容

```bash
cat /etc/environment
```

发现为空，便添加以下几行

```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```

最后成功修复了