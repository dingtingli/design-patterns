# Factory Method（工厂方法）


## 简单示例

“工厂方法（Factory Method）”，定义一个接口用于创建对象，让子类子类决定实例化的类是什么。

```c#
public abstract class Product
{
}

public class ConcreteProduct : Product
{
}

public abstract class Creator
{
    public abstract Product FactoryMethod();
}

public class ConcreteCreator : Creator
{
    public override Product FactoryMethod()
    {
        return new ConcreteProduct();
    }
}

public class Client
{
    public void Main()
    {
        Creator creator = new ConcreteCreator();
        Product product = creator.FactoryMethod();
    }
}
```

在这些代码中，Product 是一个抽象的产品，ConcreteProduct 是其具体实现。

Creator 是创建产品的抽象类，它定义了一个抽象的工厂方法 factoryMethod，这个方法返回抽象产品 Product。

ConcreteCreator 是 Creator 的一个具体实现，它重写 factoryMethod 方法以返回 ConcreteProduct。

它将具体产品的创建代码从客户端代码中分离出来，使得客户端代码不需要知道具体产品的类名，只需要知道对应的工厂即可。

主要的缺点是每次添加新产品时，都需要添加一个具体的产品类和一个对应的具体工厂类，增加了代码的复杂性。


## 迷宫

工厂方法（Factory Method）设计模式在创建对象时使用工厂方法而不是直接使用构造函数，这样可以使代码更加灵活，容易进行扩展和修改。

MazeGame类是所有迷宫游戏的基类，它有一些工厂方法用来创建迷宫的各个部分（迷宫、房间、墙、门）。这些方法在基类中有默认实现，但在子类中可以被重写，以提供不同类型的迷宫、房间、墙和门。

```c#
public class MazeGame {
    public Maze CreateMaze() {
        Maze aMaze = MakeMaze();
        Room r1 = MakeRoom(1);
        Room r2 = MakeRoom(2);
        Door theDoor = MakeDoor(r1, r2);
        aMaze.AddRoom(r1);
        aMaze.AddRoom(r2);
        r1.SetSide(Direction.North, MakeWall());
        r1.SetSide(Direction.East, theDoor);
        r1.SetSide(Direction.South, MakeWall());
        r1.SetSide(Direction.West, MakeWall());
        r2.SetSide(Direction.North, MakeWall());
        r2.SetSide(Direction.East, MakeWall());
        r2.SetSide(Direction.South, MakeWall());
        r2.SetSide(Direction.West, theDoor);
        return aMaze;
    }

    protected virtual Maze MakeMaze() {
        return new Maze();
    }

    protected virtual Room MakeRoom(int n) {
        return new Room(n);
    }

    protected virtual Wall MakeWall() {
        return new Wall();
    }

    protected virtual Door MakeDoor(Room r1, Room r2) {
        return new Door(r1, r2);
    }
}
```

我们看到了MazeGame类，它用于创建迷宫游戏。这个类有一个名为CreateMaze的方法，它使用一些工厂方法（MakeMaze，MakeRoom，MakeWall，MakeDoor）来创建迷宫游戏的各个部分。这些工厂方法在MazeGame类中有默认的实现，但是在其子类中可以被覆盖以创建不同类型的迷宫游戏。

BombedMazeGame类是MazeGame的子类，它覆盖了MakeWall和MakeRoom方法，使得它们分别返回BombedWall和RoomWithABomb对象。这样，当我们在BombedMazeGame对象上调用CreateMaze方法时，它将创建一个包含BombedWall和RoomWithABomb的迷宫。

```c#
public class BombedMazeGame : MazeGame {
    protected override Wall MakeWall() {
        return new BombedWall();
    }

    protected override Room MakeRoom(int n) {
        return new RoomWithABomb(n);
    }
}
```

EnchantedMazeGame类，它也是MazeGame的子类。这个类覆盖了MakeRoom和MakeDoor方法，使得它们分别返回EnchantedRoom和DoorNeedingSpell对象。这样，当我们在EnchantedMazeGame对象上调用CreateMaze方法时，它将创建一个包含EnchantedRoom和DoorNeedingSpell的迷宫。

```c#
public class EnchantedMazeGame : MazeGame {
    protected override Room MakeRoom(int n) {
        return new EnchantedRoom(n, CastSpell());
    }

    protected override Door MakeDoor(Room r1, Room r2) {
        return new DoorNeedingSpell(r1, r2);
    }

    private Spell CastSpell() {
        // Implement spell casting here
        return new Spell();
    }
}

```

通过覆盖工厂方法，我们可以创建出有着不同行为的对象，而这一切都是在不修改已有代码的情况下实现的。这种方式的灵活性使得我们可以在不改变原有代码的情况下添加新的功能，这是一种遵循“开放-封闭原则”的设计方法。

```c#
// 创建一个普通的迷宫游戏
MazeGame regularGame = new MazeGame();
Maze regularMaze = regularGame.CreateMaze();

// 创建一个含有炸弹的迷宫游戏
BombedMazeGame bombedGame = new BombedMazeGame();
Maze bombedMaze = bombedGame.CreateMaze();

// 创建一个魔法的迷宫游戏
EnchantedMazeGame enchantedGame = new EnchantedMazeGame();
Maze enchantedMaze = enchantedGame.CreateMaze();
```

## 完整代码

```c#
public class MazeGame {
    public Maze CreateMaze() {
        Maze aMaze = MakeMaze();
        Room r1 = MakeRoom(1);
        Room r2 = MakeRoom(2);
        Door theDoor = MakeDoor(r1, r2);
        aMaze.AddRoom(r1);
        aMaze.AddRoom(r2);
        r1.SetSide(Direction.North, MakeWall());
        r1.SetSide(Direction.East, theDoor);
        r1.SetSide(Direction.South, MakeWall());
        r1.SetSide(Direction.West, MakeWall());
        r2.SetSide(Direction.North, MakeWall());
        r2.SetSide(Direction.East, MakeWall());
        r2.SetSide(Direction.South, MakeWall());
        r2.SetSide(Direction.West, theDoor);
        return aMaze;
    }

    protected virtual Maze MakeMaze() {
        return new Maze();
    }

    protected virtual Room MakeRoom(int n) {
        return new Room(n);
    }

    protected virtual Wall MakeWall() {
        return new Wall();
    }

    protected virtual Door MakeDoor(Room r1, Room r2) {
        return new Door(r1, r2);
    }
}

public class BombedMazeGame : MazeGame {
    protected override Wall MakeWall() {
        return new BombedWall();
    }

    protected override Room MakeRoom(int n) {
        return new RoomWithABomb(n);
    }
}

public class EnchantedMazeGame : MazeGame {
    protected override Room MakeRoom(int n) {
        return new EnchantedRoom(n, CastSpell());
    }

    protected override Door MakeDoor(Room r1, Room r2) {
        return new DoorNeedingSpell(r1, r2);
    }

    private Spell CastSpell() {
        // Implement spell casting here
        return new Spell();
    }
}

```