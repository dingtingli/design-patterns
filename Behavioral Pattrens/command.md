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