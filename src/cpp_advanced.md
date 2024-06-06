# C++ 進階語法

## Try-Catch

C++ 可以 throw exception，而且可以用 try-catch 抓住。與 Java 最大的不同在於 C++ 的 throw 可以丟出任何種類的資料，catch 只要寫清楚資料的型別就好：

```cpp
#include <iostream>
using namespace std;

double division(int a, int b) {
   if( b == 0 ) {
      throw "Division by zero condition!";
   }
   return (a/b);
}

int main () {
   int x = 50;
   int y = 0;
   double z = 0;
 
   try {
      z = division(x, y);
      cout << z << endl;
   } catch (const char* msg) {
     cerr << msg << endl;
   }

   return 0;
}
```

C++ STD 有內建一系列的 exception，可以在下面網頁看到清單：

[https://cplusplus.com/reference/exception/exception/?kw=exception](https://cplusplus.com/reference/exception/exception/?kw=exception)

另外官方也鼓勵開發者繼承 `exception` class 來製造自己的錯誤種類，其中只需要 override `what()` (顯示錯誤資訊的 function) 就好。

## Template

Template 就是 Jave 的 generic，只是語法稍微不同，功能也沒那麼強而已。

定義 function template 的範例如下：

```cpp
template <typename T>
T minimum(const T& lhs, const T& rhs)
{
    return lhs < rhs ? lhs : rhs;
}
```

定義 class template 的範例如下：

```cpp
template <class T>
class Stack { 
   private: 
      vector<T> elems;    // elements 

   public: 
      void push(T const&);  // push element 
      void pop();               // pop element 
      T top() const;            // return top element 
      
      bool empty() const {      // return true if empty.
         return elems.empty(); 
      } 
}; 

template <class T>
void Stack<T>::push (T const& elem) { 
   // append copy of passed element 
   elems.push_back(elem);    
} 
```

注意當 class method 的定義沒有寫在 class 的宣告內時，也要額外宣告 `template <class T>` 的標籤。

MSFT 官網有針對 template 更加詳盡的介紹： [https://learn.microsoft.com/zh-tw/cpp/cpp/templates-cpp?view=msvc-170](https://learn.microsoft.com/zh-tw/cpp/cpp/templates-cpp?view=msvc-170)