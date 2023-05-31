## 极客时间 笔记

解释器模式更加小众，只在一些特定的领域会被用到，比如编译器、规则引擎、正则表达式。所以，解释器模式也不是我们学习的重点，你稍微了解一下就可以了。


解释器模式的英文翻译是 Interpreter Design Pattern。在 GoF 的《设计模式》一书中，它是这样定义的：

Interpreter pattern is used to defines a grammatical representation for a language and provides an interpreter to deal with this grammar.

翻译成中文就是：解释器模式为某个语言定义它的语法（或者叫文法）表示，并定义一个解释器用来处理这个语法。


解释器模式，其实就是用来实现根据语法规则解读“句子”的解释器。

比如“ 8 3 2 4 - + * ”这样一个表达式，我们按照上面的语法规则来处理，取出数字“8 3”和“-”运算符，计算得到 5，于是表达式就变成了“ 5 2 4 + * ”。然后，我们再取出“ 5 2 ”和“ + ”运算符，计算得到 7，表达式就变成了“ 7 4 * ”。最后，我们取出“ 7 4”和“ * ”运算符，最终得到的结果就是 28。

```c#
using System;
using System.Collections.Generic;

public class ExpressionInterpreter {
    private LinkedList<long> numbers = new LinkedList<long>();

    public long Interpret(string expression) {
        string[] elements = expression.Split(" ");
        int length = elements.Length;
        for (int i = 0; i < (length+1)/2; ++i) {
            numbers.AddLast(long.Parse(elements[i]));
        }

        for (int i = (length+1)/2; i < length; ++i) {
            string operatorSymbol = elements[i];
            bool isValid = operatorSymbol == "+" || operatorSymbol == "-"
                    || operatorSymbol == "*" || operatorSymbol == "/";
            if (!isValid) {
                throw new Exception("Expression is invalid: " + expression);
            }

            long number1 = numbers.First.Value; numbers.RemoveFirst();
            long number2 = numbers.First.Value; numbers.RemoveFirst();
            long result = 0;
            if (operatorSymbol == "+") {
                result = number1 + number2;
            } else if (operatorSymbol == "-") {
                result = number1 - number2;
            } else if (operatorSymbol == "*") {
                result = number1 * number2;
            } else if (operatorSymbol == "/") {
                result = number1 / number2;
            }
            numbers.AddFirst(result);
        }

        if (numbers.Count != 1) {
            throw new Exception("Expression is invalid: " + expression);
        }

        return numbers.First.Value;
    }
}
```
这段代码定义了一个名为 `ExpressionInterpreter` 的类，该类具有一个方法 `Interpret`，用于解释并执行一个包含数字和运算符的字符串表达式。

表达式的格式应该是这样的：前半部分是要计算的数字，后半部分是相应的运算符。数字和运算符之间用空格分隔。例如："3 4 5 + -"。这个例子中，数字 "3" "4" "5" 和操作符 "+" "-" 的组合表示表达式 "(3 - (4 + 5))"。

下面是代码的详细解释：

1. 首先，将输入的字符串 `expression` 分割成各个元素（数字和运算符），并存储在 `elements` 数组中。

2. 然后，将数组的前半部分（数字）解析为长整型，并将它们加入 `numbers` 链表。

3. 对于数组的后半部分（运算符），对于每一个运算符：
   - 检查这个运算符是否有效（是否为 "+", "-", "*" 或 "/"）。
   - 如果无效，抛出异常。
   - 如果有效，从 `numbers` 链表中移除并获取两个数字。
   - 根据运算符，对这两个数字进行相应的运算（加、减、乘、除），并将结果添加到 `numbers` 链表的开头。

4. 最后，如果 `numbers` 链表中不只一个数字（也就是说，表达式无效或者未完全计算），抛出异常。否则，返回 `numbers` 链表中的唯一数字（也就是最终的计算结果）。

这段代码实现了一种简单的后缀表达式（又称逆波兰表示法）计算器。在后缀表达式中，操作符位于操作数之后，这使得它可以省略括号。例如，上面的例子 "3 4 5 + -" 是一个后缀表达式，它等价于中缀表达式 "(3 - (4 + 5))"。

该怎么拆分呢？我们可以借助解释器模式。

chatgpt 的解决方案：

重构这段代码以使用解释器模式，我们可以将每种运算符定义为一个单独的解释器类。这样可以使得代码结构更加清晰，更容易扩展新的运算符。

以下是一个可能的重构方案：

首先，定义一个抽象的表达式接口 `IExpression`，它有一个方法 `Interpret` 用于解释并执行表达式：

```csharp
public interface IExpression {
    long Interpret(LinkedList<long> stack);
}
```

然后，对于每种运算符，定义一个类来实现 `IExpression` 接口。例如，这里是 `AddExpression` 类和 `SubtractExpression` 类的定义：

```csharp
public class AddExpression : IExpression {
    public long Interpret(LinkedList<long> stack) {
        long number1 = stack.First.Value; stack.RemoveFirst();
        long number2 = stack.First.Value; stack.RemoveFirst();
        long result = number1 + number2;
        stack.AddFirst(result);
        return result;
    }
}

public class SubtractExpression : IExpression {
    public long Interpret(LinkedList<long> stack) {
        long number1 = stack.First.Value; stack.RemoveFirst();
        long number2 = stack.First.Value; stack.RemoveFirst();
        long result = number1 - number2;
        stack.AddFirst(result);
        return result;
    }
}
```

类似的，你也可以定义 `MultiplyExpression` 和 `DivideExpression` 类。

然后，修改 `ExpressionInterpreter` 类，使其使用这些新的解释器类：

```csharp
public class ExpressionInterpreter {
    private LinkedList<long> numbers = new LinkedList<long>();
    private Dictionary<string, IExpression> operators = new Dictionary<string, IExpression>();

    public ExpressionInterpreter() {
        operators.Add("+", new AddExpression());
        operators.Add("-", new SubtractExpression());
        // add other operators here...
    }

    public long Interpret(string expression) {
        string[] elements = expression.Split(" ");
        int length = elements.Length;
        for (int i = 0; i < (length+1)/2; ++i) {
            numbers.AddLast(long.Parse(elements[i]));
        }

        for (int i = (length+1)/2; i < length; ++i) {
            string operatorSymbol = elements[i];
            if (!operators.ContainsKey(operatorSymbol)) {
                throw new Exception("Expression is invalid: " + expression);
            }
            operators[operatorSymbol].Interpret(numbers);
        }

        if (numbers.Count != 1) {
            throw new Exception("Expression is invalid: " + expression);
        }

        return numbers.First.Value;
    }
}
```

这种方式的好处是，每种运算符的逻辑都封装在了一个单独的类中，如果要添加新的运算符，只需要定义一个新的解释器类，然后在 `ExpressionInterpreter` 的构造函数中添加一行代码即可。同时，`Interpret` 方法也变得更加简洁。

极客时间 方案

代码实现的核心思想，就是将语法解析的工作拆分到各个小类中，以此来避免大而全的解析类。一般的做法是，将语法规则拆分成一些小的独立的单元，然后对每个单元进行解析，最终合并为对整个语法规则的解析。

前面定义的语法规则有两类表达式，一类是数字，一类是运算符，运算符又包括加减乘除。利用解释器模式，我们把解析的工作拆分到 NumberExpression、AdditionExpression、SubstractionExpression、MultiplicationExpression、DivisionExpression 这样五个解析类中。

```csharp
public interface IExpression {
    long Interpret();
}

public class NumberExpression : IExpression {
    private long number;

    public NumberExpression(long number) {
        this.number = number;
    }

    public NumberExpression(string number) {
        this.number = long.Parse(number);
    }

    public long Interpret() {
        return this.number;
    }
}

public class AdditionExpression : IExpression {
    private IExpression exp1;
    private IExpression exp2;

    public AdditionExpression(IExpression exp1, IExpression exp2) {
        this.exp1 = exp1;
        this.exp2 = exp2;
    }

    public long Interpret() {
        return exp1.Interpret() + exp2.Interpret();
    }
}
// SubtractionExpression, MultiplicationExpression, DivisionExpression are similar to AdditionExpression

public class ExpressionInterpreter {
    private LinkedList<IExpression> numbers = new LinkedList<IExpression>();

    public long Interpret(string expression) {
        string[] elements = expression.Split(" ");
        int length = elements.Length;
        for (int i = 0; i < (length+1)/2; ++i) {
            numbers.AddLast(new NumberExpression(elements[i]));
        }

        for (int i = (length+1)/2; i < length; ++i) {
            string operatorSymbol = elements[i];
            bool isValid = operatorSymbol == "+" || operatorSymbol == "-"
                    || operatorSymbol == "*" || operatorSymbol == "/";
            if (!isValid) {
                throw new Exception("Expression is invalid: " + expression);
            }

            IExpression exp1 = numbers.First.Value; numbers.RemoveFirst();
            IExpression exp2 = numbers.First.Value; numbers.RemoveFirst();
            IExpression combinedExp = null;
            if (operatorSymbol == "+") {
                combinedExp = new AdditionExpression(exp1, exp2);
            } else if (operatorSymbol == "-") {
                // replaced with new SubtractionExpression
            } else if (operatorSymbol == "*") {
                // replaced with new MultiplicationExpression
            } else if (operatorSymbol == "/") {
                // replaced with new DivisionExpression
            }
            long result = combinedExp.Interpret();
            numbers.AddFirst(new NumberExpression(result));
        }

        if (numbers.Count != 1) {
            throw new Exception("Expression is invalid: " + expression);
        }

        return numbers.First.Value.Interpret();
    }
}
```

### 设计模式 笔记

