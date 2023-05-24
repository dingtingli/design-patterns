# 模板模式（Template Method Design Pattern）

## 极客时间 笔记

>模板方法模式在一个方法中定义一个算法骨架，并将某些步骤推迟到子类中实现。模板方法模式可以让子类在不改变算法整体结构的情况下，重新定义算法中的某些步骤。

这里的“算法”，我们可以理解为广义上的“业务逻辑”，并不特指数据结构和算法中的“算法”。这里的算法骨架就是“模板”，包含算法骨架的方法就是“模板方法”，这也是模板方法模式名字的由来。

```c#
public abstract class AbstractClass 
{
    public void TemplateMethod() 
    {
        //...
        Method1();
        //...
        Method2();
        //...
    }
  
    protected abstract void Method1();
    protected abstract void Method2();
}

public class ConcreteClass1 : AbstractClass 
{
    protected override void Method1() 
    {
        //...
    }
  
    protected override void Method2() 
    {
        //...
    }
}

public class ConcreteClass2 : AbstractClass 
{
    protected override void Method1() 
    {
        //...
    }
  
    protected override void Method2() 
    {
        //...
    }
}

// 使用示例
AbstractClass demo = new ConcreteClass1();
demo.TemplateMethod();
```

### asp.net mvc action

在ASP.NET MVC中，Controller类扮演着中心角色，每一个Controller中的方法（称为Action）通常对应着一个Web请求。我们可以将每个Action视为一个模板方法，因为ASP.NET MVC的框架为每一个Action提供了一个执行的流程（或者说模板）。

这个流程包括了诸如身份验证、模型绑定、请求数据验证、ActionResult执行等一系列的步骤。ASP.NET MVC框架在执行Action之前和之后，都会调用一系列的虚方法（诸如OnActionExecuting，OnActionExecuted等），这些虚方法可以被子类重写以插入特定的行为。

以下是一个简化的示例代码：

```c#
public class BaseController : Controller
{
    protected override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        //在执行Action之前的代码，比如一些权限验证、数据初始化等
        base.OnActionExecuting(filterContext);
    }

    protected override void OnActionExecuted(ActionExecutedContext filterContext)
    {
        //在执行Action之后的代码，比如清理资源、记录日志等
        base.OnActionExecuted(filterContext);
    }
}

public class MyController : BaseController
{
    public ActionResult MyAction()
    {
        // 这是一个Action，它的执行流程已经在BaseController中被定义了
        // 开发者只需要关注这里的业务逻辑即可
        // ...
        return View();
    }
}

```

在这个例子中，BaseController类中的OnActionExecuting和OnActionExecuted方法可以被视为模板方法模式中的"钩子"，这些方法默认不做任何事情，但是子类可以重写它们以插入特定的行为。在MyController中，开发者只需要关注MyAction的业务逻辑，至于MyAction在执行之前和之后需要进行哪些操作，完全由BaseController决定。这就是模板方法模式的优势：子类可以在不改变操作流程的前提下，定制操作的某些步骤。

在ASP.NET MVC中，对于Controller的Action的执行流程的控制是由框架内部处理的，并不直接在我们自定义的Controller类中体现。换句话说，ASP.NET MVC框架定义了一种隐式的模板方法，通过在关键的执行步骤调用Controller类的虚方法，例如OnActionExecuting、OnActionExecuted等。这些方法在默认情况下可能不做任何事情，但它们提供了扩展点，让开发者可以插入自定义的行为。

如果我们要直观地表示这种模板方法，可能会是这样的：

```c#
public void ExecuteAction(Action action)
{
    OnActionExecuting();   // 这是一个虚方法，可以被子类重写
    action();             // 这里调用具体的Action方法，例如MyAction
    OnActionExecuted();   // 这也是一个虚方法，可以被子类重写
}
```

在实际的ASP.NET MVC框架中，这种模板方法的执行流程是由框架内部的路由和控制器处理机制实现的。当一个HTTP请求到达ASP.NET MVC应用时，框架会根据路由配置找到相应的Controller和Action，然后按照上述的流程去执行Action。在这个过程中，框架会在适当的时机调用OnActionExecuting、OnActionExecuted等方法，开发者可以通过重写这些方法来自定义Action的执行流程。

所以，虽然在我们自定义的Controller类中看不到显式的模板方法，但ASP.NET MVC的设计理念和模板方法模式的思想是一致的：定义一个操作的主体流程，然后提供一些扩展点让子类插入自定义的行为。


ASP.NET MVC框架的核心部分之一是它的路由和控制器处理机制。当一个HTTP请求到达ASP.NET MVC应用时，框架需要确定如何处理这个请求。这个决定的过程由多个步骤组成，形成了一种隐式的模板方法。虽然这个模板方法并不直接体现在我们编写的代码中，但它是ASP.NET MVC框架的核心部分。以下是这个模板方法的主要步骤：

1. 路由：当一个HTTP请求到达应用时，ASP.NET MVC的路由系统会根据请求的URL确定应该调用哪个Controller的哪个Action。这个决定过程是通过比较请求的URL和预先定义的路由规则进行的。如果找到了匹配的路由规则，路由系统会创建一个包含请求信息的RouteData对象。

2. 控制器选择：路由系统会将RouteData对象传递给ControllerFactory，ControllerFactory根据RouteData中的信息创建一个Controller对象。ControllerFactory首先调用Controller的构造函数创建Controller对象，然后调用Controller的Initialize方法初始化Controller。在Initialize方法中，Controller会保存RouteData对象，并设置一些其他的状态信息。

3. Action选择：接下来，Controller需要确定应该执行哪个Action。这个决定过程由ActionInvoker负责。ActionInvoker会查看RouteData中的信息，以及Controller的Action方法的签名，然后选择一个匹配的Action方法。

4. Action执行：在Action方法被执行前，会先调用Controller的OnActionExecuting方法。然后执行Action方法，执行完后，会调用Controller的OnActionExecuted方法。

以上就是ASP.NET MVC处理HTTP请求的主要步骤。可以看到，虽然我们在编写Controller和Action代码时，通常不需要考虑这些步骤，但它们构成了ASP.NET MVC框架的核心，也体现了模板方法模式的思想。这种设计方式使得开发者可以在不改变处理流程的前提下，通过重写Controller的方法，如OnActionExecuting、OnActionExecuted等来定制HTTP请求的处理过程。

好的，我会尽量简化一些细节，将这个过程以代码的形式表现出来。实际上，ASP.NET MVC框架的实现涉及到很多内部的类和机制，我这里仅仅用一些简化的代码来描绘这个过程的大概样子：

```c#
public class MvcApplication
{
    // 模拟路由配置
    Dictionary<string, Type> routeTable = new Dictionary<string, Type> {
        { "/mycontroller/myaction", typeof(MyController) }
    };

    public void ProcessRequest(string url)
    {
        // 根据路由规则，决定哪个Controller应该处理请求
        var controllerType = routeTable[url];
        // 创建Controller对象
        var controller = Activator.CreateInstance(controllerType) as Controller;
        // 初始化Controller
        controller.Initialize();

        // ActionInvoker决定执行哪个Action
        var actionInvoker = new ActionInvoker();
        var action = actionInvoker.GetAction(controller, url);

        // 执行Action
        controller.OnActionExecuting();
        action();
        controller.OnActionExecuted();
    }
}

public abstract class Controller
{
    public virtual void Initialize()
    {
        //...
    }

    public virtual void OnActionExecuting()
    {
        //...
    }

    public virtual void OnActionExecuted()
    {
        //...
    }
}

public class MyController : Controller
{
    public override void OnActionExecuting()
    {
        // 在Action执行前，可以插入一些自定义的行为
        base.OnActionExecuting();
    }

    public override void OnActionExecuted()
    {
        // 在Action执行后，可以插入一些自定义的行为
        base.OnActionExecuted();
    }

    public void MyAction()
    {
        // 这是一个Action，它的具体逻辑由开发者定义
    }
}

public class ActionInvoker
{
    public Action GetAction(Controller controller, string url)
    {
        // 在真实的ASP.NET MVC框架中，ActionInvoker会通过一些复杂的规则来决定执行哪个Action
        // 这里为了简化，我们假设它总是返回MyAction
        return ((MyController)controller).MyAction;
    }
}

```

在这个例子中，我们可以看到ASP.NET MVC处理HTTP请求的一般过程：

1. 当一个请求到达时，MvcApplication会根据URL找到应该处理这个请求的Controller类型。
2. 然后，MvcApplication创建这个Controller的实例，并调用其Initialize方法进行初始化。
3. 接着，MvcApplication使用ActionInvoker决定应该执行哪个Action，然后调用Controller的OnActionExecuting方法、执行Action、调用Controller的OnActionExecuted方法。

虽然这个例子极度简化了ASP.NET MVC框架的实现，并没有涉及到很多重要的细节，如模型绑定、ActionResult、过滤器等，但它大致描绘了ASP.NET MVC处理HTTP请求的流程，以及模板方法模式在这个流程中的应用。

## 回调函数

通过代码示例来解释模板方法模式和回调函数的相似之处和区别。

模板方法模式：
我们先来看一个C#中使用模板方法模式的例子：

```c#
public abstract class AbstractClass
{
    // This is the 'template method'
    public void TemplateMethod()
    {
        Step1();
        Step2();
    }

    public abstract void Step1();
    public abstract void Step2();
}

public class ConcreteClass : AbstractClass
{
    public override void Step1()
    {
        Console.WriteLine("ConcreteClass Step1");
    }

    public override void Step2()
    {
        Console.WriteLine("ConcreteClass Step2");
    }
}
```

// Usage:
AbstractClass demo = new ConcreteClass();
demo.TemplateMethod();
在这个例子中，TemplateMethod 是模板方法，它定义了一个算法的骨架。这个算法的一些步骤（Step1 和 Step2）被延迟到子类 ConcreteClass 中实现。

回调函数：
接下来，我们看一个C#中使用回调函数的例子：

```c#
public delegate void Callback();

public class Caller
{
    public void Call(Callback callback)
    {
        Console.WriteLine("Before callback");
        callback();
        Console.WriteLine("After callback");
    }
}

// Usage:
Caller caller = new Caller();
Callback callback = () => Console.WriteLine("Inside callback");
caller.Call(callback);
```


在这个例子中，Caller 类的 Call 方法接收一个 Callback 委托作为参数。Call 方法在调用回调函数前后执行一些操作。通过传入不同的回调函数，我们可以改变 Call 方法中的行为，而不需要修改 Caller 类的代码。

如你所见，这两种策略都允许我们在不改变不可变部分代码的情况下修改可变部分的代码。不过模板方法模式是通过继承和方法重写实现的，而回调函数是通过传递函数实现的。

## 设计模式笔记

**模板方法** 类行为模式

**意图**

在一个操作中定义算法的框架，将一些步骤推迟到子类中。模板方法让子类可以重新定义算法的某些步骤，而不改变算法的结构。

**动机**

考虑一个应用程序框架，它提供了 Application 和 Document 类。Application 类负责打开存储在外部格式中的现有文档，如文件。Document 对象代表从文件中读取的文档的信息。

使用这个框架构建的应用程序可以对 Application 和 Document 进行子类化，以满足特定需求。例如，一个绘图应用程序定义了 DrawApplication 和 DrawDocument 子类；一个电子表格应用程序定义了 SpreadsheetApplication 和 SpreadsheetDocument 子类。

抽象的 Application 类在其 OpenDocument 操作中定义了打开和读取文档的算法：

```csharp
void OpenDocument(string name) {
    if (!CanOpenDocument(name)) {
        // 无法处理此文档
        return;
    }

    Document doc = DoCreateDocument();
    if (doc != null) {
        _docs.Add(doc);
        AboutToOpenDocument(doc);
        doc.Open();
        doc.DoRead();
    }
}
```

OpenDocument 定义了打开文档的每个步骤。它检查文档是否可以被打开，创建应用程序特定的 Document 对象，将它添加到它的文档集合中，并从文件中读取 Document。

我们称 OpenDocument 为模板方法。一个模板方法以子类覆盖提供具体行为的抽象操作的形式定义一个算法。Application 子类定义了检查文档是否可以被打开（CanOpenDocument）和创建 Document（DoCreateDocument）的算法步骤。Document 类定义了读取文档（DoRead）的步骤。模板方法还定义了一个让 Application 子类知道文档即将被打开（AboutToOpenDocument）的操作，以防他们关心。

通过使用抽象操作定义算法的一些步骤，模板方法固定了它们的排序，但允许 Application 和 Document 子类根据需要变化这些步骤。

**适用性**

当需要实现算法的不变部分一次，并将可变的行为留给子类实现时，应使用模板方法模式。

当子类之间的公共行为应被提取并在一个公共类中本地化以避免代码重复时。这是 Opdyke 和 Johnson [OJ93]描述的"重构以泛化"的一个好例子。首先，你识别现有代码中的差异，然后将这些差异分解成新的操作。最后，用调用这些新操作的模板方法替换不同的代码。

当需要控制子类的扩展时。你可以定义一个模板方法，该方法在特定点调用" hook "操作（见后果），从而只允许在这些点进行扩展。

**结构**

**参与者**
- **抽象类（Application）**：定义抽象原语操作，这些操作由具体子类定义以实现算法的步骤。它实现了一个定义算法骨架的模板方法。模板方法调用原语操作以及在AbstractClass中定义的操作或其他对象的操作。
- **具体类（MyApplication）**：实现原语操作以执行子类特定的算法步骤。

**合作**
- 具体类依赖于抽象类来实现算法的不变步骤。

**后果**

模板方法是代码重用的基本技术。它们在类库中尤其重要，因为它们是在库类中提取出公共行为的手段。

模板方法导致了一个被称为"Hollywood Principle"的倒置控制结构，即"Don't call us, we'll call you" [Swe85]。这是指父类调用子类的操作，而不是反过来。

模板方法调用以下几种操作：
- 具体操作（在具体类或客户端类上）；
- 具体的AbstractClass操作（即，对子类通常有用的操作）；
- 原语操作（即，抽象操作）；
- 工厂方法（见工厂方法（107））；
- 钩子操作，它们提供了子类可以在必要时扩展的默认行为。默认情况下，钩子操作通常什么都不做。

重要的是，模板方法要指定哪些操作是钩子（可以被覆盖）和哪些操作是抽象操作（必须被覆盖）。为了有效地重用抽象类，子类的作者必须理解哪些操作是设计用于覆盖的。

一个子类可以通过覆盖操作并显式调用父操作来扩展父类操作的行为：

```csharp
void Operation() {
    base.Operation();
    // DerivedClass 扩展行为
}
```

遗憾的是，很容易忘记调用继承的操作。我们可以将这样的操作转换成一个模板方法，让父类控制子类如何扩展它。这个想法是在父类的一个模板方法中调用一个钩子操作。然后子类可以覆盖这个钩子操作：

```csharp
void Operation() {
    // 父类行为
    HookOperation();
}

void HookOperation() { }
```

子类覆盖HookOperation以扩展其行为：

```csharp
void HookOperation() {
    // 衍生类扩展
}
```

**实现**

以下三个实现问题值得注意：
1. 使用 C# 访问控制。在 C# 中，模板方法调用的原语操作可以被声明为 protected 成员。这确保它
们只被模板方法调用。必须被覆盖的原语操作被声明为抽象。模板方法本身不应该被覆盖，因此你可以将模板方法声明为非虚成员函数。
2. 最小化原语操作。设计模板方法的一个重要目标是最小化子类必须覆盖以实现算法的原语操作的数量。需要覆盖的操作越多，对客户来说就越麻烦。
3. 命名规则。你可以通过在它们的名字前面添加一个前缀来识别应该被覆盖的操作。例如，MacApp 框架为 Macintosh 应用程序 [App89] 在模板方法名称前添加了"Do-"前缀："DoCreateDocument"，"DoRead"等等。

**样例代码**

以下 C# 示例展示了一个父类如何为其子类实施一个不变式。考虑一个支持在屏幕上绘图的 View 类。View 强制要求其子类只有在成为"焦点"后才能在视图中绘图，这需要适当地设置某些绘图状态（例如，颜色和字体）。

我们可以使用一个 Display 模板方法来设置这种状态。View 定义了两个具体的操作，SetFocus 和 ResetFocus，分别设置和清理绘图状态。View 的 DoDisplay 钩子操作执行实际的绘图。Display 在 DoDisplay 之前调用 SetFocus 来设置绘图状态；在之后调用 ResetFocus 来释放绘图状态。

```csharp
void Display() {
    SetFocus();
    DoDisplay();
    ResetFocus();
}
```

为了保持不变式，视图的客户端总是调用 Display，而 View 的子类总是覆盖 DoDisplay。

在 View 中，DoDisplay 什么也不做：

```csharp
void DoDisplay() { }
```

子类覆盖它以添加它们特定的绘图行为：

```csharp
void DoDisplay() {
    // 渲染视图的内容
}
```

**已知的应用**

模板方法非常基础，以至于它们可以在几乎每一个抽象类中找到。Wirfs-Brock等人 [WBWW90, WBJ90] 提供了模板方法的一个很好的概述和讨论。

**相关模式**

Factory Methods (107) 通常被模板方法调用。在动机示例中，工厂方法 DoCreateDocument 被模板方法 OpenDocument 调用。

策略 (315)：模板方法使用继承来变化算法的一部分。策略使用委托来变化整个算法。

---

"Hollywood Principle"（好莱坞原则）是一种软件设计原则，也是一种倒置控制的表现。在这种设计中，父类（或更高级别的模块）负责决定何时以及如何调用子类（或更低级别的模块）的方法，这就形成了 "Don't call us, we'll call you" 的交互模式。这是一种防止“高度耦合”的设计，提高了代码的模块化和可重用性。

模板方法模式在其实现中体现了好莱坞原则。在这种模式中，一个算法的结构定义在一个抽象的父类方法中（模板方法），而算法的某些步骤则留给子类来实现。这就是所谓的倒置控制，因为它倒置了通常的控制流：在常规的面向对象编程中，客户端代码通常会显式调用被调用对象的方法，但在模板方法模式中，父类会在适当的时机调用子类的方法。换句话说，不是子类告诉父类何时做什么，而是父类告诉子类何时做什么。

这样的设计可以带来很多优点，例如代码复用，降低了类间的耦合度，并且提高了代码的扩展性。同时，它也限制了子类可能对整体算法产生的影响，因为整个算法的结构是由父类来控制的。