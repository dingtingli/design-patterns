## Prototype(原型)

原型设计模式是一种创建型设计模式，其主要目标是通过复制一个原型实例来创建新的对象。

## 乐谱编辑器

### 目的：

建立一个乐谱编辑器，该编辑器有一个图形编辑界面，并且能添加比如：音符（notes），休止符（rests）和五线谱（staves）等对象。

编辑器中有一个工具面板，用于将这些音乐对象添加到乐谱中。该工具面板还包括一些工具，用于选择、移动（等其他动作）这些音乐对象。

比如，用户可以点击四分音符（quarter-note）工具，用它来向乐谱中添加四分音符。或者使用移动工具，在五线谱（staff）中向上或向下移动一个音符（note），从而改变其音高。

### 面向对象分析：

提供一个抽象的 Graphic 类，用于表示图形组件，比如音符（notes）和五线谱（staves）。

提供了一个抽象的 Tool 类，用于定义工具面板中的工具。

定义了一个 GraphicTool 子类，用于创建图形组件的实例并将它们添加到乐谱中。

Graphic 类是应用程序特有的，而 GraphicTool 属于工具的一部分。GraphicTool 不知道如何将我们的 Grahic 类的实例（比如音符，休止符等）添加到乐谱。

我们可以为每个 Graphic 的音乐类对象创建一个 GraphicTool 的子类，但这将产生大量的子类（因为音乐类会有很多），而这些子类只在它们实例化的音乐对象的种类上有所不同。

如何让 GraphicTool 创建图形组件类的实例并将它们添加到乐谱中呢？

如果我们为每种图形组件对象创建一个 GraphicTool 的子类，将会产生大量的子类，这些子类的唯一区别仅在于它们实例化的图形组件对象类型。

其实，每个用于创建图形组件对象的工具都是 GraphicTool 的实例，使用不同的原型进行初始化。每个 GraphicTool 实例将通过复制其原型并将克隆体添加到乐谱中来产生音乐对象。（开始看不懂了）。

### 代码示例

Graphic类：这个是所有图形对象的基类。在这个音乐编辑器的例子中，图形对象可能包括音符，休止符和五线谱。

每个图形对象都可以被复制，所以这个基类定义了一个抽象的Clone方法。

```C#
public abstract class Graphic
{
    public abstract Graphic Clone();
    public abstract void Draw(Position position);
    // Other common graphic methods
}
```

音乐图像组件 Note(音符)

```C#
public class Note : Graphic
{
    public override Graphic Clone()
    {
        return new Note();
    }

    public override void Draw(Position position)
    {
        //Console.WriteLine("Drawing a Note");
    }

    // Other note-specific methods
}
```

音乐图像组件 rest(休止符)

```C#
public class Rest  : Graphic
{
    public override Graphic Clone()
    {
        return new Rest ();
    }

    public override void Draw(Position position)
    {
        //Console.WriteLine("Drawing a Note");
    }

    // Other Rest-specific methods
}
```

Tool类：这是所有工具的基类。在音乐编辑器中，工具可能包括创建音符的工具，移动音符的工具等等。这个类定义了一个抽象的Manipulate方法，表示使用这个工具进行操作。

```C#
public abstract class Tool
{
    public abstract void Manipulate();
    // Other common tool methods
}
```

GraphicTool类：这是Tool类的一个具体子类，用于创建和操作Graphic对象。这个类维护了一个Graphic对象的实例，也就是原型。当调用Manipulate方法时，它会复制这个原型，然后对复制出的新对象进行操作。

```c#
public class GraphicTool : Tool
{
    private Graphic _prototype;

    public GraphicTool(Graphic prototype)
    {
        _prototype = prototype;
    }

    public Graphic CreateGraphic()
    {
        return _prototype.Clone();
    }

    public override void Manipulate()
    {
        // Manipulate the graphic here
        Console.WriteLine($"Manipulating a {_prototype.GetType().Name}");
    }
}
```

减少子类化:
我们只需要一个 GraphicTool 类，它可以用于创建任何类型的 Graphic 对象。

```c#
public static void Main(string[] args)
    {
        // Create tools for each music element
        GraphicTool noteTool = new GraphicTool(new Note());
        GraphicTool restTool = new GraphicTool(new Rest());

        // Create a list to hold the music sequence
        List<Graphic> musicSequence = new List<Graphic>();

        // Use a simple random number generator to create a music sequence
        Random rng = new Random();
        for (int i = 0; i < 10; i++)
        {
            if (rng.Next(2) == 0) // Half the time, add a note
            {
                Graphic note = noteTool.CreateGraphic();
                musicSequence.Add(note);
            }
            else // The other half of the time, add a rest
            {
                Graphic rest = restTool.CreateGraphic();
                musicSequence.Add(rest);
            }
        }

        // Draw and manipulate the music sequence
        foreach (Graphic graphic in musicSequence)
        {
            graphic.Draw();

            // Here we use the appropriate tool to manipulate the graphic
            if (graphic is Note)
            {
                noteTool.Manipulate();
            }
            else if (graphic is Rest)
            {
                restTool.Manipulate();
            }
        }
    }
```

## 迷宫

