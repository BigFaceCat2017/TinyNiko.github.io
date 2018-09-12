# 0x00

前端时间看看dexclassloader 和pathclassloader  ，发现网上大面积的这种结论

>DexClassLoader：能够加载未安装的jar/apk/dex 
PathClassLoader：只能加载系统中已经安装过的apk

真的不知道他们是看的4.x的代码还是什么，感觉就是文章抄来抄去。。。
看网上的文章会发现大部分 到下面的代码就没有了

```java

807// Multidex files make it possible that some, but not all, dex files can be broken/outdated. This
808// complicates the loading process, as we should not use an iterative loading process, because that
809// would register the oat file and dex files that come before the broken one. Instead, check all
810// multidex ahead of time.
811bool ClassLinker::OpenDexFilesFromOat(const char* dex_location, const char* oat_location,
812                                      std::vector<std::string>* error_msgs,
813                                      std::vector<const DexFile*>* dex_files) {
814  // 1) Check whether we have an open oat file.
815  // This requires a dex checksum, use the "primary" one.
816  uint32_t dex_location_checksum;
817  uint32_t* dex_location_checksum_pointer = &dex_location_checksum;
818  bool have_checksum = true;
819  std::string checksum_error_msg;
820  if (!DexFile::GetChecksum(dex_location, dex_location_checksum_pointer, &checksum_error_msg)) {
821    // This happens for pre-opted files since the corresponding dex files are no longer on disk.
822    dex_location_checksum_pointer = nullptr;
823    have_checksum = false;
824  }
825
826  bool needs_registering = false;
827
828  const OatFile::OatDexFile* oat_dex_file = FindOpenedOatDexFile(oat_location, dex_location,
829                                                                 dex_location_checksum_pointer);
```

从这边看 ，确实是，如果要加载的dex不在某个列表里 就返回空 ，但是这并不能说PathClassLoader 只能加载系统中安装过的apk，因为上面的代码下面还有很多代码，
所以说其实在art下PathClassLoader是可以加载dex的，当然对于apk 我并没有测试
