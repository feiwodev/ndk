### 引言
在学习了C语言基础之后 ，我们简单的了解了C语言编程的一些范式 ， 了解了指针 ， 结构体 ， 联合体 ， 函数 ， 文件IO等等 。我们最终的目的是要学会NDK开发 ， 而NDK开发就离不开我们的JNI技术 。下面 ， 就来开始我们的JNI之旅吧 。


### JNI的概念
JNI全称 Java Native Interface ， java本地化接口 ， 可以通过JNI调用系统提供的API ， 我们知道 ， 不管是linux还是windows亦或是mac os ， 这些操作系统 ， 都是依靠C/C++写出来的 ， 还包括一些汇编语言写的底层硬件驱动 。java和C/C++不同 ， 它不会直接编译成平台机器码，而是编译成虚拟机可以运行的java字节码的.class文件，通过JIT技术即时编译成本地机器码，所以有效率就比不上C/C++代码，JNI技术就解决了这一痛点，下面我们来看看JNI调用示意图：

![JNI技术.png](http://upload-images.jianshu.io/upload_images/643851-85aeb72da3ec7186.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从上图可以得知 ，JNI技术通过JVM调用到各个平台的API ， 虽然JNI可以调用C/C++ ， 但是JNI调用还是比C/C++编写的原生应用还是要慢一点 ， 不过对高性能计算来说 ， 这点算不得什么 ， 享受它的便利 ， 也要承担它的弊端 。

### JNI开发流程
因为JNI开发并未涉及到NDK ， 所有我们的开发工具是eclipse and visual studio 。

> 第一步：编写java本地方法

```java
public static native String getStringFromC() ;
```

> 第二步：生成.h头文件 ， 需要使用到的java命令是javah

```
# 假定你以配置好java环境
# 在控制台中进入工程src目录
> cd  E:\java_workspace\Hello_JNI\src
> javah com.zeno.jni.HelloJni 
# 需要注意的是 ， com.zeno.jni.HelloJni ， 是全类名， 包名.类名
```
> 第三步：将.h头文件复制到VS的代码文件目录下 ， 在解决方案中的头文件目录-> 右键-> 添加 -> 添加现有项 。 将我们的头文件添加进来。

![添加头文件.png](http://upload-images.jianshu.io/upload_images/643851-82b09b17357b9bf8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
添加进来之后 ， 打开我们的头文件 ， 会发现 ， 头文件里面的jni.h这个头文件找不到 ， 因为它不是系统的头文件 ， 所有我们需要将它找出来 ， 并复制到我们的工程项目中 。
```
# JDK 的jni.h头文件目录
D:\Java\jdk1.8.0_60\include\jni.h
# 在jni.h头文件中，又引入了jni_md.h头文件 ， 所有这个我们也要一并赋值过来
D:\Java\jdk1.8.0_60\include\win32\jni_md.h
```
将jni.h这个头文件按照上述步骤 ， 添加到头文件目录中 ， 注意将`<>`改成`" "` ， `<>`表示引入的是系统头文件，`" "`表示引入的是第三方头文件。

> 第四步：实现头文件

```c
// 生成的头文件函数
/*
 * Class:     com_zeno_jni_HelloJni
 * Method:    getStringFormC
 * Signature: ()Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL Java_com_zeno_jni_HelloJni_getStringFromC
  (JNIEnv *, jclass);
```
新建一个.c的文件 ，引入我们生成的头文件 ，然后实现我们生成的C语言函数。
```c
/*
* Class:     com_zeno_jni_HelloJni
* Method:    getStringFormC
* Signature: ()Ljava/lang/String;
*/
JNIEXPORT jstring JNICALL Java_com_zeno_jni_HelloJni_getStringFromC
(JNIEnv *Env, jclass jclazz) {

	return (*Env)->NewStringUTF(Env, "Jni C String");
}
```

> 第五步：生成动态链接库

补充：

|库名称|特性|扩展名|
|:-----:|:----:|:----:|
|动态库|1.动态库把对一些库函数的链接载入推迟到程序运行的时期。2.可以实现进程之间的资源共享。（因此动态库也称为共享库）3.可以动态注入到程序中|win(.dll)linux(.so)|
|静态库|1.静态库对函数库的链接是放在编译时期完成的。2.程序在运行时与函数库再无瓜葛，移植方便。3.浪费空间和资源，因为所有相关的目标文件与牵涉到的函数库被链接合成一个可执行文件。|win(.lib)linux(.a)|

了解了静态库和动态库 ， 下面我们就来生成一个动态库，以VS为例：
```
选中项目 -> 右键 -> 属性 -> 常规 -> 项目默认值 -> 配置类型 , 选择动态库.dll
```

![项目属性配置.png](http://upload-images.jianshu.io/upload_images/643851-67d792e8e33cd464.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
配置完成之后 ， 选中项目 -> 生成 。即可生成动态链接库。

![动态链接库.png](http://upload-images.jianshu.io/upload_images/643851-c79d3f871eaf3fe2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 第六步：配置环境变量

我们生成了.dll文件之后 ， java环境并不知道有这个.dll动态链接库的存在 ， 所有我们需要将生成.dll的文件目录 ， 配置到环境变量中。 

![配置环境变量.png](http://upload-images.jianshu.io/upload_images/643851-364610d1de8c1d94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 注意：配置好环境变量 ， 需要重新启动eclipse ，不然会找不到动态链接库的。

> 第七步：加载动态链接库

```java
static{
		System.loadLibrary("Hello_JNI") ;
	}
```

> 第八步 ： 调用本地方法并执行

```java
/**
 * 
 * @author Zeno
 *
 *	JNI (Java Native Interface) java本地化接口
 *	
 *	Android Framework层与Native层相互通信的基石
 *	
 *
 */
public class HelloJni {
    // java调用C/C++函数的本地方法
	public static native String getStringFromC() ;
	
	public static void main(String[] args) {
		
		System.out.println("getStringFormC == "+getStringFromC());
	}
	
	static{
        // 加载动态库
		System.loadLibrary("Hello_JNI") ;
	}
}
```
输出：
```java
getStringFormC == Jni C String
```

JNI 开发的步骤虽然多 ， 但大多比较简单 ， 操作起来不难 。

### 结语
这篇是JNI系列的开篇 ， 总体来说比较简单 ， 先将流程走顺了 ， 后续的开发就会好很多 ， 流程性的东西 ， 需要多操作 ， 就会形成固定的套路 。
