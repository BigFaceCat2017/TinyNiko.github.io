# recoding somethings

# 2018.11.2

1. 测试发现了一个算法的使用bug ，挺神奇的，一个算法竟然会去改原始的iv,所以加解密用的同一个iv，那么实际上 这两个iv不一样了，解密也就回出问题。另外同一个算法，也有好几种实现，api 使用上需要注意

# 2018.11.7

1. python项目的依赖库可以通过安装pipreqs 生成req.txt 但是python3上有问题 。后来测试了一个pigar ，这个python3可以用，输出效果也很不错。赞一个

# 2018.11.22

1. 升级mac系统就是个错误的决定
2. 升级并不可怕，可怕的是不知道怎么还原

# 2018.11.25

1. arm 有个neon指令集，这个指令集主要是用于加速多媒体和信号处理算法，用到了128位寄存器。有16个128位四字到寄存器Q0-Q15，32个64位双子寄存器D0-D31，两个寄存器是重叠的。

# 2018.11.26

1. udf  函数 
2. iw wlan0 scan 扫描bssid