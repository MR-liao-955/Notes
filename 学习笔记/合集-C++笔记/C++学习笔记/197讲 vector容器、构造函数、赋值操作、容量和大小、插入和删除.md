## 197讲 vector容器、构造函数、赋值操作、容量和大小、插入和删除



#### vector基本概念

**功能：**

- vector数据结构**和数组非常相似**，也称为**单端数组**



vector 与普通数组的区别：

- 不同之处在于数组是静态空间，而vector可以**动态扩展**



动态扩展：

- 并**不是**在原空间之后续借新空间，**而是找更大的内存空间，然后将原数据拷贝新空间**，**释放原空间。**

![image-20221220121538442](C:\Users\14163\Desktop\C++学习笔记\image-20221220121538442.png)

- vector容器的迭代器是支持随机访问的迭代器（可以跳跃式访问）



#### vector构造函数

功能描述：

- 创建vector 容器



函数原型：

- `vector<T> v;` //采用模板实现类实现，默认构造函数
- `vector(v.begin(),v.end());`  //将 `v [begin(),end() )`  区间（注意开闭）中的元素拷贝给自身。
- `vector(n,elem);`  //构造函数将n 个 elem拷贝给本身。
- `vector(const vector &vec)；` //拷贝构造函数。

```c++
void printVector(vector<T> &v) {
	for (size_t i = 0; i < v.size(); i++)
	{
		cout << v[i] << " ";
	}
	cout << endl;
}

void test1() {
	vector<int> v1;
	vector<int>::iterator it;
	
	for (size_t i = 0; i < 10; i++)
	{
		v1.push_back(i);
	}
	printVector(v1);
	
	//使用vector 的vector(v.begin(),v.end());将v [begin(),end() ) 区间的元素拷贝给自身
	vector<int> v2(v1.begin(),v1.end());
	printVector(v2);

	//使用vector 的vector(const vector<T> &vec); 拷贝构造函数
	vector<int> v3(v1);
	printVector(v3);

}
```



#### vector赋值操作

功能描述：

- 给vector 容器进行赋值（注意和构造函数初始化不一样）



函数原型：

- `vector& operator= (const vector &vec);` //重载 `=` 操作符
- `assign(beg,end);`  //将 `[ beg,end )`区间中的数据拷贝复制给本身
- `assign(n,elem);`  //将n 个 elem 拷贝赋值给本身

```c++
	/*  vector的赋值操作  */
	//////////////////////////////////下方注释处会报错。。
	//vector<string> v4;
	//v4[0] = "Me";
	//v4[1] = "Handsome";
	//v4[2] = "guy";
	///////////////////////////
	vector<int> v4;
	v4 = v1;
	printVector(v4);

	vector<int> v5;
	v5.assign(v1.begin(), v1.end());
	printVector(v5);

	vector<int> v6;
	v6.assign(5,'s');
	printVector(v6);
```



#### vector容量和大小

功能描述：

- 对vector 容器的容量和大小操作



函数原型：

- `empty();`  //判断容器是否为空
- `capacity();`  //容器的容量

- `size();`  //返回容器中元素的个数
- `resize(int num);` //1.重新指定容器长度为 num, 若容器变长，则默认填充新的位置。2.若容器变短，则末尾超出容器长度的元素被删除。
- `resize(int num,elem);`  //1.重新指定容器长度为num, 若容器变长，则以elem 值填充新位置。2.若容器变短，则末尾超出容器长度的元素被删除。

 ```c++
 vector<int> v1;
 	for (size_t i = 0; i < 6; i++)
 	{
 		v1.push_back(i);
 	}
 
 	//判断容器是否为空
 	if (v1.empty())
 	{
 		cout << "容量为空" << endl;
 	}
 	else
 	{
 		cout << "容量不为空" << endl;
 	}
 
 	//容器的容量
 	cout << "容器的容量为 v1.capacity() = " << v1.capacity() << endl;
 
 	//容器中元素的个数
 	cout << "容器中元素的个数v1.size() = " << v1.size() << endl;
 
 	//重新指定容器大小。
 	v1.resize(18);
 
 	cout << "修改后容器的大小" << v1.size() << endl; //18
 	cout << "修改后容器的容量" << v1.capacity() << endl;  //18
 	printVector(v1); //0 1 2 3 4 5 0 0 0 0 0 0 0 0 0 0 0 0
 
 
 	v1.resize(3);
 	cout << "修改后容器的大小" << v1.size() << endl; //3
 	cout << "修改后容器的容量" << v1.capacity() << endl;  //18   注意这里容器缩小后，但是它的容器大小不变。因为容器缩小，并不需要拷贝一份数据到新容器，所以容器容量大小不变。
 	printVector(v1); //0 1 2 
 
 	v1.resize(8, 666);
 	printVector(v1);//0 1 2 666 666 666 666 666
 ```



#### vector插入和删除

功能描述：

- 对vector 容器进行插入、删除操作



函数原型：

- `push_back(elem);` //尾部插入元素elem
- `pop_back();`  //删除最后一个元素
- `insert(const_iterator pos, elem);`  //迭代器指向位置pos 插入元素elem
- `insert(const_iterator pos, int count , elem);`  //迭代器指向位置pos 插入count 个元素elem
- `erase(const_iterator pos);`  //删除迭代器指向的元素
- `erase(const_iterator start,const_iterator end);`  //删除迭代器从start 到end 之间的元素。（start 和end 可以任意，如果start是容器头部，end是尾部，它就等效于clear( ) 清空函数。）
- `clear();`   //删除容器中所有元素

```c++
	//容器的插入与删除
	v1.clear();
	for (size_t i = 0; i < 5; i++)
	{
		v1.push_back(i);
		
	}
	v1.pop_back();
	printVector(v1);


	vector<int>::iterator it;// 创建迭代器 , 插入和删除都需要迭代器访问
	it = v1.begin();
	v1.insert(it += 2, 100); //
	printVector(v1);  //

	vector<int>::iterator it2;
	v1.insert(v1.begin(), 2, 666);
	printVector(v1);

	v1.erase(v1.begin(), v1.end());
	printVector(v1);
	v1.clear();
```



