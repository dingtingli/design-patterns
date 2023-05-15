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

MapSite 是一个抽象基类，Room、Wall 和 Door 是 MapSite 的具体实现。每个具体的 MapSite 类都实现了 Clone 方法。

```c#
public abstract class MapSite
{
    public abstract MapSite Clone();
    // Other common methods
}
```


```c#
public class Room : MapSite
{
    public override MapSite Clone()
    {
        return new Room();
    }
    // Other room-specific methods
}
```


```c#
public class Wall : MapSite
{
    public override MapSite Clone()
    {
        return new Wall();
    }
    // Other wall-specific methods
}
```


```c#
public class Door : MapSite
{
    private Room _room1;
    private Room _room2;

    public Door(Room room1, Room room2)
    {
        _room1 = room1;
        _room2 = room2;
    }

    public void Initialize(Room room1, Room room2)
    {
        _room1 = room1;
        _room2 = room2;
    }

    public override MapSite Clone()
    {
        return new Door(_room1, _room2);
    }
    // Other door-specific methods
}
```


```c#
public class Maze
{
    // Maze methods
}
```
MazePrototypeFactory 是一个工厂类，它维护了这些类的实例，当需要创建新的对象时，它会复制（Clone）这些实例，而不是直接创建新的实例。

```c#
public class MazePrototypeFactory
{
    private Maze _prototypeMaze;
    private Room _prototypeRoom;
    private Wall _prototypeWall;
    private Door _prototypeDoor;

    public MazePrototypeFactory(Maze m, Wall w, Room r, Door d)
    {
        _prototypeMaze = m;
        _prototypeWall = w;
        _prototypeRoom = r;
        _prototypeDoor = d;
    }

    public Maze MakeMaze()
    {
        return _prototypeMaze; // It may need to implement a Clone method in Maze class.
    }

    public Room MakeRoom()
    {
        return (Room)_prototypeRoom.Clone();
    }

    public Wall MakeWall()
    {
        return (Wall)_prototypeWall.Clone();
    }

    public Door MakeDoor(Room r1, Room r2)
    {
        Door door = (Door)_prototypeDoor.Clone();
        door.Initialize(r1, r2);
        return door;
    }
}
```
MazeGame 类是使用 MazePrototypeFactory 的客户端，它通过工厂类来创建迷宫的部件（如房间、门和墙），并将这些部件组合成一个迷宫。在这个例子中，我们可以创建一个“默认”的迷宫，它使用标准的墙、房间和门。
```c#
public class MazeGame
{
    public Maze CreateMaze(MazePrototypeFactory factory)
    {
        Maze aMaze = factory.MakeMaze();
        Room r1 = factory.MakeRoom();
        Room r2 = factory.MakeRoom();
        Door theDoor = factory.MakeDoor(r1, r2);

        // add rooms into the maze and set door between r1 and r2

        return aMaze;
    }
}
```

然后，如果我们想要创建一个特殊类型的迷宫，例如一个炸弹迷宫（其中的墙和房间都可能有炸弹），我们就可以创建新的 Wall 和 Room 子类，例如 BombedWall 和 RoomWithABomb，并在 MazePrototypeFactory 中使用这些新的类的实例作为原型。

```c#
public class BombedWall : Wall
{
    private bool _bomb;

    public BombedWall()
    {
        _bomb = false;
    }

    public BombedWall(bool bomb)
    {
        _bomb = bomb;
    }

    public override Wall Clone()
    {
        return new BombedWall(_bomb);
    }

    public bool HasBomb()
    {
        return _bomb;
    }
}

public class RoomWithABomb : Room
{
    // Bombed room specific methods
}

public class MazeGameWithBomb
{
    public Maze CreateMaze(MazePrototypeFactory factory)
    {
        Maze aMaze = factory.MakeMaze();
        Room r1 = factory.MakeRoom();
        Room r2 = factory.MakeRoom();
        Door theDoor = factory.MakeDoor(r1, r2);

        // add rooms into the maze and set door between r1 and r2

        return aMaze;
    }
}
```

```c#
public static void Main()
{
    MazeGameWithBomb game = new MazeGameWithBomb();
    MazePrototypeFactory bombedMazeFactory = 
        new MazePrototypeFactory(
                new Maze(), 
                new BombedWall(true), 
                new RoomWithABomb(), 
                new Door(null, null));
    Maze maze = game.CreateMaze(bombedMazeFactory);
}
```

## 完整代码

```c#
public abstract class MapSite
{
    public abstract MapSite Clone();
    // Other common methods
}

public class Room : MapSite
{
    public override MapSite Clone()
    {
        return new Room();
    }
    // Other room-specific methods
}

public class Wall : MapSite
{
    public override MapSite Clone()
    {
        return new Wall();
    }
    // Other wall-specific methods
}

public class Door : MapSite
{
    private Room _room1;
    private Room _room2;

    public Door(Room room1, Room room2)
    {
        _room1 = room1;
        _room2 = room2;
    }

    public void Initialize(Room room1, Room room2)
    {
        _room1 = room1;
        _room2 = room2;
    }

    public override MapSite Clone()
    {
        return new Door(_room1, _room2);
    }
    // Other door-specific methods
}

public class Maze
{
    // Maze methods
}

public class MazePrototypeFactory
{
    private Maze _prototypeMaze;
    private Room _prototypeRoom;
    private Wall _prototypeWall;
    private Door _prototypeDoor;

    public MazePrototypeFactory(Maze m, Wall w, Room r, Door d)
    {
        _prototypeMaze = m;
        _prototypeWall = w;
        _prototypeRoom = r;
        _prototypeDoor = d;
    }

    public Maze MakeMaze()
    {
        return _prototypeMaze; // It may need to implement a Clone method in Maze class.
    }

    public Room MakeRoom()
    {
        return (Room)_prototypeRoom.Clone();
    }

    public Wall MakeWall()
    {
        return (Wall)_prototypeWall.Clone();
    }

    public Door MakeDoor(Room r1, Room r2)
    {
        Door door = (Door)_prototypeDoor.Clone();
        door.Initialize(r1, r2);
        return door;
    }
}

public class MazeGame
{
    public Maze CreateMaze(MazePrototypeFactory factory)
    {
        Maze aMaze = factory.MakeMaze();
        Room r1 = factory.MakeRoom();
        Room r2 = factory.MakeRoom();
        Door theDoor = factory.MakeDoor(r1, r2);

        // add rooms into the maze and set door between r1 and r2

        return aMaze;
    }
}

public class BombedWall : Wall
{
    private bool _bomb;

    public BombedWall()
    {
        _bomb = false;
    }

    public BombedWall(bool bomb)
    {
        _bomb = bomb;
    }

    public override Wall Clone()
    {
        return new BombedWall(_bomb);
    }

    public bool HasBomb()
    {
        return _bomb;
    }
}

public class RoomWithABomb : Room
{
    // Bombed room specific methods
}

public class MazeGameWithBomb
{
    public Maze CreateMaze(MazePrototypeFactory factory)
    {
        Maze aMaze = factory.MakeMaze();
        Room r1 = factory.MakeRoom();
        Room r2 = factory.MakeRoom();
        Door theDoor = factory.MakeDoor(r1, r2);

        // add rooms into the maze and set door between r1 and r2

        return aMaze;
    }
}

public static void Main()
{
    MazeGameWithBomb game = new MazeGameWithBomb();
    MazePrototypeFactory bombedMazeFactory = new MazePrototypeFactory(new Maze(), new BombedWall(true), new RoomWithABomb(), new Door(null, null));
    Maze maze = game.CreateMaze(bombedMazeFactory);
}


```
