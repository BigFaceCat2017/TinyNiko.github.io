# unity mono android 安全相关

## 0x0 编译
首先从网上了解到，github上mono源码有两种，一种是mono自己的，一种是unity3d的，我们需要的就是u3d.因此，需要直接去u3d的git那下载。 下载完后，看网上的指南，有出现了两种，一种是按unity版本来的， 比如unity-5.6，unity-5.0这种，另外一个是unity-mono-4.x这种，鉴于自己没什么经验，就试了两种方法，结果第二种不行。第一种也需要做一定的改动，在根目录下，执行external/buildscript/build_runtime_android.sh 的时候，会下载ndk_r10e,但是这个r10e提供的clang版本是3.5的，所以需要改动krait中的Applcation.mk中的clang版本，然后再次运行runtime脚本就可以编译除mono.so了。

## 0x01 修改
我们知道在mono_image_open_from_data_with_name 中可以dump出dll的源码，鉴于mono是开源的，所以大家可以修改mono的源码，在这个函数里面可以自己加一些解密的代码。可以想到，其实我们可以在这个函数的上层做一些解密的操作，因为常规的，大家都知道这个函数里有解密函数了。当然，解密放到上层其实也没有太大的意义，下层也是一样，交叉引用看一下很快就可以对比出异样来了，所以，在有源码的情况下，对攻击者还是很有利的。但是这种方式防不住hook,只要hook住了这个函数，就可以很轻松的dump出dll。那么有没有其他什么办法呢。

## 0x02 进阶
前面提到了一个现象，攻击者可以根据比对源码来分析那里做了修改，从而确定解密的方法，那么，如果我使用ollvm来混淆我的代码，就可以一定程度上增加比对的难度，但是这个还是防不住hook。我想过给源码加个函数名混淆，但是这个还是可以通过定位字符串等方法来判断出一个函数，从而慢慢还原，不管是攻击还是防御，都有一定的工作量。所以现在的一个想法就是自己hook自己,从源码中可以看到image 这个变量其实还是保存了我们的地址和长度的，所以后续一系列的函数都是可以被hook，然后可以自己进行解析，拿到data和len,所以，这个思路就和dex文件加载差不多了,后续的函数都可以自己实现，不要用到源码中提供的这些函数，然后，对于我们自己的函数，可以通过so相关技术保护起来，也就是说，如果想要知道解密算法，就必须现对so脱壳。那么，如果直接去找dll呢，我们知道，dll最终肯定会在内存里的，那么是不是可以直接dump整个内存，然后自己去匹配头呢？ 我觉得这个方法是可行的。所以防御方就要想办法不让你找到匹配的内容，或者干脆对于检测到dump的直接推出游戏等等。这也是我目前的一些想法。
```c
	datac = data;
	if (need_copy) {
		datac = g_try_malloc (data_len);
		if (!datac) {
			if (status)
				*status = MONO_IMAGE_ERROR_ERRNO;
			return NULL;
		}
		memcpy (datac, data, data_len);
	}

	image = g_new0 (MonoImage, 1);
	image->raw_data = datac;
	image->raw_data_len = data_len;
	image->raw_data_allocated = need_copy;
	image->name = (name == NULL) ? g_strdup_printf ("data-%p", datac) : g_strdup(name);
	iinfo = g_new0 (MonoCLIImageInfo, 1);
	image->image_info = iinfo;
	image->ref_only = refonly;
	image->ref_count = 1;

	image = do_mono_image_load (image, status, TRUE, TRUE);
	if (image == NULL)
		return NULL;

	return register_image (image);
```
## 0x03 后续
等逆向分析了更多的游戏再来看看这块更好的方案
