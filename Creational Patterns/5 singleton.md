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


饿汉式不支持延迟加载，懒汉式有性能问题，不支持高并发。那我们再来看一种既支持延迟加载、又支持高并发的单例实现方式，也就是双重检测实现方式。


```csharp
using System.Threading;

public class IdGenerator
{
    private long id = 0;
    private static IdGenerator instance = null;
    private static readonly object lockObject = new object();

    private IdGenerator() { }

    public static IdGenerator Instance
    {
        get
        {
            if (instance == null)
            {
                lock (lockObject)
                {
                    if (instance == null)
                    {
                        instance = new IdGenerator();
                    }
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

我们首先通过if (instance == null)来检查实例是否已经创建。如果还没有创建，我们就通过synchronized（Java）或lock（C#）来加锁，然后再次检查实例是否已经创建。这样就能确保即使有多个线程同时尝试获取实例，也只会有一个线程能够创建实例。这种方法即实现了线程安全，又避免了在每次获取实例时都进行同步的开销。

```c#
public class Singleton
{
    private static volatile Singleton instance; // volatile 关键字确保 instance 在所有线程中同步
    private static object lockObj = new object(); // lock 对象

    private Singleton() {}

    public static Singleton Instance
    {
        get
        {
            if (instance == null) // 第一次检查
            {
                lock (lockObj) // 锁定
                {
                    if (instance == null) // 第二次检查
                    {
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }
    }
}

```

上述实现方式存在问题：CPU 指令重排序可能导致在 IdGenerator 类的对象被关键字 new 创建并赋值给 instance 之后，还没来得及初始化（执行构造函数中的代码逻辑），就被另一个线程使用了。

(对于.NET/C#环境，你所描述的情况是不会发生的，因为.NET的内存模型保证了在锁的释放和获取之间的操作的正确顺序。这意味着在锁的释放操作发生后，所有在该锁内进行的写操作都将在任何后续获取该锁的线程中可见。这就保证了在构造函数完成之后实例才会被其它线程看到。)

这样，另一个线程就使用了一个没有完整初始化的 IdGenerator 类的对象。要解决这个问题，我们只需要给 instance 成员变量添加 volatile 关键字来禁止指令重排序即可。



```csharp
public class IdGenerator
{
    private long id = 0;
    private static volatile IdGenerator instance = null;
    private static readonly object lockObj = new object();

    private IdGenerator() { }

    public static IdGenerator Instance
    {
        get 
        {
            if (instance == null)
            {
                lock (lockObj)
                {
                    if (instance == null)
                    {
                        instance = new IdGenerator();
                    }
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

```


volatile关键字确保了所有的读操作都将在所有的写操作之后发生，这样就避免了你所描述的问题。然而，使用volatile关键字会引入额外的性能开销，因此你应该只在确实需要的时候才使用它。在这个特定的情况下，使用volatile关键字实际上是不必要的，因为.NET的内存模型已经提供了足够的保护。

4. 静态内部类(Static Inner Class)

在Java中，静态内部类是一种嵌套在外部类中的静态类，它可以访问外部类的静态成员，但不能直接访问外部类的实例成员。

```java
import java.util.concurrent.atomic.AtomicLong;

public class IdGenerator {
  private AtomicLong id = new AtomicLong(0);

  private IdGenerator() {}

  private static class SingletonHolder {
    private static final IdGenerator INSTANCE = new IdGenerator();
  }

  public static IdGenerator getInstance() {
    return SingletonHolder.INSTANCE;
  }

  public long getId() {
    return id.incrementAndGet();
  }
}

```
SingletonHolder 是一个静态内部类，当外部类 IdGenerator 被加载的时候，并不会创建 SingletonHolder 实例对象。只有当调用 getInstance() 方法时，SingletonHolder 才会被加载，这个时候才会创建 instance。instance 的唯一性、创建过程的线程安全性，都由 JVM 来保证。所以，这种实现方法既保证了线程安全，又能做到延迟加载。


在.NET/C#中，可以通过嵌套类（Nested Class）来实现类似的功能，但嵌套类并不区分静态或非静态，它可以访问外部类的所有成员。不过，它通常不会用来实现单例模式，因为单例模式在C#中已经有非常成熟的实现方式。

如果我们仍然想使用嵌套类来实现单例模式，可以像下面这样做：

```c#
using System.Threading;

public class IdGenerator
{
    private long id = 0;

    private IdGenerator() { }

    private class SingletonHolder
    {
        static SingletonHolder()
        {
        }

        internal static readonly IdGenerator instance = new IdGenerator();
    }

    public static IdGenerator Instance
    {
        get { return SingletonHolder.instance; }
    }

    public long GetId()
    {
        return Interlocked.Increment(ref id);
    }
}

```

我们创建了一个名为SingletonHolder的嵌套类，它包含了IdGenerator的单例。在SingletonHolder类第一次被访问时，它的静态成员instance会被初始化，从而实现了单例的延迟加载。

5. Lazy<T>

.NET 4.0 引入了 Lazy<T> 类型来支持延迟初始化，这种方式可以用来实现简单的线程安全的单例模式。

```csharp
public class Singleton
{
    private static readonly Lazy<Singleton> lazy = new Lazy<Singleton>(() => new Singleton());
    
    private Singleton() {}
    
    public static Singleton Instance
    {
        get { return lazy.Value; }
    }
}
```

我们可以使用Lazy<T>来实现懒汉式单例。Lazy<T>类提供了一种延迟初始化对象的方式。在第一次访问Lazy<T>.Value属性时，它将创建并初始化一个新的实例。这个过程是线程安全的。以下是代码示例：

```c#
public class IdGenerator
{
    private long id = 0;
    private static readonly Lazy<IdGenerator> instance = new Lazy<IdGenerator>(() => new IdGenerator());

    private IdGenerator() { }

    public static IdGenerator Instance
    {
        get { return instance.Value; }
    }

    public long GetId()
    {
        return System.Threading.Interlocked.Increment(ref id);
    }
}
```

这段代码的工作原理是这样的：

我们声明了一个类型为 Lazy<IdGenerator> 的 instance 字段，并在声明时就初始化它。Lazy<T>的构造函数接受一个委托，这个委托定义了如何创建和初始化实例。

在Instance属性的get访问器中，我们返回instance.Value。在第一次访问Value属性时，Lazy<T>将调用我们提供的委托来创建和初始化一个新的IdGenerator实例。

GetId方法的实现与之前一样，我们使用System.Threading.Interlocked.Increment方法来线程安全地增加id的值。

通过使用Lazy<T>，我们可以实现真正的懒加载：只有在实际需要IdGenerator实例时，它才会被创建和初始化。
