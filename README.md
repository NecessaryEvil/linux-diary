# linux-diary  

日常使用Linux的记录，以及网上搜索到的解决Linux疑难杂症的汇总。  
文中的命令未区分是否需要root权限，若权限不够，请自行sudo。

## 2020年10月20日  

之前下载了一个.AppImage格式的软件，打开一次后就变成开机启动项了。`systemctl list-unit-files`和`chkconfig --list`都未发现与之相关的条目，困扰了很久，完全不知道是如何开机启动的。  
今天看到了一篇博客，[linux-autostart](https://www.cnblogs.com/sztom/p/13233803.html)，很有帮助。将 ~/.config/autostart/ 下的Outline-Client.AppImage.desktop文件删除，就不再开机自启动了。  

## 2020年10月30日  

修改/etc/sysconfig/network-scripts/目录下的网络配置文件以后，如何用命令行使新的配置生效？网上搜索到的都是ifconfig命令和ip命令，但现在常用的发行版（我自己用的Fedora 33 和 CentOS 7.6.1810）都是用NetworkManager来管理网络的，上述两个命令有时候并不有效（似乎也可以用，但我还没找到简单的命令组合）。  
nmcli是NetworkManager的命令行工具，在查了它的help以后，我尝试：  
`nmcli c reload`  
`nmcli c down enp1s0`  
`nmcli c up enp1s0`  
成功地使修改后的网络配置生效。  

## 2020年11月16日  

为了防止gnome崩溃时连系统都进不去的情况发生，我将开机运行的级别调到了3，即开机默认进入命令行。这个设置很简单，只需要  
`systemctl set-default multi-user.target`  
但问题是进入命令行界面的分辨率很低，字体很大，一屏显示不了几行，如果真的需要命令行救急，会影响效率（主要还是看着不爽）。于是尝试了一下修改tty的分辨率。可以修改  
`/boot/grub2/grub.cfg`  
文件，在里面找到  
`set kernelopts=...`  
这一行，在后面的参数中加一项  
`vga=0x####`  
这个0x####和分辨率对应的关系可以用  
`hwinfo --framebuffer`  
查看。如我的机器运行这个命令的结果为  

~~~null
02: None 00.0: 11001 VESA Framebuffer  
  [Created at bios.459]  
  Unique ID: rdCR.pzSJSS7NeC7  
  Hardware Class: framebuffer  
  Model: "NVIDIA GP106 Board"  
  Vendor: "NVIDIA Corporation"  
  Device: "GP106 Board"  
  SubVendor: "NVIDIA"  
  SubDevice:  
  Revision: "Chip Rev"  
  Memory Size: 16 MB  
  Memory Range: 0x01000000-0x01ffffff (rw)  
  Mode 0x0301: 640x480 (+640), 8 bits  
  Mode 0x0303: 800x600 (+800), 8 bits  
  Mode 0x0305: 1024x768 (+1024), 8 bits  
  Mode 0x0307: 1280x1024 (+1280), 8 bits  
  Mode 0x0311: 640x480 (+1280), 16 bits  
  Mode 0x0312: 640x480 (+2560), 24 bits  
  Mode 0x0314: 800x600 (+1600), 16 bits  
  Mode 0x0315: 800x600 (+3200), 24 bits  
  Mode 0x0317: 1024x768 (+2048), 16 bits  
  Mode 0x0318: 1024x768 (+4096), 24 bits  
  Mode 0x031a: 1280x1024 (+2560), 16 bits  
  Mode 0x031b: 1280x1024 (+5120), 24 bits  
  Mode 0x0345: 1600x1200 (+1600), 8 bits  
  Mode 0x0346: 1600x1200 (+3200), 16 bits  
  Mode 0x034a: 1600x1200 (+6400), 24 bits  
  Mode 0x034b: 1920x1080 (+1920), 8 bits  
  Mode 0x034c: 1920x1080 (+3840), 16 bits  
  Mode 0x034d: 1920x1080 (+7680), 24 bits  
  Config Status: cfg=new, avail=yes, need=no, active=unknown  
~~~

于是我在grub.cfg中添加vga=0x034d。  
但我们并不推荐直接修改grub.cfg文件，推荐的做法是修改  
`/etc/default/grub`  
文件，并重新生成grub.cfg。在GRUB_CMDLINE_LINUX_DEFAULT配置项中添加vga=0x034d，保存文件后运行  
`grub2-mkconfig -o /boot/grub2/grub.cfg`  
即可生成新的grub.cfg文件。  
如果只是想在某次启动时修改tty的分辨率（毕竟不会经常使用纯命令行界面），那么不需要修改任何配置文件，只需在开机grub页面按‘e’键，在`linux ($root)/vmlinuz-...`这一行末尾添加vga=0x034d，再按Ctrl+x启动即可。  

## 2020年11月24日

装了Windows和Linux双系统后会发现，Windows和Linux的系统时间总是相差8小时，Linux领先Windows8小时。这是因为Linux把硬件时间当作是协调世界时，东八区的本地时间就在此基础上加了8小时，而Windows默认将硬件时间当作是本地时间。用命令  
`timedatectl`  
即可查看系统时间配置信息，如下

~~~null
               Local time: 二 2020-11-24 10:35:09 CST
           Universal time: 二 2020-11-24 02:35:09 UTC
                 RTC time: 二 2020-11-24 02:35:09
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
~~~

RTC in local TZ: no 即表示未将硬件时间作为本地时间。  
为了将Windows系统和Linux系统的时间统一，需将Windows也设置成把硬件时间作为协调世界时。目前只能通过修改注册表实现，命令如下  
`REG ADD HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1`  
至于为什么Wimdows迟迟不改为把硬件时间作为UTC，可以参考 [Why does Windows keep your BIOS clock on local time?](https://devblogs.microsoft.com/oldnewthing/20040902-00/?p=37983) 这篇文章，简而言之就是——历史遗留问题。  

## 2020年12月24日

Linux命令行界面下的串口数据调试一直都不是方便的事情，不像Windows下那样有很多图形界面的串口调试工具可以用。之前也听说过minicom之类的工具，但没有深入研究。而且我目前更需要在Linux命令行下面监听串口的数据，而不是手动调试。数据调试在程序开发的过程中必然是做过的，只是在生产环境中还是有可能出现意料之外的情况，又无法在仅有命令行界面的生产设备中进行Debug，因此需要进行串口数据的监听。  
socat工具功能强大，用来做串口转发和监听绰绰有余。用socat可以对串口、网络、文件、标准输入输出流等我们能想象到的绝大部分的IO形式进行相互转换，并监听。在我的应用场景中，我将串口设备的数据转发到一个虚拟串口，将调试目标应用软件连接的串口从真实串口换到虚拟串口，这样就实现了应用程序的串口数据的监听。  
命令如下  
`socat -d -d -x /dev/ttyS0,b9600,cs8,cstopb=0,icanon=0,min=128,time=1,echo=0,ignoreeof PTY,link=/tmp/pty0,rawer,echo=0,ignoreeof`  
将软件连接的串口从/dev/ttyS0切换到/tmp/pty0即可。  
监听的部分数据如下

~~~null
< 2020/12/24 18:15:40.435492  length=8 from=15464 to=15471
 01 03 00 00 00 0a c5 cd
> 2020/12/24 18:15:40.607036  length=25 from=48400 to=48424
 01 03 14 00 01 00 02 00 03 00 04 00 05 00 06 00 07 00 08 00 09 00 0a 8f 16
< 2020/12/24 18:15:41.611513  length=8 from=15472 to=15479
 01 03 00 00 00 0a c5 cd
> 2020/12/24 18:15:41.783927  length=25 from=48425 to=48449
 01 03 14 00 01 00 02 00 03 00 04 00 05 00 06 00 07 00 08 00 09 00 0a 8f 16
~~~

## 2021年12月22日

配置Linux的串口终端（在CentOS7上实际操作），编辑/etc/default/grub，添加或修改下列几项为：

~~~null
GRUB_SERIAL_COMMAND="serial -speed=115200 -unit=0 -word=8 -parity=no -stop=1"
GRUB_TERMINAL_OUTPUT="serial console"
GRUB_TERMINAL_INPUT="serial console"
~~~

在 GRUB_CMDLINE_LINUX 这一项的值的最后面添加 console=ttyS0,115200。  
运行  
`grub2-mkconfig -o $(find /boot -name grub.cfg)`  
重新生成grub.cfg。  
查看/etc/securetty文件，里面是否有一行 ttyS0，如果没有则加上。  
把开机运行级别调整到3，不然串口终端交互有问题，几乎无法使用。  
最后重启即可。  
