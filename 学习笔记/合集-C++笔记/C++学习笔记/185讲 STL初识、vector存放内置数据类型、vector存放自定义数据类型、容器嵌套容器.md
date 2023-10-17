### 185讲 STL初识、vector存放内置数据类型、vector存放自定义数据类型、容器嵌套容器

#### STL初识

> STL的诞生

- 长久以来，软件界一直希望建立一种可重复利用的东西
- C++ 的**面向对象**和**泛型编程思想**，目的就是**复用性**的提升
- 大多数情况下，数据结构和算法都未能有一套标准，导致被迫从事大量重复工作
- 为了建立数据结构和算法的一套标准，诞生了STL

> STL基本概念

- STL( Standard Template Library, 标准模板库 )
- STL 从广义上分为： **容器（container）、算法（algorithm）、迭代器（Iterator）**
- **容器** 和 **算法** 之间通过 **迭代器** 进行无缝衔接
- STL 几乎所有的代码都采用了 模板类 或者 模板函数

> STL 六大组件

**STL 大体分为六大组件，分别是：容器、算法、迭代器、仿函数、适配器（配接器）、空间配置器**

1. 容器：各种数据结构，如vector、list、deque、set、map等，用来存放数据
2. 算法：各种常用算法，如sort、find、copy、for_each等。
3. 迭代器：扮演了 **容器 **与 **算法 **之间的胶合剂。
4. 仿函数：行为类似函数，可作为算法的某种策略。
5. 适配器：一种用来修饰容器或者仿函数或迭代器接口的东西。
6. 空间配置器：负责空间的配置与管理



> STL 中容器、算法、迭代器

![image-20221207213909681](C:\Users\14163\Desktop\C++学习笔记\image-20221207213909681.png)

**迭代器：**容器和算法之间的粘合剂

提供一种方法，使之能够依序寻访某个容器所含的各个元素，而又无须暴露该容器的内部表示方式。

每个容器都有自己的专属迭代器

迭代器使用方式非常类似于指针，初学阶段我们可以先理解迭代器为指针。

![image-20221207214723010](C:\Users\14163\Desktop\C++学习笔记\image-20221207214723010.png)

- 常用的迭代器种类为双向迭代器，和随机访问迭代器

> - **算法都要通过迭代器访问容器中元素**



#### vector 存放内置数据类型

STL中最常用的容器为 vector，可以理解为数组。

容器：`vector`

算法：`for_each`

迭代器：`vector<int>::iterator`

`talk is cheap,show me the code!!!!!`

```c++

#include<iostream>
#include"string"

#include<vector>
#include<algorithm>
using namespace std;

void printVector(int x) {
	cout << x;
}

void test01() {
	//创建了一个vector容器，可以看成数组
	vector<int> v;

	//向容器中插入数据，push_back()尾插法。
	v.push_back(10);
	v.push_back(20);
	v.push_back(30);
	v.push_back(40);

	//通过迭代器访问容器中的数据
	vector<int>::iterator itBegin = v.begin(); //起始迭代器，指向容器中的第一个元素
	vector<int>::iterator itEnd = v.end();//结束迭代器,指向容器中最后一个元素的下一个位置。

	//有三种遍历方式：while、for、for_each()。
	//while 遍历容器
	while (itBegin != itEnd)
	{
		cout << *itBegin;
		itBegin++;
	}
	cout << endl;
	cout << "=====================" << endl;
	//第二种遍历方式
	for (itBegin = v.begin(); itBegin != itEnd;itBegin++ ) {
		cout << *itBegin;
	}
	cout << endl;
	cout << "============111========" ;

	//第三种方式使用 for_each()方法来遍历
	for_each(itBegin,itEnd,printVector); //第三个参数是函数名，我在这个test01();方法前定义了。但是不要加() 括号。这里底层机制是用了一个回调函数的机制
}

int main() {
	test01();
	system("pause");
	return 0;
}
```



```C++
vector<int> v;

vector<int>::iterator itEnd = v.end();//指向容器最后一个元素的下一个位置。
```

![image-20221207220332123](C:\Users\14163\Desktop\C++学习笔记\image-20221207220332123.png)

#### vector容器存放自定义数据类型

```c++
#include<iostream>
#include"string"
#include<vector>

using namespace std;

class Person {
public:
	Person(string name,int age) {
		m_Name = name;
		m_Age = age;
	}
	string m_Name;
	int m_Age;
};


Person p1("aaa", 10);
Person p2("bbb", 20);
Person p3("ccc", 30);
Person p4("ddd", 40);


void test1() {

	vector<Person> v;
	v.push_back(p1);
	v.push_back(p2);
	v.push_back(p3);
	v.push_back(p4);
	for (vector<Person>::iterator it = v.begin(); it != v.end(); it++) {
		cout << "<Person>类型的名字：" << (*it).m_Name << "\t 年龄" << it->m_Age << endl;
	}
}
void test2() {
	vector<Person *> v2;
	v2.push_back(&p1);
	v2.push_back(&p2);
	v2.push_back(&p3);
	v2.push_back(&p4);
	for (vector<Person *>::iterator it = v2.begin(); it != v2.end(); it++) {
		cout << "<Person*>类型的名字：" << (*it)->m_Name << "\t 年龄" << ( * *it).m_Age << endl;
	}

}
void main() {
	test2();
}
```

- 值得关注的是，`<Person> `这里面存放的是对象还是指针 `<Person *>`。如果是指针，那么 `(*it)` 解引用出来就是一个指针,且此处的 ` it` 是一个双重指针，可以用`(**it)`来获得它对象。



#### 容器嵌套容器

```c++
#include<iostream>
#include"string"
using namespace std;
#include<vector>

void test() {
	vector<vector<int>> vtInt;

	//初始化内容器
	vector<int> v1;
	vector<int> v2;
	vector<int> v3;

	//v1.push_back(1);
	//v1.push_back(2);
	//v1.push_back(3);
	//v2.push_back(2);
	//v2.push_back(3);
	//v2.push_back(4);
	//v3.push_back(3);
	//v3.push_back(4);
	//v3.push_back(5);
	for (int i = 1; i < 4;i ++) {
		v1.push_back(i);
		v2.push_back(i+1);
		v3.push_back(i+2);
	}

	vtInt.push_back(v1);
	vtInt.push_back(v2);
	vtInt.push_back(v3);

	//遍历容器
	for (vector<vector<int>>::iterator itVector = vtInt.begin(); itVector != vtInt.end();itVector++) {
		for (vector<int>::iterator it = (*itVector).begin(); it != (*itVector).end();it++) {
			cout << *it << "\t";
		}
		cout << endl;
	}
}

void main() {
	test();
}
```

- 容器嵌套，可以类比于二维数组。。。注意它的数据类型，同时头脑清晰，要知道它是一个对象还是一个容器，解指针外层容器来获得内层容器的对象，然后通过它来`.`访问方法（）或者成员





