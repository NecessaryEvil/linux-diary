# linux-diary
日常使用Linux的记录，以及网上搜索到的解决Linux疑难杂症的汇总。

## 2020年10月20日
之前下载了一个.AppImage格式的软件，打开一次后就变成开机启动项了。systemctl list-unit-files和chkconfig --list都未发现与之相关的条目，困扰了很久，完全不知道是如何开机启动的。  
今天看到了一篇博客，[linux-autostart](https://www.cnblogs.com/sztom/p/13233803.html)，很有帮助。将 ~/.config/autostart/ 下的Outline-Client.AppImage.desktop文件删除，就不再开机自启动了。

## 2020年10月30日
修改/etc/sysconfig/network-scripts/目录下的网络配置文件以后，如何用命令行使新的配置生效？网上搜索到的都是ifconfig命令和ip命令，但现在常用的发行版（我自己用的Fedora 33 和 CentOS 7.6.1810）都是用NetworkManager来管理网络的，上述两个命令有时候并不有效（似乎也可以用，但我还没找到简单的命令组合）。
nmcli是NetworkManager的命令行工具，在查了它的help以后，我尝试：
nmcli c reload
nmcli c down enp1s0
nmcli c up enp1s0
成功地使修改后的网络配置生效