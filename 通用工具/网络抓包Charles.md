# 网络抓包

## Charles

Charles 通过将自己设置成系统的网络访问代理服务器，使得所有的网络访问请求都通过它来完成，从而实现了网络封包的截取和分析。

- [charles 下载地址](https://www.charlesproxy.com)
- [charles 教程](https://blog.csdn.net/forebe/article/details/98945139)
- [IOS 设备教程](https://blog.csdn.net/qq_33521184/article/details/122322130)
- [Charles 抓包安卓模拟器（MuMu）](https://www.jianshu.com/p/1d0360e50a01)

### 快速上手

1.[Charles] 设置系统代理的权限来完成封包截取

选择菜单中的“Proxy” –>“Mac OS X Proxy”或“Windows Proxy”

2.[Charles] 打开代理功能

选择菜单中的“Proxy”–>“Proxy Settings”后编辑 Charles 端口号为 8888，勾上 “Enable transparent HTTP proxying”

3.[iphone] 获取 Charles 的代理功能

在 iphone 当前连接的 wifi 里编辑内容“HTTP 代理“为手动代理，填上 Charles 运行所在的电脑的 IP，以及端口号 8888。（Charles 运行所在的电脑的 IP：Charles 的顶部菜单的 “Help”–>“Local IP Address”）

苹果手机还需要在的设置-通用-证书-允许 charles\*\*\* 证书

设置代理后，需要在电脑上配置并打开 Charles 才能上网。

4.[Charles] 安装证书

选择菜单中的“Help” –> “SSL Proxying” –> “Install Charles Root Certificate”

需要注意的是，即使是安装完证书之后，Charles 默认也并不截取 Https 网络通讯的信息，如果你想对截取某个网站上的所有 Https 网络请求，有以下几种方法：

a. Charles 在该请求上右击，选择 SSL proxy

b. 直接设置所有网站：“Proxy” –> “SSL Proxying Settings”勾选 enable ssl Proxying，添加 include 的端口号`*:443`

5.[iphone] 安装证书

点击 Charles 的顶部菜单，选择 “Help” –> “SSL Proxying” –> “Install Charles Root Certificate on a Mobile Device or Remote Browser”，然后就可以看到 Charles 弹出的简单的安装教程。

如果设置完 HTTPS 代理 和证书之后，则所有的请求数据都将以明文显示，网络包不会显示错误 X

## 真机调试

真机测试：fiddler
