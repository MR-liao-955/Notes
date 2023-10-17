### 215讲 list容器、set容器、仿函数

#### list基本概念 （链表）

![image-20230205003015928](C:\Users\14163\Desktop\C++学习笔记\image-20230205003015928.png)

> 优点：
>
> 可以对任意位置进行快速插入或删除

> 缺点：
>
> 1.容器遍历速度没有数组快
>
> 2.占用空间比数组大



![image-20230205003430592](C:\Users\14163\Desktop\C++学习笔记\image-20230205003430592.png)

> 由于链表存储方式不是连续的内存空间，因此链表list 中的迭代器只支持**前移**和**后移**，属于**双向迭代器**

list 的优点：

- 采用动态存储分配，不会造成内存浪费和溢出
- 链表执行插入和删除操作十分方便，修改指针即可，不需要移动大量元素

list 的缺点：

- 链表灵活，但是空间（指针域）和 时间（遍历）额外耗费较大

list 有一个重要的性质，插入操作和删除操作都不会造成原有的list 迭代器的失效，这在vector是不成立的



> 总结：STL中**List 和 vector是两个最常被使用的容器**，各有优缺点





#### list构造函数

***功能***：

- 创建list 容器

***函数原型***：

-  `list<T> lst;`    // list 采用模板类实现，对象的默认构造形式。
-  `list(beg,end);`    //构造函数将 [ beg , end) 区间中的元素拷贝给本身。
-  `list(n,elem);`    //构造函数将 n 个 elem 拷贝给本身。
-  `list(const list &lst);`    //拷贝构造函数。

```c++
void showList(list<int> lst) {
	for (list<int>::iterator it = lst.begin();it != lst.end();it++)
	{
		cout << *it << " ";
	}
	cout << endl;
}

//list 的构造函数的使用
void test() {
	//1. 默认构造函数，使用模板类实现
	list<Person> lst1;

	vector<int> vec;
	for (int i = 0; i  <  10; i++)
	{
		vec.push_back(i);
	}

	for (vector<int>::iterator it = vec.begin();it != vec.end();it++)
	{
		cout << *it << endl;
	}
	cout << "以下是list类型" << endl;

	//2.
	list<int> list2(vec.begin(), vec.end());
	showList(list2);
	cout << "3333333333333333333333333" << endl;

	//3.list(n,m);
	list<int> list3(5, 10);
	showList(list3);

	//4.拷贝构造函数创建新的容器
	list<int> list4(list3);

}
```

总结：list构造方式和其它几个STL常用容器相似，熟练掌握即可



#### list 赋值和交换

**功能描述：**

- 给 list 容器进赋值，以及交换 list 容器

**函数原型：**

- `assign(beg,end);`   //将 [ beg, end) 区间中的数据拷贝赋值给本身。
- `assign(n,elem);`     //将n 个elem 拷贝赋值给本身

- `list& operator=(const list &list);`      //重载等号运算符
- `swap(lst);`   //将lst 与本身的元素互换



#### list 大小操作

功能：

- 对 list 容器的大小进行操作。

函数原型:

- `size();`     //返回容器中元素的个数
- `empty();`   //判断容器是否为空
- `resize(num);`    //重新指定容器的长度为num ，若容器变长，则以默认值填充新的位置。

​	             //如果容器变短，则末尾超出容器长度的元素被删除。

- `resize(num,elem);`    //同上，但默认填充的为elem。



#### list 插入和删除

![image-20230208010502932](C:\Users\14163\Desktop\C++学习笔记\image-20230208010502932.png)

![image-20230208011633178](C:\Users\14163\Desktop\C++学习笔记\image-20230208011633178.png)

> 这里多了一个 `移除 remove();`它会把所有匹配的数移除掉



#### list数据存取

**功能描述：**

- 对list 容器中数据进行存取

**函数原型：**

- `front();`    //返回第一个元素
- `back();`  //返回最后一个元素

![image-20230208012724204](C:\Users\14163\Desktop\C++学习笔记\image-20230208012724204.png)

> 总结：

- list 容器不可以通过 [ ] 或者 at( ); 方式访问数据
- 返回第一个元素  ---  front( );

- 返回最后一个元素  --- back( );



#### list 反转和排序



***功能描述***：

- 将容器中的元素反转，以及将容器中的数据进行排序

***函数原型：***

- `reverse();`   //反转链表
- `sort();`  //链表排序



![image-20230208013628789](C:\Users\14163\Desktop\C++学习笔记\image-20230208013628789.png)

```c++
void test2() {

	list<int> lst1;
	lst1.push_back(1);
	lst1.push_back(5);
	lst1.push_back(2);
	lst1.push_back(3);
	lst1.push_back(9);

	lst1.sort();

	list<int>::iterator it1 = lst1.begin();
	list<int>::iterator it2 = lst1.end();
	//sort(it1,it2); //sort(lst1.begin(),lst1.end()); //这里编译器不会报错，但是运行会报错，
    //所有不支持随机访问迭代器的容器，不可以使用<algorithm> 库中的标准算法
	//不支持随机访问呢迭代器的容器，内部会提供对应的算法。！！！

	showList(lst1);

	cout << "反转后" << endl;

	lst1.reverse();
	showList(lst1);

}
```

如果需要反向排序，则写一个比对函数，将结果提交给成员函数的 `lst1.sort( myCompare(val1,val2));`

```c++
bool myCompare(int val1, int val2)
{

	return val1 > val2;
}

void test2(){
    list<int> lst1;
	lst1.push_back(1);
	lst1.push_back(5);
	lst1.push_back(2);
	lst1.push_back(3);
	lst1.push_back(9);


    
	lst1.sort(myCompare);
}
```



#### list容器案例

```c++
#include <iostream>
#include "string"
#include <list>

using namespace std;

class Person {
public:
	Person(string name, int age, int height) {
		m_name = name;
		m_age = age;
		m_height = height;

	}
	string m_name;
	int m_age;
	int m_height;

};
bool myCompare(Person &p1 , Person &p2) {
	if (p1.m_age == p2.m_age) {
		return p1.m_height <  p2.m_height;
	}
	else
	{
		return p1.m_age > p2.m_age;
	}
	
}

void printList(list<Person> &p) 
{
	for (list<Person>::iterator it = p.begin();it != p.end(); it ++)
	{
		cout << "\t姓名：" << it->m_name  << "\t年龄:" << ( * it).m_age  << "\t身高：" << ( * it).m_height  << endl;
	}
	cout << endl;
}


void test() {
	list<Person> lst1;
	Person p1("张三" , 18 , 175);
	Person p2("李四" , 23 , 186);
	Person p3("王五" , 18 , 165);
	Person p4("赵六" , 18 , 170);
	Person p5("萨尼铁塔" , 43 , 190);
	Person p6("模拟穷" , 29 , 195);
	lst1.push_back(p1);
	lst1.push_back(p2);
	lst1.push_back(p3);
	lst1.push_back(p4);
	lst1.push_back(p5);
	lst1.push_back(p6);

	printList(lst1);

	cout << "排序后的打印如下：" << endl;

	lst1.sort(myCompare);
	printList(lst1);

}

void main() {
	test();

}

```





### set 容器

**简介：**

- 所有元素都会在插入时**自动被排序**



**本质：**

- set/multiset 属于 **关联式容器** ， 底层结构是用 **二叉树** 实现。

**set 和 multiset 区别：**

- set 不允许容器中有重复的元素
- multiset 允许容器中有重复的元素



![image-20230215124452889](C:\Users\14163\Desktop\C++学习笔记\image-20230215124452889.png)

```c++
void printSet(set<int> set1) {
	for (set<int>::iterator it = set1.begin(); it != set1.end(); it++)
	{
		cout << *it << " ";
	}
	cout << endl;
}

void test() 
{
	set<int> set1;
    
	set1.insert(1);
	set1.insert(3);
	set1.insert(5);
	set1.insert(4);
	set1.insert(6);

	printSet(set1);

}
```



#### set容器大小和交换

![image-20230215232040277](C:\Users\14163\Desktop\C++学习笔记\image-20230215232040277.png)

```c++
void test() 
{
	set<int> set1;
	

	set1.insert(1);
	set1.insert(3);
	set1.insert(5);
	set1.insert(4);
	set1.insert(6);

	//set容器的大小
	cout << "容器的大小为： " << set1.size() << endl;
	//判断容器是否为空
	if (set1.empty()) {
		cout << "容器为空" << endl;
	}
	else {
		cout << "容器不为空" << endl;
	}


	cout << "===================================" << endl;
	//容器交换
	set<int> set2;
	set2.swap(set1);
	if (set1.empty()) {
		cout << "set1 容器为空" << endl;
	}
	else {
		cout << "set1容器不为空" << endl;
		cout << "set1容器的大小为：" << set1.size() << endl;
	}
	
	if (set2.empty()) {
		cout << "set2容器为空" << endl;
	}
	else {
		cout << "set2容器不为空" << endl;
		cout << "set2容器的大小为：" << set2.size() << endl;
	}

	printSet(set1);

}
```



#### set容器的插入和删除

![image-20230215232120729](C:\Users\14163\Desktop\C++学习笔记\image-20230215232120729.png)





#### set容器查找和统计

![image-20230215232640433](C:\Users\14163\Desktop\C++学习笔记\image-20230215232640433.png)

```c++
#include <iostream> 
#include <set>

using namespace std;

void printSet(const set<int>& st)
{
	for (set<int>::const_iterator it = st.begin(); it != st.end() ; it++)
	{
		cout << *it << " ";
	}
	cout << endl;

}


void test1()
{
	set<int> st1;
	st1.insert(10);
	st1.insert(30);
	st1.insert(20);
	st1.insert(40);

	//set容器的查找
	set<int>::iterator pos =  st1.find(300);
	if (pos != st1.end())
	{
    //容器的统计    
		cout << "找到该数据,且该数据的个数为：" << st1.count(30) << endl;
		cout << &pos << endl;
		
	}
	else {
		cout << "没有找到该数据" << endl;
		//cout << "此时迭代器返回的： " << *pos << endl;  如果pos 为set.end();  那么解指针之后会运行时报错
	}
}

void main() {
	test1();
}
```



#### set 和 multiset 的区别

![image-20230215234134353](C:\Users\14163\Desktop\C++学习笔记\image-20230215234134353.png)

- set中 insert方法返回的类型。

  ![image-20230216000507565](C:\Users\14163\Desktop\C++学习笔记\image-20230216000507565.png)

![image-20230216000339703](C:\Users\14163\Desktop\C++学习笔记\image-20230216000339703.png)

- multiset 容器的insert定义。

返回类型只有一个迭代器

![image-20230216000625590](C:\Users\14163\Desktop\C++学习笔记\image-20230216000625590.png)

因此，如果还是用set的模板来接收的话编译器会报错，因为这里的模板不能写 bool

![image-20230216000719933](C:\Users\14163\Desktop\C++学习笔记\image-20230216000719933.png)

![image-20230216000911259](C:\Users\14163\Desktop\C++学习笔记\image-20230216000911259.png)

```c++

void test()
{
	set<int> st1;
	st1.insert(10);

	pair<set<int>::iterator ,bool> res = st1.insert(10);

	cout << "res.second = " << res.second << endl; //返回1代表true ，插入成功。  返回0代表false 插入失败

	if (res.second)
	{
		cout << "插入成功,且*res.first = " << *res.first << endl;
	}
	else {
		cout << "插入失败,res.second = " << res.second << endl;
	}



}

void main() {
	test();

}

```

> Summary

- 如果不允许插入重复数据，就选 set 容器；
- 如果需要插入重复数据，就选 multiset 容器；



#### pair对组创建

![image-20230216001206291](C:\Users\14163\Desktop\C++学习笔记\image-20230216001206291.png)

- 我发现对组一个好玩的地方，绿色方框中可以拆开来创建对组，但是红色方框会报错，不存在int ---> string的转换

![image-20230216155414989](C:\Users\14163\Desktop\C++学习笔记\image-20230216155414989.png)



```c++
void test()
{
	pair<string, int > p1("Tom", 18);
	cout << " 姓名：" << p1.first << "  年龄：" << p1.second << endl;

	pair<string, int > p2 = make_pair("Jerry", 16);
	cout << " 姓名：" << p2.first << "  年龄：" << p2.second << endl;
}

void main() {
	test();
}
```



### set容器排序、仿函数

学习目标：

- set 容器默认排序规则为从小到大，掌握如何改变排序规则

主要技术点：

- 利用仿函数，可以改变排序规则



![image-20230216111736521](C:\Users\14163\Desktop\C++学习笔记\image-20230216111736521.png)

```c++
#include <iostream> 
#include "string"
#include <set>	

using namespace std;


class Person {
public:
	Person(string name, int age) {
		this->name = name;
		this->age = age;
	}

	string name;
	int age;
};

//仿函数部分，使用重载() 括号，来实现set 容器排序自定义数据类型。
class myCompare {
public:
    //  注意 形参列表后面有一个 const ， 这里的const 作用是让它变成···常函数···。常函数和常对象
	bool operator()(const Person &p1,const Person &p2) const{
		return p1.age > p2.age;
	}

};

void printPerson(const set<Person,myCompare>& st) {
	for (set<Person,myCompare>::const_iterator it = st.begin();it != st.end();it++)
	{
		cout << " 姓名: " << (*it).name << " 年龄: " << (*it).age;
		cout << endl;
	}

}

void test()
{
    //set的第二个模板，通过仿函数来实现自定义数据类型的比较。
	set<Person,myCompare> st1;

	Person p1("张三", 19);
	Person p2("李四", 21);
	Person p3("王五", 16);

	st1.insert(p1);
	st1.insert(p2);
	st1.insert(p3);
	printPerson(st1);


}

void main()
{
	test();
}

```

- 仿函数部分

```c++
//仿函数部分，使用重载() 括号，来实现set 容器排序自定义数据类型。
class myCompare {
public:
    //  注意 形参列表后面有一个 const ， 这里的const 作用是让它变成···常函数···。常函数和常对象
	bool operator()(const Person &p1,const Person &p2) const{
		return p1.age > p2.age;
	}
    
};
```































