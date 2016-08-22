# Linux用户(组)管理

## 用户管理

### useradd

 useradd [options] USERNAME

- -u指定UID `useradd -u 1000 USERNAME`
- -g指定GID(基本组) 
- -G指定额外附加组，可以有多个已存在的组名或组ID，使用，隔开
- -c指定注释信息，使用双引号隔开
- -d指定家目录 `-d /path/to/somedirectory`
- -s指定shell路径，`-s /sbin/nologin`不允许用户登录
- -m -k 指定强制为用户创建家目录，并将`/etc/skel`拷贝到用户家目录
- -M不为用户创建家目录，即使`/etc/login.defs`已经存在的默认设置

### userdel

`userdel [OPTIONS] USERNAME`，默认用户家目录不会被删除

- -r删除用户的同时删除家目录

### USERMOD

修改用户的属性

- -u          修改UID
- -g          修改基本组GID（必须是已经存在的组）
- -G -a     修改附加组GID，此前的附加组失效，和-a组合变成附加到先前的附加组后
- -c          修改注释信息
- -d -m    修改家目录 -m当前的家目录下的文件会移动到新的家目录下
- -s          修改shell

### ID

显示ID号码相关信息

- -u显示UID` id -u username`
- -G显示所属所有组的GID
- -n显示名称代替显示号码

### finger

查看用户相关属性信息 `finger username`