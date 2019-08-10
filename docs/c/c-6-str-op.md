字符操作 ， 是我们写程序时 ， 最常用的一种操作 ， 例如：字符串截取 ， 字符串替换 ， 字符串转整数类型等等 ， 都是字符操作 ， 在java中`String`类就是二三十个方法 ， 还有`StringBuffer` , `StringBuilder`等辅助类 。 下面 ，我们就来看看C语言中字符处理的函数 。

### 一. 字符数组
```c
// 字符数组
void charArray() {

	// 字符数组 ， 可以修改
	char c[15] = { 'y','e','s',' ','m','a','n' };
	// 字符数组可以直接作为字符串输出
	printf("%s\n", c);

	// 修改
	c[0] = 'a';

	printf("修改字符：%s\n", c);

}
```
字符数组 ， 本身是一个数组 ， 但是也可以作为字符串格式化输出 ， 可以任意替换里面的字符 ， 和操作数组是一样的 。

### 二. 字符指针
```c
/*字符指针*/
void charPointer() {
	// 内存连续排列 , 不可修改
	char* s = "Never give up";

	printf("内存地址：%#x\n", s);

	printf("-------------------------------\n截取字符\n");
	// 截取字符
	s += 6;
	while (*s)
	{
		printf("%c", *s);
		s++;
	}
}
```
字符指针与字符数组 ， 最大的区别是一个定义了就不可以修改 ， 一个可以任意修改里面的字符 ， 就和`String`和`StringBuilder`一样 ， String定义了就无法增加或修改里面的内容 ， 一个是可以增加和修改里面的内容 。

下面就列举几个常用的操作：
> 字符函数查询网站： [字符串函数](http://www.kuqin.com/clib/string/strpbrk.html)


### 拼接字符串
```c
/*拼接函数*/
void spliceString() {
	// 合并后存储字符的容器
	char dest[40] = {'h','e','h','e',' '};

	char* c1 = "Never give up ";

	char* c2 = "I blive you ! ";

	// 会清除字符数组原有的数据
	//strcpy(dest, c1);
	strcat(dest, c1);
	strcat(dest, c2);

	printf("\n拼接字符串：\n%s\n", dest);

}
```
### 查找单个字符
```c

/*查找单个字符*/
void searchChar() {
	char* str = "I wanna dont kiss you";
	char* p = strchr(str, 'd');
	
	if (p) {
		printf("索引位置：%d\n", p - str);
		printf("打印: ");
		str += p - str;
		while (*str)
		{
			printf("%c",*str);
			str++;
		}
		printf("\n");
	}
	else {
		printf("没有找到\n");
	}
}
```
### 查找字符串
```c

/*查找字符串*/
void sreachChars() {
	char* str = "落花有意流水无情";
	char* handle = "流";

	char* p = strstr(str, handle);

	if (*p) {
		printf("索引位置：%d\n", p - str);
		printf("打印: ");
		str += p - str;
		printf("%s ---- %s\n", str,p);
	}
	else {
		printf("没找到\n");
	}
}
```
### 字符转换
```c
/*字符转换int*/
void strToInt() {
	char* p = "123";
	int toInt = atoi(p);
	int res = toInt + 123;
	printf("字符转换int数值: %d\n", res);
}
```

### 字符转换函数定义：
```c
/-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
//
// Narrow String to Number Conversions
//
//-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
_Check_return_                    _ACRTIMP double    __cdecl atof   (_In_z_ char const* _String);
_Check_return_ _CRT_JIT_INTRINSIC _ACRTIMP int       __cdecl atoi   (_In_z_ char const* _String);
_Check_return_                    _ACRTIMP long      __cdecl atol   (_In_z_ char const* _String);
_Check_return_                    _ACRTIMP long long __cdecl atoll  (_In_z_ char const* _String);
_Check_return_                    _ACRTIMP __int64   __cdecl _atoi64(_In_z_ char const* _String);

_Check_return_ _ACRTIMP double    __cdecl _atof_l  (_In_z_ char const* _String, _In_opt_ _locale_t _Locale);
_Check_return_ _ACRTIMP int       __cdecl _atoi_l  (_In_z_ char const* _String, _In_opt_ _locale_t _Locale);
_Check_return_ _ACRTIMP long      __cdecl _atol_l  (_In_z_ char const* _String, _In_opt_ _locale_t _Locale);
_Check_return_ _ACRTIMP long long __cdecl _atoll_l (_In_z_ char const* _String, _In_opt_ _locale_t _Locale);
_Check_return_ _ACRTIMP __int64   __cdecl _atoi64_l(_In_z_ char const* _String, _In_opt_ _locale_t _Locale);

_Check_return_ _ACRTIMP int __cdecl _atoflt (_Out_ _CRT_FLOAT*  _Result, _In_z_ char const* _String);
_Check_return_ _ACRTIMP int __cdecl _atodbl (_Out_ _CRT_DOUBLE* _Result, _In_z_ char*       _String);
_
```
C语言中的字符操作函数也比较多 ， 这里列举几个常用的 ， 其他的可以去[字符串函数](http://www.kuqin.com/clib/string/strpbrk.html)网址查询 ， 都有例子  。
