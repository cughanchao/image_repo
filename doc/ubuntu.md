# Ubuntu 禁止内核更新



Ubuntu 使用中会遇到kernel自动更新后，NVIDIA显卡驱动不可用的问题，需要重新卸载再安装显卡驱动。比较麻烦。

一种解决方法是禁止内核更新，锁定内核版本，从而规避这个问题。

1. 查看使用的内核版本

	```bash
	$ uname -a
	Linux dell 4.15.0-142-generic #146~16.04.1-Ubuntu SMP Tue Apr 13 09:27:15 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
	```

2. 查看所有安装的内核

	```bash
	$ dpkg -l | grep linux
	ii  console-setup-linux                          1.108ubuntu15.5                                                  all          Linux specific part of console-setup
	ii  libselinux1:amd64                            2.4-3build2                                                      amd64        SELinux runtime shared libraries
	ii  libv4l-0:amd64                               1.10.0-1                                                         amd64        Collection of video4linux support libraries
	ii  libv4lconvert0:amd64                         1.10.0-1                                                         amd64        Video4linux frame format conversion library
	ii  linux-base                                   4.5ubuntu1.2~16.04.1                                             all          Linux image base package
	ii  linux-firmware                               1.157.23                                                         all          Firmware for Linux kernel drivers
	ii  linux-generic-hwe-16.04                      4.15.0.142.137                                                   amd64        Complete Generic Linux kernel and headers
	hi  linux-headers-4.15.0-142                     4.15.0-142.146~16.04.1                                           all          Header files related to Linux kernel version 4.15.0
	hi  linux-headers-4.15.0-142-generic             4.15.0-142.146~16.04.1                                           amd64        Linux kernel headers for version 4.15.0 on 64 bit x86 SMP
	ii  linux-headers-generic-hwe-16.04              4.15.0.142.137                                                   amd64        Generic Linux kernel headers
	ii  linux-image-4.15.0-142-generic               4.15.0-142.146~16.04.1                                           amd64        Signed kernel image generic
	ii  linux-image-generic-hwe-16.04                4.15.0.142.137                                                   amd64        Generic Linux kernel image
	ii  linux-libc-dev:amd64                         4.4.0-210.242                                                    amd64        Linux Kernel Headers for development
	hi  linux-modules-4.15.0-142-generic             4.15.0-142.146~16.04.1                                           amd64        Linux kernel extra modules for version 4.15.0 on 64 bit x86 SMP
	hi  linux-modules-extra-4.15.0-142-generic       4.15.0-142.146~16.04.1                                           amd64        Linux kernel extra modules for version 4.15.0 on 64 bit x86 SMP
	ii  linux-signed-generic-hwe-16.04               4.15.0.142.137                                                   amd64        Complete Signed Generic Linux kernel and headers (dummy transitional package)
	ii  linux-sound-base                             1.0.25+dfsg-0ubuntu5                                             all          base package for ALSA and OSS sound systems
	ii  pptp-linux                                   1.8.0-1                                                          amd64        Point-to-Point Tunneling Protocol (PPTP) Client
	ii  syslinux                                     3:6.03+dfsg-11ubuntu1                                            amd64        collection of bootloaders (DOS FAT and NTFS bootloader)
	ii  syslinux-common                              3:6.03+dfsg-11ubuntu1                                            all          collection of bootloaders (common)
	ii  syslinux-legacy                              2:3.63+dfsg-2ubuntu8                                             amd64        Bootloader for Linux/i386 using MS-DOS floppies
	ii  util-linux                                   2.27.1-6ubuntu3.10                                               amd64        miscellaneous system utilities
	```
	
	可见我的电脑上只安装了`linux-image-4.15.0-142`这一个版本，自然使用的也是这个版本。一般使用中的内核版本是最新的版本，也就是版本号最大的那个版本。

3. 锁定内核版本

	将目前使用中的内核设定为hold。
	
	```bash
	sudo apt-mark hold linux-headers-4.15.0-142 linux-headers-4.15.0-142-generic linux-modules-4.15.0-142-generic linux-modules-extra-4.15.0-142-generic linux-image-4.15.0-142-generic
	```
	
	把内核相关的安装包都设定为hold。不同的Ubuntu版本内核相关的包的名称可能不一样，每个人电脑的内核版本也会不同，所说**请根据第二步查到的包名操作**，不要直接复制命令。

4. 检查设置

	检查设置是否生效。如下所示，则设置成功。
	
	```bash
	$ dpkg --get-selections | grep hold
	linux-headers-4.15.0-142			    hold
	linux-headers-4.15.0-142-generic		hold
	linux-image-4.15.0-142-generic			hold
	linux-modules-4.15.0-142-generic		hold
	linux-modules-extra-4.15.0-142-generic  hold
	```

5. 重新更新内核

	如果哪天需要使用最新的内核， 则需要取消内核的hold状态。然后更新即可。
	
	```bash
	sudo apt-mark unhold linux-headers-4.15.0-142 linux-headers-4.15.0-142-generic linux-modules-4.15.0-142-generic linux-modules-extra-4.15.0-142-generic linux-image-4.15.0-142-generic
	```

其实其他的软件禁止更新，操作也是一样的。

参考：

[ubuntu禁止内核自动更新](https://www.codenong.com/cs105454293/)