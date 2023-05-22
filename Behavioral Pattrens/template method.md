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