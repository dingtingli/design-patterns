# observer 观察者

## agile 笔记

## 极客时间 笔记
观察者模式会对应不同的代码实现方式：有同步阻塞的实现方式，也有异步非阻塞的实现方式；有进程内的实现方式，也有跨进程的实现方式


一块实现一个基于观察者模式的异步非阻塞的 EventBus

观察者模式（Observer Design Pattern）也被称为发布订阅模式（Publish-Subscribe Design Pattern）。在 GoF 的《设计模式》一书中，它的定义是这样的：


Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

翻译成中文就是：在对象之间定义一个一对多的依赖，当一个对象状态改变的时候，所有依赖的对象都会自动收到通知。

观察者模式是一个比较抽象的模式，根据不同的应用场景和需求，有完全不同的实现方式。现在，我们先来看其中最经典的一种实现方式。这也是在讲到这种模式的时候，很多书籍或资料给出的最常见的实现方式

```java
public interface Subject {
  void registerObserver(Observer observer);
  void removeObserver(Observer observer);
  void notifyObservers(Message message);
}

public interface Observer {
  void update(Message message);
}

public class ConcreteSubject implements Subject {
  private List<Observer> observers = new ArrayList<Observer>();

  @Override
  public void registerObserver(Observer observer) {
    observers.add(observer);
  }

  @Override
  public void removeObserver(Observer observer) {
    observers.remove(observer);
  }

  @Override
  public void notifyObservers(Message message) {
    for (Observer observer : observers) {
      observer.update(message);
    }
  }

}

public class ConcreteObserverOne implements Observer {
  @Override
  public void update(Message message) {
    //TODO: 获取消息通知，执行自己的逻辑...
    System.out.println("ConcreteObserverOne is notified.");
  }
}

public class ConcreteObserverTwo implements Observer {
  @Override
  public void update(Message message) {
    //TODO: 获取消息通知，执行自己的逻辑...
    System.out.println("ConcreteObserverTwo is notified.");
  }
}

public class Demo {
  public static void main(String[] args) {
    ConcreteSubject subject = new ConcreteSubject();
    subject.registerObserver(new ConcreteObserverOne());
    subject.registerObserver(new ConcreteObserverTwo());
    subject.notifyObservers(new Message());
  }
}
```

实际上，上面的代码算是观察者模式的“模板代码”，只能反映大体的设计思路。在真实的软件开发中，并不需要照搬上面的模板代码。观察者模式的实现方法各式各样，函数、类的命名等会根据业务场景的不同有很大的差别，比如 register 函数还可以叫作 attach，remove 函数还可以叫作 detach 等等。不过，万变不离其宗，设计思路都是差不多的。原理和代码实现都非常简单，也比较好理解，不需要我过多的解释。

### 实列 1

原始需求：投资理财系统，用户注册成功之后，我们会给用户发放投资体验金

简单实现：

```java
public class UserController {
  private UserService userService; // 依赖注入
  private PromotionService promotionService; // 依赖注入

  public Long register(String telephone, String password) {
    //省略输入参数的校验代码
    //省略userService.register()异常的try-catch代码
    long userId = userService.register(telephone, password);
    promotionService.issueNewUserExperienceCash(userId);
    return userId;
  }
}
```
做了两件事情，注册和发放体验金，违反单一职责原则，但是，如果没有扩展和修改的需求，现在的代码实现是可以接受的。不能过度设计

需求改变：
用户注册成功之后，不再发放体验金，而是改为发放优惠券，并且还要给用户发送一封“欢迎注册成功”的站内信。这种情况下，我们就需要频繁地修改 register() 函数中的代码，违反开闭原则。而且，如果注册成功之后需要执行的后续操作越来越多，那 register() 函数的逻辑会变得越来越复杂，也就影响到代码的可读性和可维护性

利用观察者模式，我对上面的代码进行了重构：

```java

public interface RegObserver {
  void handleRegSuccess(long userId);
}

public class RegPromotionObserver implements RegObserver {
  private PromotionService promotionService; // 依赖注入

  @Override
  public void handleRegSuccess(long userId) {
    promotionService.issueNewUserExperienceCash(userId);
  }
}

public class RegNotificationObserver implements RegObserver {
  private NotificationService notificationService;

  @Override
  public void handleRegSuccess(long userId) {
    notificationService.sendInboxMessage(userId, "Welcome...");
  }
}

public class UserController {
  private UserService userService; // 依赖注入
  private List<RegObserver> regObservers = new ArrayList<>();

  // 一次性设置好，之后也不可能动态的修改
  public void setRegObservers(List<RegObserver> observers) {
    regObservers.addAll(observers);
  }

  public Long register(String telephone, String password) {
    //省略输入参数的校验代码
    //省略userService.register()异常的try-catch代码
    long userId = userService.register(telephone, password);

    for (RegObserver observer : regObservers) {
      observer.handleRegSuccess(userId);
    }

    return userId;
  }
}
```

当我们需要添加新的观察者的时候，比如，用户注册成功之后，推送用户注册信息给大数据征信系统，基于观察者模式的代码实现，UserController 类的 register() 函数完全不需要修改，只需要再添加一个实现了 RegObserver 接口的类，并且通过 setRegObservers() 函数将它注册到 UserController 类中即可。

---
chatgpt

观察者模式也称为发布-订阅模式，可以在对象之间定义一个一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知并被自动更新。

我们可以将注册事件作为一个主题，当注册成功后，向所有订阅该事件的观察者（例如投资体验金服务、欢迎邮件服务等）发送通知。

下面是一个简化的 Java 实现，用于演示如何应用观察者模式：

```java
// 定义事件
public class UserRegisterEvent {
  private long userId;
  
  public UserRegisterEvent(long userId) {
    this.userId = userId;
  }
  
  public long getUserId() {
    return userId;
  }
}

// 定义观察者接口
public interface Observer {
  void update(UserRegisterEvent event);
}

// 实现 PromotionService 观察者
public class PromotionService implements Observer {
  public void update(UserRegisterEvent event) {
    long userId = event.getUserId();
    this.issueNewUserExperienceCash(userId);
  }
  
  public void issueNewUserExperienceCash(long userId) {
    // 发放体验金
  }
}

// 实现 MailService 观察者
public class MailService implements Observer {
  public void update(UserRegisterEvent event) {
    long userId = event.getUserId();
    this.sendWelcomeMail(userId);
  }
  
  public void sendWelcomeMail(long userId) {
    // 发送邮件
  }
}

// 定义主题
public class UserRegisterSubject {
  private List<Observer> observers = new ArrayList<>();
  
  public void register(Observer observer) {
    observers.add(observer);
  }
  
  public void notifyObservers(UserRegisterEvent event) {
    for (Observer observer : observers) {
      observer.update(event);
    }
  }
}

// 修改 UserController
public class UserController {
  private UserService userService; // 依赖注入
  private UserRegisterSubject userRegisterSubject; // 依赖注入

  public Long register(String telephone, String password) {
    //省略输入参数的校验代码
    //省略userService.register()异常的try-catch代码
    long userId = userService.register(telephone, password);
    
    // 创建事件并通知观察者
    UserRegisterEvent event = new UserRegisterEvent(userId);
    userRegisterSubject.notifyObservers(event);

    return userId;
  }
}
```

在这个实现中，`UserController` 中的 `register()` 方法现在将不会随着需求的变化而变化。新的需求可以通过添加新的观察者来实现，而不需要修改 `register()` 方法的代码。

如果你希望实现异步非阻塞的模式，那么你可能需要在通知观察者时启动新的线程。下面是一种可能的实现方式，使用了 Java 的 `ExecutorService` 来管理线程：

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

// 定义主题
public class UserRegisterSubject {
  private List<Observer> observers = new ArrayList<>();
  
  // 创建一个线程池
  private ExecutorService threadPool = Executors.newFixedThreadPool(4);

  public void register(Observer observer) {
    observers.add(observer);
  }
  
  public void notifyObservers(UserRegisterEvent event) {
    for (Observer observer : observers) {
      threadPool.execute(() -> observer.update(event));
    }
  }
}
```

在上述代码中，我们定义了一个固定大小的线程池。当通知观察者时，每个观察者的 `update` 方法都会在一个新的线程中执行。这样，`notifyObservers` 方法将立即返回，而不需要等待所有观察者完成它们的 `update` 方法。

需要注意的是，如果观察者的 `update` 方法需要大量的计算或者 I/O 操作，那么线程池的大小可能需要根据实际情况进行调整，以防止线程过多而耗尽系统资源。此外，还要注意线程安全问题，因为现在 `update` 方法可能同时被多个线程调用。

在应用程序退出时，还需要适当地关闭线程池，避免资源泄露。这可以通过在 `UserRegisterSubject` 类中添加一个 `shutdown` 方法来实现，该方法调用 `threadPool.shutdown` 来关闭线程池。

---

### chatgpt与之前的实现之间的比较：

它将需要响应的事件（注册成功）封装在观察者接口 (RegObserver) 的 handleRegSuccess() 方法中。

这两种实现都遵循了观察者模式的主要思想：当主题的状态发生改变时，所有订阅这个主题的观察者都会得到通知并被自动更新。因此，两者的差异主要在于设计的细节。

在我给出的示例中，我创建了一个单独的主题类 (UserRegisterSubject)，并在这个主题类中封装了观察者的注册和通知逻辑。这样做的好处是可以将主题的逻辑与 UserController 类解耦，使得 UserController 类更专注于用户注册的主要逻辑，而不用关心观察者的注册和通知过程。同时，主题类可以复用，适合于多个事件源的情况。

而在你的实现中，你将观察者的注册和通知逻辑直接放在了 UserController 类中。这样做的好处是更加简洁，少了一个主题类，适合于只有一个事件源（如用户注册）的简单情况。但如果观察者的注册和通知逻辑需要在多个地方复用，或者这部分逻辑比较复杂，那么这种设计可能就需要考虑一些改进。

总的来说，两种设计都有其适用的场景，可以根据具体的需求和场景来选择。



设计模式要干的事情就是解耦。创建型模式是将创建和使用代码解耦，结构型模式是将不同功能代码解耦，行为型模式是将不同的行为代码解耦，具体到观察者模式，它是将观察者和被观察者代码解耦。

借助设计模式，我们利用更好的代码结构，将一大坨代码拆分成职责更单一的小类，让其满足开闭原则、高内聚松耦合等特性，以此来控制和应对代码的复杂性，提高代码的可扩展性

### 实现方式

有同步阻塞的实现方式，也有异步非阻塞的实现方式；有进程内的实现方式，也有跨进程的实现方式。

从刚刚的分类方式上来看，它是一种同步阻塞的实现方式。观察者和被观察者代码在同一个线程内执行，被观察者一直阻塞，直到所有的观察者代码都执行完成之后，才执行后续的代码。

可以通过在每个 handleRegSuccess() 函数中，创建一个新的线程执行代码来实现

```java
public class RegPromotionObserver implements RegObserver {
  private PromotionService promotionService; // 依赖注入

  @Override
  public void handleRegSuccess(long userId) {
    new Thread(() -> promotionService.issueNewUserExperienceCash(userId)).start();
  }
}

public class RegNotificationObserver implements RegObserver {
  private NotificationService notificationService;

  @Override
  public void handleRegSuccess(long userId) {
    new Thread(() -> notificationService.sendInboxMessage(userId, "Welcome...")).start();
  }
}
```

这样做有一个弊端，那就是频繁创建和销毁线程会消耗大量的系统资源，特别是当观察者非常多的时候。

如果你非常在意系统性能，并且观察者的数量可能非常多，那么建议还是使用线程池（如上一段代码中的 ExecutorService）来管理线程。


需要为每个观察者创建一个新的线程来执行它们的更新方法。在 Java 中，这可以通过使用 `ExecutorService` 来实现。`ExecutorService` 是一个可以控制线程执行的服务，允许你安排任务在一个或多个工作线程中异步执行。

以下是如何将你的代码改为异步非阻塞模式：

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class UserController {
  private UserService userService; // 依赖注入
  private List<RegObserver> regObservers = new ArrayList<>();
  private ExecutorService executorService; // 线程池服务

  public UserController() {
    executorService = Executors.newFixedThreadPool(10); // 创建拥有10个工作线程的线程池
  }

  public void setRegObservers(List<RegObserver> observers) {
    regObservers.addAll(observers);
  }

  public Long register(String telephone, String password) {
    // 省略输入参数的校验代码
    // 省略userService.register()异常的try-catch代码
    long userId = userService.register(telephone, password);

    for (RegObserver observer : regObservers) {
      executorService.execute(new Runnable() {
        @Override
        public void run() {
          observer.handleRegSuccess(userId);
        }
      });
    }

    return userId;
  }
}
```

在上面的代码中，我在 `UserController` 类中添加了一个 `ExecutorService`，它会处理所有的观察者的更新操作。在 `register()` 方法中，我创建了一个新的 `Runnable` 对象来执行每个观察者的 `handleRegSuccess()` 方法，并将这个 `Runnable` 对象提交给 `ExecutorService`。这样，`handleRegSuccess()` 方法将在一个新的工作线程中异步执行，而 `register()` 方法可以立即返回，从而实现非阻塞。




---

c# 实现版本

当然可以，以下是使用 C# 和 ASP.NET Core 实现的代码：

首先，我们需要定义一个用于事件的类 `UserRegisterEvent`，以及一个用于监听事件的接口 `IObserver`：

```csharp
public class UserRegisterEvent
{
    public long UserId { get; private set; }

    public UserRegisterEvent(long userId)
    {
        UserId = userId;
    }
}

public interface IObserver
{
    Task Update(UserRegisterEvent userRegisterEvent);
}
```

然后，我们可以定义两个实现 `IObserver` 接口的服务，分别是 `PromotionService` 和 `MailService`：

```csharp
public class PromotionService : IObserver
{
    public async Task Update(UserRegisterEvent userRegisterEvent)
    {
        await IssueNewUserExperienceCash(userRegisterEvent.UserId);
    }

    private Task IssueNewUserExperienceCash(long userId)
    {
        // 实现发放体验金的逻辑
    }
}

public class MailService : IObserver
{
    public async Task Update(UserRegisterEvent userRegisterEvent)
    {
        await SendWelcomeMail(userRegisterEvent.UserId);
    }

    private Task SendWelcomeMail(long userId)
    {
        // 实现发送欢迎邮件的逻辑
    }
}
```

我们定义一个 `UserRegisterSubject` 类来维护观察者列表，并在事件发生时通知观察者：

```csharp
public class UserRegisterSubject
{
    private readonly IList<IObserver> _observers = new List<IObserver>();

    public void Register(IObserver observer)
    {
        _observers.Add(observer);
    }

    public async Task NotifyObservers(UserRegisterEvent userRegisterEvent)
    {
        foreach (var observer in _observers)
        {
            await observer.Update(userRegisterEvent);
        }
    }
}
```

最后，我们在 `UserController` 中使用 `UserRegisterSubject`：

```csharp
public class UserController : Controller
{
    private readonly UserService _userService;
    private readonly UserRegisterSubject _userRegisterSubject;

    public UserController(UserService userService, UserRegisterSubject userRegisterSubject)
    {
        _userService = userService;
        _userRegisterSubject = userRegisterSubject;
    }

    public async Task<long> Register(string telephone, string password)
    {
        // 省略输入参数的校验代码
        // 省略 _userService.Register() 异常的 try-catch 代码
        long userId = await _userService.Register(telephone, password);

        // 创建事件并通知观察者
        var userRegisterEvent = new UserRegisterEvent(userId);
        await _userRegisterSubject.NotifyObservers(userRegisterEvent);

        return userId;
    }
}
```

这个例子中的 `UserService` 类是一个假设的服务，它的 `Register` 方法应该实现用户注册的逻辑。由于原始需求没有给出具体的实现，我在这里没有给出具体的代码。

在 ASP.NET Core 中，你可以使用依赖注入 (DI) 来管理 `UserService`、`UserRegisterSubject`、`PromotionService` 和 `MailService` 的实例。你可以在 `Startup` 类的 `ConfigureServices` 方法中添加如下代码：

```csharp
services.AddSingleton<UserService>();
services.AddSingleton<UserRegisterSubject>();
services.AddSingleton<IObserver, PromotionService>();
services.AddSingleton<IObserver, MailService>();
```

然后在

 `Configure` 方法中，获取 `UserRegisterSubject` 实例，并注册观察者：

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, UserRegisterSubject userRegisterSubject, IEnumerable<IObserver> observers)
{
    // 其他代码...

    foreach (var observer in observers)
    {
        userRegisterSubject.Register(observer);
    }
}
```

这样，每次有用户注册成功，就会自动通知 `PromotionService` 和 `MailService`。

上面的C#实现是异步非阻塞的实现方式。由于C#的`async/await`关键字，我们可以很容易地实现异步编程。

`async`关键字用于修饰方法，表示该方法是异步的，意味着如果在方法中有用到`await`，那么控制会立即返回到方法的调用者，然后当`await`后面的任务完成后再回来继续执行。

`await`关键字只能在`async`方法中使用，用于获取异步操作的结果，如果异步操作尚未完成，则`await`会暂时返回到调用方法的上下文中，等待异步操作完成。

所以在上述代码中：

```csharp
public async Task NotifyObservers(UserRegisterEvent userRegisterEvent)
{
    foreach (var observer in _observers)
    {
        await observer.Update(userRegisterEvent);
    }
}
```

`NotifyObservers`是一个异步方法，每个观察者的`Update`方法也是异步的，`await`关键字确保了`Update`方法完成后再执行下一个观察者的`Update`方法，但并没有阻塞主线程，因为当`Update`方法在执行时，控制权会立即返回给`NotifyObservers`方法的调用者。

这样一来，`UserController`的`Register`方法也是异步的，它会在所有观察者处理完成后返回。这种方式是非阻塞的，因为主线程并没有等待所有这些操作完成才继续执行。

```csharp
public async Task<long> Register(string telephone, string password)
{
    // 省略输入参数的校验代码
    // 省略 _userService.Register() 异常的 try-catch 代码
    long userId = await _userService.Register(telephone, password);

    // 创建事件并通知观察者
    var userRegisterEvent = new UserRegisterEvent(userId);
    await _userRegisterSubject.NotifyObservers(userRegisterEvent);

    return userId;
}
```

当`Register`方法被调用时，它不会阻塞等待用户注册成功和观察者完成处理，而是立即返回一个`Task<long>`，表示这个操作将在未来完成。在C#中，这是实现异步非阻塞编程的一种常见方式。

---

在C#中，将异步操作转换为同步操作，通常只需要将异步方法调用的`await`关键字去掉，并且将返回类型从`Task`或`Task<T>`更改为你需要的类型。下面是同步实现的示例：

首先是 `IObserver` 接口以及 `PromotionService` 和 `MailService` 的实现：

```csharp
public interface IObserver
{
    void Update(UserRegisterEvent userRegisterEvent);
}

public class PromotionService : IObserver
{
    public void Update(UserRegisterEvent userRegisterEvent)
    {
        IssueNewUserExperienceCash(userRegisterEvent.UserId);
    }

    private void IssueNewUserExperienceCash(long userId)
    {
        // 实现发放体验金的逻辑
    }
}

public class MailService : IObserver
{
    public void Update(UserRegisterEvent userRegisterEvent)
    {
        SendWelcomeMail(userRegisterEvent.UserId);
    }

    private void SendWelcomeMail(long userId)
    {
        // 实现发送欢迎邮件的逻辑
    }
}
```

然后是 `UserRegisterSubject` 类的实现：

```csharp
public class UserRegisterSubject
{
    private readonly IList<IObserver> _observers = new List<IObserver>();

    public void Register(IObserver observer)
    {
        _observers.Add(observer);
    }

    public void NotifyObservers(UserRegisterEvent userRegisterEvent)
    {
        foreach (var observer in _observers)
        {
            observer.Update(userRegisterEvent);
        }
    }
}
```

最后是 `UserController` 类的实现：

```csharp
public class UserController : Controller
{
    private readonly UserService _userService;
    private readonly UserRegisterSubject _userRegisterSubject;

    public UserController(UserService userService, UserRegisterSubject userRegisterSubject)
    {
        _userService = userService;
        _userRegisterSubject = userRegisterSubject;
    }

    public long Register(string telephone, string password)
    {
        // 省略输入参数的校验代码
        // 省略 _userService.Register() 异常的 try-catch 代码
        long userId = _userService.Register(telephone, password);

        // 创建事件并通知观察者
        var userRegisterEvent = new UserRegisterEvent(userId);
        _userRegisterSubject.NotifyObservers(userRegisterEvent);

        return userId;
    }
}
```

同样，这个例子中的 `UserService` 类是一个假设的服务，它的 `Register` 方法应该实现用户注册的逻辑。

---


如何实现一个跨进程的观察者模式？

跨进程的观察者模式实现通常会借助于消息队列（如 RabbitMQ、Kafka 等）或发布/订阅模型的中间件（如 Redis Pub/Sub、Google Pub/Sub 等）。当事件发生时，被观察者会将事件发布到消息队列或中间件，观察者则从消息队列或中间件订阅并接收事件。

以下是一个概念性的代码示例，使用了 RabbitMQ 作为消息队列。假设我们有一个名为 "UserRegistered" 的队列，当用户注册事件发生时，我们会将事件发布到这个队列：

```csharp
public class UserController : Controller
{
    private readonly UserService _userService;
    private readonly IModel _channel;

    public UserController(UserService userService, IConnection connection)
    {
        _userService = userService;
        _channel = connection.CreateModel();
        _channel.QueueDeclare(queue: "UserRegistered",
                             durable: false,
                             exclusive: false,
                             autoDelete: false,
                             arguments: null);
    }

    public long Register(string telephone, string password)
    {
        long userId = _userService.Register(telephone, password);

        var userRegisterEvent = new UserRegisterEvent(userId);
        string message = JsonConvert.SerializeObject(userRegisterEvent);
        var body = Encoding.UTF8.GetBytes(message);

        _channel.BasicPublish(exchange: "",
                              routingKey: "UserRegistered",
                              basicProperties: null,
                              body: body);

        return userId;
    }
}
```

然后，我们可以在 `PromotionService` 和 `MailService` 中订阅 "UserRegistered" 队列，并处理接收到的事件：

```csharp
public class PromotionService
{
    public PromotionService(IConnection connection)
    {
        var channel = connection.CreateModel();
        channel.QueueDeclare(queue: "UserRegistered",
                             durable: false,
                             exclusive: false,
                             autoDelete: false,
                             arguments: null);

        var consumer = new EventingBasicConsumer(channel);
        consumer.Received += (model, ea) =>
        {
            var body = ea.Body.ToArray();
            var message = Encoding.UTF8.GetString(body);
            var userRegisterEvent = JsonConvert.DeserializeObject<UserRegisterEvent>(message);

            IssueNewUserExperienceCash(userRegisterEvent.UserId);
        };
        channel.BasicConsume(queue: "UserRegistered",
                             autoAck: true,
                             consumer: consumer);
    }

    private void IssueNewUserExperienceCash(long userId)
    {
        // 实现发放体验金的逻辑
    }
}

public class MailService
{
    public MailService(IConnection connection)
    {
        var channel = connection.CreateModel();
        channel.QueueDeclare(queue: "UserRegistered",
                             durable: false,
                             exclusive: false,
                             autoDelete: false,
                             arguments: null);

        var consumer = new EventingBasicConsumer(channel);
        consumer.Received += (model, ea) =>
        {
            var body = ea.Body.ToArray();
            var message = Encoding.UTF8.GetString(body);
            var userRegisterEvent = JsonConvert.DeserializeObject<UserRegisterEvent>(message);

            SendWelcomeMail(userRegisterEvent.UserId);
        };
        channel.BasicConsume(queue: "UserRegistered",
                             autoAck: true,
                             consumer: consumer);
    }

    private void SendWelcomeMail(long userId)
    {
        // 实现发送欢迎邮件的逻辑
    }
}
```

需要注意的是，上述示例只是演示了使用 RabbitMQ 实现

跨进程观察者模式的基本概念。在实际的生产环境中，你可能需要考虑更多的问题，如消息的持久化、消费者的并发处理、错误处理、消息的重试、死信队列等。

---

如果我们的需求更加极端一点，需要在同步阻塞和异步非阻塞之间灵活切换，是否就要不停地修改 UserController 的代码。除此之外，如果在项目中，不止一个业务模块需要用到异步非阻塞观察者模式，如何使用可复用的代码实现。

chatgpt 方案

在这种情况下，一种有效的解决方案是创建一个独立的事件总线（Event Bus），该总线可以灵活地处理同步和异步的观察者。在这个事件总线中，我们可以将事件和其对应的处理器列表进行映射，并允许在运行时动态地注册事件和其处理器。

以下是使用 C# 实现一个简单的事件总线：

```csharp
public interface IEventHandler
{
    Task HandleAsync(object eventData);
    void Handle(object eventData);
}

public class EventBus
{
    private readonly Dictionary<Type, List<IEventHandler>> _handlers = new Dictionary<Type, List<IEventHandler>>();
    // private readonly ExecutorService _executorService = new ExecutorService();
    //ExecutorService类，这是在Java的java.util.concurrent包中的一个类。在.NET中执行异步操作，我们通常使用的是Task类。

    public void Register<TEventData>(IEventHandler eventHandler)
    {
        if (_handlers.ContainsKey(typeof(TEventData)))
        {
            _handlers[typeof(TEventData)].Add(eventHandler);
        }
        else
        {
            _handlers.Add(typeof(TEventData), new List<IEventHandler> { eventHandler });
        }
    }

    public void Trigger<TEventData>(TEventData eventData)
    {
        var eventType = typeof(TEventData);
        if (!_handlers.ContainsKey(eventType)) return;

        foreach (var eventHandler in _handlers[eventType])
        {
            eventHandler.Handle(eventData);
        }
    }

    // public void TriggerAsync<TEventData>(TEventData eventData)
    // {
    //     var eventType = typeof(TEventData);
    //     if (!_handlers.ContainsKey(eventType)) return;

    //     foreach (var eventHandler in _handlers[eventType])
    //     {
    //         _executorService.Execute(() => eventHandler.HandleAsync(eventData));
    //     }
    // }

    public void TriggerAsync<TEventData>(TEventData eventData)
    {
        var eventType = typeof(TEventData);
        if (!_handlers.ContainsKey(eventType)) return;

        foreach (var eventHandler in _handlers[eventType])
        {
            Task.Run(() => eventHandler.HandleAsync(eventData));
        }
    }
}

public class UserRegisterEvent
{
    public long UserId { get; set; }

    public UserRegisterEvent(long userId)
    {
        UserId = userId;
    }
}
```

这里我假设 `ExecutorService` 类是一个简单的线程池管理类，可以在新的线程中执行传递给 `Execute` 方法的函数。

然后，我们可以定义 `UserRegisterEventHandler` 类来处理 `UserRegisterEvent` 事件：

```csharp
public class UserRegisterEventHandler : IEventHandler
{
    public void Handle(object eventData)
    {
        // 同步处理 UserRegisterEvent 事件
    }

    public Task HandleAsync(object eventData)
    {
        // 异步处理 UserRegisterEvent 事件
    }
}

public class PromotionEventHandler : IEventHandler
{
    public void Handle(object eventData)
    {
        // 同步处理 PromotionEvent 事件
    }

    public Task HandleAsync(object eventData)
    {
        // 异步处理 PromotionEvent 事件
    }
}

public class MailEventHandler : IEventHandler
{
    public void Handle(object eventData)
    {
        // 同步处理 MailEvent 事件
    }

    public Task HandleAsync(object eventData)
    {
        // 异步处理 MailEvent 事件
    }
}

```

在使用时，我们只需要将需要处理的事件和对应的处理器注册到 `EventBus` 中，然后在需要的时候触发事件：

```csharp
public class UserController : Controller
{
    private readonly UserService _userService;
    private readonly EventBus _eventBus;

    public UserController(UserService userService, EventBus eventBus)
    {
        _userService = userService;
        _eventBus = eventBus;
    }

    public long Register(string telephone, string password)
    {
        // 省略输入参数的校验代码
        // 省略 _userService.Register() 异常的 try-catch 代码
        long userId = _userService.Register(telephone, password);

        // 创建事件并通知观察者
        var userRegisterEvent = new UserRegisterEvent(userId);
        _eventBus.Trigger(userRegisterEvent);

        return userId;
    }

    public async Task<long> RegisterAsync(string telephone, string password)
    {
        // 省略输入参数的校验代码
        // 省略 _userService.Register() 异常的 try-catch 代码
        long userId = await _userService.RegisterAsync(telephone, password);

        // 创建事件并通知观察者
        var

 userRegisterEvent = new UserRegisterEvent(userId);
        _eventBus.TriggerAsync(userRegisterEvent);

        return userId;
    }
}

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSingleton<UserService>();
        services.AddSingleton<EventBus>();
        services.AddSingleton<UserRegisterEventHandler>();
        services.AddSingleton<PromotionEventHandler>();
        services.AddSingleton<MailEventHandler>();
        // 其他服务...
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env, EventBus eventBus, UserRegisterEventHandler userRegisterEventHandler, PromotionEventHandler promotionEventHandler, MailEventHandler mailEventHandler)
    {
        // 注册事件和它们的处理器
        eventBus.Register<UserRegisterEvent>(userRegisterEventHandler);
        eventBus.Register<PromotionEvent>(promotionEventHandler);
        eventBus.Register<MailEvent>(mailEventHandler);
        // 其他配置...
    }
}

```

这样一来，不同的业务模块只需要定义自己的事件和处理器，然后在需要的时候注册和触发事件，而不需要修改 `UserController` 的代码。在需要从同步阻塞切换到异步非阻塞时，只需要改变触发事件的方法即可。


### 解释1

在上面的代码中，Event Bus是一个负责处理事件和其对应的处理器之间映射关系的类。这个设计模式通常被用在分布式系统中，以实现解耦和增强系统的可扩展性。下面我将详细解释这段代码。

Event Bus主要由两部分组成：

1. 一个用于存储事件类型和对应处理器列表的映射关系的字典 `_handlers`。这个字典的键是事件类型，值是一个包含所有处理这个事件类型的处理器的列表。

2. 一个用于在新线程中执行函数的 `ExecutorService` 实例 `_executorService`。这个类用于实现异步的事件处理。

Event Bus提供了以下几个方法：

- `Register<TEventData>(IEventHandler eventHandler)`：这个方法用于注册事件和对应的处理器。参数 `TEventData` 是事件的类型，`eventHandler` 是处理这个事件的处理器。这个方法首先检查是否已经有处理器被注册到这个事件类型，如果有，则添加新的处理器到处理器列表；如果没有，则创建一个新的处理器列表并添加到字典。

- `Trigger<TEventData>(TEventData eventData)`：这个方法用于触发一个事件。参数 `TEventData` 是事件的类型，`eventData` 是事件的数据。这个方法首先检查是否有处理器被注册到这个事件类型，如果没有，则直接返回；如果有，则遍历处理器列表，依次调用每个处理器的 `Handle` 方法。

- `TriggerAsync<TEventData>(TEventData eventData)`：这个方法用于异步触发一个事件。参数 `TEventData` 是事件的类型，`eventData` 是事件的数据。这个方法首先检查是否有处理器被注册到这个事件类型，如果没有，则直接返回；如果有，则遍历处理器列表，依次调用 `ExecutorService` 的 `Execute` 方法在新线程中执行每个处理器的 `HandleAsync` 方法。

通过这种方式，我们可以在需要的时候动态地注册事件和处理器，然后在需要的时候触发事件。这样可以实现在系统的不同部分之间的解耦，使得系统更加可扩展。

### 解释2

在.NET中，并没有直接的ExecutorService类，这是在Java的java.util.concurrent包中的一个类。在.NET中执行异步操作，我们通常使用的是Task类。抱歉给您带来的困扰。

首先，让我们回顾一下Event Bus是什么。Event Bus是一种软件架构模式，它用于解耦组件之间的通信。通过Event Bus，组件可以发布（发送）和订阅（接收）消息（事件）。发布者不需要知道谁订阅了它们的事件，同样，订阅者也不需要知道谁发布了事件。这样就可以减少组件之间的耦合。

下面是修改后的Event Bus的实现：

```csharp
public interface IEventHandler
{
    Task HandleAsync(object eventData);
    void Handle(object eventData);
}

public class EventBus
{
    private readonly Dictionary<Type, List<IEventHandler>> _handlers = new Dictionary<Type, List<IEventHandler>>();

    public void Register<TEventData>(IEventHandler eventHandler)
    {
        if (_handlers.ContainsKey(typeof(TEventData)))
        {
            _handlers[typeof(TEventData)].Add(eventHandler);
        }
        else
        {
            _handlers.Add(typeof(TEventData), new List<IEventHandler> { eventHandler });
        }
    }

    public void Trigger<TEventData>(TEventData eventData)
    {
        var eventType = typeof(TEventData);
        if (!_handlers.ContainsKey(eventType)) return;

        foreach (var eventHandler in _handlers[eventType])
        {
            eventHandler.Handle(eventData);
        }
    }

    public void TriggerAsync<TEventData>(TEventData eventData)
    {
        var eventType = typeof(TEventData);
        if (!_handlers.ContainsKey(eventType)) return;

        foreach (var eventHandler in _handlers[eventType])
        {
            Task.Run(() => eventHandler.HandleAsync(eventData));
        }
    }
}
```

在这个实现中，Event Bus包含一个字典（_handlers），它存储了事件类型（作为键）和处理这些事件的处理器列表（作为值）。每一个处理器都实现了`IEventHandler`接口，该接口定义了两个方法：`Handle`和`HandleAsync`，分别用于同步处理和异步处理事件。

`Register<TEventData>`方法用于注册一个事件处理器，用于处理类型为`TEventData`的事件。

`Trigger<TEventData>`方法用于触发一个事件，它将遍历处理这个类型事件的所有处理器，并调用它们的`Handle`方法。

`TriggerAsync<TEventData>`方法也用于触发一个事件，但是它将异步地调用处理器的`HandleAsync`方法。这是通过`Task.Run`实现的，它会在一个新的线程中执行指定的操作。这样就可以避免阻塞主线程，实现异步处理。

这就是修改后的Event Bus的实现。这个实现提供了一个灵活的方式来解耦应用程序的组件，使得它们可以异步地进行通信，而无需关心其他组件的实现。

---

C# 手把手实现一个 Google Guava EventBus 框架

这是一个 EventBus 的 C# 版本实现。在这个例子中，我会用到 .NET 5.0 的 `ConcurrentDictionary` 和 `ConcurrentBag` 以及 `System.Reflection` 命名空间下的反射API。

首先，我们需要定义一个特性来标记事件处理的方法：

```csharp
// 1.Subscribe
[AttributeUsage(AttributeTargets.Method)]
public class SubscribeAttribute : Attribute
{
}
```

接下来，定义一个表示被 `Subscribe` 特性标记的方法的类：

```csharp
// 2.ObserverAction
public class ObserverAction
{
    public object Target { get; }
    public MethodInfo Method { get; }

    public ObserverAction(object target, MethodInfo method)
    {
        Target = target ?? throw new ArgumentNullException(nameof(target));
        Method = method ?? throw new ArgumentNullException(nameof(method));
    }

    public void Execute(object eventToPost)
    {
        Method.Invoke(Target, new object[] { eventToPost });
    }
}
```

然后，创建一个表示注册表的类：

```csharp
// 3.ObserverRegistry
public class ObserverRegistry
{
    private readonly ConcurrentDictionary<Type, ConcurrentBag<ObserverAction>> registry = new ConcurrentDictionary<Type, ConcurrentBag<ObserverAction>>();

    public void Register(object observer)
    {
        var observerActions = FindAllObserverActions(observer);
        foreach (var action in observerActions)
        {
            if (registry.ContainsKey(action.Method.GetParameters()[0].ParameterType))
            {
                registry[action.Method.GetParameters()[0].ParameterType].Add(action);
            }
            else
            {
                registry[action.Method.GetParameters()[0].ParameterType] = new ConcurrentBag<ObserverAction> { action };
            }
        }
    }

    public IEnumerable<ObserverAction> GetMatchedObserverActions(object eventToPost)
    {
        return registry[eventToPost.GetType()].ToArray();
    }

    private static IEnumerable<ObserverAction> FindAllObserverActions(object observer)
    {
        var info = observer.GetType().GetMethods(BindingFlags.Public | BindingFlags.NonPublic | BindingFlags.Instance)
            .Where(m => m.GetCustomAttributes(typeof(SubscribeAttribute), false).Any())
            .Where(m => m.GetParameters().Length == 1)
            .Select(m => new ObserverAction(observer, m));
        return info;
    }
}
```

然后，创建表示 `EventBus` 和 `AsyncEventBus` 的类：

```csharp
// 4.EventBus
public class EventBus
{
    private readonly ObserverRegistry registry = new ObserverRegistry();

    public void Register(object observer)
    {
        registry.Register(observer);
    }

    public void Post(object eventToPost)
    {
        var observerActions = registry.GetMatchedObserverActions(eventToPost);
        foreach (var observerAction in observerActions)
        {
            observerAction.Execute(eventToPost);
        }
    }
}

// 5.AsyncEventBus
public class AsyncEventBus : EventBus
{
    private readonly ObserverRegistry registry = new ObserverRegistry();
    private readonly TaskFactory taskFactory = new TaskFactory(TaskScheduler.Default);

    public void PostAsync(object eventToPost)
    {
        var observerActions = registry.GetMatchedObserverActions(eventToPost);
        foreach (var observerAction in observerActions)
        {
            taskFactory.StartNew(() => observerAction.Execute(eventToPost));
        }
    }
}
```

这就是一个基础的 `EventBus` 框架实现，不过请注意，这只是一个基础实现，还有许多可以优化的地方，如异常处理、取消任务等。此外，反射操作会产生一定的性能开销，实际使用时需要谨慎考虑。

在上面的 C# EventBus 实现中，我们创建了一种观察者模式的框架，该框架允许多个观察者监听并响应某个事件。以下是该框架每个主要部分的详细解释：

1. `SubscribeAttribute`：这是一个自定义的特性（Attribute），用于标记那些需要作为事件处理器的方法。

2. `ObserverAction`：这个类表示一个事件处理器，它包含一个方法（由 `MethodInfo` 对象表示）和该方法所属的目标对象。`Execute` 方法是用来触发事件处理器的，它使用了 .NET 反射来调用目标方法。

3. `ObserverRegistry`：这是一个注册表，用于管理所有注册的事件处理器。每种类型的事件都有自己的处理器集合，它们在 ConcurrentDictionary 中被索引和存储。注册表提供了两个主要方法：`Register`（用于添加新的事件处理器）和 `GetMatchedObserverActions`（用于根据事件类型检索匹配的处理器）。

4. `EventBus`：这是框架的主要组成部分，它提供了注册事件处理器和发布事件的方法。在发布事件时，EventBus 会查找匹配的事件处理器并同步执行它们。

5. `AsyncEventBus`：这是 `EventBus` 的一个子类，它以异步方式运行事件处理器。它使用了 `TaskFactory` 来在不同的线程上运行处理器，这样可以避免阻塞主线程。

这个框架的主要优点是它提供了一种简洁的方式来实现观察者模式，通过注解和反射，可以非常方便地定义和注册事件处理器。然而，这个实现还有许多可以改进的地方，比如异常处理，取消任务等。此外，由于反射操作的开销，你可能需要考虑在性能关键的情况下使用其他技术。

现在我们已经有了 `EventBus`，在我们的 `UserController` 中使用起来就变得很简单。首先我们需要在我们的处理器中添加 `Subscribe` 标签，然后在我们的 `UserController` 中注册这些处理器并发布事件。

下面是一个示例：

```csharp
public class UserRegisterEventHandler
{
    [Subscribe]
    public void HandleEvent(UserRegisterEvent userRegisterEvent)
    {
        // do something with the event
    }
}

public class PromotionEventHandler
{
    [Subscribe]
    public void HandleEvent(UserRegisterEvent userRegisterEvent)
    {
        // do something with the event
    }
}

public class MailEventHandler
{
    [Subscribe]
    public void HandleEvent(UserRegisterEvent userRegisterEvent)
    {
        // do something with the event
    }
}

public class UserController
{
    private EventBus eventBus = new EventBus();

    public UserController()
    {
        eventBus.Register(new UserRegisterEventHandler());
        eventBus.Register(new PromotionEventHandler());
        eventBus.Register(new MailEventHandler());
    }

    public void Register(User user)
    {
        // Save the user to the database...

        UserRegisterEvent userRegisterEvent = new UserRegisterEvent(user);
        eventBus.Post(userRegisterEvent);
    }
}
```

在这个示例中，我们在 `UserController` 的构造函数中注册了我们的事件处理器，然后在 `Register` 方法中发布了 `UserRegisterEvent`。然后 `EventBus` 会调用注册的处理器中标记为 `Subscribe` 的方法。

这样，我们就可以在处理器中处理用户注册事件了，而 `UserController` 无需知道这些处理器的具体实现。当我们需要添加新的处理器时，只需要创建新的处理器并在 `UserController` 中注册即可，无需修改 `UserController` 的其他代码，这样符合开闭原则。

注意，由于这个示例使用的是同步的 `EventBus`，事件处理器的处理方法会在同一线程中按注册的顺序依次执行。如果你希望这些处理器能并行处理事件，可以使用 `AsyncEventBus` 并将 `Post` 方法更改为 `PostAsync`。
---

Q: 社交媒体的关注功能也属于订阅-发布模式的一种，一个明星用户可以有几万甚至几千万的关注者，一个明星用户发布的一条信息如何分发到所有关注者的消息队列中？这其中会遇到什么问题？应该如何解决？

社交媒体的关注功能的确可以看作是一种发布-订阅模式的实现。当一个用户（如一个明星）发布一条新的信息时，这条信息需要被分发到所有关注者的消息队列中。对于拥有大量关注者的用户来说，这可能是一个巨大的挑战。以下是可能遇到的问题，以及一些潜在的解决方案。

**问题1：资源消耗** 

处理大量的关注者需要大量的计算和网络资源。同时，这也可能需要大量的存储空间，以保存每个用户的消息队列。

**解决方案：** 

可以使用更有效的数据结构，例如Bloom Filter，来减少存储需求。此外，可以利用分布式系统和云计算资源来处理大规模的工作负载。

**问题2：性能瓶颈** 

如果所有的信息分发都在同一台服务器上进行，那么这台服务器可能会成为性能瓶颈。

**解决方案：**

通过使用负载均衡和分布式系统，将工作负载分散到多台服务器上，可以避免单点性能瓶颈的问题。

**问题3：数据一致性** 

如果信息分发是异步进行的，那么可能会出现数据一致性问题。例如，一些用户可能会比其他用户更早地看到新的信息。

**解决方案：** 

可以使用各种数据一致性协议和算法（例如，Raft或Paxos）来保证数据一致性。然而，这可能会对性能产生影响，需要在一致性和性能之间进行权衡。

**问题4：信息流的个性化**

不是所有关注者都对所有信息感兴趣。如果信息量很大，用户可能会错过他们真正关心的信息。

**解决方案：** 

通过使用机器学习算法，可以根据每个用户的行为和喜好对信息进行个性化排序和过滤。

总的来说，实现社交媒体的关注功能需要解决许多挑战，包括处理大规模数据、保证高可用性和性能、解决数据一致性问题，以及提供个性化的用户体验。这通常需要结合多种技术和方法来实现，例如分布式系统、负载均衡、数据一致性协议、数据库优化、缓存系统，以及机器学习等。

Twitter在处理用户之间的关系和消息传递上已经经历过几个重要的架构改进阶段。在早期，Twitter的架构是基于Ruby on Rails的单体应用，后来转向了更加分布式和服务化的架构。

在2011年，为了解决因用户增长带来的规模扩展问题，Twitter引入了一种名为“雪花”的服务，用于生成全局唯一ID，使得分布式环境下可以有序、无冲突地生成消息ID。

而在处理“关注”这个特性时，Twitter在2013年时引入了一个新的系统叫做“Timelines”。这个服务的目标是生成和服务所有用户的主页时间线，也就是用户登录后看到的那个页面。

在这个服务中，Twitter采用了一种被称为“Fanout”的策略。当一个用户发布了一条新的推文，Twitter会立即将这条推文分发到所有关注该用户的人的主页时间线中。这种策略在用户量不大时非常有效，但随着用户量的增长，Twitter也引入了更复杂的策略。例如，对于拥有大量粉丝的用户（如名人），Twitter会延迟分发他们的推文，而不是立即进行。

为了保持时间线的新鲜度，Twitter会定期清理时间线中的过期推文，并且引入了缓存机制，使用了Memcached和Redis等技术来提高读取性能。

另外，Twitter还引入了机器学习算法，通过分析用户的行为和偏好，对时间线中的推文进行排序和个性化推荐，以提高用户体验。

总的来说，Twitter在处理用户关系和消息传递上，采用了包括分布式系统、全局唯一ID生成器、Fanout策略、延迟分发、缓存技术和机器学习等一系列技术和策略。但是，也需要注意，这些信息基于我在2021年9月前的知识，具体的技术细节和策略可能已经发生了变化。

Q: 对比“生产者-消费者”模式和观察者模式的区别和联系

chatgpt 回答：
"生产者 - 消费者"模型和观察者模式都是设计模式中常见的模型，它们都可以处理一对多的关系，但是它们的应用场景和设计目标是不同的。

**生产者 - 消费者模型：**

生产者-消费者模型主要关注在不同组件或系统间进行数据交换，尤其是在异步系统中。在这个模型中，生产者和消费者通常不直接进行交互，而是通过共享数据结构（如队列）或者通过中间件（如 Kafka、RabbitMQ）进行数据的传递。生产者生成数据并发送到共享数据结构或中间件，消费者从中取数据并进行处理。

**观察者模式：**

观察者模式主要关注在一个对象（被观察者）状态改变时，如何自动通知和更新其他对象（观察者）。被观察者会持有所有观察者的引用，当被观察者状态改变时，会自动调用观察者的更新方法。观察者模式常用于实现事件驱动的系统，比如 GUI 系统中，当用户进行点击操作（被观察者状态改变）时，系统需要自动更新显示内容（观察者进行操作）。

**联系和区别：**

两者的主要联系是，都能处理一对多的关系，即一个生产者可以有多个消费者，一个被观察者可以有多个观察者。

区别主要在于以下几点：

1. **通信方式：** 生产者-消费者模型通常通过队列、栈或者其他数据结构进行通信，或者通过某种中间件进行通信，因此它们之间是解耦的。观察者模式则是通过直接调用方法进行通信，因此它们之间是耦合的。

2. **同步/异步：** 生产者-消费者模型是异步的，生产者生产数据后不必等待消费者处理完毕。观察者模式可以是同步的，也可以是异步的。在同步的观察者模式中，被观察者在通知所有观察者后才能继续执行。

3. **使用场景：** 生产者-消费者模型主要用于处理异步任务，或者在不同组件或系统间进行数据传递。观察者模式主要用于实现事件驱动系统，或者在一个对象状态改变时自动更新其他对象。

4. **数据处理：** 在生产者-消费者模型中，生产者和消费者通常对数据进行不同的处理。在观察者模式中，所有观察者对同一个事件可能进行相同或者不同

## Design Patterns 笔记

## .net 框架