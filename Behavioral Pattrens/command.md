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

---

---

“COMMAND”模式还有另一种常见用途，我们将在支付问题中发现其有用：创建和执行事务。

我将创建一个模拟的员工数据库，以及一些命令类来表示添加和删除员工的操作。以下是相应的C#代码：

首先，我们需要一个表示员工的类：

```csharp
public class Employee
{
    public string Id { get; set; }
    public string Name { get; set; }
    public PayClassification PayClassification { get; set; }
    // 其他员工信息
}


```

我们创建一个表示PayClassification的类：

```csharp
public class PayClassification
{
    public decimal Salary { get; set; }
    // 其他工资分类相关的信息
}

```

然后，我们需要一个表示员工数据库的类：

```csharp
public class EmployeeDatabase
{
    private Dictionary<string, Employee> employees = new Dictionary<string, Employee>();

    public void AddEmployee(Employee employee)
    {
        employees[employee.Id] = employee;
    }

    public void RemoveEmployee(string id)
    {
        employees.Remove(id);
    }

    // 其他数据库操作
}

```

接下来，我们需要一个表示命令的接口：

```csharp

public interface ICommand
{
    bool Validate();
    void Execute();
}
```

然后，我们可以创建一些表示添加和删除员工操作的命令类：

```csharp

public class AddEmployeeCommand : ICommand
{
    private EmployeeDatabase database;
    private Employee employee;
    private PayClassification payClassification;

    public AddEmployeeCommand(EmployeeDatabase database, Employee employee, PayClassification payClassification)
    {
        this.database = database;
        this.employee = employee;
        this.payClassification = payClassification;
    }

    public bool Validate()
    {
        // 这里可以检查员工信息和工资分类是否有效，以及数据库中是否已经存在这个员工
        return true;
    }

    public void Execute()
    {
        employee.PayClassification = payClassification;
        database.AddEmployee(employee);
    }
}


```

使用 `AddEmployeeCommand` 类实例化一个对象，然后使用它的 `Validate` 方法进行验证，如果验证成功，则使用 `Execute` 方法来执行操作。以下是使用 `AddEmployeeCommand` 的示例代码：

```csharp
// 创建一个 EmployeeDatabase 实例
var database = new EmployeeDatabase();

// 创建一个 Employee 实例
var employee = new Employee 
{
    Id = "123",
    Name = "John Doe",
};

// 创建一个 PayClassification 实例
var payClassification = new PayClassification 
{
    Salary = 5000M,
};

// 创建一个 AddEmployeeCommand 实例
var command = new AddEmployeeCommand(database, employee, payClassification);

// 验证命令
if (command.Validate())
{
    // 执行命令
    command.Execute();
}
```

这段代码首先创建了一个 `EmployeeDatabase` 实例，然后创建了一个 `Employee` 实例和一个 `PayClassification` 实例。然后，它创建了一个 `AddEmployeeCommand` 实例，并使用 `EmployeeDatabase` 实例、`Employee` 实例和 `PayClassification` 实例作为参数。然后，它调用 `AddEmployeeCommand` 实例的 `Validate` 方法进行验证，如果验证成功，它将调用 `Execute` 方法来添加员工到数据库。

这段描述在强调命令模式所带来的解耦优势。下面我将逐点解释这段描述的含义。

1. **解耦用户数据采集、数据验证和操作、以及业务对象的代码：** 在我们的示例中，`AddEmployeeCommand` 就是实现了这种解耦。它从构造函数接收员工和工资分类信息（这些数据通常来自用户输入），然后在 `Validate` 方法中进行验证，最后在 `Execute` 方法中进行操作。这个过程与员工（Employee）业务对象分离，员工对象不需要知道如何验证和添加自身。

2. **GUI 代码中不包含验证和执行算法：** 如果我们有一个图形用户界面（GUI）用于添加员工，我们只需要在用户提交表单时创建一个新的 `AddEmployeeCommand` 对象，并调用其 `Validate` 和 `Execute` 方法。GUI 代码不需要知道如何验证员工数据，也不需要知道如何更新数据库。

3. **与其他接口的复用：** 命令模式使我们的验证和执行代码可以被其他接口复用。例如，我们也可以为命令行界面或者 API 端点使用 `AddEmployeeCommand`，而无需更改 `AddEmployeeCommand` 类的任何代码。

4. **分离了数据库操作代码和业务实体：** `AddEmployeeCommand` 类知道如何操作 `EmployeeDatabase` 来添加新的员工。这部分代码完全独立于 `Employee` 类。如果我们更改数据库操作的方式，我们只需要在 `AddEmployeeCommand` 中进行更改，而不会影响到 `Employee` 类。

所以，从这个例子中可以看出，使用命令模式可以显著地解耦数据采集、数据验证和操作以及业务对象的代码，增强了代码的复用性和可维护性。

这段描述在解释“时间解耦”的概念，即命令模式允许我们在数据采集后随时验证和执行操作，而不必立即执行。让我们来解释一下如何从上面的示例中理解这个概念。

在上面的 `AddEmployeeCommand` 示例中，当我们创建一个新的 `AddEmployeeCommand` 对象并从用户那里获取数据时，我们并不需要立即验证或执行这个命令。我们可以将这个 `AddEmployeeCommand` 对象保存起来，稍后再进行验证和执行。

例如，我们可以创建一个 `List<AddEmployeeCommand>`，将整天收集到的 `AddEmployeeCommand` 对象都添加到这个列表中。然后在凌晨时分，我们遍历这个列表，对每个 `AddEmployeeCommand` 对象进行验证，如果验证通过，则执行它。

以下是相关的示例代码：

```csharp
// 创建一个列表来保存 AddEmployeeCommand 对象
var commands = new List<AddEmployeeCommand>();

// 在整天中，我们可能会从用户那里获取数据，并创建 AddEmployeeCommand 对象
var command = new AddEmployeeCommand(database, employee, payClassification);
commands.Add(command);

// ... 这样的过程可能会在一天中多次发生 ...

// 在凌晨时分，我们开始验证和执行命令
foreach (var cmd in commands)
{
    if (cmd.Validate())
    {
        cmd.Execute();
    }
}
```

这就是时间解耦的概念。命令模式使我们可以在数据采集后的任何时间点验证和执行命令，而无需立即进行。这在处理需要在特定时间执行的操作时非常有用。

可以稍微修改上述代码以满足你的需求：立即进行验证，但等到午夜才执行所有命令。具体如下：

```csharp
// 创建一个列表来保存 AddEmployeeCommand 对象
var commands = new List<AddEmployeeCommand>();

// 在整天中，我们可能会从用户那里获取数据，并创建 AddEmployeeCommand 对象
var command = new AddEmployeeCommand(database, employee, payClassification);

// 立即进行验证
if (command.Validate())
{
    // 如果验证通过，将命令添加到列表中
    commands.Add(command);
}
else
{
    // 处理验证失败的情况，例如显示错误信息
}

// ... 这样的过程可能会在一天中多次发生 ...

// 在凌晨时分，我们开始执行所有验证过的命令
foreach (var cmd in commands)
{
    cmd.Execute();
}
```

在这个修改后的版本中，我们立即对每个 `AddEmployeeCommand` 对象进行验证。只有验证通过的命令才会被添加到列表中。然后，我们在午夜时分遍历列表，执行所有已验证的命令。这样，我们确保了只有有效的命令会被执行，而且所有的命令都会在午夜时分执行。


这个描述实际上在解释如何使用命令模式来实现 "撤销" 功能。在 .NET 中，我们可以定义一个命令接口（或者基类），在其中添加一个 `Undo` 方法。然后，每个具体的命令类都需要实现这个 `Undo` 方法。在执行命令时，我们把命令对象保存在一个栈中，以便之后可以执行 "撤销" 操作。下面是基于这个描述的一个简单的 C# 代码示例：

```csharp
public interface ICommand
{
    void Execute();
    void Undo();
}

public class DrawCircleCommand : ICommand
{
    private Canvas _canvas;
    private Circle _circle;

    public DrawCircleCommand(Canvas canvas, Circle circle)
    {
        _canvas = canvas;
        _circle = circle;
    }

    public void Execute()
    {
        _canvas.DrawCircle(_circle);
    }

    public void Undo()
    {
        _canvas.RemoveCircle(_circle);
    }
}

public class Canvas
{
    private Stack<ICommand> _commands = new Stack<ICommand>();

    public void ExecuteCommand(ICommand command)
    {
        command.Execute();
        _commands.Push(command);
    }

    public void UndoCommand()
    {
        if (_commands.Count > 0)
        {
            var command = _commands.Pop();
            command.Undo();
        }
    }

    public void DrawCircle(Circle circle)
    {
        // Logic to draw a circle
    }

    public void RemoveCircle(Circle circle)
    {
        // Logic to remove a circle
    }
}
```

在上述示例中，我们有一个 `ICommand` 接口，它定义了 `Execute` 和 `Undo` 方法。`DrawCircleCommand` 类实现了这个接口，并在 `Execute` 方法中画一个圆，在 `Undo` 方法中移除一个圆。

`Canvas` 类持有一个命令栈 `_commands`，在执行命令时会把命令对象推入栈中。当执行 "撤销" 操作时，会从栈中弹出最近执行的命令对象，然后调用其 `Undo` 方法。

需要注意的是，这只是一个简化的示例。在实际的应用程序中，你可能需要处理更多的细节，例如对命令的参数进行验证，处理可能的执行错误，以及支持 "重做" 操作等。


### active object

active object 模式是一种强大的设计模式，有助于实现多线程系统。这种模式将每个方法的调用、其参数和每个方法对应的返回值封装为一个对象。这些对象被异步分配，存储在一个队列中，并由一个单独的线程调用。这允许方法发送者和接收者的解耦，从而实现了方法的异步执行。


让我们假设我们有一个名为Printer的类，它有一个Print(string message)方法。现在，我们想使用Active Object模式来异步执行这个方法。下面是我们如何在C#中实现这一目标：

当然可以，让我们将上述内容翻译为中文。

首先，我们需要创建一个命令接口，该接口代表一个操作。在我们的案例中，它将有一个名为 `Execute` 的方法。

```csharp
public interface ICommand
{
    void Execute();
}
```

接下来，我们将创建一个实现此接口的类，名为 `PrintCommand`。这个类将封装 `Printer` 类的 `Print` 方法。

```csharp
public class PrintCommand : ICommand
{
    private Printer _printer;
    private string _message;

    public PrintCommand(Printer printer, string message)
    {
        _printer = printer;
        _message = message;
    }

    public void Execute()
    {
        _printer.Print(_message);
    }
}
```

然后我们将实现活动对象模式。我们将创建一个名为 `ActiveObjectEngine` 的类，该类维护了命令对象队列，并异步执行它们。

```csharp
using System.Collections.Generic;
using System.Threading;

public class ActiveObjectEngine
{
    private Queue<ICommand> _commands = new Queue<ICommand>();

    public void AddCommand(ICommand command)
    {
        lock (_commands)
        {
            _commands.Enqueue(command);
        }
        ProcessCommands();
    }

    private void ProcessCommands()
    {
        ThreadPool.QueueUserWorkItem(_ =>
        {
            while (_commands.Count > 0)
            {
                ICommand command;
                lock (_commands)
                {
                    command = _commands.Dequeue();
                }
                command.Execute();
            }
        });
    }
}
```

最后，这就是我们如何使用 `ActiveObjectEngine` 来异步执行 `Print` 方法的方式：

```csharp
var printer = new Printer();
var engine = new ActiveObjectEngine();

engine.AddCommand(new PrintCommand(printer, "Hello, world!"));
engine.AddCommand(new PrintCommand(printer, "Active Object pattern is awesome!"));
```

在这个例子中，`ActiveObjectEngine` 是一个类，它封装了方法发送者（这是 `PrintCommand` 对象）和接收者（这是 `Printer` 对象）。它在队列中存储命令对象，并使用线程池异步分派它们，从而实现异步方法执行。

---

当然可以，我会逐段翻译上述内容：

```csharp
public class DelayedTyper : Command
{
    private long itsDelay;
    private char itsChar;
    private static bool stop = false;
    private static ActiveObjectEngine engine = new ActiveObjectEngine();
```
`DelayedTyper`类继承自`Command`接口，这就意味着它需要实现`Execute()`方法。它有两个实例字段，`itsDelay`和`itsChar`，分别用来保存每次打印字符的延迟和要打印的字符。还有一个静态的`ActiveObjectEngine`实例，用来管理所有的`DelayedTyper`命令，以及一个`stop`布尔值，用来指示何时停止所有`DelayedTyper`命令。

```csharp
private class StopCommand : Command
{
    public void Execute()
    {
        DelayedTyper.stop = true;
    }
}
```
内部类`StopCommand`是`Command`的简单实现，当它被执行时，会将`DelayedTyper`类中的`stop`字段设置为`true`。这个命令用于指示所有正在运行的`DelayedTyper`命令停止打印字符。

```csharp
public static void Main(string[] args)
{
    engine.AddCommand(new DelayedTyper(100, '1'));
    engine.AddCommand(new DelayedTyper(300, '3'));
    engine.AddCommand(new DelayedTyper(500, '5'));
    engine.AddCommand(new DelayedTyper(700, '7'));
    Command stopCommand = new StopCommand();
    engine.AddCommand(new SleepCommand(20000, engine, stopCommand));
    engine.Run();
}
```
在`Main`方法中，创建了不同的`DelayedTyper`命令并将它们添加到`ActiveObjectEngine`中。同时，也创建了一个`StopCommand`，并通过一个将在20000毫秒后执行的`SleepCommand`将它添加到`ActiveObjectEngine`中。然后调用`engine.Run()`开始执行所有的命令。

```csharp
public DelayedTyper(long delay, char c)
{
    itsDelay = delay;
    itsChar = c;
}
```
`DelayedTyper`的构造器将传入的延迟和字符赋值给实例字段。

```csharp
public void Execute()
{
    Console.Write(itsChar);
    if (!stop)
        DelayAndRepeat();
}
```
在`Execute`方法中，将字符打印到控制台。然后，如果`stop`标志没有设置，就调用`DelayAndRepeat`方法。

```csharp
private void DelayAndRepeat()
{
    engine.AddCommand(new SleepCommand(itsDelay, engine, this));
}
```
`DelayAndRepeat`方法创建一个新的`SleepCommand`，并将当前实例作为唤醒命令和`itsDelay`字段作为睡眠时长，然后将这个命令添加到`ActiveObjectEngine`。这确保了在`itsDelay`毫秒后，这个`DelayedTyper`命令会再次被执行，创建一个只有当`stop`标志设置为`true`时才会停止的循环。

总的来说，`DelayedTyper`类代表一个命令，该命令会在规定的间隔内持续地在控制台打印特定的字符，直到执行一个`StopCommand`为止。


`DelayedTyper`程序的输出会有一些不同，这是因为该程序是一个多线程的系统，并且依赖于时间间隔。由于CPU时钟和真实时钟（即系统的硬件时钟）之间存在微小的不同步现象，这就使得每个`DelayedTyper`命令在精确的时间间隔内被执行的可能性并不总是100%。这就导致了每次运行程序时，字符打印的顺序可能会有所不同。

这个现象反映出了所谓的"非确定性"行为，这是多线程系统的一个典型特征。即，尽管我们能预测出这个系统的大致行为（例如，它会打印出1, 3, 5, 7字符，并且在20秒后停止），但是，我们不能精确地预知这些字符被打印出的精确顺序。这是因为它们的执行顺序受到了处理器调度和时钟同步精度的影响。

在给定的两个示例输出中：

```
135711311511371113151131715131113151731111351113711531111357...
135711131513171131511311713511131151731113151131711351113117...
```
你可以看到，尽管在每个例子中都打印出了1, 3, 5, 7字符，但是它们的顺序却有所不同。这就是由于上述非确定性行为导致的结果。


为什么说这是多线程的程序？RTC 特点不就是在一个线程了运行吗？

确实，"run-to-completion" (RTC)任务通常在同一线程中按序运行，每个任务都会完全执行完毕，然后再运行下一个任务。然而，这种方式模拟的是多线程编程的一些行为。

在这个例子中，虽然所有的`Command`实例在同一个线程中运行，但每一个`Command`实例（例如`DelayedTyper`）都表现得像是在它自己的"线程"中运行。每个`DelayedTyper`命令在一段时间后（通过`SleepCommand`模拟）执行，并且似乎与其他的`DelayedTyper`命令并发运行，因为它们各自的打印任务相互之间是交错进行的。

所以，虽然这段代码实际上在单线程环境中运行，但它通过使用`Command`模式和`SleepCommand`来模拟多线程环境中的行为。这使得这段代码能够展示出类似于多线程环境中的非确定性行为：尽管每个`DelayedTyper`命令的行为（打印特定字符并等待特定延迟）是确定的，但是这些命令的交错执行顺序是非确定的，因为它受到处理器调度和时钟同步精度的影响。

非常抱歉没有解释清楚，我会更详细地解释并提供一个时间轴作为解释的基础。

假设我们运行以下代码：

```csharp
engine.AddCommand(new DelayedTyper(100, '1'));
engine.AddCommand(new DelayedTyper(300, '3'));
engine.AddCommand(new DelayedTyper(500, '5'));
engine.AddCommand(new DelayedTyper(700, '7'));
Command stopCommand = new StopCommand();
engine.AddCommand(new SleepCommand(20000, engine, stopCommand));
engine.Run();
```

以下是一个时间轴的示例，展示了程序的执行过程：

```
0ms: 创建并添加 DelayedTyper(100, '1') 到 ActiveObjectEngine
0ms: 创建并添加 DelayedTyper(300, '3') 到 ActiveObjectEngine
0ms: 创建并添加 DelayedTyper(500, '5') 到 ActiveObjectEngine
0ms: 创建并添加 DelayedTyper(700, '7') 到 ActiveObjectEngine
0ms: 创建 StopCommand，并添加到 ActiveObjectEngine 中的 SleepCommand 中
0ms: 调用 engine.Run()，开始执行命令
```

在这个时间轴中，所有的`DelayedTyper`命令和`SleepCommand`命令被添加到了 `ActiveObjectEngine`。接下来，我们将看到每个`DelayedTyper`命令如何按照设定的延迟时间进行执行。
(下面的时间轴不大对，可能是chatgpt瞎编的)

```
0ms: DelayedTyper(100, '1') 打印字符 '1'
0ms: DelayedTyper(300, '3') 打印字符 '3'
0ms: DelayedTyper(500, '5') 打印字符 '5'
0ms: DelayedTyper(700, '7') 打印字符 '7'
100ms: DelayedTyper(100, '1') 打印字符 '1'（第二次）
200ms: DelayedTyper(100, '1') 打印字符 '1'（第三次）
300ms: DelayedTyper(300, '3') 打印字符 '3'（第二次）
400ms: DelayedTyper(100, '1') 打印字符 '1'（第四次）
500ms: DelayedTyper(500, '5') 打印字符 '5'（第二次）
600ms: DelayedTyper(100, '1') 打印字符 '1'（第五次）
700ms: DelayedTyper(700, '7') 打印字符 '7'（第二次）
...
20000ms: SleepCommand(20000) 延迟时间已过，激活 StopCommand
20000ms: 所有命令执行完毕，engine.Run() 返回

```

在这个时间轴中，我们可以看到各个`DelayedTyper`命令按照设定的延迟时间在不同的时间点打印出相应的字符。由于命令是按序执行的，每个`DelayedTyper`命令都会在前一个命令完成后立即执行。因此，我们看到了字符的交错打印。

值得注意的是，`SleepCommand(20000)`在延迟时间过后激活了`StopCommand`，停止了所有命令的执行，并且程序结束。

总结来说，尽管这段代码是在单线程环境中运行的，但通过使用`DelayedTyper`命令的设定延迟和顺序执行，我们模拟了多线程环境中的行为。这就是为什么字符打印的顺序在每次运行时可能会有所不同的原因。这种非确定性的行为是多线程系统的典型特征，因为它受到处理器调度和时钟同步精度的影响。

## .net framework



ThreadPool.QueueUserWorkItem 方法：这个方法可以将一个方法（或命令）排入队列以供线程池中的线程执行。虽然这个方法并没有直接使用命令模式，但它的工作方式类似于命令模式：我们可以将一个函数（或命令）封装在一个对象中（例如一个匿名方法或 lambda 表达式），然后将这个对象传递给 QueueUserWorkItem 方法，该方法会在一个线程中执行这个函数（或命令）。

很好，我会为你提供一个代码示例来解释这段描述。

我们可以将一个方法或命令封装在一个匿名函数或 lambda 表达式中，然后将其传递给 `ThreadPool.QueueUserWorkItem` 方法。这个方法会将该命令添加到线程池的工作队列中，并在有可用线程时执行该命令。

让我们假设我们有一个名为 `PerformAction` 的命令，我们需要在一个新的线程中执行这个命令：

```csharp
void PerformAction(object state)
{
    // 执行一些长时间运行的操作，例如访问网络或数据库。
    Console.WriteLine("Performing action...");
}
```

然后，我们可以使用 `ThreadPool.QueueUserWorkItem` 方法来将 `PerformAction` 方法添加到线程池的工作队列中：

```csharp
ThreadPool.QueueUserWorkItem(new WaitCallback(PerformAction));
```

在这里，我们创建了一个 `WaitCallback` 委托，它是 `PerformAction` 方法的封装。然后我们将这个委托传递给 `QueueUserWorkItem` 方法。

这个例子体现了命令模式的思想，即将一个操作（或命令）封装在一个对象中（在这个例子中，该对象是 `WaitCallback` 委托），然后将这个对象传递给其他方法或对象，由它们来决定何时和如何执行这个命令。虽然 .NET 框架并没有明确提供命令模式的实现，但 `ThreadPool.QueueUserWorkItem` 方法的工作方式与命令模式非常类似。


2. ASP.NET Core中间件： 虽然没有明确标明，但ASP.NET Core中间件遵循命令模式。HTTP请求管道中的每件中间件都可以被看作是具有Invoke或InvokeAsync方法的命令（命令模式术语中的Execute方法）。中间件组件是解耦的，可以独立添加、删除或重新排序。

在ASP.NET Core中，中间件（Middleware）用于处理HTTP请求和响应。你可以将每个中间件看作一个命令，其 `Invoke` 或 `InvokeAsync` 方法相当于命令模式中的 `Execute` 方法。

以下是一个简单的ASP.NET Core中间件的示例，它模仿命令模式：

```csharp
public class MyMiddleware
{
    private readonly RequestDelegate _next;

    public MyMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext httpContext)
    {
        // 在此处处理请求，类似于命令模式中的Execute方法
        // ...

        // Call the next middleware in the pipeline
        await _next(httpContext);
    }
}
```

在这个例子中，`MyMiddleware` 类相当于命令模式中的一个命令。它有一个 `InvokeAsync` 方法，这个方法接受一个 `HttpContext` 对象，然后执行一些操作。在操作完成之后，它调用 `_next` 委托来执行管道中的下一个中间件。

在你的 `Startup.cs` 文件中，你可以将这个中间件添加到你的HTTP请求管道中：

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        // Add MyMiddleware to the pipeline
        app.UseMiddleware<MyMiddleware>();

        // ...
    }
}
```

你可以看到，我们在 `Configure` 方法中调用了 `UseMiddleware` 方法来添加我们的中间件到请求管道中。这个方法接受一个类型参数，该类型必须是一个中间件类。这种方式使得中间件组件可以独立地被添加、移除或重新排序，这也是命令模式的一个重要特性。

总的来说，虽然ASP.NET Core的中间件并没有明确地标识为使用命令模式，但它的设计理念与命令模式是相符的：将请求处理逻辑封装在一个对象中，然后通过一个统一的接口（`Invoke` 或 `InvokeAsync` 方法）来调用这个对象。


## Design Patterns

例如，用户界面工具包包含像按钮和菜单这样的对象，它们会响应用户输入来执行请求。但是，工具包不能在按钮或菜单中明确地实现请求，因为只有使用工具包的应用程序才知道应该在哪个对象上做什么。

作为工具包设计者，我们无法知道请求的接收者或执行请求的操作。

命令模式允许工具包对象通过将请求本身转化为对象来向未指定的应用对象发出请求。

可以使用命令对象轻松实现菜单。菜单中的每一个选项都是MenuItem类的一个实例。一个应用程序类创建这些菜单和菜单项，以及其他用户界面。应用程序类还跟踪用户打开的文档对象。

应用程序将每个MenuItem与一个具体的Command子类实例进行配置。当用户选择一个MenuItem时，MenuItem调用其命令的Execute，而Execute执行操作。MenuItem不知道它们使用的是哪个Command子类。命令子类存储请求的接收者，并在接收者上调用一个或多个操作。


例如，PasteCommand支持从剪贴板粘贴文本到一个文档。PasteCommand的接收者是在实例化时提供的文档对象。Execute操作在接收文档上调用Paste。OpenCommand的Execute操作不同：它提示用户输入一个文档名称，创建一个相应的文档对象，将文档添加到接收应用程序，并打开文档。

有时，一个MenuItem需要执行一系列命令。例如，可以从CenterDocumentCommand对象和NormalSizeCommand对象构造一个用于将页面居中显示的MenuItem。因为这种将命令串联在一起的方式很常见，所以我们可以定义一个MacroCommand类，允许MenuItem执行开放式的命令数。

MacroCommand是一个简单地执行一系列命令的ConcreteCommand子类。MacroCommand没有明确的接收者，因为它的命令序列定义了自己的接收者。

在这些示例中，请注意命令模式是如何将调用操作的对象和知道如何执行操作的对象解耦的。这给我们在设计用户界面时提供了很大的灵活性。一个应用程序可以通过让菜单和按钮共享同一个ConcreteCommand子类的实例，同时提供一个菜单和一个按钮界面给一个功能。我们可以动态替换命令，这对于实现上下文敏感的菜单是有用的。我们还可以通过组合命令来支持命令脚本。所有这些都是可能的，因为发出请求的对象只需要知道如何发出它，而不需要知道请求如何被执行。

示例代码
这里的 C++ 代码示例展示了在上述文章中的 "动机" 部分中提到的 Command 类的实现，我们将定义 OpenCommand、PasteCommand 和 MacroCommand。首先是抽象的 Command 类：

```csharp
public abstract class Command {
    public abstract void Execute();
}
```

OpenCommand 会打开用户提供的文档名称。在其构造器中，必须传递一个 Application 对象。AskUser 是一个实现的方法，用于提示用户打开哪个文档。

```csharp
public class OpenCommand : Command {
    private Application _application;
    private string _response;

    public OpenCommand(Application application){
        _application = application;
    }

    public override void Execute() {
        string name = AskUser();
        if (name != null) {
            Document document = new Document(name);
            _application.Add(document);
            document.Open();
        }
    }

    protected virtual string AskUser() {
        // implementation of user prompt goes here
        return "";
    }
}
```

PasteCommand 必须将 Document 对象作为其接收者传递。接收者作为参数传递给 PasteCommand 的构造函数。

```csharp
public class PasteCommand : Command {
    private Document _document;

    public PasteCommand(Document doc) {
        _document = doc;
    }

    public override void Execute() {
        _document.Paste();
    }
}
```

对于不可撤销并且不需要参数的简单命令，我们可以使用一个类模板来参数化命令的接收者。我们将为此类命令定义一个模板子类 SimpleCommand。SimpleCommand 是由接收者类型参数化的，并且维护了一个接收者对象和一个作为成员函数指针存储的操作之间的绑定。

```csharp
public class SimpleCommand<TReceiver> : Command {
    private readonly Action<TReceiver> _action;
    private readonly TReceiver _receiver;

    public SimpleCommand(TReceiver receiver, Action<TReceiver> action) {
        _receiver = receiver;
        _action = action;
    }

    public override void Execute() {
        _action(_receiver);
    }
}
```

要创建一个在 MyClass 的实例上调用 Action 的命令，客户端只需写：

```csharp
MyClass receiver = new MyClass();
Command aCommand = new SimpleCommand<MyClass>(receiver, receiver.Action);
aCommand.Execute();
```

请注意，此解决方案仅适用于简单的命令。更复杂的命令需要跟踪它们的接收者，也可能需要跟踪参数和/或撤销状态，这需要一个 Command 子类。

MacroCommand 管理一系列子命令，并提供添加和删除子命令的操作。无需显式接收者，因为子命令已经定义了它们的接收者。

```csharp
public class MacroCommand : Command {
    private List<Command> _cmds;

    public MacroCommand() {
        _cmds = new List<Command>();
    }

    public void Add(Command cmd) {
        _cmds.Add(cmd);
    }

    public void Remove(Command cmd) {
        _cmds.Remove(cmd);
    }

    public override void Execute() {
        foreach (

Command cmd in _cmds) {
            cmd.Execute();
        }
    }
}
```

MacroCommand 的关键在于其 Execute 成员函数。这会遍历所有子命令，并对它们每一个执行 Execute。

请注意，如果 MacroCommand 实现一个 Unexecute 操作，那么必须按照与 Execute 的实现相反的顺序对其子命令进行 unexecute。

最后，MacroCommand 必须提供操作以管理其子命令。MacroCommand 也负责删除其子命令。