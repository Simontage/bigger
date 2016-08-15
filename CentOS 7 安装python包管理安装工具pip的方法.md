#  CentOS 7 安装python包管理安装工具pip的方法



## 下载源码

使用wget下载pip源码，目前pip版本号为8.1.2

`wget https://pypi.python.org/packages/e7/a8/7556133689add8d1a54c0b14aeff0acb03c64707ce100ecd53934da1aa13/pip-8.1.2.tar.gz#md5=87083c0b9867963b29f7aba3613e8f4a --no-check-certificate`

注意：

wget获取https的时候要加上--no-check-certificate；

wget默认下载的目录是当前执行wget时的目录

## 解压文件



使用tar命令解压下载好的文件 `tar -zxvf pip-8.1.2.tar.gz`



## 安装pip

打开解压后的目录 `cd pip-8.1.2`

使用python命令安装 `python setup.py install`



这样python的包管理安装工具就安装好了

