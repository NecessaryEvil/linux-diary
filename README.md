# linux-diary
日常使用Linux的记录，以及网上搜索到的解决Linux疑难杂症的汇总。

## 2020年10月20日
之前下载了一个.AppImage格式的软件，打开一次后就变成开机启动项了。systemctl list-unit-files和chkconfig --list都未发现与之相关的条目，困扰了很久，完全不知道是如何开机启动的。  
今天看到了一篇博客，[linux-autostart](https://www.cnblogs.com/sztom/p/13233803.html)，很有帮助。将 ~/.config/autostart/ 下的Outline-Client.AppImage.desktop文件删除，就不再开机自启动了。