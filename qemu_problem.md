#Qemu+Mips 配置
##0x00
最近在搞mips相关的东西。搞的心累。。。。
记几个遇到的问题吧
##0x01
0 首先得有qemu,已经经过buildroot 编译过的mips-gcc
这个过程比较费时，网上有相关的说明，就不多说了。
下面说说我遇到的几个问题

1 交叉编译mips程序
在我编译完buildroot 后， 在output/xxxxx/bin 目录里会有一些mips-linux-gcc
,使用这些gcc编译的时候，会出现找不到xxx.bx_real的错误，这里我的解决方法是
改名字，然后把bin这个目录加到PATH中。

2 sudo chroot . ./qemu_mipsel hello
这里的hello是使用动态链接的。
报错说 找不到 qemu_mipsel 这个 文件。 在stackoverflow上看到一个解决方案，
apt-get install qemu-user-static
然后将对应平台的static 文件放到hello所在目录就好， 同时还需要把lib目录放在一起
然后就可以执行了。

