## Ubuntu 16.04安装Paramiko模块

### 1.下载模块包paramiko-1.10.1.tar.gz

下载后把模块包拷贝到python模块文件夹下
sudo cp paramiko-1.10.1.tar.gz /usr/local/lib/python2.7/dist-packages
不知道路径可以在shell下：
`python
import sys
print sys.path`

### 2.解压模块包

使用sudo tar -zxvf paramiko-1.10.1.tar.gz 进行解压

### 3.安装模块

进入解压后的模块文件夹/usr/local/lib/python2.7/dist-packages/paramiko-1.10.1，使用sudo python setup.py install安装