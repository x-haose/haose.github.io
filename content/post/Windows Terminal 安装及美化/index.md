---
slug: 78546b8b
author: "昊色居士"
title: "Windows Terminal 安装及美化"
description: 
date: 2022-05-15T00:03:49+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "Windows Terminal", "终端美化", "工具"
]
categories: [
    "工具"
]
series:  [

]
---

# Windows Terminal 安装及美化

## 预览

![](https://files.mdnice.com/user/29990/72a9d4d4-bf2e-4e8e-bcfa-1db9f6a1117a.png)

## 安装

### Microsoft Store 安装

打开微软商店搜索 Windows Terminal 直接安装

### GitHub 安装

- 打开 GitHub 地址：[Windows Terminal](https://github.com/microsoft/terminal/releases "Windows Terminal")
- 下载 msixbundle 包
- 使用 powershell 执行命令 `add-appxpackage ./文件名.msixbundle`
- 如提示以下信息则是缺失 vc++运行时环境，按要求下载安装即可：
  ![](https://files.mdnice.com/user/29990/a07eca47-54ce-4203-b0c7-1e4689715de7.png)
  - 打开[VCLibs 环境下载链接](https://docs.microsoft.com/en-us/troubleshoot/developer/visualstudio/cpp/libraries/c-runtime-packages-desktop-bridge "VCLibs环境下载链接")
  - 使用 powershell 执行命令 `add-appxpackage ./文件名.msixbundle`

## 基本配置

### 修改安全策略

```PowerShell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
```

### 安装 oh-my-posh

使用管理员模式打开 Windows Terminal (PowerShell)，安装  `posh-git`  和  `oh-my-posh`  这两个模块。

```PowerShell
Install-Module posh-git -Scope CurrentUser
Install-Module oh-my-posh -Scope CurrentUser
Install-Module -AllowClobber Get-ChildItemColor
```

### 配置主题配置文件

通过以下命令进行配置，如果之前没有配置文件，就新建一个 PowerShell 配置文件。

```PowerShell
if (!(Test-Path -Path $PROFILE )) { New-Item -Type File -Path $PROFILE -Force }
```

选择使用 vscode 或其他编辑器打开（选择其一）

```PowerShell
code $PROFILE
notepad $PROFILE
```

在里面添加以下内容：

```PowerShell
Import-Module posh-git
Import-Module oh-my-posh
Set-PoshPrompt -Theme M365Princess
If (-Not (Test-Path Variable:PSise)) {
    Import-Module Get-ChildItemColor
    Set-Alias l Get-ChildItem -option AllScope
    Set-Alias ls Get-ChildItemColorFormatWide -option AllScope
}
```

关于  `oh-my-posh`  的主题，官方目前给出了 19 款，可以前往其  [文档](https://ohmyposh.dev/docs/themes "oh-my-posh主题")  查看并选择自己喜欢的主题，这里个人使用的是  `M365Princess`，感觉比较好看

## 主题和皮肤

### 下载终端字体

需要下载并安装专用的字体，否则有些符号会显示不出来

- 打开[字体下载网站](https://www.nerdfonts.com/font-downloads "字体下载网站"),或自行查找其他网站
- 下载喜欢的字体进行安装
- 在配置文件中进行设置：
  ```json
  "defaults": {
      "fontFace": "agave NF r"
  }
  ```

### 配置 Windows Terminal 皮肤

推荐两个网站查找喜欢的主题：

- [terminalsplash 主题网站](https://terminalsplash.com/ "terminalsplash主题网站")
- [windowsterminalthemes 主题网站](https://windowsterminalthemes.dev/ "windowsterminalthemes主题网站")
  找到喜欢的主题后把主题的配置文件置于`schemes`中:

```json
......
// Add custom color schemes to this array.
// To learn more about color schemes, visit https://aka.ms/terminal-color-schemes
	"schemes": [
    {
       主题内容
    }
  ]
  ......
```

添加完主题后，在`profiles`项下，将`"colorScheme": "你的主题名字"`  置于`defaults`中（应用于全局）

```json
"defaults": {
// Put settings here that you want to apply to all profiles.
            "colorScheme": "Night Owlish Light",
      },
```

## Pokemon-Terminal 美化

- 打开[Pokemon-Terminal](https://github.com/LazoCoder/Pokemon-Terminal "Pokemon-Terminal")地址
- 拉下代码并使用 python 安装

```PowerShell
git clone https://github.com/LazoCoder/Pokemon-Terminal
cd Pokemon-Terminal
python setup.py install
```

安装好后终端运行 `pokemon` 试试能不能切换，不能自行检查环境变量

## 基本使用

### 打开命令窗口

快捷键：Ctrl+Shift+P
![](https://files.mdnice.com/user/29990/c70c87ce-0a72-4265-aed5-0a04ebbfb730.png)

### 拆分窗口

- 向右拆分：Alt+Shift+加号（退格旁边的加号）
- 向下拆分：Alt+Shift+减号（退格旁边的减号）
- 调整拆分窗口的大小：Alt+Shift+上下左右
- 关闭拆分的窗口：Ctrl+Shift+W
