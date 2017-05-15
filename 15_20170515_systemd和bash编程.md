# systemd和bash编程

## systemd

系统启动的一般过程：POST --> Boot Sequence --> Bootloader --> kernel + initramfs(initrd) --> rootfs --> /sbin/init

init的类型：

CentOS 5: SysV init

CentOS 6: Upstart

CentOS 7: Systemd

systemd兼容之前的init类型，另外systemd还有一些新的特性：

- 系统引导时实现服务并行启动；
- 按需激活进程；
- 系统状态快照；
- 基于依赖关系定义服务控制逻辑；

对于systemd有一个核心的概念叫做unit，unit用来进行系统管理，unit使用配置文件进行标识和配置，配置文件中主要包含了系统服务、监听socket、保存的系统快照以及其它与init相关的信息；每一个unit都有一个配置文件，这些配置文件通常在/usr/lib/systemd/system，/etc/systemd/system和/run/systemd/system。

### unit的类型

Service unit: 文件扩展名为.service, 用于定义系统服务；

Target unit: 文件扩展名为.target，用于模拟实现“运行级别”；

Device unit:文件扩展名为 .device, 用于定义内核识别的设备；

Mount unit: 文件扩展名为.mount, 定义文件系统挂载点；

Socket unit: 文件扩展名为.socket, 用于标识进程间通信用的socket文件；

Snapshot unit: 文件扩展名为.snapshot, 管理系统快照；

Swap unit: 文件扩展名为.swap, 用于标识swap设备；

Automount unit: 文件扩展名为.automount，文件系统的自动挂载点；

Path unit: 文件扩展名为.path，用于定义文件系统中的一个文件或目录；

### systemd的关键特性

- 基于socket的激活机制：socket与服务程序分离；
- 基于bus的激活机制：
- 基于device的激活机制：
- 基于path的激活机制：
- 系统快照：保存各unit的当前状态信息于持久存储设备中；
- 向后兼容sysv init脚本；

### systemd管理系统服务

systemd的各种服务控制是使用systemctl命令，systemctl的命令固定不变；非由systemd启动的服务，systemctl无法与之通信。

systemctl的一般使用格式`systemctl COMMAND name.service`

```
           CentOS 6                                     CentOS 7
启动：service name start              ==>          systemctl start name.service
停止：service name stop               ==>          systemctl stop name.service
重启：service name restart            ==>          systemctl restart name.service
状态：service name status             ==>          systemctl status name.service
条件式重启：service name condrestart  ==>           systemctl try-restart name.service #服务启动才会重启，没启动不会重启

查看某服务当前激活与否的状态：systemctl is-active name.service
查看所有已经激活的服务：systemctl list-units --type service 
查看所有服务：systemctl list-units --type service --all
重载或重启服务：systemctl reload-or-restart name.service
重载或条件式重启服务：systemctl reload-or-try-restart name.service
禁止设定为开机自启：systemctl mask name.service
取消禁止设定为开机自启：systemctl unmask name.service

chkconfig命令的对应关系：
设定某服务开机自启：chkconfig name on ==> systemctl enable name.service
禁止某服务开机自启：chkconfig name off ==> systemctl disable name.service
查看所有服务的开机自启状态：chkconfig --list ==> systemctl list-unit-files --type service 

查看某服务是否开机自启：systemctl is-enabled name.service

其他命令
查看服务的依赖关系：systemctl list-dependencies name.service
```

### target units

tatget units是用来模拟系统运行级别的，使用unit配置文件.target

运行级别：

0  ==> runlevel0.target, poweroff.target

1  ==> runlevel1.target, rescue.target

2  ==> runlevel2.target, multi-user.target

3  ==> runlevel3.target, multi-user.target

4  ==> runlevel4.target, multi-user.target

5  ==> runlevel5.target, graphical.target

6  ==> runlevel6.target, reboot.target

级别切换：init N ==> systemctl isolate name.target

查看级别：runlevel ==> systemctl list-units --type target

获取默认运行级别：/etc/inittab ==> systemctl get-default

修改默认级别：/etc/inittab ==> systemctl set-default name.target

切换至紧急救援模式：systemctl rescue

切换至emergency模式：systemctl emergency

### 其它常用命令

关机：systemctl halt、systemctl poweroff

重启：systemctl reboot

挂起：systemctl suspend

快照：systemctl hibernate

快照并挂起：systemctl hybrid-sleep


## bash编程systemctl

### 数组

变量是存储单个元素的内存空间，数组是存储多个元素的连续的内存空间。对数组的引用可以使用数组名加索引。索引的编号是从0开始，属于数值索引，索引也可支持使用自定义的格式，而不仅仅是数值格式，这种数组叫做关联数组。bash的数组支持稀疏格式。

引用数组中的元素：${ARRAY_NAME[INDEX]}，省略[INDEX]表示引用下标为0的元素；

声明数组：declare -a ARRAY_NAME或declare -A ARRAY_NAME，-A声明的是关联数组

**数组元素的赋值**

(1)一次只赋值一个元素：ARRAY_NAME[INDEX]=VALUE，如weekdays[0]="Sunday"，weekdays[4]="Thursday"

(2)一次赋值全部的值：ARRAY_NAME=("VAL1" "VAL2" "VAL3" ...)

(3)只赋值特定的元素：ARRAY_NAME=([0]="VAL1" [3]="VAL2" ...)

(4)read -a ARRAY

数组的长度(数组中元素的个数)：\${#ARRAY_NAME[*]}, ${#ARRAY_NAME[@]}

生成10个随机数保存于数组中，并找出其最大值；

```shell
#!/bin/bash
declare -a nums
declare -i max=0
for i in [0..9];do
	rand[$i]=$RANDOM
	echo ${rand[$i]}
	[ ${rand[$i]} -gt $max ] && max=${rand[$i]}
done
echo "Max: $max"
```

定义一个数组，数组中的元素是/var/log目录下所有以.log结尾的文件；要统计其下标为偶数的文件中的行数之和

```shell
#!/bin/bash
declare -a files
files=(/var/log/*.log)
declare -i lines=0

for i in $(seq 0 $[${#files[*]}-1]); do
	if [ $[$i%2] -eq 0 ];then
		let lines+=$(wc -l ${files[$i]} | cut -d' ' -f1) 
	fi
done

echo "Lines: $lines."	
```

**引用数组中的元素**

所有元素：\${ARRAY[@]}, ${ARRAY[*]}

数组切片：${ARRAY[@]:offset:number}

​	offset: 要跳过的元素个数

​	number: 要取出的元素个数，取偏移量之后的所有元素：${ARRAY[@]:offset}；

向数组中追加元素：ARRAY[${#ARRAY[*]}]

删除数组中的某元素：unset ARRAY[INDEX]

关联数组：declare -A ARRAY_NAME       RRAY_NAME=([index_name1]='val1' [index_name2]='val2' ...)

### 字符串处理工具

字符串切片：\${var:offset:number}；取字符串的最右侧几个字符：${var: -lengh}，冒号后必须有一空白字符；

基于模式取子串

${var#*word}，其中word可以是指定的任意字符，其功能自左而右，查找var变量所存储的字符串中，第一次出现的word, 删除字符串开头至第一次出现word字符之间的所有字符。

${var##*word}：同上，不过，删除的是字符串开头至最后一次由word指定的字符之间的所有内容；

${var%word*}：其中word可以是指定的任意字符，功能是自右而左，查找var变量所存储的字符串中，第一次出现的word, 删除字符串最后一个字符向左至第一次出现word字符之间的所有字符。

${var%%word*}：同上，只不过删除字符串最右侧的字符向左至最后一次出现word字符之间的所有字符；

查找替换：

${var/pattern/substi}：查找var所表示的字符串中，第一次被pattern所匹配到的字符串，以substi替换之；

${var//pattern/substi}: 查找var所表示的字符串中，所有能被pattern所匹配到的字符串，以substi替换之；

${var/#pattern/substi}：查找var所表示的字符串中，行首被pattern所匹配到的字符串，以substi替换之；

${var/%pattern/substi}：查找var所表示的字符串中，行尾被pattern所匹配到的字符串，以substi替换之；

查找并删除：

${var/pattern}：查找var所表示的字符串中，删除第一次被pattern所匹配到的字符串

${var//pattern}

${var/#pattern}

${var/%pattern}：

字符大小写转换：

${var^^}：把var中的所有小写字母转换为大写；

${var,,}：把var中的所有大写字母转换为小写；

变量赋值：
${var:-value}：如果var为空或未设置，那么返回value；否则，则返回var的值；
${var:=value}：如果var为空或未设置，那么返回value，并将value赋值给var；否则，则返回var的值；

${var:+value}：如果var不空，则返回value；
${var:?error_info}：如果var为空或未设置，那么返回error_info；否则，则返回var的值；

### 为脚本程序使用配置文件

(1) 定义文本文件，每行定义“name=value”
(2) 在脚本中source此文件即可

### 为脚本创建临时文件

mktemp [OPTION]... [TEMPLATE]，其中TEMPLATE: filename.XXX，XXX至少要出现三个；使用临时文件要使用命令引用如tmpfile=$(mktmp /tmp/test.XXX)

OPTION

```
-d: 创建临时目录；
--tmpdir=/PATH/TO/SOMEDIR：指明临时文件目录位置；
```

### 其他install

install和cp命令很像

install命令：
install [OPTION]... [-T] SOURCE DEST
install [OPTION]... SOURCE... DIRECTORY
install [OPTION]... -t DIRECTORY SOURCE...
install [OPTION]... -d DIRECTORY...
选项：

```
-m MODE
-o OWNER
-g GROUP
```

