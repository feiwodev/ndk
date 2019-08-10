### 前情提要
上个系列，我们学习了Java与C/C++的交互 ， 使用Java调用C/C++函数，使用C/C++调Java的方法和创建Java对象等等 。在上个系列中 ， 我们使用的是Eclipse与VS进行的开发 ，  在NDK系列中 ， 我们将采用最新的Android Studio进行开发 ， 版本是`Android studio 2.2 RC 2` ， NDK版本采用的是最新的`r12b` 。

### 开发环境
工具下载地址 （win）（需要科学上网） : 
Android Studio 2.2 RC2 --- [Android Studio Download 2.2 RC2](https://dl.google.com/dl/android/studio/ide-zips/2.2.0.11/android-studio-ide-145.3253452-windows.zip)
Android NDK r12b --- [Android NDK r12b 64bit](https://dl.google.com/android/repository/android-ndk-r12b-windows-x86_64.zip)
Android NDK r12b --- [Android NDK r12b 32bit](https://dl.google.com/android/repository/android-ndk-r12b-windows-x86.zip)
> 国内镜像站：
[androiddevtools](http://www.androiddevtools.cn/)

### 关于开发环境的说明
因为在`Android studio 2.2`之前的版本 ， 对C/C++支持不是很好 ， 也没有语法提示 ， 写起来不是很方便 ， 构建工具也不是很完整 ， 所有采用最新的`Android studio 2.2 RC2`来进行编写 ，但 ， `Android studio 2.2 RC2` 还是Beta版本 ，所有 ， 不建议现在应用到生产环境中 ， 等google发布了Stable版本之后再应用 。 目前建议 ， 可以使用`Eclipse`编写`.so` ， 然后应用到现在的生产环境中 。

### 创建NDK项目

> 第一步 ， 创建支持C++的项目

![C++ support](http://upload-images.jianshu.io/upload_images/643851-b15dc1376c8c2430.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其他的选项使用默认的即可 。

> 第二步 ， 关联NDK

创建完成之后会报如下错误：

![ndk r12b](http://upload-images.jianshu.io/upload_images/643851-0b70deabda8b9346.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在项目配置中 ， 关联NDK之后就会ok

![config ndk](http://upload-images.jianshu.io/upload_images/643851-3fe9ba5c00088b07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 第三步 ， 编写native类及处理方法

在创建项目的时候 ， 勾选了C++ support ， 项目创建完成之后 ， 会自动帮我们生成一个`cpp/native-lib.cpp`

![auto create cpp file](http://upload-images.jianshu.io/upload_images/643851-e5a5ebc788106c83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你可以不用修改文件名 ， 在新建`native`方法的时候 ， 会提示你创建一个C++的JNI函数 ， 直接创建就会生成一个JNI函数 ， 都不用使用`javah`生成一个头文件 ， 然后再引入头文件了 ， 非常之方便 。

![auto create jni function 1](http://upload-images.jianshu.io/upload_images/643851-91231d255bd06de6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

创建函数：

![auto create jni function 2](http://upload-images.jianshu.io/upload_images/643851-a0bbc2ac186a1601.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里 ， 就不使用默认的`.cpp`文件了 ， 我们新建一个`.c`文件 ， 创建了`HelloNDK.c`文件之后 ， Android Studio会提示我们 ， 需要在`Android.mk/CMakeLists.txt`中进行声明 ， 这里 ， 我们使用默认的`CMakeLists.txt`建构工具 （创建项目的时候自动生成）。

> 第四步 ， build.gradle配置：

```java
externalNativeBuild {  
  cmake {       
     cppFlags ""       
     // 指定只用clang编译器        
    // Clang是一个C语言、Objective-C、C++语言的轻量级编译器。  
    arguments "-DANDROID_TOOLCHAIN=clang"        
    // 生成.so库的目标平台        
    abiFilters "armeabi-v7a" , "armeabi"    
    }
}

// 配置CMakeLists.txt路径
externalNativeBuild { 
   cmake {        
      path "CMakeLists.txt"   
   }
}
```

> 第五步 ， 修改CMakeLists.txt

```
add_library( # Sets the name of the library.
             HelloNDK  # 生成的.so库文件名称

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             # Associated headers in the same location as their source
             # file are automatically included.
             # 需要生成的.so库的文件路径
             src/main/cpp/HelloNDK.c
 )

target_link_libraries( # Specifies the target library.
                       # 项目链接的.so库名称
                       HelloNDK

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

> 第六步 ， 编写`native`方法，以及C函数 

```java
/**
 * Created by Zeno on 2016/9/10.
 *
 * NDK Demo
 */

public class HelloNDK {

    public static native String sayHelloNDK() ;

    static {
        System.loadLibrary("HelloNDK");
    }
}

```

```c
#include <jni.h>
JNIEXPORT jstring JNICALLJava_com_zeno_encryptanddecrypt_ndk_HelloNDK_sayHelloNDK(JNIEnv *env, jclass thiz) {   
     // TODO    
     return (*env)->NewStringUTF(env, "this String come from C ");
}
```

`native`方法的编写以及C函数的写法， 我们都非常熟悉了 ， 这里就不再解释各自的意义了 。

> 第七步 ， 编译

![make](http://upload-images.jianshu.io/upload_images/643851-f8174b491f430b00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

编译完成之后 ，我们可以切换到`project`视图，来查看`.so`文件

![make success](http://upload-images.jianshu.io/upload_images/643851-187240a411969045.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 第八步 ， 运行

![run](http://upload-images.jianshu.io/upload_images/643851-a838074a544d9331.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果使用的是genymotion模拟器 ， 这需要在`abiFilters`加入`x86` ，不然项目会运行不起来的 。 当然， 也可以使用一个genymotion的`arm`插件 ， 这样不配置`x86`也可以运行 。

```
// 生成.so库的目标平台
abiFilters "armeabi-v7a" , "armeabi" , "x86"
```

### 结语
做为Android开发者 ， 从最开始的Eclipse开发工具 ， 到现在日渐成熟的Android Studio ， 还有几乎可以看得见成长的Android System ， 我很庆幸 ， 从一开始就选择了Android平台 ， 从初学Android到现在的日渐深入 ， Android在成长 ， 我也在成长 。见证了Android从一个丑小鸭变成了 ， 一个羽翼渐丰的白天鹅 ， 不论从操作系统的易用性和UI友好性 ， 它的成长都是有目共睹的 。感谢Android 。

> 每一个人的成长背后 ， 都离不开所有人支持 ， 我们是一个单独的个体 ， 又是社会网状结构的一个点 ， 我们既独立又互相联系 。

### 关于Android Studio 2.2
我很期待这个版本 ， 对于C/C++的支持非常友好 ， 省却了很多繁琐的操作 ， 比Eclipse好用的太多 。

### 参考
[Building C++ in Android Studio with CMake or ndk-build](http://tools.android.com/tech-docs/external-c-builds)
