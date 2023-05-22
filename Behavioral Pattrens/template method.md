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