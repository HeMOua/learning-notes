# STL标准库

## string

当使用 C++ 标准库中的 `std::string` 类时，有许多常用的成员函数和方法可以帮助你对字符串进行操作。以下是一些常见的 `std::string` 成员函数：

1. **length() / size()：** 返回字符串的长度。

2. **empty()：** 检查字符串是否为空，返回 `true` 或 `false`。

3. **clear()：** 清空字符串，使其成为空字符串。

4. **c_str()：** 返回一个指向以 null 结尾的 C 风格字符串的指针。

5. **substr(start, length)：** 返回一个从 `start` 位置开始、长度为 `length` 的子字符串。

6. **append(str) / operator+=：** 将指定字符串添加到字符串的末尾。

7. **insert(pos, str)：** 在指定位置插入字符串。

8. **erase(pos, length)：** 从指定位置开始删除指定长度的字符。

9. **replace(pos, length, str)：** 用新的字符串替换指定位置开始的指定长度的字符。

10. **find(str, pos)：** 从指定位置开始搜索字符串，返回第一个匹配子串的位置。

11. **rfind(str, pos)：** 从指定位置开始反向搜索字符串，返回最后一个匹配子串的位置。

12. **find_first_of(str, pos)：** 从指定位置开始搜索字符串，返回第一个匹配参数中任一字符的位置。

13. **find_last_of(str, pos)：** 从指定位置开始反向搜索字符串，返回最后一个匹配参数中任一字符的位置。

14. **find_first_not_of(str, pos)：** 从指定位置开始搜索字符串，返回第一个不匹配参数中任一字符的位置。

15. **find_last_not_of(str, pos)：** 从指定位置开始反向搜索字符串，返回最后一个不匹配参数中任一字符的位置。

16. **compare(str)：** 比较字符串与给定字符串，返回比较结果。

17. **at(index) / operator[]：** 访问字符串中指定位置的字符。

18. **front()：** 返回字符串的第一个字符。

19. **back()：** 返回字符串的最后一个字符。

20. **resize(newSize)：** 调整字符串的大小为指定值，可能截断或填充字符。

21. **resize(newSize, fillChar)：** 调整字符串的大小为指定值，并用指定字符填充。

22. **push_back(c)：** 将字符添加到字符串末尾。

23. **pop_back()：** 删除字符串末尾的字符。

## vector

### 构造函数

+ `vector()`: 创建一个空vector
+ `vector(int nSize)`: 创建一个vector,元素个数为nSize
+ `vector(int nSize,const t& t)`: 创建一个vector，元素个数为nSize,且值均为t
+ `vector(const vector&)`: 复制构造函数
+ `vector(begin,end)`: 复制 `[begin,end)` 区间内另一个数组的元素到vector中

### 添加

- `void push_back(const T& x)`: 向量尾部增加一个元素X
- `iterator insert(iterator it,const T& x)`: 向量中迭代器指向元素前增加一个元素x
- `iterator insert(iterator it,int n,const T& x)`: 向量中迭代器指向元素前增加n个相同的元素x
- `iterator insert(iterator it,const_iterator first,const_iterator last)`: 向量中迭代器指向元素前插入另一个相同类型向量的 `[first,last)` 间的数据

### 删除

- `iterator erase(iterator it)`: 删除向量中迭代器指向元素
- `iterator erase(iterator first,iterator last)`: 删除向量中 `[first,last)` 中元素
- `void pop_back()`: 删除向量中最后一个元素
- `void clear()`: 清空向量中所有元素

### 遍历

- `reference at(int pos)`: 返回pos位置元素的引用
- `reference front()`: 返回首元素的引用
- `reference back()`: 返回尾元素的引用
- `iterator begin()`: 返回向量头指针，指向第一个元素
- `iterator end()`: 返回向量尾指针，指向向量最后一个元素的下一个位置
- `reverse_iterator rbegin()`: 反向迭代器，指向最后一个元素
- `reverse_iterator rend()`: 反向迭代器，指向第一个元素之前的位置

```c++
#include <vector>
#include <iostream>
 
using namespace std;
 
struct Point
{
	double x;
	double y;
	Point()
	{
		x = 0;
		y = 0;
	}
};
 
 
int main()
{
	vector<Point> m_testPoint;
	m_testPoint.clear();
	m_testPoint.shrink_to_fit();
 
	for (int i = 0; i<10; ++i)
	{
		Point temp;
		temp.x = i*i;
		temp.y = i*i;
		m_testPoint.push_back(temp);
	}
 
	//第一种遍历方式，下标
	cout << "第一种遍历方式，下标访问" << endl;
	for (int i = 0; i<m_testPoint.size(); ++i)
	{
 
		cout << m_testPoint[i].x << "	" << m_testPoint[i].y << endl;
	}
 
	//第二种遍历方式，迭代器
	cout << "第二种遍历方式，迭代器访问" << endl;
	for (vector<Point>::iterator iter = m_testPoint.begin(); iter != m_testPoint.end(); iter++)
	{
		cout << (*iter).x << "	" << (*iter).y << endl;
	}
 
	//第三种遍历方式，auto关键字
	cout << "C++11,第三种遍历方式，auto关键字" << endl;
	for (auto iter = m_testPoint.begin(); iter != m_testPoint.end(); iter++)
	{
		cout << (*iter).x << "	" << (*iter).y << endl;
	}
 
	//第四种遍历方式，auto关键字的另一种方式
	cout << "C++11,第四种遍历方式，auto关键字" << endl;
	for (auto i : m_testPoint)
	{
		cout << i.x << "	" << i.y << endl;
	}
 
	return 0;
}
```

### 判空

- `bool empty() const`: 判断向量是否为空，若为空，则向量中无元素

### 大小

- `int size() const`: 返回向量中元素的个数
- `int capacity() const`: 返回当前向量所能容纳的最大元素值
- `int max_size() const`: 返回最大可允许的vector元素数量值

## set

| 集合               | 底层实现 | 是否有序 | 数值是否可以重复 | 能否更改数值 | 查询效率 | 增删效率 |
| ------------------ | -------- | -------- | ---------------- | ------------ | -------- | -------- |
| std::set           | 红黑树   | 有序     | 否               | 否           | O(log n) | O(log n) |
| std::multiset      | 红黑树   | 有序     | 是               | 否           | O(logn)  | O(logn)  |
| std::unordered_set | 哈希表   | 无序     | 否               | 否           | O(1)     | O(1)     |

> **红黑树**是一种平衡二叉搜索树，所以key值是有序的，但key不可以修改，改动key值会导致整棵树的错乱，所以只能删除和增加。
>
> unordered_set 头文件 `#include <unordered_set>`

### 构造函数

+ `set()`：创建一个空set
+ `set {item1, item2, ...}`：创建时初始化
+ `set (const set&)`: 复制构造函数
+ `set (begin,end)`: 复制 `[begin,end)` 区间内另一个数组的元素到set中

### 常用方法

`std::set` 是 C++ 标准库中的一个容器，它实现了一个有序的集合。`std::set` 中的元素是唯一的且按照特定的排序顺序存储。以下是一些 `std::set` 常用的方法和操作：

1. **插入元素：**
   - `insert(value)`：向集合中插入一个值。
   - `emplace(args...)`：在集合中直接构造一个元素。

2. **删除元素：**
   - `erase(value)`：从集合中删除一个特定的值。
   - `erase(iterator)`：从集合中删除一个特定的迭代器指向的元素。
   - `clear()`：清空集合中的所有元素。

3. **查找元素：**
   - `find(value)`：查找集合中是否存在某个值，返回迭代器指向该值（如果存在）。
   - `count(value)`：返回集合中特定值的数量，由于 `std::set` 的特性，此值只能是 0 或 1。

4. **迭代：**
   - `begin()`：返回指向集合起始位置的迭代器。
   - `end()`：返回指向集合末尾位置的迭代器。
   - 可以使用迭代器遍历集合中的元素，比如使用循环 `for (auto it = set.begin(); it != set.end(); ++it)`。

5. **大小和空判断：**
   - `size()`：返回集合中元素的数量。
   - `empty()`：检查集合是否为空，返回 `true` 或 `false`。

6. **其它：**
   - `lower_bound(value)`：返回一个迭代器，指向集合中第一个不小于给定值的元素。
   - `upper_bound(value)`：返回一个迭代器，指向集合中第一个大于给定值的元素。

这些是 `std::set` 常用的一些方法和操作。需要注意的是，`std::set` 中的元素是自动有序排列的，且不允许有重复元素。在需要有序存储元素且不需要重复元素的场景中，`std::set` 是一个非常有用的容器。



## map

| 映射               | 底层实现 | 是否有序 | 数值是否可以重复 | 能否更改数值 | 查询效率 | 增删效率 |
| ------------------ | -------- | -------- | ---------------- | ------------ | -------- | -------- |
| std::map           | 红黑树   | key有序  | key不可重复      | key不可修改  | O(logn)  | O(logn)  |
| std::multimap      | 红黑树   | key有序  | key可重复        | key不可修改  | O(log n) | O(log n) |
| std::unordered_map | 哈希表   | key无序  | key不可重复      | key不可修改  | O(1)     | O(1)     |

使用