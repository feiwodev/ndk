### 前情提要
IO操作 ， 一直在开发中占据很大比重 ， 在Java中不管是网络操作还是文件操作 ， 都作为IO流来处理 ， 都依靠`InputStream`和`OutputStream`这两个输入输出流 。在上篇中 ， 使用了C语言的IO流 ， 进行了文件的加密与解密，分割与合并 。其要点是，加密解密使用了`^`运算 ，分割文件则使用了，文件大小与文件个数的`%`运算 。

### 为什么需要增量更新？
当我们开发完一个项目之后，上线维护 ， 增加新功能 ， 添加第三方库 ， APK大小从4 - 5M ， 增长到10+M ，  用户由原来的几十秒下载 ， 到现在几分钟以上的下载 ， 网络情况不好的时候 ， 或许就是十分钟不等。每次全量下载 ， 无论从体验还是流量上 ， 都是不友好的 ， 所有增量更新还是有必要的 （小公司好像没几个用 ， 一般大公司在用，如QQ空间）。

### 增量原理（图）

![差分包原理图](http://upload-images.jianshu.io/upload_images/643851-3a5a218a1c04ab11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

bsdiff 进行文件内容比较，并且使用了bzip2进行文件压缩 ， 所有得出的差分包可能比理论值要小 ， 进一步可以减少用户流量 。增量更新 ， 较为关键的部分就是生成差分包 ， 将新旧APK进行比较 ， 生成一个新的文件 。

 ### 生产资源及工具
bsdiff --- [bsdiff （win环境） ](http://sites.inka.de/tesla/download/bsdiff4.3-win32-src.zip)  生成差分包及合并差分包库 ， 源码内已包含bzip2
bzip2  --- [bzip2](http://www.bzip.org/downloads.html) bsdiff 依赖
服务器 --- [Tomcat 7.0 ](http://tomcat.apache.org/tomcat-7.0-doc/index.html) （模拟网络环境）放置差分包 ， 供APP下载
开发工具 --- Eclipse NDK开发 ， 目前建议使用Eclipse开发
开发工具 --- VS  因为服务器搭建在windows平台 ， 所以使用VS开发JNI，在JNI系列中有使用 ， 不了解的可以前去了解 [JNI开发系列①JNI概念及开发流程](http://www.jianshu.com/p/68bca86a84ce)

### 编写bsdiff JNI
当我们拿到一个C语言库 ， 首先我该怎么做 ?  

> 首其一 ， 了解其用法 ， 找main函数

下载完bsdiff之后 ， 我们看到如下目录：

![bsdiff win32](http://upload-images.jianshu.io/upload_images/643851-5f8a8d3811cfa95d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看这么多文件 ， 还有一些乱七八糟的不知道什么的文件 ， 那么我们只关注 ， 我们知道的文件 ， 将C/C++源文件以及.h头文件，找出来 ，放到一个干净的文件夹中 。

 创建一个VS项目 ， 将源文件都放入到VS项目目录下 ， 添加现有项目 ， 将源码与项目关联 ， 参考：[JNI开发系列①JNI概念及开发流程](http://www.jianshu.com/p/68bca86a84ce)

__找main函数__

![search main func](http://upload-images.jianshu.io/upload_images/643851-ccabbd4c4eedac3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在`bsdiff.cpp`文件找到带参数的`main`函数 ， 并且有一个关于用法的线索 ， 那就是：

```c
if(argc!=4) errx(1,"usage: %s oldfile newfile patchfile\n",argv[0]);
```

我们可以根据这句话来推测 ， 需要四个参数 ， 并且三个参数必须传入的文件路径 ， 还有一个不知晓 ， 暂且不管它 。

> 其二 ， 创建JNI方法 ， 修改main函数

既然知道了需要传入的参数 ， 那么就可以创建一个Java工程 ， 编写JNI方法了。

```java
/**
 * 生成差分包
 * @param oldFilePath 老版本文件路径
 * @param newFilePath 新版本文件路径
 * @param patchFilePath 生成差分包文件路径
 */
public static native void diff(String oldFilePath , String newFilePath , String patchFilePath) ;
```
使用`javah`生成头文件 ， 引入到bsdiff.cpp文件中，编写调用的C函数 ， 并修改`main`函数名称，`main`函数作为入口函数 ， 在JNI中就适用了 ， 所有将`main`函数名称改一下 ， 在JNI的C函数中调用即可 。

```c
/*文件差分*/
JNIEXPORT void JNICALL Java_com_zeno_bsdiff_BsdiffUtils_diff
(JNIEnv *env, jclass clazz, jstring joldFilePath, jstring jnewFilePath, jstring jpatchFilePath) {

	char* oldFilePath = (char*)env->GetStringUTFChars(joldFilePath, 0);
	char* newFilePath = (char*)env->GetStringUTFChars(jnewFilePath, 0);
	char* patchFilePath = (char*)env->GetStringUTFChars(jpatchFilePath, 0);

	int argc = 4; 

	char* argv[4];
	// 无效参数 , 可以为任意字符
	argv[0] = "bsdiff";
	// 老版APK文件路径
	argv[1] = oldFilePath;
	// 新版APK文件路径
	argv[2] = newFilePath;
	// 生成的差分包文件路径
	argv[3] = patchFilePath;

	// 调用bsdiff
	bsdiff_main(argc, argv);

}
```
一切修改完毕 ， 别忘了在项目属性配置类型，改为生成`.dll`动态库 。编译生成 ， 这时候会出现很多错误 ， 举几个我遇到的 。

一 ， 使用了非安全的函数 ， 在文件中声明`#define _CRT_SECURE_NO_WARNINGS`即可 。

二 ， 在VS中通不过安全语法检查 ， 在VS中进行如下设置：

![VS setting](http://upload-images.jianshu.io/upload_images/643851-c140f1c2a9d76859.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
还需要在文件中添加`#define _CRT_NONSTDC_NO_DEPRECATE`预处理指令。也可以在项目属性中配置：

![VS Config](http://upload-images.jianshu.io/upload_images/643851-922fce342dcb2671.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

三 ， 变量未初始化
```c
	u_char *old = nullptr,*_new = nullptr;
	off_t oldsize,newsize;
	off_t *I,*V = nullptr;
```
可能会是这几个 ， 将其赋值为`nullptr`就可以了。 如果还有其他错误 ， 可执行分析 ， google ， 也欢迎评论留言 ， 多多交流 。

> 其三 ， 生成.dll动态库 ， 并使用

将生成的`.dll`动态库 ， 赋值到服务器项目的目录下 ， 或是Java项目也可以 。

差分包工具类：

```java
public class BsdiffUtils {
			
		/**
		 * 生成差分包
		 * @param oldFilePath 老版本文件路径
		 * @param newFilePath 新版本文件路径
		 * @param patchFilePath 生成差分包文件路径
		 */
		public static native void diff(String oldFilePath , String newFilePath , String patchFilePath) ;
		
		
		static {
			System.loadLibrary("BsdiffJNI");
		}
}
```

使用差分包工具类：
```java
public class BsdiffApk {
	
	public static void main(String args[]) {
		
		/**
		 * 文件差分
		 */
		BsdiffUtils.diff(Constract.OLD_FILE_PATH, Constract.NEW_FILE_PATH, Constract.PATCH_FILE_PATH);
	}
}
```

常量类：
```java
public class Constract {

	public static final String OLD_FILE_PATH = "E:/javaee_workspace/AppUpdateServer/apk/app-old.apk" ;
	public static final String NEW_FILE_PATH = "E:/javaee_workspace/AppUpdateServer/apk/app-new.apk" ;
	public static final String PATCH_FILE_PATH = "E:/javaee_workspace/AppUpdateServer/apk/App_patch.patch" ;
}
```

生成差分包：

![差分包](http://upload-images.jianshu.io/upload_images/643851-aed794d00c605502.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 结语
使用第三方库的时候 ， 自己编写的代码比较少 ， 主要是学会怎样使用第三方的库，以及编译生成动态库的时候 ， 需要查错找错并修改  ， 下篇 ， 我们将实现 ， 在客户端合并差分包 。
