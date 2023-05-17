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

懒汉式相对于饿汉式的优势是支持延迟加载。

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

instance 的初始化是在 Instance 属性的 get 访问器中进行的。只有当你显式调用 IdGenerator.Instance 时，instance 才会被初始化。如果你使用了 IdGenerator 类的其他成员，但没有调用 IdGenerator.Instance，那么 instance 就不会被初始化。


懒汉式这种方式的优点是如果单例对象没有被使用，就不会初始化，节省了资源。但缺点是需要考虑多线程环境下的线程安全问题。如果多个线程同时访问并试图创建单例对象，就需要适当的同步机制来保证只创建一个单例对象。


我们给 get 访问器方法加了一把大锁（lock），导致这个函数的并发度很低。量化一下的话，并发度是 1，也就相当于串行操作了。而这个函数是在单例使用期间，一直会被调用。如果这个单例类偶尔会被用到，那这种实现方式还可以接受。但是，如果频繁地用到，那频繁加锁、释放锁及并发度低等问题，会导致性能瓶颈，这种实现方式就不可取了。

3. “双重检查锁定”（double-checked locking）

