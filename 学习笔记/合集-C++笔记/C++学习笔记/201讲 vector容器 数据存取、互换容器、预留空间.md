### 201讲 vector容器 数据存取、互换容器、预留空间



#### vector容器数据存取

功能描述：

- 对vector 中的数据得存取操作



函数原型：

- `at(int idx);`  //返回索引 idx 所指得数据
- `operator[];`  //返回索引idx 所指得数据
- `front[];`  //返回容器中的第一个数据元素
- `back[];`  //返回容器中最后一个数据元素

![image-20221221124311894](C:\Users\14163\AppData\Roaming\Typora\typora-user-images\image-20221221124311894.png)



> Summary ：
>
> - 除了用迭代器访问元素，还可以用 [ ] 、 at( ) 来访问
> - front( ) 可以访问容器中第一个元素
> - back( ) 可以访问容器中最后一个元素



#### vector 互换容器

功能描述:

- 实现两个容器内元素进行互换

函数原型：

- `swap(vec);` // 



```c++
	vector<int> v1;
	for (int i = 0; i < 10; i++)
	{
		v1.push_back(i);
	}
	cout << "交换前 v1 的元素为：  ";
	printVector(v1);

	vector<int> v2;
	for (size_t i = 10; i > 0 ; i--)
	{
		v2.push_back(i);
	}
	cout << "交换前 v2 的元素为：  ";
	printVector(v2);
	
	v1.swap(v2);
	cout << "交换后 v1 的元素为：  ";
	printVector(v1);
	cout << "交换后 v2 的元素为：  ";
	printVector(v2);
```



> Summary

swap( vector v1 ); 可以用于最开始容器扩容到很大了之后，把它 resize( ) 变小后内存并没有释放的问题，用于节省空间。

```c++
	void test3() {
		vector<int> v3;
		for (size_t i = 0; i < 50; i++)
		{
			v3.push_back(i);
		}
		cout << " 缩小前v3.capacity() = " << v3.capacity() << endl;
		cout << " 缩小前v3.size() = " << v3.size() << endl;

		v3.resize(10);

		cout << " 缩小后v3.capacity() = " << v3.capacity() << endl;
		cout << " 缩小后v3.size() = " << v3.size() << endl;

		cout << "===================================" << endl;

		vector<int>(v3).swap(v3);

		cout << " 通过匿名对象将容器内存回收后v3.capacity() = " << v3.capacity() << endl;
		cout << " v3.size() = " << v3.size() << endl;

	}
```

![image-20221228002512783](C:\Users\14163\Desktop\C++学习笔记\image-20221228002512783.png)





#### vector预留空间

功能描述：

- 减少 vector 在动态扩展容量时的扩展次数



函数原型：

- `reserve(int len);`  //容器预留 len 个元素长度，预留位置不初始化，元素不可访问。

```c++
	void test4() {

		vector<int> v4;
		//v4.reserve(100000);
		v4.reserve(100000);  /* 使用预留空间 少让容器扩几次容提高效率 */
		int count = 0,temp = v4.capacity();  //留给用容器大小判断容器是否扩容使用。
		int num = 0; //指针判断容器是否扩容的计数。
		int *  p = NULL;//指针判断容器是否扩容
		for (int i = 0 ; i < 100000 ; i++)
		{
			v4.push_back(i);

			if (temp != v4.capacity()) {  //使用大小判断容器是否扩容
				temp = v4.capacity();
				count++;
			}

			if (p != &v4[0]) { //使用指针和上面的使用大小判断容器是否扩容效果一样。
				p = &v4[0];
				num++;
			}
		 }

		cout << "count = " << count << endl;
		cout << "temp = " << temp << endl;
		cout << "num = " << num << endl;
	}
```



> 总结

如果数据量大的话，开始最好用 reserve( ) 方法来让vector 容器分配空间。

































