# CentOS系统安装和SElinux

## CentOS系统安装

对于CentOS来说，安装系统时使用的是anaconda，anaconda是一个既有文本界面(tui)又有图形界面(gui)的二进制程序，文本界面是基于curses的。实际上在系统的安装介质上就有一个微缩的内核。

以CentOS6为例：

安装光盘中MBR的bootloader是boot.cat，这也类似于grub的第一阶段，第二阶段使用的文件是isolinux/isolinux.bin，配置文件在isolinux/isolinux.cfg，加载的内核在isolinux/vmlinuz，像内核传递参数使用append。例如append initrd=initrd.img。内核加载完成后会展开rootfs，根就直接使用initrd的根，不存在根切换。然后就启动了anaconda程序，程序会根据系统的资源来判断是否使用gui。如果要强制使用文本界面，在选中的选项`install or upgrade an existing system`使用tab键进入编辑模式，在文本后面加上text，也可以使用esc键可以进入boot提示符，输入标签，就可以启动anaconda，如linux text。上面的过程一般应位于引导设备。

**anaconda应用的工作过程**

第一阶段：安装前的配置

- 安装过程中使用的语言
- 键盘类型
- 安装目标的存储设备：本地和特殊设备
- 设定主机名和网络等
- 设定时区
- 设定管理员密码
- 设定分区格式和安装mbr

第二阶段：安装过程

- 在目标磁盘创建分区，执行格式化
- 将选定的程序包安装到目标位置，安装bootloader。
- 安装完成重启后会进行系统第一次启动设置。

anaconda程序以及安装用到的程序包可以在本地光盘、本地硬盘或者ftp server、http server、nfs server。如果想指定安装源在boot命令行：linux method，也可以使用附加的方式，在文本后加method。





