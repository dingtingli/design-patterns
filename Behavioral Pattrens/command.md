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

事实上，这种模式已经跨越了一个非常有趣的界限。正是在这个界限的交叉处，所有有趣的复杂性才存在。大多数类将一套方法与相应的一组变量相关联。COMMAND模式不是这样做的。相反，它封装了一个不带任何变量的单个函数。

在严格的面向对象术语中，这是可憎的，这是功能分解的痕迹。它将一个函数的作用提升到类的级别。亵渎！然而，在两种范例相互冲突的边界处，有趣的事情开始发生。

在这个描述中，我们有硬件设备（如马达，继电器），设备有可能的操作（如打开，关闭），还有一些传感器负责在某些事件发生时执行相应的命令。在这种情况下，命令模式可以非常有效地管理这种复杂性。

首先，我们可以定义一个基础的 `Command` 接口，该接口声明了一个 `Execute` 方法：

```csharp
public interface ICommand
{
    void Execute();
}
```

然后，我们可以定义一些特定的命令，这些命令将封装对硬件设备的操作。例如，我们可能有 `RelayOnCommand` 和 `MotorOffCommand` 类，这些类都实现了 `ICommand` 接口：

```csharp
public class RelayOnCommand : ICommand
{
    private Relay _relay;

    public RelayOnCommand(Relay relay)
    {
        _relay = relay;
    }

    public void Execute()
    {
        _relay.On();
    }
}

public class MotorOffCommand : ICommand
{
    private Motor _motor;

    public MotorOffCommand(Motor motor)
    {
        _motor = motor;
    }

    public void Execute()
    {
        _motor.Off();
    }
}
```

这里的 `Relay` 和 `Motor` 类可以代表真实的硬件设备，它们可以有各自的 `On` 和 `Off` 方法等。

最后，我们有一些传感器，它们可以绑定特定的命令并在检测到特定事件时执行这些命令：

```csharp
public class Sensor
{
    private ICommand _command;

    public Sensor(ICommand command)
    {
        _command = command;
    }

    public void EventOccurred()
    {
        // 在检测到某个事件时执行命令
        _command.Execute();
    }
}
```

在这种设计中，`Sensor` 类不需要知道它正在执行什么命令，也不需要知道它正在控制什么设备。它只需要知道它有一个命令，并且在某个事件发生时需要执行这个命令。这样，我们就能够有效地解耦设备、操作和触发这些操作的事件。

这只是一个简单的实现，实际的情况可能会更复杂。例如，你可能需要定义更多的命令和设备，或者你可能需要让传感器能够在不同的事件发生时执行不同的命令。但是，基本的思想和模式仍然是相同的。

谢谢你的反馈。下面我会通过代码来展示如何在初始化函数中设置传感器和对应的命令。

首先，我们假设有一个 `SystemInitializer` 类，该类负责系统的初始化工作。在这个类中，我们可以定义一个 `Initialize` 方法，该方法创建一些传感器和命令，并将它们绑定在一起：

```csharp
public class SystemInitializer
{
    public List<Sensor> Sensors { get; private set; }

    public void Initialize()
    {
        // 创建一些设备
        Relay relay = new Relay();
        Motor motor = new Motor();

        // 创建一些命令，并将它们与设备关联起来
        ICommand relayOnCommand = new RelayOnCommand(relay);
        ICommand motorOffCommand = new MotorOffCommand(motor);

        // 创建一些传感器，并将它们与命令关联起来
        Sensor relaySensor = new Sensor(relayOnCommand);
        Sensor motorSensor = new Sensor(motorOffCommand);

        // 将所有的传感器存储起来，以便于后续的使用
        Sensors = new List<Sensor> { relaySensor, motorSensor };
    }
}
```

然后，你可以在程序的主体部分（例如在 `Main` 方法或者某个服务类中）使用这个 `SystemInitializer` 来初始化系统：

```csharp
class Program
{
    static void Main(string[] args)
    {
        SystemInitializer initializer = new SystemInitializer();
        initializer.Initialize();

        // 此时，我们已经有了一些已经初始化并绑定了命令的传感器，
        // 我们可以在需要的时候使用它们。
        List<Sensor> sensors = initializer.Sensors;
    }
}
```

这样，我们就将传感器和命令之间的逻辑关系全部移到了 `SystemInitializer` 类中，使主程序体得以简化，而且更符合单一职责原则。如果需要修改设备和命令之间的关系，只需要修改 `SystemInitializer` 类即可，无需改动其他代码。


在我们的例子中，每个命令类（如 RelayOnCommand 和 MotorOffCommand）封装了一个函数（即 Execute 方法），并且这个函数与类中的其他方法和变量是独立的。这个函数是类的唯一职责，也是类存在的唯一原因。

这种方式看起来与面向对象编程的原则背道而驰，因为它将函数提升到了类的地位。然而，正是在这种面向对象编程和函数式编程的交界处，发生了一些有趣的事情。

首先，由于每个命令类只有一个职责，它们都很小，很简单，易于理解和维护。其次，由于命令类封装了函数并对外提供了统一的接口（即 Execute 方法），我们可以在不改变代码其他部分的情况下添加新的命令类，这有助于提高系统的可扩展性。

更重要的是，命令模式使我们能够以一种抽象和高层的方式处理函数。我们可以将函数作为对象在系统中传递，我们可以将函数与其他对象（如传感器）绑定，我们甚至可以将函数存储在数据结构中以供稍后使用。这些都是在没有命令模式的情况下很难或者不可能实现的。

总的来说，命令模式是一种非常强大的设计模式，它可以帮助我们更好地管理复杂性，提高可扩展性，并打开一扇到函数式编程世界的大门。