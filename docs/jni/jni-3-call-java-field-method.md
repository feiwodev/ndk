### 前情提要
在前面 ， 我们已经熟悉了JNI的开发流程 ， .h头文件的分析 ， 生成头文件`javah`命令 ， 以及java类型在C语言中的表现形式 ， 值得注意的是 ， java中的所有引用类型都是`jobject`类型  ， `native`生成的函数 ， 以`Java_全类名_方法名`表示，包名的`.`以`_`表示 。

### 概述
在开篇的时候 ，我们就使用java的`native`方法调用过C函数 ， 返回了一个String类型的字符串 ， 使用`(*Env)->NewStringUTF(Env, "Jni C String");`函数 ， 我们将字符指针转换成`jstring` ， java类型的字符串返回给了我们的java层 。今天我们来学习 ， 使用C语言来调用Java的字段与方法 。

### part 1 :  C 函数访问java字段
>一 ， 定义Java 的`String`类型字段与修改字段的`native`方法

```java
// 使用C语言修改java字段
public String name = "zeno" ;

// C语言修改java String 类型字段本地方法
public native void accessJavaStringField() ;

// 调用
HelloJni jni = new HelloJni() ;
System.out.println("修改前 name 的值："+jni.name);
//C语言修改java字段本地方法
jni.accessJavaStringField();
System.out.println("修改后 name 的值："+jni.name);
```

> 二 ， 在C语言头文件中定义`native`方法的实现函数 ， 并实现

```c
// com.zeno.jni_HelloJNI.h
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJni_accessJavaStringField
(JNIEnv *, jobject);

// Hello_JNI.c

/*C语言访问java String类型字段*/
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJni_accessJavaStringField
(JNIEnv *env, jobject jobj) {

	// 得到jclass
	jclass jcls = (*env)->GetObjectClass(env, jobj); 

	// 得到字段ID
	jfieldID jfID = (*env)->GetFieldID(env, jcls, "name", "Ljava/lang/String;");

	// 得到字段的值
	jstring jstr = (*env)->GetObjectField(env, jobj, jfID);

	// 将jstring类型转换成字符指针
	char* cstr = (*env)->GetStringUTFChars(env, jstr, JNI_FALSE);
	//printf("is vaule：%s\n", cstr);
	// 拼接字符
	char text[30] = "  xiaojiu and ";
	strcat(text, cstr);
	//printf("modify value %s\n", text);

	// 将字符指针转换成jstring类型
	jstring new_str = (*env)->NewStringUTF(env, text);

	// 将jstring类型的变量 ， 设置到java 字段中
	(*env)->SetObjectField(env, jobj, jfID, new_str);
}
```
> 三 ， 输出

```java
修改前 name 的值：zeno
修改后 name 的值：  xiaojiu and zeno
```

> 四 ， 分析


首先来分析C函数：
```c
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJni_accessJavaStringField
(JNIEnv *env, jobject jobj)

JNIEXPORT jstring JNICALL Java_com_zeno_jni_HelloJni_getStringFromC
(JNIEnv *Env, jclass jclazz)
```

我们可以看出两处不同 ， 一处是返回值类型 ， 一处是函数参数类型 ，返回值类型没什么可讲的 ， 在分析.h头文件的时候 ， 已经详细讲述了 。那 ， 这两个函数参数类型`jobject`与`jclass`有什么区别呢 ？ 这两个类型表示 ， Java的`native`函数 ， 是成员方法还是类方法 ， 成员方法需要对象.方法名 ， 类方法则类名.方法名 ， 可以在main方法里面直接使用 。

接下来是：
```c
// 得到jclass jclass就好比java的.class对象
jclass jcls = (*env)->GetObjectClass(env, jobj);
```
为什么要得到jclass呢 ？
因为 ，我们要获取字段ID ， 在JNI中 ， 获取java字段与方法都需要签名。而签名是在类加载的时候完成 ， 所以在获取字段ID的时候需要传入jclass 。

// 得到字段ID 
```c
jfieldID jfID = (*env)->GetFieldID(env, jcls, "name", "Ljava/lang/String;");
```
通过传入jclass , 字段名称 ， 字段签名 ， 就可以得到字段ID ，也可使用GetMethodID函数得到方法ID 。

为什么传入了字段名称，还需要签名呢 ？
因为java支持重载 ， 一个方法名称可以有多个不同实现 ， 根据传入的参数不同 ，所以C语言调用函数为了区分不同的方法， 而对每个方法做了签名 ， 而字段则可用来标识类型 (仅个人理解)。

获取字段与函数签名的方式：
```
在.class的文件目录下 ，使用`javap -s -p className`   就可以列举出 ， 所有的字段与方法签名
```

```c
// 得到字段的值 ， 类比java中的 对象.字段名得到值 ， 这里是字段的ID
jstring jstr = (*env)->GetObjectField(env, jobj, jfID);

// 将jstring类型转换成字符指针 
char* cstr = (*env)->GetStringUTFChars(env, jstr, JNI_FALSE); 
//printf("is vaule：%s\n", cstr); 
// 拼接字符 char text[30] = " xiaojiu and "; strcat(text, cstr);
// 将字符指针转换成jstring类型 
jstring new_str = (*env)->NewStringUTF(env, text); 
```
因为java类型与C语言类型不是相通的 ， 所有需要一个转换 ， 类型的介绍在上一篇已经详细说明 。

```c
// 将jstring类型的变量 ， 设置到java 字段中 
// 类比java中的 对象.字段名得到值 ， 这里是字段的ID
(*env)->SetObjectField(env, jobj, jfID, new_str);
```


> 画龙点睛：
上述中 ， 我们访问修改了String类型的字段 ， 也基本上能看出访问字段的基本套路 ， 首先得到jclass , 再得到字段ID ， 继而得到字段的值 ，  进行类型转换 ， 最后将变化的值设置给Java字段 。由此可以推出 ， 访问其他类型的字段 ， 也是这样的套路  ， 只不过类型变了 ， 值得注意的是 ， java中的引用类型是需要进行类型转换的 。


### part 2 ： C函数访问Java方法

> 一 ，  定义Java 方法与调用方法的native方法

```java
// C语言调用java方法
private native void accessJavaRandomNumberMethod() ;

// 调用
jni.accessJavaRandomNumberMethod() ;
```

> 二 ，  在C语言头文件中定义native方法的实现函数 ， 并实现

```c

// com.zeno.jni_HelloJNI.h
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJni_accessJavaRandomNumberMethod
(JNIEnv *, jobject);


// Hello_JNI.c

// 访问java方法
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJni_accessJavaRandomNumberMethod
(JNIEnv *env, jobject jobj) {

	// 得到jclass
	jclass jclazz = (*env)->GetObjectClass(env, jobj);
	// 得到方法ID
	jmethodID jmtdId = (*env)->GetMethodID(env, jclazz, "getRandomNumber", "(I)I");

	// 调用方法
	jint jRandomNum = (*env)->CallIntMethod(env, jobj, jmtdId, 10);

	// 打印
	printf("得到java方法的随机数：%ld\n", jRandomNum);

}
```

> 三 ， 输出

```java
得到java方法的随机数：6
```

> 四 ， 分析

```
不论是字段访问还是方法的调用 ， 其基本的套路不变 ，调用Java方法与调用字段不同的是 ，
将得到字段ID改成得到方法ID ， 得到字段的值改成调用方法`CallXXX` ， 通过调用调用Java方法得到方法返回值 。
```

### part 3 ： C函数访问Java字段与方法（静态）
> 套路都是一样的 ， 这里仅给出代码 ， 不做详细分析（本阶段全部代码） 。

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
	
	// 使用C语言修改java字段
	public String name = "zeno" ;
	
	// 使用C语言修改java int 类型字段
	private int age = 20 ;
	
	public static String flag = "flag1" ;
	

	// 调用C语言函数方法
	public static native String getStringFromC() ;
	// 调用C++语言函数方法
	public static native String getStringFromCPP() ;
	
	// C语言修改java String 类型字段本地方法
	public native void accessJavaStringField() ;
	
	// C语言修改java String static 类型字段本地方法
	public native void accessJavaStaticStringField() ;
	
	// C语言修改java int 类型字段本地方法
	public native void accessJavaIntField() ;
	
	
	
	// C语言调用java方法
	private native void accessJavaRandomNumberMethod() ;
	
	// 调用Java静态方法
	private native void accessJavaStaticMethod() ;
	
	// 静态native方法访问字段
	private static native void staticAccessJavaField() ;
	
	public static void main(String[] args) {
		
		System.out.println("getStringFormC == "+getStringFromC());
		System.out.println("getStringFormC == "+getStringFromCPP());
		
		HelloJni jni = new HelloJni() ;
		System.out.println("修改前 name 的值："+jni.name);
		//C语言修改java字段本地方法
		jni.accessJavaStringField();
		System.out.println("修改后 name 的值："+jni.name);
		
		System.out.println("修改前 flag 的值："+flag);
		//C语言修改java static 字段本地方法
		jni.accessJavaStaticStringField();
		
		System.out.println("修改后 flag 的值："+flag);
		
		
		System.out.println("修改前 age 的值："+jni.age);
		//C语言修改java字段本地方法
		jni.accessJavaIntField();
		
		System.out.println("修改后 age 的值："+jni.age);
		
		jni.accessJavaRandomNumberMethod() ;
		
		jni.accessJavaStaticMethod() ;
		
		// 静态native方法 ，访问java字段
		
		System.out.println("修改前 flag 的值："+flag);
		 staticAccessJavaField();
		 
		 System.out.println("修改后 flag 的值："+flag);
	}
	
	static{
		// 加载动态库
		System.loadLibrary("Hello_JNI") ;
	}
	
	
	// 调用java方法
	private int getRandomNumber(int bound) {	
		
		return new Random().nextInt(bound) ;
	}
	
	// 调用java静态方法
	private static String getUUID() {
		return UUID.randomUUID().toString();
	}
}
```

> C实现 ， 这里就不贴头文件了 。
调用静态的Java字段与方法 ， 在C语言中调用相应的static函数 ， 例如：获取静态字段ID `GetStaticFieldID`  。

```c
#define _CRT_SECURE_NO_WARNINGS

#include "com_zeno_jni_HelloJni.h"

#include <string.h>
#include <stdio.h>
#include <stdlib.h>

/*
	C/C++动态库 ， 在win平台下以.dll文件标识 ， 在linux下面以.so文件表示
	在Android中 ， 以.so文件表示 ， 因为Android使用的是linux内核 。

*/



/*
* Class:     com_zeno_jni_HelloJni
* Method:    getStringFormC
* Signature: ()Ljava/lang/String;
*/
JNIEXPORT jstring JNICALL Java_com_zeno_jni_HelloJni_getStringFromC
(JNIEnv *Env, jclass jclazz) {

	return (*Env)->NewStringUTF(Env, "Jni C String");
}

/*C语言访问java String类型字段*/
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJni_accessJavaStringField
(JNIEnv *env, jobject jobj) {

	// 得到jclass , jclass就好比java的.class对象
	jclass jcls = (*env)->GetObjectClass(env, jobj); 

	// 得到字段ID ， 
	jfieldID jfID = (*env)->GetFieldID(env, jcls, "name", "Ljava/lang/String;");

	// 得到字段的值
	jstring jstr = (*env)->GetObjectField(env, jobj, jfID);

	// 将jstring类型转换成字符指针
	char* cstr = (*env)->GetStringUTFChars(env, jstr, JNI_FALSE);
	//printf("is vaule：%s\n", cstr);
	// 拼接字符
	char text[30] = "  xiaojiu and ";
	strcat(text, cstr);
	//printf("modify value %s\n", text);

	// 将字符指针转换成jstring类型
	jstring new_str = (*env)->NewStringUTF(env, text);

	// 将jstring类型的变量 ， 设置到java 字段中
	(*env)->SetObjectField(env, jobj, jfID, new_str);
}

/*C语言访问java int 类型字段*/
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJni_accessJavaIntField
(JNIEnv *env, jobject jobj) {

	// 得到jclass
	jclass jclazz = (*env)->GetObjectClass(env, jobj);

	// 得到字段ID
	jfieldID jfid = (*env)->GetFieldID(env, jclazz, "age", "I");

	// 得到字段值
	jint jAge = (*env)->GetIntField(env, jobj, jfid);

	jAge++;

	(*env)->SetIntField(env, jobj, jfid, jAge);

}

// 访问java方法
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJni_accessJavaRandomNumberMethod
(JNIEnv *env, jobject jobj) {

	// 得到jclass
	jclass jclazz = (*env)->GetObjectClass(env, jobj);
	// 得到方法ID
	jmethodID jmtdId = (*env)->GetMethodID(env, jclazz, "getRandomNumber", "(I)I");

	// 调用方法
	jint jRandomNum = (*env)->CallIntMethod(env, jobj, jmtdId, 10);

	// 打印
	printf("得到java方法的随机数：%ld\n", jRandomNum);

}


// 访问java静态字段
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJni_accessJavaStaticStringField
(JNIEnv *env, jobject jobj) {

	// 得到jclass
	jclass jclazz = (*env)->GetObjectClass(env, jobj);
	// 得到字段ID
	jfieldID jfid = (*env)->GetStaticFieldID(env, jclazz, "flag", "Ljava/lang/String;");

	// 得到字段的值
	jobject jFLagStr = (*env)->GetStaticObjectField(env, jclazz, jfid);
	
	// 将java字符串转换成C字符指针
	char* cFlagStr = (*env)->GetStringUTFChars(env, jFLagStr, JNI_FALSE);

	//printf("is vaule：%s\n", cFlagStr);

	char newStr[30] = " access static field ";
	strcat(newStr, cFlagStr);

	// 将C字符指针 ， 转换成java字符串
	jstring jNewStr = (*env)->NewStringUTF(env, newStr);

	// 将字符串设置到java字段上
	(*env)->SetStaticObjectField(env, jclazz, jfid, jNewStr);
}

// 访问java静态方法
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJni_accessJavaStaticMethod
(JNIEnv *env, jobject jobj) {

	// 得到jclass
	jclass jclazz = (*env)->GetObjectClass(env, jobj);

	// 得到静态方法ID
	jmethodID mtdid = (*env)->GetStaticMethodID(env, jclazz, "getUUID", "()Ljava/lang/String;");

	// 调用静态方法
	jobject jUUIDStr = (*env)->CallStaticObjectMethod(env, jclazz, mtdid);

	// 将java字符串转换成C字符指针
	char* cUUIDStr = (*env)->GetStringUTFChars(env, jUUIDStr, JNI_FALSE);

	printf("is vaule：%s\n", cUUIDStr);

	// 根据UUID生成临时文件
	char file_path[100] ;
	sprintf(file_path, "e:\\dn\\%s.txt", cUUIDStr);
	printf("is address：%s\n", file_path);

	FILE* fp = fopen(file_path, "w");
	if (fp == NULL) {
		printf("文件创建失败\n");
	}

	char* content = "落花有意流水无情";
	// 写入内容
	fputs(content, fp);

	// 关闭流
	fclose(fp);
}



// 静态native方法 ， 访问java字段
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJni_staticAccessJavaField
(JNIEnv *env, jclass jclazz) {

	// 得到字段ID
	jfieldID jfid = (*env)->GetStaticFieldID(env, jclazz, "flag", "Ljava/lang/String;");

	// 得到字段的值
	jstring jflag = (*env)->GetStaticObjectField(env, jclazz, jfid);

	// 将java字符串转换成字符指针
	char* cXj = (*env)->GetStringUTFChars(env, jflag, JNI_FALSE);
	printf("is value：%s\n", cflag);
	char newName[100] = "xiaojiu love ";
	char* cNewName = strcat(newName, cflag);

	// 将字符指针转换成java类型
	jstring newStr = (*env)->NewStringUTF(env, cNewName);

	// 设置
	(*env)->SetStaticObjectField(env, jclazz, jfid, newStr);

}
```

### 编写套路
C语言访问Java语言的字段与方法 ， 只要理解了一种 ， 其他的都是套路 ， 根据步骤一步一步来就可以了 。

> 步骤一 、 得到`jclass` ， 字节码对象 ， 如果是`static native`修饰 ， 则函数会以`jclass`类型传入 ， 非静态则需要得到`jclass`类型 。

```c
// 得到jclass 
jclass jclazz = (*env)->GetObjectClass(env, jobj);
```

> 步骤二 、得到字段或方法ID , 区分静态字段与对象字段 ， 静态字段或方法调用`(*env)->GetStaticFieldID`得到静态字段ID ，`(*env)->GetStaticMethodID`得到静态方法ID ， 对象字段调用`(*env)->GetFieldID`得到字段ID，`(*env)->GetMethodID`得到方法ID 。 可以得到一个套路 ， 静态修饰的 ， 则调用`static`标识的函数 ， 非静态的则调用常规函数 。

```c
// 得到字段ID ， 对象字段
 jfieldID jfid = (*env)->GetFieldID(env, jclazz, "age", "I");

// 得到字段ID ， 静态字段
jfieldID jfid = (*env)->GetStaticFieldID(env, jclazz, "flag", "Ljava/lang/String;");
```

> 步骤三 、 取得字段的值或调用方法 , 需要注意的是， 得到字段的值与调用方法 ， 都有类型的区分 。引用类型则使用`GetObjectField` ， `CallStaticObjectMethod` ， 其他类型 ， 则有对于的jxxx类型对应 。套路简写：`Get<Type>Field` ， `GetStatic<Type>Field` ， `Call<Type>Method` ， `CallStatic<Type>Method`  。

```c
// 得到字段的值 
jstring jstr = (*env)->GetObjectField(env, jobj, jfID);

// 得到字段值
 jint jAge = (*env)->GetIntField(env, jobj, jfid);

// 调用方法 
jint jRandomNum = (*env)->CallIntMethod(env, jobj, jmtdId, 10);

// 调用静态方法
 jobject jUUIDStr = (*env)->CallStaticObjectMethod(env, jclazz, mtdid);
```

> 步骤四 、 类型转换 ， 如果是Java引用类型 ， 则需要进行类型转换

```
// 将java字符串转换成字符指针
char* cflag = (*env)->GetStringUTFChars(env, jflag, JNI_FALSE);
```

### 结语
真正的高手 ， 不是乐而学得的 ， 真正的学习 ， 不是轻轻松松的 。高手 ， 需要刻意练习 ， 刻意练习不是重复相同的动作 ， 而是跳出舒适区熟悉区域 ， 刻意练习自己不熟悉感觉艰难的事情 。

参考资料：
[Java Native Interface 6.0 Specification](http://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/jniTOC.html)
