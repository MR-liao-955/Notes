## 204讲  deque容器

#### deque 容器基本概念

**功能：**

- **双端数组**，可以对头端进行插入和删除操作。（vector 在前面插入数组还要讲后面的数据都向后移位，导致效率变低）

**deque 与 vector 区别：**

- vector 对于头部的插入删除效率低，数据量越大，效率越低；
- deque 相对而言，对头部插入删除速度会比 vector 快；

- vector 访问元素时的速度会比deque 快，这和两者的内部实现有关；

![image-20221228005454194](C:\Users\14163\Desktop\C++学习笔记\image-20221228005454194.png)



**deque 内部工作原理：**

> deque内部有一个**中控器**，维护每段缓冲区中的内容，缓冲区中存放真实数据，
>
> 中控器维护的是每个缓冲区的地址，使得使用deque 时**像一片连续的内存空间**（业务外包的原型 :) ）

![image-20221228010100081](C:\Users\14163\Desktop\C++学习笔记\image-20221228010100081.png)

- deque 容器的迭代器也支持随机访问的。



#### deque 构造函数

功能描述：

- deque 容器构造

函数原型:

- `deque<T> ` deqT;   //默认构造形式
- `deque(beg,end);`  //构造函数将 [ beg,end ) 区间中的元素拷贝给本身。
- `deque(n,elem);`  //构造函数将n 个 elem 拷贝给本身。
- `deque(const deque &deq);`   //拷贝构造函数



```c++
//deque 的构造函数。
void test01() {
	deque<int> deq1;  //默认构造函数
	for (int i = 0; i < 10; i++) {
		deq1.push_back(i);
	}
	printDeque(deq1);
	
	vector<int> v1;  //写一个vector 进行测试看看deque 能不能用vector返回的迭代器？（答案是可以的）
	for (int i = 0; i < 10; i++) {
		v1.push_back(i);
	}
	
	vector<int>::iterator test;   //创建vector 测试迭代器
	test = v1.begin() + 3;

	deque<int> deq2(test, v1.end());  //答案：dqeue 可以使用vector 的迭代器
	printDeque(deq2);

	deque<int> deq3(10, 100);   //deque 的有参构造函数，构造10 个100 的数的容器
	printDeque(deq3);

	deque<int> deq4(deq3);  //deque 的拷贝构造函数
}
```

TIP:

1. `//dqeue 可以使用vector 的迭代器`

2. 如果要在形参中限制传入的容器的不小心的修改，可以加 const,但是编译器会报错，所以迭代器需要使用 `const_iterator`

   ![image-20221229000225450](C:\Users\14163\Desktop\C++学习笔记\image-20221229000225450.png)

   ![image-20221229000459313](C:\Users\14163\Desktop\C++学习笔记\image-20221229000459313.png)

​	



#### deque容器大小操作

功能描述：

- 对deque 容器的大小进行操作

函数原型：

- `deque.empty();`  //判断容器是否为空

- `deque.size();`  //得到容器包含元素的个数

- `deque.resize(len);`  

  //重新指定容器的长度为len ,若容器边长，则以默认值填充新位置。

  //如果容器变短，则末尾超出容器长度的元素被删除

- `deque.resize(num,elem);`  

  //重新指定容器的长度为num ,若容器变长，则以elem 值填充新位置。

  //如果容器变短，则末尾超出容器长度的元素被删除。



> deque 没有容量的说法哦，因为deque 可以在中控器中无限开辟空间，中控器只是加一个地址。区别与vector 容器。

> Summary

1. deque没有容量的概念
2. 判断是否为空 -- emprt( );
3. 返回元素个数 -- size( );
4. 重新指定个数 -- resize( len ); -- resize( num, elem );

```c++
void test02() {
	deque<int> d1;
	for (int i = 0; i < 10; i++)
	{
		d1.push_back(i);
	}

	if (d1.empty())  //判断容器是否为空。使用 empty() 方法。
	{
		cout << "d1容器为空" << endl;
	}
	else {
		cout << "d1容器不为空" << endl;
		cout << "d1容器的大小为: " << d1.size() << endl;
	
	}
	d1.resize(15, 6);
	d1.resize(20);

	printDeque(d1);
}
```



#### deque 插入和删除

功能描述：

- 向deque 容器中插入和删除数据



函数原型：

- `push_back(elem);`  //在容器尾部添加一个数据
- `push_front(elem);`  //在容器头部插入一个数据
- `pop_back();`  //删除容器最后一个数据
- `pop_front();`  //删除容器第一个数据

指定位置操作：

- `insert(pos,elem);`   //在pos 位置插入一个elem 元素的拷贝，返回新数据的位置
- `insert(pos,n,elem);`  //在pos 位置插入n 个elem 数据，无返回值
- `insert(pos,beg,end);`  //在pos 位置插入 [ beg,end ) 区间的数据，无返回值
- `clear();`   //清空容器中所有数据
- `erase(beg,end);`  //删除 [beg,end ) 区间的数据，返回下一个数据的位置。
- `erase(pos);`   //删除pos 位置的数据，返回下一个数据的位置

![image-20230104001928091](C:\Users\14163\Desktop\C++学习笔记\image-20230104001928091.png)



```c++
//两端插入
void test01() {
	deque<int> d1;
	
	d1.push_back(10);
	d1.push_back(20);

	d1.push_front(100);  //头插
	d1.push_front(200);  //头插
	printDeque(d1);  //200 100 10 20
	d1.pop_back(); //尾删   //200 100 10
	d1.pop_front(); // 头删  

	printDeque(d1);//100 10

	//按照区间进行插入
	deque<int> d2;
	d2.push_back(1);
	d2.push_back(2);
	d2.push_back(3);
	d2.insert(d2.begin(), d1.begin(), d1.end()); //100 10 1 2 3
	printDeque(d2);
}
```

```c++
//deque的删除
void test02() {
	deque<int> d3;
	d3.push_back(10);
	d3.push_back(20);
	d3.push_back(100);
	d3.push_back(200);
	printDeque(d3);  //10 20 100 200

	deque<int>::iterator it = d3.begin();
	it += 2;
	d3.erase(it);
	printDeque(d3); //10 20 200
	//根据区间来清空.
	d3.erase(d3.begin(), d3.end());
	printDeque(d3);
	//清空
	d3.clear();
	printDeque(d3);
}
```



#### deque 数据存取

功能描述：

- 对deque 中的数据的存取操作

函数原型：

- `at(int idx);`  //返回索引 idx 所指向的数据
- `operator[];`   //返回索引index 所指向的数据
- `front();`  //返回容器中第一个数据元素
- `back();`  //返回容器中最后一个数据元素



#### deque 排序

功能描述：

- 利用算法实现对deque 容器进行排序

算法：

- `sort(iterator beg,iterator end)`  //对beg 和 end 区间内元素进行排序



```c++
#include<deque>
#include<algorithm> //标准算法头文件

//deque容器排序
void test1() {
	deque<int> d;
	d.push_back(10);
	d.push_back(20);
	d.push_back(30);
	d.push_front(100);
	d.push_front(200);
	d.push_front(300);
	printDeque(d);  //300 200 100 10 20 30
	
	//排序
	sort(d.begin(),d.end());
	cout << "排序后的结果: ";
	printDeque(d);
//排序后的结果: 10 20 30 100 200 300
}
```

> Summary 

对于支持随机访问的迭代器的容器，都可以利用sort 算法直接对其进行排序

























