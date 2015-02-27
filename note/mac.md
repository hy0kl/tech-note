# 一个动态库报错的问题记录

```
在用户目录下安装了 libjpeg 后, pbcopy 报错了.
$ pbcopy
dyld: Symbol not found: __cg_jpeg_resync_to_restart
  Referenced from: /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libTIFF.dylib
  Expected in: /Users/hy0kl/local/lib/libJPEG.dylib
 in /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libTIFF.dylib
Trace/BPT trap: 5
```

[参考](http://stackoverflow.com/questions/17643509/conflict-between-dynamic-linking-priority-in-osx)

```
You should not set library paths using DYLD_LIBRARY_PATH. As you've discovered, that tends to explode. Executables and libraries should have their library requirements built into them at link time. Use otool -L to find out what the file is looking for:

$ otool -L /System/Library/Frameworks/ImageIO.framework/Versions/A/ImageIO
/System/Library/Frameworks/ImageIO.framework/Versions/A/ImageIO:
    /System/Library/Frameworks/ImageIO.framework/Versions/A/ImageIO (compatibility version 1.0.0, current version 1.0.0)
    ...
    /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1197.1.1)
For an example of one of my homebrew-built programs:

$ otool -L /usr/local/bin/gifcolor
/usr/local/bin/gifcolor:
    /usr/local/Cellar/giflib/4.1.6/lib/libgif.4.dylib (compatibility version 6.0.0, current version 6.6.0)
    /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 159.1.0)
Note that it references /usr/local. If you've built it in such a way that it references the wrong library, I recommend rebuilding and pointing it to the correctly library.

If that's impossible, it is possible to edit what path is used using install_name_tool, but there are cases where this doesn't work, such as if the new path is longer than the old path and you didn't link it with -header_pad_max_install_names. Rebuilding with the correct path is preferred.

Note that there are a few "special" paths available that allow libraries to be found relative to their loader. See @executable_path/ and its kin in the dyld(1) man page.
```

# aclocal: command not found

need install automake.[see](http://stackoverflow.com/questions/9575989/install-autoreconf-on-osx-lion)

[ftp下载](ftp://ftp.gnu.org/gnu/automake/)

标准安装

# pkg-config: command not found

[下载](http://pkgconfig.freedesktop.org/releases/)

[see](http://blog.csdn.net/yuanya/article/details/8801736)

```
1.检测环境是否已安装 pkg-config
再命令行中输入: pkg-config 若未安装, 则提示命令未找到.

2.安装pkg-config

curl http://pkgconfig.freedesktop.org/releases/pkg-config-0.28.tar.gz -o pkg-config-0.28.tar.gz

tar -xf pkg-config-0.28.tar.gz
cd pkg-config-0.28

./configure  --with-internal-glib

make
sudo  make install
```

# liblzma

[主页](http://tukaani.org/xz/)

# shuf 命令不存在

```
$ shuf
-bash: shuf: command not found
```

谷歌说要安装 `gnu coreutils`,下载了一份拷贝后发现,文件数挺多,为了一个 shuf 命令折腾一大圈,有点不划算.

[Coreutils - GNU core utilities](https://www.gnu.org/software/coreutils/)

先用 awk 代替,实在不成再来折腾.

