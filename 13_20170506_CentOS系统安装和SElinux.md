# CentOS系统安装和SElinux

## CentOS系统安装

对于CentOS来说，安装系统时使用的是anaconda，anaconda是一个既有文本界面(tui)又有图形界面(gui)的二进制程序，文本界面是基于curses的。实际上在系统的安装介质上就有一个微缩的内核。

以CentOS6为例：

安装光盘中MBR的bootloader是boot.cat，这也类似于grub的第一阶段，第二阶段使用的文件是isolinux/isolinux.bin，配置文件在isolinux/isolinux.cfg，加载的内核在isolinux/vmlinuz，像内核传递参数使用append。例如append initrd=initrd.img。