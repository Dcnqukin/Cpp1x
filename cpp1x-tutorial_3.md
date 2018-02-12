#C++1x_tutorial_learning
---
##三、语言运行期的强化
+ Lambda表达式
	> Lambda表达式实际上提供了一个类似匿名函数的特性，而匿名函数则是在需要一个函数，
	> 但是又不想费力去命名一个函数的情况下使用的。
	- Lambda表达式基础
	Lambda表达式的基本语法如下：
	```
	[捕获列表](参数列表) mutable(可选) 异常属性 -> 返回类型{
		//函数体
	}
	```
	捕获列表，可以将其理解为参数的一种类型，lambda表达式内部函数体在默认情况下是不能够使用函数体外部的变量的，
	此时捕获列表可以起到传递外部数据的作用。
	捕获列表的种类：
	1.值捕获
	> 与参数传值类似，值捕获的前提是变量可以拷贝，不同之处在于，被捕获的变量在lambda表达式被创建时拷贝，而非调用时才拷贝：
	```
	void learn_lambda_func_1(){
		int value_1 = 1;
		auto copy_value_1 = [value_1]{
			return value_1;
		}
		value_1 = 100;
		auto stored_value_1 = copy_value_1();
	}
	```
	2.引用捕获
	与引用传参类似，引用捕获保存的是引用，值会发生变化。
	```
	void learn_lambda_func_2(){
		int value_2 = 1;
		auto copy_value_2 = [&value_2] {
			return value_2;
		};
		value_2 = 100;
		auto stored_value_2 = copy_value_2();
	}
	```
	3.隐式捕获
	> 捕获提供了lambda表达式对外部值进行使用的功能，捕获列表的最常用的四种形式可以是：
	- []空捕获列表
	- [name1, name2, ...]捕获一系列变量
	- [&]引用捕获，让编译器自行推导捕获列表
	- [=]值捕获，让编译器执行推导应用列表
	
	4.表达式捕获(c++14)

	
---
+ 右值引用
	> 右值引用是C++11引入的与Lambda表达式齐名的重要特性之一。它的引入解决了C++中大量的历史遗留问题。
	> 使得函数对象容器std::function成为了可能。
	
	- 左值、右值的纯右值、将亡值、右值
		1.左值(lvalue, left value)，赋值符号左边的值。左值即是表达式后依然存在的持久对象。
		2.右值(rvalue, right value)，表达式结束后就不再存在的临时对象。
		3.纯右值(prvalue, pure rvalue)，要么是纯粹的字面量如```true```，要么是求值结果相当于字面量或匿名临时对象，如1+2。
			> 非引用返回的临时变量、运算表达式产生的临时变量、原始字面量、Lambda表达式都属于纯右值。
		4.将亡值(xvalue, expiring value)，是C++11为了引入右值引用而提出的概念，也就是即将被销毁、却能够被移动的值。
	
	- 右值引用和左值引用
		> 如果想要获取一个将亡值，需要用到右值引用的申明：```T&&```，其中 T 是类型。
		> C++11提供了```std::move```这个方法将左值参数无条件的转换为右值，有了它我们能够方便得获得一个右值临时对象。
	- 移动语义
		> 传统C++通过拷贝构造函数和赋值操作符为类对象设计了拷贝/复制的概念。但没有区分[移动]和[拷贝]的概念，造成了大量的数据移动，浪费时间和空间。
		> 右值引用可以解决这两个概念的混淆问题。
		```
		#include <iostream>
		class A{
		public:
			int *pointer;
			A():pointer(new int(1)) {std::cout << "构造" << pointer << std::endl;}
			A(A& a):pointer(new int(*a.pointer)) { std::cout << "拷贝" << pointer << std::endl; }
			A(A&& a):pointer(a.pointer) {a.pointer = nullptr; std::cout<< "移动" << pointer << std::endl;}
			~A(){ std::cout << "析构" << pointer << std::endl;
				  delete pointer;}
		};
		
		A return_rvalue(bool test){
			A a,b;
			if(test) return a;
			else return b;
		}
		int main() {
			A obj = return_rvalue(false);
			std::cout << "obj:" << std::endl;
			std::cout << obj.pointer << std::endl;
			std::cout << *obj.pointer << std::endl;
			
			return 0;
		}
		```
		> 在上述代码中：
			1.首先会在```return_rvalue```内部构造两个A对象。于是获得两个构造函数的输出；
			2.函数返回后，将产生一个将亡值，被A的移动构造(A(A&&))引用，从而延长生命周期，并将这个右值中的指针拿到，保存到了obj中，而将亡值的指针被设置为nullptr，防止了这块内存区域被销毁。
	- 完美转发
		> 引用坍缩规则：允许我们对引用进行引用，既能左引用，又能右引用。
		> 同时遵循以下规则：
		| 函数形参类型 | 实参参数类型 | 推导后函数形参类型 |
		|:------------:|:------------:|:------------------:|
		|	T&		   |	左引用	  |			T&		   |
		|	T&		   |	右引用	  |			T&		   |
		|	T&&		   |	左引用	  |			T&		   |
		|	T&&		   |	右引用	  |			T&&		   |
		> 无论模板参数是什么类型的引用，当且仅当实参类型为右引用时，模板参数才能被推导为右引用类型。
		- 完美转发基于上述规律产生的。即让我们在传递参数的时候，保持原来的参数类型。为了解决这个问题，应该使用std::forward来进行参数的转发：
		- 无论传递参数为左值还是右值，普通传参都会将参数作为左值进行转发，所以```std::move```总会接受到一个左值，
		  从而转发调用了```reference(int&&)```输出右值引用。
		- 唯独```std::forward```即没有造成任何多余的拷贝，同时完美转发(传递)了函数的实参给了内部调用的其他函数。