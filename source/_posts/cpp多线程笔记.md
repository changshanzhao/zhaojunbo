---
title: cpp多线程笔记
date: 2023-10-07 13:30:55
tags:
  - 教程
  - 学习笔记
categories:
	- 教程
---

# 创建线程

```c++
void print(std::string msg){
    std::cout << msg << std::endl;
    return;
}
std::thread test1(print,"Hello C++");
```

第一个参数传函数名，第二个参数传参

等待子线程结束后再往下运行,join是阻塞的

```cpp
test1.join();
```

分离线程，主线程可以结束，子线程在后台可以持续运行

```cpp
test1.detach();
```

判断该线程是否可以调用join或者detach，返回一个bool值

```cpp
test1.joinable();
```

传引用参数时使用std::ref

```cpp
void foo(int& x){}
int a = 1
std::thread t(foo, std::ref(a));
```

一种写法

```cpp
class A{
public:
	void foo(){
		std::cout << "Hello" << std::endl;
	}
};
std::shared_ptr<A> a = std::make_shared<A>();
std::thread t(&A::foo, a);
```

我的理解是&A::foo是函数指针，和函数名一致，&a是函数参数对应this指针

# 数据共享问题

## 互斥锁mutex，lock和unlock

```cpp
int a = 0;
std::mutex mtx;
void func(){
    for(int i = 0; i < 1000; i++){
        mtx.lock();
        a += 1;
        mtx.unlock();
    }

}
int main(){
    std::thread t1(func);
    std::thread t2(func);
    t1.join();
    t2.join();
    std::cout << a << std::endl;
}
```

## 死锁

```cpp
std::mutex m1,m2;
void func_1{
    m1.lock();
    m2.lock();
    m1.unlock();
    m2.unlock();
}
void func_2{
    m2.lock();
    m1.lock();
    m1.unlock();
    m2.unlock();
}
std::thread t1(func_1);
std::thread t2(func_2);
```

t1获取到m1之后，t2获取到m2，两个都在等待对方释放，卡死了，所以改进方案是

```cpp
void func_1{
    m1.lock();
    m2.lock();
    m1.unlock();
    m2.unlock();
}
void func_2{
    m1.lock();
    m2.lock();
    m1.unlock();
    m2.unlock();
}
```

顺序一致就好了

## lock_guard

当构造函数被调用的时候，该互斥量会被自动锁定。

当析构函数被调用时，该互斥量会自动解锁。

该对象不能被复制和移动，只能在局部作用域中使用。

```cpp
std::mutex mtx;
void func(){
    for(int i = 0; i < 1000; i++){
    	std::lock_guard<std::mutex> lg(mtx);
    	a++;
    }
}
```

## unique_lock

相比lock_guard额外增加延迟加锁，条件变量，超时等操作

```cpp
std::timed_mutex mtx; //时间锁
std::unique_lock<std::timed_mutex> ul(mtx); //这种写法和lock_guard基础功能一致
//这么写不自动加锁，只是构造函数有变化，至于析构，还是自动析构的
std::unique_lock<std::timed_mutex> ul(mtx, std::defer_lock); 
//尝试5秒钟内加锁，5秒钟获取不到返回false，获取到了返回true
ul.try_lock_for(std::chrono::second(5));
```

## 单例的线程安全问题

饿汉模式类定义的时候就实例化，本身是线程安全，不需要加锁。饿汉，就是不管三七二十一，这个类编译的时候就实例化，不考虑后面用不用得上。用法如下所示，直接在类外实例化，即初始化即实例化。

懒汉模式，就是第一次用到类的示例才实例化。注意到，前面new语句放在GetInstance函数，这个函数需要外界调用才会发挥作用，然后new一个实例。形象点记忆，懒汉就是不到万不得已就不会去实例化类，我用不上那就先不管。懒汉模式是不安全的实现方式，是线程不安全的，需要加锁。

解决1：加智能锁。不使用智能锁采用普通锁，lock()和unlock()这种加锁与解锁的方式，若空间没开辟成功直接返回，这时锁没解开就生成死锁。而智能锁不管有没有成功开辟空间，出了作用域的同时调用lock（）的析构函数把这个锁释放了，不会造成死锁。

解决2：使用原子锁。

解决3：使用once_flag

```cpp
// 饿汉模式，每次访问的地址都是同一个，但是该不能被释放掉
class Log{
public:
	Log(const Log& log) = delete;
    Log& operator=(const Log& log) = delete;
    static Log& GetInstance(){
        return *log;
    }
private:
    static Log *log;
    Log(){};
}
Log* Log::log = new Log;
// 懒汉模式
//定义全局锁
mutex mtx;
class Singleton
{
private:
    static Singleton* singleton;
    Singleton()
    {
        cout << "Singleton的构造" << endl;
    }
    ~Singleton()
    {
        cout <<"Singleton的析构" << endl;
    }
public:
    static Singleton* getInstance()
    {
        lock_guard<mutex> lock(mtx);
        //mtx.lock();  普通锁
        if(singleton == nullptr)
        {
            singleton = new Singleton;
        }
        //mtx.unlock();
        return singleton;
    }
    void showInfo()
    {
        cout << this << endl;
    }
    //专门设计一个释放堆区空间的逻辑
    static void destroy()
    {
        //不要把这个逻辑放在析构函数中，将会出现无限递归
        if(singleton != nullptr)
        {
            delete singleton;
        }
    }
};
Singleton* Singleton::singleton = nullptr;
int main()
{
    Singleton* s1 = Singleton::getInstance();
    Singleton* s2 = Singleton::getInstance();
    Singleton* s3 = Singleton::getInstance();
    s1->showInfo();
    s2->showInfo();
    s3->showInfo();
    Singleton::destroy();
    return 0;
}
// once_flag
class Singleton
{
private:
    static Singleton* singleton;
    static std::once_flag once;
    Singleton()
    {
        cout << "Singleton的构造" << endl;
    }
    ~Singleton()
    {
        cout <<"Singleton的析构" << endl;
    }
    static void init() {
        if(singleton == nullptr)
        {
            singleton = new Singleton;
        }
    }
public:
    static Singleton* getInstance()
    {
        std::call_once(once, init);
        return singleton;
    }
    void showInfo()
    {
        cout << this << endl;
    }
    //专门设计一个释放堆区空间的逻辑
    static void destroy()
    {
        //不要把这个逻辑放在析构函数中，将会出现无限递归
        if(singleton != nullptr)
        {
            delete singleton;
        }
    }
};
Singleton* Singleton::singleton = nullptr;
```

### 选择原则

懒汉：在访问量比较小，采用懒汉模式。到有需要的时间才实例化对象，那它就不会提前占据内存空间，代价就是后续每次访问都会判断是否为空，增加时间成本。这是以时间换空间。

饿汉：在访问量比较大，或者可能访问的线程比较多，采用饿汉模式。就算没用上实例对象，也会进行实例化，这是要占据一定内存的。但在后面需要使用的时候，就不需要判断之类语句，所以非常快速。这就是以空间换时间。

## condition_variable

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <string>
#include <condition_variable>
#include <queue>
std::queue<int> g_queue;
std::condition_variable g_cv;
std::mutex mtx;
void Producer() {
    for(int i = 0; i < 10; i++) {
        std::unique_lock<std::mutex> lock(mtx);
        g_queue.push(i); // 生产者往队列里加任务
        // 通知消费者来取任务
        g_cv.notify_one(); // 通知一个线程去取任务
        std::cout<<"Producer: "<<i<<std::endl;
    }
    std::this_thread::sleep_for(std::chrono::microseconds(100));
}
void Consumer() {
    while (1) {
        std::unique_lock<std::mutex> lock(mtx);
        // 如果队列为空，就要等待
        // bool is_empty = g_queue.empty();
        // g_cv.wait(lock, !is_empty);
        g_cv.wait(lock, [](){return !g_queue.empty();});
        int value = g_queue.front();
        g_queue.pop();
        std::cout<<"Consumer: "<<value<<std::endl;
    }
}
int main()
{
    std::thread t1(Producer);
    std::thread t2(Consumer);
    t1.join();
    t2.join();
    return 0;
}
```

# 跨平台线程池

```cpp
#include <functional>
class ThreadPool{
public:
    ThreadPool(int numThreads):stop(false){
        for(int i = 0; i < numThreads; i++){
            threads.emplace_back([this]{
                while(1){
                    std::unique_lock<std::mutex> lock(mtx);
                    condition.wait(lock, [this]{
                        return !tasks.empty()||stop;
                    });
                    if(stop && tasks.empty()){
                        return;
                    }
                    std::function<void()> task(std::move(tasks.front()));
                    tasks.pop();
                    lock.unlock();
                    task();
                }
            });
        }
    }
    ~ThreadPool(){
        {
            std::unique_lock<std::mutex> lock(mtx);
            stop = true;
        }
        condition.notify_all();
        for(auto& t : threads){
            t.join();
        }
        /*
        函数模板里面右值引用就是万能引用，std::forward完美转发（如果是左值引用就转化成左值引用，是右值引用就转化成右值引用）。使用std::bind可以将可调用对象和参数一起绑定，绑定后的结果使用std::function进行保存，并延迟调用到任何我们需要的时候。
        */
        template<class F, class... Args>
        void enqueue(F&& f,Args&&... args){
            std::function<void()> task = std::bind(std::forward<F>(f), std::forward<Args>(args)...);
            {
            	std::unique_lock<std::mutex> lock(mtx);
            	tasks.emplace(std::move(task));
            }
            condition.notify_one();
        }
    }
private:
    std::vector<std::thread> threads;
    std::queue<std::function<void()>> tasks;
    std::mutex mtx;
    std::condition_variable condition;
    bool stop;
}

// 执行
ThreadPool pool(4);
for(int i = 0; i < 10; i++) {
    pool.enqueue([i]{
        std::cout << "task : " << i << "is running" << std::endl;
        std::this_thread::sleep_for(std::second(1));
        std::cout << "task : " << i << "is done" << std::endl;
    })
}
```

#  异步

std::async()异步操作的线程不用自己手动创建

```cpp
std::future<int> future_result = std::async(std::launch::async, func)
cout << func() << endl;
cout << future_result.get() << endl;
```

这种就需要，但是就有阻塞了()

```cpp
std::packaged_task<int()> task(func);
auto future_result = task.get_future();//auto的类型就是std::future<int>
std::thread t1(std::move(task));
cout << func() << endl;
t1.join();
cout << future_result.get() << endl;
```

promise，线程间的数据传递

```cpp
int func(std::promise<int> & f) {
    f.set_value(1000);
}
int main() {
    std::promise<int> f;
    auto future_result = f.get_future();
    std::thread t1(func, std::ref(f));
    t1.join();
    cout << future_result.get() << endl;
    return 0;
}
```

# 原子操作

原子变量自带线程安全，不用加锁。

```cpp
#include <atomic>
std::atomic<int> shared_data = 0;
shared_data.load();//输出
shared_data.store();//修改
```

相比于加锁，效率更高，时间更短。

