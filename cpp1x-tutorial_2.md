#C++1x_tutorial_learning
---
##二、语言可用性的强化
+ 初始化列表
	- C++11首先把初始化列表的概念绑定到了类型上，并将其称之为std::initializer_list，
		允许构造函数或其他函数像参数一样使用初始化列表，这为类对象的初始化与
		普通数组和POD的初始化方法提供了统一的桥梁。
		如：
		```
		#include <initializer_list>
		class Magic {
		public:
			Magic(std::initializer_list<int> list);
		};
		Magic magic = {1, 2, 3, 4, 5};
		std::vector<int> v = {1, 2, 3, 4};
		```
	- 这种构造函数被叫做初始化列表构造函数，具有这种构造函数的类型将在初始化时被特殊关照。
		初始化列表除了用在对象构造上，也可以将其作为普通函数的形参。
+ 模板增强
	- 外部模板
		- 传统C++中，模板只有在使用时才会被编译器实例化。这导致了重复实例化而产生编译时间的增加。
		而且，并没有能力去通知编译器不要触发模板实例化。
		- C++11引入了外部模板，扩充了原来的强制编译器在特定位置实例化模板的语法，
		使得能够显式地告诉编译器何时可以进行模板的实例化：
		```
		template class std::vector<MagicClass>;		//强行实例化
		extern template class std::vector<MagicClass>;//不在该编译文件中实例化模板
		```
	- C++11使得连续的>>能够通过编译(不被当做右移运算符来处理)。
	- 类型别名模板
		- 模板是用于产生类型的，在传统C++中，typedef可以为类型定义一个新的名称，但是没有办法为模板定义一个新的名称。因为模板不是类型，
		- 在C++11中，使用using引入了下面这种形式的写法，并且同时支持对传统typedef相同的功效：
		> 通常我们使用typedef定义别名的语法是：typedef 原名称 新名称; 
		> ，但是对函数指针等别名的定义语法却不相同，这通常给直接阅读造成了一定程度的困难。
		```
		typedef int (*process)(void *); //定义了一个返回类型为int，参数为void* 的函数指针类型，名字叫做process
		using process = void(*)(void *);//同上，更加直观
		using NewType = SuckType<std::vector, std::string>;
		```
	- 默认模板参数
		- 在C++11中提供了一种便利，可以指定模板的默认参数：
		```
		template<typename T = int, typename U = int>
		auto add(T x, U y) -> decltype(x + y){
			return x + y;
		}
		```
	- 变长模板参数
		- C++11中允许任意个数、任意类别的模板参数，同时也不需要再定义时将参数的个数固定。
		```
		template <typename ... Ts> class Magic;
		```
		模板类Magic的对象，能够接受不受限制个数的typename作为模板的形式参数。
		```
		class Magic<int,
					std::vector<int>,
					std::map<std::string,
							std::vector<int>>> darkMagic;
		```
		同时允许存在个数为0的模板参数：```class Magic<>nothing;```
		- 如果不希望产生的模板参数个数为0，可以手动定义至少一个模板参数。
		- 在模板参数中能使用...表示不定长模板参数，同时在函数参数表示中，也可以用...代表不定长参数。如：
		```
		template<typename... Args> void printf(const std::string &str, Args... args);
		```
		- 定义变长模板参数后，可用sizeof...来计算参数的个数：
		```
		template<typename... Args>
		void magic(Args... args){
			std::cout << sizeof...(args) << std::endl;
		}
		```
		- 对参数进行解包，有两种经典的处理手法：
			1.递归模板函数
			```
			#include <iostream>
			template<typename T>
			void printf(T value){
				std::cout << value << std::endl;
			}
			template<typename T, typename... Args>
			void printf(T value, Args... args){
				std::cout << value << std::endl;
				printf(args...);
			}
			int main(){
				printf(1, 2, "123", 1.1);
				return 0;
			}
			```
			2.初始化列表展开
+ 面向对象增强
	- 委托构造
		> C++11引入了委托构造的概念，这使得构造函数可以在同一个类中一个构造函数调用另一个构造函数，从而达到简化代码的目的：
		```
		class Base{
		public:
			int value1;
			int value2;
			Base(){
				value1 = 1;
			}
			Base(int value) : Base(){	//委托Base()构造函数
				value2 = 2;
			}
		};
		int main(){
			Base b(2);
			std::cout << b.value1 << std::endl;
			std::cout << b.value2 << std::endl;
		}
		```
	- 继承构造
		> 在传统C++中，构造函数如果需要继承是需要将参数一一传递的，这将导致效率低下。
		> C++11利用关键字using引入了继承构造函数的概念：
		```
		class Base{
		public:
			int value1;
			int value2;
			Base(){
				value1 = 1;
			}
			Base(int value) : Base(){	//委托Base()构造函数
				value2 = 2;
			}
		};
		class Subclass : public Base{
		public:
			using Base::Base;
		};
		int main(){
			Base s(3);
			std::cout << s.value1 << std::endl;
			std::cout << s.value2 << std::endl;
		}
		```
	- 显式虚函数重载
		> 在传统C++中，经常容易发生意外重载虚函数的事情。
		```
		struct Base{
			virtual void foo();
		};
		struct SubClass: Base{
			void foo();
		};
		```
		> 两种情况：1.一种是程序员只是想加入一个具有相同名字的函数。
		> 2.当基类的虚函数被删除后，子类拥有旧的函数就不再重载该虚拟函数并摇身一变变成为一个普通的类方法。
		
		- 采用两个关键字来防止上述情形的发生override 和 final。
		1. override：当重载虚函数时，引入override关键字将显式地告知编译器进行重载，编译器将检查基函数是否存在这样的虚函数。
		2. final：是为了防止类被继续继承以及终止虚函数继续重载引入的。
	- 显式禁用默认函数
		> C++11提供了一种解决方案，允许显式的声明采用或拒绝编译器自带的函数。
		```
		class Magic{
		public:
			Magic() = default; //显式声明使用编译器生成的构造
			Magic& operator=(const Magic&) = delete; //显式声明拒绝编译器生成构造
			Magic(int magic_number);
		}
		```
+ 强类型枚举
	> 在传统C++中，枚举类型并非类型安全，枚举类型会被视作整数，则会让两种完全不同的枚举类型可以进行直接的比较。
	> C++11引入了枚举类(enumeration class)，并使用enum class的语法进行声明：
	```
	enum class new_enum : unsigned int {
		value1,
		value2,
		value3 = 100,
		value4 = 100
	};
	```
	> 定义的枚举实现了类型安全。相同枚举值之间如果指定的值相同，那么可以进行比较：
	```
	if (new_enum::value3 == new_enum::value4){
		std::cout << "new_enum::value3 == new_enum::value4" << std::endl;
	}
	```
	