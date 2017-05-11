# CentOS系统安装和SElinux

## CentOS系统安装

对于CentOS来说，安装系统时使用的是anaconda，anaconda是一个既有文本界面(tui)又有图形界面(gui)的二进制程序，文本界面是基于curses的。实际上在系统的安装介质上就有一个微缩的内核。

**以CentOS6为例：**

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
- 选择要安装的

第二阶段：安装过程

- 在目标磁盘创建分区，执行格式化
- 将选定的程序包安装到目标位置，安装bootloader。
- 安装完成重启后会进行系统第一次启动设置。

anaconda程序以及安装用到的程序包可以在本地光盘、本地硬盘或者ftp server、http server、nfs server。如果想指定安装源在boot命令行：linux method，也可以使用附加的方式，在文本后加method。

anaconda可以通过交互式配置，也可以通过读取事先给定的配置文件自动完成配置，这个文件是kickstart，这个文件要按特定语法进行配置。

**安装引导选项**

前面说到的引导选项有text（文本安装方式）和method（手动指定使用的安装方法）

- 与网络相关的引导选项有：

ip=

netmask=

gateway=

dns=

ifname=NAME:MAC_ADDR，要应用于哪个网络接口

- 与远程访问功能相关的引导选项：vnc=，vncpassword=


- 指定kickstart文件的位置：ks=

DVD Drive：ks=cdrom：/PATH/TO/KICKSTART_FILE
Hard Drive：ks=hd：/device/directory/kickstart_file

HTTP server：ks=http://host:port/path/to/KICKSTART_FILE

FTP server: ks=ftp://host:port/path/to/KICKSTART_FILE

HTTPS server: ks=https://host:port/path/to/KICKSTART_FILE

- 启动紧急救援模式：rescue

**kickstart文件的格式**

在root用户的家目录上会生成本次安装后，生成的配置文件anaconda-ks.cfg。一个配置文件有三个部分，分为命令段、程序包段和脚本段。

命令段中指明各种安装前配置，如键盘类型等；

程序包段指明要安装的程序包组或程序包，不安装的程序包等，以%packages开始，以%end结束，在它们之间中间@group_name表示要安装的包组，package表示安装的包，-package表示不安装。

脚本段有安装前脚本和安装后脚本，%pre安装前脚本，运行环境在安装介质上的微型Linux环境。%post:安装后脚本，运行环境为安装完成的系统。

**命令段中的命令**

- 必给命令

```
authconfig: 认证方式配置
	authconfig --useshadow  --passalgo=sha512
bootloader：bootloader的安装位置及相关配置
	bootloader --location=mbr --driveorder=sda --append="crashkernel=auto crashkernel=auto rhgb rhgb quiet quiet"
keyboard: 设定键盘类型
lang: 语言类型
part: 创建分区
rootpw: 指明root的密码
timezone: 时区
```
- 可选命令

```
install OR upgrade
text: 文本安装界面
network
firewall
selinux
halt
poweroff
reboot
repo
user：安装完成后为系统创建新用户
url: 指明安装源
```
**创建kickstart文件的方式**

(1) 直接手动编辑，可以依据某模板修改；

(2) 可使用创建工具：system-config-kickstart (CentOS 6)，这是一个图形程序，也可以依据某模板修改并生成新配置；

检查ks文件的语法错误：ksvalidator

```
# ksvalidator /PATH/TO/KICKSTART_FILE
```
在anaconda运行前读取配置文件：
```
boot：linux ip=172.16.23.12 netmsak=255.255.0.0 ks=http://172.16.0.1/centos6.x86_64.cfg
```
创建引导光盘：

```
# mkisofs -R -J -T -v --no-emul-boot --boot-load-size 4 --boot-info-table -V "CentOS 6.6 x86_64 boot" -b isolinux/isolinux.bin -c isolinux/boot.cat -o /root/boot.iso myiso/
```

## SELinux

linux的安全等级实际上和windows server相同，SELinux则是增强安全的linux，SELinux（Secure Enhanced Linux）工作于Linux内核中，实际上在生产环境中使用的比例不多。linux的安全模型是自主访问控制(DAC)，而SELinux的安全模型是强制访问控制(MAC)。

SELinux有两种工作级别：

**strict**: 每个进程都受到selinux的控制；

**targeted**: 仅有限个进程受到selinux控制；只监控容易被入侵的进程；

对于linux的进程的执行，主要就是进程的执行者、进程操作对象，和动作这三个部分。被操作的对象可以使进程也可以文件，对于文件可以进行的动作有open，read，write，close，chown，chmod等。

SELinux为每个文件提供了安全标签，也为进程提供了安全标签。标签有5段组成，后两段在rhel系用不到。user:role:type

- user是SELinux的user


- role是角色，类似组


- type是类型，每个进程和文件都有类型，对于进程来说类型叫domain，每一进程可以访问的资源被限定在domain所允许访问的类型的内部。一个大domain可包含多个小的domain。

开启selinux后，访问一个文件除了受属主、属组、其他，还受selinux的domain限制。

任何一个进程能访问哪些文件并不是严格按照进程自己的域和被访问者自己的额类型是否匹配，而是根据系统的SELinux规则。在规则库中定义了哪种域能访问哪种或哪些种类型内文件；对于selinux，遵循了“法无授权不可为”的法则。（对于民众法无禁止即可为），因为需要对规则进行对比，会造成系统运行缓慢，所以将规则库进行二进制编码后使用。

**配置SELinux**

1. SELinux是否启用
  SELinux的状态
  - enforcing: 强制，每个受限的进程都必然受限；


  - permissive: 启用，每个受限的进程违规操作不会被禁止，但会被记录于审计日志；


  - disabled: 关闭；

  相关命令

  - getenforce: 获取selinux当前状态；


  - setenforce 0|1  

    0设置为permissive  

    1设置为enforcing

    此设定重启系统后无效；

    配置文件/etc/sysconfig/selinux, /etc/selinux/config SELINUX={disabled|enforcing|permissive}

2. 给文件重新打标签chcon

   chcon [OPTION]... CONTEXT FILE...

   chcon\ [OPTION]...\ [-u USER]\ [-r ROLE] \[-t TYPE] FILE...

   chcon [OPTION]... --reference=RFILE FILE...

   -R递归打标

   还原文件的默认标签：restorecon [-R] /path/to/somewhere

3. 设定某些布尔型特性getsebool/setsebool
   getsebool命令：getsebool \[-a] [boolean]
   setsebool命令：etsebool [ -P] boolean value | bool1=val1 bool2=val2 ...



