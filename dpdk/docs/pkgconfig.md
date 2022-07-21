# 简介
针对程序需要用到比较大的第三方库，其头文件和库文件数量比较多，手动添加效率比较低，还很繁琐；  
pkg-config能够把这些头文件和库文件的位置指出来，给编译器使用。

常用--libs和--cflags

pkg-config如何找到所需“.pc”文件，根据PKG_CONFIG_PATH来自定义配置，系统默认会到/usr/lib64/pkgconfig查找  

# 查找指定库

查找libdpdk.cp库:  
pkg-config --cflags --libs libdpdk
