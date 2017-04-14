#操作系统实验1

#0x00 前言
这周操作系统课有个题目要验收，主要是把一个基于时间片的线程切换转成基于优先级的线程切换。思路大家都想的到，就是修改find函数，找到一个优先级最大且没有finished的线程。但是呢只是改find，最后会报错。于是有同学把new_int8里的find提前了，也就是只有当i不同的时候才会进行进程切换，但是问了几个同学还有老师，感觉说的都比较模糊。
对了，有一个说只要把i==count那行去掉就可以运行了。但是这到底有啥区别呢，且看Niko分析一下。

#0x01 分析
从崩溃的地方可以看到，崩溃点在进行切换处，那么可以定位是over,或者是swich函数出了问题。debug了一下， 发现线程当前状态根本不是RUNNING，在会看代码

`c
void over()
{
	if(tcb[current].state==RUNNING)
	{
		disable();
		tcb[current].state=FINISHED;
		strcpy(tcb[current].name,NULL);
		free(tcb[current].stack);
		enable();
	}

	swtch();
}

new_int8()
{
			disable();
//			asm CLI

			tcb[current].ss=_SS;
			tcb[current].sp=_SP;

			if(tcb[current].state==RUNNING)
				tcb[current].state=READY;

			i=Find();

			if(i==current)
				return;

			_SS=tcb[i].ss;
			_SP=tcb[i].sp;
			tcb[i].state=RUNNING;

			timecount=0;
			current=i;

			enable();
}
`
可以看到这里讲RUNNING的线程改成READY,但是在i==count处返回，也就是线程一直是READY状态，而over函数多了一个if条件,只有RUNNING才可以finish,所以问题的关键是在状态切换上。

##0x02 修改
`c
	去掉over里的if语句。

int Find()
{
	int j;
	int max = 0;
	for(j=0; j<NTCB; j++)
	{
		if((tcb[j].pre >= tcb[max].pre) && (tcb[j].state != FINISHED))
		{
				max=j;
		}
	}

	return max;
}
`

不修改over的话就要想办法使线程一直处于RUNNING状态，这个方法应该 99%的人都在用。。。

#0x03 感想

不知道怎么说好，进入大三后状态有点下滑吧，有点像是力不从心，在课程上总体有下滑的趋势吧，或许也有点浮躁，总是不能静下心来想东西，不过最近感觉这种状态改善了不少。

