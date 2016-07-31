#OSX在Finder中显示隐藏文件
OSX是基于FreeBSD（Unix）的，在使用时，发现很多Unix的文件和目录像/etc,/var,/bin都找不到，其实是系统隐藏了，你可以通过终端进行访问，但如何在图形化界面访问隐藏的系统文件呢？
## 显示隐藏文件

```
defaults write com.apple.finder AppleShowAllFiles -bool true
KillAll Finder
```
## 隐藏文件

```
defaults write com.apple.finder AppleShowAllFiles -bool False
KillAll Finder
```