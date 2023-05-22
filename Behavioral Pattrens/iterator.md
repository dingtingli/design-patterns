迭代器模式是一种行为型设计模式，它提供了一种方法来访问一个聚合对象中的各个元素，而无需暴露对象的内部表示。


行为型设计模式主要处理对象间的通信，为对象间的通信设定规则。这种模式涉及到类和对象之间的通信，它们可以帮助你实现类或对象之间的交互，还可以帮你划分责任。

迭代器模式通过提供一种统一的访问接口，处理对象之间的通信。

1. 提供统一的接口：迭代器模式为遍历各种类型的集合结构（如数组、列表、树等）提供了一个通用的接口。这意味着，使用迭代器模式，可以统一对象间的交互方式，无论内部的数据结构如何。

2. 隐藏内部实现：迭代器模式允许访问一个集合对象的内容而无需暴露其内部结构。这种通信方式不仅使得代码更加清晰，也增强了代码的可靠性。

3. 支持不同的遍历策略：迭代器模式可以轻易地支持不同的遍历策略，因为你可以在不修改原有代码的情况下实现新的迭代器。这为对象间的通信提供了更大的灵活性。

4. 分离职责：迭代器模式将数据的遍历和数据的存储分离开来，让使用者无需知道集合的内部结构就能遍历元素，同时也使得集合的实现可以独立于迭代的实现。这种方式明确了对象间的职责边界，使得对象间的通信更加清晰。


将访问和遍历的责任从列表对象中剥离出来，放入一个迭代器（Iterator）对象中。迭代器类定义了一个访问列表元素的接口。一个迭代器对象负责跟踪当前元素；也就是说，它知道哪些元素已经被遍历过了。

例如，List类会调用一个ListIterator，他们之间的关系如下:

在你可以实例化ListIterator之前，你必须提供要遍历的List。一旦你有了ListIterator实例，你就可以顺序地访问列表的元素。CurrentItem操作返回列表中的当前元素，First初始化当前元素为第一个元素，Next将当前元素推进到下一个元素，IsDone测试我们是否已经超过了最后一个元素——也就是说，我们是否已经完成了遍历。

将遍历机制从List对象中分离出来，让我们可以定义不同遍历策略的迭代器，而不用在List接口中枚举它们。例如，FilteringListIterator可能只提供对满足特定过滤约束的元素的访问。

注意迭代器和列表是耦合的，客户端必须知道它是一个列表正在被遍历，而不是某种其他的聚合结构。因此客户端承诺一个特定的聚合结构。如果我们能在不改变客户端代码的情况下改变聚合类，那就更好了。我们可以通过将迭代器概念普遍化来支持**多态迭代(polymorphic iteration)**，来实现这一点。

假设我们也有一个SkipList的列表实现。跳跃列表是一种具有与平衡树类似特性的概率数据结构。我们希望能够编写既适用于List也适用于SkipList对象的代码。

我们定义了一个AbstractList类，为操作列表提供了一个通用的接口。类似地，我们需要一个抽象的迭代器类，定义一个通用的迭代器接口。因此我们定义了AbstractIterator，它定义了First、Next、IsDone和CurrentItem的抽象操作，并从中派生出ListIterator和SkipListIterator。

对于List和SkipList的实现，我们创建了List和SkipList类，它们都继承自AbstractList类。这意味着它们都实现了CreateIterator操作，这个操作返回一个与之相关的迭代器。List类返回一个ListIterator的实例，SkipList返回一个SkipListIterator的实例。这些具体的迭代器都实现了AbstractIterator类的所有操作，以便正确地遍历其相关联的聚合对象。

由于我们的客户端代码现在使用的是抽象接口，我们可以在不更改客户端代码的情况下更改列表的实现。我们可以在使用列表时交换实现，或者在不同的环境中使用不同的实现。例如，我们可以在小型列表中使用List，而在大型列表中使用SkipList。因为我们使用的是抽象的迭代器，我们甚至可以在遍历的过程中交换实现。

迭代器模式在面向对象的程序设计中十分常见，许多语言的标准库都包含这个模式。例如，Java的Collection Framework和Python的迭代器协议都是迭代器模式的实现。

结构
迭代器模式涉及到以下几种角色：

抽象聚合（Abstract Aggregate）：定义创建迭代器对象的接口。

具体聚合（Concrete Aggregate）：实现抽象聚合提供的创建迭代器对象的接口，返回一个合适的具体迭代器实例。

抽象迭代器（Abstract Iterator）：定义访问和遍历元素的接口。

具体迭代器（Concrete Iterator）：实现抽象迭代器定义的接口，完成对聚合对象的遍历，跟踪遍历过程中的当前位置。

以下是迭代器模式的通用类图表示：

```plaintext
AbstractAggregate                   AbstractIterator
    | CreateIterator()                  | First()
    |                                   | Next()
    |                                   | IsDone()
ConcreteAggregate                  ConcreteIterator
    | CreateIterator()                  | First()
                                        | Next()
                                        | IsDone()
                                        | CurrentItem()
```


其中，ConcreteAggregate返回的是一个ConcreteIterator实例，而客户代码仅通过AbstractIterator接口与迭代器进行交互。

实现
迭代器有许多实现变体和替代方案。以下是一些重要的实现方式。权衡通常取决于你的语言提供的控制结构。一些语言（例如CLU [LG86]）甚至直接支持这种模式。

1. **谁控制迭代？** 一个基本问题是决定哪一方控制迭代，是迭代器还是使用迭代器的客户端。当客户端控制迭代时，迭代器被称为外部迭代器，当迭代器控制它时，迭代器是内部迭代器。使用外部迭代器的客户端必须明确地从迭代器中前进遍历并请求下一个元素。相反，客户端将一个操作交给内部迭代器执行，迭代器将该操作应用到聚合中的每个元素。
外部迭代器比内部迭代器更灵活。例如，使用外部迭代器比较两个集合是否相等很容易，但使用内部迭代器几乎是不可能的。在像C++这样的语言中，内部迭代器特别弱，因为它不提供匿名函数、闭包或像Smalltalk和CLOS那样的续延。但另一方面，内部迭代器更易于使用，因为它们为你定义了迭代逻辑。

2. **谁定义了遍历算法？** 迭代器并不是定义遍历算法的唯一地方。聚合可能定义遍历算法，并使用迭代器仅存储迭代的状态。我们称这种迭代器为光标（cursor），因为它只是指向聚合中的当前位置。客户端将光标作为参数调用聚合的Next操作，Next操作将改变光标的状态。
如果迭代器负责遍历算法，那么在同一个聚合上使用不同的迭代算法就很容易，同时在不同的聚合上复用同一算法也可能更简单。另一方面，遍历算法可能需要访问聚合的私有变量。如果是这样，将遍历算法放在迭代器中就违反了聚合的封装。

3. **迭代器有多稳健？** 在遍历一个聚合时修改它可能是危险的。如果从聚合中添加或删除元素，你可能会访问一个元素两次或完全错过它。一个简单的解决方案是复制聚合并遍历复制品，但这在一般情况下太昂贵了。
强健的迭代器确保插入和删除不会干扰遍历，并且无需复制聚合就可以做到这一点。实现强健迭代器有很多方法。大多数方法都依赖于将迭代器注册到聚合中。在插入或删除时，聚合要么调整它生成的迭代器的内部状态，要么在内部维护信息以确保正确的遍历。
Kofler提供了关于如何在ET++中实现强健迭代器的良好讨论[Kof93]。Murray讨论了为USLStandardComponents' List类实现强健迭代器的方法[Mur93]。

4. **追加的迭代器操作。** 迭代器的最小接口由First，Next，IsDone 和 CurrentItem 操作组成。一些额外的操作可能会很有用。例如，有序的聚合可以有一个Previous操作，该操作将迭代器定位到前一个元素。对于排序或索引的集合，SkipTo操作很有用。SkipTo将迭代器定位到满足特定条件的对象。

5. **迭代器可能具有特权访问。** 迭代器可以被视为其创建的聚合的扩展。迭代器和聚合紧密耦合。我们可以通过在C++中使迭代器成为其聚合的朋友来表达这种密切关系。然后你不需要定义聚合操作，其唯一的目的就是让迭代器有效地实现遍历。
但是，这样的特权访问可能使定义新的遍历变得困难，因为它需要改变聚合接口以添加另一个朋友。为避免这个问题，迭代器类可以包括用于访问聚合的重要但公开不可用成员的受保护操作。迭代器子类（仅迭代器子类）可以使用这些受保护的操作来获得对聚合的特权访问。

8. **空迭代器。** 空迭代器（NullIterator）是一种退化的迭代器，对于处理边界条件很有帮助。按定义，空迭代器总是完成遍历；也就是说，它的IsDone操作总是返回真。

    空迭代器可以使遍历树形结构的聚合（如Composite）更加容易。在遍历的每个点，我们都会询问当前元素的子元素的迭代器。聚合元素如常返回具体的迭代器。但是，叶子元素返回一个空迭代器的实例。这让我们可以以统一的方式实现整个结构的遍历。

## sample code

我们会展示两个迭代器的实现，一个用于从前向后遍历 List，另一个用于从后向前遍历（基础库只支持前者）。然后我们展示如何使用这些迭代器，以及如何避免依赖特定的实现。接着，我们修改设计以确保迭代器被正确删除。最后一个例子展示了一个内部迭代器，并将其与外部迭代器进行比较。

1. List和迭代器接口。首先让我们看一下与实现迭代器相关的List接口部分。

    ```csharp
    public class List<T>
    {
        private const int DEFAULT_LIST_CAPACITY = 10;
        private T[] items;

        public List(int size = DEFAULT_LIST_CAPACITY)
        {
            items = new T[size];
        }

        public long Count
        {
            get { return items.Length; }
        }

        public T Get(long index)
        {
            return items[index];
        }
    }
    ```

    我们不需要赋予迭代器对底层数据结构的特权访问，也就是说，迭代器类并不是 List 的朋友类。为了使不同的遍历透明使用，我们定义了一个抽象的 Iterator 类，它定义了迭代器接口。

    ```csharp
    public abstract class Iterator<T>
    {
        public abstract void First();
        public abstract void Next();
        public abstract bool IsDone();
        public abstract T CurrentItem();
        protected Iterator() { /* ... */ }
    }
    ```

2. 迭代器子类的实现。ListIterator 是 Iterator 的子类。

```csharp
public class ListIterator<T> : Iterator<T>
{
    private List<T> _list;
    private long _current;

    public ListIterator(List<T> aList)
    {
        _list = aList;
        _current = 0;
    }

    public override void First()
    {
        _current = 0;
    }

    public override void Next()
    {
        _current++;
    }

    public override bool IsDone()
    {
        return _current >= _list.Count();
    }

    public override T CurrentItem()
    {
        if (IsDone())
        {
            throw new InvalidOperationException("Iterator out of bounds");
        }

        return _list.Get(_current);
    }
}
```

ListIterator 的实现非常直接。它将 List 以及一个索引 _current 存储到列表中；

ReverseListIterator 的实现基本相同，除了它的 First 操作会将 _current 定位到列表的末尾，并且 Next 会使 _current 朝向第一个项递减。

```csharp
public class ReverseListIterator<T> : Iterator<T>
{
    private List<T> _list;
    private long _current;

    public ListIterator(List<T> aList)
    {
        _list = aList;
        _current = 0;
    }

    public override void First()
    {
        _current = _list.Count()-1;
    }

    public override void Next()
    {
        _current--;
    }

    public override bool IsDone()
    {
        return _current < 0;
    }

    public override T CurrentItem()
    {
        if (IsDone())
        {
            throw new InvalidOperationException("Iterator out of bounds");
        }

        return _list.Get(_current);
    }
}
```

3. 使用迭代器。假设我们有一个 Employee 对象的 List，我们希望打印所有包含的员工。Employee 类支持这个功能，通过一个 Print 操作。要打印列表，我们定义一个 PrintEmployees 操作，它接受一个 Employee ListIterator 作为参数：

```csharp
public void PrintEmployees<T>(Iterator<T> iterator) where T : Employee
{
    for (iterator.First(); !iterator.IsDone; iterator.Next())
    {
        iterator.CurrentItem.Print();
    }
}

var employees = new List<Employee>(20);
// Populate the employees list...
var forward = new ListIterator<Employee>(employees);
var backward = new ReverseListIterator<Employee>(employees);
PrintEmployees(forward);
PrintEmployees(backward);
```

4. 避免对特定列表实现的承诺。让我们考虑一下如何将跳跃列表（SkipList）版本的List应用到我们的迭代代码中。一个SkipList子类需要提供一个实现迭代器接口的SkipListIterator。在内部，SkipListIterator需要保留超过一个索引来有效地进行迭代。但由于SkipListIterator符合迭代器接口，当员工数据存储在SkipList对象中时，也可以使用PrintEmployees操作。

```c#
SkipList<Employee> employees = new SkipList<Employee>();
// ...

SkipListIterator<Employee> iterator = new SkipListIterator<Employee>(employees);
PrintEmployees(iterator);
```

虽然这种方法可行，但如果我们不必承诺使用特定的List实现，例如SkipList，那就更好了。我们可以引入一个AbstractList类来标准化不同列表实现的接口。List和SkipList成为AbstractList的子类。为了实现多态迭代，AbstractList定义了一个工厂方法CreateIterator，子类重写这个方法以返回对应的迭代器：

```csharp
public abstract class AbstractList<T>
{
    // ...

    public abstract Iterator<T> CreateIterator();

    // ...
}

```

List重写CreateIterator以返回一个ListIterator对象：

```c#
public class List<T> : AbstractList<T>
{
    // ...

    public override Iterator<T> CreateIterator()
    {
        return new ListIterator<T>(this);
    }

    // ...
}

```

现在我们可以写出打印员工信息的代码，而无需考虑具体的表示方法：

```c#
AbstractList<Employee> employees;
// ...

Iterator<Employee> iterator = employees.CreateIterator();
PrintEmployees(iterator);

```

6. 内部ListIterator。作为最后一个例子，让我们看一下内部或被动ListIterator类的可能实现。在这里，迭代器控制迭代，并对每个元素应用一个操作。

下面是一个使用C#的lambda表达式（匿名函数）和内部迭代器的例子：


在这个例子中，我们有一个ListTraverser类，它接收一个List<T>和一个Func<T, bool>作为参数。这个Func<T, bool>就是我们要对每个元素执行的操作，它接收一个元素作为参数，返回一个bool值来确定是否继续遍历。

```c#
public class ListTraverser<T>
{
    private readonly List<T> _list;
    private readonly Func<T, bool> _action;
    private readonly ListIterator<T> _iterator;

    public ListTraverser(List<T> list, Func<T, bool> action)
    {
        _list = list;
        _action = action;
        _iterator = new ListIterator<T>(_list);
    }

    public bool Traverse()
    {
        for (_iterator.First(); !_iterator.IsDone(); _iterator.Next())
        {
            if (!_action(_iterator.CurrentItem()))
            {
                return false;
            }
        }

        return true;
    }
}
```

我们可以用这个类来打印出 Employee 列表中的前10个元素：
```c#
List<Employee> employees = /* 假设这个列表已被填充 */;
int count = 0;

ListTraverser<Employee> traverser = new ListTraverser<Employee>(
    employees, 
    employee => 
    {
        Console.WriteLine(employee);
        count++;
        return count < 10;
    });

traverser.Traverse();
```

## 极客时间笔记

在平时开发中，特别是业务开发，我们直接使用即可，很少会自己去实现一个迭代器。不过，知其然知其所以然，弄懂原理能帮助我们更好的使用这些工具类，所以，我觉得还是有必要学习一下这个模式。

迭代器模式将集合对象的遍历操作从集合类中拆分出来，放到迭代器类中，让两者的职责更加单一。

这里为了讲解迭代器的实现原理，我们假设某个新的编程语言的基础类库中，还没有提供线性容器对应的迭代器，需要我们从零开始开发。现在，我们一块来看具体该如何去做。

假设在这种新的编程语言中，数组和链表 这两个数据结构分别对应 ArrayList 和 LinkedList 两个类。除此之外，我们从两个类中抽象出公共的接口，定义为 List 接口，以方便开发者基于接口而非实现编程，编写的代码能在两种数据存储结构之间灵活切换。

我们针对 ArrayList 和 LinkedList 两个线性容器，设计实现对应的迭代器。我们定义一个迭代器接口 Iterator，以及针对两种容器的具体的迭代器实现类 ArrayIterator 和 ListIterator。

```c#
public interface IIterator<T>
{
    void First();
    void Next();
    bool IsDone();
    T CurrentItem();
}
```

```c#
// ArrayList 迭代器
public class ArrayIterator<T> : IIterator<T>
{
    private readonly ArrayList<T> _arrayList;
    private int _current;

    public ArrayIterator(ArrayList<T> arrayList)
    {
        _arrayList = arrayList;
        _current = 0;
    }

    public void First()
    {
        _current = 0;
    }

    public void Next()
    {
        _current++;
    }

    public bool IsDone()
    {
        return _current >= _arrayList.Count;
    }

    public T CurrentItem()
    {
        if (IsDone())
        {
            throw new InvalidOperationException("Iterator out of bounds");
        }

        return _arrayList.Get(_current);
    }
}

```

```c#

// LinkedList 迭代器
public class LinkedListIterator<T> : IIterator<T>
{
    private readonly LinkedList<T> _linkedList;
    private int _current;

    public LinkedListIterator(LinkedList<T> linkedList)
    {
        _linkedList = linkedList;
        _current = 0;
    }

    public void First()
    {
        _current = 0;
    }

    public void Next()
    {
        _current++;
    }

    public bool IsDone()
    {
        return _current >= _linkedList.Count;
    }

    public T CurrentItem()
    {
        if (IsDone())
        {
            throw new InvalidOperationException("Iterator out of bounds");
        }

        return _linkedList.Get(_current);
    }
}
```

```c#
// 定义 List 接口
public interface IList<T>
{
    void Add(T item);
    T Get(int index);
    void Remove(T item);
    bool Contains(T item);
    int Count { get; }
    IIterator<T> GetIterator();
}
```

```c#
// ArrayList 实现 IList 接口
public class ArrayList<T> : IList<T>
{
    private List<T> _internalList = new List<T>();

    public void Add(T item)
    {
        _internalList.Add(item);
    }

    public T Get(int index)
    {
        return _internalList[index];
    }

    public void Remove(T item)
    {
        _internalList.Remove(item);
    }

    public bool Contains(T item)
    {
        return _internalList.Contains(item);
    }

    public int Count => _internalList.Count;

    public IIterator<T> GetIterator()
    {
        return new ArrayIterator<T>(this);
    }
}
```

```c#
public class LinkedList<T> : IList<T>
{
    private System.Collections.Generic.LinkedList<T> _internalList = new System.Collections.Generic.LinkedList<T>();

    public void Add(T item)
    {
        _internalList.AddLast(item);
    }

    public T Get(int index)
    {
        return _internalList.Skip(index).First();
    }

    public void Remove(T item)
    {
        _internalList.Remove(item);
    }

    public bool Contains(T item)
    {
        return _internalList.Contains(item);
    }

    public int Count => _internalList.Count;

    public IIterator<T> GetIterator()
    {
        return new LinkedListIterator<T>(this);
    }
}
```

```c#
static void Main(string[] args)
{
    // 创建 ArrayList 并添加元素
    IList<string> arrayList = new ArrayList<string>();
    arrayList.Add("Hello");
    arrayList.Add("World");
    arrayList.Add("from");
    arrayList.Add("ArrayList");

    // 获取 ArrayList 的迭代器并遍历
    IIterator<string> arrayListIterator = arrayList.GetIterator();
    while (!arrayListIterator.IsDone())
    {
        Console.WriteLine(arrayListIterator.CurrentItem());
        arrayListIterator.Next();
    }

    // 创建 LinkedList 并添加元素
    IList<string> linkedList = new LinkedList<string>();
    linkedList.Add("Hello");
    linkedList.Add("World");
    linkedList.Add("from");
    linkedList.Add("LinkedList");

    // 获取 LinkedList 的迭代器并遍历
    IIterator<string> linkedListIterator = linkedList.GetIterator();
    while (!linkedListIterator.IsDone())
    {
        Console.WriteLine(linkedListIterator.CurrentItem());
        linkedListIterator.Next();
    }

    Console.ReadKey();
}
```

使用迭代器遍历集合的优势是什么？

我们可以使用foreach循环遍历C#中的集合，但是要使用这种方法，我们需要实现IEnumerable和IEnumerator接口。在当前实现中，我们定义的IList接口并没有继承这两个接口。在.NET中，一般情况下我们将使用实现了IEnumerable和IEnumerator接口的集合类，这样我们可以直接使用foreach语句来遍历集合。

以下是你可以将IList和IIterator更改为与.NET集合接口一致的方式：

csharp
```c#
public interface IList<T> : IEnumerable<T>
{
    void Add(T item);
    IIterator<T> GetIterator();
}

public interface IIterator<T> : IEnumerator<T>
{
    bool HasNext();
}
```

然后在ArrayList和LinkedList类中实现这些接口的所有方法。一旦你完成了这些，你就可以使用foreach循环来遍历这些集合：

```c#
foreach (var item in arrayList)
{
    Console.WriteLine(item);
}

foreach (var item in linkedList)
{
    Console.WriteLine(item);
}
```

这就是为什么C#中的foreach循环非常方便的原因，它将迭代器模式融入了语言本身。

foreach 循环只是一个语法糖而已，底层是基于迭代器来实现的。

for 循环遍历方式比起迭代器遍历方式，代码看起来更加简洁。那我们为什么还要用迭代器来遍历容器呢？

1. 对于类似数组和链表这样的数据结构，遍历方式比较简单，直接使用 for 循环来遍历就足够了。但是，对于复杂的数据结构（比如树、图）来说，有各种复杂的遍历方式。比如，树有前中后序、按层遍历，图有深度优先、广度优先遍历等等。如果由客户端代码来实现这些遍历算法，势必增加开发成本，而且容易写错。如果将这部分遍历的逻辑写到容器类中，也会导致容器类代码的复杂性。

2. 将游标指向的当前位置等信息，存储在迭代器类中，每个迭代器独享游标信息。这样，我们就可以创建多个不同的迭代器，同时对同一个容器进行遍历而互不影响。

3. 容器和迭代器都提供了抽象的接口，方便我们在开发的时候，基于接口而非具体的实现编程。当需要切换新的遍历算法的时候，比如，从前往后遍历链表切换成从后往前遍历链表，客户端代码只需要将迭代器类从 LinkedIterator 切换为 ReversedLinkedIterator 即可，其他代码都不需要修改。

## 极客时间 迭代器模型中

迭代器模式主要作用是解耦容器代码和遍历代码，这也印证了我们前面多次讲过的应用设计模式的主要目的是解耦。

如果在使用迭代器遍历集合的同时增加、删除集合中的元素，会发生什么情况？应该如何应对？

当通过迭代器来遍历集合的时候，增加、删除集合元素会导致不可预期的遍历结果。实际上，“不可预期”比直接出错更加可怕，有的时候运行正确，有的时候运行错误，一些隐藏很深、很难 debug 的 bug 就是这么产生的。那我们如何才能避免出现这种不可预期的运行结果呢？

增删元素之后让遍历报错。解决方法更加合理。Java 语言就是采用的这种解决方案，增删元素之后，让遍历报错。接下来，我们具体来看一下如何实现。

怎么确定在遍历时候，集合有没有增删元素呢？我们在 ArrayList 中定义一个成员变量 modCount，记录集合被修改的次数，集合每调用一次增加或删除元素的函数，就会给 modCount 加 1。当通过调用集合上的 iterator() 函数来创建迭代器的时候，我们把 modCount 值传递给迭代器的 expectedModCount 成员变量，之后每次调用迭代器上的 hasNext()、next()、currentItem() 函数，我们都会检查集合上的 modCount 是否等于 expectedModCount，也就是看，在创建完迭代器之后，modCount 是否改变过。


## 极客时间 迭代器模型下

如何实现一个支持“快照”功能的迭代器？

```c#
public class SnapshotList<T> : IEnumerable<T>
{
    private List<T> _list = new List<T>();

    public void Add(T item)
    {
        _list.Add(item);
    }

    public bool Remove(T item)
    {
        return _list.Remove(item);
    }

    public IEnumerator<T> GetEnumerator()
    {
        return new SnapshotEnumerator<T>(_list);
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}

public class SnapshotEnumerator<T> : IEnumerator<T>
{
    private List<T> _snapshot;
    private int _index = -1;

    public SnapshotEnumerator(List<T> original)
    {
        _snapshot = new List<T>(original);
    }

    public T Current
    {
        get
        {
            if (_index < 0 || _index >= _snapshot.Count)
                throw new InvalidOperationException();
            return _snapshot[_index];
        }
    }

    object IEnumerator.Current => Current;

    public void Dispose() { }

    public bool MoveNext()
    {
        _index++;
        return _index < _snapshot.Count;
    }

    public void Reset()
    {
        _index = -1;
    }
}

```

```c#
var list = new SnapshotList<int>() { 1, 2, 3 };

var iterator1 = list.GetEnumerator();

list.Remove(2);

var iterator2 = list.GetEnumerator();

while (iterator1.MoveNext())
{
    Console.WriteLine(iterator1.Current);
}

while (iterator2.MoveNext())
{
    Console.WriteLine(iterator2.Current);
}

```