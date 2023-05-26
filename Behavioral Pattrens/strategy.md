# 策略模式

## 极客时间笔记

>Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

>定义一系列的算法，对每一个进行封装，使得它们可以相互替换。策略模式让算法可以独立于使用它的客户端进行变化。

策略模式解耦的是策略的定义、创建、使用这三部分。


## Design Patterns

Composition类维护一个Component实例的集合，它们代表文档中的文本和图形元素。Composition使用一个Compositor子类的实例，将组件对象排列成行，这个子类封装了一个换行策略。每个组件都有一个关联的自然大小、可伸展性和可收缩性。可伸展性定义了组件可以超出其自然大小的程度；可收缩性则定义了它可以缩小的程度。Composition将这些值传递给Compositor，Compositor利用这些值确定换行的最佳位置。

当需要新的布局时，Composition会要求其Compositor确定放置换行符的位置。Composition会向Compositor传递三个数组，定义组件的自然大小、可伸展性和可收缩性。它还传递组件的数量，行的宽度，以及一个Compositor用于填充每个换行符位置的数组。Compositor返回计算的断行数。


Compositor接口允许Composition将所有信息传递给Compositor。这是"将数据带入策略"的一个例子。需要注意的是，Compositor是一个抽象类。具体的子类定义了特定的换行策略。


要实例化Composition，你需要传递你想要使用的Compositor

Compositor的接口设计得非常仔细，以支持所有子类可能实现的布局算法。你不希望因为每一个新的子类而改变这个接口，因为这将需要改变现有的子类。总的来说，策略和上下文接口决定了模式如何实现其意图。


```csharp
public interface ICompositor
{
    int Compose(int[] natural, int[] stretch, int[] shrink, int componentCount, int lineWidth, int[] breaks);
}

public class Composition
{
    private ICompositor _compositor;
    private int[] _components; // 组件列表
    private int _componentCount; // 组件数量
    private int _lineWidth; // Composition的行宽
    private int[] _lineBreaks; // 组件中的断行位置
    private int _lineCount; // 行数量

    public Composition(ICompositor compositor)
    {
        this._compositor = compositor;
    }

    public void Repair()
    {
        int[] natural = new int[_componentCount];
        int[] stretchability = new int[_componentCount];
        int[] shrinkability = new int[_componentCount];
        int[] breaks = new int[_componentCount];

        // 准备具有所需组件大小的数组
        // ...
        
        // 确定断行位置
        int breakCount = _compositor.Compose(natural, stretchability, shrinkability, _componentCount, _lineWidth, breaks);

        // 根据断行布局组件
        // ...
    }
}

public class SimpleCompositor : ICompositor
{
    public int Compose(int[] natural, int[] stretch, int[] shrink, int componentCount, int lineWidth, int[] breaks)
    {
        // 实现SimpleCompositor的Compose方法
        // ...
        return 0;
    }
}

public class TeXCompositor : ICompositor
{
    public int Compose(int[] natural, int[] stretch, int[] shrink, int componentCount, int lineWidth, int[] breaks)
    {
        // 实现TeXCompositor的Compose方法
        // ...
        return 0;
    }
}

public class ArrayCompositor : ICompositor
{
    private int _interval;

    public ArrayCompositor(int interval)
    {
        _interval = interval;
    }

    public int Compose(int[] natural, int[] stretch, int[] shrink, int componentCount, int lineWidth, int[] breaks)
    {
        // 实现ArrayCompositor的Compose方法
        // ...
        return 0;
    }
}

```

```csharp
Composition quick = new Composition(new SimpleCompositor());
Composition slick = new Composition(new TeXCompositor());
Composition iconic = new Composition(new ArrayCompositor(100));
```

## agile patterns

## .net framework Compare

在.NET中，IComparer<T>和Comparison<T>接口提供了一种定义自定义排序策略的方式。它们通常用于List<T>.Sort方法或Array.Sort方法中，以定义如何比较类型T的两个实例。

首先，我们来看看IComparer<T>接口。这个接口定义了一个Compare方法，这个方法接受两个类型T的参数，并返回一个表示这两个对象相对排序的int值。

```c#
namespace System.Collections.Generic {
    
    using System;
    // The generic IComparer interface implements a method that compares 
    // two objects. It is used in conjunction with the Sort and 
    // BinarySearch methods on the Array, List, and SortedList classes.
    public interface IComparer<in T>
    {
        // Compares two objects. An implementation of this method must return a
        // value less than zero if x is less than y, zero if x is equal to y, or a
        // value greater than zero if x is greater than y.
        // 
        int Compare(T x, T y);
    }
}
```
以下是一个使用IComparer<T>接口来定义自定义排序策略的示例：

```c#
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
}

public class PersonComparer : IComparer<Person>
{
    public int Compare(Person x, Person y)
    {
        // 先按姓氏排序，如果姓氏相同，再按名字排序
        int result = string.Compare(x.LastName, y.LastName);
        if (result == 0)
        {
            result = string.Compare(x.FirstName, y.FirstName);
        }
        return result;
    }
}

// 使用示例
List<Person> people = new List<Person>
{
    new Person { FirstName = "John", LastName = "Doe" },
    new Person { FirstName = "Jane", LastName = "Doe" },
    new Person { FirstName = "John", LastName = "Smith" }
};

people.Sort(new PersonComparer());

foreach (var person in people)
{
    Console.WriteLine($"{person.LastName}, {person.FirstName}");
}

```

在上面的例子中，我们创建了一个名为 PersonComparer 的类，它实现了IComparer<Person> 接口。我们在 Compare 方法中定义了一个比较两个Person 对象的策略：首先按照姓（LastName）进行排序，如果姓相同，则按照名（FirstName）进行排序。

```c#
// Sorts the elements in this list.  Uses the default comparer and 
        // Array.Sort.
        public void Sort()
        {
            Sort(0, Count, null);
        }
 
        // Sorts the elements in this list.  Uses Array.Sort with the
        // provided comparer.
        public void Sort(IComparer<T> comparer)
        {
            Sort(0, Count, comparer);
        }

                // Sorts the elements in a section of this list. The sort compares the
        // elements to each other using the given IComparer interface. If
        // comparer is null, the elements are compared to each other using
        // the IComparable interface, which in that case must be implemented by all
        // elements of the list.
        // 
        // This method uses the Array.Sort method to sort the elements.
        // 
        public void Sort(int index, int count, IComparer<T> comparer) {
            if (index < 0) {
                ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.index, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
            }
            
            if (count < 0) {
                ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.count, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
            }
                
            if (_size - index < count)
                ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_InvalidOffLen);
            Contract.EndContractBlock();
 
            Array.Sort<T>(_items, index, count, comparer);
            _version++;
        }
```

```c#
        [System.Security.SecuritySafeCritical]  // auto-generated
        [ReliabilityContract(Consistency.MayCorruptInstance, Cer.MayFail)]
        public static void Sort<T>(T[] array, int index, int length, System.Collections.Generic.IComparer<T> comparer) {
            if (array==null)
                throw new ArgumentNullException("array");
            if (index < 0 || length < 0)
                throw new ArgumentOutOfRangeException((length<0 ? "length" : "index"), Environment.GetResourceString("ArgumentOutOfRange_NeedNonNegNum"));
            if (array.Length - index < length)
                throw new ArgumentException(Environment.GetResourceString("Argument_InvalidOffLen"));
            Contract.EndContractBlock();
 
            if (length > 1) {
                // <
 
 
 
                if ( comparer == null || comparer == Comparer<T>.Default ) {
                    if(TrySZSort(array, null, index, index + length - 1)) {
                        return;
                    }
                }
                
#if FEATURE_LEGACYNETCF
                if (CompatibilitySwitches.IsAppEarlierThanWindowsPhone8)
                    MangoArraySortHelper<T>.Default.Sort(array, index, length, comparer);                
                else
                    ArraySortHelper<T>.Default.Sort(array, index, length, comparer);                
#else
                ArraySortHelper<T>.Default.Sort(array, index, length, comparer);                
#endif
            }
        }
```

```c#
 public void Sort(T[] keys, int index, int length, IComparer<T> comparer)
        {
            Contract.Assert(keys != null, "Check the arguments in the caller!");
            Contract.Assert( index >= 0 && length >= 0 && (keys.Length - index >= length), "Check the arguments in the caller!");
 
            // Add a try block here to detect IComparers (or their
            // underlying IComparables, etc) that are bogus.
            try
            {
                if (comparer == null)
                {
                    comparer = Comparer<T>.Default;
                }
 
#if FEATURE_CORECLR
                // Since QuickSort and IntrospectiveSort produce different sorting sequence for equal keys the upgrade 
                // to IntrospectiveSort was quirked. However since the phone builds always shipped with the new sort aka 
                // IntrospectiveSort and we would want to continue using this sort moving forward CoreCLR always uses the new sort.
 
                IntrospectiveSort(keys, index, length, comparer);
#else
                if (BinaryCompatibility.TargetsAtLeast_Desktop_V4_5)
                {
                    IntrospectiveSort(keys, index, length, comparer);
                }
                else
                {
                    DepthLimitedQuickSort(keys, index, length + index - 1, comparer, IntrospectiveSortUtilities.QuickSortDepthThreshold);
                }
#endif
            }
            catch (IndexOutOfRangeException)
            {
                IntrospectiveSortUtilities.ThrowOrIgnoreBadComparer(comparer);
            }
            catch (Exception e)
            {
                throw new InvalidOperationException(Environment.GetResourceString("InvalidOperation_IComparerFailed"), e);
            }
        }
```

```c#
        internal static void IntrospectiveSort(T[] keys, int left, int length, IComparer<T> comparer)
        {
            Contract.Requires(keys != null);
            Contract.Requires(comparer != null);
            Contract.Requires(left >= 0);
            Contract.Requires(length >= 0);
            Contract.Requires(length <= keys.Length);
            Contract.Requires(length + left <= keys.Length);
 
            if (length < 2)
                return;
 
            IntroSort(keys, left, length + left - 1, 2 * IntrospectiveSortUtilities.FloorLog2(keys.Length), comparer);
        }
```

```c#
 private static void IntroSort(T[] keys, int lo, int hi, int depthLimit, IComparer<T> comparer)
        {
            Contract.Requires(keys != null);
            Contract.Requires(comparer != null);
            Contract.Requires(lo >= 0);
            Contract.Requires(hi < keys.Length);
 
            while (hi > lo)
            {
                int partitionSize = hi - lo + 1;
                if (partitionSize <= IntrospectiveSortUtilities.IntrosortSizeThreshold)
                {
                    if (partitionSize == 1)
                    {
                        return;
                    }
                    if (partitionSize == 2)
                    {
                        SwapIfGreater(keys, comparer, lo, hi);
                        return;
                    }
                    if (partitionSize == 3)
                    {
                        SwapIfGreater(keys, comparer, lo, hi-1);
                        SwapIfGreater(keys, comparer, lo, hi);
                        SwapIfGreater(keys, comparer, hi-1, hi);
                        return;
                    }
 
                    InsertionSort(keys, lo, hi, comparer);
                    return;
                }
 
                if (depthLimit == 0)
                {
                    Heapsort(keys, lo, hi, comparer);
                    return;
                }
                depthLimit--;
 
                int p = PickPivotAndPartition(keys, lo, hi, comparer);
                // Note we've already partitioned around the pivot and do not have to move the pivot again.
                IntroSort(keys, p + 1, hi, depthLimit, comparer);
                hi = p - 1;
            }
        }
```

```c#
private static void SwapIfGreater(T[] keys, IComparer<T> comparer, int a, int b)
        {
            if (a != b)
            {
                if (comparer.Compare(keys[a], keys[b]) > 0)
                {
                    T key = keys[a];
                    keys[a] = keys[b];
                    keys[b] = key;
                }
            }
        }
 
```

```c#
private static void InsertionSort(T[] keys, int lo, int hi, IComparer<T> comparer)
        {
            Contract.Requires(keys != null);
            Contract.Requires(lo >= 0);
            Contract.Requires(hi >= lo);
            Contract.Requires(hi <= keys.Length);
 
            int i, j;
            T t;
            for (i = lo; i < hi; i++)
            {
                j = i;
                t = keys[i + 1];
                while (j >= lo && comparer.Compare(t, keys[j]) < 0)
                {
                    keys[j + 1] = keys[j];
                    j--;
                }
                keys[j + 1] = t;
            }
        }
```

```c#
private static void Heapsort(T[] keys, int lo, int hi, IComparer<T> comparer)
        {
            Contract.Requires(keys != null);
            Contract.Requires(comparer != null);
            Contract.Requires(lo >= 0);
            Contract.Requires(hi > lo);
            Contract.Requires(hi < keys.Length);
 
            int n = hi - lo + 1;
            for (int i = n / 2; i >= 1; i = i - 1)
            {
                DownHeap(keys, i, n, lo, comparer);
            }
            for (int i = n; i > 1; i = i - 1)
            {
                Swap(keys, lo, lo + i - 1);
                DownHeap(keys, 1, i - 1, lo, comparer);
            }
        }
```

```c#
private static void DownHeap(T[] keys, int i, int n, int lo, IComparer<T> comparer)
        {
            Contract.Requires(keys != null);
            Contract.Requires(comparer != null);
            Contract.Requires(lo >= 0);
            Contract.Requires(lo < keys.Length);
 
            T d = keys[lo + i - 1];
            int child;
            while (i <= n / 2)
            {
                child = 2 * i;
                if (child < n && comparer.Compare(keys[lo + child - 1], keys[lo + child]) < 0)
                {
                    child++;
                }
                if (!(comparer.Compare(d, keys[lo + child - 1]) < 0))
                    break;
                keys[lo + i - 1] = keys[lo + child - 1];
                i = child;
            }
            keys[lo + i - 1] = d;
        }
```

接下来，我们来看看Comparison<T>。Comparison<T>是一个委托，它表示将两个类型T的对象作为输入，并返回一个表示它们相对排序的int的方法。在许多情况下，使用Comparison<T>可以使代码更简洁，因为你可以使用Lambda表达式或方法组来表示比较逻辑，而不是创建一个实现IComparer<T>的单独类。

```c#
public delegate int Comparison<in T>(T x, T y);
```

以下是一个使用Comparison<T>的示例：

```c#
List<Person> people = new List<Person>
{
    new Person { FirstName = "John", LastName = "Doe" },
    new Person { FirstName = "Jane", LastName = "Doe" },
    new Person { FirstName = "John", LastName = "Smith" }
};

Comparison<Person> personComparison = (x, y) =>
{
    int result = string.Compare(x.LastName, y.LastName);
    if (result == 0)
    {
        result = string.Compare(x.FirstName, y.FirstName);
    }
    return result;
};

people.Sort(personComparison);

foreach (var person in people)
{
    Console.WriteLine($"{person.LastName}, {person.FirstName}");
}

```

在这个例子中，我们使用Lambda表达式定义了一个Comparison<Person>。这个Lambda表达式表示的函数与前面例子中PersonComparer类的Compare方法相同。然后，我们将这个Comparison传递给List<Person>.Sort方法来对人员列表进行排序。

```c#
public void Sort(Comparison<T> comparison) {
            if( comparison == null) {
                ThrowHelper.ThrowArgumentNullException(ExceptionArgument.match);
            }
            Contract.EndContractBlock();
 
            if( _size > 0) {
                IComparer<T> comparer = new Array.FunctorComparer<T>(comparison);
                Array.Sort(_items, 0, _size, comparer);
            }
        }
```

```c#
internal sealed class FunctorComparer<T> : IComparer<T> {
            Comparison<T> comparison;
 
            public FunctorComparer(Comparison<T> comparison) {
                this.comparison = comparison;
            }
 
            public int Compare(T x, T y) {
                return comparison(x, y);
            }
        }
```