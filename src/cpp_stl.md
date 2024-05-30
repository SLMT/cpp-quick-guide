# C++ STL

Standard Template Library (STL) 是 C++ 之中最重要的函式庫，包含多項重要的元件：

- Containers： 各種儲存與操作資料的容器
- Algorithms： 各種高效的演算法
- Iterators： 用來依照某些邏輯存取容器內資料的存取器
- Function Objects： 提供將 function 做為參數傳入 function 的方法，可以幫助客製化演算法的行為
- Adapters： 可以用來修改 STL 內其他元件行為的工具

## Containers

STL container 清單： [https://cplusplus.com/reference/stl/](https://cplusplus.com/reference/stl/)

需要注意的 containers：

- `forward_list` ： 只有向前指針的 linked list，比 `list` (具備前後指針) 使用更少的空間
- `priority_queue` ：  用 heap 實作的 queue，最大的東西會在最前面
    - 一般用法：
        
        ```cpp
        int arr[6] = { 10, 2, 4, 8, 6, 9 };
        priority_queue<int> pq;
         
        // pushing array sequentially one by one
        for (int i = 0; i < 6; i++) {
            pq.push(arr[i]);
        }
         
        // printing priority queue
        cout << "Priority Queue: ";
        while (!pq.empty()) {
            cout << pq.top() << ' ';
            pq.pop();
        }
        ```
        
    - 修改成最小在最前面：
        
        ```cpp
        int arr[6] = { 10, 2, 4, 8, 6, 9 };
        priority_queue<int, vector<int>, greater<int>> gquiz(arr, arr + 6);
        ```
        
    - 使用自製的 comparator：
        
        ```cpp
        class Foo { /* data */ };
        
        class Comparator
        {
        public:
            bool operator() (Foo, Foo)
            {
        		// Should the first one be in front of the second one?
                return true;
            }
        };
        
        int main()
        {
            std::priority_queue<Foo, std::vector<Foo>, Comparator> pq;
            return 0;
        }
        ```
        
- `set` ： sorted set，用紅黑樹實作
- `map` ： sorted map，實作與 `set` 相同
- `multiset` ： 跟 `set` 一樣是 sorted set，但可以存重複資料
- `multimap` ： 跟 `map` 一樣是 sorted map，但可以存重複的 key
    - 注意用 `find(key)` 只會拿到一個 key-value pair，如果想要抓出所有重複 key 的 pair，要使用 `equal_range(key)`
        - `equal_range(key)` 回傳的是一個 pair， `first` 是 iterator 的開頭（lower bound, inclusive）， `second` 是 iterator 的結尾 (upper bound, exclusive)

## Algorithms

常用 algorithms：

- `sort(begin, end)` ： 使用 IntroSort 進行排序
    - IntroSort：先嘗試使用 quick sort。 如果發現 partition 差太多，改成用 heap sort。 如果 size 夠小，改成用 insertion sort。
    - 基本用法：
        
        ```cpp
        int arr[] = {3, 5, 1, 2, 4};
        sort(arr, arr + 5);

        vector<int> vec {3, 5, 1, 2, 4};
        sort(vec.begin(), vec.end());
        ```
        
- `binary_search(begin, end, value)` ： 使用 binary search
- `reverse(begin, end)` ： 反轉順序

## Iterator

可以看 C++ Reference 官網的介紹，簡單明瞭： [https://cplusplus.com/reference/iterator/](https://cplusplus.com/reference/iterator/)

只要搞懂 Input, Output, Forward, Bidirectional, Random Access 這五種 iterator 的差異，跟他們的從屬關係就好。

## Function Objects (Functors)

直接看一例子，這個例子是寫一個 class 幫所有 array 所有的 element 加上想要的數字：

```cpp
#include <bits/stdc++.h>
using namespace std;

// A Functor
class increment
{
private:
	int num;
public:
	increment(int n) : num(n) { }

	// This operator overloading enables calling
	// operator function () on objects of increment
	int operator () (int arr_num) const {
		return num + arr_num;
	}
};

// Driver code
int main()
{
	int arr[] = {1, 2, 3, 4, 5};
	int n = sizeof(arr)/sizeof(arr[0]);
	int to_add = 5;

	transform(arr, arr+n, arr, increment(to_add));

	for (int i=0; i<n; i++)
		cout << arr[i] << " ";
}
```

主要是提供了改寫 `()` 行為的能力，所以讓一個 object 可以被像 function 一樣呼叫。

## 其他

### Pairs

C++ STL 提供了 pair 可以使用：

```cpp
// CPP program to illustrate Pair in STL
#include <iostream>
#include <utility>

using namespace std;

// Driver Code
int main()
{
	// defining a pair
	pair<int, char> PAIR1;

	// first part of the pair
	PAIR1.first = 100;
	// second part of the pair
	PAIR1.second = 'G';

	cout << PAIR1.first << " ";
	cout << PAIR1.second << endl;

	return 0;
}

```

- Pair 可以被比較，會先比第一個再比第二個