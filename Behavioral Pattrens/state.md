
## 极客时间 笔记


有限状态机 Finite State Machine

缩写为 FSM，简称为状态机

状态机有 3 个组成部分：状态（State）、事件（Event）、动作（Action）其中，事件也称为转移条件（Transition Condition）。事件触发状态的转移及动作的执行。不过，动作不是必须的，也可能只转移状态，不执行任何动作。

## Design Patterns 笔记

状态模式（State Pattern）：行为型模式
### 意图：
允许对象在其内部状态改变时改变其行为。对象将表现得像是改变了类。
###  也被称为：
状态的对象
### 动机：
考虑一个表示网络连接的类 TCPConnection。TCPConnection 对象可以处于几种不同的状态：已建立（Established），正在监听（Listening），已关闭（Closed）。当 TCPConnection 对象从其他对象接收请求时，它根据当前状态的不同做出不同的响应。

例如， Open 请求的效果取决于连接是处于 Closed 状态还是 Established 状态。状态模式描述了 TCPConnection 如何在每种状态下表现出不同的行为。

这个模式的关键思想是引入一个名为 TCPState 的抽象类来表示网络连接的状态。TCPState 类声明了一个对所有代表不同操作状态的类都通用的接口。TCPState 的子类实现特定于状态的行为。

例如，类 TCPEstablished 和 TCPClosed 实现了特定于 TCPConnection 的 Established 和 Closed 状态的行为。

TCPConnection 类维护一个表示 TCP 连接当前状态的状态对象（TCPState子类的实例）。Connection 类将所有特定于状态的请求委托给这个状态对象。TCPConnection 使用其 TCPState 子类实例执行特定于连接状态的操作。

每当连接状态发生变化，TCPConnection 对象就会更换它使用的状态对象。例如，当连接从 Established 状态转为 Closed 状态时，TCPConnection 将会用一个 TCPClosed 实例替换其 TCPEstablished 实例。

### 适用性：

在以下任何一种情况下，都可以使用状态模式：

• 对象的行为取决于其状态，并且它必须根据这种状态在运行时改变其行为。

• 操作具有大型的、多部分的条件语句，这些语句取决于对象的状态。这个状态通常由一个或多个枚举常量表示。通常，有几个操作会包含这种相同的条件结构。状态模式将条件的每个分支放在一个单独的类中。这让你可以把对象的状态作为一个可以独立于其他对象变化的对象对待。

### 结构：

### 参与者
• 上下文（TCPConnection）
  - 定义对客户端感兴趣的接口。
  - 维护一个定义当前状态的 ConcreteState 子类的实例。

• 状态（TCPState）
  - 定义一个封装与 Context 的特定状态相关行为的接口。

• ConcreteState 子类（TCPEstablished，TCPListen，TCPClosed）
  - 每个子类实现与 Context 的状态相关的行为。

### 合作
• Context 将特定于状态的请求委托给当前的 ConcreteState 对象。

• 上下文可能将自身作为参数传递给处理请求的状态对象。这使得状态对象在必要时可以访问上下文。

• Context 是客户的主要接口。客户可以使用状态对象来配置上下文。一旦上下文配置完毕，它的客户就不必直接处理状态对象了。 

• Context 或 ConcreteState 子类可以决定哪个状态接替另一个状态以及在何种情况下。

### 结果：

状态模式有以下结果：
1. 它将特定于状态的行为本地化，并为不同状态划分行为。状态模式将所有与特定状态相关的行为都放入一个对象。因为所有特定于状态的代码都存在于一个状态子类中，所以可以通过定义新的子类轻松添加新的状态和转换。

    替代方法是使用数据值定义内部状态，并让 Context 操作显式检查数据。但那样我们就会在 Context 的实现中散布看起来相似的条件或 case 语句。添加一个新状态可能需要更改几个操作，这会使维护变得复杂。

    状态模式避免了这个问题，但可能引入另一个问题，因为该模式将不同状态的行为分布在几个状态子类中。这增加了类的数量，不如一个单一的类紧凑。但如果有许多状态，这种分布实际上是好的，否则就需要大型的条件语句。

    就像长过程一样，大型条件语句是不可取的。它们是单块的，往往使代码变得不够明确，这反过来使它们难以修改和扩展。状态模式提供了一种更好的方式来结构特定于状态的代码。确定状态转换的逻辑并不驻留在单块的if或switch语句中，而是在状态子类之间进行划分。将每个状态转换和行为封装在一个类中，提升了执行状态到完全对象状态的概念。这为代码增加了结构，并使其意图更清晰。

2. 它使状态转换显性化。当一个对象仅仅通过内部数据值定义其当前状态时，其状态转换没有显性的表示；它们只会作为一些变量的赋值出现。引入不同状态的不同对象使得转换更显性化。此外，状态对象可以保护上下文免受不一致的内部状态，因为从上下文的角度看，状态转换是原子的——它们通过重新绑定一个变量（上下文的状态对象变量），而不是多个变量来发生。

3. 状态对象可以被共享。如果状态对象没有实例变量——也就是说，它们代表的状态完全在其类型中编码——那么上下文可以共享一个状态对象。当状态以这种方式共享时，它们基本上是没有内在状态的轻量级对象（请参阅轻量级（195）），只有行为。

### 实现
状态模式提出了一系列的实现问题：

1. 谁定义状态转换？状态模式没有规定哪个参与者定义状态转换的标准。如果标准是固定的，那么可以完全在上下文中实现。然而，一般来说，让状态子类自己指定后继状态和何时进行转换更为灵活和合适。这需要向上下文添加一个接口，让状态对象明确设置上下文的当前状态。
这种将转换逻辑分散的方式使得通过定义新的状态子类来修改或扩展逻辑变得简单。分散化的一个缺点是，一个状态子类必须知道至少一个其他状态子类，这引入了子类之间的实现依赖。

2. 表格驱动的替代方案。在 C++编程风格 [Car921]中，Cargill 描述了另一种给状态驱动代码施加结构的方式：他使用表格来映射输入到状态转换。对于每个状态，一个表格映射每一个可能的输入到一个后续的状态。从本质上来说，这种方法将条件代码（以及在状态模式的情况下的虚函数）转换为表格查找。
表格的主要优点是它们的规律性：你可以通过修改数据而不是改变程序代码来改变转换标准。但也有一些缺点：
   • 表格查找通常比（虚）函数调用效率低。
   • 将转换逻辑放入统一的、表格化的格式使得转换标准变得不那么明显，因此更难理解。
   • 通常很难添加伴随状态转换的操作。
表格驱动的方法捕获了状态和它们的转换，但必须增加在每个转换上执行任意计算的能力。表格驱动的状态机和状态模式之间的关键区别可以这样总结：状态模式模拟状态特定的行为，而表格驱动的方法侧重于定义状态转换。

3. 创建和销毁状态对象。一个值得考虑的常见实现权衡是，（1）只在需要时创建状态对象，然后销毁它们，还是（2）提前创建它们，并永远不销毁它们。
当在运行时无法知道将要进入哪个状态，并且上下文不经常改变状态时，第一种选择更为可取。这种方法避免了创建不会使用的对象，如果状态对象存储了大量的信息，这是非常重要的。当状态变化迅速时，第二种方法更好，在这种情况下，你可能希望避免销毁状态，因为它们可能很快就会再次需要。实例化的成本只在前面一次付清，完全没有销毁的成本。然而，这种方法可能很不方便，因为上下文必须保留对可能进入的所有状态的引用。

4. 使用动态继承。可以通过在运行时改变对象的类来改变对特定请求的行为，但这在大多数面向对象的编程语言中是不可能的。例外包括 Self [US87]和其他提供此类机制的基于委托的语言，它们直接支持状态模式。在 Self 中，对象可以将操作委托给其他对象，以实现一种动态继承的形式。在运行时改变委托目标有效地改变了继承结构。这种机制让对象能够改变它们的行为，相当于改变它们的类。

### 示例代码
以下示例给出了在动机部分描述的 TCP 连接示例的 C++ 代码。这个示例是 TCP 协议的简化版本，它并没有描述完整的协议或所有的 TCP 连接状态。

首先，我们定义了 TCPConnection 类，它提供了一个传输数据的接口，并处理改变状态的请求。

```cpp
class TCPOctetStream;
class TCPState;
class TCPConnection {
public:
TCPConnection();
void ActiveOpen();
void PassiveOpen();
void Close();
void Send();
void Acknowledge();
void Synchronize();
void ProcessOctet(TCPOctetStream*);
private:
friend class TCPState;
void ChangeState(TCPState*);
private:
TCPState* _state;
};
```

TCPConnection 在 _state 成员变量中保留一个 TCPState 类的实例。TCPState 类复制了 TCPConnection 的状态改变接口。每个 TCPState 操作都带有一个 TCPConnection 实例作为参数，让 TCPState 从 TCPConnection 中访问数据，并改变连接的状态。

```cpp
class TCPState {
public:
virtual void Transmit(TCPConnection*, TCPOctetStream*);
virtual void ActiveOpen(TCPConnection*);
virtual void PassiveOpen(TCPConnection*);
virtual void Close(TCPConnection*);
virtual void Synchronize(TCPConnection*);
virtual void Acknowledge(TCPConnection*);
virtual void Send(TCPConnection*);
protected:
void ChangeState(TCPConnection*, TCPState*);
};
```

TCPConnection 将所有状态特定的请求委托给它的 TCPState 实例 _state。TCPConnection 还提供了一个将这个变量更改为新的 TCPState 的操作。TCPConnect ion 的构造函数初始化对象为 TCPClosed 状态（稍后定义）。

```cpp
TCPConnection::TCPConnection () {
_state = TCPClosed::Instance();
}
void TCPConnection::ChangeState (TCPState* s) {
_state = s;
}
void TCPConnection::ActiveOpen () {
_state->ActiveOpen(this);
}
void TCPConnection::PassiveOpen () {
_state->PassiveOpen(this);
}
```

其他的代码段都类似，定义了 TCPConnection 和 TCPState 的行为以及它们各自子类 TCPEstablished, TCPListen 和 TCPClosed 的特定行为。

### 已知的应用
Johnson 和 Zweig [JZ91] 描述了状态模式及其在 TCP 连接协议中的应用。

### 相关模式
享元（Flyweight）模式解释了何时以及如何共享状态对象。状态对象通常是单例（Singleton）。

### C# sample code

以下是将给出的 TCP 示例代码转换为 C# 的版本。这段代码保留了相同的对象结构和类继承关系。请注意，C# 允许在一个文件中定义多个类，并且所有的方法都默认为虚拟方法，除非声明为 sealed：

```csharp
public abstract class TCPState
{
    public virtual void Transmit(TCPConnection connection, TCPOctetStream stream) { }
    public virtual void ActiveOpen(TCPConnection connection) { }
    public virtual void PassiveOpen(TCPConnection connection) { }
    public virtual void Close(TCPConnection connection) { }
    public virtual void Synchronize(TCPConnection connection) { }
    public virtual void Acknowledge(TCPConnection connection) { }
    public virtual void Send(TCPConnection connection) { }

    protected void ChangeState(TCPConnection connection, TCPState newState)
    {
        connection.ChangeState(newState);
    }
}

public class TCPClosed : TCPState
{
    private static TCPClosed _instance = new TCPClosed();

    private TCPClosed() { }

    public static TCPState Instance()
    {
        return _instance;
    }

    public override void ActiveOpen(TCPConnection connection)
    {
        ChangeState(connection, TCPEstablished.Instance());
    }

    public override void PassiveOpen(TCPConnection connection)
    {
        ChangeState(connection, TCPListen.Instance());
    }
}

public class TCPEstablished : TCPState
{
    private static TCPEstablished _instance = new TCPEstablished();

    private TCPEstablished() { }

    public static TCPState Instance()
    {
        return _instance;
    }

    public override void Close(TCPConnection connection)
    {
        ChangeState(connection, TCPListen.Instance());
    }

    public override void Transmit(TCPConnection connection, TCPOctetStream stream)
    {
        connection.ProcessOctet(stream);
    }
}

public class TCPListen : TCPState
{
    private static TCPListen _instance = new TCPListen();

    private TCPListen() { }

    public static TCPState Instance()
    {
        return _instance;
    }

    public override void Send(TCPConnection connection)
    {
        ChangeState(connection, TCPEstablished.Instance());
    }
}

public class TCPConnection
{
    private TCPState _state;

    public TCPConnection()
    {
        _state = TCPClosed.Instance();
    }

    public void ChangeState(TCPState newState)
    {
        _state = newState;
    }

    public void ActiveOpen()
    {
        _state.ActiveOpen(this);
    }

    public void PassiveOpen()
    {
        _state.PassiveOpen(this);
    }

    public void Close()
    {
        _state.Close(this);
    }

    public void Send()
    {
        _state.Send(this);
    }

    public void Acknowledge()
    {
        _state.Acknowledge(this);
    }

    public void Synchronize()
    {
        _state.Synchronize(this);
    }

    public void ProcessOctet(TCPOctetStream stream)
    {
        // Process the octet stream
    }
}

public class TCPOctetStream
{
    // Implementation details for this class...
}
```

这个代码片段包括一个 `TCPState` 抽象基类，它定义了所有 TCP 状态类必须实现的接口。此外，还有三个 `TCPState` 的子类：`TCPClosed`、`TCPEstablished` 和 `TCPListen`。这些子类分别实现了在不同的 TCP 状态下的特定行为。

`TCPConnection` 类使用状态模式来管理它的状态。它有一个私有字段 `_state` 来保存当前状态，然后在这个状态上调用各种方法。为了改变状态，`TCPConnection` 类提供了一个 `ChangeState` 方法，这个方法被所有的 `TCPState` 子类使用。

最后，还有一个 `TCPOctetStream` 类。这个类在这个例子中并没有详细说明，但是它被 `TCPConnection` 和 `TCPEstablished` 类所使用。

## chat

.NET Framework 本身可能没有直接的例子使用数据表来实现状态迁移，这主要是因为.NET是一个通用的框架，它提供的是构建各种应用程序的基础设施，而不是某种特定应用领域的解决方案。

然而，这并不意味着你不能在.NET应用程序中使用数据表来实现状态机模型。有些场景下，特别是涉及到复杂的业务规则和状态迁移时，基于表的状态机可以带来很大的灵活性和可维护性。

例如，你可以使用数据库表来描述和跟踪每个状态和可能的状态转换。你的应用程序可以查询这个表来确定在特定状态下可以执行的操作，以及执行特定操作后应该进入的新状态。这样，业务规则和状态逻辑就可以从代码中抽取出来，存储在数据库中，便于管理和修改。

下面是一个简单的例子，这是一个描述订单状态和可能的状态转换的数据库表：

```
OrderStatus
- Id
- Name

OrderStatusTransition
- Id
- FromStatusId
- ToStatusId
- Action
```

然后，你的应用程序可以使用类似于以下的查询来确定在给定的订单状态下可以执行的操作：

```sql
SELECT Action
FROM OrderStatusTransition
WHERE FromStatusId = @currentStatusId
```

这种方法的主要优点是它提供了一种清晰、灵活的方式来管理状态逻辑，特别是在业务规则经常变化的情况下。然而，它也有一些缺点，例如性能可能不如硬编码的状态机，以及需要管理数据库表可能会增加复杂性。

### .net 框架中的 yield return 使用了状态机吗？

是的，`yield return`在.NET中被用于实现协程，这确实是一种形式的状态机。当你在方法中使用`yield return`语句时，C#编译器会生成一个隐藏的类来保存当前的状态（包括局部变量和当前执行的位置），这就形成了一种状态机。

举一个简单的例子，考虑这样一个使用`yield return`的方法：

```csharp
public IEnumerable<int> GetNumbers()
{
    for (int i = 0; i < 10; i++)
    {
        yield return i;
    }
}
```

在上面的代码中，每次调用`GetNumbers`方法都会返回一个新的迭代器。这个迭代器内部有一个状态，跟踪我们目前在循环中的位置。每次我们调用迭代器的`MoveNext`方法，它就会跳到下一个`yield return`语句，并返回`true`。当我们到达方法的末尾时，`MoveNext`返回`false`。

这个状态机背后的细节是由C#编译器自动处理的，所以作为开发者，你并不需要去管理状态机的内部工作。你只需要使用`yield return`语句，编译器就会为你生成正确的代码。

上面的示例代码，如果用状态机模式如何表达，请给出编译器的模拟代码

要手动实现使用 `yield return` 的状态机，我们需要创建一个实现 `IEnumerator<int>` 的类来保存状态，并将循环逻辑分解为多个状态。以下是一个可能的实现：

```csharp
public class NumberGenerator : IEnumerator<int>
{
    private int _state;
    private int _current;
    private int _i;

    public int Current
    {
        get
        {
            return _current;
        }
    }

    object IEnumerator.Current
    {
        get
        {
            return Current;
        }
    }

    public bool MoveNext()
    {
        switch (_state)
        {
            case 0:
                _i = 0;
                _state = 1;
                goto case 1;
            case 1:
                if (_i < 10)
                {
                    _current = _i;
                    _i++;
                    return true;
                }
                else
                {
                    _state = 2;
                }
                break;
        }
        return false;
    }

    public void Reset()
    {
        _state = 0;
        _i = 0;
    }

    public void Dispose()
    {
        _state = 2;
    }
}
```

然后，我们可以创建一个 `IEnumerable<int>` 方法来返回新的 `NumberGenerator` 实例：

```csharp
public IEnumerable<int> GetNumbers()
{
    return new NumberGenerator();
}
```

以上代码创建了一个状态机，与编译器为 `yield return` 生成的代码相似。`NumberGenerator` 类在内部跟踪其状态，`_state` 变量表示我们在循环中的位置，`_current` 和 `_i` 保存了我们的局部变量。