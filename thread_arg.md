# Linux Pthread_create 传参问题

## 0x00 前言

某东的课要求我们实现一个用socket通信的程序, 用C/C++写,虽然用
很多语言写过这个了,但是这次想在linux平台上试一下。于是就写了个代码测试一下,然而在使用pthread的时候,程序报了一个貌似函数类型错误的的错误,然后就大致的了解了一下。

## 0x1 解析

```c
#include <pthread.h> 
int  pthread_create((pthread_t  *thread,  pthread_attr_t  *attr,  void  *(*start_routine)(void  *),  void  *arg)
//另外gcc 编译的时候加上 -pthread 选项
```

这里要注意的是 线程函数的类型 必须是void (func)(void ) , 然后第四个参数是传递给func的参数,这里可以看到,只能传一个void *类型的
指针给线程函数,当然,不传就是NULL.

```c
//顺手写一个小demo
void *func(void *arg)
{
	int da=*(int *)arg ;
	dosth() ; 
	return NULL ; 
}

typedef struct
{
int a1 ;
int b2 ; 
}hehe,*heheptr;
int arg ;

pthread_t threadid ; 
if(pthread_create(&threadid ,NULL, func ,(void *)&arg)==-1)
{
	printf("error %s(%d)" ,strerror(errno) ,errno) ; 
	exit(0) ; 
}
```


上面这个展示了如何传单个参数,如果想传递多个参数的话就需要用到结构体了,
有了上面的demo后,传结构体其实也比较简单了, 就用结构体指针指一下就OK了。以上是关于create的问题 。下面讲一下如何判断线程是否还活着。


```c
pthread_kill(pthread_t id , int sig) ;
```

当sig==0 时, 用于测试线程是否还活着, 另外 当返回是0的时候线程还是活着的, 当线程返回ESRCH 表示线程不存在,EINVAL则说明sig不合法。


引用
[stackoverflow](http://stackoverflow.com/questions/11253025/pthread-create-not-working-passing-argument-3-warning)
