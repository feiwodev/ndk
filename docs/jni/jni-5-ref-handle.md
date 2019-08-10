### 前情提要
在上一篇中 ， 我们了解到了 ， 创建一个Java对象的几个步骤：

> 第一，`findClass`找到需要创建对象的类（全类名）
第二，得到构造方法的ID，构造方法名称，统一使用`<init>`
第三，使用`NewObject`创建Java对象

当创建了这个类的对象之后 ， 我们就可以使用这个类里面所提供的方法了 ， 那么我们就可以在C中使用Java中其他对象的方法了 。

### 数组引用的处理
在Java中 ， 使用`new`关键字创建对象 ， 创建之后我们就可以随意使用这个对象  ， 我们无需关心这个对象是什么时候被回收的 ， 对象的回收已经托管到了JVM的GC ， 由GC来帮我们回收无引用的对象 ， 那么，我们使用JNI技术传递给C/C++的对像要怎么做处理呢 ？

将对象引用传递给C/C++时 ， C/C++层就会持有Java对象 ， 如果不进行妥善处理 ， 对象多了就会出现内存泄漏问题 ， 所有在C/C++层使用Java对象时 ， 需要释放这个引用 。

下面就来看看数组引用的处理：
```java
// 对数组进行排序
private native void useArraySort(int[] array) ;

public static void main(String[] args) {
         // Java数组在C中排序
         int[] array = {1,60,20,10,4,90,23} ;
         jni.useArraySort(array);
         // 输出
	     for (int i = 0; i < array.length; i++) {
			System.out.println("array == "+array[i]);
		}
}
```

将`array`对象传递给C ， C中的变量将持有`array`这个引用

```c

// sort
int compare(int* a, int* b) {

	return (*a) - (*b);
}

/*对数组进行排序*/
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJNI_useArraySort
(JNIEnv *env, jobject jobj , jintArray jarray) {

	jint* arrayElemts = (*env)->GetIntArrayElements(env, jarray, NULL);

	jsize arraySize = (*env)->GetArrayLength(env, jarray);

	qsort(arrayElemts,arraySize,sizeof(jint),compare);

	// 释放引用 ， 因为数组和对象在java中都是引用 ， 都会在堆内存中开辟一块空间 ， 但我们使用完对象之后
	// 需要将引用释放掉 ， 不然会很耗内存 ， 在一定程度上可能会造成内存溢出 。
	//JNI_ABORT, Java数组不进行更新，但是释放C/C++数组
	//JNI_COMMIT，Java数组进行更新，不释放C/C++数组（函数执行完，数组还是会释放）
	(*env)->ReleaseIntArrayElements(env, jarray, arrayElemts, JNI_COMMIT);

}
```
内存示意图：

![Java对象引用和C](http://upload-images.jianshu.io/upload_images/643851-12f6bb4ff09eacbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

只要是Java对象 ， 在C中都需要释放，如String类型引用：
```c
// String类型引用释放
void (JNICALL *ReleaseStringUTFChars)
      (JNIEnv *env, jstring str, const char* chars);

```
在C中创建的对象引用也需要进行引用释放
```c
/*返回int类型的数组*/
JNIEXPORT jintArray JNICALL Java_com_zeno_jni_HelloJNI_getIntArray
(JNIEnv *env, jobject jobj,jint len) {

	// 创建一个jint类型的数组
	jintArray jArray = (*env)->NewIntArray(env, len);

	// 得到数组首个元素指针
	jint* arrayElements = (*env)->GetIntArrayElements(env, jArray, NULL);

	// 指针运算
	int i = 0;
	for (; i < len; i++)
	{
		arrayElements[i] = i;
	}

	// 同步
	(*env)->ReleaseIntArrayElements(env, jArray, arrayElements, JNI_COMMIT);

	return jArray;
}

java code
// 在C中生存数组 ， 返回到Java中
private native int[] getIntArray(int len) ;

int[] intArray = jni.getIntArray(20);
	for (int i = 0; i < intArray.length; i++) {
		System.out.println("int array === "+intArray[i]);
}

```
> 为什么在C中创建的对象也需要释放 ？ 
在上述代码中 ， 创建一个数组对象 ， 并将引用传递给了Java层 ， 将引用交给了Java之后 ， C就需要释放这个引用 ， 不然会一直持有 ， GC也不会回收这个对象 。

### 引用的分级
在Java中引用也有强弱之分 ， 使用`new`创建的对象就是强引用，也可以使用`WeakReference`将对象包装成一个弱引用对象 。在C中也不列外 ， C中也有一套`全局引用`，`局部引用`，`弱全局引用`等等 。

> 一 ， 局部引用

```c
// 局部引用
// 作用：C使用到或自行创建Java对象，需要告知虚拟机在合适的时候回收对象

//局部引用，通过DeleteLocalRef手动释放对象
//1.访问一个很大的java对象，使用完之后，还要进行复杂的耗时操作
//2.创建了大量的局部引用，占用了太多的内存，而且这些局部引用跟后面的操作没有关联性

JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJNI_localRef
(JNIEnv *env, jobject jobj) {
	
	// 找到类
	jclass dateClass = (*env)->FindClass(env, "java/util/Date");

	// 得到构造方法ID
	jmethodID dateConstructorId = (*env)->GetMethodID(env, dateClass, "<init>", "()V");

	// 创建Date对象
	jobject dateObject = (*env)->NewObject(env, dateClass, dateConstructorId);

	// 创建一个局部引用
	jobject dateLocalRef = (*env)->NewLocalRef(env, dateObject);

	// 省略N行代码

	// 不再使用对象 ， 则通知GC回收对象
	(*env)->DeleteLocalRef(env, dateLocalRef);
	// 因为dateObject也是局部对象，可以直接回收dateObject对象
	//(*env)->DeleteLocalRef(env, dateObject);

}
```

> 全局引用

```c

// 全局引用
// 定义全局引用
//共享(可以跨多个线程)，手动控制内存使用
jstring globalStr;

/*创建全局引用*/
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJNI_createGlobalRef
(JNIEnv *env, jobject jobj) {

	jstring jStr = (*env)->NewStringUTF(env, "I want your love !");

	// 创建一个全局引用
	globalStr = (*env)->NewGlobalRef(env, jStr);
}

/*使用全局引用*/
JNIEXPORT jstring JNICALL Java_com_zeno_jni_HelloJNI_useGlobalRef
(JNIEnv *env, jobject jobj) {

	return globalStr;

}

/*释放全局引用*/
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJNI_deleteGlobalRef
(JNIEnv *env, jobject jobj) {

	// 释放全局引用
	(*env)->DeleteGlobalRef(env, globalStr);

}

//弱全局引用
//节省内存，在内存不足时可以是释放所引用的对象
//可以引用一个不常用的对象，如果为NULL，临时创建
//创建：NewWeakGlobalRef,销毁：DeleteGlobalWeakRef
```

> 引用缓存

```c
/*变量缓存*/
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJNI_variableCach
(JNIEnv *env, jobject jobj) {

	// 找到String类
	jclass stringClass = (*env)->FindClass(env, "java/lang/String");

	// 得到构造方法ID
	jmethodID stringConstructorID = (*env)->GetMethodID(env, stringClass, "<init>", "()V");

	// 创建String类
	//// 缓存局部变量 ， 只创建一次 ， 关键字static
	static jobject stringObject = NULL;
	if (stringObject == NULL)
	{
		stringObject = (*env)->NewObject(env, stringClass, stringConstructorID);

		printf("------------- create String object --------------\n");
	}

	/*jobject stringObject = (*env)->NewObject(env, stringClass, stringConstructorID);

	printf("------------- create String object --------------\n");*/

}
```

引用的分级 ， 上述代码都有比较详细的注释 ，这里就不多加解释了 ， 说一下全局引用的简单使用场景。

在开发中 ， 我们常常需要初始化一些变量 ， 进行全局使用 ， 这里我们的全局引用就发挥了作用了 。

```c
// 初始化全局变量
//初始化全局变量，动态库加载完成之后，立刻缓存起来
jstring initGlobalStr;

JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJNI_initVariable
(JNIEnv *env, jclass jcls) {

	jstring initStr = (*env)->NewStringUTF(env, "create global init variable ");

	initGlobalStr = (*env)->NewGlobalRef(env, initStr);
}

/*访问初始化全局变量*/
JNIEXPORT jstring JNICALL Java_com_zeno_jni_HelloJNI_accessInitGlobalVariable
(JNIEnv *env, jobject jobj) {
	
	return initGlobalStr;
}


__java code__ 

static{
        // 加载动态库
        System.loadLibrary("Hello_JNI") ;
        // 初始化全局变量
        initVariable();
    }
```
C中引用的分级和在Java中的类型 ， 都需要在合适的环境使用 。

### 结语
这是JNI系列的最后一篇 ， 在这个系列中 ， 我们大致了解了JNI的开发流程， 以及一些常用的API和技法 ， 在下篇中 ， 我们正式进入NDK开发 ， 将我们在JNI中学到的应用起来 。NDK基础过后 ， 将会进入到C++语言的学习 ， 我们要学会使用第三方C/C++库 。


### 参考
[global_and_local_references](http://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/functions.html#global_and_local_references)
