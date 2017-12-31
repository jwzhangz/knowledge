```c
#define SYSCALL_DEFINE5(name, ...) SYSCALL_DEFINEx(5, _##name, __VA_ARGS__)

#define SYSCALL_DEFINEx(x, sname, ...)				\
	SYSCALL_METADATA(sname, x, __VA_ARGS__)			\
	__SYSCALL_DEFINEx(x, sname, __VA_ARGS__)
  
注意最后一行是没有分号的。这是用来定义函数的。
#define __SYSCALL_DEFINEx(x, name, ...)					\
	asmlinkage long sys##name(__MAP(x,__SC_DECL,__VA_ARGS__))	\
		__attribute__((alias(__stringify(SyS##name))));		\
	static inline long SYSC##name(__MAP(x,__SC_DECL,__VA_ARGS__));	\
	asmlinkage long SyS##name(__MAP(x,__SC_LONG,__VA_ARGS__));	\
	asmlinkage long SyS##name(__MAP(x,__SC_LONG,__VA_ARGS__))	\
	{								\
		long ret = SYSC##name(__MAP(x,__SC_CAST,__VA_ARGS__));	\
		__MAP(x,__SC_TEST,__VA_ARGS__);				\
		__PROTECT(x, ret,__MAP(x,__SC_ARGS,__VA_ARGS__));	\
		return ret;						\
	}								\
	static inline long SYSC##name(__MAP(x,__SC_DECL,__VA_ARGS__))
```

\_\_VA_ARGS__ 是一个可变参数的宏,这个可变参数的宏是新的C99规范中新增的,目前似乎只有gcc支持(VC6.0的编译器不支持)。


__MAP 宏的定义：
```c
#define __MAP0(m,...)
#define __MAP1(m,t,a) m(t,a)
#define __MAP2(m,t,a,...) m(t,a), __MAP1(m,__VA_ARGS__)
#define __MAP3(m,t,a,...) m(t,a), __MAP2(m,__VA_ARGS__)
#define __MAP4(m,t,a,...) m(t,a), __MAP3(m,__VA_ARGS__)
#define __MAP5(m,t,a,...) m(t,a), __MAP4(m,__VA_ARGS__)
#define __MAP6(m,t,a,...) m(t,a), __MAP5(m,__VA_ARGS__)
#define __MAP(n,...) __MAP##n(__VA_ARGS__)
```

__SC_DECL 定义，主要作用是合并参数。
```c
#define __SC_DECL(t, a)	t a
```
 
接下来看一下sys_mount的定义。
```c
SYSCALL_DEFINE5(mount, char __user *, dev_name, char __user *, dir_name,
		char __user *, type, unsigned long, flags, void __user *, data)
{
    ...
}
```
 
下面做宏替换
```c
SYSCALL_DEFINE5(mount, char __user *, dev_name, char __user *, dir_name,
		char __user *, type, unsigned long, flags, void __user *, data)
数字5表示有5个参数；
第一个参数mount表示函数名字是sys_mount；
 
先把name替换。
#define __SYSCALL_DEFINEx(5, _mount, char __user *, dev_name, char __user *, dir_name,
		char __user *, type, unsigned long, flags, void __user *, data)					\
	asmlinkage long sys_mount(__MAP(5,__SC_DECL,__VA_ARGS__))	\
		__attribute__((alias(__stringify(SyS_mount))));		\
	static inline long SYSC_mount(__MAP(5,__SC_DECL,__VA_ARGS__));	\
	asmlinkage long SyS_mount(__MAP(5,__SC_LONG,__VA_ARGS__));	\
	asmlinkage long SyS_mount(__MAP(5,__SC_LONG,__VA_ARGS__))	\
	{								\
		long ret = SYSC_mount(__MAP(5,__SC_CAST,__VA_ARGS__));	\
		__MAP(5,__SC_TEST,__VA_ARGS__);				\
		__PROTECT(5, ret,__MAP(5,__SC_ARGS,__VA_ARGS__));	\
		return ret;						\
	}								\
	static inline long SYSC_mount(__MAP(5,__SC_DECL,__VA_ARGS__)) 
 
替换 __MAP 宏
#define __MAP(n,...) __MAP##n(__VA_ARGS__)
#define __MAP5(m,t,a,...) m(t,a), __MAP4(m,__VA_ARGS__)

__MAP(5,__SC_DECL, char __user *, dev_name, char __user *, dir_name,
		char __user *, type, unsigned long, flags, void __user *, data)
-->
__MAP5(__SC_DECL,char __user *, dev_name, char __user *, dir_name,
		char __user *, type, unsigned long, flags, void __user *, data)
-->
__SC_DECL(char __user *, dev_name), __MAP4(__SC_DECL, char __user *, dir_name,
		char __user *, type, unsigned long, flags, void __user *, data)
每一个 __SC_DECL 处理一对参数，变量类型和变量名。以此类推，一共处理5对参数。
最终得到：
char __user * dev_name, char __user * dir_name,
		char __user * type, unsigned long flags, void __user * data
```
 
最终展开得到的是如下的代码。中间还有几个宏，就先不展开了。基本的脉络就出来了。
```c
long sys_mount(char __user * dev_name, char __user * dir_name,
    char __user * type, unsigned long flags, void __user * data)
    __attribute__((alias("SyS_mount")));
    
static inline long SYSC_mount(char __user * dev_name, char __user * dir_name,
    char __user * type, unsigned long flags, void __user * data);

long SyS_mount(char __user * dev_name, char __user * dir_name,
    char __user * type, unsigned long flags, void __user * data);
long SyS_mount(char __user * dev_name, char __user * dir_name,
    char __user * type, unsigned long flags, void __user * data)
{
    long ret = SYSC_mount(char __user * dev_name, char __user * dir_name,
    char __user * type, unsigned long flags, void __user * data);
    __MAP(5,__SC_TEST,__VA_ARGS__);	
    __PROTECT(5, ret,__MAP(5,__SC_ARGS,__VA_ARGS__));
    return ret;	
}

static inline long SYSC_mount(char __user * dev_name, char __user * dir_name,
    char __user * type, unsigned long flags, void __user * data) 
```
