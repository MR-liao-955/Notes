### 189讲 string容器构造函数、赋值操作、字符串拼接、字符串查找和替换、字符串比较、字符存取、字符串插入和删除、子串获取



![image-20221208000838484](C:\Users\14163\AppData\Roaming\Typora\typora-user-images\image-20221208000838484.png)

#### string构造函数

构造函数原型：

- `string();`                             //创建一个空的字符串，例如：string str;
- `string(const char * s) `  //使用字符串s 初始化

- `string( const string& str); `   //使用一个string 对象初始化另一个string 对象。
- `tring(int n, char c);`     //使用n 个字符 c初始化

```c++
#include "string"
using namespace std;
void test1() {
	//str1的默认构造函数
	std::string str1;
	cout << "str1 = " << str1 << endl;

	//str2使用C风格 的字符串进行构造
	const char* str2 = "Hello Word!";
	std::string string2(str2);
	cout << "str2 = " << str2 << endl;

	//str3使用拷贝构造函数。std::string(const string& str)拷贝另一个对象（应该是深拷贝，具体得看string源码）
	std::string str3 = "hello word!";
	std::string string3(str3);
	cout << "str3 = " << string3 << endl;

	//str4使用过n个字符 c初始化 std::string(int n , char c)；
	std::string string4(4, 'c');
	cout << "str4 = " << string4 << endl;
}
```



#### string 容器赋值操作

![image-20221208002019310](C:\Users\14163\Desktop\C++学习笔记\image-20221208002019310.png)

> summary : string的赋值方式很多，`operator= `这种方式是比较实用的。



#### string 字符串拼接

![image-20221219130324928](C:\Users\14163\Desktop\C++学习笔记\image-20221219130324928-16714262072681.png)

> summary: 字符串拼接的重载版本很多，初学阶段记住几个即可，需要的时候再查。



#### string查找和替换

![image-20221219221148438](C:\Users\14163\Desktop\C++学习笔记\image-20221219221148438.png)

```c++
void test4() {
	string st1 = "我真TM帅";
	st1.replace(2, 4, "handsome");
	cout << "st1 = " << st1 << endl; //st1 = 我handsome帅
}
```

> summary ： 
>
> - find查找是从左往后，rfind 从右往左。
> - find找到字符串后返回查找的第一个字符位置，找不到则返回 -1
> - replace 在替换时，要指定从哪个位置起，多少个字符，替换成什么样的字符串。



#### string字符串比较

![image-20221219223255192](C:\Users\14163\Desktop\C++学习笔记\image-20221219223255192.png)

```c++
void test1() {
	string str1 = "cbc";
	string str2 = "bbc";
	if (str1.compare(str2) == 0)  {
		cout << "str1 与 str2 相等" << endl;
	}
	else if(str1.compare(str2) > 0)
	{
		cout << "str1 比 str2 大" << endl;
	}
	else
	{
		cout << "str1 比 str2 小" << endl;
	}
}
```

> Summary：字符串的对比主要用于两个字符串是否相等，判断谁大谁小意义并不大。



#### string字符存取

string 中单个字符的存取方式有两种

- `char& operator[](int n);` //通过[] 方式取字符
- `char& at(int n);` //通过 at( ) 方法获取字符

```c++
void test2() {

	//访问单个字符
	string str = "helloooooo";
	cout << "str = " << str << endl;
	for (int i = 0; i < str.size(); i++) {
		cout << str[i] << " ";
	}
	cout << endl;
	for (size_t i = 0; i < str.size(); i++)
	{
		cout << str.at(i)<<  " ";
	}
	cout << endl;

	//修改单个字符
	str[0] = 'w';
	cout << "str = " << str << endl; //str = welloooooo
	str.at(0) = 't';
	cout << "str = " << str << endl;  //str = telloooooo

}
```

> Summary: string 字符串中单个字符存取有两种方式，利用 数组 [ ] 或者at( ) 方法。



#### string插入和删除

![image-20221219230734040](C:\Users\14163\Desktop\C++学习笔记\image-20221219230734040.png)

```c++
//string 的插入和删除方法。
void test3() {
	string str1 = "abc";
	str1.insert(1, "TM");
	cout << "str1 = " << str1 << endl;
	str1.erase(1, 2);
	cout << "str1 = " << str1 << endl;
}
```



#### string字串

**功能描述：**

- 从字符串中获取想要的字串



**函数原型：**

- `string substr(int pos = 0, int n = npos) const;` //返回由 pos 开始的 n 个字符组成的字符串

```c++
void test4() {
	string str1 = "abcdefgh";
	string str2 = str1.substr(2, 2);
	cout << str2 << endl; // cd

//实用性的使用。
	string str3 = "dearliao@email.com";
	int position = str3.find('@');
	string rec = str3.substr(0, position);
	cout << "rec = " << rec << endl;
}
```

> Summary ： 灵活地运用 substr( ) 的功能,有助于在开发中获取到有效的信息。

