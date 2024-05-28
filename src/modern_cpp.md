# Modern C++ (After C++11)

## 棄用語法

- 字串 litertral 應該改用 `const char *` 而不是 `char *` 宣告
- `register` 效用被移除
- `bool` 不可以 `++`
- C 語言風格的轉換方式 `(type)` 被棄用，應該改用 `static_cast`、`reinterpret_cast`、`const_cast` 等方式轉換

## 新增語法

C++11：

- 指定 Null Pointer 時改用 `nullptr` 而不是 `NULL`
    - `NULL` 可能會被解析成 int 造成額外問題，[例子](https://changkun.de/modern-cpp/zh-cn/02-usability/#nullptr)
- 加入 `constexpr` 支援，用來表示某一個 expression 或 function 輸出的結果是一個常數
    - 編譯器會去驗證這點，並且可以以此來優化程式碼
    - 範例：
        
        ```cpp
        constexpr int fibonacci(const int n) {
            return n == 1 || n == 2 ? 1 : fibonacci(n-1)+fibonacci(n-2);
        }
        ```
        
        ```cpp
        constexpr int var_constexpr = 1 + 2 + 3;
        ```
        
- 提供可以使用 initialization list 初始化任意物件的能力（但需要實作對應的 constructor），[範例](https://changkun.de/modern-cpp/zh-cn/02-usability/#%E5%88%9D%E5%A7%8B%E5%8C%96%E5%88%97%E8%A1%A8)
- `auto` ：自動型別推導
    
    ```cpp
    auto i = 5;              // i 被推导为 int
    auto arr = new auto(10); // arr 被推导为 int *
    ```
    
    - `auto` 也常用於 template 指定 function return type 時，這樣就不需要額外定義另一個 type 變數：
        
        ```cpp
        // Old way
        template<typename R, typename T, typename U>
        R add(T x, U y) {
            return x + y;
        }
        
        // After C++11
        template<typename T, typename U>
        auto add(T x, U y){
            return x + y;
        }
        ```
        
- `decltype` ： 輸出變數是何種型別
    - 通常搭配 `std::is_same<T, U>` 使用，檢查是否兩個型別是同一個型別
        
        ```cpp
        auto x = 1;
        if (std::is_same<decltype(x), int>::value)
            std::cout << "type x == int" << std::endl;
        ```
        
- Ranged for loop
    
    ```cpp
    std::vector<int> vec = {1, 2, 3, 4};
    for (auto element : vec)
        std::cout << element << std::endl; // read only
    for (auto &element : vec) {
        element += 1;                      // writeable
    }
    ```
    
    - 宣告 ranged for loop 的原則：
        - Choose `auto x` when you want to work with copies.
        - Choose `auto &x` when you want to work with original items and may modify them.
        - Choose `auto const &x` when you want to work with original items and will not modify them.
- template 別名：
    
    ```cpp
    template<typename T, typename U>
    class MagicType {
    public:
        T dark;
        U magic;
    };
    
    template<typename T>
    using TrueDarkMagic = MagicType<std::vector<T>, std::string>;
    
    int main() {
        TrueDarkMagic<bool> you;
    }
    ```
    
- Template 變動變數清單：
    
    ```cpp
    template<typename... Ts>
    void magic(Ts... args) {
        std::cout << sizeof...(args) << std::endl;
    }
    
    magic(); // Print: 0
    magic(1); // Print: 1
    magic(1, ""); // Print: 2
    ```
    
    - 關於如何處理請見[範例](https://changkun.de/modern-cpp/zh-cn/02-usability/#%E5%8F%98%E9%95%BF%E5%8F%82%E6%95%B0%E6%A8%A1%E6%9D%BF)
- Delegate constructor：允許重複使用同一個 class 內的 constructor
    
    ```cpp
    class Base {
    public:
        int value1;
        int value2;
        Base() {
            value1 = 1;
        }
        Base(int value) : Base() { // delegate Base() constructor
            value2 = value;
        }
    };
    ```
    
- 繼承 constructor：使用 `using` 關鍵字來使用 base class 的 constructor
    
    ```cpp
    class Subclass : public Base {
    public:
        using Base::Base; // inheritance constructor
    };
    ```
    
- `override` : 避免複寫不存在的 virtual function
    
    ```cpp
    struct Base {
        virtual void foo(int);
    };
    struct SubClass: Base {
        virtual void foo(int) override; // legal
        virtual void foo(float) override; // illegal, no such virtual function
    };
    ```
    
- `final` : 阻止繼承或 override
    
    ```cpp
    struct Base {
        virtual void foo() final;
    };
    struct SubClass1 final: Base {
    }; // legal
    
    struct SubClass2 : SubClass1 {
    }; // illegal, SubClass1 has final
    
    struct SubClass3: Base {
        void foo(); // illegal, foo has final
    };
    ```
    
- 拒絕或明確要求預設實作（用於 constructor 或 operator 等等）：
    
    ```cpp
    class Magic {
        public:
        Magic() = default; // explicit let compiler use default constructor
        Magic& operator=(const Magic&) = delete; // explicit declare refuse constructor
        Magic(int magic_number);
    }
    ```
    
- `enum class`
    - 與 `enum` 不同處在於：
        - 不會預設可以轉換為 int
        - 多個 `enum class` 存在相同名稱的 variant 也不會產生名稱衝突
    
    ```cpp
    enum class new_enum : unsigned int {
        value1,
        value2,
        value3 = 100,
        value4 = 100
    };
    ```
    

C++17：

- 可以在 if 條件式宣告變數
    
    ```cpp
    if (const auto itr = std::find(vec.begin(), vec.end(), 3);
        itr != vec.end()) {
        *itr = 4;
    }
    ```
    
- 可以對 if statement 使用 `constexpr`
    
    ```cpp
    #include <iostream>
    
    template<typename T>
    auto print_type_info(const T& t) {
        if constexpr (std::is_integral<T>::value) {
            return t + 1;
        } else {
            return t + 0.001;
        }
    }
    int main() {
        std::cout << print_type_info(5) << std::endl;
        std::cout << print_type_info(3.14) << std::endl;
    }
    ```
    
- 宣告變數時可以解構資料，例如下面的 `[x, y, z]`：
    
    ```cpp
    #include <iostream>
    #include <tuple>
    
    std::tuple<int, double, std::string> f() {
        return std::make_tuple(1, 2.3, "456");
    }
    
    int main() {
        auto [x, y, z] = f();
        std::cout << x << ", " << y << ", " << z << std::endl;
        return 0;
    }
    ```
    
- Template 變動長度參數展開式：
    
    ```cpp
    #include <iostream>
    template<typename ... T>
    auto sum(T ... t) {
        return (t + ...);
    }
    int main() {
        std::cout << sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10) << std::endl;
    }
    ```
    

C++20：

- `auto` 可以用於 function signature：
    
    ```cpp
    int add(auto x, auto y) {
        return x+y;
    }
    
    auto i = 5; // infered as int
    auto j = 6; // infered as int
    std::cout << add(i, j) << std::endl;
    ```
    

## Lambda Function

類似 Java 的匿名函數，語法如下：

```
[capture list] (parameter list) mutable(optional) exception attribute -> return type {
// function body
}
```

其中 `[capture list]` 代表從 lambda function 外部帶入的變數，注意該變數是在建立 lambda function 的當下就會複製，而不是在呼叫時被複製。

```cpp
void lambda_value_capture() {
    int value = 1;
    auto copy_value = [value] {
        return value;
    };
    value = 100;
    auto stored_value = copy_value();
    std::cout << "stored_value = " << stored_value << std::endl;
    // At this moment, stored_value == 1, and value == 100.
    // Because copy_value has copied when its was created.
}
```

Capture list 的格式如下：

- `[]` 空清單
- `[name1, name2, ...]` 最常見，捕獲一系列變數
- `[&]` 捕獲外部參考（copy by reference），讓編譯器自行推斷要捕獲誰
- `[=]` 捕獲外部值（copy by value），讓編譯器自行推斷要捕獲誰

C++14 加入可以在 capture list 中寫 expression 的能力：

```cpp
#include <iostream>
#include <memory>  // std::make_unique
#include <utility> // std::move

void lambda_expression_capture() {
    auto important = std::make_unique<int>(1);
    auto add = [v1 = 1, v2 = std::move(important)](int x, int y) -> int {
        return x+y+v1+(*v2);
    };
    std::cout << add(3,4) << std::endl;
}
```

C++14 開始可以在 lambda 使用 `auto` ：

```cpp
auto add = [](auto x, auto y) {
    return x+y;
};

add(1, 2);
add(1.1, 2.2);
```

C++11 使用 `std::function` 統一了所有可呼叫的型別：

```cpp
#include <functional>
#include <iostream>

int foo(int para) {
    return para;
}

int main() {
    // std::function wraps a function that take int paremeter and returns int value
    std::function<int(int)> func = foo;

    int important = 10;
    std::function<int(int)> func2 = [&](int value) -> int {
        return 1+value+important;
    };
    std::cout << func(10) << std::endl;
    std::cout << func2(10) << std::endl;
}
```

另外也加入了 `std::bind` 與 `std::placeholder` 來預先將數值綁定在某些參數上：

```cpp
int foo(int a, int b, int c) {
    ;
}
int main() {
    // bind parameter 1, 2 on function foo,
    // and use std::placeholders::_1 as placeholder for the first parameter.
    auto bindFoo = std::bind(foo, std::placeholders::_1, 1,2);
    // when call bindFoo, we only need one param left
    bindFoo(1);
}
```

## `std::move`

C++11 加入了右值參考 `T &&` 的概念，用途是可以將一個即將被銷毀的數值轉移到另一個物件內。

`std::move` 提供了將一個左值轉換為右值參考的能力，一旦在 expression 中的右側出現了右值參考，C++ 便不會呼叫 copy constructor，而是改呼叫 move constructor (編譯器預設會實作一個)。

這個新的作法主要是盡可能地只複製 pointer，而不是整份資料，並同時將舊物件內的 pointer 清除（改成 `nullptr` ）。

`std::move` 最常見的用途就是用來避免大量的資料複製。

考慮以下例子：

```cpp
template <class T>
swap(T& a, T& b) {
    T tmp(a);   // we've made a second copy of a
    a = b;      // we've made a second copy of b (and discarded a copy of a)
    b = tmp;    // we've made a second copy of tmp (and discarded a copy of b)
}
```

使用 `std::move` 改寫：

```cpp
template <class T>
swap(T& a, T& b) {
    T tmp(std::move(a));
    a = std::move(b);   
    b = std::move(tmp);
}
```

如果 `T` 是一個資料結構，例如 `vector<int>` ，那裡面可能就包含大量資料，用第一版就會浪費大量的資源與時間。

Default move constructor 基本上跟 copy constructor 差不多，只是他不會複製指標指向的資料，而是單純將指標複製一份。同時也會修改原本 class 的 member，把資料都留空。

## Containers

- `std::array`
    - 威力加強版的 array，提供了一些 container 的額外功能：
    
    ```cpp
    std::array<int, 4> arr = {1, 2, 3, 4};
    
    arr.empty(); // check if container is empty
    arr.size();  // return the size of the container
    
    // iterator support
    for (auto &i : arr)
    {
        // ...
    }
    
    // use lambda expression for sort
    std::sort(arr.begin(), arr.end(), [](int a, int b) {
        return b < a;
    });
    
    // illegal, different than C-style array, std::array will not deduce to T*
    // int *arr_p = arr;
    ```
    
- `std::forward_list`
    - 只有單向的 linked list
- `unordered_set` / `unordered_map` 跟其他東西
    - Hash map 基底的實作，提供 O(1) 插入與查找
- `std::tuple`
    - `std::pair` 威力加強版，允許任意數量的組合
    - 常用 function
        - `std::get`: Get the value of a position in the tuple
        - `std::tie`: tuple unpacking
    
    ```cpp
    #include <iostream>
    #include <stdexcept>
    #include <string>
    #include <tuple>
     
    std::tuple<double, char, std::string> get_student(int id)
    {
        switch (id)
        {
            case 0: return {3.8, 'A', "Lisa Simpson"};
            case 1: return {2.9, 'C', "Milhouse Van Houten"};
            case 2: return {1.7, 'D', "Ralph Wiggum"};
            case 3: return {0.6, 'F', "Bart Simpson"};
        }
     
        throw std::invalid_argument("id");
    }
     
    int main()
    {
        const auto student0 = get_student(0);
        std::cout << "ID: 0, "
                  << "GPA: " << std::get<0>(student0) << ", "
                  << "grade: " << std::get<1>(student0) << ", "
                  << "name: " << std::get<2>(student0) << '\n';
     
        const auto student1 = get_student(1);
        std::cout << "ID: 1, "
                  << "GPA: " << std::get<double>(student1) << ", "
                  << "grade: " << std::get<char>(student1) << ", "
                  << "name: " << std::get<std::string>(student1) << '\n';
     
        double gpa2;
        char grade2;
        std::string name2;
        std::tie(gpa2, grade2, name2) = get_student(2);
        std::cout << "ID: 2, "
                  << "GPA: " << gpa2 << ", "
                  << "grade: " << grade2 << ", "
                  << "name: " << name2 << '\n';
     
        // C++17 structured binding:
        const auto [gpa3, grade3, name3] = get_student(3);
        std::cout << "ID: 3, "
                  << "GPA: " << gpa3 << ", "
                  << "grade: " << grade3 << ", "
                  << "name: " << name3 << '\n';
    }
    ```
    

## Smart Pointers

- `std::shared_ptr` : 自動計數並銷毀的 pointer
    
    ```cpp
    auto pointer = std::make_shared<int>(10);
    auto pointer2 = pointer; // reference count+1
    auto pointer3 = pointer; // reference count+1
    int *p = pointer.get();  // no increase of reference count
    
    std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl;   // 3
    std::cout << "pointer2.use_count() = " << pointer2.use_count() << std::endl; // 3
    std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl; // 3
    
    pointer2.reset();
    std::cout << "reset pointer2:" << std::endl;
    
    std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl;   // 2
    std::cout << "pointer2.use_count() = " 
        << pointer2.use_count() << std::endl;                // pointer2 has reset, 0
    std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl; // 2
    ```
    
- `std::unique_ptr` : 只允許一個變數的 pointer
    
    ```cpp
    #include <iostream> // std::cout
    #include <memory>
    
    int main()
    {
      auto pointer = std::make_unique<int>(10);  // make_unique, from C++14
      // auto pointer2 = pointer;                // copy is illegal
      auto pointer3 = std::move(pointer);        // move is OK
      std::cout << pointer << std::endl;
      std::cout << pointer3 << std::endl;
    }
    ```
    
- `std::weak_ptr` : 弱指針，不會引起 `shared_ptr` 計數增加
    - 主要是用來避免下圖的 cycle share 問題：
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/90130fb6-82ad-49ff-9656-f23eef83d45d/bc344694-84cb-4491-96f3-8d1d26059d53/Untitled.png)
        
    - 用法：直接 assign `shared_ptr` 給 `weak_ptr` 即可
        
        ```cpp
        std::shared_ptr<int> a = std::make_shared<int>(42);
        std::weak_ptr<int> b = a;
        ```
        

## 沒搞懂的東西

- `decltype(auto)`