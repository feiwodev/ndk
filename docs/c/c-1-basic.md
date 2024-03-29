我们知道 ， Android系统是基于linux开发 ， 采用的是linux内核 ， Android APP开发大部分也要和系统打交道 ， 只是Android FrameWork 帮我们屏蔽了系统操作 ， 我们从Android 系统的分成结构可以看出 ， Android FrameWork是通过JNI与底层的C/C++库交互 ， 例如：FreeType ，OpenGL ，SQLite ， 音视频等等 。


![Android_Framework.png](http://upload-images.jianshu.io/upload_images/643851-e630d23b3a0ff53d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


做Android为什么需要学习C/C++ ?

    1. 企业需要 ， 现在大部分招聘 ， 基本上都会要求会JNI
    2. 进阶需要 ， 如果想要研究Android源码 ， 那么不会C/C++ ， 行不通
    3. 音视频时代到来 （直播） ， 音视频处理 ， 很大部分都需要C/C++完成（音视频编解码）

那么下面就一起开始学习C吧 ！ let's go 

### C语言中的变量

编写C的时候 , 首先我们需要引入头文件 ， 就像我们写JAVA的时候 ， 需要引入包一样 ， 但C语言他不会帮你自动引入 ， 所有头文件 ， 必须你自己手动引入， 最常用的两个头文件是

``` C

    #include <stdio.h>
    #include <stdlib.h>

```
> C语言中的xxx.h的头文件 ， 里面只有函数声明 ， 没有函数实现 ， 函数实现都在xxx.c里面 。

在学习一门语言的时候 ， 我们最先了解的就是变量 ， 变量的定义 ， 变量所占大小 ， 下面我们看看C语言中的变量类型 ， 和变量大小 。 和JAVA不同的是 ， C语言变量的大小 ， 是随着操作系统变化而变化的 ， 不同的操作系统 ， 变量的大小可能不一样 。 

下面我们来查看C语言的变量类型和变量类型的大小：

``` C

    void main() {
    
        int i = 90;
        printf("int 所占字节：%d\n", sizeof(int));
        printf(" i 的值：%d\n", i);
        
        short sh = 32;
        printf("short 所占字节数：%d\n", sizeof(short));
        printf("sh 的值：%d\n", sh);
    
        long l = 12312;
        printf("long 所占字节数：%d\n", sizeof(long));
        printf("l 的值：%ld\n", l);
    
        float f = 12.3;
        printf("float 所占字节数：%d\n", sizeof(float));
        printf("f 的值：%f\n", f);
    
        double d = 234.345;
        printf("double 所占字节数：%d\n", sizeof(double));
        printf("d 的值：%lf\n", d);
    
        char c = 'c';
        printf("char 所占字节数：%d\n", sizeof(char));
        printf("c 的值：%c\n", c);
    
        // 输出字符串
        printf("输出字符串：%s\n", "我是输出的字符串");
    
        // 输出八进制
        printf("输出八进制：%#o\n", 023);
    
        // 输出十六进制
        printf("输出十六进制：%#x\n", 0x23443);
    
        // 屏幕暂停，不立即关闭
        system("pause");

}

```

> 在使用`printf()`函数的时候 ， 需要标明输出数据的类型 ， 例如:`int`类型是`%d`,`char`类型是`%c`, `\n`表示换行 等等：

``` C

    /*
        
        C 语言的基本数据类型 , 输出占位符
    
        int - %d 
        short - %d 
        long - %ld 
        float - %f 
        double - %lf 
        char - %c
        字符串 - %s
        八进制 - %o
        十六进制 - %x

    */    

```

### C语言中的指针

C语言中最重要的 ， 就是指针了 ， 没有指针 ， 就没有高级语言的那些强大的特性 ， 说到指针我们就会想到内存操作 ， 指针就是为了操作内存而生 。

下面我们来看看 ， 指针的简单使用：

```C

    // 定义一个变量i , i的值是100
    int i = 100;

    // 定义一个int类型的指针 p , 指针p存储的是i变量的地址值
    int *p = &i; // & 符号是取变量的地址值

```

> 指针存储的是变量的内存地址 , 也只能存储内存地址 ， 直接赋值整数值也会被转化成内存地址

下面我们来看一个完整的例子：

```C 

    void main() {

        // 定义一个变量i , i的值是100
        int i = 100;
        // 定义一个int类型的指针 p , 指针p存储的是i变量的地址值
        int *p = &i; // & 符号是取变量的地址值
    
        printf("i 的地址：%#x\n", &i);
        printf("i 的地址：%#x\n", p);
        printf("i 的值: %d\n", *p);
    
    
        system("pause");

    }

```

如果使用的是Visual Studio开发工具 ， 可以在在代码中打一个断点，在菜单栏`调试->窗口->内存->内存1` , 将打印出的变量地址值 ， 在输入栏中填入 ， 按回车键进行地址搜索 。如果是一堆问号或者乱码 ， 则在该窗口点击右键 ，在右键菜单中， 选择按照你打印的变量的进制位显示 ， 例如int 就按4进制位显示 ， 再在右键菜单栏中找到带符号显示 ， 基本上就能看到变量的值了 。

上述的例子中 ， 如果我们想修改i的值 ， 除了给i直接赋值外 ， 还可以通过指针来操作，如下：

```C

    定义一个变量i , i的值是100
    int i = 100 ;

    // 定义一个int类型的指针 p , 指针p存储的是i变量的地址值
    int* p = &i ;
    
    // 通过*p 我们操作i变量 ， 给i变量赋值20
    *p = 20 ;

    printf("i的值 = %d\n",i) ;

```

> 指针也是一个变量 ， 如上`int* p = &i` , p就是一个指针变量 ， 这个变量存储的就是i变量的内存地址 ， 通过`printf("p的值：%#x\n",p) printf("i的地址:%#x\n",&i)` ， 我们可以打印出指针变量p的值和i变量地址 ， 可以看到两个值是一致的 。 那么指针和普通变量有什么区别呢 ？ 指针变量的强大之处就在于 ， 他能通过内存地址去操作对应内存地址的内容 。上述例子中`*p = 20` ， 就是操作i变量的地址 ， 将i变量中的100修改为20 。

我们了解了指针的概念和基本使用 ， 下次我们就要了解 ， 二级指针 ， 多级指针 ， 函数指针 等等 ， 由此 ， 我们可以看出 ， C语言的世界， 就是一个指针的世界 ， 就如同JAVA的世界 ， 就是一个对象的世界一样 ， 两者都是其各自的核心 ， 所以我们一定要把指针弄懂 ， 学透 。
