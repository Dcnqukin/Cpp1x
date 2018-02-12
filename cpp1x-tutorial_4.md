#C++11_tutorial_learning
---
##四、对标准库的扩充：新增容器
+ std::array
	1. 引入std::array 而不是直接使用 std::vector的原因？
	> std::array 保存在栈内存中， std::vector 保存在堆内存中。
	2. 已经有了传统数组，为什么要用 std::array?
	> 使用std::array科研使得代码变得更加现代，而且封装了一些操作函数，同时还能够友好地使用标准库中地容器算法。
	> std::array 会在编译时创建一个固定大小地数组， std::array不能够被隐式地转换成指针。
+ std::forward_list
	> std::forward_list使用单向链表进行实现，提供了O(1)复杂度的元素插入，不支持快速随机访问，也不提供size()方法。当不需要双向迭代时，具有比std::list更高的空间利用率。
+ 无序容器
	> 传统C++中的有序容器std::map / std::set,通过红黑树进行实现，插入和搜索的平均复杂度均为O(log(size))。
	> 无序容器中的元素是不进行排序的，内部通过Hash表实现，插入和搜索元素的平均复杂度为O(constant)。
	> C++11提供的两组无序容器分别为```std::unordered_map / std::unordered_multimap```, ```std::unordered_set / std::unordered_multiset```。
+ 元组std::tuple
	> 关于元组的使用有三个核心的函数：
	1. std::make_tuple: 构造元组
	2. std::get: 获得元组某个位置的值
	3. std::tie: 元组拆包
```
#include <tuple>
#include <iostream>
auto get_student(int id)
{
// 返回类型被推断为 std::tuple<double, char, std::string>
if (id == 0)
return std::make_tuple(3.8, 'A', "张三");
if (id == 1)
return std::make_tuple(2.9, 'C', "李四");
if (id == 2)
return std::make_tuple(1.7, 'D', "王五");
return std::make_tuple(0.0, 'D', "null");
// 如果只写 0 会出现推断错误, 编译失败
}
int main()
{
auto student = get_student(0);
std::cout << "ID: 0, "
<< "GPA: " << std::get<0>(student) << ", "
<< "成绩: " << std::get<1>(student) << ", "
<< "姓名: " << std::get<2>(student) << '\n';
double gpa;
char grade;
std::string name;
// 元组进行拆包
std::tie(gpa, grade, name) = get_student(1);
std::cout << "ID: 1, "
<< "GPA: " << gpa << ", "
<< "成绩: " << grade << ", "
<< "姓名: " << name << '\n';
}
```
	> std::get除了使用常量获取元组对象外，C++14增加了使用类型来获取元组中的对象。
	
	- 运行期索引
		> 由于std::get<>依赖于一个编译期的常量，所以下列做法是非法的:
		```
		int index = 1;
		std::get<index>(t);
		```
		> 可以使用boost库中的boost::variant配合变长模板参数实现该操作。
	- 元组合并与遍历
		> 元组的合并可以通过std::tuple_cat实现