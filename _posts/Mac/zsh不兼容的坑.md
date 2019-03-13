---
date: 2016-05-31 11:38
status: public
title: 'zsh不兼容的坑-zsh:no matches found:'
categories: Mac
---

最近使用android的adb命令，`adb logcat *:E`使用该命令过来E级别的日志的时候，发现在zsh中提示无法找到这个命令`zsh: no matches found: *:E`。后来查看了一些资料才知道，这是由于zsh导致的。
### 具体原因
因为zsh缺省情况下始终自己解释这个firefox*，而不会传递给adb logcat来解释。
### 解决办法
在~/.zshrc中加入:
setopt no_nomatch, 然后进行source .zshrc命令

