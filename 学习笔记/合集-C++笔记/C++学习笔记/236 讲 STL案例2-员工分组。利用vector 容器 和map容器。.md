### 236讲 STL案例2-员工分组。利用vector 容器 和map容器。

![image-20230219155257312](C:\Users\14163\Desktop\C++学习笔记\image-20230219155257312.png)



- 头文件和代码文件写在一起的，这个值得看一下。

```c++
#include<iostream>
#include <string>
#include <vector>
#include <map>	
#include<ctime>

using namespace std;
/*
公司招聘10个员工 （ABCDEFGHIJ）,10名员工进入公司后，需要指派在哪个工位工作。
员工信息有：姓名  工资组成；  部门分为：策划、美术、研发
随机给10名员工分配部门和工资
通过multimap 进行信息的插入，  key(部门编号)   value(员工)
分部门现实员工信息。
*/

class Worker {
public:
	string name;
	int salary;
};
enum depart {
	designed,article,research
};

void creatWorker(vector<Worker> &v)
{
	for (int i = 0; i < 10; i++)
	{
		Worker worker;
		string name = "ABCDEFGHIJ";
		string nameSeed = "员工";
		worker.name = nameSeed + name[i];

		worker.salary = rand() % 10001 + 10000;

		//cout << worker.name << "    " << worker.salary << endl;  //测试代码。
		v.push_back(worker);
	}
}

//设置部门，分别让vWorker 随机分配到里面
void Department(vector<Worker> v,multimap<int,Worker> &m)
{
	//使用一个随机数代替三个部门。0  1  2
	
	for (int i = 0 ; i < 10 ; i ++)
	{
		//遍历vector 容器，通过上方的i来实现迭代器的偏移
		vector<Worker>::iterator it = v.begin() + i;
		int departID = rand() % 3;
		//向map 中插入部门ID  和 员工
		m.insert(make_pair(departID, *it));

	}
	//m.insert(make_pair(depart::designed, *(v.begin())));

}

//单次打印每个部门的人数，主要为了实现代码的复用。
void printSinglePart(multimap<int, Worker>::iterator it, depart departID, int departCount)
{
	//取得迭代器，可以在for循环中让迭代器动起来
	//multimap<int, Worker>::const_iterator mit = m.find(depart::article);
	multimap<int, Worker>::iterator end = it;

	for (int i = 0; i < departCount; i++)
	{
		end++;
	}

	//下方代码用于计数。
	//int start = (*it).first;

	if (departCount != 0)
	{
		for (; it != end;it++)
		{
			cout << "部门：" << it->first << "\t姓名： " << it->second.name << "\t薪水" << it->second.salary << endl;
			  //让multimap的迭代器跑起来
		}
	}

}



//遍历部门，分别打印出各个部门的部门id 和 员工信息
void printWorker(multimap<int,Worker> &m)
{

	cout << "article部门：" << endl;
	multimap<int, Worker>::iterator mit = m.find(depart::article);
	
	int articleCount = m.count(depart::article);
	printSinglePart(mit, depart::article,articleCount);

	cout << "designed部门：" << endl;
	mit = m.find(depart::designed);
	int designedCount = m.count(depart::designed);
	printSinglePart(mit, depart::designed, designedCount);

	cout << "research部门：	" << endl;
	mit = m.find(depart::research);
	int researchCount = m.count(depart::research);
	printSinglePart(mit, depart::research, researchCount);

	/*    下方代码复用在printSinglePart()函数中了。
	//取得迭代器，可以在for循环中让迭代器动起来
	multimap<int, Worker>::const_iterator mit = m.find(depart::article);


	int departCount = m.count(depart::article);

	//下方代码用于计数。
	int start = (*mit).first;   //这个有问题，不应该是first，first要么是0 1 2，这就不对
	int end = start + departCount;
	if (departCount != 0)
	{
		for (; start < end ;start++ )
		{
			cout << "部门：" << mit->first << "\t姓名： " << mit->second.name << "\t薪水" << mit->second.salary << endl;
			mit++;  //让multimap的迭代器跑起来
		}
	}
	*/
}



void main()
{
	//随机数种子:
	srand((unsigned int)time(NULL));
	
	vector<Worker> vWorker;
	creatWorker(vWorker);
	multimap<int, Worker> m;
	Department(vWorker,m);
	printWorker(m);
	

	//测试代码
	multimap<int, Worker>::iterator mit;
	mit = m.begin();
	//cout <<  mit->first <<(*mit).second.name << "  " << mit->second.salary << endl;

}

```

> 自我总结：
>
> - 迭代器可以单独用来传参，传过去的迭代器，也能直接使用(而不用传原来的容器)。





























































