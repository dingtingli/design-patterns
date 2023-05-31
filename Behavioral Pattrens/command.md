# command 命令模式

## 极客时间 笔记

命令模式的英文翻译是 Command Design Pattern。在 GoF 的《设计模式》一书中，它是这么定义的：

The command pattern encapsulates a request as an object, thereby letting us parameterize other objects with different requests, queue or log requests, and support undoable operations.

翻译成中文就是下面这样。为了帮助你理解，我对这个翻译稍微做了补充和解释，也一起放在了下面的括号中。

命令模式将请求（命令）封装为一个对象，这样可以使用不同的请求参数化其他对象（将不同请求依赖注入到其他对象），并且能够支持请求（命令）的排队执行、记录日志、撤销等（附加控制）功能。

对于 GoF 给出的定义，我这里再进一步解读一下。

落实到编码实现，命令模式用的最核心的实现手段，是将函数封装成对象。我们知道，C 语言支持函数指针，我们可以把函数当作变量传递来传递去。但是，**在大部分编程语言中，函数没法儿作为参数传递给其他函数，也没法儿赋值给变量。借助命令模式，我们可以将函数封装成对象。**具体来说就是，设计一个包含这个函数的类，实例化一个对象传来传去，这样就可以实现把函数像对象一样使用。从实现的角度来说，它类似我们之前讲过的回调。

当我们把函数封装成对象之后，对象就可以存储下来，方便控制执行。所以，命令模式的主要作用和应用场景，是用来控制命令的执行，比如，异步、延迟、排队执行命令、撤销重做命令、存储命令、给命令记录日志等等，这才是命令模式能发挥独一无二作用的地方。

命令模式的优点在于，它可以将一系列的操作封装在一个对象中，这个对象可以包含额外的上下文信息，可以具有更复杂的行为。


---

线程内轮询通常是指在单一线程中循环接收并处理请求的方式。这种方式的主要思想是，一个线程依次处理每个请求，完成一个请求后再处理下一个请求，它不会在多个请求之间进行上下文切换。这种设计在某些情况下可能会更高效，因为它避免了多线程并发控制的开销。它也更简单，因为你不需要担心并发条件和锁等多线程编程问题。

然而，线程内轮询的缺点是它无法充分利用多核处理器的优势。如果你有一个多核处理器，线程内轮询通常无法使用所有的核，因为它在任何时刻只在一个核上运行。因此，如果你有大量的CPU密集型任务，使用线程池处理请求可能会更高效。

线程池处理请求是一种不同的模式。在这种模式下，有一个主线程接收请求，然后它将请求分发给线程池中的一个线程去处理。这样的好处是可以同时处理多个请求，因此可以更好地利用多核处理器的优势。同时，线程池也可以帮助我们控制系统中的线程数量，避免线程数量过多导致的系统资源耗尽。

但是，线程池处理请求的方式也有它的缺点。例如，由于多个线程可能同时处理多个请求，所以需要处理并发控制的问题，如锁和同步等，这些问题可能会导致更复杂的编程和更高的开销。另外，线程之间的上下文切换也会带来一定的性能损失。

总的来说，线程内轮询和线程池处理请求各有优缺点，需要根据具体的应用场景和需求来选择适合的方式。
---

整个手游后端服务器轮询获取客户端发来的请求，获取到请求之后，借助命令模式，把请求包含的数据和处理逻辑封装为命令对象，并存储在内存队列中。然后，再从队列中取出一定数量的命令来执行。执行完成之后，再重新开始新的一轮轮询。具体的示例代码如下所示，你可以结合着一块看下。


以下是你的Java代码的C#版本：

首先是`Command`接口和其中的一个实现类 `GotDiamondCommand`：

```csharp
public interface ICommand
{
    void Execute();
}

public class GotDiamondCommand : ICommand
{
    // 省略成员变量

    public GotDiamondCommand(/*数据*/)
    {
        //...
    }

    public void Execute()
    {
        // 执行相应的逻辑
    }
}
```

类似的，你还可以实现`GotStartCommand`, `HitObstacleCommand`, `ArchiveCommand`等。

然后是主要的`GameApplication`类：

```csharp
public class GameApplication
{
    private const int MAX_HANDLED_REQ_COUNT_PER_LOOP = 100;
    private Queue<ICommand> queue = new Queue<ICommand>();

    public void Mainloop()
    {
        while (true)
        {
            List<Request> requests = new List<Request>();

            //省略从epoll或者select中获取数据，并封装成Request的逻辑，
            //注意设置超时时间，如果很长时间没有接收到请求，就继续下面的逻辑处理。

            foreach (var request in requests)
            {
                Event @event = request.Event;
                ICommand command = null;
                if (@event == Event.GOT_DIAMOND)
                {
                    command = new GotDiamondCommand(/*数据*/);
                }
                else if (@event == Event.GOT_STAR)
                {
                    command = new GotStartCommand(/*数据*/);
                }
                else if (@event == Event.HIT_OBSTACLE)
                {
                    command = new HitObstacleCommand(/*数据*/);
                }
                else if (@event == Event.ARCHIVE)
                {
                    command = new ArchiveCommand(/*数据*/);
                } // ...一堆else if...

                queue.Enqueue(command);
            }

            int handledCount = 0;
            while (handledCount < MAX_HANDLED_REQ_COUNT_PER_LOOP)
            {
                if (queue.Count == 0)
                {
                    break;
                }
                ICommand command = queue.Dequeue();
                command.Execute();
                handledCount++;
            }
        }
    }
}
```

在C#代码中，队列的操作方式稍微不同。你需要使用`Enqueue`方法添加元素，使用`Dequeue`方法移除并返回队头元素，`Count`获取队列元素的数量。另外，如果需要比较对象是否相等，一般使用`==`操作符，而不是`equals`方法。

请注意，这段代码中的`Event`、`Request`等类型和具体的命令类(`GotStartCommand`，`HitObstacleCommand`，`ArchiveCommand`等)我在这里并没有提供，你需要根据实际的需求进行实现。

你可能会觉得，命令模式跟策略模式、工厂模式非常相似啊，那它们的区别在哪里呢？不仅如此，在留言区中我还看到有不止一个同学反映，感觉学过的很多模式都很相似。不知道你有没有类似的感觉呢？


每个设计模式都应该由两部分组成：第一部分是应用场景，即这个模式可以解决哪类问题；第二部分是解决方案，即这个模式的设计思路和具体的代码实现。不过，代码实现并不是模式必须包含的。如果你单纯地只关注解决方案这一部分，甚至只关注代码实现，就会产生大部分模式看起来都很相似的错觉。实际上，设计模式之间的主要区别还是在于设计意图，也就是应用场景。单纯地看设计思路或者代码实现，有些模式确实很相似，比如策略模式和工厂模式。


之前讲策略模式的时候，我们有讲到，策略模式包含策略的定义、创建和使用三部分，从代码结构上来，它非常像工厂模式。它们的区别在于，策略模式侧重“策略”或“算法”这个特定的应用场景，用来解决根据运行时状态从一组策略中选择不同策略的问题，而工厂模式侧重封装对象的创建过程，这里的对象没有任何业务场景的限定，可以是策略，但也可以是其他东西。从设计意图上来，这两个模式完全是两回事儿。

## agile 笔记

