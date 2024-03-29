# msys2安装与更新

## 下载MSYS2

进入官网下载[MSYS2](https://www.msys2.org/)

安装后开始菜单进入msys2 msys

更新包数据库和基础包pacman -Syu

更新其余基本软件包pacman -Su

## 安装GCC/G++

pacman -S --needed base-devel mingw-w64-x86_64-toolchain

gcc路径：msys64\mingw64\bin

## 安装命令
```
pacman -S 软件名: 安装软件。也可以同时安装多个包，只需以空格分隔包名即可。
pacman -S --needed 软件名1 软件名2: 安装软件，但不重新安装已经是最新的软件。
pacman -Sy 软件名：安装软件前，先从远程仓库下载软件包数据库(数据库即所有软件列表)。
pacman -Sv 软件名：在显示一些操作信息后执行安装。
pacman -Sw 软件名: 只下载软件包，不安装。
```

## 更新命令
```
pacman -Sy: 从服务器下载新的软件包数据库（实际上就是下载远程仓库最新软件列表到本地）。
pacman -Su: 升级所有已安装的软件包。
pacman -Syu 结合上面两个操作
#在msys2中 pacman -Syu后需要重启一下msys2(关掉shell重新打开即可)。
```

## 卸载命令
```
# usage:  pacman {-R --remove} [options] <package(s)>
pacman -R 软件名: 该命令将只删除包，保留其全部已经安装的依赖关系
pacman -Rv 软件名: 删除软件，并显示详细的信息
pacman -Rs 软件名: 删除软件，同时删除本机上只有该软件依赖的软件。
pacman -Rsc 软件名: 删除软件，并删除所有依赖这个软件的程序，慎用
pacman -Ru 软件名: 删除软件,同时删除不再被任何软件所需要的依赖
```

## 搜索
```
pacman -Ss 关键字: 在远端仓库中搜索匹配字符串的软件包（本地已安装的会标记）
pacman -Sl <repo>:显示软件仓库中所有软件的列表
pacman -Qs 关键字: 在本地已安装包中搜索匹配字符串的软件包
pacman -Qu: 列出所有可升级的软件包
pacman -Qt: 列出不被任何软件要求的软件包
pacman -Q 软件名: 查看软件包是否已安装，已安装则显示软件包名称和版本
pacman -Qi 软件名: 查看某个软件包信息，显示较为详细的信息，包括描述、构架、依赖、大小等等
pacman -Ql 软件名: 列出软件包内所有文件，包括软件安装的每个文件、文件夹的名称和路径
```

## 清理
```
pacman -Sc：清理已删除的包文件，从缓存目录（ /var/cache/pacman/pkg/）
pacman -Scc：清理所有的缓存文件。
```