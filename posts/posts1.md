# 谈谈Java枚举enum

“枚举（enum），是 Java 1.5 时引入的关键字，它表示一种特殊类型的类，继承自 java.lang.Enum。”

“我们来新建一个枚举 PlayerType。”

```java
public enum PlayerType {
    TENNIS,
    FOOTBALL,
    BASKETBALL
}
```

看一下反编译后的字节码，你就明白了。

```java
public final class PlayerType extends Enum
{

    public static PlayerType[] values()
    {
        return (PlayerType[])$VALUES.clone();
    }

    public static PlayerType valueOf(String name)
    {
        return (PlayerType)Enum.valueOf(com/cmower/baeldung/enum1/PlayerType, name);
    }

    private PlayerType(String s, int i)
    {
        super(s, i);
    }

    public static final PlayerType TENNIS;
    public static final PlayerType FOOTBALL;
    public static final PlayerType BASKETBALL;
    private static final PlayerType $VALUES[];

    static 
    {
        TENNIS = new PlayerType("TENNIS", 0);
        FOOTBALL = new PlayerType("FOOTBALL", 1);
        BASKETBALL = new PlayerType("BASKETBALL", 2);
        $VALUES = (new PlayerType[] {
            TENNIS, FOOTBALL, BASKETBALL
        });
    }
}
```

“既然枚举是一种特殊的类，那它其实是可以定义在一个类的内部的，这样它的作用域就可以限定于这个外部类中使用。”

```Java
public class Player {
    private PlayerType type;
    public enum PlayerType {
        TENNIS,
        FOOTBALL,
        BASKETBALL
    }
    
    public boolean isBasketballPlayer() {
      return getType() == PlayerType.BASKETBALL;
    }

    public PlayerType getType() {
        return type;
    }

    public void setType(PlayerType type) {
        this.type = type;
    }
}
```

“如果枚举中需要包含更多信息的话，可以为其添加一些字段，比如下面示例中的 `name`，此时需要为枚举添加一个带参的构造方法，这样就可以在定义枚举时添加对应的名称了。”

```java
public enum PlayerType {
    TENNIS("网球"),
    FOOTBALL("足球"),
    BASKETBALL("篮球");

    private String name;

    PlayerType(String name) {
        this.name = name;
    }
}
```

默认情况下，对枚举常量调用`toString()`会返回和`name()`一样的字符串。但是，`toString()`可以被覆写，而`name()`则不行。

覆写`toString()`的目的是在输出时更有可读性。

最后，枚举类可以应用在`switch`语句中。因为枚举类天生具有类型信息和有限个枚举常量，所以比`int`、`String`类型更适合用在`switch`语句中：

```Java
switch (playerType) {
    case TENNIS:
        return "网球运动员费德勒";
    case FOOTBALL:
        return "足球运动员C罗";
    case BASKETBALL:
        return "篮球运动员詹姆斯";
    case UNKNOWN:
        throw new IllegalArgumentException("未知");
    default:
        throw new IllegalArgumentException(
                "运动员类型: " + playerType);

}
```

一些方法：

`name()`

返回常量名，例如：

```java
String s = Weekday.SUN.name(); // "SUN"
```

`ordinal()`

返回定义的常量的顺序，从0开始计数，例如：

```java
int n = Weekday.MON.ordinal(); // 1
```

**注意**	改变枚举常量定义的顺序就会导致`ordinal()`返回值发生变化。

（完）

------

补充：

可以Google一下EnumSet`和`EnumMap`