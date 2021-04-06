[TOC]
# 一、Java基础

## 	1.JDK与面向对象

### **String、StringBuilder、StringBuffer**

**（1）三者区别？**

- String 不可变，StringBuilder 与 StringBuffer 是可变的。

> String 类使用private final char value []，所以不可变的**（Java 9 换成了 byte 数组，占用更少空间）**。
>
> StringBuilder 与 StringBuffer 都继承自 **AbstractStringBuilder** 类，这两种对象都是可变的。

- 线程安全性。 

> String 和 StringBuffer 是线程安全的，StringBuilder 是非线程安全的。
>
> StringBuffer 线程安全是因为对方法加了同步锁或者对调用的方法加了同步锁，即synchronized。

- 性能

> String 每次改变时都会生成新的 String 对象，所以性能较差。
>
> 而 StringBuffer/StringBuilder 性能更高，是因为每次都是对对象本身进行操作，而不是生成新的对象并改变对象引用。一般情况下性能从高到低：StringBuilder、StringBuffer、String，但是某些情况下不一定。
>
> Java 5之后，单纯的字符串+操作会自动优化为StringBuilder，例如"Hello" + "World"，但是若使用非final修饰的变量，则不会进行优化。

 **（2）String str = new String("Hello World!")；创建了几个对象？**

 - 如果字符串常量区里存在"Hello World!"，则new String()创建了一个对象，否则创建了两个对象，str为引用变量。

 **（3）什么情况下使用？**

 - 如果要操作少量的数据用 String；
 - 单线程操作字符串缓冲区下操作大量数据 StringBuilder；
 - 多线程操作字符串缓冲区下操作大量数据 StringBuffer，但是一般情况下也很少用到，因为不保证逻辑正确和调用顺序正确，大多数时候，需要的不仅仅是StringBuffer，而是锁；

**（4）String为什么要设计成final类型**

- 不可变性支持线程安全；

- 不可变性支持字符串常量池，提升性能；

- 保证hashCode的唯一性，不需要重新计算，提高存储效率；

  > 若String可变，则不能实现Map等集合的安全存储

 ### **Integer和int**

**（1）给出各 == 运算符的逻辑结果值**

```java
public static void main(String[] args) {
    Integer a = new Integer(3);
    Integer d = new Integer(3);   // 通过new来创建的两个Integer对象
    Integer b = 3;                  // 将3自动装箱成Integer类型int c = 3;
    int     c = 3;                  // 基本数据类型3
    System.out.println(a == b);     // false 两个引用没有引用同一对象
    System.out.println(a == d);     // false 两个通过new创建的Integer对象也不是同一个引用
    System.out.println(c == b);     // true b自动拆箱成int类型再和c比较
}
```

> 当两边都是 Integer 对象时，是引用比较；当其中一个是 int 基本数据类型时，另一个 Integer 对象也会自动拆箱变成 int 类型再进行值比较。

```java
public static void main(String[] args) {
    Integer f1 = 100;
    Integer f2 = 100;
    Integer f3 = 150;
    Integer f4 = 150;
    System.out.println(f1 == f2);   // true，当int在[-128,127]内时，结果会缓存起来
    System.out.println(f3 == f4);   // false，属于两个对象
}
```

>如果整型字面量的值在 - 128 到 127 之间，那么不会 new 新的 Integer 对象，而是直接引用常量池中的 Integer 对象.
>
>Integer f1 = 100;java在编译的时候,被翻译成-> Integer f1 = Integer.valueOf(100);

### **equals、== 和 hashCode**

**（1）==比较基本数据类型和引用类型的区别？ **

- 基本数据类型比较的是他们的值，引用类型（类、接口、数组）比较的是他们在内存中的存放地址；
- 引用类型==是比较地址，equals是比较地址中的值

**（2）String中equals方法判断相等的步骤？**

- String对equals方法进行了重写

- 若A==B 即是同一个String对象 返回true

- 若对比对象是String类型则继续，否则返回false

- 判断A、B长度是否一样，不一样的话返回false

- 逐个字符比较，若有不相等字符，返回false

  > equals方法：
  >
  > 自反性（x.equals (x) 必须返回 true）；
  >
  > 对称性（x.equals (y) 返回 true 时，y.equals (x) 也必须返回 true）；
  >
  > 传递性（x.equals (y) 和 y.equals (z) 都返回 true 时，x.equals (z) 也必须返回 true）；
  >
  > 一致性（当 x 和 y 引用的对象信息没有被修改时，多次调用 x.equals (y) 应该得到同样的返回值），而且对于任何非 null 值的引用 x，x.equals (null) 必须返回 false。

**（3）hashCode与equals联系以及作用？**

- 解决冲突：当集合添加新元素，先调用元素hashCode方法定位物理位置，若位置上没有元素，则直接存储，否则调用位置上元素的equals方法与新元素进行比较，相同则不存储，不相同则存储其他位置。

> 如果两个对象equals，Java运行时环境会认为他们的hashcode一定相等；
> 如果两个对象不equals，他们的hashcode有可能相等；
>如果两个对象hashcode相等，他们不一定equals；
> 如果两个对象hashcode不相等，他们一定不equals；

### **序列化**与反序列化

**（1）序列化的实现方式有哪些，有什么区别，代码如何实现？**

- 实现Serializable（可序列化）接口

- 实现Externalizable（可外部化）接口，实现writeExternal、readExternal方法

- 区别：

  | 实现Serializable接口                                         | 实现Externalizable接口   |
  | :----------------------------------------------------------- | :----------------------- |
  | 系统自动存储必要的信息                                       | 程序员决定存储哪些信息   |
  | Java内建支持，易于实现，只需要实现该接口即可，无需任何代码支持 | 必须实现接口内的两个方法 |
  | 性能略差                                                     | 性能略好                 |

  > 虽然Externalizable接口带来了一定的性能提升，但变成复杂度也提高了，所以一般通过实现Serializable接口进行序列化。

``` java
/********** Serializable方式 **********/
// 序列化
public void WriteObject() {
    try {
        // 创建一个ObjectOutputStream输出流
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("object.txt"))) {
        // 调用ObjectOutputStream对象的writeObject输出可序列化对象
        Person person = new Person("9龙", 23);
        oos.writeObject(person);
    } catch (Exception e) {
        e.printStackTrace();
    }
}

// 反序列化
public void ReadObject() {
    try (
        // 创建一个ObjectInputStream输入流
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("person.txt"))) {
      	// 调用ObjectInputStream对象的readObject()得到序列化的对象
        Person brady = (Person) ois.readObject();
      	// 反序列化并不会调用构造方法。反序列的对象是由JVM自己生成的对象，不通过构造方法生成
    } catch (Exception e) {
        e.printStackTrace();
    }
}
  
/********** Externalizable方式 **********/
@Override
public void writeExternal(ObjectOutput out) throws IOException {
    //将name反转后写入二进制流
    StringBuffer reverse = new StringBuffer(name).reverse();
    System.out.println(reverse.toString());
    out.writeObject(reverse);
    out.writeInt(age);
}

@Override
public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
    //将读取的字符串反转后赋值给name实例变量
    this.name = ((StringBuffer) in.readObject()).reverse().toString();
    System.out.println(name);
    this.age = in.readInt();
}

public static void main(String[] args) throws IOException, ClassNotFoundException {
	try {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ExPerson.txt"));
        OjectInputStream ois = new ObjectInputStream(new FileInputStream("ExPerson.txt"));
        oos.writeObject(new ExPerson("brady", 23));
        ExPerson ep = (ExPerson) ois.readObject();
        System.out.println(ep);
    }
}
```

**（2）如果实现Serializable的类中包含一个不可序列化的成员，会发生什么？如何解决？**

- 任何序列化该类的尝试都会因 NotSerializableException 而失败，可以通过在 Java 中给属性设置瞬态 (transient) 变量来轻松解决。

> transient关键字：瞬态关键字
> 被transient修饰的成员变量，不能被序列化
>
> 使用transient修饰的属性，java序列化时，会忽略掉此字段，所以反序列化出的对象，被transient修饰的属性是默认值。对于引用类型，值是null；基本类型，值是0；boolean类型，值是false。
>
> static关键字：静态关键字
> 静态优先于非静态加载到内存中（静态优先于对象进入到内存中）
> 被static修饰的成员变量同样不能被序列化，序列化的都是对象

**（3）同一对象序列化多次，会将这个对象序列化多次吗？**
- 不会，Java序列化同一对象，并不会将此对象序列化多次得到多个对象。
> Java序列化算法：
> 所有保存到磁盘的对象都有一个序列化编码号
> 当程序试图序列化一个对象时，会先检查此对象是否已经序列化过，只有此对象从未（在此虚拟机）被序列化过，才会将此对象序列化为字节序列输出。
> 如果此对象已经序列化过，则直接输出编号即可。

**（4）private static final long serialVersionUID = 1L;的作用是什么？**

- 在进行序列化工作时，会将serialVersioinUID与所要序列化的目标一起序列化，这样一来，在反序列化的过程中会使用被序列化的serialVersioinUID与类中的serialVersioinUID对比，如果两者相等，则反序列化成功，否则，反序列化失败。

  > 假设Person类序列化之后，从A端传输到B端，然后在B端进行反序列化。在序列化Person和反序列化Person的时候，A端和B端都存在一个相同的类。若B端这个类的serialVersionUID与A端不同，则B端反序列化失败。

**（5）如何自定义序列化？**

- 通过重写writeObject与readObject方法。

  ```java
  public class Person implements Serializable {
     private String name;
     private int age;
     //省略构造方法，get及set方法
  
     private void writeObject(ObjectOutputStream out) throws IOException {
         //将名字反转写入二进制流
         out.writeObject(new StringBuffer(this.name).reverse());
         out.writeInt(age);
     }
  
     private void readObject(ObjectInputStream ins) throws IOException,ClassNotFoundException{
         //将读出的字符串反转恢复回来
         this.name = ((StringBuffer)ins.readObject()).reverse().toString();
         this.age = ins.readInt();
     }
  }
  ```

  > 当序列化流不完整时，readObjectNoData()方法可以用来正确地初始化反序列化的对象。例如，使用不同类接收反序列化对象，或者序列化流被篡改时，系统都会调用readObjectNoData()方法来初始化反序列化的对象。
  
### final和static

**（1）final 有哪些用法？**

- final来修饰数值，数值不可变；
- final来修饰对象，不能改变对象的引用，但是可以修改对象的属性值；
- final来修饰参数，参数不可变；
- final修饰方法，方法不可以被重写；
- final修饰类，类不可以被继承；

**（2）static有哪些用法？**

- static方法，用以进行调用，不需要创建对象；
- static变量，用以共享变量；
- static代码块，可以用来做初始化操作；

> static是为了方便在没有创建对象的情况下来进行调用，static不会改变变量和方法的访问权限，也不允许用来修饰局部变量。

**（3）final 和 static的区别是什么？**

- final修饰，主要是为了表现“不可修改性”，从而提高安全性 。
- static重点在于共享，方便。在类里创建一个static修饰的函数，则可以直接通过类名访问，该类new出来的对象，也可以共享static函数，或者static修饰的共有属性。

### for和switch

**（1）Java 中如何跳出多重循环？**

- break + 标签。在最外层循环前加一个标签如 label，然后在最里层的循环使用用 break label；
- 通过捕获异常；
- 通过标置变量；

**（2）switch语句能否作用在byte上，能否作用在long上，能否作用在String上？**

- 可作用于char byte short int 以及对应的包装类 ；
- 不可作用于long double float boolean以及对应包装类 ；
- JDK1.7之后可以作用在String上 ；
- 可以是枚举类型 

### 抽象类（abstract class）和接口（interface）

**（1）两者有什么异同？**

- 相同点：

  都不能被实例化

  都可以包含方法声明

  派生类必须实现未实现的方法

  > Java8以后允许在接口内声明静态方法，并且可以指定接口方法的默认实现。

- 不同点：
  抽象类可以有构造方法，接口中不能有构造方法。
  象类中可以有普通成员变量，接口中没有普通成员变量(接口中的方法定义默认为public abstract类型，接口中的成员变量类型默认为public static final)
  抽象类中可以包含静态方法，接口中不能包含静态方法
   一个类可以实现多个接口，但只能继承一个抽象类。
  接口可以被多重实现，抽象类只能被单一继承
  如果抽象类实现接口，则可以把接口中方法映射到抽象类中作为抽象方法而不必实现，而在抽象类的子类中实现接口中方法
  接口定义的关键字interface;抽象类定义的关键字abstract
  抽象类继承的关键字extends，接口的实现关键字**implements**
  抽象类中的抽象方法可以用 public protected 和 default abstract 修饰符，不能用 private、static、synchronize、native 修饰；变量可以在子类中重新定义，也可以重新赋值；
  抽象类的速度比接口能快一点，因为接口需要时间去寻找在类中实现的方法
  接口的方法默认修饰符是 public abstract， Java8 开始出现静态方法，多加 static 关键字；变量默认是 public static final 型，且必须给其初值，在实现类中也不能重新定义，也不能改变其值。
  如果你往抽象类中添加新的方法，你可以给它提供默认的实现，因此你不需要改变你现在的代码；如果你往接口中添加方法，那么你必须改变实现该接口的类。
  继承一个抽象类的时候，子类必须定义父类中的所有抽象方法；

**（2）什么时候使用抽象类什么时候使用接口？**

- 大部分情况下，使用接口定义一组行为和动作时，用接口，并且1.8之后接口可以定义默认实现，更加灵活；
- 当需要定义子类的行为，有要为子类提供基础性功能时，或者做一个封装，比如适配器模式等；

### 对象的创建

**（1）创建对象的几种方式？**

- 使用 new 关键字；

- 反射，使用 java.lang.Class 类的 newInstance 方法。

> 这种方式会调用无参的构造函数来创建对象，有两种实现方式。

```java
// 方式一，使用全路径包名
User user = (User)Class.forName("com.demo.User").newInstance();　
// 方法二,使用class类
User user = User.class.newInstance();
```

- 反射，使用 java.lang.reflect.Constructor 类的 newInstance 方法。

```java
Constructor<User> constructor = User.class.getConstructor();
User user = constructor.newInstance();
```

> Class.newInstance()只能反射无参的构造器；
> Constructor.newInstance()可以反任何构造器；
>
> Class.newInstance()需要构造器可见(visible)；
> Constructor.newInstance()可以反射私有构造器；
>
> Class.newInstance()对于捕获或者未捕获的异常均由构造器抛出;
> Constructor.newInstance()通常会把抛出的异常封装成InvocationTargetException抛出；

- 使用 clone 方法。

```java
public class User implements  Cloneable {
    /** 构造方法 */
    public User(Integer age) {
        this.age = age;
    }
    public Integer getAge() {
        return age;
    }
    private Integer age;

    // 重写（Overriding）Object的clone方法
    @Override
    protected User clone() throws CloneNotSupportedException {
        return (User) super.clone();
    }

    public static void main(String[] args) throws Exception {
        User person = new User(new Integer(200));
        User clone = person.clone();
        System.out.println("person == clone, result =  " +  (person == clone));  // false，拷贝都是生成新对象
        System.out.println("person.age == clone.age, result =  " +  (person.getAge() == clone.getAge())); // true，浅拷贝的成员变量引用仍然指向原对象的变量引用
    }
}
```

> 首先需要明确两个概念：**浅拷贝**和**深拷贝**。
>
> - **浅拷贝**：被复制对象的所有变量都含有与原来的对象相同的值，对拷贝后对象的引用仍然指向原来的对象。
> - **深拷贝**：不仅要复制对象的所有非引用成员变量值，还要为引用类型的成员变量创建新的实例，并且初始化为形式参数实例值。
>
> 其他需要注意的是，clone () 是 Object 的 native 方法，但如果要标注某个类是可以调用 clone ()，该类需要实现空接口 Cloneable。

- 使用反序列化。

> 序列化是深拷贝。

| 创建对象方式            | 是否调用了构造器 |
| :---------------------- | :--------------- |
| new 关键字              | 是               |
| Class.newInstance       | 是               |
| Constructor.newInstance | 是               |
| Clone                   | 否               |
| 反序列化                | 否               |

**（3）Java反射是指什么？它的使用场景及其优缺点分别什么？**

- Java反射是指在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

- 使用场景

  主要用于根据运行时信息来发现该类/对象/方法/属性的场景，典型的场景比如Spring等框架的配置、动态代理等。其原理主要是通过访问装载到JVM中的类信息，来获取类/对象/方法/属性等信息。

- 优点：

  通过在运行期访问装载到JVM中的类信息，来动态获取类的属性方法等信息，从而根据业务参数动态执行方法、访问属性，提高了java语言的灵活性和扩展性。典型就是Spring等应用框架。而其他常用的高级语言如C/C++不具备这样的能力；

  可以提高代码复用率。

- 缺点：

  性能较差，通常慢于直接执行java代码；

  程序的可维护性相对较差，业务代码和反射的代码交织在一起。

**（4）动态代理是指什么？它有哪几种实现方法？有什么好处？**

- 动态代理是指在程序运行时生成代理类。

- 有两种实现方式：

  - JDK 动态代理，被代理对象必须实现接口，利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理；
  - 字节码实现（比如说cglib/asm等），得用ASM开源包，将代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

- Java动态代理的优势是实现无侵入式的代码扩展，也就是方法的增强；让你可以在不用修改源码的情况下，增强一些方法；在方法的前后你可以做你任何想做的事情（甚至不去执行这个方法就可以）。

  | 代理方式      | 实现                                                         | 优点                                                         | 缺点                                                         | 特点                                                       |
  | :------------ | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :--------------------------------------------------------- |
  | JDK静态代理   | 代理类与委托类实现同一接口，并且在代理类中需要硬编码接口     | 简单粗暴                                                     | 代理类需要硬编码接口，在实际应用中可能会导致重复编码，浪费存储空间并且效率很低 |                                                            |
  | JDK动态代理   | 代理类与委托类实现同一接口，主要是通过代理类实现InvocationHandler并重写invoke方法来进行动态代理的，在invoke方法中将对方法进行增强处理 | 不需要硬编码接口，代码复用率高                               | 只能够代理实现了接口的委托类                                 | 底层使用反射机制进行方法的调用                             |
  | CGLIB动态代理 | 代理类将委托类作为自己的父类并为其中的非final委托方法创建两个方法，一个是与委托方法签名相同的方法，它在方法中会通过super调用委托方法；另一个是代理类独有的方法。在代理方法中，它会判断是否存在实现了MethodInterceptor接口的对象，若存在则将调用intercept方法对委托方法进行代理 | 可以在运行时对类或者是接口进行增强操作，且被代理的类无需实现接口 | 不能对final类以及final方法进行代理                           | 底层将方法全部存入一个数组中，通过数组索引直接进行方法调用 |

  ---

## 2.集合

### Map

**（1）HashMap 的存取数据的过程是什么样的？**

- put()方法

1. 对 key 的 hashCode () 进行 hash 后计算数组获得下标 index;
```java
//获取key的hashCode，这个值是一个固定的int值，java6、7默认是返回随机数，java8默认是通过和当前线程有关的一个随机数+三个确定值，运用Marsaglia’s xorshift scheme随机数算法得到的一个随机数
int hash=key.hashCode();

// 获取数组下标：key的hash值对Entry数组长度进行取余
int index=hash%Entry[].length;
Entry[index]=value。
```

2. 如果当前数组为 null，进行容量的初始化，初始容量为 16；

3. 如果 hash 计算后没有碰撞，直接放到对应数组下标里；

4. 如果 hash 计算后发生碰撞且节点已存在，则替换掉原来的对象；

5. 如果 hash 计算后发生碰撞且节点已经是树结构，则挂载到树上。

6. 如果 hash 计算后发生碰撞且节点是链表结构，则添加到链表尾（JDK1.7是头插法，JDK1.8是尾插法），并判断链表是否需要转换成树结构（默认大于 8 的情况会转换成树结构）；

7. 完成 put 后，是否需要 resize () 操作（数据量超过 threshold，threshold 为初始容量和负载因子之积，默认为 12）。

> JKD1.7，5/6是合并的，即如果发生哈希碰撞且节点是链表结构，则放在链表头。

- get()方法

```java
int hash = (key == null) ? 0 : hash(key);
for (Entry<K,V> e = table[indexFor(hash, table.length)];e != null;e = e.next) {    
		Object k;    
  	if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))  
  			return e;
		}
		return null;
}
```

1.根据`key`的`hashcode`算出元素在数组中的下标;

2.遍历`Entry`对象链表，使用Entry的hash和key进行对比，直到找到元素返回。

**（2）HashMap初始容量，最大容量，扩容？**

- 初始容量为16

```java
// 建议初始化容量initialCapacity = (需要存储的元素个数/负载因子) + 1;
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
```

- 最大容量为2的30次幂

```java
static final int MAXIMUM_CAPACITY = 1 << 30;
```

> 该static final 类型的静态变量为int类型，原因是由于考虑到HashMap的性能问题而作的折中处理.
>
> 由于int类型限制了该变量的长度为4个字节共32个二进制位，按理说可以向左移动31位即2的31次幂。但是事实上由于二进制数字中最高的一位也就是最左边的一位是符号位，用来表示正负之分（0为正，1为负），所以只能向左移动30位，而不能移动到处在最高位的符号位。

- 默认负载因子为0.75；

```java
  static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

- HashMap的容量一定是2的次幂，若指定了初始容量，则大小为距离指定参数最近的2的整数次幂，例如7->8, 8->8, 9->16, 17->32；
- 默认当元素个数超过(容量 * 负载因子)时进行扩容，扩容为原Entry数量的两倍，

> 创建一个新的Entry空数组，长度是原数组的2倍；
>
> 遍历原Entry数组，把所有的Entry重新Hash到新数组，因为长度扩大以后，Hash的规则也随之改变。
>
> HashMap通过计算(n - 1) & hash来确定key的索引位置，当HashMap的容量是2的n次幂时，n - 1的后几位数都是为1，如15的二进制后四位为1111，这样与1或0进行与运算时，得到的结果可能为1或0，使计算出的值更均匀分布。

**（3）HashMap 初始容量设置为 10000 时，放入 10000 条数据是否需要扩容；如果初始容量设置为 1000 时，放入 1000 条数据是否需要扩容？**

- 初始容量设置为10000时，hashmap的数组长度应该为16384（大于10000的2的幂次方），加载因子0.75，则threshold为12288>10000,所以不需要扩容；当初始容量设置为1000时，hashamp的数组长度应该为1024，threshold为768<1000,所以需要扩容。

**（4）JDK8之后HashMap链表转红黑树的条件和时机是什么，红黑树转链表的条件和时机是什么，为什么这么设计？**

- 当链表元素个数达到8个时，且桶数组容量大于等于64时，再插入元素时，先将插入的元素链接到链表尾部，然后对链表转红黑树。

- 若链表元素个数小于等于6时，先将元素删除，然后树结构还原成链表。

- 因为红黑树的平均查找长度是log(n)，长度为8的时候，平均查找长度为3，如果继续使用链表，平均查找长度为8/2=4，这才有转换为树的必要。链表长度如果是小于等于6，6/2=3，虽然速度也很快的，但是转化为树结构和生成树的时间并不会太短。

  中间差值7可以有效防止链表和树频繁转换。假设链表个数超过8则链表转换成树结构，链表个数小于8则树结构转换成链表，如果一个HashMap不停的插入删除元素，链表个数在8左右徘徊，就会频繁的发生树转链表、链表转树，效率会很低。
``` java
/**
  * 使用红黑树(而不是链表)来存放元素。当向至少具有这么多节点的链表再添加元素时，链表就将转换为红黑树。
  * 该值必须大于2，并且应该至少为8，以便于删除红黑树时转回链表。
  */
static final int TREEIFY_THRESHOLD = 8;

/**
  * 当元素个数小于等于6时，红黑树转回链表。
  */
static final int UNTREEIFY_THRESHOLD = 6;
/**
  * 当桶数组容量小于该值时，优先进行扩容，而不是树化：
  */
static final int MIN_TREEIFY_CAPACITY = 64;
```

**（5）Entry的结构是什么？**

```java
final int hash;// Key的Hash值
final K key;// 键对象
V value;// 值对象
Node<K,V> next;// 下一个Entry对象引用
```

**（6）JDK8为什么将链表的头插法改成尾插法？**

- 链表太长就需要扩充数组长度进行rehash减少链表长度。如果两个线程同时触发扩容，在移动节点时会导致一个链表中的2个节点相互引用，从而生成环链表，尾插法防止这种情况的发生，因为扩容转移后前后链表顺序不变，保持之前节点的引用关系。

**（7）HashMap 和 HashTable 有什么区别？**

HashMap 是 JDK1.2 才出现的；HashTable 是 JDK1.0 就出现的。JDK 里面也说了 HashMap 可以大致相当于 HashTable（The `HashMap` class is roughly equivalent to `Hashtable`, except that it is
unsynchronized and permits nulls）。具体差异：

- HashMap 是线程不安全的，HashTable 是线程安全的。
- HashMap 的键需要重新计算对象的 hash 值，而 HashTable 直接使用对象的 hashCode。
- HashMap 的值和键都可以为 null，HashTable 的值和键都不能为 null。
- HashMap 的数组的默认初始化大小为 16，HashTable 为 11；HashMap 扩容时会扩大两倍，HashTable 扩大两倍 + 1；

**（8）HashMap、LinkHashMap、TreeMap的区别是什么？**

HashMap：数据无序，根据键的hashCode进行数据存取，访问速度快，适合插入、删除、定位元素；

TreeMap：有序的，底层为红黑树，适合按照自定义顺序或者自然顺序存储数据；

LinkedHashMap：HashMap的子类，底层维护一个双向链表，适合实现输入顺序与输出顺序相同的需求；

**（9）解释一下ConcurrentHashMap如何实现线程安全？与HashTable的实现方式有什么不同？**

- ConcurrentHashMap采用分段锁技术，主干是Segment数组，由多个Segment 链式组成，因此每个Segment都持有自己的锁，实现部分数据锁定，每一把锁用于锁容器其中一部分数据，多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争。
- HashTable是全部锁定，将全部容器加锁，效率低于ConcurrentHashMap。
- Segment使用的是reentrantLock（可重入锁），而HashTable使用的是synchronized（同步锁）。
- JDK1.7 中，ConcurrentHashMap 采用 HashEntry+Segment 的结构，ConcurrentHashMap 里默认有 16 个 Segment，Segment 是可重入锁 ReentrantLock 的子类，每个 Segment 对应一个 HashEntry 键值对数组。当对 HashEntry 数组的数据进行修改时，必须首先获得对应的 Segment 锁，因此，多线程访问容器里不同 Segment 的数据，就不会存在锁竞争，从而提升并发性能。
- JDK1.8 中则摒弃了 Segment 的概念，并发控制使用 synchronized 和 CAS 来操作，虽然在 JDK1.8 中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本。来看看核心的 put 方法。JDK1.8 中的 ConcurrentHashMap 主要通过小范围的加锁 (synchronizded) 以及大量的 CAS 操作来实现 put 方法的线程安全；而 get () 方法则没有加锁。

>不可重入锁：只判断这个锁有没有被锁上，只要被锁上申请锁的线程都会被要求等待；
>
>可重入锁：不仅判断锁有没有被锁上，还会判断锁是谁锁上的，当就是自己锁上的时候，那么他依旧可以再次访问临界资源，并把加锁次数加一；
>
> 设计了加锁次数以在解锁的时候可以确保所有加锁的过程都解锁了，其他线程才能访问。不然没有加锁的参考值，也就不知道什么时候解锁？解锁多少次？才能保证本线程已经访问完临界资源了可以唤醒其他线程访问了；
>
>这个重入的概念就是，拿到锁的代码能不能多次以不同的方式访问临界资源而不出现死锁等相关问题。经典之处在于判断了需要使用锁的线程是否为加锁的线程。如果是，则拥有重入能力。

**（10）如果想保证HashMap数据的有序性，该怎么做？数据结构是怎样的？**

- 可以使用LinkedHashMap；
- LinkedHashMap添加了一个记录Key插入顺序的链表，通过访问这个链表中key指向的Entry，实现有序遍历；

**（11）为什么 ConcurrentHashMap 的 key 和 value 不能为 null？为什么 HashMap 可以呢？**

- 会产生二义性：这个key从来没有在map中映射过；这个key的value在设置的时候，就是null；

- HashMap在单线程中可以用hashMap.containsKey(key)方法来区分含义；

- 线程A调用concurrentHashMap.get(key)方法，有一个线程B执行了concurrentHashMap.put(key,null)的操作，于是无法区分含义；

### List和Set

**（1）ArrayList和LinkedList的相同点和不同点是什么？**

- 相同点：ArrayList 和 LinkedList 都是 List 接口的实现类，因此都具有 List 的特点，即存取有序，可重复；而且都不是线程安全的。

- 不同点：ArrayList是实现了基于动态数组的数据结构，内存连续，LinkedList基于双向链表的数据结构。

  > 对于随机访问get和set，ArrayList一般优于LinkedList，因为LinkedList要移动指针。
  >
  > 对于新增和删除操作add和remove，LinkedList比较占优势，因为ArrayList要移动数据。
  >
  > ArrayList 基于数组存储数据，因此查询元素时可以直接按照数据下标进行索引，而插入元素时，通常涉及到数据元素的复制和移动，所以查询数据快而插入数据慢；
  >
  > LinkedList 基于双向链表存储数据，因此查询元素时需要前向或后向遍历，而插入数据时只需要修改本元素的前后项即可，所以查询数据慢而插入数据快。
  >
  > 所以，ArrayList 适合查询多（读多）的场景，LinkedList 适合插入多（写多）的场景。

**（2）Array和ArrayList的不同点是什么？**

- Array可以包含基本类型和对象类型，ArrayList只能包含对象类型。
- Array大小是固定的，ArrayList的大小是动态变化的。
- ArrayList提供了更多的方法和特性，比如：addAll()，removeAll()，iterator()等等。
- 对于基本类型数据，集合使用自动装箱来减少编码工作量，当处理固定大小的基本数据类型的时候，这种方式相对比较慢。

**（3）ArrayList的扩容机制是什么？**

- 当前数组是由默认构造方法生成的空数组并且第一次添加数据，此时minCapacity等于默认的容量（10），而后的数组扩容才是按照当前容量的1.5倍进行扩容。

**（4）ArrayList和Vector有何异同点？**

相同点：

- 两者都是基于索引的，内部由一个数组支持。

- 两者维护插入的顺序，我们可以根据插入顺序来获取元素。

- ArrayList和Vector的迭代器实现都是fail-fast的。

- ArrayList和Vector两者允许null值，也可以使用索引值对元素进行随机访问。

不同点：

- Vector是同步的，而ArrayList不是。然而，如果你寻求在迭代的时候对列表进行改变，你应该使用CopyOnWriteArrayList。

- ArrayList比Vector快，它因为有同步，不会过载。

- ArrayList更加通用，因为我们可以使用Collections工具类轻易地获取同步列表和只读列表。

**（5）Java集合类中的Iterator和ListIterator的区别？**

- iterator()方法在set和list接口中都有定义，但是ListIterator()仅存在于list接口中（或实现类中）；
- ListIterator有add()方法，可以向List中添加对象，而Iterator不能；
- ListIterator和Iterator都有hasNext()和next()方法，可以实现顺序向后遍历，但是ListIterator有hasPrevious()和previous()方法，可以实现逆向（顺序向前）遍历。Iterator就不可以；
- ListIterator可以定位当前的索引位置，nextIndex()和previousIndex()可以实现。Iterator没有此功能；
- 都可实现删除对象，但是ListIterator可以实现对象的修改，set()方法可以实现。Iierator仅能遍历，不能修改；

**（6）Java 集合的快速失败（fail-fast）和安全失败（fail-safe）的差别是什么？**

- 快速失败和安全失败都是 java 集合（Collection）的一种错误机制。单线程情况下，遍历集合时去执行增删等改变集合结构的操作；或者多线程情况下，一个线程遍历集合，另一个线程执行增删等改变集合结构的操作。
- 快速失败，是指失败 / 异常时立即报错，通常会抛出 ConcurrentModificationException 异常，像 java.util 包下面的集合类就是使用这种机制；
- 安全失败，是指失败 / 异常时直接忽略，java.util.concurrent 包下面的集合类都是使用这种机制。

> 快速失败的原因在于，每当迭代器在进行增删等操作时，会使用 hashNext () /next () 进行元素遍历，而元素遍历之前都会检测 modCount 变量是否为 expectedmodCount 的值，是的话就返回遍历，否则抛出异常 ConcurrentModificationException，终止遍历。
>
> 安全失败的处理方式则有两种：一是 CopyOnWriteArrayList/CopyOnWriteArraySet 这类集合，底层增删时会复制数组，如果增删操作前遍历数组，则会遍历复制前的老视图，二者并不冲突；二是 ConcurrentHashMap 这些并发集合，这些集合不存在 expectedmodCount，Iterator 也不会做相应的检查。

**（7）HashSet和HashMap的关系是怎样的？**

- HashSet本质上是用HashMap实现的；

  >``` java
  >public boolean add(E e) {
  >	return map.put(e, PRESENT)==null;
  >}
  >```
  >
  >HashSet的put方法，本质上就是通过HashMap的put方法是否返回null来判断是否插入成功；

## 3.JVM

### 内存结构

**（1）JVM的内存结构及用处是什么？**

JDK1.8之前JVM 内存的主要分为五个区：方法区（Method Area），虚拟机栈（VM Stack），本地方法栈（Native method stack），堆（Heap），程序计数器（Program Counter Register）。

**线程共享：**

- 堆

  > 在虚拟机启动时创建，几乎所有的对象实例都在这里创建，是垃圾收集器管理的主要区域；

- 方法区

  > 主要用来存储 JVM 加载的类信息，包括类的方法（如类的接口 / 父类等）、常量、静态变量、即时编译器编译后的代码等数据，还包括运行时常量池（Runtime Constant Pool），用于存放静态编译产生的字面量和符号引用；
  >
  > 很少发生 GC（Garbage Collection，垃圾回收），偶尔发生的 GC 主要是对常量池回收和类型的卸载；

**线程私有：**

- 虚拟机栈

  > 又被称为栈内存，每个方法在执行的时候都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接和方法出口等信息，每一个方法被调用直至执行完成的过程，就对应着一个栈桢在虚拟机栈中从入栈到出栈的过程；

- 本地方法栈

  > 类似于虚拟机栈，不过本地方法栈为 Native 方法服务，而虚拟机栈为 java 方法服务；

- 程序计数器

  > 内存空间小，字节码解释器工作时通过改变这个计数值可以选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理和线程恢复等功能都需要依赖这个计数器完成。该内存区域是唯一一个 java 虚拟机规范没有规定任何 OOM 情况的区域；

**JDK1.8及之后方法区分成了元数据区和直接内存**

- 元数据区

  >替代永久代，存放类信息、常量、静态变量等，字符串在1.7开始放到了堆中，而不是方法区；
  >
  >为了解决永久代大小不足导致的问题；

### 类加载

**（1）JVM 的类加载机制是什么样的？有几类加载器？**

- JVM 通过双亲委派模型进行类的加载，即当某个类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载。

-  3 类加载器：
- **启动类加载器 (Bootstrap ClassLoader)**：负责加载 JAVA_HOME\lib 目录中的，或通过 - Xbootclasspath 参数指定路径中的，且被虚拟机认可（按文件名识别，如 rt.jar，名字不符合的类库即使放在 lib 目录也不会被加载）的类。启动类加载器无法被 Java 程序直接引用；
  - **扩展类加载器 (Extension ClassLoader)**：负责加载 JAVA_HOME\jre\lib\ext 目录中的，或通过 java.ext.dirs 系统变量指定路径中的类库；
  - **应用程序类加载器 (Application ClassLoader)**：负责加载用户路径（classpath）上的类库。
> 除此之外，还可以通过继承 java.lang.ClassLoader 类实现自己的类加载器（主要是重写 findClass 方法）。

 **（2）双亲委派模型解决了什么问题，有什么好处？**

  - 基础类的统一加载问题（越基础的类由越上层的加载器进行加载）。如类 java.lang.String，无论哪一个类加载器要加载这个类，最终都是委派给启动类加载器进行加载，所以在程序的各种类加载器环境中都是同一个类。
  - 提高 java 代码的安全性。比如说用户自定义了一个与系统库里同名的 java.lang.String 类，那么这个类就不会被加载，因为最顶层的类加载器会首先加载系统的 java.lang.String 类，而不会加载自定义的 String 类，防止了恶意代码的注入。

  > 类加载还有一个比较重要的知识点是类加载过程。一个类的生命周期可以分为七个阶段：**加载、验证、准备、解析、初始化、使用、卸载**。其中前五个阶段即**类加载**。

### 垃圾回收

**（1）常见的垃圾回收算法有哪些？**

- **复制（Coping）算法**

  - 将可用内存按容量划分为相等的两部分，每次只使用其中的一块，当一块内存用完时，就将还存活的对象复制到第二块内存上，然后一次性清除第一块内存，再将第二块上的对象复制到第一块。
  - 实现方便，运行高效，不用考虑内存碎片，但是内存利用率只有一半。

- **标记 - 清除（Mark-Sweep）算法**

  - 分为**标记**和**清除**两个阶段。首先标记出所有需要回收的对象，在标记完成后统一回收被标记的对象。
  - 算法简单，但是有两个缺点：
    - 1、效率不高，标记和清除的效率都很低；
    - 2、空间问题，会产生大量不连续的内存碎片，导致以后程序在分配较大的对象时，由于没有充足的连续内存而提前触发一次 GC 动作。

- **标记 - 压缩（Mark-Compact）算法**

  - 又称**标记 - 整理算法**，标记过程仍然与 “标记 - 清除” 算法一样，但不是直接对可回收对象进行清理，而是让所有存活的对象向一端移动，然后直接清理掉边界以外的内存，形成一版连续的内存区域。
  - 解决标记 - 清除算法产生的大量内存碎片问题；当对象存活率较高时，也解决了复制算法的空间效率问题，不过它本身也存在时间效率方面的问题。

- **分代收集（Generational Collection）算法**

  - 根据对象的生存周期，将堆分为新生代和老年代，然后根据各个年代的特点采用最适当的收集算法。在新生代中，由于对象生存期短，每次回收都会有大量对象死去，那么这时就采用**复制**算法。老年代里的对象存活率较高，没有额外的空间进行分配担保，所以可以使用**标记 - 整理** 或者 **标记 - 清除**。

  - 严格地说，这并非是一种算法，而是一种思想，或者说是一种复合算法。

    > 内存区域按照 8:1:1 分为三部分（可以通过参数 - XX:SurvivorRatio 和 - XX:InititalSurvivorRatio 来进行调整），较大的内存区域称为 Eden，其余两块较小的内存区域称为 Survior（见下图）。
    > ![图片描述](https://img1.sycdn.imooc.com/5e6afe690001b58412440262.jpg)
    >
    > 当回收时，将 Eden 和 Survivor 中还存活的对象一次性拷贝到另外一块 Survivor 空间上，最后清理掉 Eden 和刚才用过的 Survivor 空间。这样内存空间的利用率有 90%(80% + 10%)，只 有 10% 的内存是会被 “浪费” 的。

**（2）JDK1.8前后JVM中堆的区别是什么？**

![img](https://pic2.zhimg.com/80/v2-cb25f0d4c52a9ffca1925e9694c6954d_1440w.jpg)



- JDK1.8之前堆内存的分为新生代、老年代和永久代。新生代又被进一步分为：Eden 区＋Survior1 区＋Survior2 区。JDK 1.8中移除整个永久代，取而代之的是一个叫元空间（Metaspace）的区域（永久代使用的是JVM的堆内存空间，而元空间使用的是物理内存，直接受到本机的物理内存限制）。

**（3）什么样的对象进入新生代？什么样的对象进入老年代？**

-  **新生对象优先在eden区分配。**当Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC；

-  **大对象（需要大量连续内存空间的对象，如字符串、数组等）直接进入老年代。**可以避免为大对象分配内存时由于分配担保机制带来的复制而降低效率；

-   **长期存活的对象将进入老年代。**虚拟机给每个对象一个对象年龄（Age）计数器，如果对象在 Eden 出生并经过第一次 Minor GC 后仍然能够存活，并且能被 Survivor 容纳的话，将被移动到 Survivor 空间中，并将对象年龄设为1，对象在 Survivor 中每熬过一次 MinorGC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

- **动态对象年龄判定。**为了更好的适应不同程序的内存情况，虚拟机不是永远要求对象年龄必须达到了某个值才能进入老年代，如果 Survivor 空间中相同年龄所有对象大小的总和大于 Survivor 空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无需达到要求的年龄。

  > 新生代GC（Minor GC）:指发生新生代的的垃圾收集动作，Minor GC非常频繁，回收速度一般也比较快，每次进行清理时，将Eden区和一个Survivor中仍然存活的对象拷贝到 另一个Survivor中，然后清理掉Eden和刚才的Survivor。控制在40ms以内。
  >
  > 老年代GC（Major GC）:指发生在老年代的GC，出现了Major GC经常会伴随至少一次的Minor GC（并非绝对），Major GC的速度一般会比Minor GC的慢10倍以上；控制在100ms以内。
  >
  > Full GC：当新生代和老年代空间都不足时触发，控制在1000ms以内。
  >
  > 出了cms和g1外，其余的major gc = full gc。


**（4）你们常用的垃圾收集器及特点和工作过程是什么？**

- 新生代的垃圾回收器是 ParNewGC，老年代的回收器是 ConcMarkSweepGC（CMS，并发标记清除 GC)。JDK1.9之后使用G1收集器。

> ParNew 是串行收集器（Serial GC）的多线程版本，会使用多个 CPU 和线程完成垃圾收集工作（默认使用的线程数和 CPU 数相同，可以使用 - XX：ParallelGCThreads 进行调整）。
>
> CMS 是一种以最短回收停顿时间为目标的收集器，基于标记 - 清除（Mark-Sweep）算法实现的，主要针对老年代进行回收，其处理过程有七个步骤：
>
> 1. **初始标记 (CMS-initial-mark)** ，从 GC Roots 开始，扫描和 GC Roots 直接关联的对象并标记。该步骤会导致 **STW**(Stop The World，即虚拟机暂停正在执行的任务)；
> 2. **并发标记 (CMS-concurrent-mark)**，从步骤 1 中标记过的对象出发，使用递归，所有可到达的对象都在本阶段中标记， **该阶段与用户线程同时运行**；
> 3. **并发预清理（CMS-concurrent-preclean）**，标记从**新生代晋升的对象**、**新分配到老年代的对象**以及在**并发阶段被修改了的对象**。通过重新扫描，减少后面第 5 步 "重新标记" 的工作，**该阶段与用户线程同时运行**；
> 4. **可被终止的预清理（CMS-concurrent-abortable-preclean）**，这个阶段会尽量尝试着承担 STW 的 Final Remark 阶段的工作。其持续的时间依赖因素较多（通常持续时间较长），因为它是重复做相同的事情直到发生 aboart 的条件（比如：重复的次数、多少量的工作、持续的时间等等）之一才会停止，**该阶段与用户线程同时运行**；
> 5. **重新标记 (CMS-remark)** ，收集器线程扫描在 CMS 堆中剩余的对象并进行标记， **是第二个并且是最后一个 STW 的阶段**；
> 6. **并发清除 (CMS-concurrent-sweep)**，清理垃圾对象，**与用户线程同时运行；**
> 7. **并发重置 (CMS-concurrent-reset)**，这个阶段，重置 CMS 收集器的数据结构，等待下一次垃圾回收， **与用户线程同时运行。**

- G1收集器的思想是用空间换时间，不再局限于两个Survivor和一个Eden，而是采用N*N的网状内存区块，每个单元可以是Eden也可以是Survivor，当进行垃圾回收的时候，G1会判断回收哪块Survivor获得的效益最高（时间、大小），从而更有利于优化垃圾回收策略。

**（5）常用的JVM参数配置有哪些？**

| JVM 参数                                                     | 说明                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Xms                                                          | 初始堆大小                                                   |
| Xmx                                                          | 最大堆大小（一般与Xms一样，因为频繁扩缩容不利于CPU运算）     |
| Xmn                                                          | 年轻代大小（表示将NewSize与MaxNewSize设为一致，同时也是为了防止频繁扩缩容） |
| Xss                                                          | 每个线程的堆栈大小                                           |
| MetaspaceSize                                                | 首次触发 Full GC 的阈值，该值越大触发 Metaspace GC 的时机就越晚 |
| MaxMetaspaceSize                                             | 设置 metaspace 区域的最大值                                  |
| +UseConcMarkSweepGC                                          | 设置老年代的垃圾回收器为 CMS                                 |
| +UseParNewGC                                                 | 设置年轻代的垃圾回收器为并行收集                             |
| CMSFullGCsBeforeCompaction=5                                 | 设置进行 5 次 full gc（CMS）后进行内存压缩。由于并发收集器不对内存空间进行压缩 / 整理，所以运行一段时间以后会产生 "碎片"，使得运行效率降低。此值设置运行多少次 full gc 以后对内存空间进行压缩 / 整理 |
| +UseCMSCompactAtFullCollection                               | 在 full gc 的时候对内存空间进行压缩，和 CMSFullGCsBeforeCompaction 配合使用 |
| +DisableExplicitGC                                           | System.gc () 调用无效                                        |
| -verbose:gc                                                  | 显示每次 gc 事件的信息                                       |
| +PrintGCDetails                                              | 开启详细 gc 日志模式                                         |
| +PrintGCTimeStamps                                           | 将自 JVM 启动至今的时间戳添加到 gc 日志                      |
| -Xloggc:/home/admin/logs/gc.log                              | 将 gc 日导输出到指定的 /home/admin/logs/gc.log               |
| +HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/admin/logs | 当堆内存空间溢出时输出堆的内存快照到指定的 /home/admin/logs  |

**（6）什么时候CMS会触发Full GC？如何调优？**

- Promotion failure：由于内存碎片导致的晋升空间不足；
- Concurrent mode failed：还未完成cms又触发了下一次major gc；

| 参数                                  | 解释                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| -XX:ParallerGCThreads=N               | 设置年轻代的并行收集线程数，避免Docker问题（默认获取物理机核数） |
| -XX:ParallelCMSThreads=N              | 设置cms的并行收集线程数                                      |
| -XX:+UseCMSCompactAtFullCollection    | FullGC情况下的Initial remark Or Final remark都整理内存碎片   |
| -XX:+CMSFullGCsBeforeCompacion=4      | 两次FullGC情况下的Initial remark Or Final remark 才整理内存碎片 |
| -XX:+UseCMSInitiatingOccupancyOnly    | 让阈值驱动cms触发时机                                        |
| -XX:CMSInitiatingOccupancyFraction=70 | 70%占满才触发cms，结合上一个参数                             |
| -XX:+CMSParallelRemarkEnabled         | 并行remark                                                   |
| -XX:+CMSScavengeBeforeRemark          | remark前先做一次minor gc                                     |



## 4.多线程和并发

### 锁

**（1）synchronized 、volatile、 java.util.concurrent.locks.Lock 的异同是什么？***

|               | synchronized        | volatile            | java.util.concurrent.locks.Lock                              |
| :-----------: | ------------------- | ------------------- | ------------------------------------------------------------ |
|   修饰范围    | 变量、方法、类      | 变量                | 代码块，更加灵活                                             |
|     性能      | 低                  | 高                  | 高                                                           |
|    释放锁     | 自动                | /                   | 手动                                                         |
|    中断性     | 不可中断            | /                   | 可以中断，tryLock (long timeout, TimeUnit unit)/ interrupt () |
|    公平性     | 非公平锁            | /                   | 默认非公平锁，构造方法可传入true 代表公平锁，false 代表非公平锁 |
|    原子性     | 保证                | 不保证              | 保证                                                         |
|     阻塞      | 可能阻塞            | 不会阻塞            | 可能阻塞                                                     |
|  编译器优化   | 可以优化            | 不会优化            | /                                                            |
| 有序性/重排序 | 有序/不能阻止重排序 | 有序/可以阻止重排序 | 有序/不能阻止重排序                                          |

> - Synchronized属于 JVM 层面，底层通过 monitorenter 和 monitorexit 完成，依赖于 monitor（监视器） 对象来完成；
> - Lock 是 java.util.concurrent.locks.lock 包下的，是 JDK1.5 以后引入的新 API 层面的锁；
> - volatile关键字用来修饰变量，主要强调变量的内存可见性，告诉线程从内存中读取变量值，synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住；
> - 一个被volatile修饰的变量，在每次数据变化之后，会向CPU发送Lock指令，Lock指令将当前处理器缓存行的数据写回系统内存，并且会使其他CPU里缓存了该地址的数据无效化。而其他处理器的缓存由于遵守了**缓存一致性协议**，会把这个变量的值从主存加载到自己的缓存中。这就保证了一个volatile在并发编程中，其值在多个缓存中是可见的。

### 线程和线程池

**（1）线程有哪几种状态？**

- **初始状态 (NEW)** ：尚未启动的线程处于此状态。通常是新创建了线程，但还没有调用 start () 方法；
- **运行状态 (RUNNABLE)**：Java 线程中将就绪（ready）和运行中（running）两种状态笼统的称为 "运行中"。比如说线程可运行线程池中，等待被调度选中，获取 CPU 的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得 CPU 时间片后变为运行中状态（running）。
- **阻塞状态 (BLOCKED)**：表示线程阻塞于锁；
- **等待状态 (WAITING)**：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）；
- **超时等待状态 (TIMED_WAITING)**：进入该状态的线程需要等待其他线程在指定时间内做出一些特定动作（通知或中断），可以在指定的时间自行返回；
- **终止状态 (TERMINATED)**：表示该线程已经执行完毕，已退出的线程处理此状态。

**（2）为什么要使用线程池？如何使用？有哪些核心参数？初始化线程池的大小的如何算？shutdown 和 shutdownNow 有什么区别？**

- 使用线程池主要是为了**降低资源消耗、提高响应速度、提高线程的可管理性**；

- 可以使用工具类 Executors生成常用的线程池；

> newSingleThreadExecutor：创建一个单线程的线程池。如果该线程因为异常而结束，那么会有一个新的线程来替代它。
>
>  newFixedThreadPool：创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大值，一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。
>
> newCachedThreadPool：创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（默认 60 秒不执行任务）的线程。当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说 JVM）能够创建的最大线程大小。
>
> newScheduledThreadPool：创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。

- 主要包括七个参数：

```java
public ThreadPoolExecutor(  
  int corePoolSize,   //常驻线程数,即使空闲时仍保留在池中的线程数  
  int maximumPoolSize, //线程池中允许的最大线程数   
  long keepAliveTime,   // 存活时间。线程数比corePoolSize多且处于闲置状态的情况下，这些闲置的线程能存活的最大时间，为0表示会立即回收；  
  TimeUnit unit,  //keepAliveTime的单位  
  BlockingQueue<Runnable> workQueue, //被提交尚未被执行的任务阻塞队   
  ThreadFactory threadFactory, // 创建线程的工厂  
  RejectedExecutionHandler handler  // 饱和拒绝策略，当队列满了并且线程个数达到maximunPoolSize后采取的策略。目前支付四种:AbortPolicy(抛出异常)，CallerRunsPolicy(调用者线程处理)，DiscardOldestPolicy(直接丢弃任务，不予处理也不抛出异常)，DiscardPolicy(默默丢弃,不抛出异常)  
 ) 
```

- 可根据线程池中的线程处理任务的不同进行初始化大小估计：

> CPU密集型任务，这类任务需要大量的运算，通常 CPU 利用率很高，无阻塞，因此应配置尽可能少的线程数量，可设置为 CPU 核数 + 1；
>
> IO密集型任务，这类任务有大量 IO 操作，伴随着大量线程被阻塞，可配置更多的线程数，通常可设置 CPU 核心数 * 2；

- shutdown有序停止。**停止接收外部提交的任务，先前提交的任务务会执行（但不保证完成执行）**；
- shutdownNow尝试立即停止。**停止接收外部提交的任务，不再处理队列里等待的任务，忽略队列里等待的任务 返回正在等待执行的任务列表。**

> shutdownNow 试图取消线程的方法是通过调用 Thread.interrupt () 方法来实现的，非强制的，如果线程中没有 sleep/wait 等应用，interrupt () 方法是无法中断当前的线程的。所以，ShutdownNow 并不代表线程池就一定立即就能退出，它也可能必须要等待所有正在执行的任务都执行完成了才能退出，但是大多数时候是能立即退出的

**（3）ThreadLocal 的作用是什么？有什么风险？**

- 提供每个线程存储自身专属的局部变量。

> 每个 Thread 维护着一个 ThreadLocalMap 的引用，ThreadLocalMap 是 ThreadLocal 的内部类，用 Entry 来进行存储。
>
> 调用 ThreadLocal 的 set () 方法时，实际上就是往 ThreadLocalMap 设置值，key 是 ThreadLocal 对象，值是传递进来的对象；调用 ThreadLocal 的 get () 方法时，实际上就是往 ThreadLocalMap 获取值，key 是 ThreadLocal 对象，ThreadLocal 本身并不存储值，它只是作为一个 key 来让线程从 ThreadLocalMap 获取 value。因为这个原理，所以 ThreadLocal 能够实现 “数据隔离”，获取当前线程的局部变量值，不受其他线程影响。

- **内存泄漏**。ThreadLocal 被 ThreadLocalMap 中的 entry 的 key 弱引用，若ThreadLocal 没有被强引用， 那么 GC 时 Entry 的 key 就会被回收，但是对应的 value 却不会回收。就会造成内存泄漏。  所以每次使用完 ThreadLocal，应调用 remove () 方法，清除数据。

## 5.IO

### AIO-BIO-NIO

**（1）解释下AIO、BIO、NIO是什么意思？**

- BIO：同步阻塞式IO，例如文件系统的Read和Write，其他操作会被阻塞，直到之前的IO操作完成；
- NIO：引入了同步非阻塞流程，不断判断数据的状态，状态改变后进行IO操作；
- AIO：异步非阻塞，发送之后等待回调；

## 6.计算机网络

**（1）服务器返回给客户端 http 响应包的状态码有哪几大类？常见的状态码有哪些，是什么意思？**

- 信息性状态码（**Informational**），表示请求已被接受，需要继续处理。**码值范围：1xx**。
- 成功状态码（**Success**），表示请求已成功被服务器接收、理解、并接受。**码值范围：2xx**。
- 重定向状态码 **（Redirection）** 。表示需要客户端采取进一步的操作才能完成请求。**码值范围：3xx**。
- 客户端错误状态码 **（Client Error）**。表示请求语法错误或者请求无法完成。**码值范围：4xx**。
- 服务器端错误状态码（**Server Error**）。表示服务器处理请求过程中发生错误或者无法执行。**码值范围：5xx**。

| 状态码 | 英文名                | 说明                                                         |
| :----- | :-------------------- | :----------------------------------------------------------- |
| 100    | Continue              | 表示迄今为止的所有内容都是可行的，客户端应该继续请求，如果已经完成，则忽略它。 |
| 101    | Switching Protocol    | 表示服务器正在切换协议。只能切换到更高级的协议，例如，切换到 HTTP 的新版本协议。 |
| 200    | OK                    | 请求已成功，请求所希望的响应头或数据体将随此响应返回。       |
| 202    | Accepted              | 服务器已接受请求，但尚未完成。最终该请求可能会也可能不会被执行，并且可能在处理发生时被禁止。 |
| 301    | Moved Permanently     | 永久性重定向。被请求的资源已永久移动到新 URI，返回信息会包括新的 URI，浏览器会自动定向到新 URI。如果可能，拥有链接编辑功能的客户端应当自动把请求的地址修改为从服务器反馈回来的地址 |
| 302    | Moved Temporarily     | 暂时性重定向。可以简单的理解为该资源原本确实存在，但已经被临时改变了位置；换而言之，就是请求的资源暂时驻留在不同的URI下。 |
| 304    | Not Modified          | 表示资源在由请求头中的 If-Modified-Since 或 If-None-Match 参数指定的这一版本之后，未曾被修改。表示客户端不用请求该资源，直接使用本地的资源即可。 |
| 400    | Bad Request           | 通常有两种情况：一是语义有误，当前请求无法被服务器理解；二是请求参数有误。 |
| 401    | Unauthorized          | 请求要求身份验证。 该响应必须包含一个适用于被请求资源的 WWW-Authenticate 信息头用以询问用户信息。客户端可以重复提交一个包含恰当的 Authorization 头信息的请求。如果当前请求已经包含了 Authorization 证书，那么 401 响应代表着服务器验证已经拒绝了那些证书。对于需要登录的网页，服务器可能返回此响应。 |
| 403    | Forbidden             | 服务器已经理解请求，但是拒绝执行它。                         |
| 404    | Not Found             | 服务器找不到请求的资源                                       |
| 405    | Method Not Allowed    | 禁用请求中指定的方法                                         |
| 500    | Internal Server Error | 服务器内部错误，不知道如何处理；                             |
| 502    | Bad Gateway           | 前面的网关代理服务器联系不到后端的服务器                     |
| 503    | Service Unavailable   | 服务器没有准备好处理请求。 常见原因是服务器因维护或重载而停机 |
| 504    | Gateway Timeout       | 前面的网关代理服务器能联系到后端的服务器，但是后端的服务器在规定的时间内没有给代理服务器响应 |

**（2）三次握手的过程是怎样的？**

- **第一次握手：**建立连接时,客户端发送syn包(syn=j)到服务器,并进入**SYN_SEND**状态,等待服务器确认；
> SYN：同步序列编号(Synchronize Sequence Numbers)
- **第二次握手：**服务器收到syn包,必须确认客户的SYN（ack=j+1）,同时自己也发送一个SYN包（syn=k）,即SYN+ACK包,此时服务器进入**SYN_RECV**状态；
- **第三次握手：**客户端收到服务器的SYN＋ACK包,向服务器发送确认包ACK(ack=k+1),此包发送完毕,客户端和服务器进入**ESTABLISHED**状态,完成三次握手。

**（3）select和epoll的区别？**

- select对于socket连接数组的大小有监听上线，一般是1024，限制了单机连接上线；
- select循环遍历轮训查找真正有信号的socket连接，浪费计算能力；
- epoll去除了1024数组个数的限制，socket连接会植入一个回调函数，如果socket连接产生了信号，则会调用回调函数，通知服务端处理；
- nginx之所以成为互联网研发领域最常用的应用服务器，很大程度归功于epoll模型的使用；

**（4）https加密是怎么实现的？**

- 浏览器发起往服务器的 443 端口发起请求，请求携带了浏览器支持的加密算法和哈希算法；

- 服务器收到请求，选择浏览器支持的加密算法和哈希算法；

- 服务器下将数字证书返回给浏览器，这里的数字证书可以是向某个可靠机构申请的，也可以是自制的；（注释：证书包括以下这些内容：1. 证书序列号。2. 证书过期时间。3. 站点组织名。4. 站点DNS主机名。5. 站点公钥。6. 证书颁发者名。7. 证书签名。因为证书就是要给大家用的，所以不需要加密传输）

  >签名是指利用上一层证书的私钥，加密一些元信息（证书所有者的信息，包括基本信息，公钥，证书生效域名等）。这样，当收到签名证书时，只需要根据CA提供的公钥对签名解密，验证元信息是否一致，就可以判断当前证书是否合法。一句话来说就是，每个证书会对下一层的证书合法性做担保 。

- 浏览器进入数字证书认证环节，这一部分是浏览器内置的 TSL 完成的；

  > 首先浏览器会从内置的证书列表中索引，找到服务器下发证书对应的机构，如果没有找到，此时就会提示用户该证书是不是由权威机构颁发，是不可信任的。如果查到了对应的机构，则取出该机构颁发的公钥。
  >
  > 用机构的证书公钥解密得到证书的内容和证书签名，内容包括网站的网址、网站的公钥、证书的有效期等。浏览器会先验证证书签名的合法性（验证过程类似上面 Bob 和 Susan 的通信）。
  >
  > 签名通过后，浏览器验证证书记录的网址是否和当前网址是一致的，不一致会提示用户。如果网址一致会检查证书有效期，证书过期了也会提示用户。这些都通过认证时，浏览器就可以安全使用证书中的网站公钥了。
  >
  > 浏览器生成一个随机数 R，并使用网站公钥对 R 进行加密。

- 浏览器将加密的 R 传送给服务器；
- 服务器用自己的私钥解密得到 R；
- 服务器以 R 为密钥使用了对称加密算法加密网页内容并传输给浏览器；
- 浏览器以 R 为密钥使用之前约定好的解密算法获取网页内容。

**（5）你对http2.0有什么了解？**

- http2.0有三个核心能力，分别为二进制传输、多路复用、服务端推送；

- 二进制传输提高了网络带宽的利用率，加大了传输的速率；

- http请求是串行化的，例如需要加载完html再加载css，http2.0通过包头加包体的形式，用包头存储信息的位置；

  >包1-4html包5-8css	|	html数据	|	css数据	|

- http2.0基于长连接实现了服务端推送，可以在服务端进行配置，实现包结果的额外相应；

  > 例如客户端请求index.html，服务端可以额外返回index.css和index.js；
  >
  > 传统的http需要一个个请求资源；

# 二、数据库

## 1.数据库基础

**（1）MySQL 支持哪些存储引擎？他们的特点是什么？**

- **InnoDB** 支持事务，行级锁定和外键，是事务型数据库的首选引擎；MySQL5.5.5 之后的默认存储引擎；

- **MyISAM** 拥有较高的插入、查询速度，但不支持事务。MySQL5.5.5 之前的默认存储引擎；

- **Memory** 基于散列，存储在内存中，对临时表有用。常见的应用场景是：临时存放数据，数据量不大，不需要较高的数据安全性；

- **Archive** 支持高并发的插入操作，但是本身不是事务安全的。常见的应用场景：存储归档数据，如记录日志信息可以使用 Archive。

  > InnoDB 和 MyISAM的区别：
  >
  > InnoDB 支持事务；而 MyISAM 不支持事务，强调的是性能，查询速度更快；
  >
  > InnoDB 支持行级锁和表级锁（默认行级锁），而 MyISAM 只支持表级锁；
  >
  > InnoDB 支持 MVCC, 而 MyISAM 不支持 MVCC；
  >
  > InnoDB 支持外键，而 MyISAM 不支持外键；
  >
  > InnoDB 早期版本不支持全文索引（从 MySQL5.6 开始支持全文索引），而 MyISAM 支持；
  >
  > InnoDB 不保存表的具体行数，count () 时要扫描一遍整个表来计算有多少行；MyISAM 则内置了一个计数器，count () 时它直接从计数器中读。

## 2.索引

**（1）MySQL为什么底层采用B+？为什么不用B树？**

- 索引文件很大，不能全部存储在内存中，只能存储到磁盘上，因此索引的数据结构要尽量减少查找过程中磁盘 I/O 的存取次数；

- 数据库系统利用了磁盘预读原理和磁盘预读，将一个节点的大小设为等于一个页，这样每个节点只需要一次 I/O 就可以完全载入。而 B + 树的高度是 2~4，叶子节点的高度不会超过1，检索一次最多只需要访问 4 个节点（4 次，即树的高度）。

  > B + 树索引并不能直接找到具体的行，只是找到被查找行所在的页，然后 DB 通过把整页读入内存（磁盘预读），再在内存中查找（局部性原理，即当一个数据被用到时，其附近的数据也通常会马上被使用）。

- B + 树所有的 Data 域在叶子节点，其余节点用来索引，而 B 树是每个索引节点都会有 Data 域；并且 B + 树所有叶子节点之间都有一个链指针。 这样遍历叶子节点就能获得全部数据，从而支持区域查询。在数据库中基于范围的查询是非常频繁的，而 B 树不支持这样的遍历操作。

![图片描述](https://img1.sycdn.imooc.com/5e7973590001a63513100450.png)
**B树**

![图片描述](https://img1.sycdn.imooc.com/5e7973670001603711940540.png)

**B+树**

**（2）MySQL 支持的索引类型是哪些？**

- **普通索引**：用表中的普通列构建的索引，没有任何限制；
- **唯一索引**：唯一索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一；
- **主键索引**：是一种特殊的唯一索引，根据主键建立索引，不允许重复，不允许空值；
- **全文索引**：通过过建立倒排索引，快速匹配文档的方式。MySQL 5.7.6 之前仅支持英文，MySQL 5.7.6 之后支持中文；
- **组合索引**：又叫联合索引。用多个列组合构建的索引，不允许有空值。可以在创建表的时候指定，也可以修改表结构。

> 聚集索引和非聚集索引
>
> **聚集索引 (clustered index)**，又称为主索引，该索引中键值的逻辑顺序决定了表中相应行的物理顺序。因为数据真正的数据只能有一种排序方式，所以一个表上只能有一个聚簇索引。
>
> **非聚集索引 (secondary index)**，又称为辅助索引、普通索引，该索引的逻辑顺序与磁盘上行的物理存储顺序不同，一个表可以包含多个非聚集索引。
>
> 聚集索引 / 非聚集索引不是一种索引类型，而是一种存储数据的方式。在 InnoDB 中它们还有一个非常重要的区别：聚集索引的叶子节点的的 data 域包含了完整的数据记录，而非聚集索引的叶子节点的 data 域记录着主键的值，因此在使用非聚集索引进行查找时，需要先查找到主键值，然后再到聚集索引中进行查找，这称之为**回表查询**。

**（3）什么情况下索引会失效？**

- 条件中有 or；
- like 查询（以 % 开头）；
- 如果列类型是字符串，那一定要在条件中将数据使用引号引用起来，否则不使用索引；
- 对列进行函数运算（如 where md5 (password) = “xxxx”）；
- 负向查询条件会导致无法使用索引，比如 NOT IN、NOT LIKE、!=、>、< 等；
- 对于联合索引，不是使用的第一部分 (第一个)，则不会使用索引（最左匹配）；
- 如果 mysql 评估使用全表扫描要比使用索引快，则不使用索引；

**（4）一张表里面有ID自增主键，当insert了17条记录之后，删除了第15,16,17条记录，再把MySQL重启，再insert一条记录，这条记录的ID是18还是15 ？InnoDB和MylSAM结果有什么不同，为什么？**

- InnoDB，如果新增一条记录（不重启mysql的情况下），这条记录的id是18。重启MySQ，这条记录的ID是15。因为InnoDB表只把自增主键的最大ID***\*记录到内存中\****，所以重启数据库或者对表OPTIMIZE操作，都会使最大ID丢失。
- MylSAM，这条记录的ID就是18。因为MylSAM表会把自增主键最大ID***\*记录到数据文件里面\****，重启MYSQL也不会丢失。
- 如果在这17条记录里面删除的是中间的几个记录（比如删除的是10、11、12三条记录）重启MySQL后insert一条记录后，ID都是18。因为内存或者数据库文件存储都是自增主键最大ID。
- **MySQL8.0**及以上版本这条记录的ID是18，因为这个版本保存ID的值是在redo日志中的，重启之后是可以恢复的。

**（5）你平时是怎样进行索引优化的？**

- 将经常被查询的**区分度高**的列做索引，区分度控制到20%-40%左右；

- **最左原则**

- **回盘顺序**，将order by的列与where的列建立联合索引，这样在查询出where的数据之后可以直接进行order by排序，不需要再根据主键值查询order by数据；

  ![WX20210310-175558@2x](/Users/wanghanqing/Documents/WX20210310-175558@2x.png)

- **覆盖索引**

  >​	简单来说，就是where的列和select的列一致，不需要回表查询，通过索引就可以查出所有的数据；	
  >
  >​	不管以任何方式查询表， 最终都会利用主键通过聚集索引来定位到数据， 聚集索引（主键）是通往真实数据所在的唯一路径。
  >
  >​	然而，有一种例外可以不使用聚集索引就能查询出所需要的数据，这种非主流的方法 称之为「**覆盖索引**」查询， **也就是平时所说的复合索引或者多字段索引查询**。文章上面的内容已经指出， 当为字段建立索引以后， 字段中的内容会被同步到索引之中， 如果为一个索引指定两个字段， 那么这个两个字段的内容都会被同步至索引之中。
  >
  >``` sql
  >//建立索引
  >create index index_birthday on user_info(birthday);
  >//查询生日在1991年11月1日出生用户的用户名
  >select user_name from user_info where birthday = '1991-11-1';
  >```
  >
  >这句SQL语句的执行过程如下:
  >
  >​	首先，通过非聚集索引index_birthday查找birthday等于1991-11-1的所有记录的主键ID值；
  >
  >　然后，通过得到的主键ID值执行聚集索引查找，找到主键ID值对就的真实数据（数据行）存储的位置；
  >
  >　最后， 从得到的真实数据中取得user_name字段的值返回， 也就是取得最终的结果。
  >
  >　我们把birthday字段上的索引改成双字段的覆盖索引。
  >
  >``` sql
  >create index index_birthday_and_user_name on user_info(birthday, user_name);
  >```
  >
  >这句SQL语句的执行过程就会变为:
  >
  >　　**通过非聚集索引index_birthday_and_user_name查找birthday等于1991-11-1的叶节点的内容，然而， 叶节点中除了有user_name表主键ID的值以外， user_name字段的值也在里面， 因此不需要通过主键ID值的查找数据行的真实所在， 直接取得叶节点中user_name的值返回即可**。 通过这种覆盖索引直接查找的方式， 可以省略不使用覆盖索引查找的后面两个步骤， 大大的提高了查询性能。

- **小表驱动大表**

### 索引调优 explain

**（1）索引调优有几种类型？分别什么意思？**

- system：仅一行
- const: 主键or唯一键的常量等值查询
- eq_ref：主键or唯一键的扫描或关联查询
- ref：非唯一索引的常量等值查询
- range：索引的范围查询
- index：索引全查询
- all：遍历表查询

优化至少要到range范围

**（2）explain结果值的意思是什么？**

- **id**：SQL执行的顺序的标识，SQL从大到小的执行；

- **select_type**：查询中每个select子句的类型；

  > **SIMPLE**(简单SELECT,不使用UNION或子查询等)
  >
  > **PRIMARY**(查询中若包含任何复杂的子部分,最外层的select被标记为PRIMARY)
  >
  > **UNION**(UNION中的第二个或后面的SELECT语句)
  >
  > **DEPENDENT** **UNION**(UNION中的第二个或后面的SELECT语句，取决于外面的查询)
  >
  > **UNION** **RESULT**(UNION的结果)
  >
  > **SUBQUERY**(子查询中的第一个SELECT)
  >
  > **DEPENDENT** SUBQUERY(子查询中的第一个SELECT，取决于外面的查询)
  >
  > **DERIVED**(派生表的SELECT, FROM子句的子查询)
  >
  > **UNCACHEABLE** **SUBQUERY**(一个子查询的结果不能被缓存，必须重新评估外链接的第一行)
  >
  
- **table**：数据属于的表，有事不是表名，看到的是derivedx(x是个数字，是第几步执行的结果)；

- **type**：MySQL在表中找到所需行的方式，又称访问类型；

  > 常用类型： **ALL/index/range/ref/eq_ref/const/system/NULL（从左到右，性能从差到好）**

- **possible_keys**：**指出MySQL能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用**

  该列完全独立于EXPLAIN输出所示的表的次序。这意味着在possible_keys中的某些键实际上不能按生成的表次序使用。
如果该列是NULL，则没有相关的索引。可以通过检查WHERE子句看是否它引用某些列或适合索引的列来提高你的查询性能。如果是这样，创造一个适当的索引并且再次用EXPLAIN检查查询。
  
- **key**：**key列显示MySQL实际决定使用的键（索引）**

  > 如果没有选择索引，键是NULL。要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。

- **key_len**：**表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度（key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的）**

  > 不损失精确性的情况下，长度越短越好 

- **ref**：**表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值**

- **rows**： **表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数**

- **Extra**：**该列包含MySQL解决查询的详细信息,有以下几种情况：**

> **Using where：**列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候，表示mysql服务器将在存储引擎检索行后再进行过滤。
>
> **Using temporary：**表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询。
>
> **Using filesort：**MySQL中无法利用索引完成的排序操作称为“文件排序”。
>
> **Using join buffer：**改值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进性能。
>
> **Impossible where：**这个值强调了where语句会导致没有符合条件的行。
>
> **Select tables optimized away：**这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行。

## 3.事务

**（1）描述一下事务的几种隔离机制？**

- **Read Uncommitted（读未提交）**：事务可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少。而且可能出现**脏读（Dirty Read）**，即一个事务读取到另一个事务还有没提交的记录。
-  **Read Committed（读提交）** ：事务只能看见已经提交的事务所做的改变。这种隔离级别可能导致**不可重复读**（Nonrepeatable Read），即同一事务的其他实例在该实例处理其间可能会有新的 commit，所以同一个查询操作执行两次或多次的结果不一致。
- **Repeatable Read（可重复读）** ， 事务的多个实例在并发读取数据时读到同样的数据行。不过理论上，这会导致另一个棘手的问题：**幻读（PhantomRead）**，它是指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的 “幻影” 数据。
- **Serializable（串行化）** ：通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。

**（2）如何实现一个分布式锁？**

- **基于数据库实现。** 利用主键唯一的特性，如果有多个请求同时提交到数据库的话，数据库会保证只有一个操作可以成功，那么我们就可以认为操作成功的那个线程获得了该方法的锁，当方法执行完毕之后，想要释放锁的话，删除这条数据库记录即可。易于理解，但是可能出现单点故障，性能也可能成为瓶颈。
- **基于 redis 实现。** 主要利用 redis 的 setnx () 和 expire () 方法。

> setnx (lockkey, 1) 如果返回 0，则说明占位失败；如果返回 1，则说明占位成功;
>
> expire () 命令对 lockkey 设置超时时间，为的是避免死锁问题；
>
> 执行完业务代码后，可以通过 delete 命令删除 key。

- **基于 Zookeeper 实现分布式锁。** 利用临时节点与 watch 机制。每个锁占用一个普通节点 /lock，当需要获取锁时在 /lock 目录下创建一个临时节点，创建成功则表示获取锁成功，失败则 watch/lock 节点，有删除操作后再去争锁。临时节点好处在于当进程挂掉后能自动上锁的节点自动删除即取消锁。

**（3）两个事务做如下操作，id 是表 t 的主键，num初始值为3，那么事务 2 的 update 后再 select 的值是多少呢？**

![图片描述](https://img1.sycdn.imooc.com/5e82a6360001d1ca15560716.png)

- 值为6。若引擎为InnoDB，琐是默认加在索引上的，如果id存在索引（主键有唯一索引，否则锁表），则事务1的两个update操作会将对应的数据锁住，其他事务无法修改（但是可以查询，因为读的是快照），事务2执行update会阻塞并等待事务1释放对应的行锁，等事务1commit之后释放锁，事务2再继续执行，所以最终结果为6。

  ```sql
  /* 事务1：*/ update t set num = num + 1 where id = 1;/* 添加行锁 此时事务中 num = 4,表中依旧是3 */
/* 事务2：*/ update t set num = num + 2 where id = 1;/* 无法执行，一直等待，因为数据被事务1锁住，超时后抛出 */
  /* 事务1：*/ commit;/* 释放锁 此时num = 4被更新到表中，num = 4 */
/* 事务2：*/ commit;/* 获取到锁，执行语句后返回，此时执行num = num + 2，即4 + 2 = 6，并更新到表中 */
  ```
  

**（4）什么是快照读？什么是当前读？**

- MySQL的事务是基于MVCC（多版本并发控制）实现的，每次查询语句时会先标记当前更新数据的版本，作为快照，然后返回数据，如果不用update、delete、for update之类的语句，例如select，在更新操作事务提交之前，读的都是老版本的快照；
- 对于update、delete、for update之类的写操作，语句会被阻塞，直到操作事务提交。

**（5）什么是行锁？表锁？间隙锁（区间锁）？**

- 行锁是粒度最细的锁，是以索引为基础进行加锁，如果没有索引，则会变成表锁，导致锁定整张表；

- 间隙锁一般发生在一般索引上，例如一般索引列A中有数据5和10，在进行当前读的时候，where条件为A=6，因为找不到为6的数据，所以锁定(5,10]，这个左开右闭的区间数据，当再insert之间的数据的时候就会被阻塞，但是对于其他没有加锁的区间没有影响；

  >| 步骤 | 事务A                                           | 事务B                                 |
  >| ---- | ----------------------------------------------- | ------------------------------------- |
  >| 1    | begin; select * from t where id = 6 for update; | -                                     |
  >| 2    | -                                               | insert into user value(12,12,12);阻塞 |
  >| 3    | commit;                                         | -                                     |
  >
  >当有如下事务A和事务B时，事务A会对数据库表增加(5,10]这个区间锁，这时对应区间数据的时候就会因为区间锁(5,10]而被锁住无法执行。

- 对于普通索引的当前读操作，不仅会对等值的行加上行锁，还会在上下区间加上区间锁，例如列A有数据5、10、15，当对10进行当前读，则会锁定(5,15)之间所有的数据，主要是为了防止幻读的产生，导致查询数量的变化；

### 分布式事务

**（1）什么是分布式事务，相关协议及实现方式是什么？**

- 分布式事务用于在分布式系统中保证不同节点之间的数据一致性；

- 要权衡CAP，即一致性、可用性、分区容错性，围绕一致性和可用性；

- 最具有代表性的是由**Oracle Tuxedo**系统提出的**XA**分布式事务协议；

  >XA协议包含**二阶段提交（2PC）**和**三阶段提交（3PC）**两种实现。
  
- 二阶段提交、TCC、异步确保型、事务型消息

- raft同步可以保证节点集群的一致性和性能平衡，Zookeeper为典型例子；

**（2）二阶段提交的过程是怎样的？三阶段提交呢？**

- 二阶段提交包括**prepare和commit**两个阶段；
- 二阶段提交牺牲了可用性，保持了强一致性，非常影响性能；
- 首先由事务管理器向分布式服务发送prepare消息，待所有服务返回给事务管理器确认准备完成后，进行第二步，由事务管理器发送commit消息，待所有服务器返回执行完成消息，完成二阶段提交；
- 如果其中任何一个commit操作失败了，则其他所有成功commit操作都要回滚，此时牺牲了可用性，请求失败的服务器会被阻塞；
- 三阶段提交是在二阶段提交的基础上增加了超时机制，如果第二阶段返回ack超时，则会发送回滚commit；

**（3）TCC协议的过程是怎样的？**

- 无法保证强一致性，通过**状态**去控制流程；
- TCC包括**try、confirm、cancel**三部分，每个模块顺序执行完try然后执行confirm，如果出现回滚，则顺序执行cancel；
- 事务独立，回滚时顺序回滚；
- 业务之间独立性较差，问题不好排查；

**（4）什么是raft同步，怎么实现的？**

- 用以平衡集群中数据的一致性和性能；
- 主要配合超过一半以上的数字去做取舍，只要n/2+1节点成功返回了ack，则对外表现为同步完成；
- 主要是为了主节点挂掉的时候，其他从节点进行选主，可以选出最新的已经同步过的从节点；

**（5）什么是异步确保型？**

- 牺牲强一直，确保最终一致性；
- 采用异步消息的方式确保事务最终一致；
- 保持业务独立性，主业务无感知，可通过消息日志查看问题，且中间件自带重试等机制；

**（6）什么是事务型消息？**

- 首先向消息中间件中添加prepare消息，中间件会持久化到磁盘，但是不会去推送这个消息；
- 等本地事务commit之后，向中间件发送一条commit消息，带上之前的prepare消息ID，中间件才会发送对应的消息；
- 中间件会有回查机制，如果中间件一直没有收到commit消息，则中间件会会查消息发送方，根据发送方的状态去处理这条prepare消息；

# 三、缓存

## 1.**Memcached**

**（1）Memcached的一致性哈希算法是什么？**

- 求出 memcached 服务器（节点）的哈希值，并将其配置到 0～2^32-1 的圆上。
- 采用同样的方法求出存储数据的键的哈希值，并映射到相同的圆上。
- 数据映射到的位置开始顺时针查找，将数据保存到找到的第一个服务器上。如果超过 2^32-1 仍然找不到服务器，就会保存到第一台 memcached 服务器上。

**（2）一致性哈希算法要考虑会出现什么问题，如何解决？**

- 哈希算法要考虑到**平衡性 (Balance)**（哈希的结果能够尽可能分布到所有的缓存中去，尽可能地利用缓存机器）、**单调性 (Monotonicity)**（哈希的结果尽可能地保证原有已分配的内容可以被映射到原有缓存中去，避免在节点增减过程中导致不能命中）、**分散性 (Spread)**：是指同一个哈希结果，应尽量存储在同一个缓存机器，以提升系统存储的效率。
- 可以采用服务器初始化、存储数据路由、单调性分析、虚拟化等方式解决。

## 2.Redis

> Redis是现在最受欢迎的NoSQL数据库之一，其有以下优点：
> 速度快，数据存在内存中，类似于 HashMap，HashMap 的优势就是查找和操作的时间复杂度都是O(1)；
> 支持丰富数据类型，支持 string，list，set，Zset，hash 等；
> 支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行；
> 应用场景多，可用于缓存，消息，按 key 设置过期时间，过期后将会自动删除。
>
> Redis是单线程的，能够保证setnx（set not exist）的原子性；

**（1）Redis支持的存储类型及应用场景有哪些？**
- **String：**Redis 中的 String 类型和其他很多编程语言中的语义类似，value 可以是 String 也可以是数字。 一般做一些复杂的计数功能的缓存，比如说微博数，粉丝数等；
- **Hash：**也可以称为 hashes， 是一个 string 类型的 key 和 value 的映射表，特别适合用于存储对象。比如存储用户信息，商品信息等等。内部可以用 hashtable 和 ziplist 两种承载方式来实现；
-  **List：**List 是 redis 最重要的数据结构之一，可以做简单的消息队列的功能，比如论坛点赞人列表、微博粉丝列表等；另外还可以利用 lrange 命令，可以从某个元素开始读取多少个元素，实现简单的高性能分页，类似微博那种下拉不断分页的东西，性能极佳，用户体验好；
- **Set：**set 类似 List，但是它是一个无序集合，且其中的元素不重复。可以做全局去重的功能，比如说是否给帖子点赞数；也可以判断某个元素是否在 set，比如说判断是否给某个回复点赞。另外还可以利用交集、并集、差集等操作来支撑更多的业务场景，比如说找出两个微博 ID 的共同好友等；
- **Sorted Set（ZSet）：**sorted set 相比 set 增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列，比如说可以用于取排行榜 top N 的用户。

**（2）Redis过期策略以及内存淘汰机制是怎样的？**

- **定期删除策略：**用一个定时器来负责检查 key，过期则删除 key，注意这里并不是检查所有的 key 而是随机抽取进行检查。定期策略虽然让内存及时释放，但也会额外消耗 CPU 资源，通常 CPU 应该将时间尽量用于处理业务请求，而不是删除 key。

  **惰性删除策略：**在你获取某个 key 的时候，redis 会检查一下，这个 key 如果设置了过期时间那么是否过期了，如果过期则删除该 key。

- **内存淘汰机制**

  如果定期删除没删除 key，然后也没及时去请求 key，即惰性删除也没生效，持续下去 redis 的内存会越来越高，当超过 redis 设置的内存最大使用量时，就会进行内存数据淘汰。redis 有 6 种淘汰策略：
  
  | 策略            | 描述                                                     |
  | :-------------- | :------------------------------------------------------- |
  | volatile-lru    | 从已设置过期时间的数据集中挑选最近最少使用的数据淘汰     |
  | volatile-ttl    | 从已设置过期时间的数据集中挑选将要过期的数据淘汰         |
  | volatile-random | 从已设置过期时间的数据集中任意选择数据淘汰               |
  | allkeys-lru     | 从所有数据集中挑选最近最少使用的数据淘汰                 |
  | allkeys-random  | 从所有数据集中任意选择数据进行淘汰                       |
  | noeviction      | 当内存不足以容纳新写入数据时，新写入操作会报错。很少使用 |

**（3）Redis的持久化方式有哪些，区别和优缺点是怎样的？**

- **RDB（Redis DataBase）**，用数据集快照的方式，定时将Redis存储的数据生成快照并存储到磁盘等介质上；

  >**优点：**
  >
  >特别适合备份；
  >
  >性能最大化，fork子进程来完成写操作，让主进程继续处理命令且不会进行任何IO操作，确保Redis性能；
  >
  >相对于数据集大时，比AOF的启动效率更高。
  >
  >**缺点：**
  >数据安全性低，RDB 是间隔一段时间进行持久化，如果持久化之间发生故障，会发生数据丢失，所以这种方式更适合数据要求不严谨的时候；

- **AOF（Append -only file）**，命令行记录以Redis命令请求协议的格式完全持久化存储，保存为.aof 文件。

  > **优点：**
  >
  > 数据安全，可以配置append fsync属性，比如无fsync，每秒钟一次 fsync，或者每次执行写入命令时 fsync，一般只会丢失一秒钟的数据，或者最后一次执行的数据，对缓存来说，这已经足够。
  >
  > 某些场景下还可以恢复数据。比如说某同学在操作Redis时，不小心执行了FLUSHALL，导致Redis内存中的数据全部被清空了。如果 AOF 文件还没有被重写（rewrite），可以暂停Redis并编辑AOF文件，将最后一行的FLUSHALL命令删除，然后重启Redis，就可以恢复所有数据到FLUSHALL之前的状态。
  >
  > **缺点：**
  >
  > AOF 文件比 RDB 文件大，且根据不同的fsync策略，其恢复速度可能较慢；
  > 数据集大的时候，比RDB启动效率低。
  >
  > **AOF 文件太大会怎么样？**
  >
  > 后台会自动地对AOF进行重写（rewrite），重写时会压缩 AOF 文件内容，只保留可以恢复数据的最小指令集，比如调用了100次INCR指令，完全可以合并成一条 SET 指令。
  >
  > 进行AOF重写时，仍然是采用先写临时文件，全部完成后再替换的流程，所以断电、磁盘满等问题都不会影响AOF文件的可用性。

**RDB 和 AOF 对比**

| 命令       | RDB    | AOF          |
| :--------- | :----- | :----------- |
| 启动优先级 | 低     | 高           |
| 体积       | 小     | 大           |
| 恢复速度   | 快     | 慢           |
| 数据安全性 | 丢数据 | 根据策略决定 |
| 轻重       | 重     | 轻           |

**（4）Redis 哈希槽和节点的关系是怎样的？**

- Redis引入了哈希槽的概念，Redis集群有16384（2^14）个哈希槽，每个key通过CRC16校验后对16384取模来决定放置哪个槽，集群的每个节点负责一部分hash槽，具体分多少可以指定。
- Redis 集群最大节点数也是16384，至少保证一个节点有一个哈希槽。

**（5）假如Redis里有1亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如何将它们全部找出来？**

- 使用 keys 指令可以扫出指定模式的 key 列表但是Redis是单线程的，keys指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用scan指令，scan可以无阻塞的提取出指定模式的 key 列表，但是会有一定的重复概率，需要在客户端做一次去重，但是整体所花费的时间会比直接用keys指令长。

![图片描述](https://img1.sycdn.imooc.com/5e9dac390001248c06660559.png)

**（6）ZSet数据结构是怎样实现的？**

- 基于红黑树，采用跳表+压缩表的方式实现；
- 一般情况下，ZSet将每个节点以双向链表的方式连接，但是当长度超过了128个节点或key特别大，则会跳跃连接，例如开始时连接方式为1-2-3-4-5，跳表就会连接1-3-5。

## 3.缓存的一致性

**（1）什么是缓存的一致性？**

- **缓存的一致性是指缓存与数据库数据的一致性**

**（2）如何保证缓存的一致性？**

- **更新缓存/缓存失效**

> 如果获取数据的带价很小，更新缓存代价也很小，那么可以先更新缓存；
>
> 如果是更新缓存的代价很大，例如需要多个接口调用和数据查询才能获得最新的结果，那么可以先淘汰缓存，后续请求通过数据库中检索；

- **先更新数据库再让缓存失效/先让缓存失效再更新数据库**

  > 更新数据库和更新缓存无法保证原子性的，需要根据业务场景选择。

- **通过MQ解决缓存一致性问题**

  >第一步更新数据库，第二步尝试让缓存失效，如果第二步对个某个key的缓存失效，可以发送失败消息到MQ中，然后自己消费MQ的消息不断的重试让缓存失效。
  >
  >这种方式的思路是实现更新数据库与缓存的异步，更新数据库之后，通过消息使缓存更新，保持一致性。

**（3）解释一下缓存穿透、缓存击穿、缓存雪崩，有什么解决方式？**

- **缓存穿透**

  缓存穿透是指**缓存和数据库中都没有的数据**，而用户不断发起请求，如发起为id为“-1”的数据或id为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。

  > 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
  >
  > 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止用户反复用同一个id暴力攻击。

- **缓存击穿**

   缓存击穿是指**缓存中没有但数据库中有的数据**（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。
> 设置热点数据永远不过期；
>
> 对访问数据加互斥锁，并发数据会等待数据查询完并放到缓存中，使用syncronized等；
>
> 使用消息组件将访问排队，由最早的访问操作Redis；

- **缓存雪崩**

   缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

  > 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
  >
  > 如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中。
  >
  > 设置热点数据永远不过期。

## 4.缓存对比

**（1）Memcached与Redis有什么区别？**

- **支持的存储类型不同**

  > Redis和Memcached都是内存型数据库；
  >
  > Memcached存储String、图片、文件、视频等格式的文件；
  >
  > Redis存储String、Hash、List、Set及ZSet(sorted set：有序集合)，适合分布式缓存的实现。

- **数据持久化及数据恢复**

  >Memcached数据不可恢复;
  >
  >Redis支持在配置里打开数据落盘（RDB）及操作持久化（AOF）来找回数据。

- **内存管理**

  >Memcached可以修改最大内存，使用LRU（Least recently used,最近最少使用）算法；
  >
  >Redis使用自身VM，突破物理内存限制；
  >
  >Value值Redis最大可以达到1GB，Memcache只有1MB。
  
- **分布式支持**

  >Memcached一般是在客户端通过一致性哈希等算法来实现分布式存储；
  >
  >Redis更偏向于在服务器端构建分布式存储，支持主从复制等集群模式；

- **内存管理**

  >Memcached使用Slab Allocation（将分配的内存分割成各种尺寸的块(Chunk)，并把尺寸相同的块分成组（Chunk的集合），每个Chunk集合被称为Slab）管理内存，其主要思想是按照预先规定的大小呈阶梯状分配好，会根据接收到数据的大小选择一个最合适的Slab Class进行存储，该方法可以避免内存碎片问题，可也不可避免地出现一定的空间浪费；
  >
  >Redis 则采用包装过的mallc/free来分配内存，什么时候需要什么时候分配，更简单一些。

- **使用场景**

  >Memcached是一个分布式内存对象缓存系统，旨在通过减轻数据库负载来加速动态 Web 应用程序；
  >
  >Redis 是内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。

|          |    Memcached    |      Redis      |
| :------: | :-------------: | :-------------: |
|   类型   | Key-Value数据库 | Key-Value数据库 |
| 过期策略 |      支持       |      支持       |
| 数据类型 |  单一数据类型   |  五大数据类型   |
|  持久化  |     不支持      |      支持       |
| 主从复制 |     不支持      |      支持       |
| 虚拟内存 |     不支持      |      支持       |

# 四、Linux

## **1.常用命令**

**（1）假定访问日志 access.log 格式如下（其中最前面的 67.21.221.37 是访问 IP）：67.21.221.37 - 10.5.9.2:28086 - - [05/Apr/2020:21:58:04 +0800] "POST http://www.xxxx.com" 200 1187 ，请给出统计访问IP top10的命令。**

- awk ‘{print $1}’ access.log |sort | uniq -c | sort -nr | head -10

>awk '{print $1}' access.log：将access.log中每一行以空格作为分隔符分成数组，取数组中第一个元素。
>
>sort | uniq -c | sort -nr：将截取的元素首先进行排序（sort），然后去重统计行数（uniq -c），再按照数字（-n）进行倒序（-r）。值得注意的是， uniq之前需要先用 sort 进行排序，因为uniq只会判断相邻的行是否重复，不能全文本进行搜索是否重复。
>
>head -10表示取前 10 条数据，即访问 IP top10。

# 五、Spring

| Spring AOP                                       | AspectJ                                                      |
| ------------------------------------------------ | ------------------------------------------------------------ |
| 在纯 Java 中实现                                 | 使用 Java 编程语言的扩展实现                                 |
| 不需要单独的编译过程                             | 除非设置 LTW，否则需要 AspectJ 编译器 (ajc)                  |
| 只能使用运行时织入                               | 运行时织入不可用。支持编译时、编译后和加载时织入             |
| 功能不强-仅支持方法级编织                        | 更强大 - 可以编织字段、方法、构造函数、静态初始值设定项、最终类/方法等…。 |
| 只能在由 Spring 容器管理的 bean 上实现           | 可以在所有域对象上实现                                       |
| 仅支持方法执行切入点                             | 支持所有切入点                                               |
| 代理是由目标对象创建的, 并且切面应用在这些代理上 | 在执行应用程序之前 (在运行时) 前, 各方面直接在代码中进行织入 |
| 比 AspectJ 慢多了                                | 更好的性能                                                   |
| 易于学习和应用                                   | 相对于 Spring AOP 来说更复杂                                 |

Spring AOP使用了AspectJ的Annotation，使用了Aspect来定义切面，使用Pointcut来定义切入点，使用Advice来定义增强处理。虽然使用了Aspect的Annotation，但是并没有使用它的编译器和织入器。其实现原理是JDK 动态代理，在运行时生成代理类。

## 1.AOP

**（1）你们项目中主要用AOP做什么，具体怎么做的？**

- 结合自定义注解，使用AOP做日志记录或者缓存

- 第一步：创建自定义注解

  > ```java
  > @Retention(RetentionPolicy.RUNTIME)
  > @Target(ElementType.METHOD)
  > public @interface PrintLog {}
  > ```

- 第二步：创建切面类

  > ```java
  > @Aspect
  > @Component
  > public class RequestLogAspect {
  >     @Pointcut("@annotation(注解类路径)")
  >     //@Pointcut("execution(public * com.example.controller..*.*(..))")
  >     public void printLog() {}
  >   
  >     @Before("printLog()")
  >     public void doBefore(JoinPoint joinPoint) throws Throwable {
  >         // TODO
  >     }
  >   
  >     @AfterReturning(returning = "ret", pointcut = "printLog()")
  >     public void doAfterReturning(Object ret) throws Throwable {
  >         // TODO
  >     }
  > }
  > ```

- 第三部：在需要的地方添加注解

**（2）什么要引入AOP的编程范式？ AOP的好处及适用场景是什么？AOP的两大核心要点是什么**

- 解决非功能性代码重复问题，如日志打印和事务控制等，实现关注点分离；
- AOP能保持编程的内聚性，使代码高可用，减少代码的耦合性，AOP是低侵入易分离的，代码的可读性较好
- 适用于独立于业务功能的服务开发；
- AOP的两大核心一是方面，指需要定义什么方面和业务（Aspect）；二是切入点（PointCut）。

**（3）Advice注解有哪几种，分别表示什么意思？**

- @Before，前置通知
- @After（finally），后置通知，方法执行完之后
- @AfterReturning，返回通知，成功执行之后
- @AfterThrowing，异常通知，抛出异常之后
- @Around，环绕通知

**（4）说说 Spring 的 AOP 的原理是什么？实现 AOP 有哪些方式？**

Spring AOP 的底层使用的是动态代理，有两种实现方式：

- **JDK 动态代理**：利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用 InvokeHandler 来处理。

- **CGlib 动态代理**：以 CGLIB（Code Generation Library）的方式进行代理，它采用底层字节码技术，将代理对象类的 class 文件加载进来，通过修改其字节码生成子类来处理。

- **区别**：JDK 代理只能对实现接口的类生成代理；CGLIB 是针对类实现代理，继承指定类并生成一个子类，因此不能代理 final 修饰的类。

> 那么在 Spring 中，优先使用哪种 AOP 呢？
>
> - 如果目标对象实现了接口，默认会采用 JDK 的动态代理，但也可以强制使用 CGLIB；
>- 如果目标对象没有实现了接口，则必须采用 CGLIB 库。
> - Spring 会自动在 JDK 动态代理和 CGLIB 之间转换。

**（5）为什么AOP动态代理需要实现接口？**

- 因为生成代理类的时候已经继承了Proxy类，只能通过实现的方式来建立与我们需要代理的类的联系。

## 2.IoC

**（1）什么是Spring IoC容器，其初始化过程是怎样的？**

- **IoC容器是指的Spring Bean工厂里存储了Bean实例的Map存储结构** 

![img](https://img-blog.csdnimg.cn/20200719195142791.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZveF9iZXJ0,size_16,color_FFFFFF,t_70)

**（2）BeanFactory、ApplicationContext和FactoryBean的区别是什么？**

- Spring的两种IoC：BeanFactory和ApplicationContext，都是Java Interface。

  > ApplicationContext 继承于 BeanFactory，BeanFactory 和 ApplicationContext 都提供了一种方式，使用getBean("bean name")获取bean。

- BeanFactory是IoC容器的顶级接口，是IoC容器的最基础实现，也是访问Spring容器的根接口，负责对bean的创建，访问等工作。实现类功能单一，特点是在启动的时候不会去实例化Bean，中有从容器中拿Bean的时候才会去实例化。

  >延迟实例化的优点：应用启动的时候占用资源很少；对资源要求较高的应用，比较有优势。

- ApplicationContext拥有BeanFactory的全部功能，并且扩展了很多高级特性，每次容器启动时就会创建所有的对象。

  > 不延迟实例化的优点： 所有的Bean在启动的时候都加载，系统运行的速度快；在启动的时候所有的Bean都加载了，我们就能在系统启动的时候，尽早的发现系统中的配置问题；建议web应用，在启动的时候就把所有的Bean都加载。

- FactoryBean也属于Spring Bean，常规的Bean都是使用Class的反射获取具体实例，如果Bean的获取过程比较复杂，那么常规的xml配置需要配置大量属性值，这个可以使用FactoryBean，实现这个接口，在其getObject()方法中初始化这个bean。

## 3.Spring Bean

（1）Bean的生命周期是怎样的？

- 大体可以分为四部分：实例化、属性注入、初始化、销毁

  ![img](https://img-blog.csdnimg.cn/img_convert/328d97e59f325652170e081b2de6091c.png)

> - Spring启动，查找并加载需要被Spring管理的bean，进行Bean的实例化
> - Bean实例化后对将Bean的引入和值注入到Bean的属性中
> - 如果Bean实现了BeanNameAware接口，Spring将Bean的Id传递给setBeanName()方法
> - 如果Bean实现了BeanFactoryAware接口，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
> - 如果Bean实现了ApplicationContextAware接口，Spring将调用Bean的setApplicationContext()方法，将应用上下文的引用传入到bean中
> - 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法
> - 如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
> - 如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法
> - 此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
> - 如果Bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用

## 4.SpringBoot

**（1）SpringBoot的启动过程是怎样的？**

**第一阶段：创建SpringApplication**

- **根据classpath判断WebApplicationType**

  > SpringMVC本质上是依赖于SpringBoot的starter-web启动的，starter-web会下载Spring MVC启动所需要的所有相关包，就会包括DispatcherServlet，于是判断WebApplicationType为SERVLET；

- **初始化初始化器ApplicationContextInitializer。**以后会向容器中注册属性

- **初始化监听器ApplicationListener。**用以之后监听启动后的事件广播，以便做其他操作

> 从spring.factories中加载所有Spring容器需要初始化的启动器和监听器；

- **获取MainApplication的类**

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
   this.resourceLoader = resourceLoader;// 获取启动类
   Assert.notNull(primarySources, "PrimarySources must not be null");// 判断加载类是否为NULL
   this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
   this.webApplicationType = deduceWebApplicationType();// 判断WebApplicationType
   setInitializers((Collection) getSpringFactoriesInstances(
         ApplicationContextInitializer.class));// 加载启动器
   setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));//加载监听器
   this.mainApplicationClass = deduceMainApplicationClass();// 获取main函数调用的类
}
```

**第二阶段：执行run()方法**

- **创建StopWatch**。用以做监听记录器，记录任务执行时间

- **获取SpringApplicatioinRunListeners并开启启动监听**

  > 通过spring.factories获取，主要是EventPublishingRunListener，用以做事件的发布推送

- **准备环境变量，配置信息，系统属性等**

- **根据WebApplicationType创建ApplicationContext容器上下文**

- **prepareContext。**执行之前的Initializer初始化，传入参数，加载主Bean

- **refreshContext。**启动Web Server

- **afterRefresh。**执行初始化完成后要做的操作

- **发布启动完成的Event事件**

- **调用Listener的方法，启动所有的监听**

```java
public ConfigurableApplicationContext run(String... args) {
   StopWatch stopWatch = new StopWatch();// 监听记录器
   stopWatch.start();// 启动监听记录器
   ConfigurableApplicationContext context = null;
   Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
   configureHeadlessProperty();// 配置setProperty能力
   SpringApplicationRunListeners listeners = getRunListeners(args);// 获取所有的监听器
   listeners.starting();// 开启启动监听
   try {
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
      ConfigurableEnvironment environment = prepareEnvironment(listeners,
            applicationArguments);// 设置环境变量
      configureIgnoreBeanInfo(environment);// 设置忽略的Bean信息
      Banner printedBanner = printBanner(environment);// 打印启动Banner
      context = createApplicationContext();// 创建ApplicationContext
      exceptionReporters = getSpringFactoriesInstances(
            SpringBootExceptionReporter.class,
            new Class[] { ConfigurableApplicationContext.class }, context);// 排查异常
      prepareContext(context, environment, listeners, applicationArguments,
            printedBanner);// 准备ApplicationContext
      refreshContext(context);// 刷新ApplicationContext
      afterRefresh(context, applicationArguments);// 做最后的收尾工作
      stopWatch.stop();// 停止监听记录
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass)
               .logStarted(getApplicationLog(), stopWatch);
      }
      listeners.started(context);// 发布启动完成的事件
      callRunners(context, applicationArguments);// 指示应该启动的所有的相关Bean
   }
   catch (Throwable ex) {
      handleRunFailure(context, ex, exceptionReporters, listeners);
      throw new IllegalStateException(ex);
   }

   try {
      listeners.running(context);// 调用监听者方法，启动所有的监听
   }
   catch (Throwable ex) {
      handleRunFailure(context, ex, exceptionReporters, null);
      throw new IllegalStateException(ex);
   }
   return context;
}
```

# 六、Dubbo

### Dubbo的线程工作模型


- Boss线程：用来接收处理客户端的链接请求

- Worker线程：处理IO读写事件

- IO线程池：Boss线程池Worker线程池统称，旨client建立连接到发送request过程
	> all：一律进工作线程池
	> direct：一律进IO线程池
	> message（推荐使用）：除了请求和响应走工作线程池，其他都是IO线程池
	> execution：除了请求，其他都走IO线程池
	>  connection：连接和断连进队列，其他都走工作线程池

- 业务工作线程池

  > fixed：固定大小线程池
  >
  > cached：缓存线程池，一分钟后释放
  >
  > limited：可伸缩线程池，只会增长不会收缩
  >
  > eager：min到max，超过后进队列，队列满后拒绝


### Dubbo的序列化方式有哪些？

- Hessian （默认）
- fastjson
- jdk
- protobuf
- protostuf（推荐）

### SpringCloud与Dubbo的区别

| 核心要素       | Dubbo            | SpringCloud                                                  |
| -------------- | ---------------- | ------------------------------------------------------------ |
| 服务注册中心   | Zookeeper、Redis | Spring Cloud Netflix Eureka                                  |
| 服务调用方式   | RPC              | HTTP REST API                                                |
| 服务网关       | 无               | Spring Cloud Netflix Eureka                                  |
| 断路器         | 不完善           | Spring Cloud Netflix Hystrix                                 |
| 分布式配置     | 无               | Spring Cloud Config                                          |
| 分布式追踪系统 | 无               | Spring Cloud Sleuth                                          |
| 消息总线       | 无               | Spring Cloud Bus                                             |
| 数据流         | 无               | Spring Cloud Stream 基于Redis，Rabbit，Kafka实现的消息微服务 |
| 批量任务       | 无               | Spring Cloud Task                                            |

### Dubbo的服务暴露，发现以及调用过程

- 服务暴露

  > 通过@Service注解，扫描实现类并封装成ServiceBean；
  >
  > 经过一系列的配置检查校验后，加载注册中心，结合配置封装Exporter，缓存并生成注册地址URL；
  >
  > 生成ProviderModel，并初始化，实现服务暴露；
  >
  > 本质上就是将自己本地的ServiceBean及配置组成一个对应的URL地址注册到注册中心上；

- 服务发现及调用

  > 通过@Reference注解，扫描并生成ReferenceBean；
  >
  > 调用ZookeeperRegistry.doRegister注册服务，完成消费者服务注册；
  >
  > 通过注册目录(RegistryDirectory)订阅subscribeUrl通知，通过读取缓存中Exporter的Key，完成服务发现；
  >
  > 调用时通过代理的方式生成ReferenceBean进行服务调用；

### RPC和HTTP的区别

- RPC要求服务提供方和服务调用方都需要使用相同的技术，要么都hessian，要么都dubbo
- 而http无需关注语言的实现，只需要遵循rest规范
- RPC的开发要求较多，像Hessian框架还需要服务器提供完整的接口代码(包名.类名.方法名必须完全一致)，否则客户端无法运行
- Hessian只支持POST请求
-  Hessian只支持JAVA语言

# 七、消息队列



# 八、设计模式

![img](https://img1.sycdn.imooc.com/5e9da9820001cdd513290562.png)

## 1.六大原则

**（1）设计模式一般遵循哪六大原则？**

- **单一职责原则（Single Responsibility Principle）**：不要存在多于一个导致类变更的原因。通俗的说，即一个类只负责一项职责。
- **里氏代换原则（Liskov Substitution Principle）**：所有引用基类的地方必须能透明地使用其子类的对象。简单地说，一个软件如果使用的是一个父类的话,那么一定适用于其子类,而察觉不出父类。
- **依赖倒置原则（Dependence Inversion Principle）**：细节应该依赖于抽象，而抽象不应该依赖于细节。依赖倒置原则的核心思想是面向接口编程。
- **接口隔离原则（Interface Segregation Principle）**：客户端不应该依赖它不需要的接口，即一个类对另一个类的依赖应该建立在最小的接口上。
- **迪米特法则（最少知道原则）（Demeter Principle）**：一个对象应该对其他对象保持最少的了解，即尽量降低耦合。
- **合成复用原则（Composite Reuse Principle）**：尽量首先使用合成/聚合的方式，而不是使用继承。

**（2）单例模式的创建方式有哪两种，分别怎么实现？**

- **饿汉式单例**

  >```java
  >public class SingletonTest {
  >        // 1. 类对象
  >        private static SingletonTest instance = new  SingletonTest();
  >
  >       // 2. 构造函数 设置为 私有权限，禁止他人创建实例
  >        private SingletonTest() {
  >        }
  >
  >        // 3. 通过调用静态方法获得单例
  >        public static SingletonTest newInstance() {
  >            return instance;
  >       }
  > }
  > ```
  > 

- **懒汉式单例**

>```java
>public class SingletonTest {
>    // 1. 创建静态内部类
>    private static class SingletonTestHolder {
>            // 静态内部类里创建单例，JVM只会加载1遍，Java虚拟机保证了线程安全性
>            private static SingletonTest instance = new SingletonTest();
>    }
>
>    // 2. 构造函数设置为私有权限，禁止他人创建实例
>    private SingletonTest() {}
>
>    // 3. 延迟加载，按需创建
>    public static  SingletonTest getInstance() {
>      // 在调用装载被初始化时，会初始化它的静态域，从而创建单例；
>          return SingletonTestHolder.instance;
>        }
>}
>```

# 九、面试问题

## 一面

### 1.自我介绍

> 一般是平级同事，需要了解面试官和面试者未来工作的关系，以及他想要从自我介绍中了解什么？
>
> 时间：3分钟之内
>
> 经历简介：个人信息、教育背景、职业生涯、工作年限、项目经历、技术总结
>
> 凸显技术经验能力（技术栈、技术实践项目经验）、学习思考能力
>
> 例：面试官好，我叫XXX，17年毕业于烟台大学软件工程专业，大四实习到19年三月在一家互联网金融公司做P2P项目，主要负责银行存管，用户中心和标的管理相关的开发。后来入职酒仙网参与新零售项目开发，目前主要负责门店和人员管理，茅台及审计数据上报相关的开发工作。公司目前的技术架构主要包括SpringBoot、SpringCloud、MyBatis、MySQL、Redis、RabbitMQ、Zookeeper等。平时我喜欢看看书，写写个人博客，最近也在学习容器相关的知识，学习之余，我喜欢弹钢琴，画画，打羽毛球。
> 

### 2.项目的业务场景，系统架构，工作职责

>流量入口、服务治理、数据选型
>
>例1：我所做的项目是一个P2P金融理财，包含一整套的客户借款投资理财服务场景，主要模块包括借款人申请模块，风控审批模块，标的管理模块，投资人投资模块、银行存管对接模块等十余个模块，我主要负责的是标的相关、存管对接和用户相关模块的开发。
>
>![WX20210224-120108@2x](/Users/wanghanqing/Documents/WX20210224-120108@2x.png)
>
>例2：我所在的是O2O酒类新零售项目组，是基于领域模型围绕门店打造的新零售服务项目，主要模块有门店管理，雇员管理，进销存管理，用户会员管理，价签管理等几十个模块，我主要负责的是门店管理，公众号，爆品奖励，报表，成本价计算，茅台数据上报等模块的开发，快速理解需求并设计领域模型和DB结构，进而更高效进行代码编写，以业务块为单位进行单元测试后提交测试上线。
>
>![D9C192EE-D7D3-4FA8-8F2E-1EEBC7125CAA](/Users/wanghanqing/Documents/D9C192EE-D7D3-4FA8-8F2E-1EEBC7125CAA.png)
>
>使用CDN做静态资源的缓存，Nginx集群做反向代理，通过Zuul网关访问注册到Eureka上的微服务，数据存储用到的是主从分离的MySQL和SQL Server，Redis做缓存，通过MQ保持缓存数据的一致性



### 3.项目中的亮点以及如何处理解决难点问题

>常见问题or偏门问题
>
>正常问题or采坑问题
>
>例1：在做银行存管对接时，我们碰到了异步消息回传接受不成功的问题，后来发现是因为用户提交单据的时候，出现了段时间内重复生成单据的情况，导致银行那边验证单号失败，回调失败。后来我们设计了一个请求缓存将用户单据缓存起来，以单据号为key，当发生单据请求时先去缓存中查询是否有请求中的单据，如果有则不会进一步请求，等待银行成功回调后再删除缓存。
>
>例2：我们新零售线下门店一开始的时候全国只有几十家门店，后来扩展成了上千家，而且分成了多个商户，导致门店商品关系数量非常多。原来我们电子价签的推送逻辑是每天凌晨开始推送，更新总公司下所有商品以及所有门店的价签数据，但是现在因为数据量非常多，导致早上门店开业的时候数据还推送不完，价签无法及时更新。我针对此情况进行了电子价签模块的重构，首先是3项去除，去除门店没有的商品，去除没有变动的商品，去除没有变动的字段，这样可以减少70%左右的数据量。然后是2个提前，提前关店推送，提前计算价格，当门店每天关店的时候，通过消息提前触发价签变更逻辑，分担价签更新压力，因为之前推送时除了查询商品消耗时间，还有的就是计算价格，我们价格需要查询进货价、零售价、会员价、促销价、优惠等多个价格进行计算，我将价格变更后的最终展示价格缓存起来，这样也减少了价格计算的耗时。最后是1个节点，原来我们电子价签相关业务只有一个节点，后来添加节点后，极大减轻了单节点服务的压力。以上几个处理方案理论上目前可以支持十万门店价格同时变更带来的数据压力。

### 4.你是怎么排查程序异常问题的？OOM怎么排查？

- jsp：虚拟机进程状态工具

>jsp -v | grep pid

- jinfo：jvm参数信息工具

  > jinfo -flags pid

- jstat：查看虚拟机各种运算状态

> jstat -gcutil pid

- java -Xms48m -Xmx48m -XX:+HeapDumpOnOutOfMemoryError XX:HeapDumpPath=./heapdump.hprof -jar mianshi.jar

  > 使用jprofiler查看dump文件及call tree分析

### 5.你有什么问题吗？

- 了解工作内容，当前职位所在项目组主要做哪块业务，用到了哪些技术栈，部门有多少成员，以后业务的发展方向和目标，要表现出充满期待和有兴趣；

### 6.一面雷点提示

- 做好充足准备，项目结构清晰，不能有盲点；
- 不是自己负责的部分要么弄清楚，要么直接说明自己没有参与，不清楚；

## 二面

### 1.制约程序性能的参数有哪些？

- 并发：同一时间多少请求访问
- TPS：Transaction per second，每秒写操作量
- QPS：Query per second，每秒读操作量
- 耗时：端到端耗时，服务端耗时，应用程序耗时
- 95线：95%的请求落在什么地方
- 99线：99%的请求落在什么地方

### **2.你如何优化应用？**

- GC优化（参考JVM模块）

- 日志文件优化

  >同步打印：注意内容的精炼和有效；
  >
  >异步打印：采用内存管道Buffer，先将内容推入管道中，再有一个线程专门从Buffer中获取日志；
  >
  >日志归档时间：零点归档时会对归档的文件上锁打包，导致日志无法记录，当打包完成后才会新建当日日志文件；
  >
  >日志大小拆分：减少锁性能的影响；
  >
  >可以使用Elastic Search；

- 池化策略

  > 线程池、连接池；
  >
  > 关注Idle数量；一般是CPU核数*2（IO密集型）；计算密集型为核心数；

### 3.你们如何进行数据库优化？

- **查询优化**

  > 主键查询	千万数据	1-10ms
  >
  > 唯一索引	千万数据	10-100ms
  >
  > 非唯一索引	千万数据	100-1000ms
  >
  > 无索引	百万数据	1000ms+

- **批量写优化**

  > 尽量使用Execute once insert into table values (1),(2),(3),(4)...;
  >
  > SQL编译N次和1次的时间与空间复杂度
  >
  > 网络消耗的时间复杂度
  >
  > 磁盘寻址的复杂度

- **索引优化**

- **Innodb相关优化**

  > **max_connection**=1000	增大最大连接数，默认为100
  >
  > **innodb_file_per_table**=1	可以存储每个innodb表和他的索引在自己的文件中，将表之间的索引数据隔离
  >
  > **innodb_buffer_pool_size**=1G	缓存池大小，设置为当前数据库服务内存的60%-80%
  >
  > **innodb_log_file_size**=256M	一般取256M可以兼顾性能和recovery的速度，写满后只能切换日志靠buffer存储
  >
  > **innodb_log_buffer_size**=16M	该参数确保有足够大的日志缓冲区来保存脏数据在被写入到日志文件之前可以继续MySQL事务操作
  >
  > **innodb_flush_log_at_trx_commit**=2
  >
  > 1时，在每个事务提交时，日志缓冲被写到日志文件，对日志文件做到磁盘操作的刷新。Truly ACID。速度慢。
  >
  > 2时，在每个事务提交时，日志缓冲被写到系统缓冲，但不对日志文件做到磁盘操作的刷新。然后根据innodb_flush_log_at_timeout（默认为1s）时间flush disk只有操作系统崩溃或掉电才会删除最后一秒的事务，不然不会丢失事务。
  >
  > 0时，效率更高，但安全性差。每秒才write日志 任何mysqld进程的崩溃会删除崩溃前最后一秒的事务。
  >
  > **innodb_data_file_path=ibdata1:1G;ibdata2:1G;ibdata3:1G:autoextend**	指定表数据和索引存储的空间，可以是一个或者多个文件。
  >
  > > 日志缓冲：跟着mysqld进程走，进程崩溃会丢失数据；
  > >
  > > 系统缓冲：跟着操作系统走，操作系统崩溃会丢失数据；

- **读写分离**

  > 一主多从：事务提交后，master记录binlog，slave监听binlog，同步数据；
  >
  > 读库延迟问题处理：应用层让步；强行读master；
  >
  > 主从切换处理：只有当slave都同步完之后再返回数据，或者完成1/2的slave后返回；

- **分库分表**

  > 垂直拆分：根据不同的业务进行拆分的，拆分成不同的数据库，比如会员数据库、订单数据库、支付数据库、消息数据库等；
  >
  > 水平拆分：把同一张表中的数据拆分到不同的数据库中进行存储，或把一张表拆分成 n 多张小表；
  >
  > 多主多从：保证可用性

### 4.如何解决网络瓶颈？

- 公网：带宽，出口调用量；

- 内网：带宽，出口调用量；

  >扩容：扩容带宽
  >
  >分散：分散数据
  >
  >压缩：压缩数据

### 5.如何解决微服务的异常问题？

- **高效限流**

  > **接口限流：**
  >
  > > **计数器限流：**计数器算法是使用计数器在周期内累加访问次数，当达到设定的限流值时，触发限流策略。下一个周期开始时，进行清零，重新计数，使用redis的incr原子自增性和线程安全即可轻松实现。
  > >
  > > **滑动窗口限流：**滑动窗口算法是将时间周期分为N个小周期，分别记录每个小周期内访问次数，并且根据时间滑动删除过期的小周期，可以很好的解决固定窗口算法的临界问题。
  > >
  > > **漏桶限流：**漏桶算法是访问请求到达时直接放入漏桶，如当前容量已达到上限（限流值），则进行丢弃（触发限流策略）。漏桶以固定的速率进行释放访问请求（即请求通过），直到漏桶为空。
  > >
  > > **令牌桶限流：**令牌桶算法是程序以r（r=时间周期/限流值）的速度向令牌桶中增加令牌，直到令牌桶满，请求到达时向令牌桶请求令牌，如获取到令牌则通过请求，否则触发限流策略。
  >
  > **总限流**

- **灵活熔断**

  > 失败率触发
  >
  > 失败次数触发
  >
  > 全恢复
  >
  > 半转全

- **配置监控**

  > 接口维度
  >
  > DB维度
  >
  > Redis维度
  >
  > 使用大众点评CAT
  >
  > Zabbix
  >
  > 客户端监控

### 二面面试雷点

- 基础知识准备充足，忌讳知识点盲区和问1扯2
- 先说面后说点，再扩展到面总结
- 避开不擅长领域

# 待整理

分布式事务

分布式锁

分库分表

自定义注解