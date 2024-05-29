# C++ 基礎知識加強

加強一些 C++ 與其他語言不同之處。

## 特別資料型別

- `bool` : boolean 型別
- `wchar_t` : wide characters (16 bits) 字元，用來補足原本的 `char` 型別大小不足的問題，以支援 Unicode
- 固定大小整數型別，避免了傳統 C 語言的 `int` 等型別大小不固定的問題，例如：
    - `int64_t`: 64-bit 整數
    - `uint32_t`: 32-bit 無號整數
    - 完整清單： [https://en.cppreference.com/w/cpp/types/integer](https://en.cppreference.com/w/cpp/types/integer)

## 取得型別的 Max & Min

- 方法一: 使用內建常數
    - `INT32_MAX` (需要引入 `stdint.h`)
- 方法二:  `std::numeric_limits<T>::min()`
    - `T` 可以換成任意內建型別

## 變數宣告修飾子

- Storage Class
    - `auto` : 跟代表自動推導的 `auto` 不同，限制變數只能被宣告的 scope 內的程式碼存取，預設情況下 local variable 前都會自動加上 `auto`
      - Example： `auto int a = 1`
    - `register` : 提示 compiler 將變數存在 CPU register (C++11 之後 deprecated)
    - `static` :
        - 用在 local variable 會讓這個 local variable 的值與記憶體空間一直保留到程式結束
        - 用在 global variable 會限制該 variable 的 scope 在所在的 file 中
        - 用在 class member 會讓該 member shared 在所有 object 中
    - `extern` : 代表變數定義在其他檔案內，linking 時才會引入
        - Example: `extern int num`
        - 說明：[https://learn.microsoft.com/en-us/cpp/cpp/extern-cpp?view=msvc-170](https://learn.microsoft.com/en-us/cpp/cpp/extern-cpp?view=msvc-170)
    - `mutable` : 只能用在 class member 上，當物件的變數被指定為 `const` 時，該 class member 仍能夠被外界修改。
- 全域變數宣告時會自動初始化為 `NULL` / `0`

## Type Qualifiers

- `const` : 常數
- `volatile` : 防止存取此變數的動作因為 compiler optimization 的關係被調換位置
    - 同時也限制 CPU 不能快取此變數在 CPU cache (multi-threading 時很重要)
- `restrict` : 限制只有此指標變數可以指向該記憶體位置
    - 屬於 C99 標準，非 C++ 標準
    - [https://wucodingroad.blogspot.com/2017/05/Cplusplus-restrict.html](https://wucodingroad.blogspot.com/2017/05/Cplusplus-restrict.html)

## Reference (參考)

Reference 是 C++ 引入給變數設定「別名」的功能，跟 pointer 關鍵差異如下：

- Null 值：Pointer 允許 `NULL`，reference 不行
- 修改指向位置：Pointer 的變數可以途中修改成指向其他記憶體位置，reference 不行
- 初始化：Reference 初始化時一定要設定指向的變數，pointer 沒有限制

宣告方式：

```cpp
// declare simple variables
int    i;
double d;

// declare reference variables
int&    r = i;
double& s = d;
```

Reference 可以從 function 被 return：

```cpp
double& setValues( int i ) {
   return vals[i];   // return a reference to the ith element
}
```

但要注意不能 return local variable 的 reference

## Arrays

- 宣告方式： `double a[10];`
- 多維陣列： `int threedim[5][10][4];`
- 初始化： `double balance[] = {1000.0, 2.0, 3.4, 17.0, 50.0};`
- 初始化二維陣列：
    
    ```cpp
    int a[3][4] = {
       {0, 1, 2, 3} ,   /*  initializers for row indexed by 0 */
       {4, 5, 6, 7} ,   /*  initializers for row indexed by 1 */
       {8, 9, 10, 11}   /*  initializers for row indexed by 2 */
    };
    ```
    
- Array 的變數其實就是 pointer：
    
    ```cpp
    double *p;
    double balance[10];
    
    p = balance;
    ```
    
    - `balance[4]` 等價於 `*(balance + 4)`
- Array 作為 paramater 傳進 function 方法有兩種常見方式：
    - Method 1: 用 pointer
        
        ```cpp
        void myFunction(int *param) {
           // ...
        }
        ```
        
    - Method 2: 用 unsized array
        
        ```cpp
        void myFunction(int param[]) {
           // ...
        }
        ```
        
- 從 function return array 只有一種做法，就是回傳 pointer：
    
    ```cpp
    int * myFunction() {
       // ...
    }
    ```
    

## C++ String

C++ 有內建 String 型別，所以可以不使用 C-style string (也就是字元陣列)

使用方式：

```cpp
#include <string>
using std::string;

string s = "aabbcc";
s.size(); // return 6
s.find("bb"); // return 2
s += "123"; // s = aabbcc123
```

字面值直接使用 string 型別（省掉一個轉換步驟，需要 `using namespace std::literals`）：

```cpp
using namespace std::literals;
string s3_2 = "hello world"s;
```

基本操作：

```cpp
os << s // 輸出 s 
is >> s // 輸入 s
s.empty() // 檢查 s 是否為空
s.size() // s 目前長度
s[n] // 直接取得 s 的第 n 個元素
s.at(n) // 同樣取得第 n 個元素，但會檢查邊界
s1 + s2 // 把 s1 加 s2 取得新的字串
s1.append(s2) // 把 s2 加到 s1 後面，不會產生新字串
s1 += s2 // 等價於 s1.append(s2)
s1 = s2 //　拷貝複製 s2
s1 != s2 // 比較 s1 和 s2 是否相同
<, <=, ==, >=, > // 做大小比較，以字典排序
```

完整 C++ String 教學： [https://tigercosmos.xyz/post/2023/06/c++/std-string-beginner/](https://tigercosmos.xyz/post/2023/06/c++/std-string-beginner/)

## Standard Input/Output

C++ 有以下幾個常用的 input/output streams

- `cin` : 標準輸入
- `cout` : 標準輸出
- `cerr` : 標準錯誤（但沒有 buffer）
- `clog` : 標準錯誤（但有 buffer）