---
title: 【java】以简单的图书管理系统讲清楚类，对象，继承，多态与接口的相关知识点
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E7%B1%BB%E5%92%8C%E5%AF%B9%E8%B1%A1.jpg
tags:
  - java
abbrlink: f39d0df0
date: 2023-03-01 19:42:17
categories: 
- java
ai: ture
---

>【前言】🍊以图书管理系统的实现，来讲清楚一些java入门学习阶段非常重要的概念与思想。包括类与对象，接口与继承，还有多态等等基础且重要的知识点。


# ⚽️ 准备
构建一个简单的图书管理系统，可以大致分为三个package。
+ user包
+ book包
+ operation包

也就是分为`用户，书，操作功能`。
其中user包包含user类，regularuser类，adminuser类。
book包包含book类，booklist类。
operation包包含addbook类，deletebook类，findbook类，borrowbook类，returnbook类。

# 🏀类与对象
上面我们提到了类，那什么是类？
java是一门`面向对象`的语言。面向对象就是依靠`对象的交互`来完成一件事。使用面向对象的思路来写程序，会更加友好。
举个例子，比如你打算去菜市场买菜，你需要准备菜篮子，换好衣服，规划好一条路线，沿途会经过哪些商贩，又会去买哪些蔬菜······，但是如果使用外送服务，你只需要下单时注明需要什么菜，几时送到什么地方即可。在这个过程里，压根不会关心菜是怎么买来的。又比如洗衣服，传统的洗衣过程是很繁琐的，但现代的洗衣机可以让步骤节省到三步，丢衣服进去，洗衣，拿出来晾干。这就是面向对象来处理问题。
如上所说，洗衣机是对象，外送服务也是对象，对象就好比是`现实生活中的实体`。对对象进行一个`抽象`所得到的就是类。
那么在java中如何去描述一个类呢？定义一个类需要`class关键字`。
例如定义user类：
```java
public class User {
    private String name;
    
}
```
括号中所包含的就是类的成员。属性用来`描述类`，称为`类的成员变量`。就像上面的user类，可以添加`名字name，年龄age，性别gender`等完善这个类，有点像c语言里的结构体。类中也可以包含`方法`，类似于c的函数。 在后面的例子里会添加方法。
类有以下几点需要注意：
+ 一般一个文件只定义`一个类`。
+ main方法所在类需要public修饰。
+ public修饰的类的名字要和`文件名相同`。

>类就好比是一个`模板`，对象就是依托于模板构建出来的`实例`，用类这个类型创建对象，这个过程我们称为`实例化`。
一个明显的类比：类实例化出对象，就相当于用设计图纸建造一栋真实的房子。

在java里，定义一个类，就相当于创造了一个新的类型，我们可以通过`new关键字`，来配合类实例化出各种对象。
+ new的对象都是在`堆上`。所以其生命周期与程序的运行周期一样。

例如实例化一个用户：
```java
 public static void main(String[] args) {
        User person1=new User("taylor");  //实例化的基本格式
        System.out.println(person1.name);
    }
```
这样就实例化出一个对象peison1，并且可以赋名为taylor（这个要配合构造方法，后面会讲）。
+ 用`.`来访问对象中的属性和方法。如上面的输出，就可以调用person1.name，来打印person1的名字。
+ 同一个类创建多个实例。

接着创建出Regularuser和Adminuser两个类来。

# 🏈 构造方法
在写这两个类前需要在user类里添加`构造方法`。
首先搞明白什么是构造方法。
就像我们熟知的，定义一个局部变量，在使用前必须要`初始化`，类也是一样。类借助`构造方法来完成初始化`。但`不`负责开辟空间。
构造方法是一个特殊的成员方法。
+ `名字必须与类名相同`。
+ 一般由`public`修饰。
+ 没有返回值类型。
>并且在创建对象（实例化）时，由编译器来`自动`进行调用，且在生命周期类只调用`一次`。

以下就是书写User类的构造方法，因为我们只定义了一个name变量，所以构造方法里也只需要对name进行初始化。
```java
public class User {
    private String name;

    public User(String name) {
        this.name = name;
    }
}
```
这是`有参数`的构造方法。如果我们没有`显式定义`，编译器在编译时也会`自动生成无参数`的构造方法。但是一但用户自己定义了，编译器就`不再生成`。
```java
public User(){
      //无参的构造方法，可以在此放入初始化的变量，如this.name="hehe"; 如果user类没有成员也可以啥都不写
}
```

在构造方法里，有一个this关键字。
其作用在于：
>this引用指向当前对象。也就是说，`谁在调用这个方法，谁就是this`。
this.name = name的意思就是将传进来的name赋给了这个类的对象的name。


构造方法可以进行`重载`。也就是在一个类里可以对构造方法`根据用户不同的需求`提供。
```java
public class User {
    private String name;
    int age;
    public User()
    {
        this.name="xiaoming";
        this.age=12;
        System.out.println("调用了无参构造方法");
    }
    public User(String name) {
        this.name = name;
        System.out.println("调用了只需name的构造方法");
    }
    public User(String name,int age){
        this.age=age;
        this.name=name;
        System.out.println("调用了需要name和age的构造方法");
    }

    public static void main(String[] args) {
        User person1=new User("taylor");
        System.out.println(person1.name);
    }
}
```

我们只初始化了name这个属性，那么就调用参数只有name的构造方法。
如果只初始化name，但要求访问age呢？
```java
   User person1=new User("taylor");
        System.out.println(person1.age);
```
输出`0`
这是为何呢？我们明明只初始化了name呀，为何age可以输出为0？
>在new的时候，编译器就自动完成了对成员变量的`默认初始化`了。即当对象`空间被申请好`，对象中所包含的成员就已经`设置好了初值`。

+ boolean   false
+ char    '/uoooo'(null)
+ byte (byte)0
+ short (short)0
+ int 0
+ long 0L
+ float 0.0f
+ double 0.0d

以上就是关于构造方法需要掌握的一些知识点。在书写完user类的构造方法后就可以去写adminuser类和regularuser类了。

# ⚾️继承
普通用户和管理员用户，虽然对系统所拥有的权限不完全相同，但也有大量相似的信息，面向对象的思想中提出了继承的概念，用以进行`共性抽取与代码的复用`。
>继承机制，允许在保持原有类特性的基础上。来进行一个拓展，并往里面添加`新的功能`。
另一个功能就是实现`多态`。

比如往类中放入名字，年龄，性别等成员变量，这些变量在adminuser类和regularuser类里都会用到，那么出于代码复用的思想，就可以将其提取出来作为一个公用的父类（也被称为基类），通过继承后，就可以使得子类可以`复用父类中的成员，并只需要添加自己所需要的特别的成员或方法`。
继承需要借助`关键字extends`:
普通用户类：
```java
public class Regularuser extends User{
    
    public Regularuser(String name) {
        super(name);
    }
}
```
管理员类：
```java
public class Adminuser extends User {
    public Adminuser(String name) {
        super(name);
    }
}
```
>可以看到，子类也是有构造方法的。但是`有其子之前必须先有父`，在对子类对象进行一个构造时，需要先去调用`父类`的构造方法，再执行`子类`的构造方法。


+ 子类对象成员有两部分组成，父类继承下来的和子类新增加的。
+ 若父类是`无参构造`，则子类的无参构造`第一行`会默认是`super();`。
+ 若父类是`有参构造`，则此时就需要为子类显式定义构造方法，并且在子类构造方法中`选择合适的父类构造方面调用`。
+ `super关键字`就可以实现父类构造方法的调用，且必须是子类构造方法中的`第一行`才行，`不能与this同时出现`。

就像上面写的两个类，都继承于父类user，那么都需要在自己的构造方法第一行通过super关键字来调用父类构造，否则`编译器报错`。
 如果父类传下来多个参数呢？那么就顺着书写就好了。
 ```java
 super(name,age,gender);
 ```
 又比如现在在regularuser类里有一个只属于regularuser的属性，其构造方法就是这样：
 ```java
 public class Regularuser extends User{

    String Specialproperties;
    public Regularuser(String name, String specialproperties) {
        super(name);
        this.Specialproperties = specialproperties;
    }
}
```

关于继承还有一个小tip：`各类代码块的执行顺序`。
代码块可以分为三种：
*静态代码块*
```java
static{}
```
*实例代码块*
```java
{}
```
*构造代码块*
```java
public 类名(){}
```

其执行顺序为：
>1.父类静态代码块
2.子类静态代码块
3.父类实例代码块
4.父类构造代码块
5.子类实例代码块
6.子类构造代码块

静态代码块会先执行，并且只在`类加载阶段执行一次`。当有`对象进行创建时`，才会去执行实例的代码块，`当实例代码块执行完毕才是构造代码块`。
父类静态代码块比子类更快，是`最早执行`的。
第`二`次实例化`子类对象`时，`父类与子类的静态代码块都不会再执行`。

讲到这里了，就可以将book包里的book类和booklist类写一下了。
book类
```java
public class Book {
    private String bookname;
    private String author;
    private String type;
    private int price;
    private boolean isborrowed;

    public Book(String bookname, String author, String type, int price) {
        this.bookname = bookname;
        this.author = author;
        this.type = type;
        this.price = price;
    }

    public String getBookname() {
        return bookname;
    }

    public void setBookname(String bookname) {
        this.bookname = bookname;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public boolean isIsborrowed() {
        return isborrowed;
    }

    public void setIsborrowed(boolean isborrowed) {
        this.isborrowed = isborrowed;
    }

    @Override
    public String toString() {
        return "Book{" +
                "bookname='" + bookname + '\'' +
                ", author='" + author + '\'' +
                ", type='" + type + '\'' +
                ", price=" + price +
                ", isborrowed=" + isborrowed +
                '}';
    }
}
```
除了成员变量和构造方法，还额外加入了`各种变量的getter and setter以及toString方法的重写`。
getter和setter很好理解，就是`提供一个单一的方法来对特定的变量来进行赋值和取值的作用`。
```java
    public static void main(String[] args) {
        Book book =new Book("aaa","bbb","ccc",12);
        System.out.println(book.getAuthor());
        System.out.println(book.isIsborrowed());

    }
```
这样就读取到了book这个对象的author以及isborrowed。也可以为其重新赋值。   
```java
        book.setAuthor("taylor swift");
        System.out.println(book.author);
```

还有toString方法，其是在`Object类`里定义的，其返回值类型为String类型，返回类名和它的引用地址。`Object类是所有类的父类`，所以我们可以在Book类中对这个方法进行`重写`。重写和重载是有很大的区别的。
在上文我们提到过重载，其是在`一个类中的多态表现`，而重写则是在`父类与子类之间的多态表现`。
+ 重载的`参数类型`是`必须修改`的，因为需要依靠传入参数不同来决定进入哪一个方法。具体可以看上文的重载实例。且`返回类型可以被修改`。
+ 而这里提到的重写，`参数列表是不可以被修改的`，返回类型一般也不可以改，除非是父子类关系。可以看出，重写是`重新书写一个方法的内部`，而重载是`提供多种同名方法的入口`。

想调用toString方法也很简单，直接sout book这个对象即可。
```java
  System.out.println(book);
```
打印出重写的结果：
>Book{bookname='aaa', author='bbb', type='ccc', price=12, isborrowed=false}

*重写的设计原则*
对于一个已经`投入使用的类`，就不要再对其进行修改了。而是定义一个新的类，来重复其中的`共性内容`，同时添加新的内容或功能。
一个很简单的例子，手机是有更新换代的。在这个迭代的过程中势必会有软件硬件上的更新。就像是接打电话的界面，可能在后续的系统更新里，会对这个界面进行一个升级，但是总会有用户不想升级，或者手机型号老旧无法升级，为了顾及到整体的用户体验，就不能去更改原有的`相关类`，而是选择开发一个新的类来进行重写。


接下来书写booklist类。
booklist类的作用就是充当一个书架。并往里面先预放置几本书。
```java
public class Booklist {
    Book []books =new Book[10]; //就将数量设定为10本
    int usedsize; //目前存储书籍的本数。

   public Booklist(){
       books[0]=new Book("局外人","加缪","文学",22);
       books[1]=new Book("人间失格","太宰治","文学",27);
       books[2]=new Book("人生的智慧","叔本华","文学",56);
       this.usedsize=3;
   }

    public Book[] getBooks() {
        return books;
    }

    public void setBooks(Book[] books) {
        this.books = books;
    }

    public int getUsedsize() {
        return usedsize;
    }

    public void setUsedsize(int usedsize) {
        this.usedsize = usedsize;
    }

    
}
```

接下来就还有主菜单和operation包了。

# 🏐多态与抽象类
先来了解一下抽象类与多态的相关知识点。
*抽象类*
在面向对象的概念里，所有的`对象都是通过类来描述`的，如果一个类`没有足够的信息去描述一个对象`，这样的类就称为`抽象类`。
例如我们现在定义一个抽象类shape，用来画图形，但是具体画什么我们不清楚，就可以写出如下代码。
```java
abstract class shape{
    abstract void draw();

    public shape() {
    }
}
class cycle extends shape{
    public cycle() {
    }
    public void draw(){
        System.out.println("画一个圆");
    }

}
class rect extends shape{
    public rect()
    {

    }
    public void draw(){
        System.out.println("画一个矩形");
    }
}
 public static void main(String[] args) {
        cycle a=new cycle();
        a.draw();
        rect b= new rect();
        b.draw();
    }

```
输出
画一个圆
画一个矩形


+ shape类由`abstract`修饰，代表他是一个抽象类。
+ cycle和rect通过`extends关键字`继承了shape类，并对draw方法进行了`重写`。
+ shape作为一个抽象类`必须被继承`，并且继承后的子类要`重写父类的抽象方法`。不然子类也会变成抽象类，需要用abstract修饰。
+ 抽象方法不可以是private修饰，抽象类shape也`不可以直接实例化`对象。
+ 抽象方法也不要用final和static修饰，抽象方法是`必须被重写`的。
+ 抽象类中可以`存在构造方法`，让子类创建对象时初始化父类成员变量。

可以看到，`调用同名方法draw，却实现了不同的效果`，这就是`多态的体现`。同一件事发生在不同的对象，就会产生不同的结果。
多态的使用有如下条件：
+ 必须处于继承的条件下。
+ 子类必须对父类方法进行重写。
+ 通过父类的引用调用重写方法。

父子类同样可以实现多态，那为何我们要采取抽象类？这不是多此一举吗？
>使用抽象类实则是多一层`编译器的校验`。因为就像shape类，画图的工作不需要shape类，而是其两个子类，cycle和rect。但是如果只是普通的继承，那么可能错误的去`调用了shape类`，此时编译器不会报错。而当父类shape是抽象类时，在实例化时就会报错。这些语法的存在就是为了`“预防出错”`。

有了基本的知识储备就可以开始着手主菜单了。新建一个class Main。
第一步就是先初始化一个书架。
```java
public class Main {
    Booklist booklist =new Booklist();
}
```
第二步就很重要了，要写登录的代码。因为管理员和普通用户的菜单不一样，管理员会比普通用户多一些功能，例如添加图书，删除图书等，所以我们展示的菜单是不一样的。根据才介绍的多态，还需要引入一个比较重要的知识点：
*向上转型和向下转型*
```java
 public static User login(){
        System.out.println("输入姓名");
        Scanner scanner =new Scanner(System.in);
        String name=scanner.toString();
        System.out.println("你是管理员(1)还是用户(0)");
        int choice =scanner.nextInt();
        if(choice==1)
            return new Adminuser(name);
        else
            return new Regularuser(name);

    }
```
login方法的返回类型是User，是return的adminuser与regularuser的`父类`，`子类转为父类`，这里就实现了一个`向上转型`。这样就很方便的使用一个方法就可以return两种不同的类型（但只会返回一个，只是有两种选择）。那么此时就可以在adminuser与regularuser两个类中放入不同的菜单来进行调用了。这里要介绍动态绑定的知识点。
*动态绑定*
>动态绑定是指在执行期间（非编译期）判断所引用对象的实际类型，根据其`实际的类型`调用其相应的方法。

也就是说，虽然adminuser与regularuser中的一个被向上转型成为了user类型，但实际调用的仍然是子类型，所以在调用同名方法时会`优先调用子类的方法`。所以往子类里添加相应的管理菜单即可。

向下转型则是父类向子类进行转换，因为当子类转换到父类后，就不可以在使用子类的特有方法了。如果想要`调用子类的特有方法`，就需要完成`向下转型`，将父类引用重`新还原`为子类对象。
```java
Adminuser user1=(Adminuser)user;
```
这样就完成了向下的转型，此时user1就可以使用adminuser类里的特有方法了。
往两个类中添加菜单：
管理员类：

```java
public void menu(){
        System.out.println("欢迎你！"+name);
        System.out.println("0.退出系统");
        System.out.println("1.借阅图书");
        System.out.println("2.归还图书");
        System.out.println("3.查找图书");
        System.out.println("4.添加图书");
        System.out.println("5.删除图书");
    }
```
普通用户类
```java
 public void menu(){
        System.out.println("欢迎你！"+name);
        System.out.println("0.退出系统");
        System.out.println("1.借阅图书");
        System.out.println("2.归还图书");
        System.out.println("3.查找图书");

    }
```
此时还要记得将`menu方法添加到user类`中，才能正常的在子类里调用。
根据抽象类的知识，就可以将user类改为一个抽象类。
```java
public abstract class User {
     String name;
    public User(){
    }
    public abstract void menu();
    public User(String name) {
        this.name = name;
        }
        }
  ```
  完善menu方法
 
```java
 public int menu(){
        System.out.println("欢迎你！"+name);
        System.out.println("0.退出系统");
        System.out.println("1.借阅图书");
        System.out.println("2.归还图书");
        System.out.println("3.查找图书");
        System.out.println("4.添加图书");
        System.out.println("5.删除图书");
        Scanner scanner=new Scanner(System.in);
        int choice =scanner.nextInt();
        return choice;

    }
```
记得user类和regularuser类也要进行更改！
在main类中完善代码：
```java
 public static void main(String[] args) {
        Booklist booklist =new Booklist();

        User user=login();
        while(true) {
            int choice = user.menu();
        }
    }
```
这样就基本完成了图书管理系统的结构。剩下的就是operation包了。

# 🎱接口

operation包里放的就是上述的menu方法中的功能了。通常在书写功能时，会用到`接口`的思想。
>接口就是公共的行为规范标准，在java中，接口可以理解为`多个类`的公共`规范`，是一种引用数据类型。

接口的定义规则如下：
```java
public interface 接口名{
  //放抽象方法。
}
```

+ 接口中的成员变量默认是由`public static final`来修饰。
+ 接口中的方法默认是`public abstract`修饰。
+ 接口不可以直接使用或者实例化，而是用一个`实现类`来实现接口。
+ 子类父类是extends实现关系，而接口与类之间则是`implements`实现关系。
+ 接口中`不能`含有静态方法和构造方法。

简单的说，就是由一个接口来规范功能的书写。
如下书写了一个简单的接口，只包含一个方法work。
```java
public interface operation {
    void work(Booklist booklist);
}
```

>这个接口可以被`多个类`共同使用，同时`一个类`可以实现`多个接口`。这就是接口与继承的区别，虽然java中不支持多继承，但却支持实现多接口。


例如此时我们再书写一个接口。
```java
public interface exit {
    void exitprogress(Booklist booklist);
}
```
这样exit和operation两个接口都可以被一个类接收。这样这个类就可以重写两个接口的方法。
这也是java中面向对象的常见用法：`一个类继承父类，同时实现多个接口`。
使用接口的好处在于`忘记类型`，只需要去关注某个类是否`具有某种能力`。
例如现有接口hunt
```java
public interface hunt {
    void dowork();
}
```
与一个继承于Animal的Snake类，那么可以写出如下代码：
```java
public class Snake extends Animal implements hunt{
    public Snake(String name) {
        super(name);
    }
    public static void huntanimal(hunt hunt){
        hunt.dowork();
    }

    @Override
    public void dowork() {
        System.out.println(name+"吃兔子");
    }

    public static void main(String[] args) {
        Snake snake=new Snake("蛇");
        huntanimal(snake);
    }
}
```
在调用方法huntanimal时，就无需考虑到类型的传入，只需要去关心传入对象是否具有`相对应的某种能力`了。那么自然可以重新建一个tiger类，wolf类，只要是具备hunt能力的我们都可以通过huntanimal这个方法来调用。


以下摘自一位大牛：
>1、重要性：在Java语言中， abstract class 和interface 是支持抽象类定义的两种机制。正是由于这两种机制的存在，才赋予了Java强大的 面向对象能力。
2、简单、规范性：如果一个项目比较庞大，那么就需要一个能理清所有业务的架构师来定义一些主要的接口，这些接口不仅告诉开发人员你需要实现那些业务，而且也将命名规范限制住了（防止一些开发人员随便命名导致别的程序员无法看明白）。
3、维护、拓展性：比如你要做一个画板程序，其中里面有一个面板类，主要负责绘画功能，然后你就这样定义了这个类。  可是在不久将来，你突然发现这个类满足不了你了，然后你又要重新设计这个类，更糟糕是你可能要放弃这个类，那么其他地方可能有引用他，这样修改起来很麻烦。   如果你一开始定义一个接口，把绘制功能放在接口里，然后定义类时实现这个接口，然后你只要用这个接口去引用实现它的类就行了，以后要换的话只不过是引用另一个类而已，这样就达到维护、拓展的方便性。
4、安全、严密性：接口是实现软件松耦合的重要手段，它描叙了系统对外的所有服务，而不涉及任何具体的实现细节。这样就比较安全、严密一些（一般软件服务商考虑的比较多）。


先实现查找图书类。
往booklist里新添加了一个方法getpos，返回i下标的book类。
```java
  public Book getPos(int i){
       return books[i];
    }
```
实现查找图书的方法work。
```java
public class findbook implements operation {
    public  void work(Booklist booklist) {
        System.out.println("请输入需要查找的图书");
        Scanner scanner=new Scanner(System.in);
        String bookname=scanner.nextLine();
        int currentsize=booklist.getUsedsize();
        for (int i = 0; i <currentsize ; i++) {
            Book book=booklist.getPos(i);
            if(bookname.equals(book.getBookname())){
                System.out.println("找到了");
                System.out.println("book");
                return;
            }
        }
        System.out.println("没找到");

    }
}
```

接下来是添加图书功能。
先修改booklist里的setbook方法(之前是使用idea自动生成的，但和我们想要的功能有差异，就重写了)
```java
 public void setBooks(Book book,int pos) {
        books[pos]=book;
    }
```
实现添加图书的work方法
```java
public class addbook implements operation{
    @Override
    public void work(Booklist booklist) {
        System.out.println("请输入要添加的图书");
        Scanner scanner=new Scanner(System.in);
        String bookname=scanner.nextLine();
        System.out.println("请输入图书作者");
        String bookauthor=scanner.nextLine();
        System.out.println("请输入图书类型");
        String booktype=scanner.nextLine();
        System.out.println("请输入图书价格");
        int bookprice=scanner.nextInt();
        Book book =new Book(bookname,bookauthor,booktype,bookprice);
        int currentsize=booklist.getUsedsize();
        booklist.setBooks(book,currentsize);
        booklist.setUsedsize(currentsize+1);

    }
}

```

 实现删除的方法
 ```java
 public class deletebook implements operation{
    @Override
    public void work(Booklist booklist) {
        System.out.println("请输入要删除的图书");
        Scanner scanner=new Scanner(System.in);
        String bookname=scanner.nextLine();
        int currentsize= booklist.getUsedsize();
        int index=-1;
        for (int i = 0; i <currentsize ; i++) {
            Book book=booklist.getPos(i);
            if(bookname.equals(book.getBookname())){
                index=i;
                break;
            }
        }
        if(index==-1)
            System.out.println(" 未找到图书");
    }
}

 ```

借书和归还则写的简单些。只需要将isborrowed进行修改即代表借出与归还两种状态。
```java
//借书，注意要放入两个文件中，一个文件只能有一个public修饰的类。
public class borrowbook implements operation{
    public void work(Booklist booklist) {
        System.out.println("请输入要借阅的图书");
        Scanner scanner=new Scanner(System.in);
        String bookname=scanner.nextLine();
        int currentsize= booklist.getUsedsize();
        for (int i = 0; i <currentsize ; i++) {
            Book book=booklist.getPos(i);
            if(bookname.equals(book.getBookname())){
               book.setIsborrowed(true);
            }
        }
        
    }
}
//还书
public class returnbook implements operati{
    public void work(Booklist booklist) {
        System.out.println("请输入要归还的图书");
        Scanner scanner=new Scanner(System.in);
        String bookname=scanner.nextLine();
        int currentsize= booklist.getUsedsize();
        for (int i = 0; i <currentsize ; i++) {
            Book book=booklist.getPos(i);
            if(bookname.equals(book.getBookname())){
                book.setIsborrowed(false);
            }
        }

    }
}
```
退出系统
```java
public class exitsystem implements  operation{
    @Override
    public void work(Booklist booklist) {
        System.out.println("退出系统");
        System.exit(0);
    }
}

```
以上就简单将功能都实现了一遍，可以看到，在实现功能时，getter and setter方法的书写是非常重要的，我们经常会用到单个的成员变量，通过getter and setter来调用是非常方便且具有可读性的。

整个图书管理系统还差最关键的一步，`通过菜单来调用对应的方法`。这点我们会用到`接口数组`的思想。
简单的说，就是让`对象存储自己的操作`。虽然操作功能的类型各不相同，但都实现了operation接口，那么就可以让`实现了这个接口的类`都存到这个数组中去。
先在user类里添加一个接口数组，但不对其进行初始化，因为此时不知道具体大小(`两个子类的菜单数是不同的`)。
```java
 protected operation[] op;
 public void dooperation(int choice, Booklist booklist){
        op[choice].work(booklist);
    }
```
而且不可以跳过user类直接在两个子类中一气呵成初始化数组，因为Main类里是user类在调用方法，如果user类`不存在`调用这个数组的方法，编译器会报错。
在adminuser和regularuser两个类的`构造方法`对数组进行初始化。
```java
//在这里就一起给出了
 public Adminuser(String name) {
        super(name);
        this.op=new operation[]{
                new exitsystem(),
                new borrowbook(),
                new returnbook(),
                new findbook(),
                new addbook(),
                new deletebook()
        };
    }
      public Regularuser(String name) {
        super(name);
        this.op=new operation[]{
                new exitsystem(),
                new borrowbook(),
                new returnbook(),
                new findbook()
        };
```

接着写在Main类里面的主函数：
```java
 while(true) {
            int choice = user.menu();
            user.dooperation(choice,booklist);
        }
```

整个简易的图书管理系统就做好了。