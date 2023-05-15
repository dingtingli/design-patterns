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

## 极客时间 笔记

如果对象的创建成本比较大，而同一个类的不同对象之间差别不大（大部分字段都相同），在这种情况下，我们可以利用对已有对象（原型）进行复制（或者叫拷贝）的方式来创建新对象，以达到节省创建时间的目的。这种基于原型来创建对象的方式就叫作原型设计模式（Prototype Design Pattern），简称原型模式。

为什么说创建对象的成本较大呢？

为什么通过复制的方式创建对象的成本就不大呢？


实际上，创建对象包含的申请内存、给成员变量赋值这一过程，本身并不会花费太多时间，或者说对于大部分业务系统来说，这点时间完全是可以忽略的。应用一个复杂的模式，只得到一点点的性能提升，这就是所谓的过度设计，得不偿失。

但是，如果对象中的数据需要经过复杂的计算才能得到（比如排序、计算哈希值），或者需要从 RPC、网络、数据库、文件系统等非常慢速的 IO 中读取，这种情况下，我们就可以利用原型模式，从其他已有对象中直接拷贝得到，而不用每次在创建新对象的时候，都重复执行这些耗时的操作。


例子1 ：

我们只需要在系统 A 中，记录当前数据的版本 Va 对应的更新时间 Ta，从数据库中捞出更新时间大于 Ta 的所有搜索关键词，也就是找出 Va 版本与最新版本数据的“差集”，然后针对差集中的每个关键词进行处理。如果它已经在散列表中存在了，我们就更新相应的搜索次数、更新时间等信息；如果它在散列表中不存在，我们就将它插入到散列表中。

```c#
public class Demo
{
    private ConcurrentDictionary<string, SearchWord> currentKeywords = new ConcurrentDictionary<string, SearchWord>();
    private long lastUpdateTime = -1;

    public void Refresh()
    {
        // Get data from the database where the update time is greater than lastUpdateTime, and put it into currentKeywords
        List<SearchWord> toBeUpdatedSearchWords = GetSearchWords(lastUpdateTime);
        long maxNewUpdatedTime = lastUpdateTime;
        foreach (SearchWord searchWord in toBeUpdatedSearchWords)
        {
            if (searchWord.LastUpdateTime > maxNewUpdatedTime)
            {
                maxNewUpdatedTime = searchWord.LastUpdateTime;
            }
            if (currentKeywords.ContainsKey(searchWord.Keyword))
            {
                currentKeywords[searchWord.Keyword] = searchWord;
            }
            else
            {
                currentKeywords.TryAdd(searchWord.Keyword, searchWord);
            }
        }

        lastUpdateTime = maxNewUpdatedTime;
    }

    private List<SearchWord> GetSearchWords(long lastUpdateTime)
    {
        // TODO: Get data from the database where the update time is greater than lastUpdateTime
        return null;
    }
}

public class SearchWord
{
    public string Keyword {get;set;}
    public int SearchCount { get; set; }
    public long LastUpdateTime { get; set; }
}

```

例子2：

现在，我们有一个特殊的要求：任何时刻，系统 A 中的所有数据都必须是同一个版本的，要么都是版本 a，要么都是版本 b，不能有的是版本 a，有的是版本 b。那刚刚的更新方式就不能满足这个要求了。除此之外，我们还要求：在更新内存数据的时候，系统 A 不能处于不可用状态，也就是不能停机更新数据。

我们把正在使用的数据的版本定义为“服务版本”，当我们要更新内存中的数据的时候，我们并不是直接在服务版本（假设是版本 a 数据）上更新，而是重新创建另一个版本数据（假设是版本 b 数据），等新的版本数据建好之后，再一次性地将服务版本从版本 a 切换到版本 b。这样既保证了数据一直可用，又避免了中间状态的存在。

```c#
public class Demo
{
    private Dictionary<string, SearchWord> currentKeywords = new Dictionary<string, SearchWord>();

    public void Refresh()
    {
        var newKeywords = new Dictionary<string, SearchWord>();

        // 从数据库中取出所有的数据，放入到newKeywords中
        List<SearchWord> toBeUpdatedSearchWords = GetSearchWords();

        foreach (var searchWord in toBeUpdatedSearchWords)
        {
            newKeywords[searchWord.Keyword] = searchWord;
        }

        currentKeywords = newKeywords;
    }

    private List<SearchWord> GetSearchWords()
    {
        // TODO: 从数据库中取出所有的数据
        return null;
    }
}

```

例子3：

拷贝 currentKeywords 数据到 newKeywords 中，然后从数据库中只捞出新增或者有更新的关键词，更新到 newKeywords 中。

```c#
public class Demo
{
    private ConcurrentDictionary<string, SearchWord> currentKeywords = new ConcurrentDictionary<string, SearchWord>();
    private long lastUpdateTime = -1;

    public void Refresh()
    {
        var newKeywords = new ConcurrentDictionary<string, SearchWord>(currentKeywords);

        // 从数据库中取出更新时间>lastUpdateTime的数据，放入到newKeywords中
        List<SearchWord> toBeUpdatedSearchWords = GetSearchWords(lastUpdateTime);
        long maxNewUpdatedTime = lastUpdateTime;

        foreach (var searchWord in toBeUpdatedSearchWords)
        {
            if (searchWord.LastUpdateTime > maxNewUpdatedTime)
            {
                maxNewUpdatedTime = searchWord.LastUpdateTime;
            }
            if (newKeywords.ContainsKey(searchWord.Keyword))
            {
                newKeywords[searchWord.Keyword] = searchWord;
            }
            else
            {
                newKeywords.TryAdd(searchWord.Keyword, searchWord);
            }
        }

        lastUpdateTime = maxNewUpdatedTime;
        currentKeywords = newKeywords;
    }

    private List<SearchWord> GetSearchWords(long lastUpdateTime)
    {
        // TODO: 从数据库中取出更新时间>lastUpdateTime的数据
        return null;
    }
}

```

newKeywords 是 currentKeywords 的浅拷贝。新的ConcurrentDictionary<string, SearchWord>是创建了一个新的字典对象newKeywords，并把currentKeywords的所有键值对复制过去。但是，字典的值SearchWord对象并没有被复制，newKeywords和currentKeywords中的相同键对应的SearchWord对象是同一个引用。换句话说，如果你修改newKeywords中某个SearchWord对象的属性，currentKeywords中对应的那个SearchWord对象的属性也会被改变。


例子4：

深拷贝方式1：

```c#
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;

public class Demo 
{
    private ConcurrentDictionary<string, SearchWord> currentKeywords = new ConcurrentDictionary<string, SearchWord>();
    private long lastUpdateTime = -1;

    public void Refresh() 
    {
        var newKeywords = new ConcurrentDictionary<string, SearchWord>();
        foreach(var kv in currentKeywords)
        {
            newKeywords[kv.Key] = new SearchWord(kv.Value);
        }

        List<SearchWord> toBeUpdatedSearchWords = GetSearchWords(lastUpdateTime);
        long maxNewUpdatedTime = lastUpdateTime;
        foreach (SearchWord searchWord in toBeUpdatedSearchWords) 
        {
            if (searchWord.LastUpdateTime > maxNewUpdatedTime) 
            {
                maxNewUpdatedTime = searchWord.LastUpdateTime;
            }
            if (newKeywords.ContainsKey(searchWord.Keyword)) 
            {
                newKeywords[searchWord.Keyword] = searchWord;
            } 
            else 
            {
                newKeywords.TryAdd(searchWord.Keyword, searchWord);
            }
        }

        lastUpdateTime = maxNewUpdatedTime;
        currentKeywords = newKeywords;
    }

    private List<SearchWord> GetSearchWords(long lastUpdateTime) 
    {
        // TODO: Retrieve from database the data where the update time > lastUpdateTime
        return null;
    }
}

public class SearchWord 
{
    public string Keyword { get; set; }
    public long LastUpdateTime { get; set; }
    public int Count { get; set; }

    // Assuming a copy constructor exists
    public SearchWord(SearchWord other) 
    {
        Keyword = other.Keyword;
        LastUpdateTime = other.LastUpdateTime;
        Count = other.Count;
    }
}

```

方法1
var newKeywords = new ConcurrentDictionary<string, SearchWord>();
        foreach(var kv in currentKeywords)
        {
            newKeywords[kv.Key] = new SearchWord(kv.Value);
        }

方法2
var newKeywords = new ConcurrentDictionary<string, SearchWord>(
    currentKeywords.ToDictionary(
        entry => entry.Key, 
        entry => new SearchWord(entry.Value)
    )
);

方法1 是显式地遍历 currentKeywords，对每个元素进行拷贝然后添加到新的 ConcurrentDictionary 中。这种方法的好处是过程清晰，容易理解。

方法2 则是利用了 LINQ (Language Integrated Query) 中的 ToDictionary 方法，然后将得到的普通 Dictionary 传给 ConcurrentDictionary 的构造函数。这种方法的优点是代码简洁，看起来更为优雅，但是需要读者对 LINQ 和构造函数的行为有一定理解。

深拷贝方式2：

我们可以使用Json.NET库来进行对象的序列化和反序列化，从而实现深拷贝。在这个例子中，我们可以先将currentKeywords序列化为json字符串，然后再将这个json字符串反序列化为新的ConcurrentDictionary<string, SearchWord>对象。这样，新的对象和原来的对象就没有任何关联了，实现了深拷贝。

```c#
using Newtonsoft.Json;

//...
var newKeywords = JsonConvert.DeserializeObject<ConcurrentDictionary<string, SearchWord>>(
    JsonConvert.SerializeObject(currentKeywords, new JsonSerializerSettings() 
    { 
        PreserveReferencesHandling = PreserveReferencesHandling.Objects 
    })
);

```

我们在调用SerializeObject方法时传入了一个JsonSerializerSettings对象，这个对象的PreserveReferencesHandling属性被设置为PreserveReferencesHandling.Objects。这是为了在序列化和反序列化过程中保留对象引用，从而正确地处理循环引用和复杂的对象图。

深拷贝方式3：

我们可以先采用浅拷贝的方式创建 newKeywords。对于需要更新的 SearchWord 对象，我们再使用深度拷贝的方式创建一份新的对象，替换 newKeywords 中的老对象。毕竟需要更新的数据是很少的。这种方式即利用了浅拷贝节省时间、空间的优点，又能保证 currentKeywords 中的中数据都是老版本的数据。

```c#
public class Demo
{
    private ConcurrentDictionary<string, SearchWord> currentKeywords = new ConcurrentDictionary<string, SearchWord>();
    private long lastUpdateTime = -1;

    public void Refresh()
    {
        var newKeywords = new ConcurrentDictionary<string, SearchWord>(currentKeywords);

        var toBeUpdatedSearchWords = GetSearchWords(lastUpdateTime);
        long maxNewUpdatedTime = lastUpdateTime;

        foreach (var searchWord in toBeUpdatedSearchWords)
        {
            if (searchWord.LastUpdateTime > maxNewUpdatedTime)
            {
                maxNewUpdatedTime = searchWord.LastUpdateTime;
            }

            // 使用深拷贝替换 newKeywords 中的老对象
            if (newKeywords.ContainsKey(searchWord.Keyword))
            {
                newKeywords[searchWord.Keyword] = new SearchWord(searchWord);
            }
            else
            {
                newKeywords[searchWord.Keyword] = searchWord;
            }
        }

        lastUpdateTime = maxNewUpdatedTime;
        currentKeywords = newKeywords;
    }

    private List<SearchWord> GetSearchWords(long lastUpdateTime)
    {
        // TODO: 从数据库中取出更新时间>lastUpdateTime的数据
        return null;
    }
}

```