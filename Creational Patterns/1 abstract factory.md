# 抽象工厂（Abstract Factory）

MazeFactory 是一个抽象工厂，它提供了创建迷宫、墙、房间和门的方法。EnchantedMazeFactory 和 BombedMazeFactory 是 MazeFactory 的子类，它们分别覆盖了一些方法以创建特定类型的迷宫组件。

MazeGame 类包含一个 createMaze 方法，它接受一个 MazeFactory 参数。通过使用不同类型的 MazeFactory，可以创建不同类型的迷宫。例如，可以使用 BombedMazeFactory 创建一个含有炸弹的迷宫，或者使用 EnchantedMazeFactory 创建一个附魔迷宫。

```c#
public abstract class MazeFactory
{
    public virtual Maze MakeMaze()
    {
        return new Maze();
    }

    public virtual Wall MakeWall()
    {
        return new Wall();
    }

    public virtual Room MakeRoom(int n)
    {
        return new Room(n);
    }

    public virtual Door MakeDoor(Room r1, Room r2)
    {
        return new Door(r1, r2);
    }
}
```

在[之前准备](./0%20prepare.md)中，CreateMaze 方法，对类名进行了硬编码，直接在方法中创建房间，墙壁和门的实例，这使得该方法无法用于创建不同组件的迷宫。

现在我们以 MazeFactory 为参数，生成一个新版本的 CreateMaze 方法。

```C#
public class MazeGame
{
    public Maze CreateMaze(MazeFactory factory)
    {
        Maze aMaze = factory.MakeMaze();
        Room r1 = factory.MakeRoom(1);
        Room r2 = factory.MakeRoom(2);
        Door aDoor = factory.MakeDoor(r1, r2);

        aMaze.AddRoom(r1);
        aMaze.AddRoom(r2);

        r1.SetSide(Direction.North, factory.MakeWall());
        r1.SetSide(Direction.East, aDoor);
        r1.SetSide(Direction.South, factory.MakeWall());
        r1.SetSide(Direction.West, factory.MakeWall());

        r2.SetSide(Direction.North, factory.MakeWall());
        r2.SetSide(Direction.East, factory.MakeWall());
        r2.SetSide(Direction.South, factory.MakeWall());
        r2.SetSide(Direction.West, aDoor);

        return aMaze;
    }
}
```
 
创建两个 MazeFactory 的子类，EnchantedMazeFactory 和 BombedMazeFactory。它们分别用于创建具有不同特性的迷宫。

EnchantedMazeFactory 用于创建具有魔法特性的迷宫。这些迷宫中的房间和门可能需要特殊的魔法来解锁或激活。为此，EnchantedMazeFactory 重写了 MazeFactory 中的部分方法，以便返回具有魔法属性的房间、门等组件。

例如，在此工厂类中重写的 MakeRoom 和 MakeDoor 方法会创建 EnchantedRoom 和 DoorNeedingSpell 实例，它们都具有与魔法相关的特性。

Enchanted 使着魔，Spell 咒语。

```c#
public class EnchantedMazeFactory : MazeFactory
{
    public override Room MakeRoom(int n)
    {
        return new EnchantedRoom(n, CastSpell());
    }

    public override Door MakeDoor(Room r1, Room r2)
    {
        return new DoorNeedingSpell(r1, r2);
    }

    protected Spell CastSpell()
    {
        return new Spell();
    }
}
```

BombedMazeFactory 用于创建可能包含炸弹的迷宫。这些迷宫中的房间可能有炸弹，而墙壁可能被炸毁。为了实现这些特性，BombedMazeFactory 重写了 MazeFactory 中的部分方法，以便返回具有炸弹和破坏属性的房间、墙等组件。

例如，在此工厂类中重写的 MakeWall 和 MakeRoom 方法会创建 BombedWall 和 RoomWithABomb 实例，它们具有与炸弹和破坏相关的特性。

```C#
public class BombedMazeFactory : MazeFactory
{
    public override Wall MakeWall()
    {
        return new BombedWall();
    }

    public override Room MakeRoom(int n)
    {
        return new RoomWithABomb(n);
    }
}
```

如果想使用 BombedMazeFactory 或者 EnchantedMazeFactory 来构建包含炸弹的迷宫或者具有魔法特性的迷宫。

可以这样实现：

```c#
// 使用 BombedMazeFactory 构建包含炸弹的迷宫
MazeGame game = new MazeGame();
BombedMazeFactory bombedFactory = new BombedMazeFactory();
Maze bombedMaze = game.CreateMaze(bombedFactory);

// 使用 EnchantedMazeFactory 构建具有魔法特性的迷宫
EnchantedMazeFactory enchantedFactory = new EnchantedMazeFactory();
Maze enchantedMaze = game.CreateMaze(enchantedFactory);
```

MazeFactory 只是一个包含工厂方法的集合，这是实现抽象工厂模式的最常见方法。请注意，MazeFactory 不是抽象类，因此它既充当抽象工厂，也充当具体工厂。这是抽象工厂模式在简单应用中的另一种常见实现。

MazeFactory 是一个完全由工厂方法组成的具体类，通过创建子类并重写需要更改的操作，可以很容易地创建新的 MazeFactory。

## 完整代码

```c#

public abstract class MazeFactory
{
    public virtual Maze MakeMaze()
    {
        return new Maze();
    }

    public virtual Wall MakeWall()
    {
        return new Wall();
    }

    public virtual Room MakeRoom(int n)
    {
        return new Room(n);
    }

    public virtual Door MakeDoor(Room r1, Room r2)
    {
        return new Door(r1, r2);
    }
}

public class EnchantedMazeFactory : MazeFactory
{
    public override Room MakeRoom(int n)
    {
        return new EnchantedRoom(n, CastSpell());
    }

    public override Door MakeDoor(Room r1, Room r2)
    {
        return new DoorNeedingSpell(r1, r2);
    }

    protected Spell CastSpell()
    {
        return new Spell();
    }
}

public class BombedMazeFactory : MazeFactory
{
    public override Wall MakeWall()
    {
        return new BombedWall();
    }

    public override Room MakeRoom(int n)
    {
        return new RoomWithABomb(n);
    }
}

public class MazeGame
{
    public Maze CreateMaze(MazeFactory factory)
    {
        Maze aMaze = factory.MakeMaze();
        Room r1 = factory.MakeRoom(1);
        Room r2 = factory.MakeRoom(2);
        Door aDoor = factory.MakeDoor(r1, r2);

        aMaze.AddRoom(r1);
        aMaze.AddRoom(r2);

        r1.SetSide(Direction.North, factory.MakeWall());
        r1.SetSide(Direction.East, aDoor);
        r1.SetSide(Direction.South, factory.MakeWall());
        r1.SetSide(Direction.West, factory.MakeWall());

        r2.SetSide(Direction.North, factory.MakeWall());
        r2.SetSide(Direction.East, factory.MakeWall());
        r2.SetSide(Direction.South, factory.MakeWall());
        r2.SetSide(Direction.West, aDoor);

        return aMaze;
    }
}

public class EnchantedRoom : Room
{
    private Spell _spell;

    public EnchantedRoom(int n, Spell spell) : base(n)
    {
        _spell = spell;
    }
}

public class DoorNeedingSpell : Door
{
    public DoorNeedingSpell(Room r1, Room r2) : base(r1, r2)
    {
    }
}

public class Spell
{
    // 在这里实现魔法相关的功能
}

public class BombedWall : Wall
{
    // 在这里实现被炸毁的墙的功能，例如受损的外观
}

public class RoomWithABomb : Room
{
    private bool _bomb;
    private bool _bombExploded;

    public RoomWithABomb(int n) : base(n)
    {
        _bomb = false;
        _bombExploded = false;
    }

    public void SetBomb(bool hasBomb)
    {
        _bomb = hasBomb;
    }

    public bool HasBomb()
    {
        return _bomb;
    }

    public void ExplodeBomb()
    {
        if (_bomb)
        {
            _bombExploded = true;
            // 在这里实现炸弹爆炸的效果，例如破坏周围的墙壁
        }
    }

    public bool IsBombExploded()
    {
        return _bombExploded;
    }
}


```