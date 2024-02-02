# 当参数足够多时你会Build模式吗！

静态工厂和构造器有个共同的局限性：它们都不能很好地扩展到大量的可选参数。

比如用一个类表示包装食品外面显示的营养成分标签。这些标签中有几个域是必须的：每份的含量、每罐的含量及每份的卡路里。还有超过20个的可选域：总脂肪量、饱和脂肪量、胆固醇、钠，等等。

对于这样的类，应该采用哪种构造器或静态工厂来编写呢？

#### 重叠构造器模式

提供的第一个构造器只有必要的参数，第二个构造器有一个可选参数，第三个有俩个可选参数，以此类推，最后一个包含所有可选参数。

```Java
public class NutritionFacts {
  private final int servingSize;    //(mL)   required
  private final int servings;       //(per)  required
  private final int calories;       //(per)  optional
  private final int fat;            //(g)    optional
  private final int sodium;         //(mg)   optional
  private final int carbohydrate;   //(g)    optional

  public NutritionFacts(int servingSize, int servings) {
    this(servingSize, servings, 0);
  }

  public NutritionFacts(int servingSize, int servings, int calories) {
    this(servingSize, servings, calories, 0);
  }

  public NutritionFacts(int servingSize, int servings, int calories, int fat) {
    this(servingSize, servings, calories, fat, 0);
  }

  public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
    this(servingSize, servings, calories, fat, sodium, 0);
  }

  public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
    this.servingSize = servingSize;
    this.servings = servings;
    this.calories = calories;
    this.fat = fat;
    this.sodium = sodium;
    this.carbohydrate = carbohydrate;
  }
}

```

当你想要创建实例的时候，就利用参数化列表最短的构造器，但该列表中包含了要设置的所有参数：

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 35, 27);
```

重叠构造器模式可行，但是当有许多参数的时候，客户端代码会很难编写，并且仍然难以阅读。（比如不小心颠倒参数的顺序）

#### JavaBeans 模式

先调用一个无参构造器来创建对象，然后调用setter方法来设置每个必要的参数，以及每个相关的可选参数。

```java
public class NutritionFacts {
  private int servingSize = -1;    //(mL)   required
  private int servings = -1;       //(per)  required
  private int calories = 0;       //(per)  optional
  private int fat = 0;            //(g)    optional
  private int sodium = 0;         //(mg)   optional
  private int carbohydrate = 0;   //(g)    optional

  public NutritionFacts() {}
  
  //setter
  public void setServingSize(int servingSize) {this.servingSize = servingSize;}
  public void setServings(int servings) {this.servings = servings;}
  public void setCalories(int calories) {this.calories = calories;}
  public void setFat(int fat) {this.fat = fat;}
  public void setSodium(int sodium) {this.sodium = sodium;}
  public void setCarbohydrate(int carbohydrate) {this.carbohydrate = carbohydrate;}
}
```

这种模式弥补了重叠构造器模式的不足。创建实例容易，生成的代码也易读。

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

遗憾的是JavaBeans模式有非常严重的缺点。

在构造过程中JavaBeans可能处于不一致的状态。
JavaBeans模式使得把类做成不可变的可能性不复存在。

#### Builder模式的一种

不直接生成想要的对象，而是让客户端利用所有必要的参数调用构造器（或者静态工厂），得到一个builder对象。然后客户端在builder对象上调用类似setter方法，来设置每个相关的可选参数。

最后，客户端调用无参的build方法来生成通常是不可变的对象。这个builder通常是它的构建类的静态成员类。

如下：

```java
public class NutritionFacts {

  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;

  public static class Builder {

    // Required parameters
    private final int servingSize;
    private final int servings;

    // Optional parameters
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0; 
    private int carbohydrate = 0;

    public Builder(int servingSize, int servings) {
      this.servingSize = servingSize;
      this.servings = servings;
    }

    public Builder calories(int calories) {
      this.calories = calories;
      return this;
    }

    public Builder fat(int fat) {
      this.fat = fat;
      return this; 
    }

    public Builder sodium(int sodium) {
      this.sodium = sodium;
      return this;
    }

    public Builder carbohydrate(int carbohydrate) {
      this.carbohydrate = carbohydrate;
      return this;
    }

    public NutritionFacts build() {
      return new NutritionFacts(this);
    }
  }

  private NutritionFacts(Builder builder) {
    servingSize = builder.servingSize;
    servings = builder.servings;
    calories = builder.calories;
    fat = builder.fat;
    sodium = builder.sodium;
    carbohydrate = builder.carbohydrate;
  }

}
```

注意`NutritionFacts`是不可变的，所有的默认参数值都单独放在一个地方。builder的设置方法返回builder本身，以便把调用链接起来，得到一个流式API。

客户端代码：

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodeium(35).fat(27).build();
```

注意要进行有效性检查。

**Builder模式也适用于类层次结构。**

使用平行层次的builder时，各自嵌套在相应的类中。抽象类有抽象的builder，具体类有具体的builder。

假设用类层次根部的一个抽象表示各种各样的披萨:

```java
public abstract class Pizza {
  public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}

  final Set<Topping> toppings;

  abstract static class Builder<T extends Builder<T>> {
    EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

    public T addTopping(Topping topping) {
      toppings.add(Objects.requireNonNull(topping));
      return self();
    }

    abstract Pizza build();

    protected abstract T self();
  }

  Pizza(Builder<?> builder) {
    toppings = builder.toppings.clone();
  }
}
```

```Java
import java.util.Objects;

public class NyPizza extends Pizza {
  public enum Size {SMALL, MEDIUM, LARGE}

  private final Size size;

  public static class Builder extends Pizza.Builder<Builder> {
    private final Size size;

    public Builder(Size size) {
      this.size = Objects.requireNonNull(size);
    }
    
    @Override
    Pizza build() {
      return new NyPizza(this);
    }

    @Override
    protected Builder self() {
      return this;
    }
  }

  private NyPizza(Builder builder) {
    super(builder);
    size = builder.size;
  }
}

```

