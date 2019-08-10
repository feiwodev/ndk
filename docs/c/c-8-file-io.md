在结构体与指针中 ， 我们了解到结构体与java中的类相似 ， 也是一种自定义类型数据结构 ， 也学习了结构的各种用法 ， 以及简单的应用 。

在编写应用程序的时候 ， 文件IO操作是不可避免的 ， 小到日志本地化收集 ， 大到数据格式化存储 ， 都需要使用文件IO ， 来操作文件流进行数据处理 。在使用java开发的时候 ， 我们有`File`类和`FileReader`,`FileWriter`类来搭配使用 ， 也有`FileInputStream`,`FileOutputStream`和`BufferedInputStream`,`BufferedOutputStream`金牌组合 。使得java中的文件IO很方便 ， 下面我们就来看看简单的java文件IO示例：

```java
     // 读取文件中的字符
	private static void readString() throws Exception{
		File _file = new File("e:"+separetor+"a.txt") ;
		if (!_file.exists()) {
			boolean createStatuts = _file.createNewFile() ;
			if (createStatuts) {
				System.out.println("创建了一个新文件 ，并且创建成功了");
			}else {
				System.out.println("创建新文件失败");
			}
		}
		
		// 创建输入流
		InputStream fileInputStream = new FileInputStream(_file) ;
		byte[] bytes = new byte[1024] ;
		int len = -1;
		StringBuffer buffer = new StringBuffer() ;
		while ((len = fileInputStream.read(bytes)) != -1) {
			buffer.append(new String(bytes,0,len)) ;
		}
		// 关闭输入流
		fileInputStream.close() ;
		
		System.out.println(buffer.toString());
	}

    // 将字符串写入文件
	private static void writeString() throws Exception{
		File _file = new File("e:"+separetor+"a.txt") ;
		if (!_file.exists()) {
			boolean createStatuts = _file.createNewFile() ;
			if (createStatuts) {
				System.out.println("创建了一个新文件 ，并且创建成功了");
			}else {
				System.out.println("创建新文件失败");
			}
		}
		
		// 创建输出流
		OutputStream fileOutputStream = new FileOutputStream(_file) ;
		String info = "《op 青空 pure rouge 君吻》《君吻 その目に映るもの piano》" ;
		fileOutputStream.write(info.getBytes()) ;
		// 关闭输出流
		fileOutputStream.close() ;
		
	}
```
### 文件IO操作步骤：
1.创建一个File对象（需要操作的文件）
2.构建输入输出流 
3.创建缓冲区 ， 缓存读写数据 （将流数据读入到内存或写入到磁盘）
3.关闭流 （关闭文件流）

语言都是相通的 ， 在C语言中文件IO的操作也是如上述几步 ， 下面我们就一起来看看：
```c
/*读取文本文件*/
void readTextFile() {
	char* path = "C:\\Users\\Zeno\\Documents\\Visual Studio 2015\\Projects\\Hello_C\\Hello_C\\StructPointer.c";

	// 打开文件
	FILE* fp = fopen(path, "r");
	if (fp == NULL)
	{
		printf("打开文件失败\n");
		return;
	}
	// 字符缓冲区 ， 每次读一个字符串 ， 都会缓存到字符数组中
	char buffer[1024];
	while (fgets(buffer, 1024, fp)) {
		printf("%s", buffer);
	}

	// 关闭文件流
	fclose(fp);

/*写入文本文件*/
void writeTextFile() {
	char* path = "E:\\document\\write.txt";

	char* content = "如果 爱情是一场花火 ,一闪即逝的花火,我也要去追求\n如果 爱情是一场花火 ,一闪即逝的花火,我也要去追求\n";

	// 打开文件
	/*
		打开只写文件，若文件存在则文件长度清为0，即该文件内容会消失。若文件不存在则建立该文件。
	*/
	FILE* fp = fopen(path, "w");
	if (fp == NULL)
	{
		printf("打开文件失败\n");
		return;
	}
	// 写入文件
	fputs(content, fp);

	// 关闭文件流
	fclose(fp);
}
}
```
首先使用`fopen`函数得到一个文件指针 ， 操作符`r`为读取文件流 ， 构建了一个`buffer`数据缓冲区 ， 通过`fgets`函数循环读取文件数据 ， `fclose`函数关闭文件流 。在操作文件IO的时候 ， 最重要的函数 ， 莫过于`fopen`函数了 ， 首先我们来看一下`fopen`函数的定义：
```c
_ACRTIMP FILE* __cdecl fopen(
    _In_z_ char const* _FileName,
    _In_z_ char const* _Mode
    );
```
我们发现`fopen`函数 ， 需要传入文件全路径名称 ， 还有一个`_Mode` ， 这个是文件操作模式 ， C语言中文件操作主要依靠操作模式来辨别是输入流还是输出流的 。
下面列举一些常用的操作模式：
```c
mode有下列几种形态字符串:

r 打开只读文件，该文件必须存在。

r+ 打开可读写的文件，该文件必须存在。

w 打开只写文件，若文件存在则文件长度清为0，即该文件内容会消失。若文件不存在则建立该文件。

w+ 打开可读写文件，若文件存在则文件长度清为零，即该文件内容会消失。若文件不存在则建立该文件。

a 以附加的方式打开只写文件。若文件不存在，则会建立该文件，如果文件存在，写入的数据会被加到文件尾，即文件原先的内容会被保留。

a+ 以附加方式打开可读写的文件。若文件不存在，则会建立该文件，如果文件存在，写入的数据会被加到文件尾后，即文件原先的内容会被保留。
```
值得注意的是 ， 上述操作模式是针对文本文件的 ， 如果要操作二进制文件 ， 则需要在上述操作符后面加上`b` ， 如`rb`,`wb`,`ab` ， 等等 。

不论是文本文件的操作还是字符文件的操作 ， 都是 ， 打开文件 ， 创建缓冲区 ， 读写文件 。

### 二进制文件读写
```c
/*读写二进制文件 -- 复制文件*/
void fileCopy() {
	char* currentPath = "E:\\android_pdf\\研磨设计模式.pdf";
	char* destPath = "E:\\android_pdf\\研磨设计模式_new.pdf";

	// 打开文件
	FILE* currentFile_P = fopen(currentPath, "rb");
	FILE* destFile_P = fopen(destPath, "wb");

	// 先读取再写入
	int buffer[1024]; // 数据缓冲区
	int len; // 每次读取数据的长度
	while ((len = fread(buffer,sizeof(int),1024,currentFile_P)) != EOF)
	{
		// 将缓冲区里的内容写入到文件中
		fwrite(buffer, sizeof(int), len, destFile_P);
	}

	// 关闭流
	fclose(destFile_P);
	fclose(currentFile_P);
}
```
读写二进制和读写文本文件没多少区别 ， 最大的区别就是`fopen`函数中的模式的不同 ， 文本文件是`r`,`w` ， 二进制文件是`rb`,`wb` 。

了解了文件IO的基本操作 ， 我们使用文件IO流写一个加密解密的小程序。我们知道 ， 所有的文件都是以二进制存储的 ， 我们看的文本文件， 图片文件 ， 视频文件 ， 都是以二进制存储在磁盘上的 ， 那么 ， 我们可以将文件读取出来 ， 进行二进制运算 ， 就可以将文件加密解密了 。下面我们通过`^`异或运算来进行文件的加密解密 ， 异或运算的规则如下:
```c
0 ^ 1 得 1
1 ^ 1 得 0
0 ^ 0 得 0
1 ^ 0 得 1
```
相同为0 不同为1 , 例如 ， 我们将`4`这个数加密 ， 异或的数是5 ， 下面我们来看看运算：
```c
4的二进制是：0100
5的二进制是：0101
异或运算结果（加密）：4 ^ 5  == 0001    
异或运算结果（解密）： 0001 ^ 0101 == 0100   
由上述可见 ， ^一次为加密 ， 再^一次就是解密
```
代码示例如下：
### 文件加密
```c
/*加密文件*/
void encryptFile() {

	// 待加密文件路径
	char* normal_path = "E:\\poto\\xj.jpg";
	// 加密后文件路径
	char* encrypt_path = "E:\\poto\\xj_encrypt.jpg";

	//打开文件
	FILE* normal_fp = fopen(normal_path, "rb");
	FILE* encrypt_fp = fopen(encrypt_path, "wb");
	// 读文件
	int buffer;
	while ((buffer = fgetc(normal_fp)) != EOF) {
		// 写入文件
		fputc(buffer ^ 8, encrypt_fp);
	}

	printf("文件加密成功\n");

	// 关闭流
	fclose(encrypt_fp);
	fclose(normal_fp);
}
```

### 文件解密
```c
/*文件解密*/
void decryptFile() {
	// 加密文件路径
	char* encrypt_path = "E:\\poto\\xj_encrypt.jpg";
	// 解密文件路径
	char* decrypt_path = "E:\\poto\\xj_deencrypt.jpg";

	// 打开文件
	FILE* encrypt_fp = fopen(encrypt_path, "rb");
	FILE* decrypt_fp = fopen(decrypt_path, "wb");

	// 读取文件
	int buffer;
	while ((buffer = fgetc(encrypt_fp)) != EOF) {
		// 写文件
		fputc(buffer ^ 8, decrypt_fp);
	}

	printf("文件解密成功\n");

	// 关闭流
	fclose(decrypt_fp);
	fclose(encrypt_fp);
}
```
了解了文件加密的原理 ， 我们也可以推导出其他形式的加密 ， 如带密码的文件加密解密 ， 混合文件加密解密等等 。

不知不觉C语言基础系列已经写了快十篇了 ， 也快告一段落了 ， 有了这些基础知识 ， 我们就可以去分析分析jni.h头文件了 ，  下一个系列是jni开发系列 ， 我们学C语言就是为了能和java打交道 ， 那么下个系列我们就来学习C与java的桥梁 ， jni （Java Native Interface）技术 。
