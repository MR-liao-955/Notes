### C++ 对象模型、this 指针  、空指针调用成员函数 、const修饰成员函数

#### 成员变量 和成员函数分开存储

在C++ 中，类内的成员变量和成员函数分开存储。

只有非静态成员变量才属于类的对象上。

```c++
#include<iostream>
#include"string"
using namespace std;
class Person {
public:
	int m_A;		//非静态成员变量
	static int m_B; //静态成员变量    TIP: 静态成员变量不属于类的对象上
    void func(){	//非静态成员函数， 也不属于类的对象上
		string name;
		int age;
	}  
	static void stfunc() {
		cout << "静态成员 函数" << endl; //静态成员函数也不属于类对象上。
	}
};
int Person::m_B = 10;//静态成员变量，类内声明，类外初始化。

void test01() {
	Person p;
	
	cout << "sizeof(p) = " << sizeof(p) << endl;//4   因为非静态成员变量属于类的对象，而静态成员变量/函数不属于它。
												//空对象占用的内存是1. 
												//但是对象有成员时候，按照成员的占用内粗空间来。
												//C++编译器会给每个空对象也分配一个字节的空间。
												//每个空对象也应该有一个独一无二的内存地址
	//但是如果对象有
}
void main() {
	 test01();
}
```





#### this指针！！

![image-20221103230929991](C:\Users\14163\AppData\Roaming\Typora\typora-user-images\image-20221103230929991.png)



> this 指针的作用

- 1.解决名称冲突
- 2.返回对象本身用 *this



```c++	 
#include <iostream>
#include "string" 
using namespace std;
class Person {
public:
	Person(int age) {
		//age = age    这里会出现问题。虽然不报错。但是编译器认为这 2个 age 都是成员变量。
		this->age = age;    //1.this-> 指针的作用之一是 解决名称冲突！！
	}
	int age;
	static void stfunc() {
		cout << "这里是类静态成员函数 stfunc()" << endl;
	}
	Person& PersonAddPerson(Person &p) {   //注意！！！ 
	//Person PersonAddPerson(Person p) {  这里Person& 。这个地址符必须加，
					//因为要链式思想编程，如果只返回对象Person 这叫做值传递，属于浅拷贝，会自动创建对象，而后面输出的结果是p1.age。哈哈就不一样，
		this->age = p.age + this->age;
		return *this;  //this表示 自身对象的指针，  这里 *this 解指针后就是自身的对象。
	}
};
void test01() {
	Person p(18);
	cout << "类中age成员的值为：" << p.age << endl;
}
void test02() {
	Person p1(10);
	Person p2(10);
	//链式编程思想，就如同 cout << << << 这个左移符号。可以多次叠加。
	p1.PersonAddPerson(p2).PersonAddPerson(p2);
	cout << "通过链式编程思想后叠加的p1的age =    " << p1.age << endl;
}

void main() {
	test01();
	test02();
}
```



#### 空指针调用成员函数

![image-20221103235526033](C:\Users\14163\AppData\Roaming\Typora\typora-user-images\image-20221103235526033.png)

```c++	
#include <iostream>
#include "string" 
using namespace std;
class Person {
public:
	void printPerson() {
		cout << "this is function of class Person" << endl;
	}
	void showPersonInfo() {
		if (this == NULL) {  //做出限制，判断是否存在对象，有对象才能访问成员变量。否则会报错，崩溃！
			return;
		}  //这里的if语句用于加强代码的健壮性
		cout << "age = " << age << endl;
	}
	int age;
};
void test01() {
	Person* p = NULL;
	p->printPerson();
	p->showPersonInfo(); //这里空指针，空指针没有对象，访问age的时候找不到对象，程序会崩溃。好在前面做了一个限制。
	cout << "函数继续运行" << endl;
}

void main() {
	test01();
}
```



#### const 修饰成员函数

> 常函数：

- 成员函数后加const 后我们称之为函数为 **常函数**。
- 常函数内不可以修改成员属性
- 成员属性声明时加关键字mutable后，在常函数中依然可以修改。

> 常对象：

- 声明对象前加const 称该对象为常对象
- 常对象只能调用常函数

```c++
#include<iostream>
#include"string"
using namespace std;
//1.常函数
class Person {
public:
	//this指针的本质 是指针常量 指针的指向是不可以修改的。
	//cosnt Person * const this;
	//在成员函数后面加const，修饰的是this指向 ，让指针指向的值也不可以修改
	void showPerson()const {
		//m_Age = 1090;  会报错。
		//this->m_Age = 100; //会报错
		m_B = 20; //加了mutable 之后可以修改。
	}
	 int m_Age = 10;
	 mutable int m_B =20;
    
	 void testfunc() {
	 }
};
//2.常对象
//常对象只能调用常函数！！！！
void test() {
	const Person p1;
	//p1.m_Age = 1000; 常对象不能修改 对象的值，除了mutable 修饰的。
	int b = p1.m_B;   //可以访问常对象的普通成员变量。。。但是不能修改它。
	int c = p1.m_Age;
	//p1.testfunc();   报错，常对象不能访问普通函数。常对象只能调用常函数
	p1.showPerson();  //常对象只能调用常函数。
	cout << "b的值为" << c << endl;
}
void main() {
	test();
}
```

