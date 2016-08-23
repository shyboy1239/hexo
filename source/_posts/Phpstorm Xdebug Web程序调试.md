---
title: Phpstorm Xdebug Web程序调试
date: 2016-08-22 14:12:25
tags:
---
平时调试php程序的时候，可以通过在代码中添加var_dump等函数来实现简单的断点调试。
下面介绍另一种方法，通过Phpstorm和Xdebug来进行调试。<!-- more -->

#### 下载Xdebug
这个是官网[下载地址](https://xdebug.org/download.php)，下载你需要的版本。
如果不清楚的话可以使用这个[工具](https://xdebug.org/wizard.php)，只要粘贴提交你phpinfo()信息，就会返回适合你的版本以及简单的安装说明。
#### 安装Xdebug
移动下载好的xdebug扩展文件至对应目录并编辑php.ini文件，添加：
```
zend_extension="你的xdebug扩展文件路径"
```
具体请参考[这里](https://xdebug.org/docs/install)
#### 启用客户端调试器
在php.ini中[Xdebug]下添加一行如：
```
[xdebug]
xdebug.remote_enable = 1
```
具体请参考[这里](https://xdebug.org/docs/remote)
#### 激活调试器
这里选择安装浏览器插件的方法，适用于通过web方式运行的php脚本。
我的浏览器是chrome，对应的插件是Xdebug Helper。
安装好后在选项里设置IDE key选择PhpStorm。
#### 调试
在phpstorm对应的文件中设置好断点。
开启监听，在菜单 -> run -> Start Listening for PHP Debug Connections。
接着在浏览器打开对应的页面，注意右上角Xdebug helper插件小图标的状态是否为开启，如果顺利的话，phpstorm下方就会弹出调试信息的面板了！（如果是第一次配置，会出现一个来自xdebug的连接配置提示，直接点击接受即可）
#### 结语
本文介绍的方法主要通过结合浏览器插件来实现，比较方便简单，其他更多相关内容还请参考Xdebug和PHPstorm的官方文档。

