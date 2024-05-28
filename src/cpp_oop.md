# C++ OOP

## Class Definition

基本例子：

```cpp
#include <iostream>
 
using namespace std;
 
class Box {
   public:
      double length;
      void setWidth( double wid );
      double getWidth( void );
 
   private:
      double width;
};
 
// Member functions definitions
double Box::getWidth(void) {
   return width ;
}
 
void Box::setWidth( double wid ) {
   width = wid;
}
 
// Main function for the program
int main() {
   Box box;
 
   // set box length without member function
   box.length = 10.0; // OK: because length is public
   cout << "Length of box : " << box.length <<endl;
 
   // set box width without member function
   // box.width = 10.0; // Error: because width is private
   box.setWidth(10.0);  // Use member function to set it.
   cout << "Width of box : " << box.getWidth() <<endl;
 
   return 0;
}
```

## Initialization List

在 constructor 上可以加上特殊的 list 初始化 properties，如下：

```cpp
C::C( double a, double b, double c): X(a), Y(b), Z(c) {
   ....
}
```

## Copy Constructor

C++ 規定每一個 class 都要有一個 copy constructor 具備可以複製資料的能力，形式如下：

```cpp
classname (const classname &obj) {
   // body of constructor
}
```

如果沒有的話 C++ 會自動創建一個預設的，做法是會呼叫所有 member variable 的 copy constructor 實現 deep copy。

## Friend Functions & Classes

Class 內可以定義 friend function 與 class 如下：

```java
class Box {
   double width;
   
   public:
      double length;
      friend void printWidth( Box box );
      void setWidth( double wid );
			friend class ClassTwo;
};
```

這樣寫代表外面的 `printWidth` 與 `ClassTwo` 就可以直接存取 Box 的變數 `width`

## Static Members

C++ 的 class 可以定義 static member，但變數的初始化必須要寫在 class definition 外面：

```java
// foo.h
class foo
{
    private:
        static int i;
};

// foo.cpp
int foo::i = 0;
```

其他特性基本跟 Java 的 static 相同。

## 繼承性

與 Java 的關鍵差異：

- 可以繼承多個 class
    - 類似 Rust 的 trait implementation
- 繼承時可以設定 access modifier
    - 例如下面的例子將 base class 設定為 public
        
        ```java
        // Base class
        class Shape {
           // ...
        };
        
        // Derived class
        class Rectangle: public Shape {
           // ...
        };
        ```
        
    - 這個用途是可以修改原本 base class 內的 member 存取性質，但只能改得更嚴格，不能放得更鬆
        - 設定為 public ⇒ 不修改 base class 的 member
        - 設定為 protected ⇒ 原本是 public 的 member 都會變成 protected
        - 設定為 private ⇒ 原本是 public 與 protected 的 member 會變成 private

## **Operators Overloading**

C++ 的一大特色就是可以修改 operator 對 class 造成的效果 (Rust 也有)

假設有一個 `Box` class，修改 `+` 的效果方式如下：

```java
class Box {
   public:
      // ...
      
      // Overload + operator to add two Box objects.
      Box operator+(const Box& b) {
         Box box;
         box.length = this->length + b.length;
         box.breadth = this->breadth + b.breadth;
         box.height = this->height + b.height;
         return box;
   // ...
}
```

## Function Override & Virtual Functions

在 C++ 中，如果衍伸類別要改寫基底類別的 method，基底類別就必須要將 method 定義為 virtual：

```java
class Shape {
   protected:
      int width, height;

   public:
      Shape( int a = 0, int b = 0){
         width = a;
         height = b;
      }
      virtual int area() {
         cout << "Parent class area :" << width * height << endl;
         return width * height;
      }
};
class Triangle: public Shape {
   public:
      Triangle( int a = 0, int b = 0):Shape(a, b) { }

      int area () {
         cout << "Triangle class area :" << (width * height)/2 << endl;
         return (width * height / 2);
      }
};
```

這是因為 C++ 在 class 建立時，預設會用 static linkage 綁定 function 的定義，而這個會導致當一個 function 被呼叫時，「只會根據其 variable type 來判斷該呼叫哪一個 function」。也就是說，無論實際上物件是 `Shape` 還是 `Triangle` ，只要他是 `Shape` 的變數，就會呼叫到 `Shape` 的 `area()` 。

如果將 function 設定為 virtual，C++ 就會知道該將 function 改用 dynamic linkage 處理，他會根據當下 object 的實際類別來判斷該呼叫哪一個 function (使用 vtable)。

簡單的比較例子：

```java
class Base
{
  public:
            void Method1 ()  {  std::cout << "Base::Method1" << std::endl;  }
    virtual void Method2 ()  {  std::cout << "Base::Method2" << std::endl;  }
};

class Derived : public Base
{
  public:
    void Method1 ()          {  std::cout << "Derived::Method1" << std::endl;  }
    void Method2 () override {  std::cout << "Derived::Method2" << std::endl;  }
    // Note - override is optional; adding it to Method1 would result in an error
};

Base* basePtr = new Derived ();
// Note - constructed as Derived, but pointer stored as Base*

basePtr->Method1 ();  //  Prints "Base::Method1"
basePtr->Method2 ();  //  Prints "Derived::Method2"
```

來源：https://stackoverflow.com/questions/2391679/why-do-we-need-virtual-functions-in-c

## Abstract Methods & Abstract Class

C++ 並沒有 `abstract` 關鍵字，但是可以用別的方式做到相同效果。

只要單純宣告但不定義 virtual function，並且加上 `= 0` ，就稱之為 pure virtual function：

```java
virtual int area() = 0;
```

這等同於 Java 的 abstract method。

只要 class 中包含至少一個 pure virtual function，那麼該 class 就不能被實體化，因此效果等同於 abstract class。

## Class vs. Struct

C++ 的 class 與 struct 其實沒有太大的差別，只是在以下幾點：

- Class 的 member 預設是 private，struct 是 public
    - 繼承的時候也類似，class 預設會是 private 繼承
- Class 可以當作 template 的關鍵字用，struct 不行