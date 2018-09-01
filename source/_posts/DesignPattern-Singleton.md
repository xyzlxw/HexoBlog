---
title: 单件 (SINGLETON)
tags: 设计模式
categories: 软件工程
---

单件模式是一个简单的模式，要正确的实现此模式仍需要考虑很多细节，由于单件模式不符合对象
的单一职责原则，论坛上对是否应该使用单件模式的争议很多，因单件模式适用的场景很少，在
程序设计中又极容易被滥用，如果没有急迫并且有意义的需要时，应该避免使用这个模式。

<!-- more -->

## 目的

* 确保一个类只有一个实例 (Ensure a class only has one instance)
* 提供一个全局访问点 (Provide a global point of access to it)

## 动机

有一些对象可能只需要一个，例如：线程池(thread pool), 缓存(cache), 注册表对象，日志
对象，打印机驱动程序对象，窗口管理器等。

如何可以保证一个类只有一个实例？全局变量可以被访问，但不能防止被实例化多次。一个办法是
让类自身保存它的唯一实例，这个类提供一个访问该实例的方法，即单件模式。

单件模式提供了全局访问点，同全局变量一样方便，但避免了全局变量的缺点。比如，将对象赋值
给全局变量，通常在程序开始时创建对象（JVM是用到时创建），如果此对象非常耗费资源，而程序
中没有使用到它，则形成浪费。通过使用单件，可以在需要时创建这个对象。

## 参与者 (Participants)

* Singleton
  
  - 定义Instance方法，允许客户访问它的唯一实例。
  - 负责创建它自己的唯一实例

## 协作 (Collaborations)

* 客户只能通过Singleton的Instance方法访问Singleton的实例。

## 效果

### 好处

* 跨平台：使用合适的中间件（例如：RMI），可以把单件模式扩展为跨多个JVM或多个计算机工作。
* 适用于任何类：将类的构造函数变为私有的，然后增加相应的静态函数和变量，就可实现单件。
* 可以透过派生创建：给定一个类，可以创建它的一个单件子类
* 延迟求值（Lazy evaluation）：如果单件从未使用，就绝不创建它。

### 代价

* 摧毁方法未定义：没有好的方法去destroy一个单件或解除其职责。此问题在C++中尤其严重。
* 不能继承：从单件类继承的子类并不是单件，需要添加静态函数和变量从而变成单件。
* 效率问题：当未使用静态函数的静态变量，每次调用Instance方法都会执行一个if语句。
* 不透明性：单件的使用者知道在调用单件模式，因为需要调用Instance方法。

## 实现

### C++的相关问题

* 是否支持延迟求值可依照具体情况而定
* 考虑默认拷贝构造函数
* 私有化构造函数，确保只通过Instance方法得到实例
* 线程安全
  由于C++ 2011已经支持线程安全，参考下面的实例代码。
  对于C++ 2003及之前版本，线程安全可采用如下方式：

  - 在Instance方法中增加同步锁
  - 在Instance方法中使用Double-Checked Locking
  - 使用类的静态变量

### Java的相关问题

* Java中实现单件模式需要私有的构造器，一个静态方法，一个静态变量
* 确定性能和资源上的限制，然后小心选择适当的方案，以解决多线程问题（最好认定所有程序
  都是多线程的）
* 只有Java 5或以上版本,才支持双重检查锁 (Double-Checked Locking)
* 小心，如果你使用多个类加载器，可能导致单件失效而产生多个实例
* 如果使用JVM 1.2或之前版本，你必须建立单件注册表，以免垃圾回收期将单件回收

## 代码

### C++ 2003 单线程版本

```C++
class Singleton
{
public:
    static Singleton& Instance()
    {
        static Singleton instance;
        return instance;
    }
private:
    Singleton() {}
    Singleton(Singleton const&);
    void operator=(Singleton const&);
};
```
### C++ 2011 线程安全版本

```C++
class Singleton
{
public:
    static Singleton& Instance()
    {
        static Singleton instance;
        return instance;
    }
    Singleton(Singleton const&) = delete;
    void operator=(Singleton const&) = delete;
private:
    Singleton() {}
};
```

### Java静态实例化

```Java
public class Singleton
{
    private static Singleton _instance = new Singleton();
    private Singleton() {}
    public  static Singleton Instance() { return _instance; }
}
```

### Java延迟实例化 - 单线程

```Java
public class Singleton
{
    private static Singleton _instance;
    private Singleton() {}
    public  static Singleton Instance() 
    {
        if (_instance == null)
           _instance = new Singleton();
        return _instance; 
    }
}
```

### Java延迟实例化 - 多线程

多线程同步的问题:

* 增加synchronized同步后，不会有两个线程同时进入方法，同步会令执行效率下降100倍。
* 只有第一次执行此方法时需要同步，对象创建后，同步成为一个影响性能的累赘。
* 如果资源创建和运行方面的负担不繁重，可考虑“静态实例化”来替代同步。

```Java
public class Singleton
{
    private static Singleton _instance;
    private Singleton() {}
    public  static synchronized Singleton Instance() 
    {
        if (_instance == null)
           _instance = new Singleton();
        return _instance; 
    }
}
```

### Java双重检查锁(Double-Checked Locking) - 多线程

《设计模式精解》指出：Java中双重检查锁无效果，理由如下：

* Java内存管理指出，只在两个线程在同一个对象上进行同步时，它才会保证一个线程 B 能够
  看到另一个线程 A 的改变，从而使得线程 A 得 synchronized 块对线程 B 变为原子，即
  要么什么都不做，要么完全改好。 
* Java并不是一种按照源代码顺序执行的语言。Java对编译器和虚拟机的要求时满足 as-if-serial:
  也就是只要它能够达到和严格顺序执行一样的效果，指令执行的顺序可以随便安排。

《Head First设计模式》指出：Java 5或之后版本可通过 volatile 支持双重检查锁

```Java
public class Singleton
{
    private volatile static Singleton _instance;
    private Singleton() {}
    public  static Singleton Instance() 
    {
        if （_instance == null)
        {
            synchronized (Singleton.class)
            {
                if （_instance == null)
                    _instance = new Singleton();
            }
        }
        return _instance; 
    }
}
```

### Java接口应用示例 

```Java
public interface IDatabase
{
    void Write();
}
public class UserDatabase implements IDatabase
{
    private static UserDatabase _instance = new UserDatabase();
    private UserDatabase() {}
    public  static UserDatabase Instance() { return _instance; }
    public void Write() 
    {
        // Some Implementation
    }
}
```

### C++注册表应用示例 

* Singleton.h

```C++
class IDatabase
{
public: virtual void Write() {}
};
class Singleton : public IDatabase
{
protected:
    Singleton() {}
public:
    Singleton(Singleton const&) = delete;
    void operator= (Singleton const&) = delete;
    static Singleton* Instance();
    static void Register(const char* name, Singleton* instance);
    static Singleton* Lookup(const char* name);
private:
    static Singleton* _instance;
};
```

* Singleton.cpp

```C++
static std::map<std::string, Singleton*> _registry;
Singleton* Singleton::_instance;
Singleton* Singleton::Instance()
{
    if (!_instance)
    {
        const char* name = getenv("SINGLETON");
        _instance = Lookup(name);
    }
    return _instance;
}
void Singleton::Register(const char* name, Singleton* instance)
{
    _registry.insert(std::make_pair(name, instance));
}
Singleton* Singleton::Lookup(const char* name)
{
    return _registry[name];
}
```

* MySingleton.h

```C++
class MySingleton : public Singleton
{
public:
    MySingleton() { Singleton::Register("MySingleton", this); }
    virtual void Write()
    {
        // Some Implementation
    }
};
```

* client.cpp

```C++
static MySingleton theSingleton;
int main()
{
    _putenv("SINGLETON=MySingleton");
    Singleton::Instance()->Write();
    return 0;
}
```

### C++工厂应用示例

```C++
class MazeFactory
{
public:
    static MazeFactory* Instance();
protected:
    MazeFactory() {}
private:
    static MazeFactory* _instance;
};
class BombedMazeFactory : public MazeFactory
{
};
class EnchantedMazeFactory : public MazeFactory
{
};
MazeFactory* MazeFactory::Instance()
{
    if (_instance == 0)
    {
        const char* mazeStyle = getenv("MAZESTYLE");
        if (strcmp(mazeStyle, "bombed") == 0) 
            _instance = new BombedMazeFactory();
        else if (strcmp(mazeStyle, "enchanted") == 0) 
            _instance = new EnchantedMazeFactory();
		// ...
        else 
            _instance = new MazeFactory();
    }
    return _instance;
}
```

## 相关模式 (Related Patterns)

如下模式或可用Singleton模式实现：

* Abstract Factory
* Builder
* Prototype

## 参考 (Reference)

* [设计模式：可复用面向对象软件的基础](https://book.douban.com/subject/1052241/)
* [设计模式精解](https://book.douban.com/subject/1219912/)
* [敏捷软件开发](https://book.douban.com/subject/1140457/)
* [Head First 设计模式](https://book.douban.com/subject/2243615/)
* [Design Patterns](https://book.douban.com/subject/1099305/)

