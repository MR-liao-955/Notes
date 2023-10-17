### 211讲stack容器、queue容器



#### stack 基本概念

概念：stack 是一种 **先进后出** （First In Last Out,FILO) 的数据结构，它只有一个出口。

![image-20230105223513304](C:\Users\14163\Desktop\C++学习笔记\image-20230105223513304.png)

栈中只有顶端的元素才可以被外界使用，因此栈不允许有遍历行为

栈中进入数据称为  ---- **入栈** `push`

栈中弹出数据称为  ---- **出栈** `pop`

类似于 `弹匣` `挤人多地铁`



#### stack常用接口

功能描述: 栈容器中常用对外接口



构造函数：

- `stack<T> stk;`   //stack 采用模板类实现，  stack对象的默认构造形式。
- `stack(const stack &stk);`   //拷贝构造函数

赋值操作：

- `stack& operator=(const stack &stk);`   //重载 等号 操作符

数据存取：

- `push(elem);`   //压栈，向栈顶添加元素
- `pop();`  //出栈, 从栈顶移除一个元素
- `top();`  //返回栈顶元素

大小操作：

- `empty();` //判断栈是否为空
- `size();`   //返回栈的大小

```c++
//stack 的使用
void test01() {
	stack<int> stk1;
	stk1.push(10);
	stk1.push(20);
	stk1.push(30);
	stk1.push(40);

	//判断栈是否为空，否则遍历并出栈
	//while (!stk1.empty())
	//{
	//	//循环遍历栈的元素
	//	cout << " 栈顶元素为 ---- " << stk1.top() << endl;
	//	cout << " 栈的大小为 ---- " << stk1.size() << endl;
	//	stk1.pop();

	//}

	cout << " 栈的大小为 ---- " << stk1.size() << endl;

	//stack<int> stk2(stk1);    //使用拷贝构造
	stack<int> stk3;
	stk3 = stk1;   //使用了运算符重载
	cout << "stk3的大小为: " << stk3.size();
}
```



#### queue容器

概念：Queue 是一种**先进先出** (First In First Out , FIFO) 的数据结构，它有两个出口

![image-20230105225557630](C:\Users\14163\Desktop\C++学习笔记\image-20230105225557630.png)

队列容器允许从一端新增元素，从另一端移除元素

队列中只有队头和队尾的才可以被外界使用，因此队列不允许有遍历行为

队列中进数据称为 --- **入队** `push `

队列中出数据称为 --- **出队** `pop`



生活中举例：排队打饭，吃饭拉屎(粗俗)。





#### queue常用接口

功能描述：队列容器常用的对外接口



构造函数：

- `queue<T> que;`   //queue采用模板类实现，queue对象的默认构造形式

- `queue(const queue &que);`  //拷贝构造函数

赋值操作：

- `queue& operator=(const queue &que);`  //重载 等号 运算符

数据存取:

- `push(elem);`  // 入队操作。往队尾插入数据
- `pop();`  //出队操作。移除队头的第一个元素（先入先出）  。。这点区别于 双端数组deque 容器;
- `front();`  //返回第一个元素
- `back();`  //返回最后一个元素

大小操作：

- `empty();`  //判断堆栈是否为空
- `size();`  //返回栈的大小



```c++
class Person {
public:
	Person(string name, int age) {
		this->name = name;
		this->age = age;
	}
	string getName() {
		return name;
	}
	int getAge() {
		return age;
	}
private:
	string name;
	int age;
};

//queue队列的使用
void test02() {

	queue<Person> deq;
	Person p1("唐僧", 100);
	Person p2("猴子", 1000);
	Person p3("猪王", 800);
	Person p4("和尚", 300);

	deq.push(p1);
	deq.push(p2);
	deq.push(p3);
	deq.push(p4);

	//拷贝构造
	queue<Person> que2(deq);
	//使用重载等号运算符
	deque<Person> que3;
	//que3 = que2;   报错!!!! 没有匹配到 "=" 运算符。问题在于这里的模板是自定义数据类型

	cout << "出队前队列的大小: " << deq.size() << endl;
	while (!deq.empty())
	{
		//返回名称及年龄
		cout << " 队列中头部的--- 姓名: " << deq.front().getName() << " 年龄：" << deq.front().getAge() << endl;
		cout << " 队列中尾部的--- 姓名: " << deq.back().getName() << " 年龄：" << deq.back().getAge() << endl;
		
		//出队
		deq.pop();
	}
	cout << " 最后队列中的大小为" << deq.size() << endl;
}

//测试queue 的 “=” 的重载使用
void test() {
	queue<string> deq1;
	deq1.push("10");
	deq1.push("20");
	deq1.push("30");
	deq1.push("40");

	queue<string> que2;
	que2 = deq1;  //测试重载运算符：得到结论，自定义数据类型的queue 队列容器不能使用重载的"=" 运算符
}
```































