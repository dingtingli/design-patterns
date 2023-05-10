# 准备

## 1. 迷宫游戏需求

只关注迷宫（Maze）的创建，不关注迷宫的其他细节。

1. 迷宫由一系列房间（Room）组成。

2. 每个房间有四个方向（Direction），每个方向上可以是另外一个房间、墙壁（Wall）或者是门（Door）。

3. 一个房间和相邻的另一个房间，有三种情况：a.可以直接相通，b.可以通过门来连接，c. 可以通过墙来阻隔。

4. 门有两种状态：开和关。当门是开着的时候，玩家可以通过门进入另一个房间；当门是关着的时候，玩家无法通过门进入另一个房间。

## 2. 代码准备

```
+----------------+     +------------+     +---------+     +-----------------+
|     Maze       |<>---|  Room      |<>---| Wall    |     | Door            |
+----------------+     +------------+     +---------+     +-----------------+
|                |     |            |     |         |     |                 |
| + AddRoom()    |     | + Enter()  |     | +Enter()|     | +Enter()        |
| + RoomNo()     |     | + GetSide()|     |         |     | +OtherSideFrom()|
|                |     | + SetSide()|     |         |     |                 |
+----------------+     +------------+     +---------+     +-----------------+
                           /_\                                     
                            |                                      
                      +------------+                               
                      |  MapSite   |                               
                      +------------+                               
                      | + Enter()  |                               
                      +------------+ 
```

### 2.1 每个房间有四个方向（Direction），使用枚举类型 Direction 来指定房间的四个方向。

```c#
public enum Direction { North, South, East, West };
```
### 2.2 迷宫的组件的基类

迷宫的组件(MapSite)：房间、门和墙是迷宫的基本组件，它们都是 MapSite 类的子类。

```C#
public abstract class MapSite {
    public abstract void Enter();
}
```

Enter 方法为更复杂的游戏操作提供了一个简单的基础。

例如，如果你在一个房间中说“向东走”，游戏可以简单地确定哪个MapSite 在东边，然后调用它的 Enter 方法。特定子类的 Enter 操作会计算出你的位置是否改变，或者是否碰壁。

### 2.3 迷宫组件，房间（Room）

1. 房间（Room）是 MapSite的一个具体的子类，它定义了迷宫中组件之间的关键关系。

2. 房间（Room）有指向其他方向 MapSite 对象（即相邻的房间、墙壁和门）的引用，以便在需要时访问这些相邻对象。

3. 房间（Room）保存了自己的编号 roomNumber，这个编号将在迷宫中标识每个房间。

```c#
public class Room : MapSite {
    private Dictionary<Direction, MapSite> _sides = new Dictionary<Direction, MapSite>();
    //private Dictionary<Direction, MapSite> _sides = new Dictionary<Direction, MapSite>() {
    //        { Direction.North, null },
    //        { Direction.East, null },
    //        { Direction.South, null },
    //        { Direction.West, null },
    //    };
    private int _roomNumber;

    public Room(int roomNo) {
        _roomNumber = roomNo;
    }

    public MapSite GetSide(Direction direction) {
        return _sides[direction];
    }

    public void SetSide(Direction direction, MapSite mapSite) {
        _sides[direction] = mapSite;
    }

    public override void Enter() {
        // Implementation
    }
}
```

### 2.4 迷宫组件，墙（Wall）

```C#
public class Wall : MapSite {
    public Wall() {
    }

    public override void Enter() {
        // Implementation
    }
}
```

### 2.5 迷宫组件，门（Door）

表示房间之间的连接通道。一个门可以连接两个房间，并允许玩家在这两个房间之间移动。

1. 连接两个房间：Door 对象包含指向两个相邻房间的引用，表示这两个房间是通过门相连的。

2. 控制通行状态：Door 对象具有一个状态 isOpen（开或关），决定了是否能够通过这扇门从一个房间进入另一个房间。

3. 提供辅助功能：Door 类还提供了一些辅助方法，例如 OtherSideFrom() 方法，用于确定给定房间的另一侧是哪个房间。

```C#
public class Door : MapSite {
    private Room _room1;
    private Room _room2;
    private bool _isOpen;

    public Door(Room room1 = null, Room room2 = null) {
        _room1 = room1;
        _room2 = room2;
    }

    public override void Enter() {
        // Implementation
    }

    public Room OtherSideFrom(Room room) {
        // Implementation
    }
}
```
### 2.6 迷宫房间的集合 Maze

Maze 类在迷宫游戏中表示整个迷宫结构。它主要负责组织和管理迷宫中的房间。

1. 添加房间：Maze 类提供了一个 AddRoom() 方法，用于将新的房间添加到迷宫中。

2. 查找房间：Maze 类提供了一个 RoomNo() 方法，根据房间号查找并返回对应的房间。 RoomNo() 可以使用线性搜索、hash表，甚至简单的数组进行查找。下面的示例中使用 Dictionary 进行查找。

```C#
public class Maze {
    private Dictionary<int, Room> _rooms = new Dictionary<int, Room>();

    public Maze() {
    }

    public void AddRoom(Room room) {
        _rooms[room.RoomNumber] = room;
    }

    public Room RoomNo(int roomNumber) {
        return _rooms[roomNumber];
    }
}
```

### 2.7 创建迷宫 MazeGame

包含了用于创建迷宫组件（如房间、墙壁和门）并将它们组合成完整迷宫结构的方法。

1. 创建迷宫：MazeGame 类提供了一个方法（ CreateMaze()），用于生成一个具有特定布局和结构的迷宫。

2. 定义迷宫构建逻辑：MazeGame 类实现了构建迷宫的具体逻辑。这包括初始化房间、门和墙的对象，以及正确地将它们组织在一起。

```C#
public class MazeGame {
    public Maze CreateMaze() {
        Maze aMaze = new Maze();
        Room r1 = new Room(1);
        Room r2 = new Room(2);
        Door theDoor = new Door(r1, r2);

        aMaze.AddRoom(r1);
        aMaze.AddRoom(r2);

        r1.SetSide(Direction.North, new Wall());
        r1.SetSide(Direction.East, theDoor);
        r1.SetSide(Direction.South, new Wall());
        r1.SetSide(Direction.West, new Wall());

        r2.SetSide(Direction.North, new Wall());
        r2.SetSide(Direction.East, new Wall());
        r2.SetSide(Direction.South, new Wall());
        r2.SetSide(Direction.West, theDoor);

        return aMaze;
    }
}
```

## 3. 局限和解决方法

上面的示例代码中创建了一个包含两个房间和一个门构成的简单迷宫。这种方法有一定的局限，因为迷宫布局是硬编码的，要更改布局就需要修改这个方法。

创建型设计模式可以提高这个设计的灵活性，使得更改组件类变得更容易。例如，使用工厂方法、抽象工厂、建造者或原型模式，可以更方便地修改迷宫布局和组件类型。以下简要概述了这些模式如何应用于迷宫构建场景：

1. 工厂方法（Factory Method）模式：通过在 MazeGame 类中使用虚拟函数（而非构造函数）来创建房间、墙和门，可以通过创建 MazeGame 的子类并重新定义这些虚拟函数来更改实例化的类。

2. 抽象工厂（Abstract Factory）模式：如果 CreateMaze 方法接受一个用于创建房间、墙和门的对象作为参数，那么可以通过传递不同的参数来更改房间、墙和门的类。

3. 建造者（Builder）模式：如果将一个可以使用添加房间、门和墙操作创建整个迷宫的对象作为参数传递给 CreateMaze 方法，那么可以使用继承来更改迷宫的部分或构建方式。

4. 原型（Prototype）模式：如果 CreateMaze 方法根据各种原型房间、门和墙对象进行参数化，然后将这些原型对象复制并添加到迷宫中，那么可以通过将这些原型对象替换为其他对象来更改迷宫的组成。

5. 单例（Singleton）模式：可以确保每个游戏只有一个迷宫，并且所有游戏对象都可以轻松访问它，而无需使用全局变量或函数。单例模式还可以轻松扩展或替换迷宫，而无需修改现有代码。


## 4.完整代码

```C#
using System.Collections.Generic;

public enum Direction { North, South, East, West };

public abstract class MapSite {
    public abstract void Enter();
}

public class Room : MapSite {
    private Dictionary<Direction, MapSite> _sides = new Dictionary<Direction, MapSite>();
    private int _roomNumber;

    public Room(int roomNo) {
        _roomNumber = roomNo;
    }

    public MapSite GetSide(Direction direction) {
        return _sides[direction];
    }

    public void SetSide(Direction direction, MapSite mapSite) {
        _sides[direction] = mapSite;
    }

    public override void Enter() {
        // Implementation
    }
}

public class Wall : MapSite {
    public Wall() {
    }

    public override void Enter() {
        // Implementation
    }
}

public class Door : MapSite {
    private Room _room1;
    private Room _room2;
    private bool _isOpen;

    public Door(Room room1 = null, Room room2 = null) {
        _room1 = room1;
        _room2 = room2;
    }

    public override void Enter() {
        // Implementation
    }

    public Room OtherSideFrom(Room room) {
        // Implementation
    }
}

public class Maze {
    private Dictionary<int, Room> _rooms = new Dictionary<int, Room>();

    public Maze() {
    }

    public void AddRoom(Room room) {
        _rooms[room.RoomNumber] = room;
    }

    public Room RoomNo(int roomNumber) {
        return _rooms[roomNumber];
    }
}

public class MazeGame {
    public Maze () {
        Maze aMaze = new Maze();
        Room r1 = new Room(1);
        Room r2 = new Room(2);
        Door theDoor = new Door(r1, r2);

        aMaze.AddRoom(r1);
        aMaze.AddRoom(r2);

        r1.SetSide(Direction.North, new Wall());
        r1.SetSide(Direction.East, theDoor);
        r1.SetSide(Direction.South, new Wall());
        r1.SetSide(Direction.West, new Wall());

        r2.SetSide(Direction.North, new Wall());
        r2.SetSide(Direction.East, new Wall());
        r2.SetSide(Direction.South, new Wall());
        r2.SetSide(Direction.West, theDoor);

        return aMaze;
    }
}

```