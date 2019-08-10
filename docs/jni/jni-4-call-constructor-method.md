### 前情提要

Java调用C方法很简单 ， 只需要编写`native`方法即可 ， 通过C去调用Java的字段与方法 ， 则需要比较复杂的操作 ， 上篇中介绍了 ， C调用的Java字段与方法的几个套路：

> 步骤一 、 得到jclass， 字节码对象 ， 如果是static native修饰 ， 则函数会以jclass类型传入 ， 非静态则需要得到jclass类型 。

>步骤二 、得到字段或方法ID , 区分静态字段与对象字段 ， 静态字段或方法调用(*env)->GetStaticFieldID，(*env)->GetMethodID函数得到ID ， 对象字段调用(*env)->GetFieldID，(*env)->GetStaticMethodID得到ID 。 可以得到一个套路 ， 静态修饰的 ， 则调用static标识的函数 ， 非静态的则调用常规函数 。

> 步骤三 、 取得字段的值或调用方法 , 需要注意的是， 得到字段的值与调用方法 ， 都有类型的区分 。引用类型则使用GetObjectField， CallStaticObjectMethod， 其他类型 ， 则有对于的jxxx类型对应 。套路简写：Get<Type>Field， GetStatic<Type>Field， Call<Type>Method， CallStatic<Type>Method 。

> 步骤四 、 类型转换 ， 如果是Java引用类型 ， 则需要进行类型转换

### C 调用Java对象的构造方法
> C调用Java的构造方法与调用普通方法略有不同 ， 其不同之处在于方法名称上 ， 普通方法直接使用`方法名` ， 构造方法则不是使用类名 ， 而使用一个固定写法`<init>` 。

```java
java code  使用C语言创建一个Date对象，获取时间戳

private native long accessConstructorMethod() ;

public static void main(String[] args) {
     long timeLong = jni.accessConstructorMethod() ;
		 
     SimpleDateFormat sdf = new SimpleDateFormat();
     sdf.applyPattern("yyyy-MM-dd  HH:mm:ss") ;
	 String dateString = sdf.format(new Date(timeLong)) ;
}
```
> 使用C创建Java对象 ， 并调用方法

```c
/*访问java对象的构造方法*/
JNIEXPORT jlong JNICALL Java_com_zeno_jni_HelloJni_accessConstructorMethod
(JNIEnv *env, jobject jobj) {

	// 找到Date的jclass
	jclass dateCls = (*env)->FindClass(env, "java/util/Date");

	// 得到构造方法id
	jmethodID dateConstructMid = (*env)->GetMethodID(env, dateCls, "<init>", "()V");

	// 创建Date对象
	jobject dateObj = (*env)->NewObject(env, dateCls, dateConstructMid);

	// 得到getTime方法ID
	jmethodID getTimeMid = (*env)->GetMethodID(env, dateCls, "getTime", "()J");

	// 调用getTime方法
	jlong timeLong = (*env)->CallLongMethod(env, dateObj, getTimeMid);

	printf("new Date().getTime : %lld\n", timeLong);

	return timeLong;
}
```
_Tips:_

C实例化Java对象的时候 ， 首先需要找到对象的class , 以全类名表示 ， `.`用`/`代替 （`java.util.Date  --> Java/util/Date`）。Java的构造函数名称为固定的`<init>`名称 ， 通过`NewObject()`函数来创建Java对象 。需要注意的是签名 ， 我们可以通过`javap -s -p`来获取签名 。常见签名如下：

|数据类型|签名|
|:---:|:---:|
|boolean|Z|
|byte|B|
|char|C|
|short|S|
|int|I|
|long|L|
|float|F|
|double|D|
|void|V|
|object|L开头，然后以/分隔的完整类型，后面再加`;`例如：String类型签名 `Ljava/lang/String`|
|Array|以[开头，再加上数组元素的签名。例如int[]的签名是 [I , int[][]是[[I|

### C与Java的字符转换

在开发中 ， 我们常常都会遇到字符编码问题 ， web开发的时候 ， 时常需要指定网页的编码 ， 国内一般常用GBK与UTF-8 ，这两种编码格式 。 我们在开发项目的时候 ， 也会指定项目的文件编码 ， 因为不同文件编码对于汉字的支持与处理不同 ， GBK采用的是一字节和双字节编码 ， 而UTF-8则采用的是 ， 一至四个字节编码 ， 单个字符使用一个直接表示 ， 汉字则使用四个字节表示 。

```java
java code

private native String cTransformChar(String str) ;

public static void main(String[] args) {
    HelloJNI jni = new HelloJNI() ;
    System.out.println(jni.cTransformChar("住"));
}

```

> C处理字符编码

```c
/*C的字符转换*/

JNIEXPORT jstring JNICALL Java_com_zeno_jni_HelloJNI_cTransformChar
(JNIEnv *env, jobject jobj,jstring jStr) {

	// 将jstring转换成c字符指针
	char *cStr = (*env)->GetStringUTFChars(env, jStr, NULL);

	jsize js = (*env)->GetStringLength(env, jStr);


	printf("传进来的值size ：  %d\n", js);
	printf("传进来的值 ：  %s\n", cStr);

	char destBuffer[50] = "非我 and 小九";

	// 得到String类
	jclass stringCls = (*env)->FindClass(env, "java/lang/String");

	// 得到构造方法的ID
	jmethodID stringConstructMid = (*env)->GetMethodID(env, stringCls, "<init>", "([BLjava/lang/String;)V");

	/*
    使用到的Java构造方法
	String(byte[] bytes, String charsetName) 
     通过使用指定的 charset 解码指定的 byte 数组，构造一个新的 String。
	*/

	// 构建一个bytes数组
	jbyteArray strBytes = (*env)->NewByteArray(env, 50);

    // 设置字符数组
	(*env)->SetByteArrayRegion(env, strBytes, 0, strlen(destBuffer), destBuffer);

	// 构建字符编码
	jstring charSetName = (*env)->NewStringUTF(env, "GBK"); 
    // 创建String类的对象
	jstring transformStr = (*env)->NewObject(env, stringCls, stringConstructMid, strBytes, charSetName);

	return transformStr;
}
```
_Tips_

如果是使用的eclipse控制台输出 ， 一定要注意控制台的字符编码 ， 如果编码不统一 ， 控制台就会输出乱码 ，如：如果要输出的字符是UTF-8而控制台是GB2312 ， 则会输出乱码 。 如果出现乱码 ， 可以将字符写入到文件中 ， 看看是否会乱码 ，如果乱码 ，可以使用notepad++看一下文件的编码 ， 如果编码一致 ， 则会显示正常 。

### 结语
这几天公司产品上线 ， 有点忙碌 ， 没多少时间写文章 ， 今天恢复写文章的进度。昨天和一个创业公司的CEO聊了聊 ， 发现自己欠缺的东西还有很多很多 ， 对于产品的架构 ， 如何从0到1 ， 将想法转换为产品 ， 没有一个完整的产品思路 ，也没有产品经理的思维能力，有的只是对界面的吹毛求疵 ， 对部分功能的自以为是 。任何行业 ， 没有深入进去 ， 看的永远只是表面 ， 都是别人玩剩下的 。 

### 刻意练习技术 ， 时刻思考产品 。
