---
title: cmder下使用virtualenv和condaenv
tags:
  - python
  - cmder
  - virtualenv/condaenv
categories:
  - python
date: 2020-02-23 16:03:34
---
>&#160;&#160;&#160;&#160; 今天用到了一个很强大的工具——[cmder](https://cmder.net/)
&#160;&#160;&#160;&#160; 简洁美观的界面，便捷的操作，我现在已经看不下去windows的原生cmd了。
<!--more-->

## CMDer的使用
&#160;&#160;&#160;&#160; 使用了cmder之后，我遇到一个问题：如何连接我的anaconda虚拟环境，并将它设为默认？
&#160;&#160;&#160;&#160; 摸索一番之后，发现如下解决方法：
1. 进入cmder的settings-startup-Tasks
2. “+”号新建一个task，我取名叫anaconda，emmm.....格式具体原因没有深究。
3. 修改task参数

![](https://s2.ax1x.com/2020/02/23/31uJt1.png)

&#160;&#160;&#160;&#160; 箭头1处是我启动anaconda的base虚拟环境的脚本文件(activate.bat)地址，箭头2处是新窗口打开时的默认路径，再勾选下default task for new console（箭头3），然后save settings。
  
&#160;&#160;&#160;&#160; 搞定！下次再ctrl + t 新建tab的时候，就会默认使用这个虚拟环境的配置啦。

----
## virtualenv和condaenv的使用

&#160;&#160;&#160;&#160; (当然前提是你已经安装了virtualenv、virtualenvwrapper和anaconda)

|virtualenv操作|condaenv操作|解释|
|--|--|--|
|workon|conda env list|查看创建的所有虚拟环境|
|mkvirtualenv --python=绝对路径\python.exe 虚拟环境名|conda create -n 虚拟环境名 python=PY版本|根据python版本创建虚拟环境|
|workon 虚拟环境名|activate 虚拟环境名|进入虚拟环境|
|deactivate|deactivate|退出虚拟环境|
|rmvirtualenv 虚拟环境名|conda remove -n 虚拟环境名 --all|删除虚拟环境|

