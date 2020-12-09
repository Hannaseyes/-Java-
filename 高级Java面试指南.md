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
> 而 StringBuffer/StringBuilder 性能更高，是因为每次都是对对象本身进行操作，而不是生成新的对象并改变对象引用。一般情况下性能从高到低：StringBuilderStringBufferString，但是某些情况下不一定。
>
> Java 5之后，单纯的字符串+操作会自动优化为StringBuilder，例如"Hello" + "World"，但是若使用非final修饰的变量，则不会进行优化。

 **（2）String str = new String("Hello World!")；创建了几个对象？**

 - 如果字符串常量区里存在"Hello World!"，则new String()创建了一个对象，否则创建了两个对象，str为引用变量。

 **（3）什么情况下使用？**

 - 如果要操作少量的数据用 String；
 - 单线程操作字符串缓冲区下操作大量数据 StringBuilder；
 - 多线程操作字符串缓冲区下操作大量数据 StringBuffer，但是一般情况下也很少用到，因为不保证逻辑正确和调用顺序正确，大多数时候，需要的不仅仅是线程安全，而是锁；

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
>Integer f1 = 100;java在编译的时候,被翻译成-> Integer f1 = Integer.valueOf(127);

### **equals、== 和 hashCode**

**（1）==比较基本数据类型和引用类型的区别？ **

- 基本数据类型比较的是他们的值，引用类型（类、接口、数组）比较的是他们在内存中的存放地址；

**（2）String中equals方法判断相等的步骤？**

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

- 解决冲突：当集添加新元素，先调用元素hashCode方法定位应防止物理位置，若位置上没有元素，则直接存储，否则调用位置上元素的equals方法与新元素进行比较，相同则不存储，不相同则存储其他位置。

> 如果两个对象equals，Java运行时环境会认为他们的hashcode一定相等；
> 如果两个对象不equals，他们的hashcode有可能相等；
>如果两个对象hashcode相等，他们不一定equals；
> 如果两个对象hashcode不相等，他们一定不equals；

### **序列化**与反序列化

**（1）序列化的实现方式有哪些，有什么区别，代码如何实现？**

- 实现Serializable接口

- 实现Externalizable接口，实现writeExternal、readExternal方法

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
    try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ExPerson.txt"));
         ObjectInputStream ois = new ObjectInputStream(new FileInputStream("ExPerson.txt"))) {
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

- 在进行序列化工作时，会将serialVersioinUid与所要序列化的目标一起序列化，这样一来，在反序列化的过程中会使用被序列化的serialVersioinUid与类中的serialVersioinUid对比，如果两者相等，则反序列化成功，否则，反序列化失败。

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
- JDK1.7 中，ConcurrentHashMap 采用 HashEntry+Segment 的结构，ConcurrentHashMap 里一共 16 个 Segment，Segment 是可重入锁 ReentrantLock 的子类，每个 Segment 对应一个 HashEntry 键值对数组。当对 HashEntry 数组的数据进行修改时，必须首先获得对应的 Segment 锁，因此，多线程访问容器里不同 Segment 的数据，就不会存在锁竞争，从而提升并发性能。
- JDK1.8 中则摒弃了 Segment 的概念，并发控制使用 synchronized 和 CAS 来操作，虽然在 JDK1.8 中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本。来看看核心的 put 方法。JDK1.8 中的 ConcurrentHashMap 主要通过小范围的加锁 (synchronizded) 以及大量的 CAS 操作来实现 put 方法的线程安全；而 get () 方法则没有加锁。

>不可重入锁：只判断这个锁有没有被锁上，只要被锁上申请锁的线程都会被要求等待；
>
>可重入锁：不仅判断锁有没有被锁上，还会判断锁是谁锁上的，当就是自己锁上的时候，那么他依旧可以再次访问临界资源，并把加锁次数加一；
>
> 设计了加锁次数以在解锁的时候可以确保所有加锁的过程都解锁了，其他线程才能访问。不然没有加锁的参考值，也就不知道什么时候解锁？解锁多少次？才能保证本线程已经访问完临界资源了可以唤醒其他线程访问了；
>
>这个重入的概念就是，拿到锁的代码能不能多次以不同的方式访问临界资源而不出现死锁等相关问题。经典之处在于判断了需要使用锁的线程是否为加锁的线程。如果是，则拥有重入能力。
>

### List和Set

**（1）ArrayList和LinkedList的相同点和不同点是什么？**

- 相同点：ArrayList 和 LinkedList 都是 List 接口的实现类，因此都具有 List 的特点，即存取有序，可重复；而且都不是线程安全的。

- 不同点：ArrayList是实现了基于动态数组的数据结构，LinkedList基于双向链表的数据结构。

  > 对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
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

**（7）为什么 ConcurrentHashMap 的 key 和 value 不能为 null？为什么 HashMap 可以呢？**

- 会产生二义性：这个key从来没有在map中映射过；这个key的value在设置的时候，就是null；

- HashMap在单线程中可以用hashMap.containsKey(key)方法来区分含义；

- 线程A调用concurrentHashMap.get(key)方法，有一个线程B执行了concurrentHashMap.put(key,null)的操作，于是无法区分含义；

## 3.JVM

### 内存结构

**（1）JVM的内存结构及用处是什么？**

JVM 内存的主要分为五个区：方法区（Method Area），虚拟机栈（VM Stack），本地方法栈（Native method stack），堆（Heap），程序计数器（Program Counter Register）。

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

  > 新生代GC（Minor GC）:指发生新生代的的垃圾收集动作，Minor GC非常频繁，回收速度一般也比较快，每次进行清理时，将Eden区和一个Survivor中仍然存活的对象拷贝到 另一个Survivor中，然后清理掉Eden和刚才的Survivor。
  >
  > 老年代GC（Major GC/Full GC）:指发生在老年代的GC，出现了Major GC经常会伴随至少一次的Minor GC（并非绝对），Major GC的速度一般会比Minor GC的慢10倍以上；


**（4）你们常用的垃圾收集器及特点和工作过程是什么？**

- 新生代的垃圾回收器是 ParNewGC，老年代的回收器是 ConcMarkSweepGC（CMS，并发标记清除 GC)。

> ParNew 是串行收集器（Serial GC）的多线程版本，会使用多个 CPU 和线程完成垃圾收集工作（默认使用的线程数和 CPU 数相同，可以使用 - XX：ParallelGCThreads 进行调整）。
>
> CMS 是一种以最短回收停顿时间为目标的收集器，基于标记 - 清除（Mark-Sweep）算法实现的，主要针对老年代进行回收，其处理过程有七个步骤：
>
> 1. **初始标记 (CMS-initial-mark)** ，从 GC Roots 开始，扫描和 GC Roots 直接关联的对象并标记。该步骤会导致 **STW**(Stop The World，即虚拟机暂停正在执行的任务)；
> 2. **并发标记 (CMS-concurrent-mark)**，从步骤 1 中标记过的对象出发，所有可到达的对象都在本阶段中标记， **该阶段与用户线程同时运行**；
> 3. **并发预清理（CMS-concurrent-preclean）**，标记从**新生代晋升的对象**、**新分配到老年代的对象**以及在**并发阶段被修改了的对象**。通过重新扫描，减少后面第 5 步 "重新标记" 的工作，**该阶段与用户线程同时运行**；
> 4. **可被终止的预清理（CMS-concurrent-abortable-preclean）**，这个阶段会尽量尝试着承担 STW 的 Final Remark 阶段的工作。其持续的时间依赖因素较多（通常持续时间较长），因为它是重复做相同的事情直到发生 aboart 的条件（比如：重复的次数、多少量的工作、持续的时间等等）之一才会停止，**该阶段与用户线程同时运行**；
> 5. **重新标记 (CMS-remark)** ，收集器线程扫描在 CMS 堆中剩余的对象并进行标记， **是第二个并且是最后一个 STW 的阶段**；
> 6. **并发清除 (CMS-concurrent-sweep)**，清理垃圾对象，**与用户线程同时运行；**
> 7. **并发重置 (CMS-concurrent-reset)**，这个阶段，重置 CMS 收集器的数据结构，等待下一次垃圾回收， **与用户线程同时运行。**

**（5）常用的JVM参数配置有哪些？**

| JVM 参数                                                     | 说明                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Xms                                                          | 初始堆大小                                                   |
| Xmx                                                          | 最大堆大小                                                   |
| Xmn                                                          | 年轻代大小                                                   |
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
public ThreadPoolExecutor(  int corePoolSize,   //常驻线程数,即使空闲时仍保留在池中的线程数  
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

# 二、数据库

## 1.数据库基础

**（1）MySQL 支持哪些存储引擎？他们的特点是什么？**

- **InnoDB** 支持事务，行级锁定和外键，是事务型数据库的首选引擎；MySQL5.5.5 之后的默认存储引擎；

- **MyISAM** 拥有较高的插入、查询速度，但不支持事务。MySQL5.5.5 之前的默认存储引擎；

- **Memory** 基于散列，存储在内存中，对临时表有用。常见的应用场景是：临时存放数据，数据量不大，不需要较高的数据安全性；

- **Archive** 支持高并发的插入操作，但是本身不是事务安全的。常见的应用场景：存储归档数据，如记录日志信息可以使用 Archive。

  > InnoDB 和 MyISAM的区别：
  >
  > InnoDB 支持事务；而 MyISAM 不支持事物，强调的是性能，查询速度更快；
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

- 数据库系统利用了磁盘预读原理和磁盘预读，将一个节点的大小设为等于一个页，这样每个节点只需要一次 I/O 就可以完全载入。而 B + 树的高度是 2~4，检索一次最多只需要访问 4 个节点（4 次，即树的高度）。

  > B + 树索引并不能直接找到具体的行，只是找到被查找行所在的页，然后 DB 通过把整页读入内存（磁盘预读），再在内存中查找（局部性原理，即当一个数据被用到时，其附近的数据也通常会马上被使用）。

- B + 树所有的 Data 域在叶子节点，其余节点用来索引，而 B 树是每个索引节点都会有 Data 域；并且 B + 树所有叶子节点之间都有一个链指针。 这样遍历叶子节点就能获得全部数据，从而支持区分查询。在数据库中基于范围的查询是非常频繁的，而 B 树不支持这样的遍历操作。

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

## 3.事务

**（1）描述一下事务的几种隔离机制？**

- **Read Uncommitted（读未提交）**：事务可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少。而且可能出现**脏读（Dirty Read）**，即一个事务读取到另一个事务还有没提交的记录。
-  **Read Committed（读提交）** ：事务只能看见已经提交的事务所做的改变。这种隔离级别可能导致**不可重复读**（Nonrepeatable Read），即同一事务的其他实例在该实例处理其间可能会有新的 commit，所以同一个查询操作执行两次或多次的结果不一致。
- **Repeatable Read（可重复读）** ， 事务的多个实例在并发读取数据时读到同样的数据行。不过理论上，这会导致另一个棘手的问题：**幻读（PhantomRead）**，它是指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的 “幻影” 数据。
- **Serializable（串行化）** ：通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。

