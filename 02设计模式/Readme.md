# 九、设计模式

## 目录

## 1、设计模式

### 1.1设计模式的五个原则

* S-单一职责原则：

> 一个模块负责一个功能
>
> 一个类负责一个业务
>
> 一个API去实现一个功能
>
> 不同的功能分隔开。一个类如果需要变化，最好只能有一个维度的原因。

* O-开放封闭原则：

>对什么开放：对扩展开放
>
> 
>
>扩展：新功能à可以（提倡）去增加代码（增加一个类）
>
>项目不断迭代（增加新的需求）
>
> 
>
>对什么封闭：对修改封闭
>
>不建议去修改代码 （尤其是一些底层的API）
>
>牵一发而动全身。

* L-Liskov原则：

> 李氏替换原则：任何一个基类可以出现的地方，子类一定可以出现。

* I-接口隔离原则：

> 接口：interface （功能）
>
> 隔离：分开
>
> 不同功能的接口，最好放在不同的interface里。
>
> 一个接口里最好不要集成太多的功能，
>
> 否则用户去实现该接口会比较麻烦。

* D-依赖倒置原则：

> 依赖 
>
> 具体依赖于抽象，而非抽象依赖于具体。
>
> 先去设计抽象的DAO  （接口）

### 1.2常见的设计模式

#### 1.2.1单例模式(Singleton Pattern)

**概念：**

> 何为单例？单一实例。全局只有一个。
>
> 哪些场合需要用到单例？
>
> Servlet
>
> ServletContext

**特点：**

1、单例类只能有一个实例。

2、单例类必须自己创建自己的唯一实例。→构造方法私有

3、单例类必须给所有其他对象提供这一实例。 

**实现：**

*线程安全的懒加载——静态内部类的实现*

``` java
public class Singleton5 {
    private Singleton5(){}

    static class Inner{
        static Singleton5 singleton5 = new Singleton5();

        public static Singleton5 provideSingleton5(){
            return singleton5;
        }
    }

    public static Singleton5 getInstance(){
        return Inner.provideSingleton5();
    }
}
```

#### 1.2.2工厂模式(Factory Pattern)

**概念：**

>生成产品的工厂，生产对象
>
>在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。 
>
>Animal animal = AnimalFactory.create（）；
>
>去生产实例，工厂要提供create方法。
>
>通过工厂可以把实例的实例化细节隐藏起来。

**简单工厂：**

给我需要使用的类的名字。简单工厂返回一个这种类实例。

**工厂方法模式：**

工厂方法模式，创建一个工厂接口和创建多个工厂实现类 

完全遵从**开闭原则**，不对任何已有代码进行修改，直接新增一个类。

```java
public class AnimalFactory {

    //提供一个返回值为animal的方法
    public Animal create(String name){
        Animal animal = null;
        if ("bull".equals(name)){
            animal = new Bull();
        }else if ("pig".equals(name)){
            animal = new Pig();
        }else if (name.equals("rabbit")){
            animal = new Rabbit();
        }else if ("mouse".equals(name)){
            animal = new Mouse();
        }

        return animal;
    }
}
```

```java
public interface Animal {
}
```

```java
public class Bull implements Animal{
}
```

```java
public class Mouse implements Animal{
}
```

```java
public interface CarFactory {

    public Car createCar();
}
```

```java
public class AudiCarFactory implements CarFactory{
    @Override
    public Car createCar() {
        return new AudiCar();
    }
}
```

```java
//提供静态方法获得实例的工厂，称之为静态工厂
public class DreamOneCarFactory implements CarFactory{
    @Override
    public Car createCar() {
        return null;
    }

    public static Car staticFactoryCreate(){
        return new DreamOneCar();
    }
}
```

#### 1.2.3代理模式(Proxy Pattern)(Key)

![](img/factory1.png)

##### 1.2.3.1静态代理

委托类对象：房东

代理类对象：中介

```java
public class HouseOwner {

    public boolean rentHouse(Integer money){
        if (money >= 1500){
            return true;
        }
        return false;
    }
}
```

```java
public class HouseProxy {
	//通过包含房东成员变量，来执行到委托类的方法
    HouseOwner houseOwner = new HouseOwner();

    public boolean rentHouse(Integer money){
        //做增强
        money = money - 500;
        return houseOwner.rentHouse(money);
    }
}
```

```java
public class HouseProxy2 extends HouseOwner{
	//通过继承调用父类的方法完成增强
    public boolean rentHouse(Integer money){
        money = money - 500;
        return super.rentHouse(money);
    }
}
```

静态代理，是需要我们自己来定义类的HouseProxy

##### 1.2.3.2动态代理

###### 1.2.3.2.1jdk动态代理

概念：

jdk提供的api来获得代理对象（增强对象）

jdk动态代理使用要有接口的实现

```java
public interface HelloService {

    public void sayHello();

    public void method2();
}
```

```java
public class HelloServiceImpl implements HelloService{
    @Override
    public void sayHello() {
        System.out.println("hello world");
    }

    @Override
    public void method2() {
        System.out.println("method2");
    }
}
```

```java
/**
 *动态代理
 */
public class ProxyTest {

    /**
     * 没有使用动态代理
     */
    @Test
    public void mytest(){
        //获得委托类对象
        HelloService helloService = new HelloServiceImpl();
        System.out.println("起床");
        helloService.sayHello();
        System.out.println("编程");
    }

    /**
     * 使用jdk动态代理
     */
    @Test
    public void mytest2(){
        //在hello world之前 起床

        //hello world之后 编程

        //先获得委托类对象的实例
        HelloService helloService = new HelloServiceImpl();
        //去获得一个代理对象来完成增强 → jdk动态代理获得增强对象（代理对象）
        //classloader都是和委托类对象相关的
        //interfaces都是和委托类对象相关的
        HelloService helloServiceProxy = (HelloService) Proxy.newProxyInstance(HelloServiceImpl.class.getClassLoader(),
                helloService.getClass().getInterfaces(), new InvocationHandler() {
            //invoke中
            //1是要去执行委托类的方法
            //2可以去做增强
            //返回值：Object 对应委托类方法执行的返回值
            //参数：
            //  proxy :代理对象
            //  method: 委托类方法的method
            //  args: 委托类方法的参数
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("起床");
                //invoke方法中可以使用反射来执行委托类的方法
                //第一个参数是委托类对象，而不是代理对象
                Object invoke = method.invoke(helloService, args);
                System.out.println("编程");
                return invoke;
            }
        });

        //helloServiceProxy.sayHello(); //使用代理对象去执行才会增强

        //helloService.sayHello(); //只会输出hello world

        helloServiceProxy.method2();

    }
    /**
     * 使用jdk动态代理
     * 相对于mytest2 修改了jdk动态代理接收的类型
     * 结论：jdk动态代理不能够使用实现类来接收代理对象
     */
    @Test
    public void mytest3(){
        //在hello world之前 起床

        //hello world之后 编程

        //先获得委托类对象的实例
        HelloService helloService = new HelloServiceImpl();
        //去获得一个代理对象来完成增强 → jdk动态代理获得增强对象（代理对象）
        //classloader都是和委托类对象相关的
        //interfaces都是和委托类对象相关的
        HelloServiceImpl helloServiceProxy = (HelloServiceImpl) Proxy.newProxyInstance(HelloServiceImpl.class.getClassLoader(),
                helloService.getClass().getInterfaces(), new InvocationHandler() {
            //invoke中
            //1是要去执行委托类的方法
            //2可以去做增强
            //返回值：Object 对应委托类方法执行的返回值
            //参数：
            //  proxy :代理对象
            //  method: 委托类方法的method
            //  args: 委托类方法的参数
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("起床");
                //invoke方法中可以使用反射来执行委托类的方法
                //第一个参数是委托类对象，而不是代理对象
                Object invoke = method.invoke(helloService, args);
                System.out.println("编程");
                return invoke;
            }
        });

        //helloServiceProxy.sayHello(); //使用代理对象去执行才会增强

        //helloService.sayHello(); //只会输出hello world

        helloServiceProxy.method2();

    }
}
```

###### 1.2.3.2.2cglib动态代理

不需要有接口的实现，cglib基于继承去实现的，proxy对象是继承委托类对象

代码写起来基本是一样，获得代理对象的过程使用api不同

```xml
<dependency>
   <groupId>cglib</groupId>
   <artifactId>cglib</artifactId>
   <version>3.2.12</version>
</dependency>
```

```java
/**
 * 用cglib单元测试使用
 * 这个类没有接口的实现
 */
public class HelloService2 {

    public void sayHello2(){
        System.out.println("hello world2");
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void method2(){
        System.out.println("method2");
    }
}
```

```java
/**
 * cglib动态代理
 */
public class CglibProxyTest {

    /**
     * 计算helloService2中的方法的执行时间
     */
    @Test
    public void mytest1(){
        //jdk没有提供，导包
        HelloService2 helloService2 = new HelloService2();
        //第二个参数也是InvocationHandler,和jdk动态代理的InvocationHandler不同
        HelloService2 helloService2Proxy = (HelloService2) Enhancer.create(HelloService2.class, new InvocationHandler() {
            //这三个参数和jdk动态代理是一样的
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                if ("method2".equals(method.getName())){
                    return method.invoke(helloService2,args);
                }
                long start = System.currentTimeMillis();
                Object invoke = method.invoke(helloService2, args);
                long end = System.currentTimeMillis();
                long cost = end - start;
                System.out.println(method.getName() + "执行时间是：" + cost);
                return invoke;
            }
        });
        helloService2Proxy.sayHello2();

        helloService2Proxy.method2();
    }
}
```

#### 1.2.4builder建造者

也是生产实例

更侧重参数的设置

造人 → human

```java
public class Foot {
    Integer size;

    public Integer getSize() {
        return size;
    }

    public void setSize(Integer size) {
        this.size = size;
    }

    @Override
    public String toString() {
        return "Foot{" +
                "size=" + size +
                '}';
    }
}
```

```java
public class Human {
    Foot foot = new Foot();
    Head head = new Head();
    Leg leg = new Leg();

    public Foot getFoot() {
        return foot;
    }

    public void setFoot(Foot foot) {
        this.foot = foot;
    }

    public Head getHead() {
        return head;
    }

    public void setHead(Head head) {
        this.head = head;
    }

    public Leg getLeg() {
        return leg;
    }

    public void setLeg(Leg leg) {
        this.leg = leg;
    }

    @Override
    public String toString() {
        return "Human{" +
                "foot=" + foot +
                ", head=" + head +
                ", leg=" + leg +
                '}';
    }
}
```

```java
public class HumanBuilder {

    Human human = new Human(); //保证了new一个builder的时候同时new了一个human，就是空白的human
    //这一些方法要给同一个人来创建
    //保证这一个方法中使用的是同一个human
    public HumanBuilder setFootSize(Integer size){
        human.getFoot().setSize(size);
        return this;
    }
    //需要保证都是同一个head
    public HumanBuilder setHeadIq(Integer iq){
        human.getHead().setIq(iq);
        return this;
    }
    public HumanBuilder setHeadEq(Integer eq){
        human.getHead().setEq(eq);
        return this;
    }
    public HumanBuilder setHeadName(String name){
        human.getHead().setName(name);
        return this;
    }
    public HumanBuilder setLegLength(Integer length){
        human.getLeg().setLength(length);
        return this;
    }
    public HumanBuilder setLegStrong(boolean strong){
        human.getLeg().setStrong(strong);
        return this;
    }

    public Human build(){
        return human;
    }

    public Human getHuman() {
        return human;
    }

    public void setHuman(Human human) {
        this.human = human;
    }
}
```

