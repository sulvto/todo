
 源码编译升级安装了gcc后，编译程序或运行其它程序时，有时会出现类似/usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.21' not found的问题。这是因为升级gcc时，生成的动态库没有替换老版本gcc的动态库导致的，将gcc最新版本的动态库替换系统中老版本的动态库即可解决。

1. 问题原因分析

为了安装最新版本的Node.js（最新版本的Node.js使用了C++ 11中，而C++ 11需要code>gcc 4.8+才能支持），将gcc升级到了当前最新版本v 5.2.0。升级后，成功编译安装了新版本的Node.js（v 4.2.1）,但运行时程序时出现了以下错误： 

node: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by node)
node: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.15' not found (required by node)
node: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.20' not found (required by node)

运行以下命令检查动态库： 

strings /usr/lib64/libstdc++.so.6 | grep GLIBC

 输出结果如下： 

GLIBCXX_3.4
GLIBCXX_3.4.1
GLIBCXX_3.4.2
GLIBCXX_3.4.3
GLIBCXX_3.4.4
GLIBCXX_3.4.5
GLIBCXX_3.4.6
GLIBCXX_3.4.7
GLIBCXX_3.4.8
GLIBCXX_3.4.9
GLIBCXX_3.4.10
GLIBCXX_3.4.11
GLIBCXX_3.4.12
GLIBCXX_3.4.13
GLIBCXX_FORCE_NEW
GLIBCXX_DEBUG_MESSAGE_LENGTH

 从以上输出可以看出，gcc的动态库还是旧版本的。说明出现这些问题，是因为升级gcc时，生成的动态库没有替换老版本gcc的动态库。


2. 问题处理

执行以下命令，查找编译gcc时生成的最新动态库： 

find / -name "libstdc++.so*"

 输出如下： 

/home/gcc-5.2.0/gcc-temp/stage1-x86_64-unknown-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so
/home/gcc-5.2.0/gcc-temp/stage1-x86_64-unknown-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6
/home/gcc-5.2.0/gcc-temp/stage1-x86_64-unknown-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6.0.21  //最新动态库
……

 /home/gcc-5.2.0/gcc-temp是升级gcc时的输出目录。

将上面的最新动态库libstdc++.so.6.0.21复制到/usr/lib64目录下： 

cp /home/gcc-5.2.0/gcc-temp/stage1-x86_64-unknown-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6.0.21 /usr/lib64

 复制后，修改系统默认动态库的指向，即：重建默认库的软连接。

切换工作目录至/usr/lib64：

cd /usr/lib64

 删除原来软连接： 

rm -rf libstdc++.so.6

 将默认库的软连接指向最新动态库：

ln -s libstdc++.so.6.0.21 libstdc++.so.6


默认动态库升级完成。重新运行以下命令检查动态库：

strings /usr/lib64/libstdc++.so.6 | grep GLIBC

现在输出如下：

GLIBCXX_3.4
GLIBCXX_3.4.1
GLIBCXX_3.4.2
GLIBCXX_3.4.3
GLIBCXX_3.4.4
GLIBCXX_3.4.5
GLIBCXX_3.4.6
GLIBCXX_3.4.7
GLIBCXX_3.4.8
GLIBCXX_3.4.9
GLIBCXX_3.4.10
GLIBCXX_3.4.11
GLIBCXX_3.4.12
GLIBCXX_3.4.13
GLIBCXX_3.4.14
GLIBCXX_3.4.15
GLIBCXX_3.4.16
GLIBCXX_3.4.17
GLIBCXX_3.4.18
GLIBCXX_3.4.19
GLIBCXX_3.4.20
GLIBCXX_3.4.21
GLIBC_2.3
GLIBC_2.2.5
GLIBC_2.3.2
GLIBCXX_FORCE_NEW
GLIBCXX_DEBUG_MESSAGE_LENGTH
