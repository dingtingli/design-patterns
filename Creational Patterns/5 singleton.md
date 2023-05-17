# Singleton（单例模式）


## 极客时间笔记


什么是单例模式？



为什么要使用单例？

如何使用单例模式？

单例存在哪些问题？

单例与静态类的区别？

有何替代的解决方案？


什么是单例模式？

单例设计模式（Singleton Design Pattern）理解起来非常简单。一个类只允许创建一个对象（或者实例），那这个类就是一个单例类，这种设计模式就叫作单例设计模式，简称单例模式。

为什么要使用单例？


如何使用单例模式？

单例是通过私有构造函数和一个静态的只读的 instance 成员来实现的。

* 构造函数需要是 private 访问权限的，这样才能避免外部通过 new 创建实例；
* 考虑对象创建时的线程安全问题；
* 考虑是否支持延迟加载；
* 考虑 getInstance() 性能是否高（是否加锁）。


1."饿汉式"Eager Initialization

```c#
using System.Threading;

public class IdGenerator
{
    private long id = 0;
    private static readonly IdGenerator instance = new IdGenerator();

    private IdGenerator() { }

    public static IdGenerator Instance
    {
        get { return instance; }
    }

    public long GetId()
    {
        return Interlocked.Increment(ref id);
    }
}

// IdGenerator使用举例
long id = IdGenerator.Instance.GetId();
```

在示例代码中，IdGenerator 类被加载的时机有两种可能：

当 IdGenerator.Instance 代码被执行的时候。这是因为 Instance 是 IdGenerator 类的一个静态属性，在访问这个属性的时候，IdGenerator 类会被加载。这也是你示例中的情况。

如果在其他地方访问了 IdGenerator 的其他静态成员（例如静态方法或静态字段）。

举个例子，假如你在另一个类中访问了 IdGenerator 的静态方法（假设有这样一个静态方法）：

```c#
public class AnotherClass
{
    public void SomeMethod()
    {
        IdGenerator.SomeStaticMethod();
    }
}
```

在这种情况下，当 SomeMethod 方法被调用，并执行到 IdGenerator.SomeStaticMethod() 这一行代码时，IdGenerator 类就会被加载。

值得注意的是，由于 IdGenerator 类的实例是在类的静态成员 instance 中初始化的，所以实际上，这个实例在类被加载的时候就已经创建好了。也就是说，无论何时你通过 IdGenerator.Instance 访问到这个实例，你都会得到同一个已经创建好的实例，这就实现了单例模式。

如果在你的代码中，第一次访问 IdGenerator 类的静态成员是在 IdGenerator.Instance.GetId() 这行代码，那么 IdGenerator 类会在这个时刻被加载，并且它的所有静态成员（包括 instance）会在此刻被初始化。

2. 懒汉式 Lazy Initialization

```csharp
using System.Threading;

public class IdGenerator
{
    private long id = 0;
    private static IdGenerator instance;
    private static readonly object lockObject = new object();

    private IdGenerator() {}

    public static IdGenerator Instance
    {
        get
        {
            lock(lockObject)
            {
                if (instance == null)
                {
                    instance = new IdGenerator();
                }
            }
            return instance;
        }
    }

    public long GetId()
    {
        return Interlocked.Increment(ref id);
    }
}

// IdGenerator使用举例
long id = IdGenerator.Instance.GetId();
```

