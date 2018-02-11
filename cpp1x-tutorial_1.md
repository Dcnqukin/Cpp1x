##C++1x_tutorial_Learning
---
#一、被弃用的特性
* 不允许字符串字面值常量赋值给一个char *, 需要通过const char * 或者 auto替代。、
---
#二、语言可用性的强化
* nullptr作为新引入的关键字，专门区分空指针和0(在传统C++中，空指针NULL和0视为同一种东西);
* nullptr的类型为nullptr_t，能隐式转换为任何指针或成员指针的类型，也能和他们进行相等或不等的比较。
* constexpr 让用户显式的声明函数或对象构造函数在编译器会成为常数。
* constexpr 可以使用递归(C++1x)
```
constexpr int fibonacci(const int n){
	return n == 1 || n == 2 ? 1 : fibonacci(n - 1) + fibonacci(n - 2);
}
```
* constexpr 在C++14及以上的版本中，可以在内部使用局部变量、循环和分支等简单语句，下列代码无法通过C++11的编译:
```
constexpr int fibonacci(const int n){
	if(n == 1) return 1;
	if(n == 2) return 2;
	return fibonacci(n - 1) + fibonacci(n - 2);
}
```
---
+ 类型推导：C++11引入了auto和decltype这两个关键字实现了类型推导，让编译器来操心变量的类型。
	- auto改变了迭代器的写法:
	```
	for(vector<int>::const_iterator itr = vec.cbegin(); itr != vec.cend(); ++itr)
	```
	而有了auto之后：
	```
	for(auto itr = vec.cbegin(); itr != vec.cend(); ++itr);
	```
	- 但是auto不可以用于函数传参， 也不可以用于推导数据类型。此时我们应选择template来进行相关的实现。
	- decltype关键字是为了解决auto关键字只能对变量进行类型推导的缺陷而出现的。它的用法和sizeof很相似:
	```
	decltype(表达式)
	```
+ 尾返回类型、auto与decltype配合
	- 从下面这个例子看起：
	```
	template<typename R, typename T, typename U>
	R add(T x, U y){
		return x+y;
	}
	```
	上面这种写法使得程序员在使用该函数时面临明确指出返回类型的情形。C++11对此提出的解决方案为：
	```
	template<typename T, typename U>
	auto add(T x, U y) -> decltype(x + y){
		return x + y;
	}
	```
	此时，C++11引入了一个叫做尾返回类型，利用auto关键字将返回类型后置。
+ 区间迭代
	- 基于范围的for循环
		C++11引入了基于范围的迭代写法，拥有了能够写出像Python一样简洁的循环语句：
		```
		int array[] = {1, 2, 3, 4, 5};
		for(auto &x : array){
			std::cout << x << std::endl;
		}
		```
		最常用的std::vector遍历将从原来的样子：
		```
		std::vector<int> arr(5, 100);
		for(std::vector<int>::iterator i = arr.begin(); i != arr.end(); ++i)
		{
			std::cout << *i << std::endl;
		}
		```
		变得非常简单：
		```
		for(auto &i : arr){
			std::cout << i << std::endl;
		}
		```