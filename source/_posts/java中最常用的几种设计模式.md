# Java中最常用的几种设计模式

## 1、单例模式

常见写法：懒汉式、饿汉式、静态代码块、静态内部类、枚举等。推荐采用后3种。

```
//懒汉式
public class Singleton {  
    /* 持有私有静态实例，防止被引用，此处赋值为null，目的是实现延迟加载 */  
    private static Singleton instance = null;  
  
    /* 私有构造方法，防止被实例化 */  
    private Singleton() {}  
  
    /* 1:懒汉式，静态工程方法，创建实例 */  
    public static Singleton getInstance() {  
        if (instance == null) {  
            instance = new Singleton();  
        }  
        return instance;  
    }  
}  
```

```
//静态内部类，推荐使用
public class SingletonInner {  
    /** 
     * 内部类实现单例模式 
     * 延迟加载，减少内存开销   
     */  
    private static class SingletonHolder {  
        private static SingletonInner instance = new SingletonInner();  
    }  
  
    /** 
     * 私有的构造函数 
     */  
    private SingletonInner() {}  
  
    public static SingletonInner getInstance() {  
        return SingletonHolder.instance;  
    }  
  
    protected void method() {  
        System.out.println("SingletonInner");  
    }  
}  
```

其他模式不再赘述。

## 2、工厂模式

- 简单工厂模式 `Simple Factory`（静态工厂）：不利于长生系列产品

- 工厂方式模式 `Facatory Method` : 多形性工厂

- 抽象工厂模式 `Abstract Factory`：又称为工具箱，产生产品族，但不利于产生新的产品

  

  ### 静态工厂

```
public class Factory{ //getClass 产生Sample 一般可使用动态类装载装入类。
    public static Sample creator(int which){ 
        if (which==1)
            return new SampleA();
        else if (which==2)
            return new SampleB();
    }
}
```

目前比较流行的规范是把静态工厂方法命名为`valueOf`或者`getInstance`

```
Integer a=Integer.valueOf(100); //返回取值为100的Integer对象
Calendar cal=Calendar.getInstance(Locale.CHINA); //返回符合中国标准的日历
```

### 工厂方法模式

工厂方法模式是简单工厂模式的进一步抽象化和推广，工厂方法模式里不再只由一个工厂类决定那一个产品类应当被实例化,这个决定被交给抽象工厂的子类去做。
来看下它的组成：

- 抽象工厂角色： 这是工厂方法模式的核心，它与应用程序无关。是具体工厂角色必须实现的接口或者必须继承的父类。在java中它由抽象类或者接口来实现。
- 具体工厂角色：它含有和具体业务逻辑有关的代码。由应用程序调用以创建对应的具体产品的对象
- 抽象产品角色：它是具体产品继承的父类或者是实现的接口。在java中一般有抽象类或者接口来实现。
- 具体产品角色：具体工厂角色所创建的对象就是此角色的实例。在java中由具体的类来实现。

工厂方法模式使用继承自抽象工厂角色的多个子类来代替简单工厂模式中的“上帝类”。正如上面所说，这样便分担了对象承受的压力；而且这样使得结构变得灵活 起来——当有新的产品（即暴发户的汽车）产生时，只要按照抽象产品角色、抽象工厂角色提供的合同来生成，那么就可以被客户使用，而不必去修改任何已有的代 码。可以看出工厂角色的结构也是符合开闭原则的！

```
//抽象产品角色
public interface Moveable {
    void run();
}
//具体产品角色
public class Plane implements Moveable {
    @Override
    public void run() {
        System.out.println("plane....");
    }
}
//具体产品角色
public class Broom implements Moveable {
    @Override
    public void run() {
        System.out.println("broom.....");
    }
}

//抽象工厂
public abstract class VehicleFactory {
    abstract Moveable create();
}
//具体工厂
public class PlaneFactory extends VehicleFactory{
    public Moveable create() {
        return new Plane();
    }
}
//具体工厂
public class BroomFactory extends VehicleFactory{
    public Moveable create() {
        return new Broom();
    }
}
//测试类
public class Test {
    public static void main(String[] args) {
        VehicleFactory factory = new BroomFactory();
        Moveable m = factory.create();
        m.run();
    }
}
```

**简单工厂和工厂方法模式的比较**

工厂方法模式和简单工厂模式在定义上的不同是很明显的。工厂方法模式的核心是一个抽象工厂类,而不像简单工厂模式, 把核心放在一个实类上。工厂方法模式可以允许很多实的工厂类从抽象工厂类继承下来, 从而可以在实际上成为多个简单工厂模式的综合,从而推广了简单工厂模式。 
反过来讲,简单工厂模式是由工厂方法模式退化而来。设想如果我们非常确定一个系统只需要一个实的工厂类, 那么就不妨把抽象工厂类合并到实的工厂类中去。而这样一来,我们就退化到简单工厂模式了。

**抽象工厂模式**

```
//抽象工厂类
public abstract class AbstractFactory {
    public abstract Vehicle createVehicle();
    public abstract Weapon createWeapon();
    public abstract Food createFood();
}
//具体工厂类，其中Food,Vehicle，Weapon是抽象类，
public class DefaultFactory extends AbstractFactory{
    @Override
    public Food createFood() {
        return new Apple();
    }
    @Override
    public Vehicle createVehicle() {
        return new Car();
    }
    @Override
    public Weapon createWeapon() {
        return new AK47();
    }
}
//测试类
public class Test {
    public static void main(String[] args) {
        AbstractFactory f = new DefaultFactory();
        Vehicle v = f.createVehicle();
        v.run();
        Weapon w = f.createWeapon();
        w.shoot();
        Food a = f.createFood();
        a.printName();
    }
}
```

在抽象工厂模式中，抽象产品 (AbstractProduct) 可能是一个或多个，从而构成一个或多个产品族(Product Family)。 在只有一个产品族的情况下，抽象工厂模式实际上退化到工厂方法模式。

**总结**

1. 简单工厂模式是由一个具体的类去创建其他类的实例，父类是相同的，父类是具体的。 
2. 工厂方法模式是有一个抽象的父类定义公共接口，子类负责生成具体的对象，这样做的目的是将类的实例化操作延迟到子类中完成。
3. 抽象工厂模式提供一个创建一系列相关或相互依赖对象的接口，而无须指定他们具体的类。它针对的是有多个产品的等级结构。而工厂方法模式针对的是一个产品的等级结构。

## 3、建造者模式（builder）

基本概念：是一种对象构建的设计模式，它可以将复杂对象的建造过程抽象出来（抽象类别），使这个抽象过程的不同实现方法可以构造出不同表现（属性）的对象。

Builder模式是一步一步创建一个复杂的对象，它允许用户可以只通过指定复杂对象的类型和内容就可以构建它们。用户不知道内部的具体构建细节。Builder模式是非常类似抽象工厂模式，细微的区别大概只有在反复使用中才能体会到。

swagger中一个对builder的经典运用：

```

public class ModelBuilder {
    private String id;
    private String name;
    private String qualifiedType;
    private String description;
    private String baseModel;
    private String discriminator;
    private ResolvedType modelType;
    private String example;
    private Map<String, ModelProperty> properties = Maps.newHashMap();
    private List<String> subTypes = Lists.newArrayList();

    public ModelBuilder() {
    }

    public ModelBuilder id(String id) {
        this.id = (String)BuilderDefaults.defaultIfAbsent(id, this.id);
        return this;
    }

    public ModelBuilder name(String name) {
        this.name = (String)BuilderDefaults.defaultIfAbsent(name, this.name);
        return this;
    }

    public ModelBuilder qualifiedType(String qualifiedType) {
        this.qualifiedType = (String)BuilderDefaults.defaultIfAbsent(qualifiedType, this.qualifiedType);
        return this;
    }

    public ModelBuilder properties(Map<String, ModelProperty> properties) {
        this.properties.putAll(BuilderDefaults.nullToEmptyMap(properties));
        return this;
    }

    public ModelBuilder description(String description) {
        this.description = (String)BuilderDefaults.defaultIfAbsent(description, this.description);
        return this;
    }

    public ModelBuilder baseModel(String baseModel) {
        this.baseModel = (String)BuilderDefaults.defaultIfAbsent(baseModel, this.baseModel);
        return this;
    }

    public ModelBuilder discriminator(String discriminator) {
        this.discriminator = (String)BuilderDefaults.defaultIfAbsent(discriminator, this.discriminator);
        return this;
    }

    public ModelBuilder subTypes(List<String> subTypes) {
        if (subTypes != null) {
            this.subTypes.addAll(subTypes);
        }

        return this;
    }

    public ModelBuilder example(String example) {
        this.example = (String)BuilderDefaults.defaultIfAbsent(example, this.example);
        return this;
    }

    public ModelBuilder type(ResolvedType modelType) {
        this.modelType = (ResolvedType)BuilderDefaults.defaultIfAbsent(modelType, this.modelType);
        return this;
    }

    public Model build() {
        return new Model(this.id, this.name, this.modelType, this.qualifiedType, this.properties, this.description, this.baseModel, this.discriminator, this.subTypes, this.example);
    }
}
```



## 4、观察者模式

观察者模式定义了一种一对多的依赖关系，让多个观察者对象同时监听某一主题对象。这个主题对象在状态发生变化时，会通知所有观察者对象，使它们能够自动更新自己。观察者模式又叫发布-订阅(Publish/Subscribe)模式。

- Subject类：它把所有对观察者对象的引用保存在一个聚集里，每个主题都可以有任何数量的观察着。抽象主题提供一个接口，可以增加和删除观察着对象。
- Observer类：抽象观察者，为所有的具体观察者定义一个接口，在得到主题的通知时更新自己。
- ConcreteSubject类：具体主题，将有关状态存入具体观察者对象；在具体主题的内部状态改变时，给所有登记过的观察者发出通知。
- ConcreteObserver类：具体观察者，实现抽象观察者角色所要求的更新接口，以便使本身的状态与主题的状态相协调。

观察者模式何时适用？

- 当一个抽象模型有两个方面，其中一个方面依赖于另一方面。将这二者封装在独立的对象中可以使他们各自独立地改变和复用。
- 当对一个对象的改变需要同时改变其它对象，而不知道具体由多少对象有待改变。
- 当一个对象必须通知其他对象，而它又不能假定其他对象是谁，换言之，你不希望这些对象是紧密耦合的。让耦合的双方都依赖于抽象，而不是依赖于具体。

具体的代码就不贴了，spring中的ApplicationListener 和ApplicationEvent 就是典型的观察者模式的应用。

```
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {

	/**
	 * Handle an application event.
	 * @param event the event to respond to
	 */
	void onApplicationEvent(E event);

}
```

## 5、适配器模式（Adapter）

适配器模式把一个类的接口变换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作。

**适配器模式的结构**
适配器模式有`类的适配器模式`和`对象的适配器模式`两种不同的形式。

类适配器：

```
public interface Target {
    /**
     * 这是源类Adaptee也有的方法
     */
    public void sampleOperation1(); 
    /**
     * 这是源类Adapteee没有的方法
     */
    public void sampleOperation2(); 
}
```

```
public class Adaptee {
    public void sampleOperation1(){}
}
```

```
public class Adapter extends Adaptee implements Target {
    /**
     * 由于源类Adaptee没有方法sampleOperation2()
     * 因此适配器补充上这个方法
     */
    @Override
    public void sampleOperation2() {
        //写相关的代码
    }
}
```

对象适配器模式：

```
public interface Target {
    /**
     * 这是源类Adaptee也有的方法
     */
    public void sampleOperation1(); 
    /**
     * 这是源类Adapteee没有的方法
     */
    public void sampleOperation2(); 
}

public class Adaptee {
    public void sampleOperation1(){}
}
```

适配器：

```
public class Adapter {
    private Adaptee adaptee;
    public Adapter(Adaptee adaptee){
        this.adaptee = adaptee;
    }
    /**
     * 源类Adaptee有方法sampleOperation1
     * 因此适配器类直接委派即可
     */
    public void sampleOperation1(){
        this.adaptee.sampleOperation1();
    }
    /**
     * 源类Adaptee没有方法sampleOperation2
     * 因此由适配器类需要补充此方法
     */
    public void sampleOperation2(){
        //写相关的代码
    }
}
```

**类适配器和对象适配器的权衡**

- **类适配器**使用对象继承的方式，是静态的定义方式；而**对象适配器**使用对象组合的方式，是动态组合的方式。
- 对于**类适配器**由于适配器直接继承了Adaptee，使得适配器不能和Adaptee的子类一起工作，因为继承是静态的关系，当适配器继承了Adaptee后，就不可能再去处理  Adaptee的子类了。
- 对于**对象适配器**一个适配器可以把多种不同的源适配到同一个目标。换言之，同一个适配器可以把源类和它的子类都适配到目标接口。因为对象适配器采用的是对象组合的关系，只要对象类型正确，是不是子类都无所谓。
- 对于**类适配器**适配器可以重定义Adaptee的部分行为，相当于子类覆盖父类的部分实现方法。
- 对于**对象适配器**要重定义Adaptee的行为比较困难，这种情况下，需要定义Adaptee的子类来实现重定义，然后让适配器组合子类。虽然重定义Adaptee的行为比较困难，但是想要增加一些新的行为则方便的很，而且新增加的行为可同时适用于所有的源。
- 对于**类适配器**，仅仅引入了一个对象，并不需要额外的引用来间接得到Adaptee。
- 对于**对象适配器**，需要额外的引用来间接得到Adaptee。

建议尽量使用对象适配器的实现方式，多用合成或聚合、少用继承。当然，具体问题具体分析，根据需要来选用实现方式，最适合的才是最好的。

**适配器模式的优点**

- 更好的复用性：系统需要使用现有的类，而此类的接口不符合系统的需要。那么通过适配器模式就可以让这些功能得到更好的复用。
- 更好的扩展性：在实现适配器功能的时候，可以调用自己开发的功能，从而自然地扩展系统的功能。

**适配器模式的缺点**

　　过多的使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是A接口，其实内部被适配成了B接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。



# 6、代理模式

为其他对象提供一种代理以控制对这个对象的访问。也可以说，在出发点到目的地之间有一道中间层，意为代理。

**为什么要使用**

- 授权机制不同级别的用户对同一对象拥有不同的访问权利，如在论坛系统中，就使用Proxy进行授权机制控制，访问论坛有两种人：注册用户和游客(未注册用户)，论坛就通过类似ForumProxy这样的代理来控制这两种用户对论坛的访问权限。
- 某个客户端不能直接操作到某个对象，但又必须和那个对象有所互动。

举例两个具体情况：

- 如果那个对象是一个是很大的图片，需要花费很长时间才能显示出来，那么当这个图片包含在文档中时，使用编辑器或浏览器打开这个文档，打开文档必须很迅速，不能等待大图片处理完成，这时需要做个图片Proxy来代替真正的图片。
- 如果那个对象在Internet的某个远端服务器上，直接操作这个对象因为网络速度原因可能比较慢，那我们可以先用Proxy来代替那个对象。

总之原则是，对于开销很大的对象，只有在使用它时才创建，这个原则可以为我们节省很多宝贵的Java内存。所以，有些人认为Java耗费资源内存，我以为这和程序编制思路也有一定的关系。

**流程图**

![代理模式](https://images2015.cnblogs.com/blog/331079/201606/331079-20160629172552327-327363457.jpg)

# 7、装饰模式

基本概念：装饰模式(Decorator)，动态地给一个对象添加一些额外的职责，就增加功能来说，装饰模式比生成子类更为灵活。

![装饰模式](https://images2015.cnblogs.com/blog/331079/201606/331079-20160629172628015-2044014507.png)

上图是Decorator 模式的结构图,让我们可以进行更方便的描述:

- `Component`是定义一个对象接口，可以给这些对象动态地添加职责。
- `ConcreteComponent`是定义了一个具体的对象，也可以给这个对象添加一些职责。

Decorator是装饰抽象类，继承了Component,从外类来扩展Component类的功能，但对于Component来说，是无需知道Decorator存在的。ConcreteDecorator就是具体的装饰对象，起到给Component添加职责的功能。

**总结**

`Decorator`模式有以下的优缺点：

- 比静态继承更灵活与对象的静态继承相比，Decorator模式提供了更加灵活的向对象添加职责的方式，可以使用添加和分离的方法，用装饰在运行时刻增加和删除职责。使用继承机制增加职责需要创建一个新的子类，如果需要为原来所有的子类都添加功能的话，每个子类都需要重写，增加系统的复杂度，此外可以为一个特定的Component类提供多个Decorator，这种混合匹配是适用继承很难做到的。
- 避免在层次结构高层的类有太多的特征，Decorator模式提供了一种“即用即付”的方法来添加职责，他并不试图在一个复杂的可订制的类中支持所有可预见的特征，相反可以定义一个简单的类，并且用Decorator类给他逐渐的添加功能，从简单的部件组合出复杂的功能。
- Decorator 与它的Component不一样Decorator是一个透明的包装，如果我们从对象标识的观点出发，一个被装饰了的组件与这个组件是有差别的，因此使用装饰时不应该以来对象标识。
- 产生许多小对象，采用Decorator模式进行系统设计往往会产生许多看上去类似的小对象，这些对象仅仅在他们相互连接的方式上有所不同。