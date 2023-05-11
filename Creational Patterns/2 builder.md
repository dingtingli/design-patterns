# Builder(生成器/建造者/构建者)



## 极客时间笔记

创建一个对象最常用的方式是，使用 new 关键字调用类的构造函数来完成。我的问题是，什么情况下这种方式就不适用了，就需要采用建造者模式来创建对象呢？

与工厂模式有何区别？

工厂模式是用来创建不同但是相关类型的对象（继承同一父类或者接口的一组子类），由给定的参数来决定创建哪种类型的对象。建造者模式是用来创建一种类型的复杂对象，通过设置不同的可选参数，“定制化”地创建不同的对象。

网上有一个经典的例子很好地解释了两者的区别。

顾客走进一家餐馆点餐，我们利用工厂模式，根据用户不同的选择，来制作不同的食物，比如披萨、汉堡、沙拉。对于披萨来说，用户又有各种配料可以定制，比如奶酪、西红柿、起司，我们通过建造者模式根据用户选择的不同配料来制作披萨。

## Design Patterns 笔记

在这个例子中，我们使用生成器模式创建一个迷宫，迷宫包含房间和门。

首先，我们有一个MazeBuilder类，这个类定义了创建迷宫的接口：

```c#
public interface IMazeBuilder
{
    void BuildMaze();
    void BuildRoom(int room);
    void BuildDoor(int roomFrom, int roomTo);
    Maze GetMaze();//builder()
}

```

接着，我们有一个 MazeGame 类，这个类有一个 createMaze 方法，这个方法接收一个 MazeBuilder 对象作为参数，然后使用这个 MazeBuilder 对象来创建迷宫：

```c#
public class MazeGame {
    public Maze CreateMaze(IMazeBuilder builder)
    {
        builder.BuildMaze();
        builder.BuildRoom(1);
        builder.BuildRoom(2);
        builder.BuildDoor(1, 2);
        return builder.GetMaze();
    }
}
```

MazeBuilder类只是定义了创建迷宫的接口，真正创建迷宫的是MazeBuilder的子类。例如，我们有一个StandardMazeBuilder类，这个类继承自MazeBuilder类，它实现了创建迷宫的具体方法：

```c#
public class StandardMazeBuilder : IMazeBuilder
{
    private Maze _currentMaze;

    public StandardMazeBuilder()
    {
        _currentMaze = null;
    }

    public void BuildMaze()
    {
        _currentMaze = new Maze();
    }

    public void BuildRoom(int room)
    {
        if (_currentMaze.RoomNo(room) == null)
        {
            Room roomObj = new Room(room);
            _currentMaze.AddRoom(roomObj);

            // Assuming SetSide() method and Direction enum exist in Room class
            roomObj.SetSide(Direction.North, new Wall());
            roomObj.SetSide(Direction.South, new Wall());
            roomObj.SetSide(Direction.East, new Wall());
            roomObj.SetSide(Direction.West, new Wall());
        }
    }

    public void BuildDoor(int roomFrom, int roomTo)
    {
        Room r1 = _currentMaze.RoomNo(roomFrom);
        Room r2 = _currentMaze.RoomNo(roomTo);
        Door d = new Door(r1, r2);

        // Assuming CommonWall() method exists to find the common wall between two rooms
        r1.SetSide(CommonWall(r1, r2), d);
        r2.SetSide(CommonWall(r2, r1), d);
    }

    public Maze GetMaze()
    {
        return _currentMaze;
    }

    // Method to find the common wall between two rooms
    private Direction CommonWall(Room r1, Room r2)
    {
        // Logic to find the common wall
    }
}

```

我们还有一个 CountingMazeBuilder 类，这个类也继承自 MazeBuilder 类，但它并不真正创建迷宫，它只是计算创建迷宫的过程中会创建多少个房间和门：

```c#
public class CountingMazeBuilder : IMazeBuilder
{
    private int _rooms;
    private int _doors;

    public CountingMazeBuilder()
    {
        _rooms = 0;
        _doors = 0;
    }

    public void BuildMaze() { }

    public void BuildRoom(int room)
    {
        _rooms++;
    }

    public void BuildDoor(int roomFrom, int roomTo)
    {
        _doors++;
    }

    public Maze GetMaze()
    {
        return null;
    }

    public void GetCounts(out int rooms, out int doors)
    {
        rooms = _rooms;
        doors = _doors;
    }
}

```

我们可以这样使用 StandardMazeBuilder 和 CountingMazeBuilder 类：

```c#
MazeGame game = new MazeGame();
StandardMazeBuilder builder = new StandardMazeBuilder();
game.CreateMaze(builder);
Maze maze = builder.GetMaze();

CountingMazeBuilder countingBuilder = new CountingMazeBuilder();
game.CreateMaze(countingBuilder);
countingBuilder.GetCounts(out int rooms, out int doors);
Console.WriteLine($"The maze has {rooms} rooms and {doors} doors");
```

## 完整代码

```c#

public interface IMazeBuilder
{
    IMazeBuilder BuildMaze();
    IMazeBuilder BuildRoom(int room);
    IMazeBuilder BuildDoor(int roomFrom, int roomTo);
    Maze GetMaze();
}

public class StandardMazeBuilder : IMazeBuilder
{
    private Maze _currentMaze;
    public IMazeBuilder BuildMaze()
    {
        _currentMaze = new Maze();
        return this;
    }

    public IMazeBuilder BuildRoom(int room)
    {
        if (_currentMaze.RoomNo(room) == null)
        {
            Room roomObj = new Room(room);
            _currentMaze.AddRoom(roomObj);

            // Assuming SetSide() method and Direction enum exist in Room class
            roomObj.SetSide(Direction.North, new Wall());
            roomObj.SetSide(Direction.South, new Wall());
            roomObj.SetSide(Direction.East, new Wall());
            roomObj.SetSide(Direction.West, new Wall());
        }
        return this;
    }

    public IMazeBuilder BuildDoor(int roomFrom, int roomTo)
    {
        Room r1 = _currentMaze.RoomNo(roomFrom);
        Room r2 = _currentMaze.RoomNo(roomTo);
        Door d = new Door(r1, r2);

        // Assuming CommonWall() method exists to find the common wall between two rooms
        r1.SetSide(CommonWall(r1, r2), d);
        r2.SetSide(CommonWall(r2, r1), d);

        return this;
    }

    public Maze GetMaze()
    {
        return _currentMaze;
    }

    // Method to find the common wall between two rooms
    private Direction CommonWall(Room r1, Room r2)
    {
        // Logic to find the common wall
    }
}

public class MazeGame
{
    public Maze CreateMaze(IMazeBuilder builder)
    {
        return builder.BuildMaze()
            .BuildRoom(1)
            .BuildRoom(2)
            .BuildDoor(1, 2)
            .GetMaze();
    }
}

```