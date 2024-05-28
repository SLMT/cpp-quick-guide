# C++ Concurrency

## `std::thread`

C++11 引入了標準的 thread 函式庫，用法如下：

```cpp
#include <iostream>
#include <thread>

int main() {
    std::thread t([](){
        std::cout << "hello world." << std::endl;
    });
    t.join();
    return 0;
}
```

## Locks

- `std::mutex` :  lock 的基本元件，提供 lock 與 unlock 功能建立 critical section
    - 不建議直接使用
- `std::lock_guard` : `std::mutex` 的 wrapper，在創建時自動 lock mutex，解構時自動 unlock
- `std::unique_lock` : `std::mutex` 的 wrapper，創建時自動 lock，中途可以選擇 unlock 並再度 lock，解構時會自動 unlock
    - 建議優先選擇用這個
    - 範例：
        
        ```cpp
        #include <iostream>
        #include <mutex>
        #include <thread>
        
        int v = 1;
        
        void critical_section(int change_v) {
            static std::mutex mtx;
            std::unique_lock<std::mutex> lock(mtx);
            // do contention operations
            v = change_v;
            std::cout << v << std::endl;
            // release the lock
            lock.unlock();
        
            // during this period, others are allowed to acquire v
        
            // start another group of contention operations
            // lock again
            lock.lock();
            v += 1;
            std::cout << v << std::endl;
            // Unlock when it disappears
        }
        
        int main() {
            std::thread t1(critical_section, 2), t2(critical_section, 3);
            t1.join();
            t2.join();
            return 0;
        }
        ```
        

## `std::future`

一個用來建立 task，等待未來回收結果的 utility 元件。

詳情：[https://changkun.de/modern-cpp/zh-cn/07-thread/#7-3-期物](https://changkun.de/modern-cpp/zh-cn/07-thread/#7-3-%E6%9C%9F%E7%89%A9)

## `std::condition_variable`

提供 `wait()` 與 `notify_all()` 與 `notify_one()` 的功能，讓 thread 可以進入等待狀態，其他 thread 也可以叫醒等待的 thread

範例：[https://changkun.de/modern-cpp/zh-cn/07-thread/#7-4-条件变量](https://changkun.de/modern-cpp/zh-cn/07-thread/#7-4-%E6%9D%A1%E4%BB%B6%E5%8F%98%E9%87%8F)

## `std::atomic`

如果要對一個變數進行 atomic 操作，除了使用 `std::mutex` 之外，也可以使用 `std::atomic` 。 `std::mutex` 實際上在編譯成 CPU 指令其實非常複雜，但對一個只是單純需要 atomic 操作的狀況來說 overkilled 了。

```cpp
#include <atomic>
#include <thread>
#include <iostream>

std::atomic<int> count = {0};

int main() {
    std::thread t1([](){
        count.fetch_add(1);
    });
    std::thread t2([](){
        count++;        // identical to fetch_add
        count += 1;     // identical to fetch_add
    });
    t1.join();
    t2.join();
    std::cout << count << std::endl;
    return 0;
}
```

因為其是一個 template，可以容納任何類型的資料，因此 C++ 有提供額外的 API 檢查該類型資料是否有保障原子性：

```cpp
#include <atomic>
#include <iostream>

struct A {
    float x;
    int y;
    long long z;
};

int main() {
    std::atomic<A> a;
    std::cout << std::boolalpha << a.is_lock_free() << std::endl;
    return 0;
}
```