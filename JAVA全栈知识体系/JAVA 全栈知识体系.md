# JAVA 全栈知识体系

#### Jdk Jre Jvm 的关系

1、JDK（*java development kit*）是Java开发工具包，其中包括**编译工具(javac.exe)打包工具(jar.exe)等，也包括JRE**。在JDK的安装目录下有一个jre目录，里面有两个文件夹bin和lib，在这里可以认为bin里的就是jvm，lib中则是jvm工作所需要的类库，而jvm和 lib合起来就称为jre

2、JRE，即Java运行环境，包含JVM标准实现（Jvm虚拟机）与Java核心类库。
单独有JVM不能执行class，因为在解释class的时候JVM需要调用解释所需要的类库lib（jre里有运行.class的java.exe）。在JRE目录里有bin和lib两个文件夹，可以认为**bin里的就是jvm，lib中则是jvm工作所需要的类库，而jvm和 lib和起来就称为jre**。所以，当写完java程序编译成.class之后，可以把这个.class文件和jre一起打包发给朋友，这样你的朋友就可以运行你写程序了。

3、JVM，即java虚拟机，是java运行时的环境，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。**Java虚拟机在执行字节码时，把字节码解释成具体平台上的机器指令执行**，这就是Java能够“一次编译，到处运行”的原因。  

## java 基础

##### 泛型

引入泛型的意义在于：

- **适用于多种数据类型执行相同的代码**（代码复用）

泛型有三种使用方式，分别为：**泛型类、泛型接口、泛型方法**。

```java
<?> 无限制通配符
<? extends E> extends 关键字声明了类型的上界，表示参数化的类型可能是所指定的类型，或者是此类型的子类
<? super E> super 关键字声明了类型的下界，表示参数化的类型可能是指定的类型，或者是此类型的父类

// 使用原则《Effictive Java》
// 为了获得最大限度的灵活性，要在表示 生产者或者消费者 的输入参数上使用通配符，使用的规则就是：生产者有上限、消费者有下限
1. 如果参数化类型表示一个 T 的生产者，使用 < ? extends T>;
2. 如果它表示一个 T 的消费者，就使用 < ? super T>；
3. 如果既是生产又是消费，那使用通配符就没什么意义了，因为你需要的是精确的参数类型。
```

**泛型的类型擦除原则**是：

- 消除类型参数声明，即删除`<>`及其包围的部分。
- 根据类型参数的上下界推断并替换所有的类型参数为原生态类型：如果类型参数是无限制通配符或没有上下界限定则替换为**Object**，如果存在上下界限定则根据子类替换原则取类型参数的最左边限定类型（即父类）。
- 为了保证类型安全，必要时插入强制类型转换代码。
- 自动产生“**桥接方法**”以保证擦除类型后的代码仍然具有泛型的“多态性”。



1. 既然说类型变量会在编译的时候擦除掉，那为什么我们往 ArrayList 创建的对象中添加整数会报错呢？**

   Java编译器是通过(在编译之前的检查)先检查代码中泛型的类型，然后在进行类型擦除，再进行编译。

   **类型检查就是针对引用的，谁是一个引用，用这个引用调用泛型方法，就会对这个引用调用的方法进行类型检测，而无关它真正引用的对象**。

   

2. **类型擦除会造成多态的冲突，而JVM解决方法就是桥接方法。**

3. **如何理解基本类型不能作为泛型类型？**

   因为当类型擦除后，ArrayList的原始类型变为Object，但是Object类型不能存储int值，只能引用Integer的值。

   **如何理解泛型类型不能实例化？**

   因为在 **Java 编译期没法确定泛型参数化类型**，也就找不到对应的类字节码文件，所以自然就不行了，此外由于`T` 被擦除为 `Object`，如果可以 `new T()` 则就变成了 `new Object()`，失去了本意。   如果我们确实需要实例化一个泛型，应该如何做呢？可以通过反射实现：

   ```java
   static <T> T newTclass (Class < T > clazz) throws InstantiationException, IllegalAccessException {
       T obj = clazz.newInstance();
       return obj;
   }
   ```

   **如何理解泛型类中的静态方法和静态变量？**

   > 泛型类中的静态方法和静态变量不可以使用泛型类所声明的泛型类型参数
   >
   > 因为泛型类中的泛型参数的实例化是在**定义对象的时候指定的**，而静态变量和静态方法不需要使用对象来调用。对象都没有创建，如何确定这个泛型参数是何种类型，所以当然是错误的。

   **如何理解异常中使用泛型？**

   - **不能抛出也不能捕获泛型类的对象**。事实上，泛型类扩展Throwable都不合法。例如：下面的定义将不会通过编译：

   ```java
   public class Problem<T> extends Exception {
   
   }
   ```

4. **如何获取泛型的参数类型？**

   反射

   ```
   public class GenericType<T> {
       private T data;
   
       public T getData() {
           return data;
       }
   
       public void setData(T data) {
           this.data = data;
       }
   
       public static void main(String[] args) {
           GenericType<String> genericType = new GenericType<String>() {};
           Type superclass = genericType.getClass().getGenericSuperclass();
           //getActualTypeArguments 返回确切的泛型参数, 如Map<String, Integer>返回[String, Integer]
           Type type = ((ParameterizedType) superclass).getActualTypeArguments()[0]; 
           System.out.println(type);//class java.lang.String
       }
   }
   ```

   ## java 集合

   #### String

   缓存池：Integer.valueOf(123)  【-128-128】

   使用 String.intern() 可以保证相同内容的字符串变量引用同一的内存对象。

   intern() 首先把 s1 引用的对象放到 String Pool(字符串常量池)中，然后返回这个对象引用。

   如果是采用 "bbb" 这种使用**双引号**的形式创建字符串实例，会自动地将新建的对象放入 String Pool 中。

   

   #### static初始化顺序

   静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于它们在**代码中的顺序**。

   存在继承的情况下，初始化顺序为:

   - 父类(静态变量、静态语句块)

- 子类(静态变量、静态语句块)

  - 父类(实例变量、普通语句块)

- 父类(构造函数)

  - 子类(实例变量、普通语句块)

- 子类(构造函数)

  

**变量**也就是我们所说的编译期常量

   

   ##### Java 中，Serializable 与 Externalizable 的区别?

   Serializable 接口是一个序列化 Java 类的接口，以便于它们可以在网络上传输或者可以将它们的状态保存在磁盘上，是 JVM 内嵌的默认序列化方式，成本高、脆弱而且不安全。Externalizable 允许你控制整个序列化过程，指定特定的二进制格式，增加安全机制。

   ##### switch能否用String做参数

   Java1.7开始支持，但实际这是一颗Java语法糖。除此之外，byte，short，int，枚举均可用于switch，而boolean和浮点型不可以。

   

   ### 序列化

   声明为static和transient类型的数据不能被序列化， 反序列化需要一个无参构造函数

   

   ### 反射调用流程小结

**反射的概念**

 1、 Java的反射机制是在运行状态中，对于任意一个类，都能够获得这个类的所有属性和方法；对于任意一个对象，都能够调用它的任何方法和属性。这种动态获取类的信息以及动态调用方法的功能称为Java语言的反射(reflection)机制。

2、反射的作用
        Java程序中一些对象在运行时会出现两种类型：**运行时类型和编译时类型**，例如一个向上转型的代码：Bird bird = new Animal();其中，bird在编译时类型为Bird，运行时类型为Animal，通过反射机制就能在程序运行时发现该对象和类的真实信息。

   最后，用几句话总结反射的实现原理：

      1. 反射类及反射方法的获取，都是通过从列表中搜寻查找匹配的方法，所以查找性能会随类的大小方法多少而变化；
      2. 每个类都会有一个与之对应的Class实例，从而每个类都可以获取method反射方法，并作用到其他实例身上；
      3. 反射也是考虑了线程安全的，放心使用；
      4. 反射使用软引用relectionData缓存class信息，避免每次重新从jvm获取带来的开销；
      5. 反射调用多次生成新代理Accessor, 而通过字节码生存的则考虑了卸载功能，所以会使用独立的类加载器；
      6. 当找到需要的方法，都会copy一份出来，而不是使用原来的实例，从而保证数据隔离；
      7. 调度反射方法，最终是由jvm执行invoke0()执行；

------

   ### 什么是SPI机制

   SPI（Service Provider Interface），是JDK内置的一种 服务提供发现机制，可以用来启用框架扩展和替换组件，主要是被框架的开发人员使用，比如java.sql.Driver接口，其他不同厂商可以针对同一接口做出不同的实现，MySQL和PostgreSQL都有不同的实现提供给用户，而Java的SPI机制可以为某个接口寻找服务实现。Java中SPI机制主要思想是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要，其核心思想就是 **解耦**。

------

   

   ![img](https://www.pdai.tech/images/java/java-advanced-spi-8.jpg)

   当服务的提供者提供了一种接口的实现之后，需要在classpath下的`META-INF/services/`目录里创建一个以服务接口命名的文件，这个文件里的内容就是这个接口的具体的实现类。当其他的程序需要这个服务的时候，就可以通过查找这个jar包（一般都是以jar包做依赖）的`META-INF/services/`中的配置文件，配置文件中有接口的具体实现类名，可以根据这个类名进行加载实例化，就可以使用该服务了。JDK中查找服务的实现的工具类是：`java.util.ServiceLoader`。

   ![img](https://www.pdai.tech/images/java/java-advanced-spi-2.jpg)

------

   ***ArrayDeque***底层通过数组实现，为了满足可以同时在数组两端插入或删除元素的需求，该数组还必须是循环的，即**循环数组(circular array)**，也就是说数组的任何一点都可能被看作起点或者终点。*ArrayDeque*是**非线程安全**的(not thread-safe)，当多个线程同时使用的时候，需要程序员手动同步；另外，该容器**不允许放入`null`**元素。

------

   **Fail-Fast机制:**

ArrayList也采用了快速失败的机制，通过记录modCount参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险

   **HashMap中HashCode的实现原理**

   https://blog.csdn.net/qq_46548855/article/details/129772345

   ```
   static final int hash(Object key) {
           int h;
           return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
   ```

   ```
   if ((p = tab[i = (n - 1) & hash]) == null)
               tab[i] = newNode(hash, key, value, null);
   ```

### TreeSet和HashMap

***TreeMap\*底层通过红黑树(Red-Black tree)实现**

**红黑树是一种近似平衡的二叉查找树，它能够确保任何一个节点的左右子树的高度差不会超过二者中较低那个的一倍**。具体来说，红黑树是满足如下条件的二叉查找树(binary search tree):

1. 每个节点要么是红色，要么是黑色。

2. 根节点必须是黑色

3. 红色节点不能连续(也即是，红色节点的孩子和父亲都不能是红色)。

4. 对于每个节点，从该点至`null`(树尾端)的任何路径，都含有相同个数的黑色节点。

   **左旋**

   左旋的过程是将`x`的右子树绕`x`逆时针旋转，使得`x`的右子树成为`x`的父亲，同时修改相关节点的引用。

![TreeMap_rotateLeft.png](https://www.pdai.tech/images/collection/TreeMap_rotateLeft.png)

   **右旋**

   右旋的过程是将`x`的左子树绕`x`顺时针旋转，使得`x`的左子树成为`x`的父亲，同时修改相关节点的引用。

![TreeMap_rotateRight.png](https://www.pdai.tech/images/collection/TreeMap_rotateRight.png)

   **寻找节点后继**

   对于一棵二叉查找树，给定节点t，其后继(树中比大于t的最小的那个元素)可以通过如下方式找到:

      1. t的右子树不空，则t的后继是其右子树中最小的那个元素。
      2. t的右孩子为空，则t的后继是其第一个向左走的祖先。

### LinkedHashMap经典用法

*LinkedHashMap*除了可以保证迭代顺序外，还有一个非常有用的用法: 可以轻松实现一个采用了FIFO替换策略的缓存。具体说来，LinkedHashMap有一个子类方法`protected boolean removeEldestEntry(Map.Entry eldest)`，该方法的作用是告诉Map是否要删除“最老”的Entry，所谓最老就是当前Map中最早插入的Entry，如果该方法返回`true`，最老的那个元素就会被删除。在每次插入新元素的之后LinkedHashMap会自动询问removeEldestEntry()是否要删除最老的元素。这样只需要在子类中重载该方法，当元素个数超过一定数量时让removeEldestEntry()返回true，就能够实现一个固定大小的FIFO策略的缓存。示例代码如下:

```java
/** 一个固定大小的FIFO替换策略的缓存 */
class FIFOCache<K, V> extends LinkedHashMap<K, V>{
    private final int cacheSize;
    public FIFOCache(int cacheSize){
        this.cacheSize = cacheSize;
    }

    // 当Entry个数超过cacheSize时，删除最老的Entry
    @Override
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
       return size() > cacheSize;
    }
}
```

### WeakHashMap

*WeakHashMap*，从名字可以看出它是某种 *Map*。它的特殊之处在于 *WeakHashMap* 里的`entry`可能会被GC自动删除，即使程序员没有调用`remove()`或者`clear()`方法。

***WeakHashMap\* 的这个特点特别适用于需要缓存的场景**。在缓存场景下，由于内存是有限的，不能缓存所有对象；对象缓存命中可以提高系统效率，但缓存MISS也不会造成错误，因为可以通过计算重新得到。

是否有 *WeekHashSet* 呢? 答案是没有:( 。不过Java *Collections*工具类给出了解决方案，`Collections.newSetFromMap(Map map)`方法可以将任何 *Map*包装成一个*Set*。通过如下方式可以快速得到一个 *Weak HashSet*:

```
// 将WeakHashMap包装成一个Set
Set<Object> weakHashSet = Collections.newSetFromMap(
        new WeakHashMap<Object, Boolean>());
```

### sql执行顺序

![image-20240110174404660](C:\Users\1\Desktop\JAVA全栈知识体系\JAVA 全栈知识体系.assets\image-20240110174404660.png)

## java线程

#### 为什么需要多线程

众所周知，CPU、内存、I/O 设备的速度是有极大差异的，为了合理利用 CPU 的高性能，平衡这三者的速度差异，计算机体系结构、操作系统、编译程序都做出了贡献，主要体现为:

- CPU 增加了缓存，以均衡与内存的速度差异；// 导致 `可见性`问题
- 操作系统增加了进程、线程，以分时复用 CPU，进而均衡 CPU 与 I/O 设备的速度差异；// 导致 `原子性`问题
- 编译程序优化指令执行次序，使得缓存能够得到更加合理地利用。// 导致 `有序性`问题

##### 并发出现问题的根源：并发三要素

可见性：cpu缓存引起的

原子性：分时复用（线程切换）引起的

​		原子性：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

有序性：重排序引起的

​	在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。重排序分三种类型：

- **编译器优化的重排序**。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。

- **指令级并行的重排序**。现代处理器采用了指令级并行技术（Instruction-Level Parallelism， ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。

- **内存系统的重排序**。由于处理器使用缓存和读 / 写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

  从 java 源代码到最终实际执行的指令序列，会分别经历下面三种重排序：

![img](https://www.pdai.tech/images/jvm/java-jmm-3.png)



#### Java是怎么解决并发问题的？（JMM java 内存模型）

**关键字：volatile,synchronized和final**

Java提供了**volatile**关键字来保证可见性，通过**synchronized和Lock**也能够保证可见性

可以通过**volatile**关键字来保证一定的“有序性”。另外可以通过**synchronized和Lock**来保证有序性，当然JMM是通过Happens-Before 规则来保证有序性的。

**Happens-Before规则**

**单一线程规则：**在一个线程内，在程序前面的操作先行发生于后面的操作。

**管程锁定规则：**一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。

**volatile变量规则：**对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。

**线程启动规则：**Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。

**线程加入规则：**Thread 对象的结束先行发生于 join() 方法返回。

**线程中断规则：**对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 interrupted() 方法检测到是否有中断发生。

**对象终结规则：**一个对象的初始化完成(构造函数执行结束)先行发生于它的 finalize() 方法的开始。

**传递性：**如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。

#### 线程安全：不是一个非真即假的命题

1. 不可变

   不可变(Immutable)的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。

2. 绝对线程安全

   不管运行时环境如何，调用者都不需要任何额外的同步措施。

3. 相对线程安全

   相对线程安全需要保证对**这个对象单独的操作**是线程安全的，在调用的时候不需要做额外的保障措施。但是对于一些特定顺序的连续调用，就可能需要在调用端使用额外的同步手段来保证调用的正确性。

   在 Java 语言中，大部分的线程安全类都属于这种类型，例如 Vector、HashTable、Collections 的 synchronizedCollection() 方法包装的集合等。

4. 线程兼容

   线程兼容是指**对象本身并不是线程安全的**，但是可以通过在调用端正确地使用同步手段来保证对象在并发环境中可以安全地使用，我们平常说一个类不是线程安全的，绝大多数时候指的是这一种情况。Java API 中大部分的类都是属于线程兼容的，如与前面的 Vector 和 HashTable 相对应的集合类 ArrayList 和 HashMap 等。

5. 线程对立

   线程对立是指无论调用端是否采取了同步措施，都**无法在多线程环境中并发使用的代码**。由于 Java 语言天生就具备多线程特性，线程对立这种排斥多线程的代码是很少出现的，而且通常都是有害的，应当尽量避免。

##### 线程安全的实现方法

1. 互斥同步（阻塞同步）

   synchronized 和 ReentrantLock。

2. 非阻塞同步

   **CAS(compare and swap) :** CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

   AtomicInteger：J.U.C 包里面的整数原子类 AtomicInteger，其中的 compareAndSet() 和 getAndIncrement() 等方法都使用了 **Unsafe 类的 CAS 操作**。

   **ABA**：如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。（**添加版本号来解决**）

3. 无同步方案

   要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

   **栈封闭**、**线程本地存储(Thread Local Storage)**、**可重入代码(Reentrant Code)**

##### 线程的几种状态

**新建（New）**：创建后尚未启动。

**就绪（Runnable）**：可能正在运行，也可能正在等待 CPU 时间片。包含了操作系统线程状态中的 Running 和 Ready。

**阻塞（Blocking）**：等待获取一个排它锁，如果其线程释放了锁就会结束此状态。

**无限期等待（Waiting）**：等待其它线程显式地唤醒，否则不会被分配 CPU 时间片。

| 进入方法                                   | 退出方法                             |
| ------------------------------------------ | ------------------------------------ |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
| LockSupport.park() 方法                    | -                                    |

**限期等待（Timed Waiting）**：无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

| 进入方法                                 | 退出方法                                        |
| ---------------------------------------- | ----------------------------------------------- |
| Thread.sleep() 方法                      | 时间结束                                        |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                 |
| LockSupport.parkNanos() 方法             | -                                               |
| LockSupport.parkUntil() 方法             | -                                               |

**死亡（Terminated）**

可以是线程结束任务之后自己结束，或者产生了异常而结束。

##### 线程的使用方式

- 实现 Runnable 接口；
- 实现 Callable 接口；
- 继承 Thread 类。

##### 实现接口 VS 继承 Thread

实现接口会更好一些，因为:

- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。

##### 基础线程机制

**Executor**：主要有三种 Executor:

- CachedThreadPool: 一个任务创建一个线程；
- FixedThreadPool: 所有任务只能使用固定大小的线程；
- SingleThreadExecutor: 相当于大小为 1 的 FixedThreadPool。

**Daemon**：守护线程是程序运行时在后台提供服务的线程，**不属于**程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时**会杀死所有守护线程**。

main() 属于非守护线程。

**sleep()**：Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

**yield()**：对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

##### 线程中断

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

**InterruptedException**

通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于**阻塞、限期等待或者无限期等待状态**，那么就会抛出 **InterruptedException**，从而提前结束该线程。但是**不能中断 I/O 阻塞和 synchronized 锁阻塞**。

**interrupted()**

如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会**设置线程的中断标记**，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来**判断线程是否处于中断状态**，从而提前结束线程。

**Executor 的中断操作**

调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 **shutdownNow()** 方法，则相当于**调用每个线程的 interrupt()** 方法。

##### 线程互斥同步

Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 **synchronized**，而另一个是 JDK 实现的 **ReentrantLock**。

###### 比较

**1. 锁的实现**

synchronized 是 **JVM** 实现的，而 ReentrantLock 是 **JDK** 实现的。

**2. 性能**

**新版本** Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock **大致相同**。

**3. 等待可中断**

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

**ReentrantLock 可中断，而 synchronized 不行**。

**4. 公平锁**

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是**非公平**的，ReentrantLock 默认情况下**也是非公平的，但是也可以是公平的**。

**5. 锁绑定多个条件**

一个 ReentrantLock 可以同时绑定多个 Condition 对象。

**[#](#使用选择) 使用选择**

除非需要使用 ReentrantLock 的高级功能，否则**优先使用 synchronized**。这是因为 synchronized 是 JVM 实现的一种锁机制，**JVM 原生地支持它**，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 **JVM 会确保锁的释放**。

##### 线程之间的协作

**join()**：在线程中调用另一个线程的 join() 方法，会将当前线程**挂起**，而不是忙等待，直到目标线程结束。

**wait() notify() notifyAll()**：调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。wait()会释放锁

**wait() 和 sleep() 的区别**

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- **wait() 会释放锁，sleep() 不会。**

**await() signal() signalAll()**：java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 **signal() 或 signalAll() 方法唤醒等待的线程**。相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

### Java中所有锁

![img](https://www.pdai.tech/images/thread/java-lock-1.png)

- **悲观锁适合写操作多的场景**，先加锁可以保证写操作时数据正确。
- **乐观锁适合读操作多的场景**，不加锁的特点能够使其读操作的性能大幅提升。

**自旋锁和适应性自旋锁**：**阻塞或唤醒**一个Java线程需要操作系统切换CPU状态来完成，这种状态转换需要**耗费处理器时间。**

而为了让当前线程“稍等一下”，我们需让**当前线程进行自旋**，如果在自旋完成后**前面锁定同步资源的线程**已经**释放了锁**，那么当前线程就可以不必阻塞而是直接获取同步资源，从而避免切换线程的开销。这就是自旋锁。

如果锁被占用的时间很长，那么自旋的线程只会白浪费处理器资源。所以，自旋等待的时间必须要有一定的限度，如果自旋超过了限定次数（默认是10次，可以使用-XX:PreBlockSpin来更改）没有成功获得锁，就应当挂起线程。

![img](https://www.pdai.tech/images/thread/java-lock-6.png)

总结而言： **偏向锁通过对比Mark Word解决加锁问题，避免执行CAS操作。而轻量级锁是通过用CAS操作和自旋来解决加锁问题，避免线程阻塞和唤醒而影响性能。重量级锁是将除了拥有锁的线程以外的线程都阻塞。**

**可重入锁 VS 非可重入锁**

可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会**自动获取锁**（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。Java中**ReentrantLock和synchronized**都是可重入锁，可重入锁的一个优点是可一定程度避免死锁。

首先ReentrantLock和NonReentrantLock都继承父类AQS，其父类AQS中维护了一个同步状态status来计数重入次数，status初始值为0。

![img](https://www.pdai.tech/images/thread/java-lock-14.png)

**独享锁(排他锁) VS 共享锁**

**独享锁也叫排他锁**，是指该锁一次只能被一个线程所持有。如果线程T对数据A加上排它锁后，则其他线程不能再对A加任何类型的锁。获得排它锁的线程即能读数据又能修改数据。JDK中的**synchronized和JUC中Lock的实现类**就是互斥锁。

**共享锁**是指该锁可被多个线程所持有。如果线程T对数据A加上共享锁后，则其他线程只能对A再加共享锁，不能加排它锁。获得**共享锁的线程只能读数据**，**不能修改数据**。

在**ReentrantReadWriteLock**里面，读锁和写锁的锁主体都是Sync，但读锁和写锁的加锁方式不一样。**读锁是共享锁，写锁是独享锁。**

### 关键字: synchronized详解

![image-20240115172524709](C:\Users\1\Desktop\JAVA全栈知识体系\JAVA 全栈知识体系.assets\image-20240115172524709.png)

#### 问题：

**Synchronized本质上是通过什么保证线程安全的?** 分三个方面回答：加锁和释放锁的原理，可重入原理，保证可见性原理。

**Synchronized由什么样的缺陷?Java Lock是怎么弥补这些缺陷的。**

synchronized的缺陷

- `效率低`：锁的释放情况少，只有代码执行完毕或者异常结束才会释放锁；试图获取锁的时候不能设定超时，不能中断一个正在使用锁的线程，相对而言，Lock可以中断和设置超时
- `不够灵活`：加锁和释放的时机单一，每个锁仅有一个单一的条件(某个对象)，相对而言，读写锁更加灵活
- `无法知道是否成功获得锁`，相对而言，Lock可以拿到状态，如果成功获取锁，....，如果获取失败，.....

Lock解决相应问题

Lock类这里不做过多解释，主要看里面的4个方法:

- `lock()`: 加锁
- `unlock()`: 解锁
- `tryLock()`: 尝试获取锁，返回一个boolean值
- `tryLock(long,TimeUtil)`: 尝试获取锁，可以设置超时

**Synchronized和Lock的对比，和选择? Synchronized在使用时有何注意事项?**

锁对象不能为空，因为锁的信息都保存在对象头里

作用域不宜过大，影响程序执行的速度，控制范围过大，编写代码也容易出错

避免死锁

在能选择的情况下，既不要用Lock也不要用synchronized关键字，用java.util.concurrent包中的各种各样的类，如果不用该包下的类，在满足业务的情况下，可以使用synchronized关键，因为代码量少，避免出错

#### Synchronized使得同时只有一个线程可以执行，性能比较差，有什么提升的方法?

1. 消除锁（判断该对象是否有多个线程调用，若无则消除该对象上的锁）
2. 使用轻量级锁
3. 锁粗化（在一个代码块代码总是释放锁又加锁的情况下，又是同一线程这时就会锁粗化把这一整块代码块加锁）

**什么是锁的升级和降级?** **什么是JVM里的偏斜锁、轻量级锁、重量级锁? 不同的JDK中对Synchronized有何优化?**
**锁的升级与降级：**为了减少获得锁和释放锁带来的性能开销，引入了偏向锁、轻量级锁的概念。锁的状态根据竞争激烈的程度从低到高不断升级或者降级。（偏向锁-》轻量级锁-》重量级锁）
**偏向锁：**当一个线程访问加了同步锁的代码块时，会在对象头中存储当前线程的id，后续这个线程进入和退出这个代码块时，不需要在次加锁，和释放锁，而是直接比较对象头里面是否存储了指向当前线程的偏向锁，相等就表示偏向锁偏向当前线程，不需要尝试获得锁
**轻量级锁：**轻量级意味着相比于传统的锁，它使用的是系统互斥量来实现的，目的是在 没有多线程竞争的情况下，减少系统互斥量操作产生的性能消耗（轻量级锁，先进行cas操作，然后若是没获得锁就进行自旋的操作，在短暂时间里再次去获取锁并且大几率能够获得锁，这就是轻量级锁）
**重量级锁：**两个线程之间的竞争不能得到很好的交替（自旋之后获得不到锁），会上升为重量级锁，
**JDk1.6**之后引入了偏向锁，和轻量级锁

在应用Sychronized关键字时需要把握如下注意点：

- **一把锁只能同时被一个线程获取**，没有获得锁的线程只能等待；
- 每个实例都对应有自己的一把锁(this),不同实例之间互不影响；例外：锁对象是*.class以及synchronized修饰的是static方法的时候，所有对象公用同一把锁
- synchronized修饰的方法，无论方法正常执行完毕还是抛出异常，**都会释放锁**

**对象锁**：方法锁，同步代码块锁

**可重入原理：加锁次数计数器**

**可重入锁**：又名递归锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。

**保证可见性的原理：内存模型和happens-before规则**

锁的类型：**锁膨胀方向： 无锁 → 偏向锁 → 轻量级锁 → 重量级锁 (此过程是不可逆的)**

##### JVM锁优化

**`锁粗化(Lock Coarsening)`：**也就是减少不必要的紧连在一起的unlock，lock操作，将多个连续的锁扩展成一个范围更大的锁。

**`锁消除(Lock Elimination)`：**被检测到不可能存在共享数据竞争的锁进行消除。锁消除的主要判定依据来源于逃逸分析的数据支持。意思就是：JVM会判断再一段程序中的同步明显不会逃逸出去从而被其他线程访问到，那JVM就把它们当作栈上数据对待，认为这些数据是线程独有的，不需要加同步。此时就会进行锁消除。

**`轻量级锁(Lightweight Locking)`：**这种锁实现的背后基于这样一种假设，即在真实的情况下我们程序中的大部分同步代码一般都处于无锁竞争状态(即单线程执行环境)，在无锁竞争的情况下完全可以避免调用操作系统层面的重量级互斥锁，取而代之的是在monitorenter和monitorexit中只需要依靠一条CAS原子指令就可以完成锁的获取及释放。当存在锁竞争的情况下，执行CAS指令失败的线程将调用操作系统互斥锁进入到阻塞状态，当锁被释放的时候被唤醒(具体处理步骤下面详细讨论)。

轻量级锁加锁：在线程执行同步块之前，JVM会先在当前线程的栈帧中创建一个名为**锁记录(`Lock Record`)**的空间，用于存储锁对象目前的`Mark Word`的拷贝(JVM会将对象头中的`Mark Word`拷贝到锁记录中，官方称为`Displaced Mark Ward`)。



![img](https://www.pdai.tech/images/thread/java-thread-x-key-schronized-7.png)

**`偏向锁(Biased Locking)`：**是为了在无锁竞争的情况下避免在锁获取过程中执行不必要的CAS原子指令，因为CAS原子指令虽然相对于重量级锁来说开销比较小但还是存在非常可观的本地延迟。

背景：在大多实际环境下，锁不仅不存在多线程竞争，而且总是由同一个线程多次获取，那么在同一个线程反复获取所释放锁中，其中并还没有锁的竞争，那么这样看上去，多次的获取锁和释放锁带来了很多不必要的性能开销和上下文切换。

当一个线程访问同步块并获取锁时，会在对象头和栈帧中的**锁记录里存储锁偏向的线程ID**，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁。只需要简单的测试一下对象头的`Mark Word`里**是否存储着指向当前线程的偏向锁**。如果成功，表示线程已经获取到了锁。

![img](https://www.pdai.tech/images/thread/java-thread-x-key-schronized-8.png)

**`适应性自旋(Adaptive Spinning)`：**当线程在获取轻量级锁的过程中执行CAS操作失败时，在进入与monitor相关联的操作系统重量级锁(mutex semaphore)前会进入忙等待(Spinning)然后再次尝试，当尝试一定的次数后如果仍然没有成功则调用与该monitor关联的semaphore(即互斥锁)进入到阻塞状态。

### 锁的优缺点对比

| 锁       | 优点                                                         | 缺点                                                         | 使用场景                           |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------- |
| 偏向锁   | 加锁和解锁不需要CAS操作，没有额外的性能消耗，和执行非同步方法相比仅存在纳秒级的差距 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗               | 适用于只有一个线程访问同步块的场景 |
| 轻量级锁 | 竞争的线程不会阻塞，提高了响应速度                           | 如线程始终得不到锁竞争的线程，使用自旋会消耗CPU性能          | 追求响应时间，同步块执行速度非常快 |
| 重量级锁 | 线程竞争不适用自旋，不会消耗CPU                              | 线程阻塞，响应时间缓慢，在多线程下，频繁的获取释放锁，会带来巨大的性能消耗 | 追求吞吐量，同步块执行速度较长     |

**什么是锁降级？**

锁降级指的是写锁降级成为读锁。

如果当前线程拥有写锁，然后将其释放，最后再获取读锁，这种分段完成的过程不能称之为锁降级。锁降级是指把持住（当前拥有的）写锁，再获取到读锁，随后释放（先前拥有的）写锁的过程。

### 关键字: volatile详解

##### 面试问题：

**volatile关键字的作用是什么?**

防重排序、实现可见性、保证原子性：单次读/写

**volatile能保证原子性吗?**

不能，只能保证单次读/写原子性

**之前32位机器上共享的long和double变量的为什么要用volatile? 现在64位机器上是否也要设置呢?**

因为long和double两种数据类型的操作可分为**高32位和低32位**两部分，因此普通的long或double类型读/写可能不是原子的。因此，鼓励大家将共享的long和double变量设置为volatile类型，这样能保证任何情况下对long和double的单次读/写操作都具有原子性。

目前各种平台下的商用虚拟机都选择把 64 位数据的读写操作作为原子操作来对待，因此我们在编写代码时一般不把long 和 double 变量专门声明为 volatile多数情况下也是不会错的。

**i++为什么不能保证原子性?**

对volatile变量的**单次读/写操作可以保证原子性的**，如long和double类型变量，但是并不能保证i++这种操作的原子性，因为本质上i++是读、写两次操作。

i++其实是一个复合操作，包括三步骤：

- 读取i的值。
- 对i加1。
- 将i的值写回内存。

**volatile是如何实现可见性的? 内存屏障。**

**volatile是如何实现有序性的? happens-before等**

**说下volatile的应用场景?**

使用 volatile 必须具备的条件

- 对变量的写操作不依赖于当前值。

- 该变量没有包含在具有其他变量的不变式中。

- 只有在状态真正独立于程序内其他内容时才能使用 volatile

  ##### 模式1：状态标志

  ​	也许实现 volatile 变量的规范使用仅仅是使用一个**布尔状态标志，用于指示发生了一个重要的一次性事件**，例如完成初始化或请求停机。

  

  ##### 模式2：一次性安全发布(one-time safe publication)

  ​	**缺乏同步会导致无法实现可见性**，这使得确定何时写入对象引用而不是原始值变得更加困难。

  ​	在缺乏同步的情况下，可能会遇到某个对象引用的更新值(由另一个线程写入)和该对象状态的旧值同时存在。(这就是造成著名的双重检查锁定(double-checked-locking)问题的根源，其中对象引用在没有同步的情况下进行读操作，产生的问题是您可能会看到一个更新的引用，但是仍然会通过该引用看到不完全构造的对象)。

  

  ##### 模式3：独立观察(independent observation)

  ​	安全使用 volatile 的另一种简单模式是定期 发布 观察结果供程序内部使用。例如，假设有一种环境传感器能够感觉环境温度。一个后台线程可能会每隔几秒读取一次该传感器，并更新包含当前文档的 volatile 变量。然后，其他线程可以读取这个变量，从而随时能够看到最新的温度值。

  

  ##### 模式4：volatile bean 模式

  ​	在 volatile bean 模式中，JavaBean 的所有数据成员都是 volatile 类型的，并且 getter 和 setter 方法必须非常普通 —— 除了获取或设置相应的属性外，不能包含任何逻辑。此外，对于对象引用的数据成员，引用的对象必须是有效不可变的。(这将禁止具有数组值的属性，因为当数组引用被声明为 volatile 时，只有引用而不是数组本身具有 volatile 语义)。对于任何 volatile 变量，不变式或约束都不能包含 JavaBean 属性。

  

  ##### 模式5：开销较低的读－写锁策略

  ​	volatile 的功能还不足以实现计数器。因为 ++x 实际上是三种操作(读、添加、存储)的简单组合，如果多个线程凑巧试图同时对 volatile 计数器执行增量操作，那么它的更新值有可能会丢失。 如果读操作远远超过写操作，可以结合使用内部锁和 volatile 变量来减少公共代码路径的开销。 **安全的计数器使用 synchronized 确保增量操作是原子的，并使用 volatile 保证当前结果的可见性。**如果更新不频繁的话，该方法可实现更好的性能，因为读路径的开销仅仅涉及 volatile 读操作，这通常要优于一个无竞争的锁获取的开销。

  

  ##### 模式6：双重检查(double-checked)

  ​	单例模式的一种实现方式，但很多人会忽略 volatile 关键字，因为没有该关键字，程序也可以很好的运行，只不过代码的稳定性总不是 100%，说不定在未来的某个时刻，隐藏的 bug 就出来了。

  ```
  class Singleton {
      private volatile static Singleton instance;
      private Singleton() {
      }
      public static Singleton getInstance() {
          if (instance == null) {
              syschronized(Singleton.class) {
                  if (instance == null) {
                      instance = new Singleton();
                  }
              }
          }
          return instance;
      } 
  }
  ```

  

### volatile 的实现原理

#### 可见性实现

volatile 变量的内存可见性是基于内存屏障(Memory Barrier)实现:，是一个cpu指令

在程序运行时，为了提高执行性能，编译器和处理器会对指令进行重排序，JMM **为了保证在不同的编译器和 CPU 上有相同的结果，**通过插入特定类型的内存屏障来禁止+ 特定类型的编译器重排序和处理器重排序，**插入一条内存屏障会告诉编译器和 CPU：不管什么指令都不能和这条 Memory Barrier 指令重排序。**

```bash
在 volatile 修饰的共享变量进行写操作的时候会多出 lock 前缀的指令
```

lock 前缀的指令在多核处理器下会引发两件事情:

- **将当前处理器缓存行的数据写回到系统内存。**
- **写回内存的操作会使在其他 CPU 里缓存了该内存地址的数据无效。**

#### 有序性实现

##### volatile 的 happens-before 关系

happens-before 规则中有一条是 volatile 变量规则：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读。

##### volatile 禁止重排序

为了性能优化，JMM 在**不改变正确语义**的前提下，会允许编译器和处理器对指令序列进行重排序。JMM 提供了内存屏障阻止这种重排序。

Java 编译器会在生成指令系列时在适当的位置会插入内存屏障指令来禁止特定类型的处理器重排序。



### 关键字: final详解

##### 问题：

**所有的final修饰的字段都是编译期常量吗?**

不是

````
public class Test {
    //编译期常量
    final int i = 1;
    final static int J = 1;
    final int[] a = {1,2,3,4};
    //非编译期常量
    //k的值由随机数对象决定，所以不是所有的final修饰的字段都是编译期常量，只是k的值在被初始化后无法被更改。
    Random r = new Random();
    final int k = r.nextInt();

    public static void main(String[] args) {

    }
}
````



**如何理解private所修饰的方法是隐式的final?**

类中所有private方法都隐式地指定为final的，由于无法取用private方法，所以也就不能覆盖它。可以对private方法增添final关键字，但这样做并没有什么好处。

**说说final类型的类如何拓展? 比如String是final类型，我们想写个MyString复用所有String中方法，同时增加一个新的toMyString()的方法，应该如何做?**

使用组合关系，如下代码

```
class MyString{

    private String innerString;

    // ...init & other methods

    // 支持老的方法
    public int length(){
        return innerString.length(); // 通过innerString调用老的方法
    }

    // 添加新方法
    public String toMyString(){
        //...
    }
}
```

**final方法可以被重载吗? 可以**

**父类的final方法能不能够被子类重写? 不可以**

**说说final域重排序规则?**

**说说final的原理?**

写final域会要求编译器在final域写之后，构造函数返回前插入一个StoreStore屏障。读final域的重排序规则会要求编译器在读final域的操作前插入一个LoadLoad屏障。

很有意思的是，如果以X86处理为例，X86不会对写-写重排序，所以StoreStore屏障可以省略。由于不会对有间接依赖性的操作重排序，所以在X86处理器中，读final域需要的LoadLoad屏障也会被省略掉。也就是说，以X86为例的话，对final域的读/写的内存屏障都会被省略！具体是否插入还是得看是什么处理器

**使用 final 的限制条件和局限性?**

当声明一个 final 成员时，必须在构造函数退出前设置它的值。

将指向对象的成员声明为 final 只能将该引用设为不可变的，而非所指的对象。下面的方法仍然可以修改该 list。

```
private final List myList = new ArrayList();
myList.add("Hello");
```

如果一个对象将会在多个线程中访问并且你并没有将其成员声明为 final，则必须提供其他方式保证线程安全。

" 其他方式 " 可以包括声明成员为 volatile，使用 synchronized 或者显式 Lock 控制所有该成员的访问。

#### final基础使用

**修饰类**

当某个类的整体定义为final时，就表明了你不能打算继承该类，而且也不允许别人这么做。即**这个类是不能有子类的**。

注意：final类中的**所有方法都隐式为final**，因为无法覆盖他们，所以在final类中给任何方法添加final关键字是没有任何意义的。

**修饰方法**

- private 方法是隐式的final
- final方法是可以被重载的

**修饰参数**

Java允许在参数列表中以声明的方式将参数指明为final，这意味这你**无法在方法中更改参数引用**所指向的对象。这个特性主要用来向匿名内部类传递数据。

**修饰变量**

**static final**

一个既是static又是final 的字段只占据一段不能改变的存储空间，它必须在定义的时候进行赋值，否则编译器将不予通过。

```
import java.util.Random;
public class Test {
    static Random r = new Random();
    final int k = r.nextInt(10);
    static final int k2 = r.nextInt(10); 
    public static void main(String[] args) {
        Test t1 = new Test();
        System.out.println("k="+t1.k+" k2="+t1.k2);
        Test t2 = new Test();
        System.out.println("k="+t2.k+" k2="+t2.k2);
    }
}
//输出结果
//k=2 k2=7
//k=8 k2=7
```

我们可以发现对于不同的对象k的值是不同的，但是k2的值却是相同的，这是为什么呢? 因为static关键字所修饰的字段并不属于一个对象，而是**属于这个类的**。也可简单的理解为static final所修饰的字段仅占据内存的一个一份空间，一旦被初始化之后便不会被更改。

**blank final**

Java允许生成空白final，也就是说被声明为final但又没有给出定值的字段,但是必须在该字段被使用之前被赋值，这给予我们两种选择：

- 在定义处进行赋值(这不叫空白final)
- 在**构造器中进行赋值**，保证了该值在被使用前赋值。

这增强了final的灵活性。



##### final重排序规则

- 基本数据类型

  **写final域的重排序规则**禁止对final域的写重排序到构造函数之外，这个规则的实现主要包含了两个方面：

  - **JMM禁止编译器把final域的写重排序到构造函数之外；**
  - 编译器会在final域写之后，构造函数return之前，插入一个storestore屏障。这个屏障可以禁止处理器把final域的写重排序到构造函数之外。

  **读final域的重排序规则**可以确保：在读一个对象的final域之前，一定会先读这个包含这个final域的对象的引用。

- 引用数据类型：

  ​		对final基本数据类型的重排序规则在这里还是使用

  - `额外增加约束`：禁止在构造函数对一个final修饰的对象的成员域的写入与随后将这个被构造的对象的引用赋值给引用变量 重排序

## JUC

- JUC框架包含几个部分?
  - Lock框架和Tools类
  - Collections: 并发集合
  - Atomic: 原子类
  - Executors: 线程池
- 每个部分有哪些核心的类?
- 最最核心的类有哪些?

### JUC原子类: CAS, Unsafe和原子类详解

##### 线程安全的实现方法包含:

- 互斥同步: synchronized 和 ReentrantLock
- 非阻塞同步: CAS, AtomicXXXX
- 无同步方案: 栈封闭，Thread Local，可重入代码

**什么是CAS?**

 简单解释：CAS操作需要输入两个数值，一个旧值(期望操作前的值)和一个新值，在操作期间先比较下在旧值有没有发生变化，如果没有发生变化，才交换成新值，发生了变化则不交换。

**CAS操作是原子性的**，所以多线程并发使用CAS更新数据时，可以不使用锁。JDK中大量使用了CAS来更新数据而防止加锁(synchronized 重量级锁)来保持原子更新。

**CAS问题**

CAS 方式为乐观锁，synchronized 为悲观锁。因此使用 CAS 解决并发问题通常情况下性能更优。

**ABA问题**

因为CAS需要在操作值的时候，检查值有没有发生变化，比如没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时则会发现它的值没有发生变化，但是实际上却变化了。

ABA问题的解决思路就是**使用版本号**。在变量前面追加上版本号，每次变量更新的时候把版本号加1，那么A->B->A就会变成1A->2B->3A。

从Java 1.5开始，JDK的Atomic包里提供了一个类**AtomicStampedReference**来解决ABA问题。

**循环时间长开销大**

自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令，那么效率会有一定的提升。pause指令有两个作用：第一，它可以延迟流水线执行命令(de-pipeline)，使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零；第二，它可以避免在退出循环的时候因内存顺序冲突(Memory Order Violation)而引起CPU流水线被清空(CPU Pipeline Flush)，从而提高CPU的执行效率。

**只能保证一个共享变量的原子操作**

当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁。

还有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如，有两个共享变量i = 2，j = a，合并一下ij = 2a，然后用CAS来操作ij。

从Java 1.5开始，JDK提供了AtomicReference类来保证引用对象之间的原子性，就可以把多个变量放在一个对象里来进行CAS操作。

**UnSafe类详解**

UnSafe总体功能

![img](https://www.pdai.tech/images/thread/java-thread-x-atomicinteger-unsafe.png)

**原子类**

**原子更新基本类型**

- AtomicBoolean: 原子更新布尔类型。
- AtomicInteger: 原子更新整型。
- AtomicLong: 原子更新长整型。

**原子更新数组**

- AtomicIntegerArray: 原子更新整型数组里的元素。
- AtomicLongArray: 原子更新长整型数组里的元素。
- AtomicReferenceArray: 原子更新引用类型数组里的元素。

这三个类的最常用的方法是如下两个方法：

- get(int index)：获取索引为index的元素值。
- compareAndSet(int i,E expect,E update): 如果当前值等于预期值，则以原子方式将数组位置i的元素设置为update值。

**原子更新引用类型**

- AtomicReference: 原子更新引用类型。
- AtomicStampedReference: 原子更新引用类型, 内部使用Pair来存储元素值及其版本号。
- AtomicMarkableReferce: 原子更新带有标记位的引用类型。

**原子更新字段类**

- AtomicIntegerFieldUpdater: 原子更新整型的字段的更新器。
- AtomicLongFieldUpdater: 原子更新长整型字段的更新器。
- AtomicReferenceFieldUpdater: 上面已经说过此处不在赘述。

### JUC锁:LockSupport详解

**问题：**

**为什么LockSupport也是核心基础类?** 

​	AQS框架借助于两个类：Unsafe(提供CAS操作)和LockSupport(提供park/unpark操作)

**写出分别通过wait/notify和LockSupport的park/unpark实现同步?**

**wait/notify**

```
class MyThread extends Thread {
    
    public void run() {
        synchronized (this) {
            System.out.println("before notify");            
            notify();
            System.out.println("after notify");    
        }
    }
}

public class WaitAndNotifyDemo {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();            
        synchronized (myThread) {
            try {        
                myThread.start();
                // 主线程睡眠3s
                Thread.sleep(3000);
                System.out.println("before wait");
                // 阻塞主线程
                myThread.wait();
                System.out.println("after wait");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }            
        }        
    }
}
//运行结果
before wait
before notify
after notify
after wait
```

**park/unpark**

```
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before unpark");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 获取blocker
        System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));
        // 释放许可
        LockSupport.unpark((Thread) object);
        // 休眠500ms，保证先执行park中的setBlocker(t, null);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 再次获取blocker
        System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));

        System.out.println("after unpark");
    }
}

public class test {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
//运行结果
before park
before unpark
Blocker info ParkAndUnparkDemo
after park
Blocker info null
after unpark
```

**LockSupport.park()会释放锁资源吗? 那么Condition.await()呢?**

不会，它只负责阻塞当前线程，释放锁资源实际上是在Condition的await()方法中实现的。

**Thread.sleep()、Object.wait()、Condition.await()、LockSupport.park()的区别? 重点**

​	Thread.sleep()不会释放锁资源，Object.wait()会释放锁资源。

​	Object.wait()和Condition.await()的原理是基本一致的，不同的是Condition.await()底层是调用LockSupport.park()来实现阻塞当前线程的。Condition.await()会释放锁资源

​	Thread.sleep()和LockSupport.park()方法区别

- ​		Thread.sleep()和LockSupport.park()方法类似，都是阻塞当前线程的执行，且**都不会释放当前线程占有的锁资源**；
- ​		Thread.sleep()没法从外部唤醒，只能自己醒过来；
- ​		LockSupport.park()方法可以被另一个线程调用LockSupport.unpark()方法唤醒；
- ​		Thread.sleep()方法声明上抛出了InterruptedException中断异常，所以调用者需要捕获这个异常或者再抛出；
- ​		LockSupport.park()方法不需要捕获中断异常；
- ​		Thread.sleep()本身就是一个native方法；

​	Object.wait()和LockSupport.park()的区别

- ​		Object.wait()方法需要在synchronized块中执行；
- ​		LockSupport.park()可以在任意地方执行；
- ​		Object.wait()方法声明抛出了中断异常，调用者需要捕获或者再抛出；
- ​		LockSupport.park()不需要捕获中断异常；
- ​		Object.wait()不带超时的，需要另一个线程执行notify()来唤醒，但不一定继续执行后续内容；
- ​		LockSupport.park()不带超时的，需要另一个线程执行unpark()来唤醒，一定会继续执行后续内容；
- ​		LockSupport.park()底层是调用的Unsafe的native方法；

park()/unpark()底层的原理是“二元信号量”，你可以把它相像成只有一个许可证的Semaphore，只不过这个信号量在重复执行unpark()的时候也不会再增加许可证，**最多只有一个许可证。**

**park和unpark 与 wait和notify/notifyAll 的比较？**

​	1、wait和notify/notifyAll方法只能在同步代码块里用，而park和uppark 不需要。

​	2、unpark函数可以先于park调用，所以不需要担心线程间的执行的先后顺序。

**如果在wait()之前执行了notify()会怎样?**

A线程调用object的wait方法后挂起，B线程调用notify/notifyAll方法后，A线程才能继续运行;反过来：如果B线程调用notify/notifyAll方法后，A线程才调用object的wait方法，那么A线程一值处于挂起状态。

**如果在park()之前执行了unpark()会怎样?**

A线程调用LockSupport.park方法后挂起自己，B线程调用LockSupport.unpark(A)方法后，A线程能继续运行;反过来：如果B线程调用LockSupport.unpark(A)方法后，A线程调用LockSupport.park方法，那么A线程依然能继续运行。

#### LockSupport简介

LockSupport用来创建锁和其他同步类的基本线程阻塞原语。

简而言之，当调用LockSupport.park时，表示当前线程将会等待，直至获得许可，

当调用LockSupport.unpark时，必须把等待获得许可的线程作为参数进行传递，好让此线程继续运行。

为什么LockSupport也是核心基础类? 

- **AQS框架借助于两个类：Unsafe(提供CAS操作)和LockSupport(提供park/unpark操作)**

**核心函数**

park函数，阻塞线程，并且该线程在下列情况发生之前都会被阻塞: ① 调用unpark函数，释放该线程的许可。② 该线程被中断。③ 设置的时间到了。并且，当time为绝对时间时，isAbsolute为true，否则，isAbsolute为false。当time为0时，表示无限等待，直到unpark发生。

![img](https://img-blog.csdnimg.cn/20190330144907789.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hvbmcxMDA4Ng==,size_16,color_FFFFFF,t_70)

```
public static void park(Object blocker) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 设置Blocker
    setBlocker(t, blocker);
    // 获取许可
    UNSAFE.park(false, 0L);
    // 重新可运行后再此设置Blocker
    setBlocker(t, null);
}
//为什么要在此park函数中要调用两次setBlocker函数呢? 
//原因其实很简单，调用park函数时，当前线程首先设置好parkBlocker字段，然后再调用Unsafe的park函数，此后，当前线程就已经阻塞了，等待该线程的unpark函数被调用，所以后面的一个setBlocker函数无法运行，unpark函数被调用，该线程获得许可后，就可以继续运行了，也就运行第二个setBlocker，把该线程的parkBlocker字段设置为null，这样就完成了整个park函数的逻辑。如果没有第二个setBlocker，那么之后没有调用park(Object blocker)，而直接调用getBlocker函数，得到的还是前一个park(Object blocker)设置的blocker，显然是不符合逻辑的。总之，必须要保证在park(Object blocker)整个函数执行完后，该线程的parkBlocker字段又恢复为null。所以，park(Object)型函数里必须要调用setBlocker函数两次。

```

unpark函数，释放线程的许可，即激活调用park后阻塞的线程。这个函数不是安全的，调用这个函数时要确保线程依旧存活。

在主线程调用park阻塞后，在myThread线程中发出了中断信号，此时主线程会继续运行，也就是说明此时**interrupt起到的作用与unpark一样。**



### JUC锁: 锁核心类AQS详解

**问题**

- **什么是AQS? 为什么它是核心?**

  AQS是一个用来**构建锁和同步器的框架**，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的ReentrantLock，Semaphore，其他的诸如ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。

- **AQS的核心思想是什么? 它是怎么实现的? 底层数据结构等**

  AQS核心思想是，如果被请求的共享资源**空闲**，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为**锁定状态**。

  如果被请求的共享资源**被占用**，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

  ```
  CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列(虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系)。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点(Node)来实现锁的分配。
  ```

  AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。

  ```
  private volatile int state;//共享变量，使用volatile修饰保证线程可见性
  ```

  AQS数据结构

  AbstractQueuedSynchronizer类底层的数据结构是使用`CLH(Craig,Landin,and Hagersten)队列`是一个虚拟的双向队列(虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系)。

  AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点(Node)来实现锁的分配。其中Sync queue，即同步队列，是双向链表，包括head结点和tail结点，head结点主要用作后续的调度。

  而Condition queue不是必须的，其是一个单向链表，只有当使用Condition时，才会存在此单向链表。并且可能会有多个Condition queue。

  ![image](https://www.pdai.tech/images/thread/java-thread-x-juc-aqs-1.png)



- **AQS有哪些核心的方法?**

  ```
  public final void acquire(int arg) {
      if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
          selfInterrupt();
  }
  ```

  ```
  public final boolean release(int arg) {
      if (tryRelease(arg)) { // 释放成功
          // 保存头节点
          Node h = head; 
          if (h != null && h.waitStatus != 0) // 头节点不为空并且头节点状态不为0
              unparkSuccessor(h); //释放头节点的后继结点
          return true;
      }
      return false;
  }
  ```

  AQS使用了模板方法模式，自定义同步器时需要重写下面几个AQS提供的模板方法：

  ```
  isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
  tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
  tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
  tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
  tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
  ```

- **AQS定义什么样的资源获取方式?**

  AQS定义了两种资源获取方式：

  `独占`(只有一个线程能访问执行，又根据是否按队列的顺序分为`公平锁`和`非公平锁`，如`ReentrantLock`) 

  和`共享`(多个线程可同时访问执行，如`Semaphore`、`CountDownLatch`、 `CyclicBarrier` )。`ReentrantReadWriteLock`可以看成是组合式，允许多个线程同时对某一资源进行读。

- **AQS底层使用了什么样的设计模式? 模板**

- **AQS的应用示例?**

它提供了一个基于FIFO队列，可以用于构建锁或者其他相关同步装置的基础框架。

AQS定义两种资源共享方式

- Exclusive(独占)：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁： 
  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
- Share(共享)：多个线程可同时执行，如Semaphore/CountDownLatch。Semaphore、CountDownLatCh、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。

对于AbstractQueuedSynchronizer的分析，最核心的就是sync queue的分析。

- 每一个结点都是由前一个结点唤醒
- 当结点发现前驱结点是head并且尝试获取成功，则会轮到该线程运行。
- condition queue中的结点向sync queue中转移是通过signal操作完成的。
- 当结点的状态为SIGNAL时，表示后面的结点需要运行。



### JUC锁: ReentrantLock详解

**问题：**

**ReentrantLock的核心是AQS，那么它怎么来实现的，继承吗? 说说其类内部结构关系。**

![image](https://www.pdai.tech/images/thread/java-thread-x-juc-reentrantlock-1.png)

​	ReentrantLock类内部总共存在Sync、NonfairSync、FairSync三个类，NonfairSync与FairSync类继承自Sync类，Sync类继承自AbstractQueuedSynchronizer抽象类。

**ReentrantLock是如何实现公平锁的?**

```
// 公平锁
static final class FairSync extends Sync {
    // 版本序列化
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        // 以独占模式获取对象，忽略中断
        acquire(1);
    }

    /**
        * Fair version of tryAcquire.  Don't grant access unless
        * recursive call or no waiters or is first.
        */
    // 尝试公平获取锁
    protected final boolean tryAcquire(int acquires) {
        // 获取当前线程
        final Thread current = Thread.currentThread();
        // 获取状态
        int c = getState();
        if (c == 0) { // 状态为0
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) { // 不存在已经等待更久的线程并且比较并且设置状态成功
                // 设置当前线程独占
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) { // 状态不为0，即资源已经被线程占据
            // 下一个状态
            int nextc = c + acquires;
            if (nextc < 0) // 超过了int的表示范围
                throw new Error("Maximum lock count exceeded");
            // 设置状态
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

**ReentrantLock是如何实现非公平锁的?**

```
// 非公平锁
static final class NonfairSync extends Sync {
    // 版本号
    private static final long serialVersionUID = 7316153563782823691L;

    // 获得锁
    final void lock() {
        if (compareAndSetState(0, 1)) // 比较并设置状态成功，状态0表示锁没有被占用
            // 把当前线程设置独占了锁
            setExclusiveOwnerThread(Thread.currentThread());
        else // 锁已经被占用，或者set失败
            // 以独占模式获取对象，忽略中断
            acquire(1); 
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

**ReentrantLock默认实现的是公平还是非公平锁?**

```java
默认非公平锁
```

**使用ReentrantLock实现公平和非公平锁的示例?**

**ReentrantLock和Synchronized的对比?**

![image-20240119113839041](C:\Users\1\Desktop\JAVA全栈知识体系\JAVA 全栈知识体系.assets\image-20240119113839041.png)

Sync类存在如下方法和作用如下。

![image](https://www.pdai.tech/images/thread/java-thread-x-juc-reentrantlock-2.png)

ReentrantLock类的sync非常重要，对ReentrantLock类的操作大部分都直接转化为对Sync和AbstractQueuedSynchronizer类的操作。

### JUC锁: ReentrantReadWriteLock详解

ReentrantReadWriteLock表示可重入读写锁，ReentrantReadWriteLock中包含了两种锁，读锁ReadLock和写锁WriteLock，可以通过这两种锁实现线程间的同步

**问题**

- **为什么有了ReentrantLock还需要ReentrantReadWriteLock?**

  ReentrantReadWriteLock表示可重入读写锁，ReentrantReadWriteLock中包含了两种锁，读锁ReadLock和写锁WriteLock，可以通过这两种锁实现线程间的同步。

- **ReentrantReadWriteLock底层实现原理?**

  ReentrantReadWriteLock底层是基于ReentrantLock和AbstractQueuedSynchronizer来实现的，所以，ReentrantReadWriteLock的数据结构也依托于AQS的数据结构。

  ReentrantReadWriteLock有五个内部类，五个内部类之间也是相互关联的。内部类的关系如下图所示。

  ![img](https://www.pdai.tech/images/thread/java-thread-x-readwritelock-1.png)

  Sync继承自AQS、NonfairSync继承自Sync类、FairSync继承自Sync类；ReadLock实现了Lock接口、WriteLock也实现了Lock接口。

- **ReentrantReadWriteLock底层读写状态如何设计的? 高16位为读锁，低16位为写锁**

- **读锁和写锁的最大数量是多少?**

  1<<16 -1 = 2 的16次方-1

- **本地线程计数器ThreadLocalHoldCounter是用来做什么的?**

  ThreadLocalHoldCounter**重写了ThreadLocal的initialValue方法**，**ThreadLocal类可以将线程与对象相关联**。在没有进行set的情况下，get到的均是initialValue方法里面生成的那个HolderCounter对象。

  ```
  // 本地线程计数器
  static final class ThreadLocalHoldCounter
      extends ThreadLocal<HoldCounter> {
      // 重写初始化方法，在没有进行set的情况下，获取的都是该HoldCounter值
      public HoldCounter initialValue() {
          return new HoldCounter();
      }
  }
  ```

- **缓存计数器HoldCounter是用来做什么的?**

  **HoldCounter主要与读锁配套使用**， HoldCounter主要有两个属性，count和tid，其中count表示某个读线程重入的次数，tid表示该线程的tid字段的值，该字段可以用来唯一标识一个线程。

  ```
  // 计数器
  static final class HoldCounter {
      // 计数
      int count = 0;
      // Use id, not reference, to avoid garbage retention
      // 获取当前线程的TID属性的值
      final long tid = getThreadId(Thread.currentThread());
  }
  ```

- **写锁的获取与释放是怎么实现的?**

  **写锁获取  tryAcquire(acquires)函数**

  此函数用于获取写锁，首先会获取state，判断是否为0，若为0，表示此时没有读锁线程，再判断写线程是否应该被阻塞，而在非公平策略下总是不会被阻塞，在公平策略下会进行判断(判断同步队列中是否有等待时间更长的线程，若存在，则需要被阻塞，否则，无需阻塞)，之后在设置状态state，然后返回true。

  若state不为0，则表示此时存在读锁或写锁线程，若写锁线程数量为0或者当前线程为独占锁线程，则返回false，表示不成功，否则，判断写锁线程的重入次数是否大于了最大值，若是，则抛出异常，否则，设置状态state，返回true，表示成功。

  ```
  protected final boolean tryAcquire(int acquires) {
      // 获取当前线程
      Thread current = Thread.currentThread();
      // 获取状态
      int c = getState();
      // 写线程数量
      int w = exclusiveCount(c);
      if (c != 0) { // 状态不为0
          // (Note: if c != 0 and w == 0 then shared count != 0)
          if (w == 0 || current != getExclusiveOwnerThread()) // 写线程数量为0或者当前线程没有占有独占资源
              return false;
          if (w + exclusiveCount(acquires) > MAX_COUNT) // 判断是否超过最高写线程数量
              throw new Error("Maximum lock count exceeded");
          // Reentrant acquire
          // 设置AQS状态
          setState(c + acquires);
          return true;
      }
      if (writerShouldBlock() ||
          !compareAndSetState(c, c + acquires)) // 写线程是否应该被阻塞
          return false;
      // 设置独占线程
      setExclusiveOwnerThread(current);
      return true;
  }
  ```

  **写锁释放    tryRelease()函数**

  此函数用于释放写锁资源，首先会判断该线程是否为独占线程，若不为独占线程，则抛出异常，否则，计算释放资源后的写锁的数量，若为0，表示成功释放，资源不将被占用，否则，表示资源还被占用。

  ```
  protected final boolean tryRelease(int releases) {
      // 判断是否伪独占线程
      if (!isHeldExclusively())
          throw new IllegalMonitorStateException();
      // 计算释放资源后的写锁的数量
      int nextc = getState() - releases;
      boolean free = exclusiveCount(nextc) == 0; // 是否释放成功
      if (free)
          setExclusiveOwnerThread(null); // 设置独占线程为空
      setState(nextc); // 设置状态
      return free;
  }
  ```

- **读锁的获取与释放是怎么实现的?**

  **读锁获取锁	tryAcquireShared(int unused)**

  此函数表示读锁线程获取读锁。

  首先判断写锁是否为0并且当前线程不占有独占锁，直接返回；

  否则，判断读线程是否需要被阻塞并且读锁数量是否小于最大值并且比较设置状态成功，若当前没有读锁，则设置第一个读线程firstReader和firstReaderHoldCount；若当前线程线程为第一个读线程，则增加firstReaderHoldCount；否则，将设置当前线程对应的HoldCounter对象的值。

  ```
  // 共享模式下获取资源
  protected final int tryAcquireShared(int unused) {
      // 获取当前线程
      Thread current = Thread.currentThread();
      // 获取状态
      int c = getState();
      if (exclusiveCount(c) != 0 &&
          getExclusiveOwnerThread() != current) // 写线程数不为0并且占有资源的不是当前线程
          return -1;
      // 读锁数量
      int r = sharedCount(c);
      if (!readerShouldBlock() &&
          r < MAX_COUNT &&
          compareAndSetState(c, c + SHARED_UNIT)) { // 读线程是否应该被阻塞、并且小于最大值、并且比较设置成功
          if (r == 0) { // 读锁数量为0
              // 设置第一个读线程
              firstReader = current;
              // 读线程占用的资源数为1
              firstReaderHoldCount = 1;
          } else if (firstReader == current) { // 当前线程为第一个读线程
              // 占用资源数加1
              firstReaderHoldCount++;
          } else { // 读锁数量不为0并且不为当前线程
              // 获取计数器
              HoldCounter rh = cachedHoldCounter;
              if (rh == null || rh.tid != getThreadId(current)) // 计数器为空或者计数器的tid不为当前正在运行的线程的tid
                  // 获取当前线程对应的计数器
                  cachedHoldCounter = rh = readHolds.get();
              else if (rh.count == 0) // 计数为0
                  // 设置
                  readHolds.set(rh);
              rh.count++;
          }
          return 1;
      }
      return fullTryAcquireShared(current);
  }
  ```

  **读锁释放	tryReleaseShared(int unused)** 

  此函数表示读锁线程释放锁。

  首先判断当前线程是否为第一个读线程firstReader，若是，则判断第一个读线程占有的资源数firstReaderHoldCount是否为1，若是，则设置第一个读线程firstReader为空，否则，将第一个读线程占有的资源数firstReaderHoldCount减1；

  若当前线程不是第一个读线程，那么首先会获取缓存计数器(上一个读锁线程对应的计数器 )，若计数器为空或者tid不等于当前线程的tid值，则获取当前线程的计数器，如果计数器的计数count小于等于1，则移除当前线程对应的计数器，如果计数器的计数count小于等于0，则抛出异常，之后再减少计数即可。无论何种情况，都会进入无限循环，该循环可以确保成功设置状态state。

  ```
  protected final boolean tryReleaseShared(int unused) {
      // 获取当前线程
      Thread current = Thread.currentThread();
      if (firstReader == current) { // 当前线程为第一个读线程
          // assert firstReaderHoldCount > 0;
          if (firstReaderHoldCount == 1) // 读线程占用的资源数为1
              firstReader = null;
          else // 减少占用的资源
              firstReaderHoldCount--;
      } else { // 当前线程不为第一个读线程
          // 获取缓存的计数器
          HoldCounter rh = cachedHoldCounter;
          if (rh == null || rh.tid != getThreadId(current)) // 计数器为空或者计数器的tid不为当前正在运行的线程的tid
              // 获取当前线程对应的计数器
              rh = readHolds.get();
          // 获取计数
          int count = rh.count;
          if (count <= 1) { // 计数小于等于1
              // 移除
              readHolds.remove();
              if (count <= 0) // 计数小于等于0，抛出异常
                  throw unmatchedUnlockException();
          }
          // 减少计数
          --rh.count;
      }
      for (;;) { // 无限循环
          // 获取状态
          int c = getState();
          // 获取状态
          int nextc = c - SHARED_UNIT;
          if (compareAndSetState(c, nextc)) // 比较并进行设置
              // Releasing the read lock has no effect on readers,
              // but it may allow waiting writers to proceed if
              // both read and write locks are now free.
              return nextc == 0;
      }
  }
  ```

  

- **RentrantReadWriteLock为什么不支持锁升级?**

  RentrantReadWriteLock不支持锁升级(把持读锁、获取写锁，最后释放读锁的过程)。**目的也是保证数据可见性**，如果读锁已被多个线程获取，其中任意线程成功获取了写锁并更新了数据，则其更新对其他获取到读锁的线程是不可见的。

- **什么是锁的升降级? **

  锁降级指的是写锁降级成为读锁。如果当前线程拥有写锁，然后将其释放，最后再获取读锁，这种分段完成的过程不能称之为锁降级。锁降级是指把持住(当前拥有的)写锁，再获取到读锁，随后释放(先前拥有的)写锁的过程。

  **锁降级中读锁的获取是否必要呢?** 

  答案是必要的。主要是为了保证数据的可见性，如果当前线程不获取读锁而是直接释放写锁，假设此刻另一个线程(记作线程T)获取了写锁并修改了数据，那么当前线程无法感知线程T的数据更新。如果当前线程获取读锁，即遵循锁降级的步骤，则线程T将会被阻塞，直到当前线程使用数据并释放读锁之后，线程T才能获取写锁进行数据更新。

### JUC集合

### JUC集合: ConcurrentHashMap详解

- **为什么HashTable慢? 它的并发度是什么? 那么ConcurrentHashMap并发度是什么?**

  Hashtable之所以效率低下主要是因为其实现使用了**synchronized关键字**对put等操作进行加锁，而synchronized关键字加锁是对整个对象进行加锁，也就是说在进行put等修改Hash表的操作时，**锁住了整个Hash表**，从而使得其表现的效率低下。

   Hashtable 最大并发度1

  ConcurrentHashMap 最大并发度16

- **ConcurrentHashMap在JDK1.7和JDK1.8中实现有什么差别? JDK1.8解決了JDK1.7中什么问题**

  锁机制+底层数据结构 数组中的槽位，链表过长的时候查询较慢。

   JDK1.7 Segment数组+链表，每个Segment 都继承ReentrantLock，在高并发场景下，锁竞争强烈，会影响性能

   JDK1.8 Node数组+链表+红黑树 采用CAS+Synchronized机制，减少锁竞争。

- **ConcurrentHashMap JDK1.7实现的原理是什么? 分段锁机制**

- **ConcurrentHashMap JDK1.8实现的原理是什么? 数组+链表+红黑树，CAS**

- **ConcurrentHashMap JDK1.7中Segment数(concurrencyLevel)默认值是多少? 为何一旦初始化就不可再扩容?**

  16 、因为扩容机制是针对Segment数组内部

- **ConcurrentHashMap JDK1.7说说其put的机制?**

  put之前获取Segment的独占锁，之后执行put操作，之后释放锁

- **ConcurrentHashMap JDK1.7是如何扩容的? rehash(注：segment 数组不能扩容，扩容是 segment 数组某个位置内部的数组 HashEntry<K,V>[] 进行扩容)**

- **ConcurrentHashMap JDK1.8是如何扩容的? tryPresize**

- **ConcurrentHashMap JDK1.8链表转红黑树的时机是什么? 临界值为什么是8?**

  判断是否达到临界值8，达到之后判断数组长度是否大于64

  ```java
  如果数组长度小于 64 的时候，其实也就是 32 或者 16 或者更小的时候，会进行数组扩容，不会转红黑树
  ```

- **ConcurrentHashMap JDK1.8是如何进行数据迁移的? transfer**



### JUC集合: CopyOnWriteArrayList详解

**问题**

- **请先说说非并发集合中Fail-fast机制?**

- **再为什么说ArrayList查询快而增删慢?**

  因为ArayList底层是数组实现的..根据下标查询不需要比较,查询方式为，首地址+ (元素长度 *下标)，基于这个位置
  读取相应的字节数就可以了，所以非常快;
  增删会带来元素的移动，增加数据会向后移动，删除数据会向前移动，所以影响效率。
  相反，在添加或删除数据的时候，LinkedList只改变节点之间的引用关系，这就是LinkedList在添加和删除数据的时候
  通常比ArrayList要快的原因,

- **对比ArrayList说说CopyOnWriteArrayList的增删改查实现原理? COW基于拷贝**

- **再说下弱一致性的迭代器原理是怎么样的? `COWIterator`**

  COWIterator表示迭代器，其也有一个Object类型的数组作为CopyOnWriteArrayList数组的快照，这种快照风格的迭代器方法在创建迭代器时使用了对当时数组状态的引用。此数组在迭代器的生存期内不会更改，因此不可能发生冲突，并且迭代器保证不会抛出 ConcurrentModificationException。创建迭代器以后，迭代器就不会反映列表的添加、移除或者更改。在迭代器上进行的元素更改操作(remove、set 和 add)不受支持。这些方法将抛出 UnsupportedOperationException。

  ![image-20240123153438821](C:\Users\1\Desktop\JAVA全栈知识体系\JAVA 全栈知识体系.assets\image-20240123153438821.png)

- **CopyOnWriteArrayList为什么并发安全且性能比Vector好?**

  Vector对单独的add，remove等方法都是在方法上加了synchronized; 并且如果一个线程A调用size时，另一个线程B 执行了remove，然后size的值就不是最新的，然后线程A调用remove就会越界(这时就需要再加一个Synchronized)。这样就导致有了双重锁，效率大大降低，何必呢。于是vector废弃了，要用就用CopyOnWriteArrayList 吧。

- **CopyOnWriteArrayList有何缺陷，说说其应用场景?**

  CopyOnWriteArrayList 有几个缺点：

  - 由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致young gc或者full gc
  - 不能用于实时读的场景，像拷贝数组、新增元素都需要时间，所以调用一个set操作后，读取到数据可能还是旧的,虽然CopyOnWriteArrayList 能做到最终一致性,但是还是没法满足实时性要求；

  **CopyOnWriteArrayList 合适读多写少的场景，不过这类慎用**

  因为谁也没法保证CopyOnWriteArrayList 到底要放置多少数据，万一数据稍微有点多，每次add/set都要重新复制数组，这个代价实在太高昂了。在高性能的互联网应用中，这种操作分分钟引起故障。

#### CopyOnWriteArrayList

CopyOnWriteArrayList是**ArrayList 的一个线程安全的变体**，其中所有可变操作(add、set 等等)都是通过对底层数组进行一次新的拷贝来实现的。COW模式的体现。

#### [Java集合的快速失败机制 “fail-fast”和安全失败机制“fail-safe”](https://www.cnblogs.com/sha-Pao-Zi/p/16314916.html)

```
系统运行中，如果有错误发生，那么系统立即结束，这种设计就是快速失败。
系统运行中，如果有错误发生，系统不会停止运行，它忽略错误（但是会有地方记录下来），继续运行，这种设计就是失败安全。
```

##### fast-fail” 快速失败机制

- Java的快速失败机制是**Java集合框架中的一种错误检测机制**，当多个线程同时对集合中的内容进行修改时可能就会抛出`ConcurrentModificationExceptioon`异常.

- 其实不仅仅是在多线程状态下，在单线程中用**增强 for 循环**中一边遍历集合一边修改集合的元素也会抛出`ConcurrentModificationException` 异常. 

- `多线程的情况下即使用了迭代器调用 remove() 方法，还是会报 ConcurrentModificationException 异常`。这又是为什么呢？

  还是要从 **expectedModCount** 和**modCount** 这两个变量入手分析，刚刚说了 **modCount 在 AbstractList 类中定义，而expectedModCount 在 ArrayList 内部类中定义，所以 modCount 是个共享变量而expectedModCount 是属于线程各自的。**

##### fail-safe 安全失败

采用安全失败机制的集合容器，在**`遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历`。**

所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会抛出ConcurrentModificationException 异常。

缺点是**迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生了修改，迭代器是无法访问到修改后的内容**。



### JUC集合: ConcurrentLinkedQueue详解

**问题**

- **要想用线程安全的队列有哪些选择? **

  Vector，`Collections.synchronizedList(List list)`, ConcurrentLinkedQueue等

- **ConcurrentLinkedQueue实现的数据结构?**

  ConcurrentLinkedQueue采用的**链表**结构，并且包含有一个头节点和一个尾结点。

  ConcurerntLinkedQueue一个基于链接节点的无界线程安全队列。

  此队列按照 FIFO(先进先出)原则对元素进行排序。

  队列的头部是队列中时间最长的元素。队列的尾部 是队列中时间最短的元素。新的元素插入到队列的尾部，队列获取操作从队列头部获得元素。

  当多个线程共享访问一个公共 collection 时，ConcurrentLinkedQueue是一个恰当的选择。

  此队列不允许使用null元素。

- **ConcurrentLinkedQueue底层原理? 全程无锁(CAS)**

- **ConcurrentLinkedQueue的核心方法有哪些? offer()，poll()，peek()，isEmpty()等队列常用方法**

  **offer()函数**：

  说明: offer函数用于将指定元素插入此队列的尾部。下面模拟offer函数的操作，队列状态的变化(假设单线程添加元素，连续添加10、20两个元素)。

  ![img](https://www.pdai.tech/images/thread/java-thread-x-juc-concurrentlinkedqueue-2.png)

  - 若ConcurrentLinkedQueue的初始状态如上图所示，即队列为空。单线程添加元素，此时，添加元素10，则状态如下所示

    ![img](https://www.pdai.tech/images/thread/java-thread-x-juc-concurrentlinkedqueue-3.png)

  - 如上图所示，添加元素10后，tail没有变化，还是指向之前的结点，继续添加元素20，则状态如下所示

  ![img](https://www.pdai.tech/images/thread/java-thread-x-juc-concurrentlinkedqueue-4.png)

  - 如上图所示，添加元素20后，tail指向了最新添加的结点。

    

    **poll()函数**

    说明: 此函数用于获取并移除此队列的头，如果此队列为空，则返回null。下面模拟poll函数的操作，队列状态的变化(假设单线程操作，状态为之前offer10、20后的状态，poll两次)。

    ![img](https://www.pdai.tech/images/thread/java-thread-x-juc-concurrentlinkedqueue-5.png)

    - 队列初始状态如上图所示，在poll操作后，队列的状态如下图所示

    ![img](https://www.pdai.tech/images/thread/java-thread-x-juc-concurrentlinkedqueue-6.png)

    - 如上图可知，poll操作后，head改变了，并且head所指向的结点的item变为了null。再进行一次poll操作，队列的状态如下图所示。

    ![img](https://www.pdai.tech/images/thread/java-thread-x-juc-concurrentlinkedqueue-7.png)

    - 如上图可知，poll操作后，head结点没有变化，只是指示的结点的item域变成了null。

      

    **remove()函数**

    说明: succ用于获取结点的下一个结点。如果结点的next域指向自身，则返回head头节点，否则，返回next结点。下面模拟remove函数的操作，队列状态的变化(假设单线程操作，状态为之前offer10、20后的状态，执行remove(10)、remove(20)操作)。

    ![img](https://www.pdai.tech/images/thread/java-thread-x-juc-concurrentlinkedqueue-8.png)

    - 如上图所示，为ConcurrentLinkedQueue的初始状态，remove(10)后的状态如下图所示

    ![img](https://www.pdai.tech/images/thread/java-thread-x-juc-concurrentlinkedqueue-9.png)

    - 如上图所示，当执行remove(10)后，head指向了head结点之前指向的结点的下一个结点，并且head结点的item域置为null。继续执行remove(20)，状态如下图所示

    ![img](https://www.pdai.tech/images/thread/java-thread-x-juc-concurrentlinkedqueue-10.png)

    - 如上图所示，执行remove(20)后，head与tail指向同一个结点，item域为null。

    

    

- **说说ConcurrentLinkedQueue的HOPS(延迟更新的策略)的设计?**

  通过上面对offer和poll方法的分析，我们发现tail和head是延迟更新的，两者更新触发时机为：

  - `tail更新触发时机`：当tail指向的节点的下一个节点不为null的时候，会执行定位队列真正的队尾节点的操作，找到队尾节点后完成插入之后才会通过casTail进行tail更新；当tail指向的节点的下一个节点为null的时候，只插入节点不更新tail。

  - `head更新触发时机`：当head指向的节点的item域为null的时候，会执行定位队列真正的队头节点的操作，找到队头节点后完成删除之后才会通过updateHead进行head更新；当head指向的节点的item域不为null的时候，只删除节点不更新head。

    **可以减少CAS操作的次数，提高并发性能。**

- **ConcurrentLinkedQueue适合什么样的使用场景?**

  ConcurrentLinkedQueue通过无锁来做到了更高的并发量，是个高性能的队列，但是使用场景相对不如阻塞队列常见，毕竟取数据也要不停的去循环，不如阻塞的逻辑好设计，但是在并发量特别大的情况下，是个不错的选择，性能上好很多，而且这个队列的设计也是特别费力，尤其的使用的改良算法和对哨兵的处理。整体的思路都是比较严谨的，这个也是使用了无锁造成的，我们自己使用无锁的条件的话，这个队列是个不错的参考。

  

### JUC集合: BlockingQueue详解

**问题**

- **什么是BlockingDeque?**

  JUC里的 BlockingQueue 接口表示一个线程安放入和提取实例的队列。

- **BlockingQueue大家族有哪些? **

  数组阻塞队列 ArrayBlockingQueue、延迟队列 DelayQueue、同步队列 SynchronousQueue、链阻塞队列LinkedBlockingQueue

- **BlockingQueue适合用在什么样的场景?**

  BlockingQueue 通常用于一个线程生产对象，而另外一个线程消费这些对象的场景。

  ![img](https://www.pdai.tech/images/thread/java-thread-x-blocking-queue-1.png)

  **一个线程将会持续生产新对象并将其插入到队列之中，直到队列达到它所能容纳的临界点。**

  也就是说，它是有限的。如果该阻塞队列到达了其临界点，负责生产的线程将会在往里边插入新对象时发生阻塞。

  它会一直处于阻塞之中，直到负责消费的线程从队列中拿走一个对象。 

  负责消费的线程将会一直从该阻塞队列中拿出对象。如果消费线程尝试去从一个空的队列中提取对象的话，这个消费线程将会处于**阻塞**之中，直到一个生产线程把一个对象丢进队列。

- **BlockingQueue常用的方法?**

  |      | 抛异常    | 特定值   | 阻塞   | 超时                        |
  | ---- | --------- | -------- | ------ | --------------------------- |
  | 插入 | add(o)    | offer(o) | put(o) | offer(o, timeout, timeunit) |
  | 移除 | remove()  | poll()   | take() | poll(timeout, timeunit)     |
  | 检查 | element() | peek()   |        |                             |

- **BlockingQueue插入方法有哪些? 这些方法(`add(o)`,`offer(o)`,`put(o)`,`offer(o, timeout, timeunit)`)的区别是什么?**

  抛异常: 如果试图的操作无法立即执行，抛一个异常。

  特定值: 如果试图的操作无法立即执行，返回一个特定的值(常常是 true / false)。

  阻塞: 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。

  超时: 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)。

- **BlockingDeque 与BlockingQueue有何关系，请对比下它们的方法?**

  **BlockingDeque 接口继承自 BlockingQueue 接口。**这就意味着你可以像使用一个 BlockingQueue 那样使用 BlockingDeque。如果你这么干的话，各种插入方法将会把新元素添加到双端队列的尾端，而移除方法将会把双端队列的首端的元素移除。正如 BlockingQueue 接口的插入和移除方法一样。

  以下是 BlockingDeque 对 BlockingQueue 接口的方法的具体内部实现:

  | BlockingQueue | BlockingDeque   |
  | ------------- | --------------- |
  | add()         | addLast()       |
  | offer() x 2   | offerLast() x 2 |
  | put()         | putLast()       |
  | remove()      | removeFirst()   |
  | poll() x 2    | pollFirst()     |
  | take()        | takeFirst()     |
  | element()     | getFirst()      |
  | peek()        | peekFirst()     |

- **BlockingDeque适合用在什么样的场景?**

  BlockingDeque 类是一个双端队列，在不能够插入元素时，它将阻塞住试图插入元素的线程；在不能够抽取元素时，它将阻塞住试图抽取的线程。 deque(双端队列) 是 "Double Ended Queue" 的缩写。因此，双端队列是一个你可以从任意一端插入或者抽取元素的队列。

  在线程既是一个队列的生产者又是这个队列的消费者的时候可以使用到 BlockingDeque。如果生产者线程需要在队列的两端都可以插入数据，消费者线程需要在队列的两端都可以移除数据，这个时候也可以使用 BlockingDeque。BlockingDeque 图解:

  ![img](https://www.pdai.tech/images/thread/java-thread-x-blocking-deque-1.png)

- **BlockingDeque大家族有哪些?**

  链阻塞双端队列 LinkedBlockingDeque

- **BlockingDeque 与BlockingQueue实现例子?**

  ```
  BlockingQueue queue = new ArrayBlockingQueue(1024);
  queue.put("1");
  Object object = queue.take();
  ```

  ```
  BlockingDeque<String> deque = new LinkedBlockingDeque<String>();
  deque.addFirst("1");
  deque.addLast("2");
   
  String two = deque.takeLast();
  String one = deque.takeFirst();
  ```

### JUC线程池: FutureTask详解

**问题**

- **FutureTask用来解决什么问题的? 为什么会出现?**

  FutureTask 为 Future 提供了基础实现，如**获取任务执行结果(get)和取消任务(cancel)等**。

  FutureTask 常用来**封装 Callable 和 Runnable**，也可以**作为一个任务提交到线程池**中执行。除了作为一个独立的类之外，此类也提供了一些功能性函数供我们**创建自定义 task** 类使用。

- **FutureTask类结构关系怎么样的?**

  ![img](https://www.pdai.tech/images/thread/java-thread-x-juc-futuretask-1.png)

  可以看到,FutureTask实现了RunnableFuture接口，则RunnableFuture接口继承了Runnable接口和Future接口，所以FutureTask既能当做一个Runnable直接被Thread执行，也能作为Future用来得到Callable的计算结果。

- **FutureTask的线程安全是由什么保证的?**

  FutureTask 的线程安全由CAS来保证。

  状态的可见性：FutureTask 中的状态是使用 volatile 修饰的，保证状态变化的可见性。多个线程可以看到 FutureTask 中状态的最新值。

  原子性操作：FutureTask 内部使用了 CAS（Compare And Swap）操作来确保同步状态的正确性。在多个线程同时对 FutureTask 中的状态进行修改时，CAS 操作可以确保只有一个线程能够成功修改状态。

  同步机制：FutureTask 通过 AQS（AbstractQueuedSynchronizer）类实现了同步机制。AQS 本质上是一个基于锁和等待队列实现的同步框架，它可以支持多线程之间的竞争。

  线程池：FutureTask 内部使用了线程池来执行任务，线程池会根据实际情况动态调整线程池的大小和资源分配，保证多个任务之间的执行不会相互影响。

- **FutureTask结果返回机制?**

  如果任务尚未完成，获取任务执行结果时将会阻塞。一旦执行结束，任务就不能被重启或取消(除非使用runAndReset执行计算)。

- **FutureTask内部运行状态的转变?**

  其中需要注意的是state是volatile类型的，也就是说只要有任何一个线程修改了这个变量，那么其他所有的线程都会知道最新的值。7种状态具体表示：

  - `NEW`:表示是个新的任务或者还没被执行完的任务。这是初始状态。

  - `COMPLETING`:任务已经执行完成或者执行任务的时候发生异常，但是任务执行结果或者异常原因还没有保存到outcome字段(outcome字段用来保存任务执行结果，如果发生异常，则用来保存异常原因)的时候，状态会从NEW变更到COMPLETING。但是这个状态会时间会比较短，属于中间状态。

  - `NORMAL`:任务已经执行完成并且任务执行结果已经保存到outcome字段，状态会从COMPLETING转换到NORMAL。这是一个最终态。

  - `EXCEPTIONAL`:任务执行发生异常并且异常原因已经保存到outcome字段中后，状态会从COMPLETING转换到EXCEPTIONAL。这是一个最终态。

  - `CANCELLED`:任务还没开始执行或者已经开始执行但是还没有执行完成的时候，用户调用了cancel(false)方法取消任务且不中断任务执行线程，这个时候状态会从NEW转化为CANCELLED状态。这是一个最终态。

  - `INTERRUPTING`: 任务还没开始执行或者已经执行但是还没有执行完成的时候，用户调用了cancel(true)方法取消任务并且要中断任务执行线程但是还没有中断任务执行线程之前，状态会从NEW转化为INTERRUPTING。这是一个中间状态。

  - `INTERRUPTED`:调用interrupt()中断任务执行线程之后状态会从INTERRUPTING转换到INTERRUPTED。这是一个最终态。 

    有一点需要注意的是，**所有值大于COMPLETING的状态都表示任务已经执行完成(任务正常执行完成，任务执行异常或者任务被取消)。**

  各个状态之间的可能转换关系如下图所示:

  ![img](https://www.pdai.tech/images/thread/java-thread-x-juc-futuretask-2.png)

- **FutureTask通常会怎么用? 举例说明。**

  **常用使用方式：**

  - 第一种方式: Future + ExecutorService

  - 第二种方式: FutureTask + ExecutorService

  - 第三种方式: FutureTask + Thread

    ```
    public class CallDemo {
     
        public static void main(String[] args) throws ExecutionException, InterruptedException {
     
            /**
             * 第一种方式:Future + ExecutorService
             * Task task = new Task();
             * ExecutorService service = Executors.newCachedThreadPool();
             * Future<Integer> future = service.submit(task1);
             * service.shutdown();
             */
     
     
            /**
             * 第二种方式: FutureTask + ExecutorService
             * ExecutorService executor = Executors.newCachedThreadPool();
             * Task task = new Task();
             * FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
             * executor.submit(futureTask);
             * executor.shutdown();
             */
     
            /**
             * 第三种方式:FutureTask + Thread
             */
     
            // 2. 新建FutureTask,需要一个实现了Callable接口的类的实例作为构造函数参数
            FutureTask<Integer> futureTask = new FutureTask<Integer>(new Task());
            // 3. 新建Thread对象并启动
            Thread thread = new Thread(futureTask);
            thread.setName("Task thread");
            thread.start();
     
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
     
            System.out.println("Thread [" + Thread.currentThread().getName() + "] is running");
     
            // 4. 调用isDone()判断任务是否结束
            if(!futureTask.isDone()) {
                System.out.println("Task is not done");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            int result = 0;
            try {
                // 5. 调用get()方法获取任务结果,如果任务没有执行完成则阻塞等待
                result = futureTask.get();
            } catch (Exception e) {
                e.printStackTrace();
            }
     
            System.out.println("result is " + result);
     
        }
     
        // 1. 继承Callable接口,实现call()方法,泛型参数为要返回的类型
        static class Task  implements Callable<Integer> {
     
            @Override
            public Integer call() throws Exception {
                System.out.println("Thread [" + Thread.currentThread().getName() + "] is running");
                int result = 0;
                for(int i = 0; i < 100;++i) {
                    result += i;
                }
     
                Thread.sleep(3000);
                return result;
            }
        }
    }
    ```

Future 表示了一个任务的生命周期，是一个可取消的异步运算，

可以把它看作是一个异步操作的结果的占位符，它将在未来的某个时刻完成，并提供对其结果的访问。在并发包中许多异步任务类都继承自Future，其中最典型的就是 FutureTask。

### JUC线程池: ThreadPoolExecutor详解

**问题**

- **为什么要有线程池?**

  线程池能够对线程进行统一分配，调优和监控:

  - 降低资源消耗(线程无限制地创建，然后使用完毕后销毁)
  - 提高响应速度(无须创建线程)
  - 提高线程的可管理性

- **Java是实现和管理线程池有哪些方式? 请简单举例如何使用。**

- **java 线程池实现原理？**

  一个线程集合workerSet和一个阻塞队列workQueue。当用户向线程池提交一个任务(也就是线程)时，线程池会先将任务放入workQueue中。workerSet中的线程会不断的从workQueue中获取线程然后执行。当workQueue中没有任务的时候，worker就会阻塞，直到队列中有任务了就取出来继续执行。

  ![img](https://www.pdai.tech/images/thread/java-thread-x-executors-1.png)

- **Execute原理**？

  当一个任务提交至线程池之后:

  1. 线程池首先当前运行的线程数量是否少于corePoolSize。如果是，则创建一个新的工作线程来执行任务。如果都在执行任务，则进入2.
  2. 判断BlockingQueue是否已经满了，倘若还没有满，则将线程放入BlockingQueue。否则进入3.
  3. 如果创建一个新的工作线程将使当前运行的线程数量超过maximumPoolSize，则交给RejectedExecutionHandler来处理任务。

  当ThreadPoolExecutor创建新线程时，通过CAS来更新线程池的状态ctl.

- **为什么很多公司不允许使用Executors去创建线程池? 那么推荐怎么使用呢?**

  线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明：Executors各个方法的弊端：

  - newFixedThreadPool和newSingleThreadExecutor:   主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至OOM。
  - newCachedThreadPool和newScheduledThreadPool:   主要问题是线程数最大数是Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至OOM。

  推荐使用**ThreadPoolExecutor**类手动配置线程池参数，还可以使用第三方线程池工具类，比如Apache Commons Lang库中的**BasicThreadFactory**类，可以方便地创建线程池并设置线程工厂等参数，避免手动配置的繁琐。

  #### 推荐方式 1

  首先引入：commons-lang3包

  ```java
  ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(1,
          new BasicThreadFactory.Builder().namingPattern("example-schedule-pool-%d").daemon(true).build());
  ```

  #### [#](#推荐方式-2) 推荐方式 2

  首先引入：com.google.guava包

  ```java
  ThreadFactory namedThreadFactory = new ThreadFactoryBuilder().setNameFormat("demo-pool-%d").build();
  
  //Common Thread Pool
  ExecutorService pool = new ThreadPoolExecutor(5, 200, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());
  
  // excute
  pool.execute(()-> System.out.println(Thread.currentThread().getName()));
  
   //gracefully shutdown
  pool.shutdown();
  ```

  #### [#](#推荐方式-3) 推荐方式 3

  spring配置线程池方式：自定义线程工厂bean需要实现ThreadFactory，可参考该接口的其它默认实现类，使用方式直接注入bean调用execute(Runnable task)方法即可

  ```xml
      <bean id="userThreadPool" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
          <property name="corePoolSize" value="10" />
          <property name="maxPoolSize" value="100" />
          <property name="queueCapacity" value="2000" />
  
      <property name="threadFactory" value= threadFactory />
          <property name="rejectedExecutionHandler">
              <ref local="rejectedExecutionHandler" />
          </property>
      </bean>
      
      //in code
      userThreadPool.execute(thread);
  ```

- **ThreadPoolExecutor有哪些核心的配置参数? 请简要说明**

  ```
  public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler)
  ```

  - `corePoolSize` 线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize, **即使有其他空闲线程能够执行新来的任务, 也会继续创建线程**；如果当前线程数为corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果执行了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有核心线程。
  - `workQueue` 用来保存等待被执行的任务的阻塞队列. 在JDK中提供了如下阻塞队列: 具体可以参考[JUC 集合: BlockQueue详解]()
    - `ArrayBlockingQueue`: 基于数组结构的有界阻塞队列，按FIFO排序任务；
    - `LinkedBlockingQueue`: 基于链表结构的阻塞队列，按FIFO排序任务，吞吐量通常要高于ArrayBlockingQueue；
    - `SynchronousQueue`: 一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue；
    - `PriorityBlockingQueue`: 具有优先级的无界阻塞队列；

  - `maximumPoolSize ` 线程池中允许的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize；当阻塞队列是无界队列, 则maximumPoolSize则不起作用, 因为无法提交至核心线程池的线程会一直持续地放入workQueue.
  - `keepAliveTime ` 线程空闲时的存活时间，即当线程没有任务执行时，该线程继续存活的时间；默认情况下，该参数只在线程数大于corePoolSize时才有用, 超过这个时间的空闲线程将被终止；
  - `unit ` keepAliveTime的单位
  - `threadFactory ` 创建线程的工厂，通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名。默认为DefaultThreadFactory
  - `handler ` 线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，当然也可以根据应用场景实现RejectedExecutionHandler接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务。

- **ThreadPoolExecutor可以创建哪是哪三种线程池呢?**

  **newFixedThreadPool**：线程池的线程数量达corePoolSize后，即使线程池没有可执行任务时，也不会释放线程。

  FixedThreadPool的工作队列为无界队列LinkedBlockingQueue(队列容量为Integer.MAX_VALUE), 这会导致以下问题:

  - 线程池里的线程数量不超过corePoolSize,这导致了maximumPoolSize和keepAliveTime将会是个无用参数
  - 由于使用了无界队列, 所以**FixedThreadPool永远不会拒绝, 即饱和策略失效**

  **newSingleThreadExecutor**：初始化的线程池中只有一个线程，如果该线程异常结束，会重新创建一个新的线程继续执行任务，唯一的线程可以保证所提交任务的顺序执行.

  由于使用了无界队列, 所以SingleThreadPool永远不会拒绝, 即饱和策略失效

  **newCachedThreadPool**：线程池的线程数可达到Integer.MAX_VALUE，即2147483647，内部使用SynchronousQueue作为阻塞队列； 和newFixedThreadPool创建的线程池不同，newCachedThreadPool在**没有任务执行时，当线程的空闲时间超过keepAliveTime，会自动释放线程资源**，当提交新任务时，如果**没有空闲线程，则创建新线程执行任务**，会导致一定的系统开销； 

- **当队列满了并且worker的数量达到maxSize的时候，会怎么样?**

  交给RejectedExecutionHandler来处理任务。

- **说说ThreadPoolExecutor有哪些RejectedExecutionHandler策略? 默认是什么策略?**

  线程池提供了4种策略:

  - `AbortPolicy`: 直接抛出异常，**默认策略**；
  - `CallerRunsPolicy`: 用调用者所在的线程来执行任务；
  - `DiscardOldestPolicy`: 丢弃阻塞队列中靠最前的任务，并执行当前任务；
  - `DiscardPolicy`: 直接丢弃任务；

- **简要说下线程池的任务执行机制?**

  ** execute –> addWorker –>runworker (getTask)**

  addWorker主要负责创建新的线程并执行任务

  线程池的工作线程通过Woker类实现，在ReentrantLock锁的保证下，把Woker实例插入到HashSet后，并启动Woker中的线程。 从Woker类的构造方法实现可以发现: 线程工厂在创建线程thread时，将Woker实例本身this作为参数传入，当执行start方法启动线程thread时，**本质是执行了Worker的runWorker方法**。 firstTask执行完成之后，通过getTask方法从阻塞队列中获取等待的任务，如果队列中没有任务，getTask方法会被阻塞并挂起，不会占用cpu资源；

- **线程池内部运行状态？**

  其中AtomicInteger变量ctl的功能非常强大: 利用低29位表示线程池中线程数，通过高3位表示线程池的运行状态:

  - RUNNING: -1 << COUNT_BITS，即高3位为111，该状态的线程池会接收新任务，并处理阻塞队列中的任务；
  - SHUTDOWN: 0 << COUNT_BITS，即高3位为000，该状态的线程池不会接收新任务，但会处理阻塞队列中的任务；
  - STOP : 1 << COUNT_BITS，即高3位为001，该状态的线程不会接收新任务，也不会处理阻塞队列中的任务，而且会中断正在运行的任务；
  - TIDYING : 2 << COUNT_BITS，即高3位为010, 所有的任务都已经终止；
  - TERMINATED: 3 << COUNT_BITS，即高3位为011, terminated()方法已经执行完成

  ![img](https://www.pdai.tech/images/thread/java-thread-x-executors-2.png)

- **线程池中任务是如何提交的?**

  ![img](https://www.pdai.tech/images/thread/java-thread-x-executors-3.png)

  submit任务，等待线程池execute

  执行FutureTask类的get方法时，会把主线程封装成WaitNode节点并保存在waiters链表中， 并阻塞等待运行结果；

  FutureTask任务执行完成后，通过UNSAFE设置waiters相应的waitNode为null，并通过LockSupport类unpark方法唤醒主线程；

- **线程池中任务是如何关闭的?**

  遍历线程池中的所有线程，然后逐个调用线程的interrupt方法来中断线程.

  关闭方式 - shutdown

  将线程池里的线程状态设置成SHUTDOWN状态, 然后中断所有没有正在执行任务的线程.

  关闭方式 - shutdownNow

  将线程池里的线程状态设置成STOP状态, 然后停止所有正在执行或暂停任务的线程. 只要调用这两个关闭方法中的任意一个, isShutDown() 返回true. 当所有任务都成功关闭了, isTerminated()返回true

- **在配置线程池的时候需要考虑哪些配置因素?**

  从任务的优先级，任务的执行时间长短，任务的性质(CPU密集/ IO密集)，任务的依赖关系这四个角度来分析。并且近可能地使用有界的工作队列。

  性质不同的任务可用使用不同规模的线程池分开处理:

  - CPU密集型: 尽可能少的线程，Ncpu+1
  - IO密集型: 尽可能多的线程, Ncpu*2，比如数据库连接池
  - 混合型: CPU密集型的任务与IO密集型任务的执行时间差别较小，拆分为两个线程池；否则没有必要拆分。

- **如何监控线程池的状态?**

  可以使用ThreadPoolExecutor以下方法:

  `getTaskCount（）`返回计划执行的任务的大致总数。

  `getCompletedTaskCount（）`返回已完成执行的任务的大致总数。返回结果少于getTaskCount（）

  `getLargestPoolSize（）`返回池中同时存在的最大线程数。返回结果小于等于最大池大小

  `getPoolSize（）`返回池中当前线程数。

  `getActiveCount（）`返回活动执行任务的线程的大致数量。

**Java是如何实现和管理线程池的?**

从JDK 5开始，把工作单元与执行机制分离开来，工作单元包括Runnable和Callable，而执行机制由Executor框架提供。



### JUC线程池: ScheduledThreadPoolExecutor详解

**问题**

- **ScheduledThreadPoolExecutor要解决什么样的问题?**

  在很多业务场景中，我们可能需要周期性的运行某项任务来获取结果，比如周期数据统计，定时发送数据等。

  ScheduledThreadPoolExecutor是Java中的一个线程池，用于解决需要定时执行任务或定期执行任务的场景。通常在需要周期性地执行某个任务，或在未来的某个时间点执行某个任务时使用。

- **ScheduledThreadPoolExecutor相比ThreadPoolExecutor有哪些特性?**

  ScheduledThreadPoolExecutor继承自 ThreadPoolExecutor，为任务提供延迟或周期执行，属于线程池的一种。和 ThreadPoolExecutor 相比，它还具有以下几种特性:

  - 使用专门的任务类型—**ScheduledFutureTask 来执行周期任务**，也可以接收不需要时间调度的任务(这些任务通过 ExecutorService 来执行)。
  - 使用专门的存储队列—**DelayedWorkQueue 来存储任务**，DelayedWorkQueue 是无界延迟队列DelayQueue 的一种。相比ThreadPoolExecutor也简化了执行机制(delayedExecute方法，后面单独分析)。
  - **支持可选的run-after-shutdown参数**，在池被关闭(shutdown)之后支持可选的逻辑来决定是否继续运行周期或延迟任务。并且当任务(重新)提交操作与 shutdown 操作重叠时，复查逻辑也不相同。

- **ScheduledThreadPoolExecutor有什么样的数据结构，核心内部类和抽象类?**

  ![img](https://www.pdai.tech/images/thread/java-thread-x-stpe-1.png)

  ScheduledThreadPoolExecutor内部维护了一个DelayedWorkQueue延迟队列，其中的元素都是ScheduledFutureTask类型的任务，而ScheduledFutureTask是继承自FutureTask，同时实现了ScheduledFuture接口，可以定时或延迟执行任务。

  核心内部类：

  **ScheduledFutureTask**：继承自FutureTask，实现了ScheduledFuture接口，封装了需要延迟执行的任务。

  **ScheduledThreadPoolExecutor**.**DelayedWorkQueue**：继承自AbstractQueue，实现了Delayed接口，是一个延迟队列，其中存放着需要延迟执行的ScheduledFutureTask。

  抽象类：

  **ScheduledExecutorService**：是ExecutorService接口的子接口，定义了一些调度方法，例如schedule、scheduleAtFixedRate、scheduleWithFixedDelay等方法，用于按一定的时间间隔执行任务。ScheduledThreadPoolExecutor实现了ScheduledExecutorService接口，因此可以使用这些调度方法。

- **ScheduledThreadPoolExecutor有哪两个关闭策略? 区别是什么?**

  1. shutdown()：调用该方法后，线程池进入关闭状态，已提交但未执行的任务会继续执行，直到执行完所有任务后停止。

  1. shutdownNow()：调用该方法后，线程池立即进入关闭状态，正在执行的任务会被中断，已提交但未执行的任务会被取消，方法会返回已提交但未执行的任务列表。

- **ScheduledThreadPoolExecutor中scheduleAtFixedRate 和 scheduleWithFixedDelay区别是什么?**

  调度周期的计算方式不同：

  scheduleAtFixedRate是基于固定的周期来进行调度，无论上一个任务是否执行完成，都会按照固定的周期进行调度。

  scheduleWithFixedDelay是基于上一个任务执行完成的时间，再加上固定的延迟时间，来进行下一次任务的调度。

  调度任务的时间处理方式不同：

  scheduleAtFixedRate是基于系统时间的绝对时间进行调度的，如果任务执行时间过长，则后续任务会被延迟，会导致任务重叠。

  scheduleWithFixedDelay是基于上一个任务执行完成的时间，再加上固定的延迟时间，计算出下一个任务的执行时间，任务的执行时间不受前一个任务的影响。

- **为什么ThreadPoolExecutor 的调整策略却不适用于 ScheduledThreadPoolExecutor?**

  **ThreadPoolExecutor 中的调整策略是通过 allowCoreThreadTimeOut 和 keepAliveTime** 两个参数实现的，可以动态调整线程池中核心线程的数量。

  但是 ScheduledThreadPoolExecutor 中不支持动态调整核心线程数，因为它的调度器需要提前计算好所有任务的执行时间，所以无法随时动态调整核心线程数。

  在 ScheduledThreadPoolExecutor 中**，如果想要调整线程池大小，只能通过调整 corePoolSize 和 maximumPoolSize 这两个参数来实现**。但这样做会有一些限制，比如如果当前线程数已经超过了 corePoolSize，那么只有在所有任务都执行完毕后，才能缩小线程池的大小，否则新的任务可能得不到执行。

- **Executors 提供了几种方法来构造 ScheduledThreadPoolExecutor?**

  - newScheduledThreadPool: 可指定核心线程数的线程池。

  - newSingleThreadScheduledExecutor: 只有一个工作线程的线程池。如果内部工作线程由于执行周期任务异常而被终止，则会新建一个线程替代它的位置

 **schedule方法：**schedule主要用于执行一次性(延迟)任务。

**scheduleAtFixedRate方法：**创建一个周期执行的任务，第一次执行延期时间为initialDelay，之后每隔period执行一次，不等待第一次执行完成就开始计时

s**cheduleWithFixedDelay方法：**创建一个周期执行的任务，第一次执行延期时间为initialDelay，在第一次执行完之后延迟delay后开始下一次执行



### JUC线程池: Fork/Join框架详解

- **Fork/Join主要用来解决什么样的问题?**

  Fork/Join 技术是分治算法(Divide-and-Conquer)的并行实现，它是一项可以获得良好的并行性能的简单且高效的设计技术。目的是为了帮助我们更好地利用多处理器带来的好处，使用所有可用的运算能力来提升应用的性能。

- **Fork/Join框架是在哪个JDK版本中引入的?**

  jdk7

- **Fork/Join框架主要包含哪三个模块? 模块之间的关系是怎么样的?**

  Fork/Join框架主要包含三个模块:

  - 任务对象: `ForkJoinTask` (包括`RecursiveTask`、`RecursiveAction` 和 `CountedCompleter`)
  - 执行Fork/Join任务的线程: `ForkJoinWorkerThread`
  - 线程池: `ForkJoinPool

  这三者的关系是: **ForkJoinPool可以通过池中的ForkJoinWorkerThread来处理ForkJoinTask任务。**

- **ForkJoinPool类继承关系?**

  ![img](https://www.pdai.tech/images/thread/java-thread-x-forkjoin-1.png)

  ForkJoinWorkerThreadFactory: 内部线程工厂接口，用于创建工作线程ForkJoinWorkerThread

  DefaultForkJoinWorkerThreadFactory: ForkJoinWorkerThreadFactory 的默认实现类

  InnocuousForkJoinWorkerThreadFactory: 实现了 ForkJoinWorkerThreadFactory，无许可线程工厂，当系统变量中有系统安全管理相关属性时，默认使用这个工厂创建工作线程。

  EmptyTask: 内部占位类，用于替换队列中 join 的任务。

  ManagedBlocker: 为 ForkJoinPool 中的任务提供扩展管理并行数的接口，一般用在可能会阻塞的任务(如在 Phaser 中用于等待 phase 到下一个generation)。

  WorkQueue: ForkJoinPool 的核心数据结构，本质上是work-stealing 模式的双端任务队列，内部存放 ForkJoinTask 对象任务，使用 @Contented 注解修饰防止伪共享。

  - 工作线程在运行中产生新的任务(通常是因为调用了 fork())时，此时可以把 WorkQueue 的数据结构视为一个栈，新的任务会放入栈顶(top 位)；工作线程在处理自己工作队列的任务时，按照 LIFO 的顺序。
  - 工作线程在处理自己的工作队列同时，会尝试窃取一个任务(可能是来自于刚刚提交到 pool 的任务，或是来自于其他工作线程的队列任务)，此时可以把 WorkQueue 的数据结构视为一个 FIFO 的队列，窃取的任务位于其他线程的工作队列的队首(base位)。

  伪共享状态: 缓存系统中是以缓存行(cache line)为单位存储的。缓存行是2的整数幂个连续字节，一般为32-256个字节。最常见的缓存行大小是64个字节。当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享

- **ForkJoinTask抽象类继承关系? **

  ![img](https://www.pdai.tech/images/thread/java-thread-x-forkjoin-4.png)

  ForkJoinTask 实现了 Future 接口，说明它也是一个**可取消的异步运算**任务，实际上ForkJoinTask 是 Future 的轻量级实现，主要用在纯粹是计算的函数式任务或者操作完全独立的对象计算任务。

  fork 是主运行方法，用于异步执行；而 join 方法在任务结果计算完毕之后才会运行，用来合并或返回计算结果。

   其内部类都比较简单，ExceptionNode 是用于**存储**任务执行期间的**异常信息的单向链表**；

  其余四个类是为 Runnable/Callable 任务提供的**适配器类**，用于把 Runnable/Callable 转化为 ForkJoinTask 类型的任务(因为 ForkJoinPool 只可以运行 ForkJoinTask 类型的任务)。

- **整个Fork/Join 框架的执行流程/运行机制是怎么样的?**

  ![img](https://www.pdai.tech/images/thread/java-thread-x-forkjoin-5.png)

- **具体阐述Fork/Join的分治思想和work-stealing 实现方式?**

  **分治思想**

  分治算法(Divide-and-Conquer)把任务递归的拆分为各个子任务，这样可以更好的利用系统资源，尽可能的使用所有可用的计算能力来提升应用性能。首先看一下 Fork/Join 框架的任务运行机制:

  ![img](https://www.pdai.tech/images/thread/java-thread-x-forkjoin-2.png)

  **核心思想: work-stealing(工作窃取)算法**

  work-stealing(工作窃取)算法: 线程池内的所有工作线程都尝试【找到并执行】**已经提交的任务**，或者是**被其他活动任务创建的子任务(如果不存在就阻塞等待)**。

  这种特性使得 ForkJoinPool 在运行多个可以产生子任务的任务，或者是提交的许多小任务时效率更高。尤其是构建异步模型的 ForkJoinPool 时，对不需要合并(join)的事件类型任务也非常适用。

  在 ForkJoinPool 中，线程池中每个工作线程(ForkJoinWorkerThread)都对应一个任务队列(WorkQueue)，工作线程**优先处理来自自身队列的任务**(LIFO或FIFO顺序，参数 mode 决定)，然后以**FIFO的顺序随机窃取其他队列中的任务。**

  具体思路如下:

  - 每个线程都有自己的一个WorkQueue，该工作队列是一个双端队列。
  - 队列支持三个功能push、pop、poll
  - **push/pop只能被队列的所有者线程调用，而poll可以被其他线程调用**。
  - 划分的子任务调用fork时，都会被push到自己的队列中。
  - 默认情况下，工作线程从自己的双端队列获出任务并执行。
  - **当自己的队列为空时，线程随机从另一个线程的队列末尾调用poll方法窃取任务。**

  ![img](https://www.pdai.tech/images/thread/java-thread-x-forkjoin-3.png)

  **上图可以看出ForkJoinPool 中的任务执行分两种:**

  - 直接通过 FJP 提交的外部任务(external/submissions task)，存放在 workQueues 的偶数槽位；
  - 通过内部 fork 分割的子任务(Worker task)，存放在 workQueues 的奇数槽位。

- **有哪些JDK源码中使用了Fork/Join思想?**

  我们常用的数组工具类 Arrays 在JDK 8之后新增的并行排序方法(parallelSort)就运用了 ForkJoinPool 的特性，还有 ConcurrentHashMap 在JDK 8之后添加的函数式方法(如forEach等)也有运用

- **如何使用Executors工具类创建ForkJoinPool?**

  ```
  // parallelism定义并行级别
  public static ExecutorService newWorkStealingPool(int parallelism);
  // 默认并行级别为JVM可用的处理器个数
  // Runtime.getRuntime().availableProcessors()
  public static ExecutorService newWorkStealingPool();
  ```

- **写一个例子: 用ForkJoin方式实现1+2+3+...+100000?**

  ```
  public class Test {
  	static final class SumTask extends RecursiveTask<Integer> {
  		private static final long serialVersionUID = 1L;
  		
  		final int start; //开始计算的数
  		final int end; //最后计算的数
  		
  		SumTask(int start, int end) {
  			this.start = start;
  			this.end = end;
  		}
  
  		@Override
  		protected Integer compute() {
  			//如果计算量小于1000，那么分配一个线程执行if中的代码块，并返回执行结果
  			if(end - start < 1000) {
  				System.out.println(Thread.currentThread().getName() + " 开始执行: " + start + "-" + end);
  				int sum = 0;
  				for(int i = start; i <= end; i++)
  					sum += i;
  				return sum;
  			}
  			//如果计算量大于1000，那么拆分为两个任务
  			SumTask task1 = new SumTask(start, (start + end) / 2);
  			SumTask task2 = new SumTask((start + end) / 2 + 1, end);
  			//执行任务
  			task1.fork();
  			task2.fork();
  			//获取任务执行的结果
  			return task1.join() + task2.join();
  		}
  	}
  	
  	public static void main(String[] args) throws InterruptedException, ExecutionException {
  		ForkJoinPool pool = new ForkJoinPool();
  		ForkJoinTask<Integer> task = new SumTask(1, 10000);
  		pool.submit(task);
  		System.out.println(task.get());
  	}
  }
  ```

- **Fork/Join在使用时有哪些注意事项? 结合JDK中的斐波那契数列实例具体说明。**

  **避免不必要的fork()**

  划分成两个子任务后，不要同时调用两个子任务的fork()方法。

  表面上看上去两个子任务都fork()，然后join()两次似乎更自然。但事实证明，直接调用compute()效率更高。因为直接调用子任务的compute()方法实际上就是在当前的工作线程进行了计算(线程重用)，这比“将子任务提交到工作队列，线程又从工作队列中拿任务”快得多。

  > 当一个大任务被划分成两个以上的子任务时，尽可能使用前面说到的三个衍生的invokeAll方法，因为使用它们能避免不必要的fork()。

  **[#](#注意fork-、compute-、join-的顺序) 注意fork()、compute()、join()的顺序**

  为了两个任务并行，三个方法的调用顺序需要万分注意。

  ```java
  right.fork(); // 计算右边的任务
  long leftAns = left.compute(); // 计算左边的任务(同时右边任务也在计算)
  long rightAns = right.join(); // 等待右边的结果
  return leftAns + rightAns;
  ```

  如果我们写成:

  ```java
  left.fork(); // 计算完左边的任务
  long leftAns = left.join(); // 等待左边的计算结果
  long rightAns = right.compute(); // 再计算右边的任务
  return leftAns + rightAns;
  ```

  或者

  ```java
  long rightAns = right.compute(); // 计算完右边的任务
  left.fork(); // 再计算左边的任务
  long leftAns = left.join(); // 等待左边的计算结果
  return leftAns + rightAns;
  ```

  这两种实际上都没有并行。

  **[#](#选择合适的子任务粒度) 选择合适的子任务粒度**

  选择划分子任务的粒度(顺序执行的阈值)很重要，因为使用Fork/Join框架并不一定比顺序执行任务的效率高: 如果任务太大，则无法提高并行的吞吐量；如果任务太小，子任务的调度开销可能会大于并行计算的性能提升，我们还要考虑创建子任务、fork()子任务、线程调度以及合并子任务处理结果的耗时以及相应的内存消耗。

  官方文档给出的粗略经验是: 任务应该执行`100~10000`个基本的计算步骤。决定子任务的粒度的最好办法是实践，通过实际测试结果来确定这个阈值才是“上上策”。

  > 和其他Java代码一样，Fork/Join框架测试时需要“预热”或者说执行几遍才会被JIT(Just-in-time)编译器优化，所以测试性能之前跑几遍程序很重要。

  **[#](#避免重量级任务划分与结果合并) 避免重量级任务划分与结果合并**

  Fork/Join的很多使用场景都用到数组或者List等数据结构，子任务在某个分区中运行，最典型的例子如并行排序和并行查找。拆分子任务以及合并处理结果的时候，应该尽量避免System.arraycopy这样耗时耗空间的操作，从而最小化任务的处理开销。

**外部任务(external/submissions task)提交**

**向 ForkJoinPool 提交任务有三种方式:**

- invoke()会等待任务计算完毕并**返回计算结果**；
- execute()是直接向池提交一个任务来异步执行，**无返回结果**；
- submit()也是异步执行，但是会**返回提交的任务**，在适当的时候可通过task.get()获取执行结果。

**子任务(Worker task)提交**

子任务的提交相对比较简单，由任务的fork()方法完成。通过上面的流程图可以看到任务被分割(fork)之后调用了ForkJoinPool.WorkQueue.push()方法直接把任务放到队列中等待被执行。

**获取任务结果 - ForkJoinTask.join() / ForkJoinTask.invoke()**

ForkJoinTask的join()和invoke()方法都可以用来获取任务的执行结果(另外还有get方法也是调用了doJoin来获取任务结果，但是会响应运行时异常)，它们对外部提交任务的执行方式一致，都是通过externalAwaitDone方法等待执行结果。

不同的是**invoke()方法会直接执行当前任务**；

而**join()方法则是在当前任务在队列 top 位时(通过tryUnpush方法判断)才能执行**，如果当前任务不在 top 位或者任务执行失败调用ForkJoinPool.awaitJoin方法帮助执行或阻塞当前 join 任务。

### JUC工具类: CountDownLatch详解

**问题：**

- **什么是CountDownLatch?**

  从源码可知，其底层是由AQS提供支持，所以其数据结构可以参考AQS的数据结构，而AQS的数据结构核心就是两个虚拟队列: 同步队列sync queue 和条件队列condition queue，不同的条件会有不同的条件队列。

  CountDownLatch典型的用法是将一个程序分为n个互相独立的可解决任务，并创建值为n的CountDownLatch。当每一个任务完成时，都会在这个锁存器上调用countDown，等待问题被解决的任务调用这个锁存器的await，将他们自己拦住，直至锁存器计数结束。

- **CountDownLatch底层实现原理?**

- **CountDownLatch一次可以唤醒几个任务? **

  **多个**

- **CountDownLatch有哪些主要方法? await(),countDown()**

  **await函数**      此函数将会使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断

  **countDown函数**    此函数将递减锁存器的计数，如果计数到达零，则释放所有等待的线程

- **CountDownLatch适用于什么场景?**

  1.等待多个线程执行完毕：如果有多个线程需要执行，但是必须等待所有线程都执行完毕才能进行下一步操作，可以使用CountDownLatch来实现。我们可以创建一个CountDownLatch对象，并将计数器的值初始化为线程数，每个线程执行完毕后，调用countDown()方法将计数器减1。最后，在主线程中调用await()方法等待所有线程执行完毕。

  2.控制线程的执行顺序：如果有多个线程需要按照特定的顺序执行，可以使用CountDownLatch来实现。我们可以创建多个CountDownLatch对象，每个对象的计数器的值都为1，表示只有一个线程可以执行。线程执行完毕后，调用下一个CountDownLatch对象的countDown()方法，唤醒下一个线程。

  3.等待外部事件的发生：如果我们需要等待一个外部事件的发生，例如某个网络连接的建立或某个文件的读取完成，可以使用CountDownLatch来实现。我们可以在主线程中创建一个CountDownLatch对象，并将计数器的值初始化为1，然后在另一个线程中等待外部事件的发生。当外部事件发生时，调用CountDownLatch对象的countDown()方法，唤醒主线程继续执行。

  4.控制并发线程数：如果我们需要控制并发线程的数量，可以使用CountDownLatch来实现。我们可以创建一个CountDownLatch对象，并将计数器的值初始化为线程数量，每个线程执行完毕后，调用countDown()方法将计数器减1。如果某个线程需要等待其他线程执行完毕，可以调用await()方法等待计数器的值变为0。

- **写道题：实现一个容器，提供两个方法，add，size 写两个线程，线程1添加10个元素到容器中，线程2实现监控元素的个数，当个数到5个时，线程2给出提示并结束? 使用CountDownLatch 代替wait notify 好处。**

  ```java
  使用CountDownLatch 代替wait notify 好处是通讯方式简单，不涉及锁定  Count 值为0时当前线程继续执行，
  ```

### JUC工具类: CyclicBarrier详解

**问题：**

- **什么是CyclicBarrier?**

  对于CountDownLatch，其他线程为游戏玩家，比如英雄联盟，主线程为控制游戏开始的线程。在所有的玩家都准备好之前，主线程是处于等待状态的，也就是游戏不能开始。当所有的玩家准备好之后，下一步的动作实施者为主线程，即开始游戏。

  对于CyclicBarrier，假设有一家公司要全体员工进行团建活动，活动内容为翻越三个障碍物，每一个人翻越障碍物所用的时间是不一样的。但是公司要求所有人在翻越当前障碍物之后再开始翻越下一个障碍物，也就是所有人翻越第一个障碍物之后，才开始翻越第二个，以此类推。类比地，每一个员工都是一个“其他线程”。当所有人都翻越的所有的障碍物之后，程序才结束。**而主线程可能早就结束了，这里我们不用管主线程。**

- **CyclicBarrier底层实现原理?**

  CyclicBarrier底层是基于ReentrantLock和AbstractQueuedSynchronizer来实现的

- **CountDownLatch和CyclicBarrier对比?**

  CountDownLatch减计数，CyclicBarrier加计数。

  CountDownLatch是一次性的，CyclicBarrier可以重用。

  CountDownLatch和CyclicBarrier都有让多个线程等待同步然后再开始下一步动作的意思，但是CountDownLatch的下一步的动作实施者是主线程，具有不可重复性；而CyclicBarrier的下一步动作实施者还是“其他线程”本身，具有往复多次实施动作的特点。

- **CyclicBarrier的核心函数有哪些?**

  doawait()：

  ![img](https://www.pdai.tech/images/thread/java-thread-x-cyclicbarrier-1.png)

  nextGeneration()：此函数在所有线程进入屏障后会被调用，即生成下一个版本，所有线程又可以重新进入到屏障中

  ```
  private void nextGeneration() {
      // signal completion of last generation
      // 唤醒所有线程
      trip.signalAll();
      // set up next generation
      // 恢复正在等待进入屏障的线程数量
      count = parties;
      // 新生一代
      generation = new Generation();
  }
  ```

- **CyclicBarrier适用于什么场景?**

  非常适合固定大小的线程的程序中使用，这些线程间或彼此等待。

  ### JUC工具类: Semaphore详解

  **问题：**

  - **什么是Semaphore?**

    Semaphore底层是基于AbstractQueuedSynchronizer来实现的。

    Semaphore称为计数信号量，它允许n个任务同时访问某个资源，可以将信号量看做是在向外分发使用资源的许可证，只有成功获取许可证，才能使用资源。

  - **Semaphore内部原理?**

    Semaphore总共有三个内部类，并且三个内部类是紧密相关的，下面先看三个类的关系。

    ![img](https://www.pdai.tech/images/thread/java-thread-x-semaphore-1.png)

    Sync类存在如下方法和作用如下。

    ![img](https://www.pdai.tech/images/thread/java-thread-x-semaphore-2.png)

  - **Semaphore常用方法有哪些? 如何实现线程同步和互斥的?**

    acquire()方法：此方法从信号量获取一个(多个)许可，在提供一个许可前一直将线程阻塞，或者线程被中断。

    release()方法：此方法释放一个(多个)许可，将其返回给信号量

  - **Semaphore适合用在什么场景?**

    主要⽤于那些资源**有明确访问数量限制的场景，常⽤于限流** 。⽐如：[数据库连接池](https://so.csdn.net/so/search?q=数据库连接池&spm=1001.2101.3001.7020)，同时进⾏连接的线程有数量限制，连接不能超过⼀定的数量，当连接达到了限制数量后，后⾯的线程只能排队等前⾯的线程释放了数据库连接才能获得数据库连接。

  - **单独使用Semaphore是不会使用到AQS的条件队列?**

    不同于CyclicBarrier和ReentrantLock，单独使用Semaphore是不会使用到AQS的条件队列的，其实，只有进行**await操作才会进入条件队列**，其他的都是在同步队列中，只是当前线程会被park。

  - **Semaphore中申请令牌(acquire)、释放令牌(release)的实现?**

    acquire()方法：

    ![img](https://www.pdai.tech/images/thread/java-thread-x-semaphore-3.png)

    release()方法：

    ![img](https://www.pdai.tech/images/thread/java-thread-x-semaphore-4.png)

  - **Semaphore初始化有10个令牌，11个线程同时各调用1次acquire方法，会发生什么?**

    拿不到令牌的线程阻塞，不会继续往下运行。

  - **Semaphore初始化有10个令牌，一个线程重复调用11次acquire方法，会发生什么?**

    线程阻塞，不会继续往下运行。可能你会考虑类似于锁的重入的问题，很好，但是，令牌没有重入的概念。你只要调用一次acquire方法，就需要有一个令牌才能继续运行。

  - **Semaphore初始化有1个令牌，1个线程调用一次acquire方法，然后调用两次release方法，之后另外一个线程调用acquire(2)方法，此线程能够获取到足够的令牌并继续运行吗?**

    能，原因是release方法会添加令牌，并不会以初始化的大小为准。

  - **Semaphore初始化有2个令牌，一个线程调用1次release方法，然后一次性获取3个令牌，会获取到吗?**

    能，原因是release会添加令牌，并不会以初始化的大小为准。Semaphore中release方法的调用并没有限制要在acquire后调用。

### JUC工具类: Phaser详解

**问题：**

- **Phaser主要用来解决什么问题?**

  Phaser是**JDK 7**新增的一个同步辅助类，它可以实现CyclicBarrier和CountDownLatch类似的功能，而且它支持**对任务的动态调整，并支持分层结构来达到更高的吞吐量**。

- **Phaser与CyclicBarrier和CountDownLatch的区别是什么?**

- **如果用CountDownLatch来实现Phaser的功能应该怎么实现?**

  

- **Phaser运行机制是什么样的?**

  

  ![img](https://www.pdai.tech/images/thread/java-thread-x-juc-phaser-1.png)

  - **Registration(注册)**

  跟其他barrier不同，在phaser上**注册的parties会随着时间的变化而变化**。任务可以随时注册(使用方法register,bulkRegister注册，或者由构造器确定初始parties)，并且在任何抵达点**可以随意地撤销注册**(方法arriveAndDeregister)。就像大多数基本的同步结构一样，注册和撤销只影响内部count；不会创建更深的内部记录，所以**任务不能查询他们是否已经注册**。(不过，可以通过继承来实现类似的记录)

  - **Synchronization(同步机制)**

  和CyclicBarrier一样，Phaser也可以重复await。方法arriveAndAwaitAdvance的效果类似CyclicBarrier.await。phaser的每一代都有一个相关的phase number，初始值为0，当所有注册的任务都到达phaser时phase+1，到达最大值(Integer.MAX_VALUE)之后清零。使用phase number可以独立控制 到达phaser 和 等待其他线程 的动作，通过下面两种类型的方法:

  > - **Arrival(到达机制)** arrive和arriveAndDeregister方法记录到达状态。这些方法不会阻塞，但是会返回一个相关的arrival phase number；也就是说，phase number用来确定到达状态。当所有任务都到达给定phase时，可以执行一个可选的函数，这个函数通过重写onAdvance方法实现，通常可以用来控制终止状态。重写此方法类似于为CyclicBarrier提供一个barrierAction，但比它更灵活。
  > - **Waiting(等待机制)** awaitAdvance方法需要一个表示arrival phase number的参数，并且在phaser前进到与给定phase不同的phase时返回。和CyclicBarrier不同，即使等待线程已经被中断，awaitAdvance方法也会一直等待。中断状态和超时时间同样可用，但是当任务等待中断或超时后未改变phaser的状态时会遭遇异常。如果有必要，在方法forceTermination之后可以执行这些异常的相关的handler进行恢复操作，Phaser也可能被ForkJoinPool中的任务使用，这样在其他任务阻塞等待一个phase时可以保证足够的并行度来执行任务。

  - **Termination(终止机制)** :

  可以用isTerminated方法检查phaser的终止状态。在终止时，所有同步方法立刻返回一个负值。在终止时尝试注册也没有效果。当调用onAdvance返回true时Termination被触发。当deregistration操作使已注册的parties变为0时，onAdvance的默认实现就会返回true。也可以重写onAdvance方法来定义终止动作。forceTermination方法也可以释放等待线程并且允许它们终止。

  - **Tiering(分层结构)** :

  **Phaser支持分层结构(树状构造)来减少竞争。**注册了大量parties的Phaser可能会因为同步竞争消耗很高的成本， 因此可以设置一些子Phaser来共享一个通用的parent。这样的话即使每个操作消耗了更多的开销，但是会提高整体吞吐量。 在一个分层结构的phaser里，**子节点phaser的注册和取消注册都通过父节点管理**。子节点phaser通过构造或方法register、bulkRegister进行首次注册时，在其父节点上注册。子节点phaser通过调用arriveAndDeregister进行最后一次取消注册时，也在其父节点上取消注册。

  - **Monitoring(状态监控)** :

  由于同步方法可能只被已注册的parties调用，所以**phaser的当前状态也可能被任何调用者监控**。在任何时候，可以通过getRegisteredParties获取parties数，其中getArrivedParties方法返回已经到达当前phase的parties数。当剩余的parties(通过方法getUnarrivedParties获取)到达时，phase进入下一代。这些方法返回的值可能只表示短暂的状态，所以一般来说在同步结构里并没有啥卵用。

- **给一个Phaser使用的示例?**

  ```
  public class PhaserTest1 {
      public static void main(String[] args) {
          Phaser phaser = new Phaser();
          for (int i = 0; i < 10; i++) {
              phaser.register();                  // 注册各个参与者线程
         new Thread(new Task(phaser), "Thread-" + i).start();
          }
      }
  }
  
  class Task implements Runnable {
      private final Phaser phaser;
  
      Task(Phaser phaser) {
          this.phaser = phaser;
      }
  
      @Override
      public void run() {
          int i = phaser.arriveAndAwaitAdvance();     // 等待其它参与者线程到达
       // do something
          System.out.println(Thread.currentThread().getName() + ": 执行完任务，当前phase =" + i + "");
      }
  }
  ```

  Phaser使用一个long型state值来标识内部状态:

  - 低0-15位表示未到达parties数；
  - 中16-31位表示等待的parties数；
  - 中32-62位表示phase当前代；
  - 高63位表示当前phaser的终止状态。

  函数列表：

```
//构造方法
public Phaser() {
    this(null, 0);
}
public Phaser(int parties) {
    this(null, parties);
}
public Phaser(Phaser parent) {
    this(parent, 0);
}
public Phaser(Phaser parent, int parties)
//注册一个新的party
public int register()
//批量注册
public int bulkRegister(int parties)
//使当前线程到达phaser，不等待其他任务到达。返回arrival phase number
public int arrive() 
//使当前线程到达phaser并撤销注册，返回arrival phase number
public int arriveAndDeregister()
/*
 * 使当前线程到达phaser并等待其他任务到达，等价于awaitAdvance(arrive())。
 * 如果需要等待中断或超时，可以使用awaitAdvance方法完成一个类似的构造。
 * 如果需要在到达后取消注册，可以使用awaitAdvance(arriveAndDeregister())。
 */
public int arriveAndAwaitAdvance()
//阻塞等待，直到phase前进到下一代，返回下一代的phase number
public int awaitAdvance(int phase) 
//响应中断版awaitAdvance，调用阻塞时会记录线程的中断状态
public int awaitAdvanceInterruptibly(int phase) throws InterruptedException
public int awaitAdvanceInterruptibly(int phase, long timeout, TimeUnit unit)
    throws InterruptedException, TimeoutException
//使当前phaser进入终止状态，已注册的parties不受影响，如果是分层结构，则终止所有phaser
public void forceTermination()
```

### JUC工具类: Exchanger详解

- **Exchanger主要解决什么问题?**

  Exchanger是用于线程协作的工具类, 主要用于两个线程之间的数据交换。

  它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过exchange()方法交换数据，当一个线程先执行exchange()方法后，它会一直等待第二个线程也执行exchange()方法，当这两个线程到达同步点时，这两个线程就可以交换数据了。

- **对比SynchronousQueue，为什么说Exchanger可被视为 SynchronousQueue 的双向形式?**

  Exchanger是一种线程间安全交换数据的机制。可以和之前分析过的SynchronousQueue对比一下：线程A通过SynchronousQueue将数据a交给线程B；线程A通过Exchanger和线程B交换数据，线程A把数据a交给线程B，同时线程B把数据b交给线程A。可见，**SynchronousQueue是交给一个数据，Exchanger是交换两个数据。**

- **Exchanger在不同的JDK版本中实现有什么差别?**

  在JDK5中Exchanger被设计成一个容量为1的容器，存放一个等待线程，直到有另外线程到来就会发生数据交换，然后清空容器，等到下一个到来的线程。

  从JDK6开始，Exchanger用了类似ConcurrentMap的分段思想，提供了多个slot，增加了并发执行时的吞吐量。

- **Exchanger实现机制?**

  ```
  for (;;) {
      if (slot is empty) { // offer
          // slot为空时，将item 设置到Node 中        
          place item in a Node;
          if (can CAS slot from empty to node) {
              // 当将node通过CAS交换到slot中时，挂起线程等待被唤醒
              wait for release;
              // 被唤醒后返回node中匹配到的item
              return matching item in node;
          }
      } else if (can CAS slot from node to empty) { // release
           // 将slot设置为空
          // 获取node中的item，将需要交换的数据设置到匹配的item
          get the item in node;
          set matching item in node;
          // 唤醒等待的线程
          release waiting thread;
      }
      // else retry on CAS failure
  }
  ```

  比如有2条线程A和B，A线程交换数据时，发现slot为空，则将需要交换的数据放在slot中等待其它线程进来交换数据，等线程B进来，读取A设置的数据，然后设置线程B需要交换的数据，然后唤醒A线程，原理就是这么简单。

- **Exchanger已经有了slot单节点，为什么会加入arena node数组? 什么时候会用到数组?**

  当**多个线程之间**进行交换数据时会存在严重伸缩性问题，所以Exchanger加入了slot数组。

  Exchanger不是一来就会生成arena数组来降低竞争，只有当**产生竞争是才会生成arena数组**

- **arena可以确保不同的slot在arena中是不会相冲突的，那么是怎么保证的呢?**

  ```
  arena = new Node[(FULL + 2) << ASHIFT];
  // 这个arena到底有多大呢? 我们先看FULL 和ASHIFT的定义：
  static final int FULL = (NCPU >= (MMASK << 1)) ? MMASK : NCPU >>> 1;
  private static final int ASHIFT = 7;
  
  private static final int NCPU = Runtime.getRuntime().availableProcessors();
  private static final int MMASK = 0xff;        // 255
  // 假如我的机器NCPU = 8 ，则得到的是768大小的arena数组。然后通过以下代码取得在arena中的节点：
  
  Node q = (Node)U.getObjectVolatile(a, j = (i << ASHIFT) + ABASE);
  // 它仍然是通过右移ASHIFT位来取得Node的，ABASE定义如下：
  
  Class<?> ak = Node[].class;
  ABASE = U.arrayBaseOffset(ak) + (1 << ASHIFT);
  // U.arrayBaseOffset获取对象头长度，数组元素的大小可以通过unsafe.arrayIndexScale(T[].class) 方法获取到。这也就是说要访问类型为T的第N个元素的话，你的偏移量offset应该是arrayOffset+N*arrayScale。也就是说BASE = arrayOffset+ 128 。
  ```

- **什么是伪共享，Exchanger中如何体现的?**

  **伪共享说明**：假设一个类的两个相互独立的属性a和b在内存地址上是连续的(比如FIFO队列的头尾指针)，那么它们通常会被加载到相同的cpu cache line里面。并发情况下，如果一个线程修改了a，会导致整个cache line失效(包括b)，这时另一个线程来读b，就需要从内存里再次加载了，这种多线程频繁修改ab的情况下，虽然a和b看似独立，但它们会互相干扰，非常影响性能。

  我们再看Node节点的定义, 在Java 8 中我们是可以利用**sun.misc.Contended**来规避伪共享的。所以说通过 << ASHIFT方式加上sun.misc.Contended，所以使得**任意两个可用Node不会再同一个缓存行**中。

  ```java
  @sun.misc.Contended static final class Node{
  ....
  }
  ```

- **Exchanger实现举例**

  来一个非常经典的并发问题：你有相同的数据buffer，一个或多个数据生产者，和一个或多个数据消费者。只是Exchange类只能同步2个线程，所以你只能在你的生产者和消费者问题中只有一个生产者和一个消费者时使用这个类。

  ```
  public class Test {
      static class Producer extends Thread {
          private Exchanger<Integer> exchanger;
          private static int data = 0;
          Producer(String name, Exchanger<Integer> exchanger) {
              super("Producer-" + name);
              this.exchanger = exchanger;
          }
  
          @Override
          public void run() {
              for (int i=1; i<5; i++) {
                  try {
                      TimeUnit.SECONDS.sleep(1);
                      data = i;
                      System.out.println(getName()+" 交换前:" + data);
                      data = exchanger.exchange(data);
                      System.out.println(getName()+" 交换后:" + data);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }
      }
  
      static class Consumer extends Thread {
          private Exchanger<Integer> exchanger;
          private static int data = 0;
          Consumer(String name, Exchanger<Integer> exchanger) {
              super("Consumer-" + name);
              this.exchanger = exchanger;
          }
  
          @Override
          public void run() {
              while (true) {
                  data = 0;
                  System.out.println(getName()+" 交换前:" + data);
                  try {
                      TimeUnit.SECONDS.sleep(1);
                      data = exchanger.exchange(data);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println(getName()+" 交换后:" + data);
              }
          }
      }
  
      public static void main(String[] args) throws InterruptedException {
          Exchanger<Integer> exchanger = new Exchanger<Integer>();
          new Producer("", exchanger).start();
          new Consumer("", exchanger).start();
          TimeUnit.SECONDS.sleep(7);
          System.exit(-1);
      }
  }
  可以看到，其结果可能如下：Consumer- 交换前:0
  Producer- 交换前:1
  Consumer- 交换后:1
  Consumer- 交换前:0
  Producer- 交换后:0
  Producer- 交换前:2
  Producer- 交换后:0
  Consumer- 交换后:2
  Consumer- 交换前:0
  Producer- 交换前:3
  Producer- 交换后:0
  Consumer- 交换后:3
  Consumer- 交换前:0
  Producer- 交换前:4
  Producer- 交换后:0
  Consumer- 交换后:4
  Consumer- 交换前:0
  ```

### Java 并发 - ThreadLocal详解

- **什么是ThreadLocal? 用来解决什么问题的?**

  ThreadLocal是一个将在多线程中为每一个线程创建单独的变量副本的类; 当使用ThreadLocal来维护变量时, ThreadLocal会为每个线程创建单独的变量副本, 避免因多线程操作共享变量而导致的数据不一致的情况。

- **说说你对ThreadLocal的理解**

  提到ThreadLocal被提到应用最多的是session管理和数据库链接管理，这里以数据访问为例帮助你理解ThreadLocal：

  因为ThreadLocal在每个线程中对该变量会创建一个副本，即每个线程内部都会有一个该变量

  ```
  import java.sql.Connection;
  import java.sql.DriverManager;
  import java.sql.SQLException;
  
  public class ConnectionManager {
  
      private static final ThreadLocal<Connection> dbConnectionLocal = new ThreadLocal<Connection>() {
          @Override
          protected Connection initialValue() {
              try {
                  return DriverManager.getConnection("", "", "");
              } catch (SQLException e) {
                  e.printStackTrace();
              }
              return null;
          }
      };
  
      public Connection getConnection() {
          return dbConnectionLocal.get();
      }
  }
  ```

- **ThreadLocal是如何实现线程隔离的?**

  **主要是用到了Thread对象中的一个ThreadLocalMap类型的变量threadLocals, 负责存储当前线程的关于Connection的对象**, dbConnectionLocal(以上述例子中为例) 这个变量为Key, 以新建的Connection对象为Value; 这样的话, 线程第一次读取的时候如果不存在就会调用ThreadLocal的initialValue方法创建一个Connection对象并且返回;

- **为什么ThreadLocal会造成内存泄露? 如何解决**

  网上有这样一个例子：

  ```java
  import java.util.concurrent.LinkedBlockingQueue;
  import java.util.concurrent.ThreadPoolExecutor;
  import java.util.concurrent.TimeUnit;
  
  public class ThreadLocalDemo {
      static class LocalVariable {
          private Long[] a = new Long[1024 * 1024];
      }
  
      // (1)
      final static ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(5, 5, 1, TimeUnit.MINUTES,
              new LinkedBlockingQueue<>());
      // (2)
      final static ThreadLocal<LocalVariable> localVariable = new ThreadLocal<LocalVariable>();
  
      public static void main(String[] args) throws InterruptedException {
          // (3)
          Thread.sleep(5000 * 4);
          for (int i = 0; i < 50; ++i) {
              poolExecutor.execute(new Runnable() {
                  public void run() {
                      // (4)
                      localVariable.set(new LocalVariable());
                      // (5)
                      System.out.println("use local varaible" + localVariable.get());
                      localVariable.remove();
                  }
              });
          }
          // (6)
          System.out.println("pool execute over");
      }
  }
  ```

  如果用线程池来操作ThreadLocal 对象确实会造成内存泄露, 因为对于线程池里面不会销毁的线程, 里面总会存在着`<ThreadLocal, LocalVariable>`的强引用, 因为final static 修饰的 ThreadLocal 并不会释放, 而ThreadLocalMap 对于 Key 虽然是弱引用, 但是强引用不会释放, 弱引用当然也会一直有值, 同时创建的LocalVariable对象也不会释放, 就造成了内存泄露; 如果LocalVariable对象不是一个大对象的话, 其实泄露的并不严重, `泄露的内存 = 核心线程数 * LocalVariable`对象的大小;

  所以, 为了避免出现内存泄露的情况, ThreadLocal提供了一个清除线程中对象的方法, 即 remove, 其实内部实现就是调用 ThreadLocalMap 的remove方法:

  找到Key对应的Entry, 并且清除Entry的Key(ThreadLocal)置空, 随后清除过期的Entry即可避免内存泄露。

- **还有哪些使用ThreadLocal的应用场景?**

  **每个线程维护了一个“序列号”**

  > 再回想上文说的，如果我们希望通过某个类将状态(例如用户ID、事务ID)与线程关联起来，那么通常在这个类中定义private static类型的ThreadLocal 实例。

  ```java
  public class SerialNum {
      // The next serial number to be assigned
      private static int nextSerialNum = 0;
  
      private static ThreadLocal serialNum = new ThreadLocal() {
          protected synchronized Object initialValue() {
              return new Integer(nextSerialNum++);
          }
      };
  
      public static int get() {
          return ((Integer) (serialNum.get())).intValue();
      }
  }
  ```

  **Session的管理**

  经典的另外一个例子：

  ```java
  private static final ThreadLocal threadSession = new ThreadLocal();  
    
  public static Session getSession() throws InfrastructureException {  
      Session s = (Session) threadSession.get();  
      try {  
          if (s == null) {  
              s = getSessionFactory().openSession();  
              threadSession.set(s);  
          }  
      } catch (HibernateException ex) {  
          throw new InfrastructureException(ex);  
      }  
      return s;  
  }  
  ```

  **在线程内部创建ThreadLocal**

  还有一种用法是在线程类内部创建ThreadLocal，基本步骤如下：

  - 在多线程的类(如ThreadDemo类)中，创建一个ThreadLocal对象threadXxx，用来保存线程间需要隔离处理的对象xxx。
  - 在ThreadDemo类中，创建一个获取要隔离访问的数据的方法getXxx()，在方法中判断，若ThreadLocal对象为null时候，应该new()一个隔离访问类型的对象，并强制转换为要应用的类型。
  - 在ThreadDemo类的run()方法中，通过调用getXxx()方法获取要操作的数据，这样可以保证每个线程对应一个数据对象，在任何时刻都操作的是这个对象。

  ```java
  public class ThreadLocalTest implements Runnable{
      
      ThreadLocal<Student> StudentThreadLocal = new ThreadLocal<Student>();
  
      @Override
      public void run() {
          String currentThreadName = Thread.currentThread().getName();
          System.out.println(currentThreadName + " is running...");
          Random random = new Random();
          int age = random.nextInt(100);
          System.out.println(currentThreadName + " is set age: "  + age);
          Student Student = getStudentt(); //通过这个方法，为每个线程都独立的new一个Studentt对象，每个线程的的Studentt对象都可以设置不同的值
          Student.setAge(age);
          System.out.println(currentThreadName + " is first get age: " + Student.getAge());
          try {
              Thread.sleep(500);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          System.out.println( currentThreadName + " is second get age: " + Student.getAge());
          
      }
      
      private Student getStudentt() {
          Student Student = StudentThreadLocal.get();
          if (null == Student) {
              Student = new Student();
              StudentThreadLocal.set(Student);
          }
          return Student;
      }
  
      public static void main(String[] args) {
          ThreadLocalTest t = new ThreadLocalTest();
          Thread t1 = new Thread(t,"Thread A");
          Thread t2 = new Thread(t,"Thread B");
          t1.start();
          t2.start();
      }
      
  }
  
  class Student{
      int age;
      public int getAge() {
          return age;
      }
      public void setAge(int age) {
          this.age = age;
      }
      
  }
  ```

  **java 开发手册中推荐的 ThreadLocal**

  看看阿里巴巴 java 开发手册中推荐的 ThreadLocal 的用法:

  ```java
  import java.text.DateFormat;
  import java.text.SimpleDateFormat;
   
  public class DateUtils {
      public static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>(){
          @Override
          protected DateFormat initialValue() {
              return new SimpleDateFormat("yyyy-MM-dd");
          }
      };
  }
  ```

  然后我们再要用到 DateFormat 对象的地方，这样调用：

  ```java
  DateUtils.df.get().format(new Date());
  ```

**ThreadLocalMap对象是什么**

本质上来讲, 它就是一个Map, 但是这个ThreadLocalMap与我们平时见到的Map有点不一样

- 它没有实现Map接口;
- 它没有public的方法, 最多有一个default的构造方法, 因为这个ThreadLocalMap的方法仅仅在ThreadLocal类中调用, 属于静态内部类
- ThreadLocalMap的Entry实现继承了WeakReference<ThreadLocal<?>>
- 该方法仅仅用了一个Entry数组来存储Key, Value; Entry并不是链表形式, 而是每个bucket里面仅仅放一个Entry。



## Java IO/NIO/AIO

## 知识体系

<img src="https://www.pdai.tech/images/io/java-io-overview2.png" alt="img" style="zoom: 200%;" />

## Java IO

### Java IO-分类（传输，操作）

**字节流和字符流的区别**

- 字节流读取单个字节，字符流读取单个字符(一个字符根据编码的不同，对应的字节也不同，如 **UTF-8 编码中文汉字是 3 个字节，GBK编码中文汉字是 2 个字节。**)
- 字节流用来处理二进制文件(图片、MP3、视频文件)，字符流用来处理文本文件(可以看做是特殊的二进制文件，使用了某种编码，人可以阅读)。

简而言之，**字节是给计算机看的，字符才是给人看的**。

**编码就是把字符转换为字节，而解码是把字节重新组合成字符。**

如果编码和解码过程使用不同的编码方式那么就出现了乱码。

- **GBK 编码中，中文字符占 2 个字节，英文字符占 1 个字节；**
- **UTF-8 编码中，中文字符占 3 个字节，英文字符占 1 个字节；**
- **UTF-16be 编码中，中文字符和英文字符都占 2 个字节。**

UTF-16be 中的 be 指的是 Big Endian，也就是大端。相应地也有 UTF-16le，le 指的是 Little Endian，也就是小端。

Java 使用双字节编码 UTF-16be，这不是指 Java 只支持这一种编码方式，而是说 char 这种类型使用 UTF-16be 进行编码。char 类型占 16 位，也就是两个字节，Java 使用这种双字节编码是为了让一个中文或者一个英文都能使用一个 char 来存储。

**IO理解分类 - 从数据操作上**

从数据来源或者说是操作对象角度看，IO 类可以分为:

![img](https://www.pdai.tech/images/io/java-io-category-3.png)

### Java IO-装饰者模式

**装饰者模式**

装饰者(Decorator)和具体组件(ConcreteComponent)都继承自组件(Component)，具体组件的方法实现不需要依赖于其它对象，而装饰者组合了一个组件，这样它可以装饰其它装饰者或者具体组件。所谓装饰，就是把这个装饰者套在被装饰者之上，从而动态扩展被装饰者的功能。装饰者的方法有一部分是自己的，这属于它的功能，然后调用被装饰者的方法实现，从而也保留了被装饰者的功能。可以看到，具体组件应当是装饰层次的最低层，因为只有具体组件的方法实现不需要依赖于其它对象。

以 InputStream 为例，

- InputStream 是抽象组件；

- FileInputStream 是 InputStream 的子类，属于具体组件，提供了字节流的输入操作；

- FilterInputStream 属于抽象装饰者，装饰者用于装饰组件，为组件提供额外的功能。例如 BufferedInputStream 为 FileInputStream 提供缓存的功能。

  ![image](https://www.pdai.tech/images/pics/DP-Decorator-java.io.png)

try-with-resource 可以自动自动释放资源，需要是实现了 AutoCloseable 接口的类。

### Java IO - 源码: InputStream

InputStream 类重要方法设计如下：

```java
// 读取下一个字节，如果没有则返回-1
public abstract int read() 

// 将读取到的数据放在 byte 数组中，该方法实际上调用read(byte b[], int off, int len)方法
public int read(byte b[]) 

// 从第 off 位置读取<b>最多(实际可能小于)</b> len 长度字节的数据放到 byte 数组中，流是以 -1 来判断是否读取结束的; 此方法会一直阻止，直到输入数据可用、检测到stream结尾或引发异常为止。
public int read(byte b[], int off, int len) 

// JDK9新增：读取 InputStream 中的所有剩余字节，调用readNBytes(Integer.MAX_VALUE)方法
public byte[] readAllBytes()

// JDK11更新：读取 InputStream 中的剩余字节的指定上限大小的字节内容；此方法会一直阻塞，直到读取了请求的字节数、检测到流结束或引发异常为止。此方法不会关闭输入流。
public byte[] readNBytes(int len)

// JDK9新增：从输入流读取请求的字节数并保存在byte数组中； 此方法会一直阻塞，直到读取了请求的字节数、检测到流结束或引发异常为止。此方法不会关闭输入流。
public int readNBytes(byte[] b, int off, int len)

// 跳过指定个数的字节不读取
public long skip(long n) 

// 返回可读的字节数量
public int available() 

// 读取完，关闭流，释放资源
public void close() 

// 标记读取位置，下次还可以从这里开始读取，使用前要看当前流是否支持，可以使用 markSupport() 方法判断
public synchronized void mark(int readlimit) 

// 重置读取位置为上次 mark 标记的位置
public synchronized void reset() 

// 判断当前流是否支持标记流，和上面两个方法配套使用
public boolean markSupported() 

// JDK9新增：读取 InputStream 中的全部字节并写入到指定的 OutputStream 中
public long transferTo(OutputStream out)
```

**jdk9新增**

类 java.io.InputStream 中增加了新的方法来读取和复制 InputStream 中包含的数据。

- `readAllBytes`：读取 InputStream 中的所有剩余字节。
- `readNBytes`： 从 InputStream 中读取指定数量的字节到数组中。
- `transferTo`：读取 InputStream 中的全部字节并写入到指定的 OutputStream 中 。

**`read(byte[], int, int)` 和 `readNBytes(byte[], int, int)`看似是实现的相同功能，为何会设计readNBytes方法呢**？

1. read(byte[], int, int)是尝试读到最多len个bytes，但是**读取到的内容长度可能是小于len**的。
2. readNBytes(byte[], int, int) 会一直（while循环）查找直到stream尾为止

举个例子：如果文本内容是`12345`, read(s,0,10)是允许返回`123`的, 而readNbytes(s,0,10)会一直（while循环）查找直到stream尾为止，并返回`12345`.

**JDK11新增**

增加了一个nullInputStream，即空模式实现，以便可以直接调用而不用判空

## Java IO - 源码: OutputStream

OutputStream 类重要方法设计如下：

```java
// 写入一个字节，可以看到这里的参数是一个 int 类型，对应上面的读方法，int 类型的 32 位，只有低 8 位才写入，高 24 位将舍弃。
public abstract void write(int b)

// 将数组中的所有字节写入，实际调用的是write(byte b[], int off, int len)方法。
public void write(byte b[])

// 将 byte 数组从 off 位置开始，len 长度的字节写入
public void write(byte b[], int off, int len)

// 强制刷新，将缓冲中的数据写入; 默认是空实现，供子类覆盖
public void flush()

// 关闭输出流，流被关闭后就不能再输出数据了; 默认是空实现，供子类覆盖
public void close()
```

### Java IO - 常见类使用

**Java 的 I/O 大概可以分成以下几类:**

- 磁盘操作: File
- 字节操作: InputStream 和 OutputStream
- 字符操作: Reader 和 Writer
- 对象操作: Serializable
- 网络操作: Socket

序列化就是将一个对象转换成字节序列，方便存储和传输。

- 序列化: ObjectOutputStream.writeObject()
- 反序列化: ObjectInputStream.readObject()

**不会对静态变量进行序列化，因为序列化只是保存对象的状态，静态变量属于类的状态。**

序列化的类需要**实现 Serializable 接口**，它只是一个标准，没有任何方法需要实现，但是如果不去实现它的话而进行序列化，会抛出异常。

**transient**

transient 关键字可以使一些属性不会被序列化。

**Java 中的网络支持:**

- InetAddress: 用于表示网络上的硬件资源，即 IP 地址；
- URL: 统一资源定位符；
- Sockets: 使用 TCP 协议实现网络通信；
- Datagram: 使用 UDP 协议实现网络通信。

**InetAddress**

没有公有的构造函数，只能通过静态方法来创建实例。

```java
InetAddress.getByName(String host);
InetAddress.getByAddress(byte[] address);
```

**URL：可以直接从 URL 中读取字节流数据。**

可以直接从 URL 中读取字节流数据。

```java
public static void main(String[] args) throws IOException {

    URL url = new URL("http://www.baidu.com");

    /* 字节流 */
    InputStream is = url.openStream();

    /* 字符流 */
    InputStreamReader isr = new InputStreamReader(is, "utf-8");

    /* 提供缓存功能 */
    BufferedReader br = new BufferedReader(isr);

    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }

    br.close();
}
```

**Sockets**

- ServerSocket: 服务器端类
- Socket: 客户端类
- 服务器和客户端通过 InputStream 和 OutputStream 进行输入输出。
- ![image](https://www.pdai.tech/images/pics/ClienteServidorSockets1521731145260.jpg)

**Datagram**

- DatagramSocket: 通信类
- DatagramPacket: 数据包类

**Java 字节读取流的read方法返回int的原因**

**read()的底层是由C++实现的**，返回的是unsigned byte，**取值范围为[0~255]**，在java中没有对应的类型，所以只能用int类型接收，由Java接收得到的就是int[0、255]。

### IO 模型 - Unix IO 模型

Unix 下有五种 I/O 模型:

- 阻塞式 I/O（BIO）
- 非阻塞式 I/O(NIO)
- I/O 复用(select 和 poll)
- 信号驱动式 I/O(SIGIO)
- 异步 I/O(AIO)

一个输入操作通常包括两个阶段:

- 等待数据准备好
- 从内核向进程复制数据

对于一个套接字上的输入操作，第一步通常涉及等待数据从网络中到达。当所等待分组到达时，它被复制到内核中的某个缓冲区。第二步就是把数据从内核缓冲区复制到应用进程缓冲区。

##### 阻塞式 I/O

应用进程被阻塞，直到数据复制到应用进程缓冲区中才返回。

应该注意到，在阻塞的过程中，其它程序还可以执行，因此阻塞不意味着整个操作系统都被阻塞。因为其他程序还可以执行，因此不消耗 CPU 时间，这种模型的执行效率会比较高。

![img](https://www.pdai.tech/images/io/java-io-model-0.png)

##### 非阻塞式 I/O

应用进程执行系统调用之后，内核返回一个错误码。应用进程可以继续执行，但是需要不断的执行系统调用来获知 I/O 是否完成，这种方式称为轮询(polling)。

由于 CPU 要处理更多的系统调用，因此这种模型是比较**低效**的。

![img](https://www.pdai.tech/images/io/java-io-model-1.png)

##### I/O 复用

使用 select 或者 poll 等待数据，并且可以等待多个套接字中的任何一个变为可读，这一过程会被**阻塞**，当某一个套接字可读时返回。之后再使用 recvfrom 把数据从内核复制到进程中。

它可以让单个进程具有处理多个 I/O 事件的能力。又被称为 Event Driven I/O，即事件驱动 I/O。

**如果一个 Web 服务器没有 I/O 复用，那么每一个 Socket 连接都需要创建一个线程去处理。**如果同时有几万个连接，那么就需要创建相同数量的线程。并且相比于多进程和多线程技术，I/O 复用不需要进程线程创建和切换的开销，系统开销更小。

![img](https://www.pdai.tech/images/io/java-io-model-2.png)

##### 信号驱动 I/O

应用进程使用 sigaction 系统调用，内核立即返回，应用进程可以继续执行，也就是说等待数据阶段应用进程是**非阻塞**的。内核在数据到达时向应用进程发送 SIGIO 信号，应用进程收到之后在信号处理程序中调用 recvfrom 将数据从内核复制到应用进程中。

相比于非阻塞式 I/O 的轮询方式，信号驱动 I/O 的 CPU 利用率更高。

![img](https://www.pdai.tech/images/io/java-io-model-3.png)

##### 异步 I/O

进行 aio_read 系统调用会立即返回，应用进程继续执行，**不会被阻塞**，内核会在所有操作完成之后向应用进程发送信号。

异步 I/O 与信号驱动 I/O 的区别在于，**异步 I/O 的信号是通知应用进程 I/O 完成，而信号驱动 I/O 的信号是通知应用进程可以开始 I/O。**

![img](https://www.pdai.tech/images/io/java-io-model-4.png)

##### I/O 模型比较

###### 同步 I/O 与异步 I/O

- 同步 I/O: 应用进程在调用 recvfrom 操作时会阻塞。
- 异步 I/O: 不会阻塞。

阻塞式 I/O、非阻塞式 I/O、I/O 复用和信号驱动 I/O 都是同步 I/O，虽然非阻塞式 I/O 和信号驱动 I/O 在**等待数据阶段不会阻塞，但是在之后的将数据从内核复制到应用进程这个操作会阻塞。**

前四种 I/O 模型的主要区别在于第一个阶段，而第二个阶段是一样的: 将数据从**内核复制到应用进程过程**中，应用进程会被阻塞。

##### IO多路复用

###### IO多路复用工作模式

epoll 的描述符事件有两种触发模式: LT(level trigger)和 ET(edge trigger)。

**[#](#_1-lt-模式) 1. LT 模式**

当 epoll_wait() 检测到描述符事件到达时，将此事件通知进程，**进程可以不立即处理该事件**，下次调用 epoll_wait() 会再次通知进程。是默认的一种模式，并且同时**支持 Blocking 和 No-Blocking**。

**[#](#_2-et-模式) 2. ET 模式**

和 LT 模式不同的是，**通知之后进程必须立即处理事件**，下次再调用 epoll_wait() 时不会再得到事件到达的通知。

很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。**只支持 No-Blocking**，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

**[#](#应用场景) 应用场景**

很容易产生一种错觉认为只要用 epoll 就可以了，select 和 poll 都已经过时了，其实它们都有各自的使用场景。

**[#](#_1-select-应用场景) 1. select 应用场景**

select 的 timeout 参数精度为 1ns，而 poll 和 epoll 为 1ms，因此 **select 更加适用于实时要求更高的场景**，比如**核反应堆的控制**。

**select 可移植性更好，几乎被所有主流平台所支持**。

**[#](#_2-poll-应用场景) 2. poll 应用场景**

poll **没有最大描述符数量的限制**，如果**平台支持并且对实时性要求不高**，应该使用 poll 而不是 select。

需要同时监控小于 1000 个描述符，就没有必要使用 epoll，因为这个应用场景下并不能体现 epoll 的优势。

需要监控的描述符状态变化多，而且都是非常短暂的，也没有必要使用 epoll。因为 epoll 中的所有描述符都存储在内核中，造成每次需要对描述符的状态改变都需要通过 epoll_ctl() 进行系统调用，频繁系统调用降低效率。并且epoll 的描述符存储在内核，不容易调试。

**[#](#_3-epoll-应用场景) 3. epoll 应用场景**

只需要运行在 **Linux** 平台上，并且有非常大量的描述符需要**同时轮询**，而且这些连接**最好是长连接**。

### Java IO - BIO 详解

- `阻塞IO` 和 `非阻塞IO`

这两个概念是`程序级别`的。主要描述的是程序请求操作系统IO操作后，如果IO资源没有准备好，那么程序该如何处理的问题: 前者等待；后者继续执行(并且使用线程一直轮询，直到有IO资源准备好了)

- `同步IO` 和 `非同步IO`

这两个概念是`操作系统级别`的。主要描述的是操作系统在收到程序请求IO操作后，如果IO资源没有准备好，该如何响应程序的问题: 前者不响应，直到IO资源准备好以后；后者返回一个标记(好让程序和自己知道以后的数据往哪里通知)，当IO资源准备好以后，再用事件机制返回给程序。

**传统的BIO的问题**

- 同一时间，服务器只能接受来自于客户端A的请求信息；虽然客户端A和客户端B的请求是同时进行的，但客户端B发送的请求信息只能等到服务器接受完A的请求数据后，才能被接受。
- 由于**服务器一次只能处理一个客户端请求**，当处理完成并返回后(或者异常时)，才能进行第二次请求的处理。很显然，这样的处理方式在高并发的情况下，是不能采用的。

**多线程方式-伪异步**：服务器接收到数据报文后的“业务处理过程”可以多线程，但是**数据报文的接收还是需要一个一个的来**

如果操作系统没有发现有套接字从指定的端口X来，那么操作系统就会等待。这样serverSocket.accept()方法就会一直等待。这就是为什么accept()方法为什么会阻塞: 它内部的实现是使用的操作系统级别的同步IO。

## Java NIO

### Java NIO - 基础详解

I/O 与 NIO 最重要的区别是数据打包和传输的方式，I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。

##### **通道与缓冲区**

**Selector**

Selector的英文含义是“选择器”，不过根据我们详细介绍的Selector的岗位职责，您可以把它称之为“轮询代理器”、“事件订阅器”、“channel容器管理机”都行。

**1. 通道**

通道 Channel 是对原 I/O 包中的流的模拟，可以通过它读取和写入数据。

通道与流的不同之处在于，流只能在一个方向上移动(一个流必须是 InputStream 或者 OutputStream 的子类)，而**通道是双向的**，可以**用于读、写或者同时用于读写**。

通道，被建立的一个**应用程序和操作系统交互事件、传递内容的渠道**(注意是连接到操作系统)。一个通道会有一个专属的文件状态描述符。那么既然是和操作系统进行内容的传递，那么说明应用程序可以通过通道读取数据，也可以通过通道向操作系统写数据。

通道包括以下类型:

- FileChannel: 从文件中读写数据；
- DatagramChannel: 通过 UDP 读写网络中数据；
- SocketChannel: 通过 TCP 读写网络中数据；
- ServerSocketChannel: 可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。

**2. 缓冲区**

**发送给一个通道的所有数据都必须首先放到缓冲区中**，同样地，从通道中读取的任何数据都要先读到缓冲区中。也就是说，不会直接对通道进行读写数据，而是要先经过缓冲区。

**缓冲区实质上是一个数组**，但它不仅仅是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

缓冲区包括以下类型:

- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer

**缓冲区状态变量**

- capacity: 最大容量；
- position: 当前已经读写的字节数；
- limit: 还可以读写的字节数。
- flip():切换读写

以下展示了使用 NIO 快速复制文件的实例:

```java
public static void fastCopy(String src, String dist) throws IOException {

    /* 获得源文件的输入字节流 */
    FileInputStream fin = new FileInputStream(src);

    /* 获取输入字节流的文件通道 */
    FileChannel fcin = fin.getChannel();

    /* 获取目标文件的输出字节流 */
    FileOutputStream fout = new FileOutputStream(dist);

    /* 获取输出字节流的通道 */
    FileChannel fcout = fout.getChannel();

    /* 为缓冲区分配 1024 个字节 */
    ByteBuffer buffer = ByteBuffer.allocateDirect(1024);

    while (true) {

        /* 从输入通道中读取数据到缓冲区中 */
        int r = fcin.read(buffer);

        /* read() 返回 -1 表示 EOF */
        if (r == -1) {
            break;
        }

        /* 切换读写 */
        buffer.flip();

        /* 把缓冲区的内容写入输出文件中 */
        fcout.write(buffer);
        
        /* 清空缓冲区 */
        buffer.clear();
    }
}
```

##### **选择器**

NIO 实现了 IO 多路复用中的 Reactor 模型，一个线程 Thread 使用一个选择器 Selector **通过轮询的方式去监听多个通道 Channel 上的事件，从而让一个线程就可以处理多个事件。**

通过配置监听的通道 Channel 为非阻塞，那么当 Channel 上的 IO 事件还未到达时，就不会进入阻塞状态一直等待，而是继续轮询其它 Channel，找到 IO 事件已经到达的 Channel 执行。

应该注意的是，只有套接字 Channel 才能配置为非阻塞，而 FileChannel 不能，为 FileChannel 配置非阻塞也没有意义。

![image](https://www.pdai.tech/images/pics/4d930e22-f493-49ae-8dff-ea21cd6895dc.png)

**1. 创建选择器**

```java
Selector selector = Selector.open();
```

**2. 将通道注册到选择器上**

```java
ServerSocketChannel ssChannel = ServerSocketChannel.open();
ssChannel.configureBlocking(false);
ssChannel.register(selector, SelectionKey.OP_ACCEPT);
```

**3. 监听事件**

```java
int num = selector.select();
```

使用 select() 来监听到达的事件，它会一直阻塞直到有至少一个事件到达。

**4. 获取到达的事件**

```java
Set<SelectionKey> keys = selector.selectedKeys();
Iterator<SelectionKey> keyIterator = keys.iterator();
while (keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if (key.isAcceptable()) {
        // ...
    } else if (key.isReadable()) {
        // ...
    }
    keyIterator.remove();
}
```

**5. 事件循环**

因为一次 select() 调用不能处理完所有的事件，并且服务器端有可能需要一直监听事件，因此服务器端处理事件的代码一般会放在一个死循环内。

```java
while (true) {
    int num = selector.select();
    Set<SelectionKey> keys = selector.selectedKeys();
    Iterator<SelectionKey> keyIterator = keys.iterator();
    while (keyIterator.hasNext()) {
        SelectionKey key = keyIterator.next();
        if (key.isAcceptable()) {
            // ...
        } else if (key.isReadable()) {
            // ...
        }
        keyIterator.remove();
    }
}
```

**内存映射文件**

内存映射文件 I/O 是一种读和写文件数据的方法，它可以比常规的基于流或者基于通道的 I/O 快得多。

向内存映射文件写入可能是危险的，只是改变数组的单个元素这样的简单操作，就可能会直接修改磁盘上的文件。修改数据与将数据保存到磁盘是没有分开的。

下面代码行将文件的前 1024 个字节映射到内存中，map() 方法返回一个 MappedByteBuffer，它是 ByteBuffer 的子类。因此，可以像使用其他任何 ByteBuffer 一样使用新映射的缓冲区，操作系统会在需要时负责执行映射。

```java
MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_WRITE, 0, 1024);
```

**对比**

NIO 与普通 I/O 的区别主要有以下两点:

- NIO 是非阻塞的
- NIO 面向块，I/O 面向流

### Java NIO-IO多路复用

目前流程的多路复用IO实现主要包括四种: `select`、`poll`、`epoll`、`kqueue`。下表是他们的一些重要特性的比较:

| IO模型 | 相对性能 | 关键思路         | 操作系统      | JAVA支持情况                                                 |
| ------ | -------- | ---------------- | ------------- | ------------------------------------------------------------ |
| select | 较高     | Reactor          | windows/Linux | 支持,Reactor模式(反应器设计模式)。Linux操作系统的 kernels 2.4内核版本之前，默认使用select；而目前windows下对同步IO的支持，都是select模型 |
| poll   | 较高     | Reactor          | Linux         | Linux下的JAVA NIO框架，Linux kernels 2.6内核版本之前使用poll进行支持。也是使用的Reactor模式 |
| epoll  | 高       | Reactor/Proactor | Linux         | Linux kernels 2.6内核版本及以后使用epoll进行支持；Linux kernels 2.6内核版本之前使用poll进行支持；另外一定注意，由于Linux下没有Windows下的IOCP技术提供真正的 异步IO 支持，所以Linux下使用epoll模拟异步IO |
| kqueue | 高       | Proactor         | Linux         | 目前JAVA的版本不支持                                         |

**多路复用IO技术最适用的是“高并发”场景**，所谓高并发是指1毫秒内至少同时有上千个连接请求准备好。其他情况下多路复用IO技术发挥不出来它的优势。另一方面，使用JAVA NIO进行功能实现，相对于传统的Socket套接字实现要复杂一些，所以实际应用中，需要根据自己的业务需求进行技术选择。

#### **Reactor事件驱动模型**

**传统IO模型**

![img](https://www.pdai.tech/images/io/java-io-reactor-1.png)

从图中可以看出，传统IO的特点在于：

- 每个客户端连接到达之后，服务端会分配一个线程给该客户端，该线程会处理包括读取数据，解码，业务计算，编码，以及发送数据整个过程；
- 同一时刻，服务端的吞吐量与服务器所提供的线程数量是呈线性关系的。

**Reactor事件驱动模型**

在传统IO模型中，由于线程在等待连接以及进行IO操作时都会阻塞当前线程，这部分损耗是非常大的。因而jdk 1.4中就提供了一套非阻塞IO的API。该API本质上是以事件驱动来处理网络事件的，而Reactor是基于该API提出的一套IO模型。

![img](https://www.pdai.tech/images/io/java-io-reactor-2.png)

在Reactor模型中，主要有四个角色：客户端连接，Reactor，Acceptor和Handler。这里Acceptor会不断地接收客户端的连接，然后将接收到的连接交由Reactor进行分发，最后有具体的Handler进行处理。

改进后的Reactor模型相对于传统的IO模型主要有如下优点：

- 从模型上来讲，如果仅仅还是只使用一个线程池来处理客户端连接的网络读写，以及业务计算，那么Reactor模型与传统IO模型在效率上并没有什么提升。但是**Reactor模型是以事件进行驱动的**，其能够将接收客户端连接，+ 网络读和网络写，以及业务计算进行**拆分**，从而极大的提升处理效率；
- Reactor模型是异步非阻塞模型，**工作线程在没有网络事件时可以处理其他的任务**，而不用像传统IO那样必须阻塞等待。

**Reactor模型----业务处理与IO分离**

在上面的Reactor模型中，由于网络读写和业务操作都在**同一个线程**中，在高并发情况下，这里的系统瓶颈主要在两方面：

- **高频率的网络读写事件处理；**
- **大量的业务操作处理**；

基于上述两个问题，这里在单线程Reactor模型的基础上提出了使用**线程池**的方式处理业务操作的模型。如下是该模型的示意图：

![img](https://www.pdai.tech/images/io/java-io-reactor-3.png)

从图中可以看出，在多线程进行业务操作的模型下，该模式主要具有如下特点：

- 使用**一个线程**进行**客户端连接的接收**以及**网络读写事件**的处理；
- 在接收到客户端连接之后，将该连接交由**线程池进行数据的编解码以及业务计算**。

这种模式相较于前面的模式性能有了很大的提升，主要在于在进行网络读写的同时，也进行了业务计算，从而大大提升了系统的吞吐量。但是这种模式也有其不足，主要在于：

- **网络读写**是一个比较**消耗CPU**的操作，在**高并发**的情况下，将会有大量的客户端数据需要进行网络读写，此时一个线程将**不足以处理这么多请求**。

**Reactor模型----并发读写**

对于使用线程池处理业务操作的模型，由于**网络读写在高并发**情况下会成为系统的一个**瓶颈**，因而针对该模型这里提出了一种改进后的模型，即使用**线程池**进行网络读写，而仅仅只使用一个线程专门接收客户端连接。如下是该模型的示意图：

![img](https://www.pdai.tech/images/io/java-io-reactor-4.png)

可以看到，改进后的Reactor模型将Reactor拆分为了mainReactor和subReactor。这里mainReactor主要进行客户端连接的处理，处理完成之后将该连接交由subReactor以处理客户端的网络读写。这里的**subReactor则是使用一个线程池来支撑**的，其读写能力将会随着线程数的增多而大大增加。对于**业务操作，这里也是使用一个线程池，而每个业务请求都只需要进行编解码和业务计算**。通过这种方式，服务器的性能将会大大提升，在可见情况下，其基本上可以支持百万连接。

#### **JAVA对多路复用IO的支持**

![img](https://www.pdai.tech/images/io/java-io-nio-1.png)

**重要概念: Channel**

通道，被建立的一个应用程序和操作系统**交互事件**、**传递内容**的渠道(注意是连接到操作系统)。一个通道会有一个专属的文件状态描述符。那么既然是和操作系统进行内容的传递，那么说明应用程序可以通过通道读取数据，也可以通过通道向操作系统写数据。

所有被Selector(选择器)注册的通道，只能是继承了SelectableChannel类的子类。

- ServerSocketChannel: **应用服务器程序**的监听通道。只有通过这个通道，应用程序才能向操作系统注册支持“多路复用IO”的端口监听。同时支持UDP协议和TCP协议。
- ScoketChannel: **TCP Socket套接字**的监听通道，一个Socket套接字对应了一个客户端IP: 端口 到 服务器IP: 端口的通信连接。
- DatagramChannel: **UDP 数据报文**的监听通道。

**重要概念: Buffer**

数据缓存区: 在JAVA NIO 框架中，为了保证每个通道的数据读写速度JAVA NIO 框架为每一种需要支持数据读写的通道集成了Buffer的支持。

这句话怎么理解呢? 例如ServerSocketChannel通道它只支持对OP_ACCEPT事件的监听，所以它是不能直接进行网络数据内容的读写的。所以ServerSocketChannel是没有集成Buffer的。

Buffer有两种工作模式: **写模式**和**读模式**。在读模式下，应用程序只能从Buffer中读取数据，不能进行写操作。但是在写模式下，应用程序是可以进行读操作的，这就表示可能会出现脏读的情况。所以一旦您决定要从Buffer中读取数据，一定要将Buffer的状态改为读模式。

**重要概念: Selector**

Selector的英文含义是“选择器”，不过根据我们详细介绍的Selector的岗位职责，您可以把它称之为“轮询代理器”、“事件订阅器”、“channel容器管理机”都行。

- 事件订阅和Channel管理

应用程序将向Selector对象注册需要它关注的Channel，以及具体的某一个Channel会对哪些IO事件感兴趣。Selector中也会维护一个“已经注册的Channel”的容器。

- 轮询代理

应用层不再通过阻塞模式或者非阻塞模式直接询问操作系统“事件有没有发生”，而是由Selector代其询问。

- 实现不同操作系统的支持

之前已经提到过，多路复用IO技术 是需要操作系统进行支持的，其特点就是操作系统可以同时扫描同一个端口上不同网络连接的事件。所以作为上层的JVM，必须要为 不同操作系统的多路复用IO实现 编写不同的代码。同样我使用的测试环境是Windows，它对应的实现类是sun.nio.ch.WindowsSelectorImpl



**客户端是否使用多路复用IO技术，对整个系统架构的性能提升相关性不大**



serverChannel.register(Selector sel, int ops, Object att): 实际上register(Selector sel, int ops, Object att)方法是ServerSocketChannel类的父类AbstractSelectableChannel提供的一个方法，表示只要继承了AbstractSelectableChannel类的子类都可以注册到选择器中。通过观察整个AbstractSelectableChannel继承关系，下图中的这些类可以被注册到选择器中:

| 通道类              | 通道作用     | 可关注的事件                                                 |
| ------------------- | ------------ | ------------------------------------------------------------ |
| ServerSocketChannel | 服务器端通道 | SelectionKey.OP_ACCEPT                                       |
| DatagramChannel     | UDP协议通道  | SelectionKey.OP_READ、SelectionKey.OP_WRITE                  |
| SocketChannel       | TCP协议通道  | SelectionKey.OP_READ、SelectionKey.OP_WRITE、SelectionKey.OP_CONNECT |

实际上通过每一个AbstractSelectableChannel子类所实现的public final int validOps()方法，就可以查看这个通道“可以关心的IO事件”。

selector.selectedKeys().iterator(): 当选择器Selector收到操作系统的IO操作事件后，它的selectedKeys将在下一次轮询操作中，收到这些事件的关键描述字(不同的channel，就算关键字一样，也会存储成两个对象)。但是每一个“事件关键字”被处理后都必须移除，否则下一次轮询时，这个事件会被重复处理。

**多路复用IO的优缺点**

- 不用再使用多线程来进行IO处理了(包括操作系统内核IO管理模块和应用程序进程而言)。当然实际业务的处理中，应用程序进程还是可以引入线程池技术的
- 同一个端口可以处理多种协议，例如，使用ServerSocketChannel测测的服务器端口监听，既可以处理TCP协议又可以处理UDP协议。
- 操作系统级别的优化: 多路复用IO技术可以是操作系统级别在一个端口上能够同时接受多个客户端的IO事件。同时具有之前我们讲到的阻塞式同步IO和非阻塞式同步IO的所有特点。Selector的一部分作用更相当于“轮询代理器”。
- 都是同步IO: 目前我们介绍的 阻塞式IO、非阻塞式IO甚至包括多路复用IO，这些都是基于操作系统级别对“同步IO”的实现。我们一直在说“同步IO”，一直都没有详细说，什么叫做“**同步IO**”。实际上一句话就可以说清楚: **只有上层(包括上层的某种代理机制)系统询问我是否有某个事件发生了，否则我不会主动告诉上层系统事件发生了**

### Java AIO - 异步IO详解

阻塞式同步IO、非阻塞式同步IO、多路复用IO 这三种IO模型，以及JAVA对于这三种IO模型的支持。重点说明了IO模型是由操作系统提供支持，且这三种IO模型都是同步IO，都是采用的“**应用程序不询问我，我绝不会主动通知**”的方式。

异步IO则是采用“**订阅-通知**”模式: 即应用程序向操作系统注册IO监听，然后继续做自己的事情。当操作系统发生IO事件，并且准备好数据后，在**主动通知**应用程序，触发相应的函数:

![img](https://www.pdai.tech/images/io/java-io-aio-1.png)

Linux下由于没有这种异步IO技术，所以使用的是epoll(上文介绍过的一种多路复用IO技术的实现)对异步IO进行模拟。

**为什么还有Netty**

虽然JAVA NIO 和 JAVA AIO框架提供了 多路复用IO/异步IO的支持，但是并没有提供上层“信息格式”的良好封装。例如前两者并**没有提供**针对 Protocol Buffer、JSON这些**信息格式的封装**，但是Netty框架提供了这些数据格式封装(基于责任链模式的编码和解码功能)

要编写一个可靠的、易维护的、高性能的(注意它们的排序)NIO/AIO 服务器应用。除了框架本身要兼容实现各类操作系统的实现外。更重要的是它应该还要处理很多上层特有服务，例如: **客户端的权限、还有上面提到的信息格式封装、简单的数据读取。这些Netty框架都提供了响应的支持**。

JAVA NIO框架存在一个poll/epoll bug: Selector doesn’t block on Selector.select(timeout)，不能block意味着CPU的使用率会变成100%(这是底层JNI的问题，上层要处理这个异常实际上也好办)。当然这个bug只有在Linux内核上才能重现。

这个问题在JDK 1.7版本中还没有被完全解决: http://bugs.java.com/bugdatabase/view_bug.do?bug_id=2147719。虽然Netty 4.0中也是基于JAVA NIO框架进行封装的(上文中已经给出了Netty中NioServerSocketChannel类的介绍)，但是Netty已经将这个bug进行了处理。

### Java N(A)IO - 框架: Netty

> Netty是一个高性能、异步事件驱动的NIO框架，提供了对TCP、UDP和文件传输的支持。作为当前最流行的NIO框架，Netty在互联网领域、大数据分布式计算领域、游戏行业、通信行业等获得了广泛的应用，一些业界著名的开源组件也基于Netty构建，比如RPC框架、zookeeper等。

**优点**

- api简单，开发门槛低
- 功能强大，内置了多种编码、解码功能
- 与其它业界主流的NIO框架对比，netty的综合性能最优
- 社区活跃，使用广泛，经历过很多商业应用项目的考验
- 定制能力强，可以对框架进行灵活的扩展

### Java NIO - 零拷贝实现

#### Linux - 零拷贝技术

**一、为什么要有DMA技术？**

DMA技术，全称为Direct Memory Access，即直接存储器访问（*直接读写内存的方法*）

在没有 DMA 技术前，I/O 的过程是这样的：

- CPU 发出对应的指令给磁盘控制器，然后返回；
- 磁盘控制器收到指令后，于是就开始准备数据，会把数据放入到磁盘控制器的内部缓冲区中，然后产生一个中断；
- CPU 收到中断信号后，停下手头的工作，接着把磁盘控制器的缓冲区的数据一次一个字节地读进自己的寄存器，然后再把寄存器里的数据写入到内存，而在数据传输的期间 CPU 是无法执行其他任务的。

![img](https://www.pdai.tech/images/io/java-io-copy-1.png)

可以看到，**整个数据的传输过程，都要需要 CPU 亲自参与搬运数据的过程，而且这个过程，CPU 是不能做其他事情的。**

于是就发明了 DMA 技术，也就是**直接内存访问（Direct Memory Access）** 技术。

什么是 DMA 技术？简单理解就是，**在进行 I/O 设备和内存的数据传输的时候，数据搬运的工作全部交给 DMA 控制器，而 CPU 不再参与任何与数据搬运相关的事情，这样 CPU 就可以去处理别的事务**。

**DMA 控制器进行数据传输的过程**

**![img](https://www.pdai.tech/images/io/java-io-copy-2.png)**

具体过程：

- 用户进程调用 read 方法，向操作系统发出 I/O 请求，请求读取数据到自己的内存缓冲区中，进程进入阻塞状态；
- 操作系统收到请求后，进一步将 I/O 请求发送 DMA，然后让 CPU 执行其他任务；
- DMA 进一步将 I/O 请求发送给磁盘；
- 磁盘收到 DMA 的 I/O 请求，把数据从磁盘读取到磁盘控制器的缓冲区中，当磁盘控制器的缓冲区被读满后，向 DMA 发起中断信号，告知自己缓冲区已满；
- DMA 收到磁盘的信号，将磁盘控制器缓冲区中的数据拷贝到内核缓冲区中，此时不占用 CPU，CPU 可以执行其他任务；
- 当 DMA 读取了足够多的数据，就会发送中断信号给 CPU；
- CPU 收到 DMA 的信号，知道数据已经准备好，于是将数据从内核拷贝到用户空间，系统调用返回；

可以看到， 整个数据传输的过程，CPU 不再参与数据搬运的工作，而是全程由 DMA 完成，但是 CPU 在这个过程中也是必不可少的，因为传输什么数据，从哪里传输到哪里，都需要 CPU 来告诉 DMA 控制器。

**了解一下传统的文件传输**

如果服务端要提供**文件传输**的功能，我们能想到的最简单的方式是：**将磁盘上的文件读取出来，然后通过网络协议发送给客户端。**

传统 I/O 的工作方式是，数据读取和写入是从用户空间到内核空间来回复制，而内核空间的数据是通过操作系统层面的 I/O 接口从磁盘读取或写入。

![img](https://www.pdai.tech/images/io/java-io-copy-3.png)

**期间共发生了 4 次用户态与内核态的上下文切换**，因为发生了两次系统调用，一次是 read() ，一次是 write()，每次系统调用都得先从用户态切换到内核态，等内核完成任务后，再从内核态切换回用户态。

**要想提高文件传输的性能，就需要减少「用户态与内核态的上下文切换」和「内存拷贝」的次数**。

上下文切换原因：读取磁盘数据的时候，**之所以要发生上下文切换，这是因为用户空间没有权限操作磁盘或网卡，内核的权限最高**，这些操作设备的过程都需要交由操作系统内核来完成，所以一般要通过内核去完成某些任务的时候，就需要使用操作系统提供的系统调用函数。

如何减少「数据拷贝」的次数？

传统的文件传输方式会历经 4 次数据拷贝，而且这里面，「从内核的读缓冲区拷贝到用户的缓冲区里，再从用户的缓冲区里拷贝到 socket 的缓冲区里」，这个过程是没有必要的。

因为文件传输的应用场景中，在用户空间我们并不会对数据「再加工」，所以数据实际上可以不用搬运到用户空间，因此**用户的缓冲区是没有必要存在的**。

#### 如何实现零拷贝？

零拷贝技术实现的方式通常有 2 种：

- mmap + write
- sendfile



**mmap + write**

在前面我们知道，read() 系统调用的过程中会把内核缓冲区的数据拷贝到用户的缓冲区里，于是为了减少这一步开销，我们可以用 mmap() 替换 read() 系统调用函数。

```c
buf = mmap(file, len);
write(sockfd, buf, len);
```

mmap() 系统调用函数会直接把内核缓冲区里的数据「**映射**」到用户空间，这样，操作系统内核与用户空间就不需要再进行任何的数据拷贝操作。

![img](https://www.pdai.tech/images/io/java-io-copy-4.png)

但这还不是最理想的零拷贝，因为仍然需要通过 CPU 把内核缓冲区的数据拷贝到 socket 缓冲区里，而且仍然需要 **4 次上下文切换和3次数据拷贝**，因为系统调用还是 2 次。

**sendfile**

在 Linux 内核版本 2.1 中，提供了一个专门发送文件的系统调用函数 sendfile()，函数形式如下：

```c
#include <sys/socket.h>
ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```

它的前两个参数分别是目的端和源端的文件描述符，后面两个参数是源端的偏移量和复制数据的长度，返回值是实际复制数据的长度。

该系统调用，可以直接把内核缓冲区里的数据拷贝到 socket 缓冲区里，不再拷贝到用户态，这样就只有 **2 次上下文切换，和 3 次数据拷贝**。

![img](https://www.pdai.tech/images/io/java-io-copy-5.png)

但是这还不是真正的零拷贝技术，如果网卡支持 SG-DMA（**The Scatter-Gather Direct Memory Access**）技术（和普通的 DMA 有所不同），我们可以进一步减少通过 CPU 把内核缓冲区里的数据拷贝到 socket 缓冲区的过程。

![img](https://www.pdai.tech/images/io/java-io-copy-6.png)

这就是所谓的**零拷贝（Zero-copy）技术，因为我们没有在内存层面去拷贝数据，也就是说全程没有通过 CPU 来搬运数据，所有的数据都是通过 DMA 来进行传输的**。

零拷贝技术的文件传输方式相比传统文件传输的方式，**减少了 2 次上下文切换和数据拷贝次数，只需要 2 次上下文切换和数据拷贝次数，就可以完成文件的传输，而且 2 次的数据拷贝过程，都不需要通过 CPU，2 次都是由 DMA 来搬运**。

所以，总体来看，零拷贝技术可以把文件传输的性能提高至少一倍以上。

文件传输过程，其中第一步都是先需要先把磁盘文件数据拷贝「内核缓冲区」里，这个「**内核缓冲区**」实际上是**磁盘高速缓存**（PageCache）。由于零拷贝使用了 PageCache 技术，可以使得零拷贝进一步提升了性能，但是，内存空间远比磁盘要小，内存注定只能拷贝磁盘里的**一小部分**数据。

PageCache 的优点主要是两个：

- 缓存最近被访问的数据；
- 预读功能；

针对**大文件**的传输，不应该使用 PageCache，也就是说不应该使用零拷贝技术，因为可能由于 PageCache 被大文件占据，而导致「热点」小文件无法利用到 PageCache，这样在高并发的环境下，会带来严重的性能问题。

**在高并发的场景下，针对大文件的传输的方式，应该使用「异步 I/O + 直接 I/O」来替代零拷贝技术**。

**绕开 PageCache 的 I/O 叫直接 I/O**，使用 PageCache 的 I/O 则叫缓存 I/O。通常，对于磁盘，异步 I/O 只支持直接 I/O。

![img](https://www.pdai.tech/images/io/java-io-copy-9.png)

它把读操作分为两部分：

- 前半部分，内核向磁盘发起读请求，但是可以不等待数据就位就可以返回，于是进程此时可以处理其他任务；
- 后半部分，当内核将磁盘中的数据拷贝到进程缓冲区后，进程将接收到内核的通知，再去处理数据；

传输文件的时候，我们要根据文件的大小来使用不同的方式：

- 传输大文件的时候，使用「**异步 I/O + 直接 I/O**」；
- 传输小文件的时候，则使用「**零拷贝技术**」；

#### Java NIO零拷贝

 Java NIO 对零拷贝的实现，主要包括基于**内存映射（mmap）方式**的 MappedByteBuffer 以及基于 **sendfile 方式**的 FileChannel。

在 Java NIO 中的**通道**（Channel）就相当于操作系统的**内核空间（kernel space）的缓冲区**，而**缓冲区**（Buffer）对应的相当于操作系统的用户空间（user space）中的**用户缓冲区**（user buffer）。

- **通道**（Channel）是全双工的（双向传输），它既可能是读缓冲区（read buffer），也可能是网络缓冲区（socket buffer）。
- **缓冲区**（Buffer）分为堆内存（HeapBuffer）和堆外内存（DirectBuffer），这是通过 malloc() 分配出来的用户态内存。

堆外内存（DirectBuffer）在使用后需要应用程序手动回收，而堆内存（HeapBuffer）的数据在 GC 时可能会被自动回收。因此，在使用 HeapBuffer 读写数据时，为了避免缓冲区数据因为 GC 而丢失，**NIO 会先把 HeapBuffer 内部的数据拷贝到一个临时的 DirectBuffer 中的本地内存（native memory）**，这个拷贝涉及到 `sun.misc.Unsafe.copyMemory()` 的调用，背后的实现原理与 `memcpy()` 类似。 最后，将临时生成的 DirectBuffer 内部的数据的内存地址传给 I/O 调用函数，这样就避免了再去访问 Java 对象处理 I/O 读写。

**MappedByteBuffer**

MappedByteBuffer 是 NIO 基于**内存映射（mmap）**这种零拷贝方式的提供的一种实现，它继承自 ByteBuffer。

MappedByteBuffer 的特点和不足之处：

- **MappedByteBuffer 使用是堆外的虚拟内存**，因此分配（map）的内存大小不受 JVM 的 -Xmx 参数限制，但是也是有大小限制的。 如果当文件超出 Integer.MAX_VALUE 字节限制时，可以通过 position 参数重新 map 文件后面的内容。
- **MappedByteBuffer 在处理大文件时性能的确很高，但也存内存占用、文件关闭不确定等问题**，被其打开的文件只有在垃圾回收的才会被关闭，而且这个时间点是不确定的。
- MappedByteBuffer 提供了文件映射内存的 mmap() 方法，也提供了释放映射内存的 unmap() 方法。然而 unmap() 是 FileChannelImpl 中的私有方法，无法直接显示调用。因此，**用户程序需要通过 Java 反射的调用 sun.misc.Cleaner 类的 clean() 方法手动释放映射占用的内存区域**。

**DirectByteBuffer**

DirectByteBuffer 的对象引用位于 Java 内存模型的堆里面，JVM 可以对 DirectByteBuffer 的对象进行内存分配和回收管理，一般使用 DirectByteBuffer 的静态方法 allocateDirect() 创建 DirectByteBuffer 实例并分配内存。

DirectByteBuffer 内部的字节缓冲区位在于堆外的（用户态）直接内存，它是通过 Unsafe 的本地方法 allocateMemory() 进行内存分配，底层调用的是操作系统的 malloc() 函数。

除此之外，初始化 DirectByteBuffer 时还会创建一个 Deallocator 线程，并通过 Cleaner 的 freeMemory() 方法来对直接内存进行回收操作，freeMemory() 底层调用的是操作系统的 free() 函数。

DirectByteBuffer 和零拷贝有什么关系？

**DirectByteBuffer 是 MappedByteBuffer 的具体实现类**。实际上，Util.newMappedByteBuffer() 方法通过反射机制获取 DirectByteBuffer 的构造器，然后创建一个 DirectByteBuffer 的实例。

除了允许分配操作系统的直接内存以外，DirectByteBuffer 本身也具有文件内存映射的功能，这里不做过多说明。我们需要关注的是，DirectByteBuffer 在 MappedByteBuffer 的基础上提供了内存映像文件的随机读取 get() 和写入 write() 的操作。

**FileChannel**

FileChannel 是一个用于**文件读写、映射和操作的通道**，同时它在并发环境下是线程安全的，基于 FileInputStream、FileOutputStream 或者 RandomAccessFile 的 getChannel() 方法可以创建并打开一个文件通道。FileChannel 定义了 transferFrom() 和 transferTo() 两个抽象方法，它通过在通道和通道之间建立连接实现数据传输的。

#### 其他零拷贝实现

**Netty零拷贝**

Netty 中的零拷贝和上面提到的操作系统层面上的零拷贝不太一样, 我们所说的 Netty 零拷贝完全是基于（Java 层面）用户态的，它的更多的是偏向于数据操作优化这样的概念

**RocketMQ和Kafka对比**

RocketMQ 选择了 mmap + write 这种零拷贝方式，适用于业务级消息这种小块文件的数据持久化和传输；而 Kafka 采用的是 sendfile 这种零拷贝方式，适用于系统日志消息这种高吞吐量的大块文件的数据持久化和传输。但是值得注意的一点是，Kafka 的索引文件使用的是 mmap + write 方式，数据文件使用的是 sendfile 方式。

![img](https://www.pdai.tech/images/io/java-io-copy-11.jpg)

## Java8特性详解

![image-20240223113129375](C:\Users\1\Desktop\JAVA全栈知识体系\JAVA 全栈知识体系.assets\image-20240223113129375.png)

## JVM知识体系

![img](https://www.pdai.tech/images/jvm/jvm-overview.png)

![img](https://www.pdai.tech/images/jvm/java-jvm-overview.png)

### JVM 基础 - 类字节码详解

为什么jvm不能直接运行java代码呢？

这是因为在**cpu**层面看来计算机中所有的操作都是一个个**指令**的运行汇集而成的，java是高级语言，只有人类才能理解其逻辑，计算机是无法识别的，所以java代码必须要先**编译**成字节码文件，jvm才能正确识别代码转换后的指令并将其运行。

#### Java字节码文件

class文件本质上是一个以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑的排列在class文件中。jvm根据其特定的规则解析该二进制数据，从而得到相关信息。

**class文件结构属性**

![img](https://www.pdai.tech/images/jvm/java-jvm-class-2.png)

使用到java内置的一个反编译工具javap可以反编译字节码文件, 用法: `javap  `

例子：

```
public class Main {
    
    private int m;
    
    public int inc() {
        return m + 1;
    }
}
```

输入命令`javap -verbose -p Main.class`查看输出内容:

```java
Classfile /E:/JavaCode/TestProj/out/production/TestProj/com/rhythm7/Main.class
  Last modified 2018-4-7; size 362 bytes
  MD5 checksum 4aed8540b098992663b7ba08c65312de
  Compiled from "Main.java"
public class com.rhythm7.Main
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#18         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#19         // com/rhythm7/Main.m:I
   #3 = Class              #20            // com/rhythm7/Main
   #4 = Class              #21            // java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lcom/rhythm7/Main;
  #14 = Utf8               inc
  #15 = Utf8               ()I
  #16 = Utf8               SourceFile
  #17 = Utf8               Main.java
  #18 = NameAndType        #7:#8          // "<init>":()V
  #19 = NameAndType        #5:#6          // m:I
  #20 = Utf8               com/rhythm7/Main
  #21 = Utf8               java/lang/Object
{
  private int m;
    descriptor: I
    flags: ACC_PRIVATE

  public com.rhythm7.Main();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/rhythm7/Main;

  public int inc();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field m:I
         4: iconst_1
         5: iadd
         6: ireturn
      LineNumberTable:
        line 8: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       7     0  this   Lcom/rhythm7/Main;
}
SourceFile: "Main.java"
```

------



**字节码文件信息**

开头的7行信息包括:Class文件当前所在位置，最后修改时间，文件大小，MD5值，编译自哪个文件，类的全限定名，jdk次版本号，主版本号。

然后紧接着的是该类的访问标志：ACC_PUBLIC, ACC_SUPER，访问标志的含义如下:

| 标志名称       | 标志值 | 含义                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| ACC_PUBLIC     | 0x0001 | 是否为Public类型                                             |
| ACC_FINAL      | 0x0010 | 是否被声明为final，只有类可以设置                            |
| ACC_SUPER      | 0x0020 | 是否允许使用invokespecial字节码指令的新语义．                |
| ACC_INTERFACE  | 0x0200 | 标志这是一个接口                                             |
| ACC_ABSTRACT   | 0x0400 | 是否为abstract类型，对于接口或者抽象类来说，次标志值为真，其他类型为假 |
| ACC_SYNTHETIC  | 0x1000 | 标志这个类并非由用户代码产生                                 |
| ACC_ANNOTATION | 0x2000 | 标志这是一个注解                                             |
| ACC_ENUM       | 0x4000 | 标志这是一个枚举                                             |



**常量池**

`Constant pool`意为常量池。

常量池可以理解成Class文件中的资源仓库。主要存放的是两大类常量：**字面量**(Literal)和符号引用(Symbolic References)。字面量类似于java中的常量概念，如文本字符串，final常量等，而符号引用则属于编译原理方面的概念，包括以下三种:

- 类和接口的全限定名(Fully Qualified Name)
- 字段的名称和描述符号(Descriptor)
- 方法的名称和描述符

关于字节码的类型对应如下：

| 标识字符 | 含义                                       |
| -------- | ------------------------------------------ |
| B        | 基本类型byte                               |
| C        | 基本类型char                               |
| D        | 基本类型double                             |
| F        | 基本类型float                              |
| I        | 基本类型int                                |
| J        | 基本类型long                               |
| S        | 基本类型short                              |
| Z        | 基本类型boolean                            |
| V        | 特殊类型void                               |
| L        | 对象类型，以分号结尾，如Ljava/lang/Object; |

对于数组类型，每一位使用一个前置的`[`字符来描述，如定义一个`java.lang.String[][]`类型的维数组，将被记录为`[[Ljava/lang/String;`

**方法表集合**

**stack**: 最大操作数栈，JVM运行时会根据这个值来分配栈帧(Frame)中的操作栈深度,此处为1

**locals**: 局部变量所需的存储空间，单位为Slot, Slot是虚拟机为局部变量分配内存时所使用的最小单位，为4个字节大小。方法参数(包括实例方法中的隐藏参数this)，显示异常处理器的参数(try catch中的catch块所定义的异常)，方法体中定义的局部变量都需要使用局部变量表来存放。值得一提的是，locals的大小并不一定等于所有局部变量所占的Slot之和，因为局部变量中的Slot是可以重用的。

**args_size**: 方法参数的个数，这里是1，因为每个实例方法都会有一个隐藏参数this

**attribute_info**: 方法体内容，0,1,4为字节码"行号"，该段代码的意思是将第一个引用类型本地变量推送至栈顶，然后执行该类型的实例方法，也就是常量池存放的第一个变量，也就是注释里的`java/lang/Object."":()V`, 然后执行返回语句，结束方法。

**LineNumberTable**: 该属性的作用是描述源码行号与字节码行号(字节码偏移量)之间的对应关系。可以使用 -g:none 或-g:lines选项来取消或要求生成这项信息，如果选择不生成LineNumberTable，当程序运行异常时将无法获取到发生异常的源码行号，也无法按照源码的行数来调试程序。

**LocalVariableTable**: 该属性的作用是描述帧栈中局部变量与源码中定义的变量之间的关系。可以使用 -g:none 或 -g:vars来取消或生成这项信息，如果没有生成这项信息，那么当别人引用这个方法时，将无法获取到参数名称，取而代之的是arg0, arg1这样的占位符。 start 表示该局部变量在哪一行开始可见，length表示可见行数，Slot代表所在帧栈位置，Name是变量名称，然后是类型签名。

### JVM 基础 - 字节码的增强技术

#### 字节码增强技术

字节码增强技术就是一类对现有字节码进行**修改**或者**动态生成**全新字节码文件的技术。

![img](https://www.pdai.tech/images/jvm/java-class-enhancer-1.png)

##### ASM

对于需要手动操纵字节码的需求，可以使用ASM，它可以**直接生产** .class字节码文件，也可以在类被加载入JVM之前**动态修改**类行为。ASM的应用场景有AOP（Cglib就是基于ASM）、热部署、修改其他jar包中的类等。

![img](https://www.pdai.tech/images/jvm/java-class-enhancer-2.png)

字节码文件的结构是由JVM固定的，所以很适合利用**访问者模式**对字节码文件进行修改。

**ClassReader** 类相当于访问者模式中的 Element 元素。它将字节数组或 class 文件读入内存中，并以树的数据结构表示。该类定义了一个 accept 方法用来和 visitor 交互。

**ClassVisitor** 相当于抽象访问者接口。ClassReader 对象创建之后，需要调用 accept () 方法，传入一个 ClassVisitor 对象。在 ClassReader 的不同时期会调用 ClassVisitor 对象中不同的 visit () 方法，从而实现对字节码的修改。

**ClassWriter** 是 ClassVisitor 的是实现类，它负责将修改后的字节码输出为字节数组。

访问者模式定义：**把不变的固定起来，变化的开放出去。**

访问者模式的使用场景：**某些较为稳定的东西（数据结构或算法），不想直接被改变但又想扩展功能，这时候适合用访问者模式。**

说到对于访问者模式使用场景的定义，我们会觉得**模板方法模式**与这个使用场景的定义很像。但它们还是有些许差别的。**访问者模式的变化与非变化（即访问者与被访问者）之间，它们只是简单的包含关系，而模板方法模式的变化与非变化则是继承关系。** 但它们也确实有类似的地方，即都是封装了固定不变的东西，开放了变动的东西。



###### ASM API

XML解析方式分为两种：dom和sax

**dom**：(Document Object Model, 即文档对象模型) 

   1.将整个XML使用类似树的结构保存在内存中，再对其进行操作。
   2.是 W3C 组织推荐的处理 XML 的一种方式。
   3.需要等到XML**完全加载**进内存才可以进行操作
   4.**耗费内存**，当解析超大的XML时慎用。
   5.可以方便的对xml进行**增删改查**操作

**sax**： (Simple API for XML) 不是官方标准，但它是 XML 社区事实上的标准，几乎所有的 XML 解析器都支持它。

   1.逐行扫描XML文档，当遇到标签时触发解析处理器，采用事件处理的方式解析xml
    2.(Simple API for XML) 不是官方标准，但它是 XML 社区事实上的标准，几乎所有的 XML 解析器都支持它。
   3.在读取文档的**同时**即可对xml进行处理，不必等到文档加载结束，相对快捷
   4.不需要加载进内存，因此**不存在占用内存**的问题，可以解析超大XML
   5.只能用来读取XML中数据，**无法进行增删改**

**核心API**

**ASM Core API可以类比解析XML文件中的SAX方式，不需要把这个类的整个结构读取进来，就可以用流式的方法来处理字节码文件。好处是非常节约内存，但是编程难度较大。**然而出于性能考虑，一般情况下编程都使用Core API。在Core API中有以下几个关键类：

- ClassReader：用于读取已经编译好的.class文件。
- ClassWriter：用于重新构建编译后的类，如修改类名、属性以及方法，也可以生成新的类的字节码文件。
- 各种Visitor类：如上所述，CoreAPI根据字节码从上到下依次处理，对于字节码文件中不同的区域有不同的Visitor，比如用于访问方法的MethodVisitor、用于访问类变量的FieldVisitor、用于访问注解的AnnotationVisitor等。为了实现AOP，重点要使用的是MethodVisitor。

**树形API**

**ASM Tree API可以类比解析XML文件中的DOM方式，把整个类的结构读取到内存中，缺点是消耗内存多，但是编程比较简单。**TreeApi不同于CoreAPI，TreeAPI通过各种Node类来映射字节码的各个区域，类比DOM节点，就可以很好地理解这种编程方式。

##### Javassist

ASM是在**指令**层次上操作字节码的，

利用Javassist实现字节码增强时，直接使用**java编码的形式**，而不需要了解虚拟机指令，就能动态改变类的结构或者动态生成类。其中最重要的是ClassPool、CtClass、CtMethod、CtField这四个类：

- ClassPool：从开发视角来看，ClassPool是一张保存CtClass信息的HashTable，key为类名，value为类名对应的CtClass对象。当我们需要对某个类进行修改时，就是通过pool.getCtClass(“className”)方法从pool中获取到相应的CtClass。

- CtClass（compile-time class）：编译时类信息，它是一个class文件在代码中的抽象表现形式，可以通过一个类的全限定名来获取一个CtClass对象，用来表示这个类文件。
- CtMethod、CtField：这两个比较好理解，对应的是类中的方法和属性。

```
import com.meituan.mtrace.agent.javassist.*;

public class JavassistTest {
    public static void main(String[] args) throws NotFoundException, CannotCompileException, IllegalAccessException, InstantiationException, IOException {
        ClassPool cp = ClassPool.getDefault();
        CtClass cc = cp.get("meituan.bytecode.javassist.Base");
        CtMethod m = cc.getDeclaredMethod("process");
        m.insertBefore("{ System.out.println(\"start\"); }");
        m.insertAfter("{ System.out.println(\"end\"); }");
        Class c = cc.toClass();
        cc.writeFile("/Users/zen/projects");
        Base h = (Base)c.newInstance();
        h.process();
    }
}
```

#### 运行时类的重载

在上文中我们避重就轻将ASM实现AOP的过程分为了两个main方法：第一个是利用MyClassVisitor对已编译好的class文件进行修改，第二个是new对象并调用。

这期间并不涉及到JVM运行时对类的**重加载**，而是在第一个main方法中，通过ASM对已编译类的**字节码**进行**替换**，在第二个main方法中，直接**使用已替换**好的**新类**信息。另外在Javassist的实现中，我们也只加载了一次Base类，也不涉及到运行时重加载类。

**JVM是不允许在运行时动态重新加载一个类的**

我们期望的效果是：在一个持续运行并已经加载了所有类的JVM中，还能利用字节码增强技术对其中的类行为做替换并重新加载。所以需要借助的Java类库：Instrument

##### Instrument

instrument是JVM提供的一个**可以修改已加载类**的类库，专门为Java语言编写的插桩服务提供支持。

它需要依赖JVMTI的Attach API机制实现

在JDK 1.6以前，instrument只能在JVM刚启动开始加载类时生效，而在JDK 1.6之后，instrument支持了在运行时对类定义的修改。

要使用instrument的类修改功能，我们需要实现它提供的ClassFileTransformer接口，定义一个类文件转换器。接口中的transform()方法会在类文件被加载时调用，而在transform方法里，我们可以利用上文中的**ASM或Javassist对传入的字节码进行改写或替换**，生成新的字节码数组后返回。

我们定义一个实现了ClassFileTransformer接口的类TestTransformer，依然在其中利用Javassist对Base类中的process()方法进行增强，在前后分别打印“start”和“end”，代码如下：

```java
import java.lang.instrument.ClassFileTransformer;

public class TestTransformer implements ClassFileTransformer {
    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) {
        System.out.println("Transforming " + className);
        try {
            ClassPool cp = ClassPool.getDefault();
            CtClass cc = cp.get("meituan.bytecode.jvmti.Base");
            CtMethod m = cc.getDeclaredMethod("process");
            m.insertBefore("{ System.out.println(\"start\"); }");
            m.insertAfter("{ System.out.println(\"end\"); }");
            return cc.toBytecode();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

现在有了Transformer，那么它要如何**注入**到正在运行的JVM呢？还需要定义一个Agent，借助Agent的能力将Instrument注入到JVM中。我们将在下一小节介绍Agent，现在要介绍的是Agent中用到的另一个类Instrumentation。在JDK 1.6之后，Instrumentation可以做启动后的Instrument、本地代码（Native Code）的Instrument，以及动态改变Classpath等等。我们可以向Instrumentation中添加上文中定义的Transformer，并指定要被重加载的类，代码如下所示。这样，**当Agent被Attach到一个JVM中时，就会执行类字节码替换并重载入JVM的操作。**

```java
import java.lang.instrument.Instrumentation;

public class TestAgent {
    public static void agentmain(String args, Instrumentation inst) {
        //指定我们自己定义的Transformer，在其中利用Javassist做字节码替换
        inst.addTransformer(new TestTransformer(), true);
        try {
            //重定义类并载入新的字节码
            inst.retransformClasses(Base.class);
            System.out.println("Agent Load Done.");
        } catch (Exception e) {
            System.out.println("agent load failed!");
        }
    }
}
```

##### JVMTI & Agent & Attach API

上一小节中，我们给出了Agent类的代码，追根溯源需要先介绍JPDA（Java Platform Debugger Architecture）。如果JVM启动时开启了JPDA，那么类是允许被重新加载的。在这种情况下，已被加载的旧版本类信息可以被卸载，然后重新加载新版本的类。正如JDPA名称中的Debugger，JDPA其实是一套用于调试Java程序的标准，任何JDK都必须实现该标准。

JPDA定义了一整套完整的体系，它将调试体系分为三部分，并规定了三者之间的通信接口。三部分由低到高分别是Java 虚拟机工具接口（JVMTI），Java 调试协议（JDWP）以及 Java 调试接口（JDI），三者之间的关系如下图所示：

![img](https://www.pdai.tech/images/jvm/java-class-enhancer-6.png)

JVM TI（JVM TOOL INTERFACE，JVM工具接口）是JVM提供的一套对JVM进行操作的工具接口。通过JVMTI，可以实现对JVM的多种操作，它通过接口注册各种事件勾子，在JVM事件触发时，同时触发预定义的勾子，以实现对各个JVM事件的响应，事件包括类文件加载、异常产生与捕获、线程启动和结束、进入和退出临界区、成员变量修改、GC开始和结束、方法调用进入和退出、临界区竞争与等待、VM启动与退出等等。

而Agent就是JVMTI的一种实现，Agent有两种启动方式，一是随Java进程启动而启动，经常见到的java -agentlib就是这种方式；二是运行时载入，通过attach API，将模块（jar包）动态地Attach到指定进程id的Java进程内。

**Attach API 的作用是提供JVM进程间通信的能力**，比如说我们为了让另外一个JVM进程把线上服务的线程Dump出来，会运行jstack或jmap的进程，并传递pid的参数，告诉它要对哪个进程进行线程Dump，这就是Attach API做的事情。

在下面，我们将通过Attach API的loadAgent()方法，将打包好的Agent jar包动态Attach到目标JVM上。具体实现起来的步骤如下：

- 定义Agent，并在其中实现AgentMain方法，如上一小节中定义的代码块7中的TestAgent类；
- 然后将TestAgent类打成一个包含MANIFEST.MF的jar包，其中MANIFEST.MF文件中将Agent-Class属性指定为TestAgent的全限定名，如下图所示；

![img](https://www.pdai.tech/images/jvm/java-class-enhancer-7.png)

- 最后利用Attach API，将我们打包好的jar包Attach到指定的JVM pid上，代码如下：

```java
import com.sun.tools.attach.VirtualMachine;

public class Attacher {
    public static void main(String[] args) throws AttachNotSupportedException, IOException, AgentLoadException, AgentInitializationException {
        // 传入目标 JVM pid
        VirtualMachine vm = VirtualMachine.attach("39333");
        vm.loadAgent("/Users/zen/operation_server_jar/operation-server.jar");
    }
}
```

- 由于在MANIFEST.MF中指定了Agent-Class，所以在Attach后，目标JVM在运行时会走到TestAgent类中定义的agentmain()方法，而在这个方法中，我们利用Instrumentation，将指定类的字节码通过定义的类转化器TestTransformer做了Base类的字节码替换（通过javassist），并完成了类的重新加载。由此，我们达成了“在JVM运行时，改变类的字节码并重新载入类信息”的目的。

##### 使用场景

至此，字节码增强技术的可使用范围就不再局限于JVM加载类前了。通过上述几个类库，我们可以在运行时对JVM中的类进行修改并重载了。通过这种手段，可以做的事情就变得很多了：

- 热部署：不部署服务而对线上服务做修改，可以做打点、增加日志等操作。
- Mock：测试时候对某些服务做Mock。
- 性能诊断工具：比如bTrace就是利用Instrument，实现无侵入地跟踪一个正在运行的JVM，监控到类和方法级别的状态信息。

#### 总结

字节码增强技术相当于是一把打开运行时JVM的钥匙，利用它可以**动态**地对**运行**中的程序做修改，也可以**跟踪**JVM运行中程序的状态。此外，我们平时使用的动态代理、AOP也与字节码增强密切相关，它们实质上还是利用各种手段生成符合规范的字节码文件。综上所述，掌握字节码增强后可以高效地定位并快速修复一些棘手的问题（如线上性能问题、方法出现不可控的出入参需要紧急加日志等问题），也可以在开发中减少冗余代码，大大提高开发效率。

### JVM 基础 - Java 类加载机制

#### 类的生命周期

其中类加载的过程包括了`加载`、`验证`、`准备`、`解析`、`初始化`五个阶段。`加载`、`验证`、`准备`和`初始化`这四个阶段发生的顺序是确定的，*而`解析`阶段则不一定*.

![img](https://www.pdai.tech/images/jvm/java_jvm_classload_2.png)

##### 类的加载: 查找并加载类的二进制数据

加载时类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情:

- 通过一个类的全限定名来获取其定义的二进制字节流。
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
- 在Java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口。

![img](https://www.pdai.tech/images/jvm/java_jvm_classload_1.png)

相对于类加载的其他阶段而言，*加载阶段(准确地说，是加载阶段获取类的二进制字节流的动作)是**可控性最强**的阶段*，因为开发人员既可以使用**系统**提供的类加载器来完成加载，也可以**自定义**自己的类加载器来完成加载。

类加载器并不需要等到某个类被“首次主动使用”时再加载它，JVM规范允许类加载器在预料某个类**将要被使用时就预先加载**它，如果在预先加载的过程中遇到了.class文件缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误(LinkageError错误)如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误。

加载.class文件的方式

- 从本地系统中直接加载
- 通过网络下载.class文件
- 从zip，jar等归档文件中加载.class文件
- 从专有数据库中提取.class文件
- 将Java源文件动态编译为.class文件

类加载有三种方式:

1、命令行启动应用时候由JVM初始化加载

2、通过Class.forName()方法动态加载

3、通过ClassLoader.loadClass()方法动态加载

##### 连接

**验证: 确保被加载的类的正确性**

为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

验证阶段大致会完成4个阶段的检验动作:

- `文件格式验证`: 验证字节流是否符合Class文件格式的规范；例如: 是否以`0xCAFEBABE`开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。
- `元数据验证`: 对字节码描述的信息进行语义分析(注意: 对比`javac`编译阶段的语义分析)，以保证其描述的信息符合Java语言规范的要求；例如: 这个类是否有父类，除了`java.lang.Object`之外。
- `字节码验证`: 通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
- `符号引用验证`: 确保解析动作能正确执行。

验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，*如果所引用的类经过反复验证，那么可以考虑采用`-Xverifynone`参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。*

**准备: 为类的静态变量分配内存，并将其初始化为默认值**

- 这时候进行**内存分配的仅包括类变量(`static`)**，而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中。

这里所设置的初始值通常情况下是数据类型默认的零值(如`0`、`0L`、`null`、`false`等)，而不是被在Java代码中被显式地赋予的值。

引用数据类型`reference`，系统都会为其赋予默认的零值，即`null`

假设一个类变量的定义为: `public static int value = 3`；那么变量value在准备阶段过后的初始值为`0`。

假设类变量value被定义为: `public static final int value = 3`；那么变量value在准备阶段过后的初始值为3。

**解析: 把类中的符号引用转换为直接引用**

解析阶段是虚拟机将**常量池**内的符号引用替换为直接引用的过程，解析动作主要针对`类`或`接口`、`字段`、`类方法`、`接口方法`、`方法类型`、`方法句柄`和`调用点`限定符7类符号引用进行。

`符号引用`就是一组符号来描述目标，可以是任何字面量。

`直接引用`就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

##### 初始化

初始化，为类的**静态变量**赋予正确的初始值，

JVM负责对**类**进行初始化，

主要对**类变量**进行初始化。

**JVM初始化步骤**

- 假如这个类还没有被加载和连接，则程序先加载并连接该类
- 假如该类的直接父类还没有被初始化，则先初始化其直接父类
- 假如类中有初始化语句，则系统依次执行这些初始化语句

**类初始化时机**: 只有当对类的主动使用的时候才会导致类的初始化，类的主动使用包括以下六种:

- 创建类的实例，也就是new的方式
- 访问某个类或接口的静态变量，或者对该静态变量赋值
- 调用类的静态方法
- 反射(如Class.forName("com.pdai.jvm.Test"))
- 初始化某个类的子类，则其父类也会被初始化
- Java虚拟机启动时被标明为启动类的类(Java Test)，直接使用java.exe命令来运行某个主类

##### 使用

类访问方法区内的数据结构的接口， 对象是Heap区的数据。

##### 卸载

**Java虚拟机将结束生命周期的几种情况**

- 执行了System.exit()方法
- 程序正常执行结束
- 程序在执行过程中遇到了异常或错误而异常终止
- 由于操作系统出现错误而导致Java虚拟机进程终止

#### 类加载器， JVM类加载机制

**类加载器的层次**

![img](https://www.pdai.tech/images/jvm/java_jvm_classload_3.png)

**站在Java开发人员的角度来看，类加载器可以大致划分为以下三类** :

`启动类加载器`: Bootstrap ClassLoader，负责加载存放在**JDK\jre\lib**(JDK代表JDK的安装目录，下同)下，或被-Xbootclasspath参数指定的路径中的，并且能被虚拟机识别的类库(如rt.jar，所有的java.*开头的类均被Bootstrap ClassLoader加载)。**启动类加载器是无法被Java程序直接引用的**。

`扩展类加载器`: Extension ClassLoader，该加载器由`sun.misc.Launcher$ExtClassLoader`实现，它负责加载**JDK\jre\lib\ext**目录中，或者由java.ext.dirs系统变量指定的路径中的所有类库(如javax.*开头的类)，开发者可以**直接使用扩展类加载器**。

`应用程序类加载器`: Application ClassLoader，该类加载器由`sun.misc.Launcher$AppClassLoader`来实现，它负责加载**用户类路径(**ClassPath)所指定的类，开发者可以**直接使用**该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

应用程序都是由这三种类加载器互相配合进行加载的，如果有必要，我们还可以加入**自定义**的类加载器。



> **Class.forName()和ClassLoader.loadClass()区别?**

- Class.forName(): 将类的.class文件加载到jvm中之外，还会对类进行解释，**执行类中的static块**；
- ClassLoader.loadClass(): 只干一件事情，就是将.class文件加载到jvm中，**不会执行static中的内容**,只有在newInstance才会去执行static块。
- Class.forName(name, initialize, loader)带参函数也可控制是否加载static块。并且只有调用了newInstance()方法采用调用构造函数，创建类的对象 。

**JVM类加载机制**

- `全盘负责`，当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入
- `父类委托`，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类
- `缓存机制`，缓存机制将会保证**所有加载过的Class都会被缓存**，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效
- `双亲委派机制`, 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

**双亲委派机制过程？**

1. 当AppClassLoader加载一个class时，它首先不会自己去尝试加载这个类，而是把**类加载请求**委派给父类加载器ExtClassLoader去完成。
2. 当ExtClassLoader加载一个class时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给BootStrapClassLoader去完成。
3. 如果BootStrapClassLoader加载失败(例如在$JAVA_HOME/jre/lib里未查找到该class)，会使用ExtClassLoader来尝试加载；
4. 若ExtClassLoader也加载失败，则会使用AppClassLoader来加载，如果AppClassLoader也加载失败，则会报出异常ClassNotFoundException。

**双亲委派优势**

- 系统类防止内存中出现多份同样的字节码
- 保证Java程序安全稳定运行

### JVM 基础 - JVM 内存结构

#### 运行时数据区

内存是非常重要的系统资源，是硬盘和 CPU 的中间仓库及桥梁，承载着操作系统和应用程序的实时运行。JVM 内存布局规定了 Java 在运行过程中内存申请、分配、管理的策略，保证了 JVM 的高效稳定运行。不同的 JVM 对于内存的划分方式和管理机制存在着部分差异。

下图是 JVM 整体架构，中间部分就是 Java 虚拟机定义的各种运行时数据区域。

![jvm-framework](https://www.pdai.tech/images/jvm/jvm/0082zybply1gc6fz21n8kj30u00wpn5v.jpg)

- **线程私有**：程序计数器、虚拟机栈、本地方法栈
- **线程共享**：堆、方法区, 堆外内存（Java7的永久代或JDK8的元空间、代码缓存）

#### 一、程序计数器

程序计数器是一块**较小的内存空间**，可以看作是当前线程所执行的字节码的**行号指示器**。也是运行速度最快的存储区域

PC 寄存器用来存储指向下一条指令的地址，即将要执行的指令代码。由执行引擎读取下一条指令。

**它是唯一一个在 JVM 规范中没有规定任何 `OutOfMemoryError` 情况的区域**

- **使用PC寄存器存储字节码指令地址有什么用呢？为什么使用PC寄存器记录当前线程的执行地址呢？**

因为CPU需要**不停的切换各个线程**，这时候切换回来以后，就得知道接着从哪开始继续执行。JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令。

- **PC寄存器为什么会被设定为线程私有的？**

多线程在一个特定的时间段内只会执行其中某一个线程方法，CPU会不停的做任务切换，这样必然会**导致经常中断或恢复**。为了能够**准确的记录**各个线程正在执行的当前字节码**指令地址**，所以为每个线程都分配了一个PC寄存器，**每个线程都独立计算**，不会互相影响。

#### 二、虚拟机栈

**作用**：主管 Java 程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。

**特点**：

- 栈是一种快速有效的分配存储方式，访问速度**仅次于**程序计数器
- JVM 直接对虚拟机栈的操作只有两个：每个方法执行，伴随着**入栈**（进栈/压栈），方法执行结束**出栈**
- **栈不存在垃圾回收问题**

**栈中可能出现的异常**：

Java 虚拟机规范允许 **Java虚拟机栈的大小是动态的或者是固定不变的**

- 如果采用**固定大小**的 Java 虚拟机栈，那每个线程的 Java 虚拟机栈容量可以在线程创建的时候独立选定。如果线程请求分配的栈容量超过 Java 虚拟机栈允许的最大容量，Java 虚拟机将会抛出一个 **StackOverflowError** 异常
- 如果 Java 虚拟机栈可以**动态扩展**，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的虚拟机栈，那 Java 虚拟机将会抛出一个**OutOfMemoryError**异常

可以通过参数`-Xss`来设置线程的**最大栈空间**，栈的大小直接决定了函数调用的**最大可达深度**。

**栈中存储什么？**

- 每个线程都有自己的栈，栈中的数据都是以**栈帧（Stack Frame）的格式存在**
- 在这个线程上正在执行的**每个方法**都各自有对应的**一个栈帧**
- 栈帧是一个**内存区块**，是一个数据集，维系着方法执行过程中的各种数据信息

**基本单位**:栈帧

**栈运行原理**

- JVM 直接对 Java 栈的操作只有两个，对栈帧的**压栈**和**出栈**，遵循“先进后出/后进先出”原则
- 在一条活动线程中，一个时间点上，只会有一个活动的栈帧。即只有当前正在执行的方法的栈帧（**栈顶栈帧**）是有效的，这个栈帧被称为**当前栈帧**（Current Frame），与当前栈帧对应的方法就是**当前方法**（Current Method），定义这个方法的类就是**当前类**（Current Class）
- 执行引擎运行的所有字节码指令只针对**当前栈帧**进行操作
- 如果在该方法中调用了其他方法，对应的新的栈帧会被创建出来，放在栈的**顶端**，称为新的**当前栈帧**
- **不同线程**中所包含的栈帧是**不允许存在相互引用的**，即不可能在一个栈帧中引用另外一个线程的栈帧
- 如果当前方法调用了其他方法，方法返回之际，当前栈帧会传回此方法的执行结果给前一个栈帧，接着，虚拟机会丢弃当前栈帧，使得前一个栈帧重新成为当前栈帧
- Java 方法有两种返回函数的方式，**一种是正常的函数返回，使用 return 指令，另一种是抛出异常，不管用哪种方式，都会导致栈帧被弹出**

IDEA 在 debug 时候，可以在 debug 窗口看到 Frames 中各种方法的压栈和出栈情况

**栈帧内部结构：**

每个**栈帧**（Stack Frame）中存储着：

- 局部变量表（Local Variables）
- 操作数栈（Operand Stack）(或称为表达式栈)
- 动态链接（Dynamic Linking）：指向运行时常量池的方法引用
- 方法返回地址（Return Address）：方法正常退出或异常退出的地址
- 一些附加信息

![jvm-stack-frame](https://www.pdai.tech/images/jvm/jvm/0082zybply1gc8tjehg8bj318m0lbtbu.jpg)

**一、局部变量表**：

- 是一组变量值**存储空间**，**主要用于存储方法参数和定义在方法体内的局部变量**，包括**基本数据类型**、**对象引用**（reference类型，它并不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此相关的位置）和 **returnAddress** 类型（指向了一条字节码指令的地址，已被异常表取代）;
- **局部变量表所需要的容量大小是编译期确定下来的**;
- 方法嵌套调用的次数由栈的大小决定。一般来说，**栈越大，方法嵌套调用次数越多**。
- **局部变量表中的变量只在当前方法调用中有效**。

**槽 Slot**

- 局部变量表最基本的**存储单元**是 Slot（变量槽）
- 在局部变量表中，**32 位**以内的类型只占用**一个 Slot**(包括returnAddress类型)，**64 位**的类型（long和double）占用**两个连续**的 Slot
- - byte、short、char 在存储前被转换为int，boolean也被转换为int，0 表示 false，非 0 表示 true
  - long 和 double 则占据两个 Slot
- **栈帧中的局部变量表中的槽位是可以重用的**
- 如果当前帧是由构造方法或实例方法创建的，那么该对象引用 **this** 将会存放在 index 为 **0** 的 Slot 处，其余的参数按照参数表顺序继续排列（这里就引出一个问题：静态方法中为什么不可以引用 this，就是因为this 变量不存在于当前方法的局部变量表中）
- 局部变量表中的变量也是重要的**垃圾回收根节点**，只要被局部变量表中直接或间接引用的对象都不会被回收

**二、操作数栈:**

- **操作数栈，在方法执行过程中，根据字节码指令，往操作数栈中写入数据或提取数据，即入栈（push）、出栈（pop）**

- 主要用于保存计算过程的**中间结果**，同时作为计算过程中变量**临时的存储空间**
- 栈中的任何一个元素都可以是任意的 Java 数据类型
  - 32bit 的类型占用一个栈单位深度
  - 64bit 的类型占用两个栈单位深度
- **如果被调用的方法带有返回值的话，其返回值将会被压入当前栈帧的操作数栈中**，并更新 PC 寄存器中下一条需要执行的字节码指令
- **Java虚拟机的解释引擎是基于栈的执行引擎**，其中的栈指的就是操作数栈

栈顶缓存（Top-of-stack-Cashing）：由于**操作数**是**存储在内存**中的，因此频繁的执行内存读/写操作必然会影响执行速度。为了解决这个问题，HotSpot JVM 设计者们提出了栈顶缓存技术，**将栈顶元素全部缓存在物理 CPU 的寄存器中，以此降低对内存的读/写次数，提升执行引擎的执行效率**

**三、动态链接（指向运行时常量池的方法引用）:**

- **每一个栈帧内部都包含一个指向运行时常量池中该栈帧所属方法的引用**。包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接(Dynamic Linking)。
- 动态链接的作用就是为了将这些**符号引用转换**为调用方法的**直接引用**
- 在 JVM 中，将符号引用转换为调用方法的直接引用与方法的**绑定机制**有关
  - **静态链接**：当一个字节码文件被装载进 JVM 内部时，如果被调用的**目标方法在编译期可知**，且运行期保持不变时。这种情况下将调用方法的符号引用转换为直接引用的过程称之为静态链接
  - **动态链接**：如果被调用的方法在**编译期无法被确定**下来，也就是说，只能在程序运行期将调用方法的符号引用转换为直接引用，由于这种引用转换过程具备动态性，因此也就被称之为动态链接

对应的方法的绑定机制为：早期绑定（Early Binding）和晚期绑定（Late Binding）。**绑定是一个字段、方法或者类在符号引用被替换为直接引用的过程，这仅仅发生一次**。

- 早期绑定：**早期绑定就是指被调用的目标方法如果在编译期可知，且运行期保持不变时**，即可将这个方法与所属的类型进行绑定，这样一来，由于明确了被调用的目标方法究竟是哪一个，因此也就可以使用静态链接的方式将符号引用转换为直接引用。

  晚期绑定：如果被调用的方法在**编译期无法被确定**下来，只能够在程序运行期根据实际的类型绑定相关的方法，这种绑定方式就被称为晚期绑定。

**虚方法和非虚方法**

- 如果方法在**编译期**就确定了具体的调用版本，这个版本在运行时是不可变的。这样的方法称为非虚方法，比如静态方法、私有方法、final 方法、实例构造器、父类方法都是非虚方法
- 其他方法称为虚方法（在Java中虛方法是指在编译阶段和类加载阶段都不能确定方法的调用入口地址，在运行阶段才能确定的方法，即**可能被重写的方法**。）

**四、方法返回地址（return address）：**

用来存放调用该方法的 **PC 寄存器**的值。

**五、附加信息：**

栈帧中还允许携带与 Java 虚拟机实现相关的一些附加信息。例如，对**程序调试**提供支持的信息，但这些信息取决于具体的虚拟机实现。

#### 三、本地方法栈

**本地方法接口**

一个 Native Method 就是一个 Java 调用**非 Java 代码**的接口。

有时 Java 应用需要与 **Java 外面的环境交互**，这就是本地方法存在的原因

**本地方法栈（Native Method Stack）**

- Java 虚拟机栈用于管理 **Java 方法**的调用，而本地方法栈用于管理**本地方法**的调用
- 本地方法栈也是线程**私有**的
- 允许线程**固定或者可动态扩展**的内存大小
- 本地方法是使用 **C 语言**实现的
- 它的具体做法是 `Native Method Stack` 中登记 native 方法，在 `Execution Engine` 执行时加载本地方法库当某个线程调用一个本地方法时，它就进入了一个全新的并且不再受虚拟机限制的世界。它和虚拟机拥有同样的权限。
- 本地方法可以通过本地方法接口来访问虚拟机内部的运行时数据区，它甚至可以直接使用本地处理器中的寄存器，直接从本地内存的堆中分配任意数量的内存
- 并不是所有 JVM 都支持本地方法。
- 在 Hotspot JVM 中，直接将本地方法栈和虚拟机栈合二为一

**栈是运行时的单位，而堆是存储的单位**。

#### 四、堆内存

**一、内存划分**

对于大多数应用，Java 堆是 Java 虚拟机管理的**内存中最大**的一块，被所有线程**共享**。此内存区域的唯一目的就是**存放对象实例**，几乎所有的对象实例以及数据都在这里**分配内存**。

内存划分（分代的唯一理由就是**优化 GC** 性能）：新生代，老年代，元空间

新生代（年轻代）：新对象和没达到一定年龄的对象都在新生代

老年代（养老区）：被长时间使用的对象，老年代的内存空间应该要比年轻代更大（默认是 15 次）

元空间（JDK1.8 之前叫永久代）：像一些方法中的操作临时对象等，JDK1.8 之前是占用 JVM 内存，JDK1.8 之后直接使用物理内存

![JDK7](https://www.pdai.tech/images/jvm/jvm/00831rSTly1gdbr7ek6pfj30ci0560t4.jpg)



新生代：垃圾收集称为 **Minor GC**。年轻一代被分为三个部分——伊甸园（**Eden Memory**）和两个幸存区（**Survivor Memory**，被称为from/to或s0/s1），默认比例是**8:1:1**

老年代：老年代垃圾收集称为 主GC（**Major GC**），大对象直接进入老年代（大对象是指需要大量连续内存空间的对象）。这样做的目的是**避免**在 Eden 区和两个Survivor 区之间发生**大量的内存拷贝**

元空间：不管是 JDK8 之前的永久代，还是 JDK8 及以后的元空间，都可以看作是 Java 虚拟机规范中**方法区**的实现。

**二、设置堆内存大小和 OOM**

- `-Xms` 用来表示堆的起始内存，等价于 `-XX:InitialHeapSize`

- `-Xmx` 用来表示堆的最大内存，等价于 `-XX:MaxHeapSize`

- 如果堆的内存大小超过 `-Xmx` 设定的最大内存， 就会抛出 `OutOfMemoryError` 异常。

  我们通常会将 `-Xmx` 和 `-Xms` 两个参数配置为相同的值，其目的是为了能够在垃圾回收机制清理完堆区后不再需要**重新分隔计算堆的大小**，从而提高性能

  - 默认情况下，初始堆内存大小为：**电脑内存大小/64**
  - 默认情况下，最大堆内存大小为：**电脑内存大小/4**

- –XX:NewRatio 新生代，老年代比例  ，默认情况下新生代和老年代的比例是 1:2

- -XX:SurvivorRatio 新生代比例 **Eden**:**From Survivor**:**To Survivor**，默认是**8:1:1**

- -XX:+UseAdaptiveSizePolicy   JVM 会动态调整 JVM 堆中各个区域的大小以及进入老年代的年龄 此时 `–XX:NewRatio` 和 `-XX:SurvivorRatio` 将会失效，而 JDK 8 是默认开启`-XX:+UseAdaptiveSizePolicy`

- -XX:PetenureSizeThreshold    如果分配的对象超过了`-XX:PetenureSizeThreshold`，对象会**直接被分配到老年代**

**三、GC 垃圾回收简介**

**Minor GC、Major GC、Full GC**

针对 HotSpot VM 的实现，它里面的 GC 按照回收区域又分为两大类：部分收集（Partial GC），整堆收集（Full GC）

- 部分收集：不是完整收集整个 Java 堆的垃圾收集。其中又分为： 
  - 新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集
  - 老年代收集（Major GC/Old GC）：只是老年代的垃圾收集 
    - 目前，只有 CMS GC 会有单独收集老年代的行为
    - 很多时候 Major GC 会和 Full GC 混合使用，需要具体分辨是老年代回收还是整堆回收
  - 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集 
    - 目前只有 G1 GC 会有这种行为
- 整堆收集（Full GC）：收集整个 Java 堆和方法区的垃圾

**四、TLAB（Thread Local Allocation Buffer）**

- 多线程同时分配内存时，使用 TLAB 可以避免一系列的**非线程安全**问题，同时还能提升内存分配的吞吐量，因此我们可以将这种内存分配方式称为**快速分配策略**

**五、堆是分配对象存储的唯一选择吗**

> 随着 JIT 编译期的发展和**逃逸分析**技术的逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得**不那么“绝对”**了。 ——《深入理解 Java 虚拟机》

**逃逸分析**

**这是一种可以有效减少 Java 程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法**。通过逃逸分析，Java Hotspot 编译器能够分析出一个新的对象的**引用的使用范围**从而**决定是否要将这个对象分配到堆上。**

逃逸分析的基本行为就是**分析对象动态作用域**：

- 当一个对象在方法中被定义后，对象只在方法**内部**使用，则认为没有发生逃逸。
- 当一个对象在方法中被定义后，它被**外部方法所引用**，则认为发生逃逸。例如作为调用参数传递到其他地方中，称为方法逃逸。

使用逃逸分析，编译器可以对代码做优化：

- **栈上分配**：将**堆分配转化为栈分配**。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配
- **同步省略**：如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步
- **分离对象或标量替换**：有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而**存储在 CPU 寄存器**

**代码优化之标量替换**

**标量**（Scalar）是指一个无法再分解成更小的数据的数据。Java 中的原始数据类型就是标量。

相对的，那些的还可以分解的数据叫做**聚合量**（Aggregate），Java 中的对象就是聚合量，因为其还可以分解成其他聚合量和标量。

在 JIT 阶段，通过逃逸分析确定该对象不会被外部访问，并且对象可以被进一步分解时，JVM 不会创建该对象，而会将该对象成员变量分解若干个被这个方法使用的成员变量所代替。这些代替的成员变量在栈帧或寄存器上分配空间。这个过程就是**标量替换**。

通过 `-XX:+EliminateAllocations` 可以开启标量替换，`-XX:+PrintEliminateAllocations` 查看标量替换情况。

**总结：**

关于逃逸分析的论文在1999年就已经发表了，但直到JDK 1.6才有实现，而且这项技术到如今也并不是十分成熟的。

**其根本原因就是无法保证逃逸分析的性能消耗一定能高于他的消耗。虽然经过逃逸分析可以做标量替换、栈上分配、和锁消除。但是逃逸分析自身也是需要进行一系列复杂的分析的，这其实也是一个相对耗时的过程。**

#### 五、方法区

**运行时常量池**（Runtime Constant Pool）是方法区的一部分。

字符串常量池在堆中。

元数据区大小可以使用参数 `-XX:MetaspaceSize` 和 `-XX:MaxMetaspaceSize` 指定

默认值依赖于平台。Windows 下，`-XX:MetaspaceSize` 是 21M，`-XX:MaxMetaspacaSize` 的值是 -1，即没有限制

方法区的垃圾收集主要回收两部分内容：**常量池中废弃的常量和不再使用的类型**。

先来说说方法区内常量池之中主要存放的两大类常量：字面量和符号引用。

**方法区内部结构**

方法区用于存储已被虚拟机加载的**类型信息、常量、静态变量、即时编译器编译后的代码缓存**等。

**类型信息**

对每个加载的类型（类 class、接口 interface、枚举 enum、注解 annotation），JVM 必须在方法区中存储以下类型信息

- 这个类型的完整有效名称（全名=包名.类名）
- 这个类型直接父类的完整有效名（对于 interface或是 java.lang.Object，都没有父类）
- 这个类型的修饰符（public，abstract，final 的某个子集）
- 这个类型直接接口的一个有序列表

**域（Field）信息（类的成员变量）**

- JVM 必须在方法区中保存类型的所有域的相关信息以及域的声明顺序
- 域的相关信息包括：域名称、域类型、域修饰符（public、private、protected、static、final、volatile、transient 的某个子集）

**方法（Method）信息**

JVM 必须保存所有方法的

- 方法名称
- 方法的返回类型
- 方法参数的数量和类型
- 方法的修饰符（public，private，protected，static，final，synchronized，native，abstract 的一个子集）
- 方法的字符码（bytecodes）、操作数栈、局部变量表及大小（abstract 和 native 方法除外）
- 异常表（abstract 和 native 方法除外） 
  - 每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引



只有 HotSpot 才有永久代的概念

| jdk1.6及之前 | 有永久代，静态变量存放在永久代上                             |
| ------------ | ------------------------------------------------------------ |
| jdk1.7       | 有永久代，但已经逐步“去永久代”，**字符串常量池、静态变量**移除，保存在堆中 |
| jdk1.8及之后 | 取消永久代，类型信息、字段、方法、常量保存在本地内存的元空间，但字符串常量池、静态变量仍在堆中 |

HotSpot也是发展的，由于[一些问题在新窗口打开](http://openjdk.java.net/jeps/122)的存在，HotSpot考虑逐渐去永久代，对于不同版本的JDK，**实际的存储位置**是有差异的，具体看如下表格：

| JDK版本      | 是否有永久代，字符串常量池放在哪里？                         | 方法区逻辑上规范，由哪些实际的部分实现的？                   |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| jdk1.6及之前 | 有永久代，运行时常量池（包括字符串常量池），静态变量存放在永久代上 | 这个时期方法区在HotSpot中是由永久代来实现的，以至于**这个时期说方法区就是指永久代** |
| jdk1.7       | 有永久代，但已经逐步“去永久代”，字符串常量池、静态变量移除，保存在堆中； | 这个时期方法区在HotSpot中由**永久代**（类型信息、字段、方法、常量）和**堆**（字符串常量池、静态变量）共同实现 |
| jdk1.8及之后 | 取消永久代，类型信息、字段、方法、常量保存在本地内存的元空间，但字符串常量池、静态变量仍在堆中 | 这个时期方法区在HotSpot中由本地内存的**元空间**（类型信息、字段、方法、常量）和**堆**（字符串常量池、静态变量）共同实现 |

### JVM 基础 - Java 内存模型详解

**JVM内存结构**：通常被叫做JVM内存模型，也叫做Java内存区域，Java运行时数据区。

**JAVA内存模型**：是JMM（Java Memory Model，简称 JMM）。是定义了线程和主内存之间的抽象关系，即 JMM 定义了 JVM 在计算机内存中的工作方式。理解好 Java 内存模型，对于我们想深入了解 Java并发编程有先导作用

#### 基础

##### 并发编程模型的分类

在并发编程中，我们需要处理两个关键问题：**线程之间如何通信及线程之间如何同步**（这里的线程是指并发执行的活动实体）。通信是指线程之间以何种机制来交换信息。在命令式编程中，线程之间的通信机制有两种：**共享内存和消息传递**。

在**共享内存**的并发模型里，线程之间共享程序的公共状态，线程之间通过写 - 读内存中的公共状态来**隐式**进行通信。在**消息传递**的并发模型里，线程之间没有公共状态，线程之间必须通过明确的发送消息来**显式**进行通信。

**同步**是指**程序用于控制不同线程之间操作发生相对顺序的机制。**在**共享内存并发模型**里，同步是**显式**进行的。程序员必须显式指定某个方法或某段代码需要在线程之间互斥执行。在**消息传递**的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是**隐式**进行的。

java 的**并发**采用的是**共享内存**模型，Java 线程之间的**通信**总是**隐式**进行

##### Java 内存模型的抽象

Java 线程之间的**通信**由 **Java 内存模型（本文简称为 JMM）**控制，JMM 决定一个线程对共享变量的写入何时对另一个线程可见。

从抽象的角度来看，**JMM 定义了线程和主内存之间的抽象关系**：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读 / 写共享变量的副本。本地内存是 JMM 的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化。Java 内存模型的抽象示意图如下：

![img](https://www.pdai.tech/images/jvm/java-jmm-1.png)

![img](https://www.pdai.tech/images/jvm/java-jmm-2.png)

从整体来看，这两个步骤实质上是**线程 A 在向线程 B 发送消息**，而且这个**通信过程**必须要经过主内存。JMM 通过控制主内存与每个线程的本地内存之间的交互，来为 java 程序员提供内存可见性保证。

##### 重排序

在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。重排序分三种类型：

- **编译器优化的重排序**。编译器在不改变单线程程序语义的前提下，可以**重新安排语句的执行顺序**。
- **指令级并行的重排序**。现代处理器采用了指令级并行技术（Instruction-Level Parallelism， ILP）来将**多条指令重叠执行**。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
- **内存系统的重排序**。由于处理器使用**缓存和读 / 写缓冲区**，这使得加载和存储操作看上去可能是在乱序执行。

从 java 源代码到最终实际执行的指令序列，会分别经历下面三种重排序：

![img](https://www.pdai.tech/images/jvm/java-jmm-3.png)

上述的 1 属于编译器重排序，2 和 3 属于处理器重排序。这些重排序都可能会导致多线程程序出现内存可见性问题。

通过**内存屏障指令**来禁止特定类型的处理器重排序（不是所有的处理器重排序都要禁止）

JMM 属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供**一致的内存可见性**保证。



##### **处理器重排序与内存屏障指令**

JMM 把内存屏障指令分为下列四类：

| 屏障类型            | 指令示例                   | 说明                                                         |
| ------------------- | -------------------------- | ------------------------------------------------------------ |
| LoadLoad Barriers   | Load1; LoadLoad; Load2     | 确保 Load1 数据的装载，之前于 Load2 及所有后续装载指令的装载。 |
| StoreStore Barriers | Store1; StoreStore; Store2 | 确保 Store1 数据对其他处理器可见（刷新到内存），之前于 Store2 及所有后续存储指令的存储。 |
| LoadStore Barriers  | Load1; LoadStore; Store2   | 确保 Load1 数据装载，之前于 Store2 及所有后续的存储指令刷新到内存。 |
| StoreLoad Barriers  | Store1; StoreLoad; Load2   | 确保 Store1 数据对其他处理器变得可见（指刷新到内存），之前于 Load2 及所有后续装载指令的装载。 |

StoreLoad Barriers 会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令。

StoreLoad Barriers 是一个“全能型”的屏障，它同时具有其他三个屏障的效果。现代的多处理器大都支持该屏障（其他类型的屏障不一定被所有处理器支持）。执行该屏障开销会很昂贵，因为当前处理器通常要把写缓冲区中的数据全部刷新到内存中（buffer fully flush）。

##### **happens-before**

JSR-133 提出了 happens-before 的概念，通过这个概念来阐述操作之间的**内存可见性**。

如果一个操作执行的结果需要对另一个操作**可见**，那么这两个操作之间**必须存在** happens-before 关系。这里提到的两个操作既可以是在**一个线程**之内，也可以是在**不同线程**之间。 

与程序员密切相关的 happens-before 规则如下：

- 程序顺序规则：一个线程中的每个操作，happens- before 于该线程中的任意后续操作。
- 监视器锁规则：对一个监视器锁的解锁，happens- before 于随后对这个监视器锁的加锁。
- volatile 变量规则：对一个 volatile 域的写，happens- before 于任意后续对这个 volatile 域的读。
- 传递性：如果 A happens- before B，且 B happens- before C，那么 A happens- before C。

两个操作之间具有 happens-before 关系，**并不意味**着前一个操作必须要在后一个操作**之前执行**！happens-before 仅仅要求前一个操作（执行的结果）对后一个操作**可见**，**且**前一个操作按顺序排在第二个操作**之前**。

happens-before 与 JMM 的关系如下图所示：

![img](https://www.pdai.tech/images/jvm/java-jmm-5.png)

#### 重排序

**数据依赖性**：编译器和处理器在**重排序**时，会**遵守数据依赖性**，编译器和处理器**不会改变存在数据依赖关系的两个操作的执行顺序**。（只是针对单个处理器和单个线程，不同处理器之间和不同线程之间的数据依赖性不被编辑器和处理器考虑）

如果两个操作**访问同一个变量**，且这两个操作中有一个为**写操作**，此时这两个操作之间就存在数据依赖性。

数据依赖类型：

| 名称   | 代码示例     | 说明                           |
| ------ | ------------ | ------------------------------ |
| 写后读 | a = 1;b = a; | 写一个变量之后，再读这个位置。 |
| 写后写 | a = 1;a = 2; | 写一个变量之后，再写这个变量。 |
| 读后写 | a = b;b = 1; | 读一个变量之后，再写这个变量。 |

**as-if-serial**： 不管怎么重排序（编译器和处理器为了提高并行度），**（单线程）程序的执行结果不能被改变**。编译器，runtime 和处理器都必须遵守 as-if-serial 语义。

示例：

```java
double pi  = 3.14;    //A
double r   = 1.0;     //B
double area = pi * r * r; //C
```

**程序顺序规则** :如果 A happens- before B，且 B happens- before C，那么 A happens- before C。

JMM **并不要求** A 一定要在 B 之前执行。JMM 仅仅要求前一个操作（执行的结果）对后一个操作**可见**，且前一个操作按顺序排在第二个操作**之前**。

**重排序对多线程的影响：**在单线程程序中，对存在控制依赖的操作重排序，不会改变执行结果（这也是 as-if-serial 语义允许对存在控制依赖的操作做重排序的原因）；但在多线程程序中，对存在控制依赖的操作重排序，可能会改变程序的执行结果。

#### 顺序一致性

**数据竞争与顺序一致性保证**

当程序未正确同步时，就会存在数据竞争。java 内存模型规范对**数据竞争的定义**如下：

- 在一个线程中写一个变量，
- 在另一个线程读同一个变量，
- 而且写和读没有通过同步来排序。

JMM 对正确同步的多线程程序的内存一致性做了如下**保证**：如果程序是**正确同步**的，程序的执行将具有顺序一致性（sequentially consistent）

**顺序一致性内存模型**

顺序一致性内存模型有两大特性：

- 一个线程中的所有操作必须按照**程序的顺序**来执行。
- （不管程序是否同步）所有线程都只能看到一个单一的操作**执行顺序**。在顺序一致性内存模型中，每个操作都必须原子执行且**立刻对所有线程可见**。

**未同步程序的执行特性**

和顺序一致性模型一样，未同步程序在 JMM 中的执行时，整体上也是无序的，其执行结果也无法预知。同时，未同步程序在这两个模型中的执行特性有下面几个差异：

- 顺序一致性模型保证单线程内的操作会按程序的顺序执行，而 JMM 不保证单线程内的操作会按程序的顺序执行（比如上面正确同步的多线程程序在临界区内的重排序）。这一点前面已经讲过了，这里就不再赘述。
- 顺序一致性模型保证所有线程只能看到一致的操作执行顺序，而 JMM 不保证所有线程能看到一致的操作执行顺序。这一点前面也已经讲过，这里就不再赘述。
- JMM 不保证对 64 位的 long 型和 double 型变量的读 / 写操作具有原子性，而顺序一致性模型保证对所有的内存读 / 写操作都具有原子性。

#### 总结

**处理器内存模型**

顺序一致性内存模型是一个理论参考模型，JMM 和处理器内存模型在设计时通常会把顺序一致性内存模型作为参照。JMM 和处理器内存模型在设计时会对顺序一致性模型做一些放松，因为如果完全按照顺序一致性模型来实现处理器和 JMM，那么很多的处理器和编译器优化都要被禁止，这对执行性能将会有很大的影响。

下图展示了 JMM 在不同处理器内存模型中需要插入的内存屏障的示意图：

![img](https://www.pdai.tech/images/jvm/java-jmm-x01.png)

**JMM，处理器内存模型与顺序一致性内存模型之间的关系**

- JMM 是一个语言级的内存模型，
- 处理器内存模型是硬件级的内存模型，
- 顺序一致性内存模型是一个理论参考模型。

下面是语言内存模型，处理器内存模型和顺序一致性内存模型的强弱对比示意图：

![img](https://www.pdai.tech/images/jvm/java-jmm-x02.png)

从上图我们可以看出：常见的 4 种处理器内存模型比常用的 3 中语言内存模型要弱，处理器内存模型和语言内存模型都比顺序一致性内存模型要弱。同处理器内存模型一样，越是追求执行性能的语言，内存模型设计的会越弱。

**JMM 的内存可见性保证**

Java 程序的内存可见性保证按程序类型可以分为下列三类：

- **单线程程序**。单线程程序不会出现内存可见性问题。编译器，runtime 和处理器会共同确保单线程程序的执行结果与该程序在顺序一致性模型中的执行结果相同。
- **正确同步的多线程程序**。正确同步的多线程程序的执行将具有顺序一致性（程序的执行结果与该程序在顺序一致性内存模型中的执行结果相同）。这是 JMM 关注的重点，JMM 通过限制编译器和处理器的重排序来为程序员提供内存可见性保证。
- **未同步 / 未正确同步的多线程程序**。JMM 为它们提供了**最小安全性保障**：线程执行时读取到的值，要么是之前某个线程写入的值，要么是默认值（0，null，false）。

![img](https://www.pdai.tech/images/jvm/java-jmm-x04.png)

### GC - Java 垃圾回收基础知识

垃圾收集主要是针对**堆和方法区**进行；**程序计数器、虚拟机栈和本地方法栈**这三个区域属于线程私有的，只存在于线程的生命周期内，线程结束之后也会消失，因此不需要对这三个区域进行垃圾回收。

#### 判断一个对象是否可被回收：

**引用计数算法**：给对象添加一个引用计数器，当对象增加一个引用时计数器加 1，引用失效时计数器减 1。引用计数为 0 的对象可被回收。可能会出现循环引用情况，计数器永不为0

**可达性分析算法**：通过 GC Roots 作为起始点进行搜索，能够到达到的对象都是存活的，不可达的对象可被回收。

Java 虚拟机使用该算法来判断对象是否可被回收，在 Java 中 **GC Roots** 一般包含以下内容:

- 虚拟机栈中引用的对象
- 本地方法栈中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中的常量引用的对象

**方法区的回收：**主要是对常量池的回收和对类的卸载。

类的卸载条件很多，需要满足以下三个条件，并且满足了也不一定会被卸载:

- 该类所有的实例都已经被回收，也就是堆中不存在该类的任何实例。
- 加载该类的 ClassLoader 已经被回收。
- 该类对应的 Class 对象没有在任何地方被引用，也就无法在任何地方通过反射访问该类方法。

**finalize()：**用来做**关闭外部资源**等工作。当一个对象可被回收时，如果需要执行该对象的 finalize() 方法，那么就有可能通过在该方法中让对象重新被引用，从而实现**自救**。自救只能进行一次，如果回收的对象之前调用了 finalize() 方法自救，后面回收时不会调用 finalize() 方法。

#### 引用类型

**强引用：**被强引用关联的对象不会被回收。

**软引用：**被软引用关联的对象只有在内存不够的情况下才会被回收。

**弱引用：**被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。

**虚引用：**一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用取得一个对象。为一个对象设置虚引用关联的**唯一目的就是能在这个对象被回收时收到一个系统通知**。

直接内存会用到虚引用，虚引用与直接内存释放方法关联，通过引用关联释放外部直接内存（堆外内存）

#### 垃圾回收算法

**标记-清除**：将存活的对象进行标记，然后清理掉未被标记的对象。

不足:

- 标记和清除过程效率都不高；
- 会产生大量不连续的内存碎片，导致无法给大对象分配内存。

**标记-整理**：让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

**复制**：将内存划分为大小相等的两块，每次只使用其中一块，当这一块内存用完了就将还存活的对象复制到另一块上面，然后再把使用过的内存空间进行一次清理。

主要不足是只使用了内存的一半。

在回收时，将 Eden 和 Survivor 中还存活着的对象一次性复制到另一块 Survivor 空间上，最后清理 Eden 和使用过的那一块 Survivor。

![image](https://www.pdai.tech/images/pics/e6b733ad-606d-4028-b3e8-83c3a73a3797.jpg)

**分代收集**：

一般将堆分为新生代和老年代。

- 新生代使用: 复制算法
- 老年代使用: 标记 - 清除 或者 标记 - 整理 算法

#### 垃圾收集器

![image](https://www.pdai.tech/images/pics/c625baa0-dde6-449e-93df-c3a67f2f430f.jpg)

以上是 HotSpot 虚拟机中的 7 个垃圾收集器，连线表示垃圾收集器可以配合使用。

单线程与多线程: 单线程指的是垃圾收集器只使用一个线程进行收集，而多线程使用多个线程；

串行与并行: 串行指的是垃圾收集器与用户程序交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序；并形指的是垃圾收集器和用户程序同时执行。除了 **CMS 和 G1** 之外，其它垃圾收集器都是以串行的方式执行。

**1、Serial 收集器**：单线程

Serial 翻译为串行，也就是说它以串行的方式执行

它是 Client 模式下的默认新生代收集器，因为在用户的桌面应用场景下，分配给虚拟机管理的内存一般来说不会很大。

![image](https://www.pdai.tech/images/pics/22fda4ae-4dd5-489d-ab10-9ebfdad22ae0.jpg)

**2、ParNew 收集器**：serial收集器多线程版本

是 Server 模式下的虚拟机首选新生代收集器，除了性能原因外，主要是因为除了 Serial 收集器，只有它能与 CMS 收集器配合工作。

![image](https://www.pdai.tech/images/pics/81538cd5-1bcf-4e31-86e5-e198df1e013b.jpg)

**3、Parallel Scavenge 收集器**：多线程，“吞吐量优先”收集器。这里的吞吐量指 **CPU 用于运行用户代码的时间占总时间的比值**。其它收集器关注点是尽可能缩短垃圾收集时用户线程的停顿时间，

**4、Serial Old 收集器**：Serial 收集器的老年代版本

![image](https://www.pdai.tech/images/pics/08f32fd3-f736-4a67-81ca-295b2a7972f2.jpg)

**5、Parallel Old 收集器**：Parallel Scavenge 收集器的老年代版本

在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器。

![image](https://www.pdai.tech/images/pics/278fe431-af88-4a95-a895-9c3b80117de3.jpg)

**6、CMS 收集器**：CMS(Concurrent Mark Sweep)，Mark Sweep 指的是标记 - 清除算法。

![image](https://www.pdai.tech/images/pics/62e77997-6957-4b68-8d12-bfd609bb2c68.jpg)

分为以下四个流程:

- **初始标记:** 仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，**需要停顿**。
- **并发标记:** 进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，**不需要停顿**。
- **重新标记:** 为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，**需要停顿**。
- **并发清除:** **不需要停顿**。

在整个过程中**耗时最长**的**并发标记和并发清除**过程中，收集器线程都可以与用户线程一起工作，**不需要进行停顿**。

具有以下缺点:

- **吞吐量低:** 低停顿时间是以牺牲吞吐量为代价的，导致 CPU 利用率不够高。
- **无法处理浮动垃圾**，可能出现 Concurrent Mode Failure。浮动垃圾是指并发清除阶段由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次 GC 时才能进行回收。由于浮动垃圾的存在，因此需要预留出一部分内存，意味着 CMS 收集不能像其它收集器那样等待老年代快满的时候再回收。如果预留的内存不够存放浮动垃圾，就会出现 Concurrent Mode Failure，这时虚拟机将临时启用 Serial Old 来替代 CMS。
- **标记 - 清除算法导致的空间碎片**，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发一次 Full GC。

**7、 G1 收集器**：G1 可以直接对新生代和老年代一起回收。

G1(Garbage-First)，它是一款面向**服务端**应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。

G1 把堆划分成多个大小相等的独立区域(Region)，新生代和老年代不再物理隔离。

![image](https://www.pdai.tech/images/pics/9bbddeeb-e939-41f0-8e8e-2b1a0aa7e0a7.png)

每个 Region 都有一个 Remembered Set，用来记录该 Region 对象的引用对象所在的 Region。通过使用 Remembered Set，在做**可达性分析的时候就可以避免全堆扫描**。

![image](https://www.pdai.tech/images/pics/f99ee771-c56f-47fb-9148-c0036695b5fe.jpg)

如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤:

- 初始标记
- 并发标记
- 最终标记: 为了修正在并发标记期间因**用户程序继续运作**而**导致标记产生变动的那一部分标记记录**，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。这阶段**需要停顿线程**，但是可并行执行。
- 筛选回收: 首先对各个 Region 中的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。

具备如下特点:

- **空间整合**: 整体来看是基于“标记 - 整理”算法实现的收集器，从局部(两个 Region 之间)上来看是基于“复制”算法实现的，这意味着运行期间不会产生内存空间碎片。
- 可预测的停顿: 能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒。



#### 内存分配策略

- **对象优先在 Eden 分配**

- #### **大对象直接进入老年代**

- #### **长期存活的对象进入老年代**

- **动态对象年龄判定：**

虚拟机并不是永远地要求对象的年龄必须达到 MaxTenuringThreshold 才能晋升老年代，如果在 Survivor 中**相同年龄所有对象大小的总和大于 Survivor 空间的一半**，则**年龄大于或等于该年龄的对象**可以**直接进入老年代**，无需等到 MaxTenuringThreshold 中要求的年龄。

- **空间分配担保**

在发生 Minor GC 之前，虚拟机先检查**老年代最大可用的连续空间是否大于新生代所有对象总空间**，如果条件成立的话，那么 Minor GC 可以确认是安全的。

如果不成立的话虚拟机会查看 HandlePromotionFailure 设置值是否允许担保失败，如果允许那么就会继续检查**老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小**，如果大于，将尝试着进行一次 Minor GC；如果小于，或者 HandlePromotionFailure 设置不允许冒险，那么就要进行一次 Full GC。

#### Full GC 的触发条件：

**1. 调用 System.gc()**

只是建议虚拟机执行 Full GC，但是虚拟机不一定真正去执行。不建议使用这种方式，而是让虚拟机管理内存。

**[#](#_2-老年代空间不足) 2. 老年代空间不足**

老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等。

为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过 -Xmn 虚拟机参数调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过 -XX:MaxTenuringThreshold 调大对象进入老年代的年龄，让对象在新生代多存活一段时间。

**[#](#_3-空间分配担保失败) 3. 空间分配担保失败**

使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。

**[#](#_4-jdk-1-7-及以前的永久代空间不足) 4. JDK 1.7 及以前的永久代空间不足**

在 JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静态变量等数据。

当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC。如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError。

为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC。

**[#](#_5-concurrent-mode-failure) 5. Concurrent Mode Failure**

执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足(可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足)，便会报 Concurrent Mode Failure 错误，并触发 Full GC。

### GC - Java 垃圾回收器之G1详解

G1是一个分代的，增量的，并行与并发的**标记-复制**垃圾回收器。它的设计目标是**为了适应现在不断扩大的内存和不断增加的处理器数量，进一步降低暂停时间（pause time），同时兼顾良好的吞吐量。**

G1回收器和CMS比起来，有以下不同：

- G1垃圾回收器是**compacting**的，因此其回收得到的空间是连续的。
- G1将内存划分一个个固定大小的region，每个region可以是年轻代、老年代的一个。**内存的回收是以region作为基本单位的**；
- G1还有一个及其重要的特性：**软实时**（soft real-time）。所谓的实时垃圾回收，是指**在要求的时间内完成垃圾回收**。“软实时”则是指，用户可以指定垃圾回收时间的限时，G1会**努力在这个时限内完成垃圾回收**，但是G1并不担保每次都能在这个时限内完成垃圾回收。通过设定一个合理的目标，可以让达到90%以上的垃圾回收时间都在这个时限内。

#### G1的内存模型

##### 分区概念

G1分区示意图

![img](https://www.pdai.tech/images/java/java-jvm-gc-g1-1.jpeg)

**分区Region**

G1采用了分区(Region)的思路，将整个堆空间分成若干个大小相等的内存区域，每次分配对象空间将逐段地使用内存。

因此，在堆的使用上，G1并不要求对象的存储一定是物理上连续的，只要**逻辑上连续**即可；每个分区也不会确定地为某个代服务，可以按需在年轻代和老年代之间切换。

启动时可以通过参数-XX:G1HeapRegionSize=n可指定**分区大小**(1MB~32MB，且必须是2的幂)，默认将整堆划分为**2048**个分区。

**卡片Card**

在每个分区内部又被分成了若干个大小为**512 Byte卡片**(Card)，标识堆内存最小可用粒度所有分区的卡片将会记录在**全局卡片表**(Global Card Table)中，分配的对象会占用物理上连续的若干个卡片，当查找对分区内对象的引用时便可通过**记录卡片来查找该引用对象**(见RSet)。每次对内存的回收，都是对指定分区的卡片进行处理。

**堆Heap**

G1同样可以通过-Xms/-Xmx来指定堆空间大小。当发生年轻代收集或混合收集时，通过计算GC与应用的耗费时间比，自动调整堆空间大小。如果GC频率太高，则通过增加堆尺寸，来减少GC频率，相应地GC占用的时间也随之降低；目标参数-XX:GCTimeRatio即为GC与应用的耗费时间比，G1默认为9，而CMS默认为99，因为CMS的设计原则是耗费在GC上的时间尽可能的少。另外，当空间不足，如对象空间分配或转移失败时，G1会首先尝试增加堆空间，如果扩容失败，则发起担保的Full GC。Full GC后，堆尺寸计算结果也会调整堆空间。

#####  分代模型

![img](https://www.pdai.tech/images/java/java-jvm-gc-g1-2.jpeg)

与其他垃圾收集器类似，G1将内存在**逻辑上**划分为**年轻代和老年代**，其中年轻代又划分为Eden空间和Survivor空间。但年轻代空间并不是固定不变的，当现有年轻代分区占满时，JVM会分配新的空闲分区加入到年轻代空间。

整个年轻代内存会在初始空间`-XX:G1NewSizePercent`(默认整堆5%)与最大空间(默认60%)之间动态变化，且由参数目标暂停时间`-XX:MaxGCPauseMillis`(默认200ms)、需要扩缩容的大小以`-XX:G1MaxNewSizePercent`及分区的已记忆集合(RSet)计算得到。当然，G1依然可以设置固定的年轻代大小(参数-XX:NewRatio、-Xmn)，但同时暂停目标将失去意义。

##### 分区模型

![img](https://www.pdai.tech/images/java/java-jvm-gc-g1-3.jpeg)

G1对**内存的使用**以**分区**(Region)为单位，而对**对象的分配**则以**卡片**(Card)为单位。

**巨形对象Humongous Region**：一个大小达到甚至超过分区大小一半的对象称为巨型对象(Humongous Object)。**直接在老年代分配**

G1内部做了一个优化，一旦发现没有引用指向巨型对象，则可直接在**年轻代收集周期中被回收**。

**已记忆集合Remember Set (RSet)**：然而G1为了**避免STW式的整堆扫描**，在每个分区记录了一个已记忆集合(RSet)，内部类似一个反向指针，记录引用分区内对象的卡片索引。当要回收该分区时，通过扫描分区的RSet，来确定引用本分区内的对象是否存活，进而确定本分区内的对象存活情况。

不需要记录在RSet的引用：1、源自本分区的对象 2、G1 GC每次都会对年轻代进行整体收集，因此引用源自年轻代的对象也不需要。

最后只有**老年代**的分区**可能**会有RSet记录，这些分区称为拥有RSet分区(an RSet’s owning region)。

**Per Region Table (PRT)**：RSet在内部使用Per Region Table(PRT)**记录分区的引用情况**。由于RSet的记录要占用分区的空间，如果一个分区非常"受欢迎"，那么RSet占用的空间会上升，从而降低分区的可用空间。G1应对这个问题采用了改变RSet的密度的方式，在PRT中将会以三种模式记录引用：

- 稀少：直接记录引用对象的卡片索引
- 细粒度：记录引用对象的分区索引
- 粗粒度：只记录引用情况，每个分区对应一个比特位

由上可知，粗粒度的PRT只是记录了引用数量，需要通过整堆扫描才能找出所有引用，因此扫描速度也是最慢的。

##### 收集集合 (CSet)

![img](https://www.pdai.tech/images/java/java-jvm-gc-g1-4.jpeg)

收集集合(CSet)代表每次GC暂停时**回收的一系列目标分区**。

G1的收集都是**根据CSet进行操作的**，年轻代收集与混合收集没有明显的不同，最大的区别在于两种收集的触发条件。

**年轻代收集集合 CSet of Young Collection**：应用线程不断活动后，年轻代空间会被逐渐填满。当**JVM分配对象到Eden区域失败(Eden区已满)**时，便会触发一次STW式的年轻代收集。在年轻代收集中，Eden分区存活的对象将被拷贝到Survivor分区；原有Survivor分区存活的对象，将根据任期阈值(tenuring threshold)分别晋升到PLAB中，新的survivor分区和老年代分区。而原有的年轻代分区将被整体回收掉。

**混合收集集合 CSet of Mixed Collection**：年轻代收集不断活动后，老年代的空间也会被逐渐填充。当**老年代占用空间超过整堆比IHOP阈值**-XX:InitiatingHeapOccupancyPercent(默认45%)时，G1就会启动一次混合垃圾收集周期。

为了满足暂停目标，G1可能不能一口气将所有的候选分区收集掉，因此G1可能会产生**连续多次的混合收集与应用线程交替执行**，每次STW的混合收集与年轻代收集过程相类似。

为了确定包含到年轻代收集集合CSet的老年代分区，JVM通过参数混合周期的最大总次数-XX:G1MixedGCCountTarget(默认8)、堆废物百分比-XX:G1HeapWastePercent(默认5%)。通过候选老年代分区总数与混合周期最大总次数，确定每次包含到CSet的最小分区数量；根据堆废物百分比，当收集达到参数时，不再启动新的混合收集。而每次添加到CSet的分区，则通过计算得到的GC效率进行安排。

**并发标记算法（三色标记法）**

CMS和G1在并发标记时使用的是同一个算法：三色标记法，使用白灰黑三种颜色标记对象。**白色是未标记；灰色自身被标记，引用的对象未标记；黑色自身与引用对象都已标记。**

![img](https://www.pdai.tech/images/java/java-jvm-gc-g1-5.png)

**GC 开始前所有对象都是白色，GC 一开始所有根能够直达的对象被压到栈中，待搜索，此时颜色是灰色。然后灰色对象依次从栈中取出搜索子对象，子对象也会被涂为灰色，入栈。当其所有的子对象都涂为灰色之后该对象被涂为黑色。当 GC 结束之后灰色对象将全部没了，剩下黑色的为存活对象，白色的为垃圾。**

![img](https://www.pdai.tech/images/java/java-jvm-gc-g1-5-1.gif)

**漏标问题**

在remark过程中，黑色指向了白色，如果不对黑色重新扫描，则会漏标。会把白色D对象当作没有新引用指向从而回收掉。

![img](https://www.pdai.tech/images/java/java-jvm-gc-g1-6.png)

并发标记过程中，Mutator（java应用线程）删除了所有从灰色到白色的引用，会产生漏标。此时白色对象应该被回收

在remark过程中，产生漏标问题的条件有两个：

- 黑色对象指向了白色对象
- 灰色对象指向白色对象的引用消失

所以要解决漏标问题，打破两个条件之一即可：

- **跟踪黑指向白的增加** incremental update：增量更新，关注引用的增加，把**黑色重新标记为灰色**，下次重新扫描属性。**CMS采用该方法。**
- **记录灰指向白的消失** SATB snapshot at the beginning：关注引用的删除，当灰–>白消失时，要把这个 **引用 推到GC的堆栈**，保证白还能被GC扫描到。**G1采用该方法。**

**为什么G1采用SATB而不用incremental update**？

因为采用incremental update把黑色重新标记为灰色后，之前扫描过的**还要再扫描一遍，效率太低**。

也就是说 灰色–>白色 引用消失时，如果没有 黑色–>白色，**引用会被push到堆栈，下次扫描时拿到这个引用，由于有RSet的存在，不需要扫描整个堆去查找指向白色的引用，效率比较高**。SATB配合RSet浑然天成。

#### G1的活动周期

##### RSet的维护

通过维护RSet，得到准确的分区引用信息。

在G1中，RSet的维护主要来源两个方面：**写栅栏(Write Barrier)和并发优化线程(Concurrence Refinement Threads)**

**栅栏Barrier:**

![img](https://www.pdai.tech/images/java/java-jvm-gc-g1-8.jpeg)

栅栏是指在原生代码片段中，当某些语句被执行时，栅栏代码也会被执行。而G1主要在赋值语句中，使用写前栅栏(Pre-Write Barrrier)和写后栅栏(Post-Write Barrrier)。

**写前栅栏 Pre-Write Barrrier**

即将执行一段赋值语句时，**等式左侧对象将修改引用到另一个对象**，那么等式左侧对象原先引用的对象所在分区将因此丧失一个引用，那么JVM就需要在赋值语句生效之前，**记录丧失引用的对象。JVM并不会立即维护RSet，而是通过批量处理，在将来RSet更新(见SATB)。**

**写后栅栏 Post-Write Barrrier**

当执行一段赋值语句后，**等式右侧对象获取了左侧对象的引用**，那么**等式右侧对象所在分区的RSet也应该得到更新**。同样为了降低开销，写后栅栏发生后，**RSet也不会立即更新，同样只是记录此次更新日志，在将来批量处理(见Concurrence Refinement Threads)。**

**起始快照算法Snapshot at the beginning (SATB)**：主要针对标记-清除垃圾收集器的并发标记阶段，

**并发优化线程Concurrence Refinement Threads**：并发优化线程(Concurrence Refinement Threads)，只专注扫描日志缓冲区记录的卡片来维护更新RSet

##### 并发标记周期Concurrent Marking Cycle

并发标记周期是G1中非常重要的阶段，这个阶段将会为混合收集周期**识别垃圾最多的老年代分区**。

当**达到IHOP阈值**`-XX:InitiatingHeapOccupancyPercent`(老年代占整堆比，默认45%)时，便会触发并发标记周期。

整个并发标记周期将由**初始标记(Initial Mark)、根分区扫描(Root Region Scanning)、并发标记(Concurrent Marking)、重新标记(Remark)、清除(Cleanup)**几个阶段组成。

**初始标记 Initial Mark（STW）**

初始标记(Initial Mark)负责标记所有能被**直接可达的根对象**(原生栈对象、全局对象、JNI对象)，根是对象图的起点，因此初始标记需要将Mutator线程(Java应用线程)**暂停**掉，也就是需要一个STW的时间段。

事实上，当达到IHOP阈值时，G1并不会立即发起并发标记周期，而是等待下一次年轻代收集，**利用年轻代收集的STW时间段，完成初始标记**，这种方式称为借道(Piggybacking)。

**根分区扫描 Root Region Scanning**

在初始标记暂停结束后，年轻代收集也完成的对象复制到Survivor的工作，应用线程开始活跃起来。此时为了保证标记算法的正确性，**所有新复制到Survivor分区的对象，都需要被扫描并标记成根，这个过程称为根分区扫描(Root Region Scanning)，同时扫描的Suvivor分区也被称为根分区(Root Region)。**根分区扫描必须在**下一次年轻代垃圾收集启动前完成**(并发标记的过程中，可能会被若干次年轻代垃圾收集打断)，因为每次GC会产生新的存活对象集合。

**并发标记 Concurrent Marking（非STW）**

和应用线程并发执行，并发标记线程在并发标记阶段启动，由参数`-XX:ConcGCThreads`(默认GC线程数的1/4，即`-XX:ParallelGCThreads/4`)控制启动数量，每个线程每次只扫描一个分区，从而标记出存活对象图。

**存活数据计算 Live Data Accounting**

存活数据计算(Live Data Accounting)是标记操作的附加产物，**只要一个对象被标记，同时会被计算字节数，并计入分区空间**。

**重新标记 Remark（STW）**

重新标记(Remark)是最后一个标记阶段。在该阶段中，G1需要一个**暂停**的时间，去处理剩下的SATB日志缓冲区和所有更新，找出所有未被访问的存活对象，同时安全完成存活数据计算。这个阶段也是**并行执行**的，通过参数-XX:ParallelGCThread可设置GC暂停时可用的GC线程数。

**清除 Cleanup（STW）**

紧挨着重新标记阶段的清除(Clean)阶段也是**STW**的。Previous/Next标记位图、以及PTAMS/NTAMS，都会在清除阶段交换角色。清除阶段主要执行以下操作：

- **RSet梳理**，启发式算法会根据活跃度和RSet尺寸对分区定义不同等级，同时RSet数理也**有助于发现无用的引用**。
- **整理堆分区**，为混合收集周期**识别回收收益高(**基于释放空间和暂停目标)的**老年代分区集合**；
- **识别所有空闲分区**，即**发现无存活对象的分区**。该分区可在清除阶段直接回收，无需等待下次收集周期。

##### 年轻代收集/混合收集周期

年轻代收集和混合收集周期，是G1**回收空间**的主要活动。

G1并不会马上开始一次混合收集，而是让应用线程先运行一段时间，**等待触发一次年轻代收集**。在这次STW中，G1将保准整理混合收集周期。接着再次让应用线程运行，**当接下来的几次年轻代收集时，将会有老年代分区加入到CSet中，即触发混合收集**，这些连续多次的混合收集称为混合收集周期(Mixed Collection Cycle)。

**GC工作线程数**:-XX:ParallelGCThreads,默认与CPU核数相等

 **年轻代收集 Young Collection**

每次收集过程中，既有**并行执行**的活动，也有**串行执行**的活动，但都可以是多线程的。

**并行活动**

- `外部根分区扫描 Ext Root Scanning`：此活动对堆外的根(JVM系统目录、VM数据结构、JNI线程句柄、硬件寄存器、全局变量、线程对栈根)进行扫描，发现那些没有加入到暂停收集集合CSet中的对象。如果系统目录(单根)拥有大量加载的类，最终可能其他并行活动结束后，该活动依然没有结束而带来的等待时间。
- `更新已记忆集合 Update RS`：并发优化线程会对脏卡片的分区进行扫描更新日志缓冲区来更新RSet，但只会处理全局缓冲列表。作为补充，所有被记录但是还没有被优化线程处理的剩余缓冲区，会在该阶段处理，变成已处理缓冲区(Processed Buffers)。为了限制花在更新RSet的时间，可以设置暂停占用百分比-XX:G1RSetUpdatingPauseTimePercent(默认10%，即-XX:MaxGCPauseMills/10)。值得注意的是，如果更新日志缓冲区更新的任务不降低，单纯地减少RSet的更新时间，会导致暂停中被处理的缓冲区减少，将日志缓冲区更新工作推到并发优化线程上，从而增加对Java应用线程资源的争夺。
- `RSet扫描 Scan RS`：在收集当前CSet之前，考虑到分区外的引用，必须扫描CSet分区的RSet。如果RSet发生粗化，则会增加RSet的扫描时间。开启诊断模式-XX:UnlockDiagnosticVMOptions后，通过参数-XX:+G1SummarizeRSetStats可以确定并发优化线程是否能够及时处理更新日志缓冲区，并提供更多的信息，来帮助为RSet粗化总数提供窗口。参数-XX：G1SummarizeRSetStatsPeriod=n可设置RSet的统计周期，即经历多少此GC后进行一次统计
- `代码根扫描 Code Root Scanning`：对代码根集合进行扫描，扫描JVM编译后代码Native Method的引用信息(nmethod扫描)，进行RSet扫描。事实上，只有CSet分区中的RSet有强代码根时，才会做nmethod扫描，查找对CSet的引用。
- `转移和回收 Object Copy`：通过选定的CSet以及CSet分区完整的引用集，将执行暂停时间的主要部分：CSet分区存活对象的转移、CSet分区空间的回收。通过工作窃取机制来负载均衡地选定复制对象的线程，并且复制和扫描对象被转移的存活对象将拷贝到每个GC线程分配缓冲区GCLAB。G1会通过计算，预测分区复制所花费的时间，从而调整年轻代的尺寸。
- `终止 Termination`：完成上述任务后，如果任务队列已空，则工作线程会发起终止要求。如果还有其他线程继续工作，空闲的线程会通过工作窃取机制尝试帮助其他线程处理。而单独执行根分区扫描的线程，如果任务过重，最终会晚于终止。
- `GC外部的并行活动 GC Worker Other`：该部分并非GC的活动，而是JVM的活动导致占用了GC暂停时间(例如JNI编译)。

**串行活动**

- `代码根更新 Code Root Fixup`：根据转移对象更新代码根。
- `代码根清理 Code Root Purge`：清理代码根集合表。
- `清除全局卡片标记 Clear CT`：在任意收集周期会扫描CSet与RSet记录的PRT，扫描时会在全局卡片表中进行标记，防止重复扫描。在收集周期的最后将会清除全局卡片表中的已扫描标志。
- `选择下次收集集合 Choose CSet`：该部分主要用于并发标记周期后的年轻代收集、以及混合收集中，在这些收集过程中，由于有老年代候选分区的加入，往往需要对下次收集的范围做出界定；但单纯的年轻代收集中，所有收集的分区都会被收集，不存在选择。
- `引用处理 Ref Proc`：主要针对软引用、弱引用、虚引用、final引用、JNI引用。当Ref Proc占用时间过多时，可选择使用参数`-XX:ParallelRefProcEnabled`激活多线程引用处理。G1希望应用能小心使用软引用，因为软引用会一直占据内存空间直到空间耗尽时被Full GC回收掉；即使未发生Full GC，软引用对内存的占用，也会导致GC次数的增加。
- `引用排队 Ref Enq`：此项活动可能会导致RSet的更新，此时会通过记录日志，将关联的卡片标记为脏卡片。
- `卡片重新脏化 Redirty Cards`：重新脏化卡片。
- `回收空闲巨型分区 Humongous Reclaim`：G1做了一个优化：通过查看所有根对象以及年轻代分区的RSet，如果确定RSet中巨型对象没有任何引用，则说明G1发现了一个不可达的巨型对象，该对象分区会被回收。
- `释放分区 Free CSet`：回收CSet分区的所有空间，并加入到空闲分区中。
- `其他活动 Other`：GC中可能还会经历其他耗时很小的活动，如修复JNI句柄等。

##### 并发标记周期后的年轻代收集 Young Collection Following Concurrent Marking Cycle

当G1发起并发标记周期之后，并不会马上开始混合收集。G1会先等待下一次年轻代收集，然后在该收集阶段中，确定下次混合收集的CSet(Choose CSet)。

**混合收集周期 Mixed Collection Cycle**

单次的混合收集与年轻代收集并无二致。根据暂停目标，老年代的分区可能不能一次暂停收集中被处理完，G1会发起连续多次的混合收集，称为混合收集周期(Mixed Collection Cycle)。

G1会计算每次加入到CSet中的分区数量、混合收集进行次数，并且在上次的年轻代收集、以及接下来的混合收集中，G1会确定下次加入CSet的分区集(Choose CSet)，并且确定是否结束混合收集周期。

**转移失败的担保机制 Full GC**

转移失败(Evacuation Failure)是指当G1无法在堆空间中申请新的分区时，G1便会触发担保机制，执行一次STW式的、单线程的Full GC。**Full GC会对整堆做标记清除和压缩，最后将只包含纯粹的存活对象**。参数-XX:G1ReservePercent(默认10%)可以保留空间，来应对晋升模式下的异常情况，最大占用整堆50%，更大也无意义。

G1在以下场景中会触发Full GC，同时会在日志中记录to-space-exhausted以及Evacuation Failure：

- 从年轻代分区拷贝存活对象时，无法找到可用的空闲分区
- 从老年代分区转移存活对象时，无法找到可用的空闲分区
- 分配巨型对象时在老年代无法找到足够的连续分区

由于G1的应用场合往往堆内存都比较大，所以Full GC的收集代价非常昂贵，应该避免Full GC的发生

####  总结

G1是一款非常优秀的垃圾收集器，不仅适合堆内存大的应用，同时也简化了调优的工作。通过主要的参数初始和最大堆空间、以及最大容忍的GC暂停目标，就能得到不错的性能；同时，我们也看到G1对内存空间的浪费较高，但通过**首先收集尽可能多的垃圾**(Garbage First)的设计原则，可以及时发现过期对象，从而让内存占用处于合理的水平。

虽然G1也有类似CMS的收集动作：初始标记、并发标记、重新标记、清除、转移回收，并且也以一个串行收集器做担保机制，但单纯地以类似前三种的过程描述显得并不是很妥当。

- G1的设计原则是"**首先收集尽可能多的垃圾**(Garbage First)"。因此，**G1并不会等内存耗尽(串行、并行)或者快耗尽(CMS)的时候开始垃圾收集**，而是在内部采用了启发式算法，在老年代找出具有高收集收益的分区进行收集。同时G1可以根据用户设置的暂停时间目标自动调整年轻代和总堆大小，暂停目标越短年轻代空间越小、总空间就越大；
- **G1采用内存分区(Region)的思路**，将内存划分为一个个相等大小的内存分区，回收时则以分区为单位进行回收，存活的对象复制到另一个空闲分区中。由于都是以相等大小的分区为单位进行操作，因此G1天然就是一种压缩方案(局部压缩)；
- G1虽然也是分代收集器，但整个内存分区不存在物理上的年轻代与老年代的区别，也不需要完全独立的survivor(to space)堆做复制准备。**G1只有逻辑上的分代概念，或者说每个分区都可能随G1的运行在不同代之间前后切换；**
- **G1的收集都是STW的**（新生代回收所有阶段全STW，混合回收除了并发标记全是STW），但年轻代和老年代的收集界限比较模糊，采用了混合(mixed)收集的方式。即每次收集既可能只收集年轻代分区(年轻代收集)，也可能在收集年轻代的同时，包含部分老年代分区(混合收集)，这样即使堆内存很大时，也可以限制收集范围，从而降低停顿。

### GC - Java 垃圾回收器之ZGC详解

ZGC（The Z Garbage Collector）是JDK 11中推出的一款低延迟垃圾回收器, 是JDK 11+ 最为重要的更新之一，适用于**大内存低延迟**服务的内存管理和回收。

> 它的设计目标包括：

- 停顿时间不超过10ms；
- 停顿时间不会随着堆的大小，或者活跃对象的大小而增加（对程序吞吐量影响小于15%）；
- 支持8MB~4TB级别的堆（未来支持16TB）。

#### GC之痛

很多低延迟高可用Java服务的系统可用性经常受**GC停顿**的困扰。GC停顿指垃圾回收期间STW（Stop The World），当STW时，所有应用线程停止活动，等待GC停顿结束。

##### CMS与G1停顿时间瓶颈

G1的混合回收过程可以分为标记阶段、清理阶段和复制阶段。

G1四个STW过程中，**初始标记**因为只标记GC Roots，耗时较短。**再标记**因为对象数少，耗时也较短。**清理阶段**因为内存分区数量少，耗时也较短。转移阶段要处理所有存活的对象，耗时会较长。因此，**G1停顿时间的瓶颈主要是标记-复制中的转移阶段STW**。为什么转移阶段不能和标记阶段一样并发执行呢？主要**是G1未能解决转移过程中准确定位对象地址的问题。**

G1的Young GC和CMS的Young GC，其标记-复制全过程STW

#### ZGC原理

##### **全并发的ZGC**

与CMS中的ParNew和G1类似，**ZGC也采用标记-复制算法**，不过ZGC对该算法做了重大改进：**ZGC在标记、转移和重定位阶段几乎都是并发**的，这是ZGC实现停顿时间小于10ms目标的最关键原因。

垃圾回收周期

![img](https://www.pdai.tech/images/java/jvm/java-jvm-zgc-2.png)

ZGC只有三个STW阶段：**初始标记，再标记，初始转移**。ZGC几乎所有暂停都只**依赖于GC Roots集合大小**，停顿时间不会随着堆的大小或者活跃对象的大小而增加。

##### ZGC关键技术

ZGC通过**着色指针和读屏障技术**，**解决了转移过程中准确访问对象的问题，实现了并发转移**。

大致原理描述如下：并发转移中“并发”意味着GC线程在转移对象的过程中，应用线程也在不停地访问对象。假设对象发生转移，但对象地址未及时更新，那么应用线程可能访问到旧地址，从而造成错误。而在ZGC中，应用线程访问对象将触发“读屏障”，如果发现对象被移动了，那么**“读屏障”会把读出来的指针更新到对象的新地址上，这样应用线程始终访问的都是对象的新地址**。

那么，JVM是如何判断对象被移动过呢？就是**利用对象引用的地址，即着色指针**。

**着色指针**

ZGC仅支持**64位**系统，它把64位虚拟地址空间划分为多个子空间，

![img](https://www.pdai.tech/images/java/jvm/java-jvm-zgc-3.png)

其中，[0~4TB) 对应Java堆，[4TB ~ 8TB) 称为M0地址空间，[8TB ~ 12TB) 称为M1地址空间，[12TB ~ 16TB) 预留未使用，[16TB ~ 20TB) 称为Remapped空间。

当应用程序创建对象时，首先在堆空间申请一个虚拟地址，但该虚拟地址并不会映射到真正的物理地址。ZGC同时会为该对象在M0、M1和Remapped地址空间分别申请一个虚拟地址，且**这三个虚拟地址对应同一个物理地址**，但这**三个空间在同一时间有且只有一个空间有效。**

它使用**“空间换时间”**思想，去降低GC停顿时间。“空间换时间”中的空间是虚拟空间，而不是真正的物理空间。

ZGC实际仅使用64位地址空间的第0-41位，而第42-45位存储元数据，第47~63位固定为0。

![img](https://www.pdai.tech/images/java/jvm/java-jvm-zgc-4.png)

ZGC将对象存活信息存储在42~45位中，这与传统的垃圾回收并将对象存活信息放在对象头中完全不同。

**读屏障**

读屏障是JVM向应用代码**插入一小段代码**的技术。当应用线程从堆中读取对象引用时，就会执行这段代码。需要注意的是，仅“**从堆中读取对象引用**”才会触发这段代码。

ZGC中读屏障的代码作用：**在对象标记和转移过程中，用于确定对象的引用地址是否满足条件，并作出相应动作**。

##### ZGC并发处理演示

ZGC一次垃圾回收周期中地址视图的切换过程：

- **初始化**：ZGC初始化之后，整个内存空间的地址视图被设置为Remapped。程序正常运行，在内存中分配对象，满足一定条件后垃圾回收启动，此时进入标记阶段。
- **并发标记阶段**：第一次进入标记阶段时视图为M0，如果对象被GC标记线程或者应用线程访问过，那么就将对象的地址视图从Remapped调整为M0。所以，在标记阶段结束之后，对象的地址要么是M0视图，要么是Remapped。如果对象的地址是M0视图，那么说明对象是活跃的；如果对象的地址是Remapped视图，说明对象是不活跃的。
- **并发转移阶段**：标记结束后就进入转移阶段，此时地址视图再次被设置为Remapped。如果对象被GC转移线程或者应用线程访问过，那么就将对象的地址视图从M0调整为Remapped。

其实，在标记阶段存在两个地址视图M0和M1，上面的过程显示只用了一个地址视图。**之所以设计成两个，是为了区别前一次标记和当前标记**。也即，第二次进入并发标记阶段后，地址视图调整为M1，而非M0。

![img](https://www.pdai.tech/images/java/jvm/java-jvm-zgc-5.png)

#### ZGC调优实践

##### 调优基础知识

**理解ZGC重要配置参数**

重要参数配置样例：

```java
-Xms10G -Xmx10G 
-XX:ReservedCodeCacheSize=256m -XX:InitialCodeCacheSize=256m 
-XX:+UnlockExperimentalVMOptions -XX:+UseZGC 
-XX:ConcGCThreads=2 -XX:ParallelGCThreads=6 
-XX:ZCollectionInterval=120 -XX:ZAllocationSpikeTolerance=5 
-XX:+UnlockDiagnosticVMOptions -XX:-ZProactive 
-Xlog:safepoint,classhisto*=trace,age*,gc*=info:file=/opt/logs/logs/gc-%t.log:time,tid,tags:filecount=5,filesize=50m 
```

 `-Xms -Xmx`：堆的最大内存和最小内存，这里都设置为10G，程序的堆内存将保持10G不变。（把两者设置为一致,是为了避免频繁扩容和GC释放[堆内存](https://so.csdn.net/so/search?q=堆内存&spm=1001.2101.3001.7020)造成的系统开销/压力）

`-XX:ReservedCodeCacheSize -XX:InitialCodeCacheSize`：设置CodeCache的大小， JIT编译的代码都放在CodeCache中，一般服务64m或128m就已经足够。我们的服务因为有一定特殊性，所以设置的较大，后面会详细介绍。

`-XX:+UnlockExperimentalVMOptions -XX:+UseZGC`：启用ZGC的配置。

`-XX:ConcGCThreads`：并发回收垃圾的线程。默认是总核数的12.5%，8核CPU默认是1。调大后GC变快，但会占用程序运行时的CPU资源，吞吐会受到影响。

`-XX:ParallelGCThreads`：STW阶段使用线程数，默认是总核数的60%。

`-XX:ZCollectionInterval`：ZGC发生的最小时间间隔，单位秒。

`-XX:ZAllocationSpikeTolerance`：ZGC触发自适应算法的修正系数，默认2，数值越大，越早的触发ZGC。

`-XX:+UnlockDiagnosticVMOptions -XX:-ZProactive`：是否启用主动回收，默认开启，这里的配置表示关闭。

`-Xlog`：设置GC日志中的内容、格式、位置以及每个日志的大小。 

**理解ZGC触发时机**

ZGC有多种GC触发机制:

**阻塞内存分配请求触发**：当垃圾来不及回收，垃圾将堆占满时，会导致部分线程阻塞。我们应当避免出现这种触发方式。日志中关键字是“**Allocation Stall**”。

**基于分配速率的自适应算法**：最主要的GC触发方式，其算法原理可简单描述为”ZGC根据近期的对象分配速率以及GC时间，计算出当内存占用达到什么阈值时触发下一次GC”。自适应算法的详细理论可参考彭成寒《新一代垃圾回收器ZGC设计与实现》一书中的内容。通过ZAllocationSpikeTolerance参数控制阈值大小，该参数默认2，数值越大，越早的触发GC。我们通过调整此参数解决了一些问题。日志中关键字是“**Allocation Rate**”。

**基于固定时间间隔**：**通过ZCollectionInterval控制，适合应对突增流量场景**。流量平稳变化时，自适应算法可能在堆使用率达到95%以上才触发GC。流量突增时，自适应算法触发的时机可能会过晚，导致部分线程阻塞。我们通过调整此参数解决流量突增场景的问题，比如定时活动、秒杀等场景。日志中关键字是“**Timer**”。

**主动触发规则**：类似于固定间隔规则，但时间间隔不固定，是ZGC自行算出来的时机，我们的服务因为已经加了基于固定时间间隔的触发机制，所以通过-ZProactive参数将该功能关闭，以免GC频繁，影响服务可用性。 日志中关键字是“**Proactive**”。

**预热规则**：服务刚启动时出现，一般不需要关注。日志中关键字是“**Warmup**”。

**外部触发**：代码中显式调用System.gc()触发。 日志中关键字是“**System.gc()**”。

**元数据分配触发**：元数据区不足时导致，一般不需要关注。 日志中关键字是“**Metadata GC Threshold**”。

**理解ZGC日志**

一次完整的GC过程，需要注意的点已在图中标出。

![img](https://www.pdai.tech/images/java/jvm/java-jvm-zgc-6.png)

注意：该日志过滤了进入安全点的信息。正常情况，在一次GC过程中还穿插着进入安全点的操作。

GC日志中每一行都注明了GC过程中的信息，关键信息如下：

- **Start**：开始GC，并标明的GC触发的原因。上图中触发原因是自适应算法。
- **Phase-Pause Mark Start**：初始标记，会STW。
- **Phase-Pause Mark End**：再次标记，会STW。
- **Phase-Pause Relocate Start**：初始转移，会STW。
- **Heap信息**：记录了GC过程中Mark、Relocate前后的堆大小变化状况。High和Low记录了其中的最大值和最小值，我们一般关注High中Used的值，如果达到100%，在GC过程中一定存在内存分配不足的情况，需要调整GC的触发时机，更早或者更快地进行GC。
- **GC信息统计**：可以定时的打印垃圾收集信息，观察10秒内、10分钟内、10个小时内，从启动到现在的所有统计信息。利用这些统计信息，可以排查定位一些异常点。

**理解ZGC停顿原因**

我们在实战过程中共发现了6种使程序停顿的场景，分别如下：

- **GC时，初始标记**：日志中Pause Mark Start。
- **GC时，再标记**：日志中Pause Mark End。
- **GC时，初始转移**：日志中Pause Relocate Start。
- **内存分配阻塞**：当内存不足时线程会阻塞等待GC完成，关键字是”Allocation Stall”。

- **安全点**：所有线程进入到安全点后才能进行GC，ZGC定期进入安全点判断是否需要GC。先进入安全点的线程需要等待后进入安全点的线程直到所有线程挂起。
- **dump线程、内存**：比如jstack、jmap命令。

![img](https://www.pdai.tech/images/java/jvm/java-jvm-zgc-9.png)

![img](https://www.pdai.tech/images/java/jvm/java-jvm-zgc-10.png)

**调优案例**

我们维护的服务名叫Zeus，它是美团的规则平台，常用于风控场景中的规则管理。规则运行是基于开源的表达式执行引擎Aviator。Aviator内部将每一条表达式转化成Java的一个类，通过调用该类的接口实现表达式逻辑。

Zeus服务内的规则数量超过万条，且每台机器每天的请求量几百万。这些客观条件导致Aviator生成的类和方法会产生很多的ClassLoader和CodeCache，这些在使用ZGC时都成为过GC的性能瓶颈。接下来介绍两类调优案例。

> **第一类：内存分配阻塞，系统停顿可达到秒级**

**[#](#案例一-秒杀活动中流量突增-出现性能毛刺) 案例一：秒杀活动中流量突增，出现性能毛刺**

日志信息：对比出现性能毛刺时间点的GC日志和业务日志，发现JVM停顿了较长时间，且停顿时GC日志中有大量的“Allocation Stall”日志。

分析：**这种案例多出现在“自适应算法”为主要GC触发机制的场景中**。ZGC是一款并发的垃圾回收器，GC线程和应用线程同时活动，在GC过程中，还会产生新的对象。GC完成之前，新产生的对象将堆占满，那么应用线程可能因为申请内存失败而导致线程阻塞。当秒杀活动开始，大量请求打入系统，但自适应算法计算的GC触发间隔较长，导致GC触发不及时，引起了内存分配阻塞，导致停顿。

解决方法：

（1）**开启”基于固定时间间隔“的GC触发机制**：-XX:ZCollectionInterval。比如调整为5秒，甚至更短。 

（2）**增大修正系数-XX:ZAllocationSpikeTolerance，更早触发GC**。ZGC采用正态分布模型预测内存分配速率，模型修正系数ZAllocationSpikeTolerance默认值为2，值越大，越早的触发GC，Zeus中所有集群设置的是5。

**[#](#案例二-压测时-流量逐渐增大到一定程度后-出现性能毛刺) 案例二：压测时，流量逐渐增大到一定程度后，出现性能毛刺**

日志信息：平均1秒GC一次，两次GC之间几乎没有间隔。

分析：GC触发及时，但**内存标记和回收速度过慢**，引起内存分配阻塞，导致停顿。

解决方法：**增大-XX:ConcGCThreads， 加快并发标记和回收速度**。ConcGCThreads默认值是核数的1/8，8核机器，默认值是1。该参数影响系统吞吐，如果GC间隔时间大于GC周期，不建议调整该参数。

> **第二类：GC Roots 数量大，单次GC停顿时间长**

**[#](#案例三-单次gc停顿时间30ms-与预期停顿10ms左右有较大差距) 案例三： 单次GC停顿时间30ms，与预期停顿10ms左右有较大差距**

日志信息：观察ZGC日志信息统计，“Pause Roots ClassLoaderDataGraph”一项耗时较长。

分析：dump内存文件，发现系统中有上万个ClassLoader实例。我们知道ClassLoader属于GC Roots一部分，且ZGC停顿时间与GC Roots成正比，GC Roots数量越大，停顿时间越久。再进一步分析，ClassLoader的类名表明，这些ClassLoader均由Aviator组件生成。分析Aviator源码，发现Aviator对每一个表达式新生成类时，会创建一个ClassLoader，这导致了ClassLoader数量巨大的问题。在更高Aviator版本中，该问题已经被修复，即仅创建一个ClassLoader为所有表达式生成类。

解决方法：**升级Aviator组件版本，避免生成多余的ClassLoader**。

**[#](#案例四-服务启动后-运行时间越长-单次gc时间越长-重启后恢复) 案例四：服务启动后，运行时间越长，单次GC时间越长，重启后恢复**

日志信息：观察ZGC日志信息统计，“Pause Roots CodeCache”的耗时会随着服务运行时间逐渐增长。

分析：CodeCache空间用于存放Java热点代码的JIT编译结果，而CodeCache也属于GC Roots一部分。通过添加-XX:+PrintCodeCacheOnCompilation参数，打印CodeCache中的被优化的方法，发现大量的Aviator表达式代码。定位到根本原因，每个表达式都是一个类中一个方法。随着运行时间越长，执行次数增加，这些方法会被JIT优化编译进入到Code Cache中，导致CodeCache越来越大。

解决方法：JIT有一些参数配置可以调整JIT编译的条件，但对于我们的问题都不太适用。我们最终通过业务优化解决，**删除不需要执行的Aviator表达式，从而避免了大量Aviator方法进入CodeCache中**。

#### 升级ZGC效果

##### 延迟降低

TP(Top Percentile)是一项衡量系统延迟的指标：TP999表示99.9%请求都能被响应的最小耗时；TP99表示99%请求都能被响应的最小耗时。

在Zeus服务不同集群中，ZGC在低延迟（TP999 < 200ms）场景中收益较大：

- TP999：下降12-142ms，下降幅度18%-74%。
- TP99：下降5-28ms，下降幅度10%-47%。

超低延迟（TP999 < 20ms）和高延迟（TP999 > 200ms）服务收益不大，原因是这些服务的响应时间瓶颈不是GC，而是外部依赖的性能。

##### 吞吐下降

**对吞吐量优先的场景，ZGC可能并不适合**。例如，Zeus某离线集群原先使用CMS，升级ZGC后，系统吞吐量明显降低。究其原因有二：

- 第一，ZGC是单代垃圾回收器，而CMS是分代垃圾回收器。**单代垃圾回收器每次处理的对象更多，更耗费CPU资源；**
- 第二，**ZGC使用读屏障，读屏障操作需耗费额外的计算资源。**

#### 总结

ZGC作为下一代垃圾回收器，性能非常优秀。ZGC垃圾回收过程**几乎全部是并发**，实际**STW停顿时间极短**，不到10ms。这得益于其采用的**着色指针和读屏障技术**。

### GC - Java 垃圾回收器之CMS GC问题分析与解决

**Mutator**： 生产垃圾的角色，也就是我们的应用程序，垃圾制造者，通过 Allocator 进行 allocate 和 free。

收集器：

![img](https://www.pdai.tech/images/jvm/cmsgc/cms-gc-5.jpg)

目前使用最多的是 CMS 和 G1 收集器，二者都有分代的概念，主要内存结构如下：

![img](https://www.pdai.tech/images/jvm/cmsgc/cms-gc-7.png)

定位分析GC工具：

**命令行终端**

- 标准终端类：jps、jinfo、jstat、jstack、jmap
- 功能整合类：jcmd、vjtools、arthas、greys

**可视化界面**

- 简易：JConsole、JVisualvm、HA、GCHisto、GCViewer
- 进阶：MAT、JProfiler
- 命令行推荐 [arthas]() ，可视化界面推荐 JProfiler，此外还有一些在线的平台 gceasy、heaphero、fastthread ，美团内部的 Scalpel（一款自研的 JVM 问题诊断工具，暂时未开源）也比较好用。

**评判 GC 的两个核心指标：**

- **延迟（Latency）**： 也可以理解为最大停顿时间，即垃圾收集过程中一次 STW 的最长时间，越短越好，一定程度上可以接受频次的增大，GC 技术的主要发展方向。
- **吞吐量（Throughput）**： 应用系统的生命周期内，由于 GC 线程会占用 Mutator 当前可用的 CPU 时钟周期，吞吐量即为 Mutator 有效花费的时间占系统总运行时间的百分比，例如系统运行了 100 min，GC 耗时 1 min，则系统吞吐量为 99%，吞吐量优先的收集器可以接受较长的停顿。

目前各大互联网公司的系统基本都更追求低延时，避免一次 GC 停顿的时间过长对用户体验造成损失，衡量指标需要结合一下应用服务的 SLA，主要如下两点来判断：

![img](https://www.pdai.tech/images/jvm/cmsgc/cms-gc-8.jpg)

简而言之，即为一次停顿的时间不超过应用服务的 TP9999，GC 的吞吐量不小于 99.99%。举个例子，假设某个服务 A 的 TP9999 为 80 ms，平均 GC 停顿为 30 ms，那么该服务的最大停顿时间最好不要超过 80 ms，GC 频次控制在 5 min 以上一次。如果满足不了，那就需要调优或者通过更多资源来进行并联冗余。（大家可以先停下来，看看监控平台上面的 gc.meantime 分钟级别指标，如果超过了 6 ms 那单机 GC 吞吐量就达不到 4 个 9 了。）

**读懂 GC Cause**

拿到 GC 日志，我们就可以简单分析 GC 情况了，通过一些工具，我们可以比较直观地看到 Cause 的分布情况，如下图就是使用 gceasy 绘制的图表：

![img](https://www.pdai.tech/images/jvm/cmsgc/cms-gc-9.png)

重点需要关注的几个GC Cause：

- `System.gc()`： 手动触发GC操作。
- `CMS`： CMS GC 在执行过程中的一些动作，重点关注 CMS Initial Mark 和 CMS Final Remark 两个 STW 阶段。
- `Promotion Failure`： Old 区没有足够的空间分配给 Young 区晋升的对象（即使总可用内存足够大）。
- `Concurrent Mode Failure`： CMS GC 运行期间，Old 区预留的空间不足以分配给新的对象，此时收集器会发生退化，严重影响 GC 性能，下面的一个案例即为这种场景。
- `GCLocker Initiated GC`： 如果线程执行在 JNI 临界区时，刚好需要进行 GC，此时 GC Locker 将会阻止 GC 的发生，同时阻止其他线程进入 JNI 临界区，直到最后一个线程退出临界区时触发一次 GC。



#### GC 问题分类

笔者选取了九种不同类型的 GC 问题，覆盖了大部分场景，如果有更好的场景，欢迎在评论区给出。

- **Unexpected GC**： 意外发生的 GC，实际上不需要发生，我们可以通过一些手段去避免。

  - `Space Shock`： 空间震荡问题，参见“场景一：动态扩容引起的空间震荡”。
  - `Explicit GC`： 显示执行 GC 问题，参见“场景二：显式 GC 的去与留”。

- **Partial GC**： 部分收集操作的 GC，只对某些分代/分区进行回收。

  - ```
    Young GC
    ```

    ： 分代收集里面的 Young 区收集动作，也可以叫做 Minor GC。 

    - `ParNew`： Young GC 频繁，参见“场景四：过早晋升”。

  - ```
    Old GC
    ```

    ： 分代收集里面的 Old 区收集动作，也可以叫做 Major GC，有些也会叫做 Full GC，但其实这种叫法是不规范的，在 CMS 发生 Foreground GC 时才是 Full GC，CMSScavengeBeforeRemark 参数也只是在 Remark 前触发一次Young GC。 

    - CMS： Old GC 频繁，参见“场景五：CMS Old GC 频繁”。
    - CMS： Old GC 不频繁但单次耗时大，参见“场景六：单次 CMS Old GC 耗时长”。

- **Full GC**： 全量收集的 GC，对整个堆进行回收，STW 时间会比较长，一旦发生，影响较大，也可以叫做 Major GC，参见“场景七：内存碎片&收集器退化”。

- **MetaSpace**： 元空间回收引发问题，参见“场景三：MetaSpace 区 OOM”。

- **Direct Memory**： 直接内存（也可以称作为堆外内存）回收引发问题，参见“场景八：堆外内存 OOM”。

- **JNI**： 本地 Native 方法引发问题，参见“场景九：JNI 引发的 GC 问题”。

#### 常见场景分析与解决

**场景一：动态扩容引起的空间震荡**

现象：服务刚刚启动时 GC 次数较多，最大空间剩余很多但是依然发生 GC

原因：在 JVM 的参数中 `-Xms` 和 `-Xmx` 设置的不一致，在初始化时只会初始 `-Xms` 大小的空间存储信息，每当空间不够用时再向操作系统申请，这样的话必然要进行一次 GC。

策略：尽量将**成对出现的空间大小配置参数设置成固定**的，如 `-Xms` 和 `-Xmx`，`-XX:MaxNewSize` 和 `-XX:NewSize`，`-XX:MetaSpaceSize` 和 `-XX:MaxMetaSpaceSize` 等。

**场景二：显式 GC 的去与留**

现象：除了扩容缩容会触发 CMS GC 之外，还有 Old 区达到回收阈值、MetaSpace 空间不足、Young 区晋升失败、大对象担保失败等几种触发条件，如果这些情况都没有发生却触发了 GC ？这种情况有可能是代码中**手动调用了 System.gc** 方法，

**保留 System.gc**：显式触发full-gc ,使用 Foreground Collector 时将会带来非常长的 STW。如果在应用程序中 System.gc 被频繁调用，那就非常危险了

**去掉 System.gc**:如果禁用掉的话就会带来另外一个内存泄漏问题，DirectByteBuffer会使用堆外内存，堆外内存必须要手动释放，它的 Native Memory 的清理工作是通过 `sun.misc.Cleaner` 自动完成的，是一种基于 PhantomReference 的清理工具。

为 DirectByteBuffer 分配空间过程中会显式调用 System.gc ，希望通过 Full GC 来强迫已经无用的 DirectByteBuffer 对象释放掉它们关联的 Native Memory

策略：通过上面的分析看到，无论是保留还是去掉都会有一定的风险点，不过目前互联网中的 RPC 通信会大量使用 NIO，所以笔者在这里建议保留。此外 JVM 还提供了 -XX:+ExplicitGCInvokesConcurrent 和 -XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses 参数来将 System.gc 的触发类型从 Foreground 改为 Background，同时 Background 也会做 Reference Processing，这样的话就能大幅降低了 STW 开销，同时也不会发生 NIO Direct Memory OOM。

 **场景三：MetaSpace 区 OOM**

现象：JVM 在启动后或者某个时间点开始，**MetaSpace 的已使用大小在持续增长，同时每次 GC 也无法释放，调大 MetaSpace 空间也无法彻底解决**。

原因：由场景一可知，为了避免弹性伸缩带来的额外 GC 消耗，我们会将 `-XX:MetaSpaceSize` 和 `-XX:MaxMetaSpaceSize` 两个值设置为固定的，但是这样也会导致在空间不够的时候**无法扩容**，然后**频繁地触发 GC，最终 OOM**。所以关键原因就是 **ClassLoader 不停地在内存中 load 了新的 Class** ，一般这种问题都**发生在动态类加载等情况上**。

**场景四：过早晋升**

现象：分配速率接近于晋升速率，对象晋升年龄较小。

GC 日志中出现`“Desired survivor size 107347968 bytes, new threshold 1(max 6)”`等信息，说明此时经历过一次 GC 就会放到 Old 区。

**Full GC 比较频繁**，且经历过一次 GC 之后 Old 区的变化比例非常大。

过早晋升的危害：

- Young GC 频繁，总的吞吐量下降。
- Full GC 频繁，可能会有较大停顿。

原因：**Young/Eden 区过小**‘、**分配速率过大**： 可以观察出问题前后 Mutator 的分配速率，如果有明显波动可以尝试观察网卡流量、存储类中间件慢查询日志等信息，看是否有大量数据被加载到内存中。

**场景五：CMS Old GC 频繁**

现象：Old 区频繁的做 CMS GC，但是每次耗时不是特别长，整体最大 STW 也在可接受范围内，但由于 GC 太频繁导致吞吐下降比较多。

原因：这种情况比较常见，基本都是一次 Young GC 完成后，负责处理 CMS GC 的一个后台线程 concurrentMarkSweepThread 会不断地轮询，使用 shouldConcurrentCollect() 方法做一次检测，判断是否达到了回收条件。如果达到条件，使用 collect_in_background() 启动一次 Background 模式 GC。

**场景六：单次 CMS Old GC 耗时长**

现象：CMS GC 单次 STW 最大超过 1000ms，不会频繁发生。

原因：CMS 在回收的过程中，STW 的阶段主要是 Init Mark 和 Final Remark 这两个阶段耗时时间长，也是导致 CMS Old GC 最多的原因。

**场景七：内存碎片&收集器退化**

现象：并发的 CMS GC 算法，退化为 Foreground 单线程串行 GC 模式，STW 时间超长，有时会长达十几秒。其中 CMS 收集器退化后单线程串行 GC 算法有两种：

- 带压缩动作的算法，称为 MSC，上面我们介绍过，使用标记-清理-压缩，单线程全暂停的方式，对整个堆进行垃圾收集，也就是真正意义上的 Full GC，暂停时间要长于普通 CMS。
- 不带压缩动作的算法，收集 Old 区，和普通的 CMS 算法比较相似，暂停时间相对 MSC 算法短一些。

**原因：**

**晋升失败**：（是短时间将 Old 区的剩余空间迅速填满，例如上文中说的动态年龄判断导致的过早晋升（见下文的增量收集担保失败）或者内存碎片导致的）。

**显式gc**

**并发模式失败**：这种是由于并发 Background CMS GC 正在执行，同时又有 Young GC 晋升的对象要放入到了 Old 区中，而此时 Old 区空间不足造成的。

 **策略：**

**内存碎片**： 通过配置 -XX:UseCMSCompactAtFullCollection=true 来**控制 Full GC的过程中是否进行空间的整理**（默认开启，注意是Full GC，不是普通CMS GC），以及 -XX: CMSFullGCsBeforeCompaction=n 来控制多少次 Full GC 后进行一次压缩。

**增量收集**： **降低触发 CMS GC 的阈值**，即参数 -XX:CMSInitiatingOccupancyFraction 的值，**让 CMS GC 尽早执行**，以保证有足够的连续空间，也减少 Old 区空间的使用大小，另外需要使用 -XX:+UseCMSInitiatingOccupancyOnly 来配合使用，不然 JVM 仅在第一次使用设定值，后续则自动调整。

**浮动垃圾**： 视情况**控制每次晋升对象的大小**，**或者缩短每次 CMS GC 的时间**，必要时可调节 NewRatio 的值。另外就是使用 -XX:+CMSScavengeBeforeRemark 在过程中**提前触发一次 Young GC，防止后续晋升过多对象**。

**场景八：堆外内存 OOM**

现象：内存使用率不断上升，甚至开始使用 SWAP 内存，同时可能出现 GC 时间飙升，线程被 Block 等现象，通过 top 命令发现 Java 进程的 RES 甚至超过了 -Xmx 的大小。出现这些现象时，基本可以确定是出现了堆外内存泄漏。

原因：1、通过 `UnSafe#allocateMemory，ByteBuffer#allocateDirect` **主动申请了堆外内存而没有释放**，常见于 NIO、Netty 等相关组件。

2、代码中有通过 **JNI 调用 Native Code** 申请的内存没有释放。

**场景九：JNI 引发的 GC 问题**

现象：在 GC 日志中，出现 GC Cause 为 GCLocker Initiated GC。

原因：JNI 如果需要获取 JVM 中的 String 或者数组，有两种方式：

- 拷贝传递。
- 共享引用（指针），性能更高。

由于 Native 代码直接使用了 JVM 堆区的指针，如果这时发生 GC，就会导致数据错误。因此，在发生此类 JNI 调用时，禁止 GC 的发生，同时阻止其他线程进入 JNI 临界区，直到最后一个线程退出临界区时触发一次 GC。

策略：

- 添加 -XX+PrintJNIGCStalls 参数，可以打印出发生 JNI 调用时的线程，进一步分析，找到引发问题的 JNI 调用。
- JNI 调用需要谨慎，不一定可以提升性能，反而可能造成 GC 问题。

#### 处理流程（SOP）

##### 根因鱼骨图

送上一张问题根因鱼骨图，一般情况下我们在处理一个 GC 问题时，只要能定位到问题的“病灶”，有的放矢，其实就相当于解决了 80%，如果在某些场景下不太好定位，大家可以借助这种根因分析图通过排除法去定位。

下图为整体 GC 问题普适的处理流程，重点的地方下面会单独标注，其他的基本都是标准处理流程，此处不再赘述，最后在整个问题都处理完之后有条件的话建议做一下复盘。

![img](https://www.pdai.tech/images/jvm/cmsgc/cms-gc-30.png)

##### 调优建议

- **Trade Off**： 与 CAP 注定要缺一角一样，GC 优化要在延迟（Latency）、吞吐量（Throughput）、容量（Capacity）三者之间进行权衡。
- **最终手段**： GC 发生问题不是一定要对 JVM 的 GC 参数进行调优，大部分情况下是通过 GC 的情况找出一些业务问题，切记上来就对 GC 参数进行调整，当然有明确配置错误的场景除外。
- **控制变量**： 控制变量法是在蒙特卡洛（Monte Carlo）方法中用于减少方差的一种技术方法，我们调优的时候尽量也要使用，每次调优过程尽可能只调整一个变量。
- **善用搜索**： 理论上 99.99% 的 GC 问题基本都被遇到了，我们要学会使用搜索引擎的高级技巧，重点关注 StackOverFlow、Github 上的 Issue、以及各种论坛博客，先看看其他人是怎么解决的，会让解决问题事半功倍。能看到这篇文章，你的搜索能力基本过关了~
- **调优重点**： 总体上来讲，我们开发的过程中遇到的问题类型也基本都符合正态分布，太简单或太复杂的基本遇到的概率很低，笔者这里将中间最重要的三个场景添加了“*”标识，希望阅读完本文之后可以观察下自己负责的系统，是否存在上述问题。
- **GC 参数**： 如果堆、栈确实无法第一时间保留，一定要保留 GC 日志，这样我们最起码可以看到 GC Cause，有一个大概的排查方向。关于 GC 日志相关参数，最基本的 -XX:+HeapDumpOnOutOfMemoryError 等一些参数就不再提了，笔者建议添加以下参数，可以提高我们分析问题的效率。

![img](https://www.pdai.tech/images/jvm/cmsgc/cms-gc-31.png)

### 调试排错

![img](https://www.pdai.tech/images/jvm/java-jvm-debug.png)

### 调试排错 - JVM 调优参数

#### jvm参数

- -Xms

堆最小值

- -Xmx

堆最大堆值。-Xms与-Xmx 的单位默认字节都是以k、m做单位的。

通常这两个配置参数相等，避免每次空间不足，动态扩容带来的影响。

- -Xmn

新生代大小

- -Xss

每个线程池的堆栈大小。在jdk5以上的版本，每个线程堆栈大小为1m，jdk5以前的版本是每个线程池大小为256k。一般在相同物理内存下，如果减少－xss值会产生更大的线程数，但不同的操作系统对进程内线程数是有限制的，是不能无限生成。

- -XX:NewRatio

设置新生代与老年代比值，-XX:NewRatio=4 表示新生代与老年代所占比例为1:4 ，新生代占比整个堆的五分之一。如果设置了-Xmn的情况下，该参数是不需要在设置的。

- -XX:PermSize

设置持久代初始值，默认是物理内存的六十四分之一

- -XX:MaxPermSize

设置持久代最大值，默认是物理内存的四分之一

- -XX:MaxTenuringThreshold

新生代中对象存活次数，默认15。(若对象在eden区，经历一次MinorGC后还活着，则被移动到Survior区，年龄加1。以后，对象每次经历MinorGC，年龄都加1。达到阀值，则移入老年代)

- -XX:SurvivorRatio

Eden区与Subrvivor区大小的比值，如果设置为8，两个Subrvivor区与一个Eden区的比值为2:8，一个Survivor区占整个新生代的十分之一

- -XX:+UseFastAccessorMethods

原始类型快速优化

- -XX:+AggressiveOpts

编译速度加快

- -XX:PretenureSizeThreshold

对象超过多大值时直接在老年代中分配

```text
说明: 
整个堆大小的计算公式: JVM 堆大小 ＝ 年轻代大小＋年老代大小＋持久代大小。
增大新生代大小就会减少对应的年老代大小，设置-Xmn值对系统性能影响较大，所以如果设置新生代大小的调整，则需要严格的测试调整。而新生代是用来存放新创建的对象，大小是随着堆大小增大和减少而有相应的变化，默认值是保持堆大小的十五分之一，-Xmn参数就是设置新生代的大小，也可以通过-XX:NewRatio来设置新生代与年老代的比例，java 官方推荐配置为3:8。

新生代的特点就是内存中的对象更新速度快，在短时间内容易产生大量的无用对象，如果在这个参数时就需要考虑垃圾回收器设置参数也需要调整。推荐使用: 复制清除算法和并行收集器进行垃圾回收，而新生代的垃圾回收叫做初级回收。
StackOverflowError和OutOfMemoryException。当线程中的请求的栈的深度大于最大可用深度，就会抛出前者；若内存空间不够，无法创建新的线程，则会抛出后者。栈的大小直接决定了函数的调用最大深度，栈越大，函数嵌套可调用次数就越多。
```

**经验** :

1. Xmn用于设置新生代的大小。过小会增加Minor GC频率，过大会减小老年代的大小。一般设为整个堆空间的1/4或1/3.
2. XX:SurvivorRatio用于设置新生代中survivor空间(from/to)和eden空间的大小比例； XX:TargetSurvivorRatio表示，当经历Minor GC后，survivor空间占有量(百分比)超过它的时候，就会压缩进入老年代(当然，如果survivor空间不够，则直接进入老年代)。默认值为50%。
3. 为了性能考虑，一开始尽量将新生代对象留在新生代，避免新生的大对象直接进入老年代。因为新生对象大部分都是短期的，这就造成了老年代的内存浪费，并且回收代价也高(Full GC发生在老年代和方法区Perm).
4. 当Xms=Xmx，可以使得堆相对稳定，避免不停震荡
5. 一般来说，MaxPermSize设为64MB可以满足绝大多数的应用了。若依然出现方法区溢出，则可以设为128MB。若128MB还不能满足需求，那么就应该考虑程序优化了，减少**动态类**的产生。

#### 垃圾回收

**垃圾回收算法** :

- 引用计数法: 会有循环引用的问题，古老的方法；
- Mark-Sweep: 标记清除。根可达判断，最大的问题是空间碎片(清除垃圾之后剩下不连续的内存空间)；
- Copying: 复制算法。对于短命对象来说有用，否则需要复制大量的对象，效率低。**如Java的新生代堆空间中就是使用了它(survivor空间的from和to区)；**
- Mark-Compact: 标记整理。对于老年对象来说有用，无需复制，不会产生内存碎片

**GC考虑的指标**

- 吞吐量: 应用耗时和实际耗时的比值；
- 停顿时间: 垃圾回收的时候，由于Stop the World，应用程序的所有线程会挂起，造成应用停顿。

```text
吞吐量和停顿时间是互斥的。
对于后端服务(比如后台计算任务)，吞吐量优先考虑(并行垃圾回收)；
对于前端应用，RT响应时间优先考虑，减少垃圾收集时的停顿时间，适用场景是Web系统(并发垃圾回收)
```

**回收器的JVM参数**

- -XX:+UseSerialGC

串行垃圾回收，现在基本很少使用。

- -XX:+UseParNewGC

新生代使用并行，老年代使用串行；

- -XX:+UseConcMarkSweepGC

新生代使用并行，老年代使用CMS(一般都是使用这种方式)，CMS是Concurrent Mark Sweep的缩写，并发标记清除，一看就是老年代的算法，所以，它可以作为老年代的垃圾回收器。CMS不是独占式的，它关注停顿时间

- -XX:ParallelGCThreads

指定并行的垃圾回收线程的数量，最好等于CPU数量

- -XX:+DisableExplicitGC

禁用System.gc()，因为它会触发Full GC，这是很浪费性能的，JVM会在需要GC的时候自己触发GC。

- -XX:CMSFullGCsBeforeCompaction

在多少次GC后进行内存压缩，这个是因为并行收集器不对内存空间进行压缩的，所以运行一段时间后会产生很多碎片，使得运行效率降低。

- -XX:+CMSParallelRemarkEnabled

降低标记停顿

- -XX:+UseCMSCompactAtFullCollection

在每一次Full GC时对老年代区域碎片整理，因为CMS是不会移动内存的，因此会非常容易出现碎片导致内存不够用的

- -XX:+UseCmsInitiatingOccupancyOnly

使用手动触发或者自定义触发cms 收集，同时也会禁止hostspot 自行触发CMS GC

- -XX:CMSInitiatingOccupancyFraction

使用CMS作为垃圾回收，使用70%后开始CMS收集

- -XX:CMSInitiatingPermOccupancyFraction

设置perm gen使用达到多少％比时触发垃圾回收，默认是92%

- -XX:+CMSIncrementalMode

设置为增量模式

- -XX:+CmsClassUnloadingEnabled

CMS是不会默认对永久代进行垃圾回收的，设置此参数则是开启

- -XX:+PrintGCDetails

开启详细GC日志模式，日志的格式是和所使用的算法有关

- -XX:+PrintGCDateStamps

将时间和日期也加入到GC日志中

### 调试排错 - Java 内存分析之堆内存和MetaSpace内存

#### 常见的内存溢出问题(内存和MetaSpace内存)

##### **Java 堆内存溢出**

Java 堆内存（Heap Memory)主要有两种形式的错误：

1. OutOfMemoryError: Java heap space
2. OutOfMemoryError: GC overhead limit exceeded：GC overhead limt exceed检查是Hotspot VM 1.6定义的一个策略，通过统计GC时间来预测是否要OOM了，提前抛出异常，防止OOM发生。

`-XX:-UseGCOverheadLimit` 设置为false可以禁用这个检查。其实这个参数解决不了内存问题，只是把错误的信息延后，替换成 java.lang.OutOfMemoryError: Java heap space。

##### MetaSpace (元数据) 内存溢出

可以使用 `-XX:MaxMetaspaceSize=10M` 来限制最大元数据。这样当不停的创建类时将会占满该区域并出现 `OOM`。

#### 分析案例

##### 堆内存dump

- **通过OOM获取**

即在OutOfMemoryError后获取一份HPROF二进制Heap Dump文件，在jvm中添加参数：

```bash
-XX:+HeapDumpOnOutOfMemoryError
```

- **主动获取**

在虚拟机添加参数如下，然后在Ctrl+Break组合键即可获取一份Heap Dump

```bash
-XX:+HeapDumpOnCtrlBreak
```

- **使用HPROF agent**

使用Agent可以在程序执行结束时或受到SIGOUT信号时生成Dump文件

配置在虚拟机的参数如下：

```bash
-agentlib:hprof=heap=dump,format=b
```

- **jmap获取** (常用)

jmap可以在cmd里执行，命令如下：

```bash
jmap -dump:format=b file=<文件名XX.hprof> <pid>
```

- **使用JConsole**

Acquire Heap Dump

- **使用JProfile**

Acquire Heap Dump

##### 使用MAT分析内存

### 调试排错 - Java 内存分析之堆外内存

见https://www.pdai.tech/md/java/jvm/java-jvm-oom-offheap.html

#### 排查过程

##### 使用Java层面的工具定位内存区域

使用Java层面的工具可以定位出堆内内存、Code区域或者使用unsafe.allocateMemory和DirectByteBuffer申请的堆外内存

在项目中添加`-XX:NativeMemoryTracking=detailJVM`参数重启项目，使用命令`jcmd pid VM.native_memory detail`查看到的内存分布如下：

![img](https://www.pdai.tech/images/jvm/offheap/jvm-gc-offheap-2.png)

发现命令显示的committed的内存小于物理内存，因为jcmd命令显示的内存包含堆内内存、Code区域、通过unsafe.allocateMemory和DirectByteBuffer申请的内存，但是**不包含其他Native Code（C代码）申请的堆外内存**。所以猜测是使用Native Code申请内存所导致的问题。

为了防止误判，笔者使用了pmap查看内存分布，发现大量的64M的地址；而这些地址空间不在jcmd命令所给出的地址空间里面，基本上就断定就是这些64M的内存所导致。

### 调试排错 - Java 线程分析之线程Dump分析

见https://www.pdai.tech/md/java/jvm/java-jvm-thread-dump.html

#### 线程分析

##### 问题场景

- **CPU飙高，load高，响应很慢**
  1. 一个请求过程中多次dump；
  2. 对比多次dump文件的runnable线程，如果执行的方法有比较大变化，说明比较正常。如果在执行同一个方法，就有一些问题了；
- **查找占用CPU最多的线程**
  1. 使用命令：top -H -p pid（pid为被测系统的进程号），找到导致CPU高的线程ID，对应thread dump信息中线程的nid，只不过一个是十进制，一个是十六进制；
  2. 在thread dump中，根据top命令查找的线程id，查找对应的线程堆栈信息；
- **CPU使用率不高但是响应很慢**

进行dump，查看是否有很多thread struck在了i/o、数据库等地方，定位瓶颈原因；

- **请求无法响应**

多次dump，对比是否所有的runnable线程都一直在执行相同的方法，如果是的，恭喜你，锁住了！

**死锁**

死锁经常表现为程序的停顿，或者不再响应用户的请求。从操作系统上观察，对应进程的CPU占用率为零，很快会从top或prstat的输出中消失。

**热锁**

热锁，也往往是导致系统性能瓶颈的主要因素。其表现特征为：**由于多个线程对临界区，或者锁的竞争**，可能出现：

- **频繁的线程的上下文切换**：从操作系统对线程的调度来看，当线程在等待资源而阻塞的时候，操作系统会将之切换出来，放到等待的队列，当线程获得资源之后，调度算法会将这个线程切换进去，放到执行队列中。
- **大量的系统调用**：因为线程的上下文切换，以及热锁的竞争，或者临界区的频繁的进出，都可能导致大量的系统调用。
- **大部分CPU开销用在“系统态”**：线程上下文切换，和系统调用，都会导致 CPU在 “系统态 ”运行，换而言之，虽然系统很忙碌，但是CPU用在 “用户态 ”的比例较小，应用程序得不到充分的 CPU资源。
- **随着CPU数目的增多，系统的性能反而下降**。因为CPU数目多，同时运行的线程就越多，可能就会造成更频繁的线程上下文切换和系统态的CPU开销，从而导致更糟糕的性能。

#### JVM重要线程

JVM运行过程中产生的一些比较重要的线程罗列如下：

| 线程名称                        | 解释说明                                                     |
| ------------------------------- | ------------------------------------------------------------ |
| Attach Listener                 | Attach Listener 线程是负责接收到外部的命令，而对该命令进行执行的并把结果返回给发送者。通常我们会用一些命令去要求JVM给我们一些反馈信息，如：java -version、jmap、jstack等等。 如果该线程在JVM启动的时候没有初始化，那么，则会在用户第一次执行JVM命令时，得到启动。 |
| Signal Dispatcher               | 前面提到Attach Listener线程的职责是接收外部JVM命令，当命令接收成功后，会交给signal dispather线程去进行分发到各个不同的模块处理命令，并且返回处理结果。signal dispather线程也是在第一次接收外部JVM命令时，进行初始化工作。 |
| CompilerThread0                 | 用来调用JITing，实时编译装卸class 。 通常，JVM会启动多个线程来处理这部分工作，线程名称后面的数字也会累加，例如：CompilerThread1。 |
| Concurrent Mark-Sweep GC Thread | 并发标记清除垃圾回收器（就是通常所说的CMS GC）线程， 该线程主要针对于老年代垃圾回收。ps：启用该垃圾回收器，需要在JVM启动参数中加上：-XX:+UseConcMarkSweepGC。 |
| DestroyJavaVM                   | 执行main()的线程，在main执行完后调用JNI中的 jni_DestroyJavaVM() 方法唤起DestroyJavaVM 线程，处于等待状态，等待其它线程（Java线程和Native线程）退出时通知它卸载JVM。每个线程退出时，都会判断自己当前是否是整个JVM中最后一个非deamon线程，如果是，则通知DestroyJavaVM 线程卸载JVM。 |
| Finalizer Thread                | 这个线程也是在main线程之后创建的，其优先级为10，主要用于在垃圾收集前，调用对象的finalize()方法；关于Finalizer线程的几点：1) 只有当开始一轮垃圾收集时，才会开始调用finalize()方法；因此并不是所有对象的finalize()方法都会被执行；2) 该线程也是daemon线程，因此如果虚拟机中没有其他非daemon线程，不管该线程有没有执行完finalize()方法，JVM也会退出；3) JVM在垃圾收集时会将失去引用的对象包装成Finalizer对象（Reference的实现），并放入ReferenceQueue，由Finalizer线程来处理；最后将该Finalizer对象的引用置为null，由垃圾收集器来回收；4) JVM为什么要单独用一个线程来执行finalize()方法呢？如果JVM的垃圾收集线程自己来做，很有可能由于在finalize()方法中误操作导致GC线程停止或不可控，这对GC线程来说是一种灾难； |
| Low Memory Detector             | 这个线程是负责对可使用内存进行检测，如果发现可用内存低，分配新的内存空间。 |
| Reference Handler               | JVM在创建main线程后就创建Reference Handler线程，其优先级最高，为10，它主要用于处理引用对象本身（软引用、弱引用、虚引用）的垃圾回收问题 。 |
| VM Thread                       | 这个线程就比较牛b了，是JVM里面的线程母体，根据hotspot源码（vmThread.hpp）里面的注释，它是一个单个的对象（最原始的线程）会产生或触发所有其他的线程，这个单个的VM线程是会被其他线程所使用来做一些VM操作（如：清扫垃圾等）。 |

### 什么是Thread Dump

Thread Dump是非常有用的诊断Java应用问题的工具。每一个Java虚拟机都有及时生成所有线程在某一点状态的thread-dump的能力，虽然各个 Java虚拟机打印的thread dump略有不同，但是 大多都提供了**当前活动线程的快照，及JVM中所有Java线程的堆栈跟踪信息**，堆栈信息一般包含完整的类名及所执行的方法，如果可能的话还有源代码的行数。

### Jvm问题排查工具

#### 入门工具

**jps: **jps是jdk提供的一个**查看当前java进程**的小工具， 可以看做是JavaVirtual Machine Process Status Tool的缩写。

**jstack**：jstack是**jdk自带的线程堆栈分析工具**，使用该命令可以查看或导出 Java 应用程序中线程堆栈信息。

**jinfo**：jinfo 是 JDK 自带的命令，可以用来**查看正在运行的 java 应用程序的扩展参数**，包括Java System属性和JVM命令行参数；也可以动态的修改正在运行的 JVM 一些参数。当系统崩溃时，jinfo可以从core文件里面知道崩溃的Java应用程序的配置信息

**jmap**：命令jmap是一个多功能的命令。它可以生成 java 程序的 dump 文件， 也可以查看堆内对象示例的统计信息、查看 ClassLoader 的信息以及 finalizer 队列。

**jstat**:Jstat是java虚拟机统计信息工具，利用JVM内建的指令**对Java应用程序的资源和性能进行实时的命令行的监控**。

```bash
jstat -gcutil 2815 1000 
```

- S0：幸存1区当前使用比例
- S1：幸存2区当前使用比例
- E：伊甸园区使用比例
- O：老年代使用比例
- M：元数据区使用比例
- CCS：压缩使用比例
- YGC：年轻代垃圾回收次数
- FGC：老年代垃圾回收次数
- FGCT：老年代垃圾回收消耗时间
- GCT：垃圾回收消耗总时间

**jdb**：jdb可以用来预发debug,假设你预发的java_home是/opt/java/，远程调试端口是8000.那么jdb -attach 8000 出现以上代表jdb启动成功。后续可以进行设置断点进行调试。

#### Java调试进阶工具

##### **btrace**

**原理**：BTrace是基于动态字节码修改技术（Hotswap）向目标程序的字节码注入追踪代码。

**示例：**

control方法

1. `@GetMapping(value =``"/arg2")`
2. `public``User arg2(User user)``{`
3. `return user;`
4. `}`

BTrace脚本

1. `/**`
2. `* 拦截构造函数`
3. `*/`
4. `@BTrace`
5. `public``class``PrintConstructor``{`
6. `@OnMethod(clazz =``"com.techstar.monitordemo.domain.User", method =``"")`
7. `public``static``void anyRead(@ProbeClassName``String pcn,``@ProbeMethodName``String pmn,``AnyType[] args)``{`
8. `BTraceUtils.println(pcn +``","``+ pmn);`
9. `BTraceUtils.printArray(args);`
10. `BTraceUtils.println();`
11. `}`
12. `}`



拦截结果

1. `192:Btrace apple$ btrace 34119``PrintConstructor.java`
2. `com.techstar.monitordemo.domain.User,`
3. `[1, zuozewei,``]`

**小结**

BTrace是一个事后工具，所谓的事后工具就是在**服务已经上线或者压测后**，但是发现有问题的时候，可以使用BTrace动态跟踪分析。

1. 比如哪些方法执行太慢，例如监控方法执行时间超过1秒的方法；
2. 查看哪些方法调用了system.gc( )，调用栈是怎样的；
3. 查看方法的参数和属性
4. 哪些方法发生了异常

##### greys

**简介**： Greys是一个Java进程的异常诊断工具，可以在不停止程序的前提下，对一些问题进行检测。这个框架主要是采用Java的探针技术，可以做到动态修改java的字节码技术。前提是Jdk版本6+。

![img](https://img2020.cnblogs.com/blog/2112382/202105/2112382-20210518195746637-496610964.png)

- `sc -df xxx`: 输出当前类的详情,包括源码位置和classloader结构
- `trace class method`: 打印出当前方法调用的耗时情况，细分到每个方法, 对排查方法性能时很有帮助。

##### Arthas

**简介**：Arthas 是Alibaba开源的Java诊断工具，深受开发者喜爱。

**当你遇到以下类似问题而束手无策时， Arthas 可以帮助你解决：**

- 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
- 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
- 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
- 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
- 是否有一个全局视角来查看系统的运行状况？
- 有什么办法可以监控到JVM的实时运行状态？
- 怎么快速定位应用的热点，生成火焰图？

Arthas 支持JDK 6+，支持Linux/Mac/Winodws，采用命令行交互模式，同时提供丰富的 Tab 自动补
全功能，进一步方便进行问题的定位和诊断。

**monitor**：**监控方法的执行情况**，用来监视一个时间段中指定方法的执行次数，成功次数，失败次数，耗时等这些信息。

监控`demo.MathGame`类，并且每5S更新一次状态。

```bash
monitor demo.MathGame primeFactors -c 5
```

**watch：检测函数返回值**

> 方法执行数据观测，让你能方便的观察到指定方法的调用情况。
>
> 能观察到的范围为：`返回值`、`抛出异常`、`入参`，通过编写OGNL 表达式进行对应变量的查看。

**trace：根据路径追踪，并记录消耗时间**

对方法内部调用路径进行追踪，并输出方法路径上的每个节点上耗时。

**`stack`：输出当前方法被调用的调用路径**

输出当前方法被调用的调用路径

很多时候我们都知道一个方法被执行，但这个方法被执行的路径非常多，或者你根本就不知道这个方法是从那里被执行了，此时你需要的是 stack 命令。

**`tt`：时间隧道，记录多个请求**

> time-tunnel 时间隧道。
>
> 记录下指定方法每次调用的入参和返回信息，并能对这些不同时间下调用的信息进行观测

案例：

```
#	最基本的使用来说，就是记录下当前方法的每次调用环境现场。
tt -t demo.MathGame primeFactors
```

![请添加图片描述](https://img-blog.csdnimg.cn/93e068a0cad54ab3afc2ac04a59dae85.png)

**项目使用**

**`trace`：查询最耗时应用**

```shell
#	在浏览器上进行登录操作，检查最耗时的方法
trace *.DispatcherServlet *
```

**`jad`：反编译耗时代码**

```shell
#	trace结果里把调用的行号打印出来了，我们可以直接在IDE里查看代码（也可以用jad命令反编译）
jad --source-only *.DispatcherServlet doDispatch
```

**`watch`：捕获耗时应用入参、返回值**

```
watch com.itleima.controller.* * {params,returnObj} -x 2
```

**基础命令**

![image-20240308151447448](C:\Users\1\Desktop\JAVA全栈知识体系\JAVA 全栈知识体系.assets\image-20240308151447448.png)

##### javOSize

就说一个功能:

- `classes`：通过修改了字节码，改变了类的内容，即时生效。 所以可以做到快速的在某个地方打个日志看看输出，缺点是对代码的侵入性太大。但是如果自己知道自己在干嘛，的确是不错的玩意儿。

##### dmesg

如果发现自己的java进程悄无声息的消失了，几乎没有留下任何线索，那么dmesg一发，很有可能有你想要的。

sudo dmesg|grep -i kill|less 去找关键字oom_killer。找到的结果类似如下:

```bash
[6710782.021013] java invoked oom-killer: gfp_mask=0xd0, order=0, oom_adj=0, oom_scoe_adj=0
[6710782.070639] [<ffffffff81118898>] ? oom_kill_process+0x68/0x140 
[6710782.257588] Task in /LXC011175068174 killed as a result of limit of /LXC011175068174 
[6710784.698347] Memory cgroup out of memory: Kill process 215701 (java) score 854 or sacrifice child 
[6710784.707978] Killed process 215701, UID 679, (java) total-vm:11017300kB, anon-rss:7152432kB, file-rss:1232kB
```

以上表明，对应的java进程被系统的OOM Killer给干掉了，得分为854. 解释一下OOM killer（Out-Of-Memory killer），该机制会监控机器的内存资源消耗。当机器内存耗尽前，该机制会扫描所有的进程（按照一定规则计算，内存占用，时间等），挑选出得分最高的进程，然后杀死，从而保护机器。

#### 可视化工具

**Jconsole** ：Jconsole （Java Monitoring and Management Console），JDK自带的基于JMX的可视化监视、管理工具。 

**Visual VM**：VisualVM 是一款免费的，集成了多个 JDK 命令行工具的可视化工具，它能为您提供强大的分析能力，对 Java 应用程序做**性能分析和调优**。这些功能包括**生成和分析海量数据、跟踪内存泄漏、监控垃圾回收器、执行内存和 CPU 分析，同时它还支持在 MBeans 上进行浏览和操作**。

**Visual GC**：visual gc 是 visualvm 中的图形化查看 gc 状况的插件。

**JProfile**：Profiler 是一个商业的主要用于检查和跟踪系统（限于Java开发的）的性能的工具。JProfiler可以通过时时的监控系统的内存使用情况，随时监视垃圾回收，线程运行状况等手段，从而很好的监视JVM运行情况及其性能。

**Eclipse Memory Analyzer (MAT)**：MAT 是一种快速且功能丰富的 Java 堆分析器，可帮助你发现内存泄漏并减少内存消耗。 MAT在的堆内存分析问题使用极为广泛，需要重点掌握。

**Arthas**： 是Alibaba开源的Java诊断工具



## Spring

### Spring基础 - Spring和Spring框架组成



#### Spring的特性和优势

从Spring 框架的**特性**来看：

- 非侵入式：基于Spring开发的应用中的对象可以不依赖于Spring的API
- 控制反转：IOC——Inversion of Control，指的是将对象的创建权交给 Spring 去创建。使用 Spring 之前，对象的创建都是由我们自己在代码中new创建。而使用 Spring 之后。对象的创建都是给了 Spring 框架。
- 依赖注入：DI——Dependency Injection，是指依赖的对象不需要手动调用 setXX 方法去设置，而是通过配置赋值。
- 面向切面编程：Aspect Oriented Programming——AOP
- 容器：Spring 是一个容器，因为它包含并且管理应用对象的生命周期
- 组件化：Spring 实现了使用简单的组件配置组合成一个复杂的应用。在 Spring 中可以使用XML和Java注解组合这些对象。
- 一站式：在 IOC 和 AOP 的基础上可以整合各种企业应用的开源框架和优秀的第三方类库（实际上 Spring 自身也提供了表现层的 SpringMVC 和持久层的 Spring JDBC）

#### Spring有哪些组件?

Spring Framwork5.0 M4 版本图

![img](https://www.pdai.tech/images/spring/springframework/spring-framework-introduce-8.png)

##### Core Container（Spring的核心容器）

Spring 的核心容器是其他模块建立的基础，由 Beans 模块、Core 核心模块、Context 上下文模块和 SpEL 表达式语言模块组成，没有这些核心容器，也不可能有 AOP、Web 等上层的功能。具体介绍如下。

- **Beans 模块**：提供了框架的基础部分，包括控制反转和依赖注入。
- **Core 核心模块**：封装了 Spring 框架的底层部分，包括资源访问、类型转换及一些常用工具类。
- **Context 上下文模块**：建立在 Core 和 Beans 模块的基础之上，集成 Beans 模块功能并添加资源绑定、数据验证、国际化、Java EE 支持、容器生命周期、事件传播等。ApplicationContext 接口是上下文模块的焦点。
- **SpEL 模块**：提供了强大的表达式语言支持，支持访问和修改属性值，方法调用，支持访问及修改数组、容器和索引器，命名变量，支持算数和逻辑运算，支持从 Spring 容器获取 Bean，它也支持列表投影、选择和一般的列表聚合等。



##### Data Access/Integration（数据访问／集成）

数据访问／集成层包括 JDBC、ORM、OXM、JMS 和 Transactions 模块，具体介绍如下。

- **JDBC 模块**：提供了一个 JDBC 的样例模板，使用这些模板能消除传统冗长的 JDBC 编码还有必须的事务控制，而且能享受到 Spring 管理事务的好处。
- **ORM 模块**：提供与流行的“对象-关系”映射框架无缝集成的 API，包括 JPA、JDO、Hibernate 和 MyBatis 等。而且还可以使用 Spring 事务管理，无需额外控制事务。
- **OXM 模块**：提供了一个支持 Object /XML 映射的抽象层实现，如 JAXB、Castor、XMLBeans、JiBX 和 XStream。将 Java 对象映射成 XML 数据，或者将XML 数据映射成 Java 对象。
- **JMS 模块**：指 Java 消息服务，提供一套 “消息生产者、消息消费者”模板用于更加简单的使用 JMS，JMS 用于用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。
- **Transactions 事务模块**：支持编程和声明式事务管理。



##### Web模块

Spring 的 Web 层包括 Web、Servlet、WebSocket 和 Webflux 组件，具体介绍如下。

- **Web 模块**：提供了基本的 Web 开发集成特性，例如多文件上传功能、使用的 Servlet 监听器的 IOC 容器初始化以及 Web 应用上下文。
- **Servlet 模块**：提供了一个 Spring MVC Web 框架实现。Spring MVC 框架提供了基于注解的请求资源注入、更简单的数据绑定、数据验证等及一套非常易用的 JSP 标签，完全无缝与 Spring 其他技术协作。
- **WebSocket 模块**：提供了简单的接口，用户只要实现响应的接口就可以快速的搭建 WebSocket Server，从而实现双向通讯。
- **Webflux 模块**： Spring WebFlux 是 Spring Framework 5.x中引入的新的响应式web框架。与Spring MVC不同，它不需要Servlet API，是完全异步且非阻塞的，并且通过Reactor项目实现了Reactive Streams规范。Spring WebFlux 用于创建基于事件循环执行模型的完全异步且非阻塞的应用程序。

此外Spring4.x中还有Portlet 模块，在Spring 5.x中已经移除

- **Portlet 模块**：提供了在 Portlet 环境中使用 MVC 实现，类似 Web-Servlet 模块的功能。



##### AOP、Aspects、Instrumentation和Messaging

在 Core Container 之上是 AOP、Aspects 等模块，具体介绍如下：

- **AOP 模块**：提供了面向切面编程实现，提供比如日志记录、权限控制、性能统计等通用功能和业务逻辑分离的技术，并且能动态的把这些功能添加到需要的代码中，这样各司其职，降低业务逻辑和通用功能的耦合。
- **Aspects 模块**：提供与 AspectJ 的集成，是一个功能强大且成熟的面向切面编程（AOP）框架。
- **Instrumentation 模块**：提供了类工具的支持和类加载器的实现，可以在特定的应用服务器中使用。
- **messaging 模块**：Spring 4.0 以后新增了消息（Spring-messaging）模块，该模块提供了对消息传递体系结构和协议的支持。
- **jcl 模块**： Spring 5.x中新增了日志框架集成的模块。

##### Test模块

Test 模块：Spring 支持 Junit 和 TestNG 测试框架，而且还额外提供了一些基于 Spring 的测试功能，比如在测试 Web 框架时，模拟 Http 请求的功能。

包含Mock Objects, TestContext Framework, Spring MVC Test, WebTestClient。

### Spring基础 - Spring简单例子引入Spring要点

#### 控制反转 - IOC

1. Spring框架管理这些Bean的创建工作，即由用户管理Bean转变为框架管理Bean，这个就叫**控制反转 - Inversion of Control (IoC)**
2. Spring 框架托管创建的Bean放在哪里呢？ 这便是**IoC Container**;
3. Spring 框架为了更好让用户配置Bean，必然会引入**不同方式来配置Bean？ 这便是xml配置，Java配置，注解配置**等支持
4. Spring 框架既然接管了Bean的生成，必然需要**管理整个Bean的生命周期**等；
5. 应用程序代码从Ioc Container中获取依赖的Bean，注入到应用程序中，这个过程叫 **依赖注入(Dependency Injection，DI)** ； 所以说控制反转是通过依赖注入实现的，其实它们是同一个概念的不同角度描述。通俗来说就是**IoC是设计思想，DI是实现方式**
6. 在依赖注入时，有哪些方式呢？这就是构造器方式，@Autowired, @Resource, @Qualifier... 同时Bean之间存在依赖（可能存在先后顺序问题，以及**循环依赖问题**等）

#### 面向切面 - AOP

1. Spring 框架通过定义切面, 通过拦截切点实现了不同业务模块的解耦，这个就叫**面向切面编程 - Aspect Oriented Programming (AOP)**
2. 为什么@Aspect注解使用的是aspectj的jar包呢？这就引出了**Aspect4J和Spring AOP的历史渊源**，只有理解了Aspect4J和Spring的渊源才能理解有些注解上的兼容设计
3. 如何支持**更多拦截方式**来实现解耦， 以满足更多场景需求呢？ 这就是@Around, @Pointcut... 等的设计
4. 那么Spring框架又是如何实现AOP的呢？ 这就引入**代理技术，分静态代理和动态代理**，动态代理又包含JDK代理和CGLIB代理等

#### XML配置方式

```java
package tech.pdai.springframework;

import java.util.List;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import tech.pdai.springframework.entity.User;
import tech.pdai.springframework.service.UserServiceImpl;

/**
 * @author pdai
 */
public class App {

    /**
     * main interfaces.
     *
     * @param args args
     */
    public static void main(String[] args) {
        // create and configure beans
        ApplicationContext context =
                new ClassPathXmlApplicationContext("aspects.xml", "daos.xml", "services.xml");

        // retrieve configured instance
        UserServiceImpl service = context.getBean("userService", UserServiceImpl.class);

        // use configured instance
        List<User> userList = service.findUserList();

        // print info from beans
        userList.forEach(a -> System.out.println(a.getName() + "," + a.getAge()));
    }
}
```



#### Java 配置方式改造

在前文的例子中， 通过xml配置方式实现的，这种方式实际上比较麻烦； 我通过Java配置进行改造：

- User，UserDaoImpl, UserServiceImpl，LogAspect不用改
- 将原通过.xml配置转换为Java配置

```java
package tech.pdai.springframework.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import tech.pdai.springframework.aspect.LogAspect;
import tech.pdai.springframework.dao.UserDaoImpl;
import tech.pdai.springframework.service.UserServiceImpl;

/**
 * @author pdai
 */
@EnableAspectJAutoProxy
@Configuration
public class BeansConfig {

    /**
     * @return user dao
     */
    @Bean("userDao")
    public UserDaoImpl userDao() {
        return new UserDaoImpl();
    }

    /**
     * @return user service
     */
    @Bean("userService")
    public UserServiceImpl userService() {
        UserServiceImpl userService = new UserServiceImpl();
        userService.setUserDao(userDao());
        return userService;
    }

    /**
     * @return log aspect
     */
    @Bean("logAspect")
    public LogAspect logAspect() {
        return new LogAspect();
    }
}
```

- **在App中加载BeansConfig的配置**

```java
package tech.pdai.springframework;

import java.util.List;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import tech.pdai.springframework.config.BeansConfig;
import tech.pdai.springframework.entity.User;
import tech.pdai.springframework.service.UserServiceImpl;

/**
 * @author pdai
 */
public class App {

    /**
     * main interfaces.
     *
     * @param args args
     */
    public static void main(String[] args) {
        // create and configure beans
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BeansConfig.class);

        // retrieve configured instance
        UserServiceImpl service = context.getBean("userService", UserServiceImpl.class);

        // use configured instance
        List<User> userList = service.findUserList();

        // print info from beans
        userList.forEach(a -> System.out.println(a.getName() + "," + a.getAge()));
    }
}
```

### 注解配置方式改造

更进一步，Java 5开始提供注解支持，Spring 2.5 开始完全支持基于注解的配置并且也支持JSR250 注解。在Spring后续的版本发展倾向于通过注解和Java配置结合使用.

- **BeanConfig 不再需要Java配置**

- **UserDaoImpl 增加了 @Repository注解**

```java
/**
 * @author pdai
 */
@Repository
public class UserDaoImpl {

    /**
     * mocked to find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return Collections.singletonList(new User("pdai", 18));
    }
}
```

- **UserServiceImpl 增加了@Service 注解，并通过@Autowired注入userDao**.

```java
/**
 * @author pdai
 */
@Service
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    @Autowired
    private UserDaoImpl userDao;

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return userDao.findUserList();
    }

}
```

- **在App中扫描tech.pdai.springframework包**

```java
package tech.pdai.springframework;

import java.util.List;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import tech.pdai.springframework.entity.User;
import tech.pdai.springframework.service.UserServiceImpl;

/**
 * @author pdai
 */
public class App {

    /**
     * main interfaces.
     *
     * @param args args
     */
    public static void main(String[] args) {
        // create and configure beans
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext("tech.pdai.springframework");

        // retrieve configured instance
        UserServiceImpl service = context.getBean(UserServiceImpl.class);

        // use configured instance
        List<User> userList = service.findUserList();

        // print info from beans
        userList.forEach(a -> System.out.println(a.getName() + "," + a.getAge()));
    }
}
```

### Spring基础 - Spring核心之控制反转(IOC)

**Spring Bean是什么**:**Spring bean是Spring框架中由IoC容器管理的对象**。它们是应用程序的基本构建模块，通常包含应用程序的主要业务逻辑。Spring bean可以是任何Java对象，根据bean规范编写的类，由Spring IOC容器实例化、组装和管理。这些bean负责维护自己的状态，并且可以通过依赖注入来管理它们之间的依赖关系。

控制反转是通过依赖注入实现的，其实它们是同一个概念的不同角度描述。通俗来说就是**IoC是设计思想，DI是实现方式**。

#### Ioc 配置的三种方式

##### xml 配置

顾名思义，就是将bean的信息配置.xml文件里，通过Spring加载文件为我们创建bean。这种方式出现很多早前的SSM项目中，将第三方类库或者一些配置工具类都以这种方式进行配置，主要原因是由于第三方类不支持Spring注解。

- **优点**： 可以使用于任何场景，结构清晰，通俗易懂
- **缺点**： 配置繁琐，不易维护，枯燥无味，扩展性差

##### Java 配置

将类的创建交给我们配置的JavcConfig类来完成，Spring只负责维护和管理，采用纯Java创建方式。其本质上就是把在XML上的配置声明转移到Java配置类中

- **优点**：适用于任何场景，配置方便，因为是纯Java代码，扩展性高，十分灵活
- **缺点**：由于是采用Java类的方式，声明不明显，如果大量配置，可读性比较差

##### 注解配置

通过在类上加注解的方式，来声明一个类交给Spring管理，Spring会自动扫描带有@Component，@Controller，@Service，@Repository这四个注解的类，然后帮我们创建并管理，前提是需要先配置Spring的注解扫描器。

- **优点**：开发便捷，通俗易懂，方便维护。
- **缺点**：具有局限性，对于一些第三方资源，无法添加注解。只能采用XML或JavaConfig的方式配置



**依赖注入三种方式：**setter方式，构造函数，注解注入

**AOP配置方式：** xml方式、Aspectj注解方式

**@Autowired和@Resource以及@Inject等注解注入有何区别？**

1、@Autowired是Spring自带的，@Resource是JSR250规范实现的，@Inject是JSR330规范实现的

2、@Autowired、@Inject用法基本一样，不同的是@Inject没有required属性

3、@Autowired、@Inject是默认按照类型匹配的，@Resource是按照名称匹配的

4、@Autowired如果需要按照名称匹配需要和@Qualifier一起使用；@Inject和@Named一起使用；@Resource则通过name进行指定

5、@Autowired可以作用在CONSTRUCTOR( 构造函数)、METHOD(方法)、PARAMETER(方法参数)、FIELD(字段、枚举的常量)、ANNOTATION_TYPE(注解)

6、@Resource可以作用TYPE(接口、类、枚举、注解)、FIELD(字段、枚举的常量)、METHOD(方法)上

7、@Inject可以作用CONSTRUCTOR( 构造函数)、METHOD(方法)、FIELD(字段、枚举的常量)上

### Spring基础 - Spring核心之面向切面编程(AOP)

#### **如何理解AOP？**

AOP的本质也是为了**解耦**，它是一种**设计思想**

**AOP是什么？**AOP最早是AOP联盟的组织提出的,指定的一套规范,spring将AOP的思想引入框架之中,通过**预编译方式**和**运行期间动态代理**实现程序的统一维护的一种技术,

**AOP术语**

**连接点（Jointpoint）**：表示需要在程序中插入横切关注点的扩展点，**连接点可能是类初始化、方法执行、方法调用、字段调用或处理异常等等**，Spring只支持方法执行连接点，在AOP中表示为**在哪里干**；

**切入点（Pointcut）**： 选择一组相关连接点的模式，即可以认为连接点的集合，Spring支持perl5正则表达式和AspectJ切入点模式，Spring默认使用AspectJ语法，在AOP中表示为**在哪里干的集合**；

**通知（Advice）**：在连接点上执行的行为，通知提供了在AOP中需要在切入点所选择的连接点处进行扩展现有行为的手段；包括前置通知（before advice）、后置通知(after advice)、环绕通知（around advice），在Spring中通过代理模式实现AOP，并通过拦截器模式以环绕连接点的拦截器链织入通知；在AOP中表示为**干什么**；

**方面/切面（Aspect）**：横切关注点的模块化，比如上边提到的日志组件。可以认为是通知、引入和切入点的组合；在Spring中可以使用Schema和@AspectJ方式进行组织实现；在AOP中表示为**在哪干和干什么集合**；

**引入（inter-type declaration）**：也称为内部类型声明，为已有的类添加额外新的字段或方法，Spring允许引入新的接口（必须对应一个实现）到所有被代理对象（目标对象）, 在AOP中表示为**干什么（引入什么）**；

**目标对象（Target Object）**：需要被织入横切关注点的对象，即该对象是切入点选择的对象，需要被通知的对象，从而也可称为被通知对象；由于Spring AOP 通过代理模式实现，从而这个对象永远是被代理对象，在AOP中表示为**对谁干**；

**织入（Weaving）**：把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时（例如使用AspectJ编译器），类加载时和运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入。在AOP中表示为**怎么实现的**；（动态织入和静态织入）**AspectJ应用到java代码的过程（这个过程称为织入）**

**AOP代理（AOP Proxy）**：AOP框架使用代理模式创建的对象，从而实现在连接点处插入通知（即应用切面），就是通过代理来对目标对象应用切面。在Spring中，AOP代理可以用JDK动态代理或CGLIB代理实现，而通过拦截器模型应用切面。在AOP中表示为**怎么实现的一种典型方式**；

**通知类型**

**前置通知（Before advice）**：在某连接点之前执行的通知，但这个通知不能阻止连接点之前的执行流程（除非它抛出一个异常）。

**后置通知（After returning advice）**：在某连接点正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。

**异常通知（After throwing advice）**：在方法抛出异常退出时执行的通知。

**最终通知（After (finally) advice）**：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。

**环绕通知（Around Advice）**：包围一个连接点的通知，如方法调用。这是最强大的一种通知类型。环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它自己的返回值或抛出异常来结束执行。

![img](https://www.pdai.tech/images/spring/springframework/spring-framework-aop-3.png)

**Spring AOP和AspectJ是什么关系**

1. AspectJ是更强的AOP框架，是实际意义的**AOP标准**；

了解**AspectJ应用到java代码的过程**（这个过程称为织入），对于织入这个概念，可以简单理解为aspect(切面)应用到目标函数(类)的过程。

对于这个过程，一般分为**动态织入**和**静态织入**：

1. **动态织入的方式是在运行时动态将要增强的代码织入到目标类中**，这样往往是通过动态代理技术完成的，如Java JDK的动态代理(Proxy，底层通过反射实现)或者CGLIB的动态代理(底层通过继承实现)，Spring AOP采用的就是基于运行时增强的代理技术
2. **ApectJ采用的就是静态织入的方式**。ApectJ主要采用的是**编译期织入**，在这个期间使用AspectJ的acj编译器(类似javac)把aspect类编译成class字节码后，在java目标类编译时织入，即先编译aspect类再编译目标类。

#### AOP的配置方式

##### **XML Schema配置方式**

- **定义切面类**

```java
package tech.pdai.springframework.aspect;

import org.aspectj.lang.ProceedingJoinPoint;

/**
 * @author pdai
 */
public class LogAspect {

    /**
     * 环绕通知.
     *
     * @param pjp pjp
     * @return obj
     * @throws Throwable exception
     */
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("-----------------------");
        System.out.println("环绕通知: 进入方法");
        Object o = pjp.proceed();
        System.out.println("环绕通知: 退出方法");
        return o;
    }

    /**
     * 前置通知.
     */
    public void doBefore() {
        System.out.println("前置通知");
    }

    /**
     * 后置通知.
     *
     * @param result return val
     */
    public void doAfterReturning(String result) {
        System.out.println("后置通知, 返回值: " + result);
    }

    /**
     * 异常通知.
     *
     * @param e exception
     */
    public void doAfterThrowing(Exception e) {
        System.out.println("异常通知, 异常: " + e.getMessage());
    }

    /**
     * 最终通知.
     */
    public void doAfter() {
        System.out.println("最终通知");
    }

}
```

- **XML配置AOP**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd
 http://www.springframework.org/schema/aop
 http://www.springframework.org/schema/aop/spring-aop.xsd
 http://www.springframework.org/schema/context
 http://www.springframework.org/schema/context/spring-context.xsd
">

    <context:component-scan base-package="tech.pdai.springframework" />

    <aop:aspectj-autoproxy/>

    <!-- 目标类 -->
    <bean id="demoService" class="tech.pdai.springframework.service.AopDemoServiceImpl">
        <!-- configure properties of bean here as normal -->
    </bean>

    <!-- 切面 -->
    <bean id="logAspect" class="tech.pdai.springframework.aspect.LogAspect">
        <!-- configure properties of aspect here as normal -->
    </bean>

    <aop:config>
        <!-- 配置切面 -->
        <aop:aspect ref="logAspect">
            <!-- 配置切入点 -->
            <aop:pointcut id="pointCutMethod" expression="execution(* tech.pdai.springframework.service.*.*(..))"/>
            <!-- 环绕通知 -->
            <aop:around method="doAround" pointcut-ref="pointCutMethod"/>
            <!-- 前置通知 -->
            <aop:before method="doBefore" pointcut-ref="pointCutMethod"/>
            <!-- 后置通知；returning属性：用于设置后置通知的第二个参数的名称，类型是Object -->
            <aop:after-returning method="doAfterReturning" pointcut-ref="pointCutMethod" returning="result"/>
            <!-- 异常通知：如果没有异常，将不会执行增强；throwing属性：用于设置通知第二个参数的的名称、类型-->
            <aop:after-throwing method="doAfterThrowing" pointcut-ref="pointCutMethod" throwing="e"/>
            <!-- 最终通知 -->
            <aop:after method="doAfter" pointcut-ref="pointCutMethod"/>
        </aop:aspect>
    </aop:config>

    <!-- more bean definitions for data access objects go here -->
</beans>
```

##### AspectJ注解方式

| 注解名称        | 解释                                                         |
| --------------- | ------------------------------------------------------------ |
| @Aspect         | 用来定义一个切面。                                           |
| @pointcut       | 用于定义切入点表达式。在使用时还需要定义一个包含名字和任意参数的方法签名来表示切入点名称，这个方法签名就是一个返回值为void，且方法体为空的普通方法。 |
| @Before         | 用于定义前置通知，相当于BeforeAdvice。在使用时，通常需要指定一个value属性值，该属性值用于指定一个切入点表达式(可以是已有的切入点，也可以直接定义切入点表达式)。 |
| @AfterReturning | 用于定义后置通知，相当于AfterReturningAdvice。在使用时可以指定pointcut / value和returning属性，其中pointcut / value这两个属性的作用一样，都用于指定切入点表达式。 |
| @Around         | 用于定义环绕通知，相当于MethodInterceptor。在使用时需要指定一个value属性，该属性用于指定该通知被植入的切入点。 |
| @After-Throwing | 用于定义异常通知来处理程序中未处理的异常，相当于ThrowAdvice。在使用时可指定pointcut / value和throwing属性。其中pointcut/value用于指定切入点表达式，而throwing属性值用于指定-一个形参名来表示Advice方法中可定义与此同名的形参，该形参可用于访问目标方法抛出的异常。 |
| @After          | 用于定义最终final 通知，不管是否异常，该通知都会执行。使用时需要指定一个value属性，该属性用于指定该通知被植入的切入点。 |
| @DeclareParents | 用于定义引介通知，相当于IntroductionInterceptor (不要求掌握)。 |

**使用JDK代理**

- **定义切面**

```java
package tech.pdai.springframework.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.stereotype.Component;

/**
 * @author pdai
 */
@EnableAspectJAutoProxy
@Component
@Aspect
public class LogAspect {

    /**
     * define point cut.
     */
    @Pointcut("execution(* tech.pdai.springframework.service.*.*(..))")
    private void pointCutMethod() {
    }


    /**
     * 环绕通知.
     *
     * @param pjp pjp
     * @return obj
     * @throws Throwable exception
     */
    @Around("pointCutMethod()")
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("-----------------------");
        System.out.println("环绕通知: 进入方法");
        Object o = pjp.proceed();
        System.out.println("环绕通知: 退出方法");
        return o;
    }

    /**
     * 前置通知.
     */
    @Before("pointCutMethod()")
    public void doBefore() {
        System.out.println("前置通知");
    }


    /**
     * 后置通知.
     *
     * @param result return val
     */
    @AfterReturning(pointcut = "pointCutMethod()", returning = "result")
    public void doAfterReturning(String result) {
        System.out.println("后置通知, 返回值: " + result);
    }

    /**
     * 异常通知.
     *
     * @param e exception
     */
    @AfterThrowing(pointcut = "pointCutMethod()", throwing = "e")
    public void doAfterThrowing(Exception e) {
        System.out.println("异常通知, 异常: " + e.getMessage());
    }

    /**
     * 最终通知.
     */
    @After("pointCutMethod()")
    public void doAfter() {
        System.out.println("最终通知");
    }

}
```

#### AOP使用小结

##### 切入点（pointcut）的申明规则

ret-type-pattern 返回类型模式, name-pattern名字模式和param-pattern参数模式是必选的， 其它部分都是可选的。返回类型模式决定了方法的返回类型必须依次匹配一个连接点。 你会使用的最频繁的返回类型模式是`*`，**它代表了匹配任意的返回类型**。

declaring-type-pattern, 一个全限定的类型名将只会匹配返回给定类型的方法。

name-pattern 名字模式匹配的是方法名。 你可以使用`*`通配符作为所有或者部分命名模式。

param-pattern 参数模式稍微有点复杂：()匹配了一个不接受任何参数的方法， 而(..)匹配了一个接受任意数量参数的方法（零或者更多）。 模式(*)匹配了一个接受一个任何类型的参数的方法。 模式(*,String)匹配了一个接受两个参数的方法，第一个可以是任意类型， 第二个则必须是String类型。

![img](https://www.pdai.tech/images/spring/springframework/spring-framework-aop-7.png)

**如果有多个通知想要在同一连接点运行会发生什么？**Spring AOP遵循跟AspectJ一样的优先规则来确定通知执行的顺序。

 在“进入”连接点的情况下，**最高优先级的通知会先执行**（所以给定的两个前置通知中，优先级高的那个会先执行）。

 在“退出”连接点的情况下，**最高优先级的通知会最后执行**。（所以给定的两个后置通知中， 优先级高的那个会第二个执行）。

##### Spring AOP 和 AspectJ 之间的关键区别？

AspectJ可以做Spring AOP干不了的事情，**它是AOP编程的完全解决方案，Spring AOP则致力于解决企业级开发中最普遍的AOP**（方法织入）。

下表总结了 Spring AOP 和 AspectJ 之间的关键区别:

| Spring AOP                                       | AspectJ                                                      |
| ------------------------------------------------ | ------------------------------------------------------------ |
| 在纯 Java 中实现                                 | 使用 Java 编程语言的扩展实现                                 |
| 不需要单独的编译过程                             | 除非设置 LTW，否则需要 AspectJ 编译器 (ajc)                  |
| 只能使用运行时织入                               | 运行时织入不可用。支持编译时、编译后和加载时织入             |
| 功能不强-仅支持方法级编织                        | 更强大 - 可以编织字段、方法、构造函数、静态初始值设定项、最终类/方法等......。 |
| 只能在由 Spring 容器管理的 bean 上实现           | 可以在所有域对象上实现                                       |
| 仅支持方法执行切入点                             | 支持所有切入点                                               |
| 代理是由目标对象创建的, 并且切面应用在这些代理上 | 在执行应用程序之前 (在运行时) 前, 各方面直接在代码中进行织入 |
| 比 AspectJ 慢多了                                | 更好的性能                                                   |
| 易于学习和应用                                   | 相对于 Spring AOP 来说更复杂                                 |

##### Spring AOP还是完全用AspectJ？

以下Spring官方的回答：（总结来说就是 **Spring AOP更易用，AspectJ更强大**）。

- Spring AOP比完全使用AspectJ更加简单， 因为它不需要引入AspectJ的编译器／织入器到你开发和构建过程中。 如果你**仅仅需要在Spring bean上通知执行操作，那么Spring AOP是合适的选择**。
- 如果你需要通知domain对象或其它没有在Spring容器中管理的任意对象，那么你需要使用AspectJ。
- 如果你想通知除了简单的方法执行之外的连接点（如：调用连接点、字段get或set的连接点等等）， 也需要使用AspectJ。

### Spring基础 - SpringMVC请求流程和案例

**什么是MVC？**

MVC英文是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计规范。本质上也是一种解耦。

**Model**（模型）是应用程序中用于处理应用程序数据逻辑的部分。通常模型对象负责在数据库中存取数据。

**View**（视图）是应用程序中处理数据显示的部分。通常视图是依据模型数据创建的。

**Controller**（控制器）是应用程序中处理用户交互的部分。通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。



**什么是Spring MVC？**

简单而言，Spring MVC是Spring在Spring Container Core和AOP等技术基础上，**遵循上述Web MVC的规范**推出的**web开发框架**，目的是为了**简化Java栈的web开发**。

**Spring MVC的请求流程**

![img](https://www.pdai.tech/images/spring/springframework/spring-springframework-mvc-5.png)

**核心架构的具体流程步骤**如下：

1. **首先用户发送请求——>DispatcherServlet**，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行 处理，作为统一访问点，进行全局的流程控制；
2. **DispatcherServlet——>HandlerMapping**， HandlerMapping 将会把请求映射为 HandlerExecutionChain 对象（包含一 个Handler 处理器（页面控制器）对象、多个HandlerInterceptor 拦截器）对象，通过这种策略模式，很容易添加新 的映射策略；
3. **DispatcherServlet——>HandlerAdapter**，HandlerAdapter 将会把处理器包装为适配器，从而支持多种类型的处理器， 即适配器设计模式的应用，从而很容易支持很多类型的处理器；
4. **HandlerAdapter——>处理器功能处理方法的调用**，HandlerAdapter 将会根据适配的结果调用真正的处理器的功能处 理方法，完成功能处理；并返回一个ModelAndView 对象（包含模型数据、逻辑视图名）；
5. **ModelAndView 的逻辑视图名——> ViewResolver**，ViewResolver 将把逻辑视图名解析为具体的View，通过这种策 略模式，很容易更换其他视图技术；
6. **View——>渲染**，View 会根据传进来的Model 模型数据进行渲染，此处的Model 实际是一个Map 数据结构，因此 很容易支持其他视图技术；
7. **返回控制权给DispatcherServlet**，由DispatcherServlet 返回响应给用户，到此一个流程结束。

### Spring进阶- Spring IOC实现原理详解之IOC体系结构设计

在设计时，首先需要考虑的是IOC容器的功能（输入和输出), 承接前面的文章，我们初步的画出IOC容器的整体功能。

![img](https://www.pdai.tech/images/spring/springframework/spring-framework-ioc-source-71.png)

在此基础上，我们初步的去思考，如果作为一个IOC容器的设计者，主体上应该包含哪几个部分：

- 加载Bean的配置（比如xml配置） 
  - 比如不同类型资源的加载，解析成生成统一Bean的定义
- 根据Bean的定义加载生成Bean的实例，并放置在Bean容器中 
  - 比如Bean的依赖注入，Bean的嵌套，Bean存放（缓存）等
- 除了基础Bean外，还有常规针对企业级业务的特别Bean 
  - 比如国际化Message，事件Event等生成特殊的类结构去支撑
- 对容器中的Bean提供统一的管理和调用 
  - 比如用工厂模式管理，提供方法根据名字/类的类型等从容器中获取Bean

#### BeanFactory和BeanRegistry：IOC容器功能规范和Bean的注册

- **BeanFactory： 工厂模式定义了IOC容器的基本功能规范**
- **BeanRegistry： 向IOC容器手工注册 BeanDefinition 对象的方法**

**BeanFactory为何要定义这么多层次的接口？定义了哪些接口？**

主要是为了**区分在 Spring 内部在操作过程中对象的传递和转化过程中，对对象的数据访问所做的限制**。

有哪些接口呢？

- **ListableBeanFactory**：该接口定义了**访问容器中 Bean 基本信息的若干方法**，如查看Bean 的个数、获取某一类型 Bean 的配置名、查看容器中是否包括某一 Bean 等方法；
- **HierarchicalBeanFactory**：**父子级联 IoC 容器的接口**，子容器可以通过接口方法访问父容器； 通过 HierarchicalBeanFactory 接口， Spring 的 IoC 容器可以建立父子层级关联的容器体系，子容器可以访问父容器中的 Bean，但父容器不能访问子容器的 Bean。Spring 使用父子容器实现了很多功能，比如在 Spring MVC 中，展现层 Bean 位于一个子容器中，而业务层和持久层的 Bean 位于父容器中。这样，展现层 Bean 就可以引用业务层和持久层的 Bean，而业务层和持久层的 Bean 则看不到展现层的 Bean。
- **ConfigurableBeanFactory**：是一个重要的接口，**增强了 IoC 容器的可定制性**，它定义了设置类装载器、属性编辑器、容器初始化后置处理器等方法；
- **ConfigurableListableBeanFactory**: **ListableBeanFactory 和 ConfigurableBeanFactory的融合**；
- **AutowireCapableBeanFactory**：定义了将容器中的 Bean **按某种规则（如按名字匹配、按类型匹配等）进行自动装配的方法**；

**如何将Bean注册到BeanFactory中？BeanRegistry**

Spring 配置文件中每一个`<bean>`节点元素在 Spring 容器里都通过一个 BeanDefinition 对象表示，它描述了 Bean 的配置信息。而 BeanDefinitionRegistry 接口提供了向容器手工注册 BeanDefinition 对象的方法。

#### BeanDefinition：各种Bean对象及其相互的关系

**BeanDefinition 定义了各种Bean对象及其相互的关系**

**BeanDefinitionReader 这是BeanDefinition的解析器**

**BeanDefinitionHolder 这是BeanDefination的包装类，用来存储BeanDefinition，name以及aliases等。**



#### ApplicationContext：IOC接口设计和实现

> IoC容器的接口类是ApplicationContext，很显然它必然继承BeanFactory对Bean规范（最基本的ioc容器的实现）进行定义。而ApplicationContext表示的是应用的上下文，除了对Bean的管理外，还至少应该包含了
>
> - **访问资源**： 对不同方式的Bean配置（即资源）进行加载。(实现ResourcePatternResolver接口)
> - **国际化**: 支持信息源，可以实现国际化。（实现MessageSource接口）
> - **应用事件**: 支持应用事件。(实现ApplicationEventPublisher接口)

##### ApplicationContext接口的设计

我们来看下ApplicationContext整体结构

![img](https://www.pdai.tech/images/spring/springframework/spring-framework-ioc-source-51.png)

- **HierarchicalBeanFactory 和 ListableBeanFactory**： ApplicationContext 继承了 HierarchicalBeanFactory 和 ListableBeanFactory 接口，在此基础上，还通过多个其他的接口扩展了 BeanFactory 的功能：
- **ApplicationEventPublisher**：让容器**拥有发布应用上下文事件的功能**，包括容器启动事件、关闭事件等。实现了 ApplicationListener 事件监听接口的 Bean 可以接收到容器事件 ， 并对事件进行响应处理 。 在 ApplicationContext 抽象实现类AbstractApplicationContext 中，我们可以发现存在一个 ApplicationEventMulticaster，它负责保存所有监听器，以便在容器产生上下文事件时通知这些事件监听者。
- **MessageSource**：为应用提供 **i18n 国际化消息访问**的功能；
- **ResourcePatternResolver** ： 所 有ApplicationContext 实现类都实现了类似于PathMatchingResourcePatternResolver 的功能，可以通过带前缀的 Ant 风格的资源文件路径**装载 Spring 的配置文件。**
- **LifeCycle**：该接口是 Spring 2.0 加入的，该接口提供了 start()和 stop()两个方法，主要用于**控制异步处理过程**。在具体使用时，该接口同时被 ApplicationContext 实现及具体 Bean 实现， ApplicationContext 会将 start/stop 的信息传递给容器中所有实现了该接口的 Bean，以达到管理和控制 JMX、任务调度等目的。

##### ApplicationContext接口的实现

在考虑ApplicationContext接口的实现时，关键的点在于，不同Bean的配置方式（比如xml,groovy,annotation等）有着不同的资源加载方式，这便衍生除了众多ApplicationContext的实现类。

![img](https://www.pdai.tech/images/spring/springframework/spring-framework-ioc-source-61.png)

**第一，从类结构设计上看， 围绕着是否需要Refresh容器衍生出两个抽象类**：

- **GenericApplicationContext**： 是**初始化的时候就创建容器，往后的每次refresh都不会更改**
- **AbstractRefreshableApplicationContext**： AbstractRefreshableApplicationContext及子类的每次refresh都是**先清除已有(如果不存在就创建)的容器，然后再重新创建**；AbstractRefreshableApplicationContext及子类无法做到GenericApplicationContext**混合搭配从不同源头获取bean的定义信息**

**第二， 从加载的源来看（比如xml,groovy,annotation等）， 衍生出众多类型的ApplicationContext, 典型比如**:

- **FileSystemXmlApplicationContext**： 从**文件系统**下的一个或多个xml配置文件中加载上下文定义，也就是说**系统盘符中加载xml配置文件**。
- **ClassPathXmlApplicationContext**： 从**类路径**下的一个或多个xml配置文件中加载上下文定义，适用于**xml配置**的方式。
- **AnnotationConfigApplicationContext**： 从一个或多个基于java的配置类中加载上下文定义，适用于**java注解**的方式。
- **ConfigurableApplicationContext**： 扩展于 ApplicationContext，它新增加了两个主要的方法： **refresh()和 close()**，让 ApplicationContext 具有启动、刷新和关闭应用上下文的能力。在应用上下文关闭的情况下调用 refresh()即可启动应用上下文，在已经启动的状态下，调用 refresh()则清除缓存并重新装载配置信息，而调用close()则可关闭应用上下文。这些接口方法为容器的控制管理带来了便利，但作为开发者，我们并不需要过多关心这些方法。

***设计者在设计时AnnotationConfigApplicationContext为什么是继承GenericApplicationContext***？ 因为基于注解的配置，是**不太会被运行时修改**的，这意味着不需要进行动态Bean配置和刷新容器，所以只需要GenericApplicationContext。

而**基于XML**这种配置文件，这种文件**是容易修改**的，需要动态性刷新Bean的支持，所以XML相关的配置必然继承AbstractRefreshableApplicationContext； 

我们把之前的设计要点和设计结构结合起来看：

![img](https://www.pdai.tech/images/spring/springframework/spring-framework-ioc-source-71.png)

### Spring进阶- Spring IOC实现原理详解之IOC初始化流程

Spring如何实现将资源配置（以xml配置为例）通过加载，解析，生成BeanDefination并注册到IoC容器中的（就是我们圈出来的部分）

![img](https://www.pdai.tech/images/spring/springframework/spring-framework-ioc-source-73.png)

**初始化入口**

```
 // create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("aspects.xml", "daos.xml", "services.xml");
```

```
public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, @Nullable ApplicationContext parent) throws BeansException {
    // 设置Bean资源加载器
    super(parent);

    // 设置配置路径(对Bean定义资源文件的定位)
    this.setConfigLocations(configLocations);

    // 初始化容器
    if (refresh) {
        this.refresh();
    }
}
```

**初始化主体流程** refresh()函数

Spring IoC容器对Bean定义资源的载入是从refresh()函数开始的，refresh()是一个模板方法，refresh()方法的作用是：在创建IoC容器前，如果已经有容器存在，则需要把已有的容器销毁和关闭，以保证在refresh之后使用的是新建立起来的IoC容器。

refresh的作用类似于对IoC容器的重启，在新建立好的容器中对容器进行初始化，对Bean定义资源进行载入。

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

        // Prepare this context for refreshing.
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            postProcessBeanFactory(beanFactory);

            StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
            // Invoke factory processors registered as beans in the context.
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            registerBeanPostProcessors(beanFactory);
            beanPostProcess.end();

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                        "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
            contextRefresh.end();
        }
    }
}
```

这里的设计上是一个非常典型的资源类加载处理型的思路，头脑中需要形成如下图的**顶层思路**（而不是只停留在流水式的方法上面）：

- **模板方法设计模式**，模板方法中使用典型的**钩子方法**
- 将**具体的初始化加载方法**插入到钩子方法之间
- 将初始化的阶段封装，用来记录当前初始化到什么阶段；常见的设计是xxxPhase/xxxStage；
- 资源加载初始化有失败等处理，必然是**try/catch/finally**...

![img](https://www.pdai.tech/images/spring/springframework/spring-framework-ioc-source-8.png)

**IOC容器初始化的基本步骤**

![img](https://www.pdai.tech/images/spring/springframework/spring-framework-ioc-source-9.png)

初始化的入口在容器实现中的 refresh()调用来完成

对 bean 定义载入 IOC 容器使用的方法是 loadBeanDefinition,其中的大致过程如下：

- 通过 ResourceLoader 来完成资源文件位置的定位，DefaultResourceLoader 是默认的实现，同时上下文本身就给出了 ResourceLoader 的实现，可以从类路径，文件系统, URL 等方式来定为资源位置。如果是 XmlBeanFactory作为 IOC 容器，那么需要为它指定 bean 定义的资源，也就是说 bean 定义文件时通过抽象成 Resource 来被 IOC 容器处理的
- 通过 BeanDefinitionReader来完成定义信息的解析和 Bean 信息的注册, 往往使用的是XmlBeanDefinitionReader 来解析 bean 的 xml 定义文件 - 实际的处理过程是委托给 BeanDefinitionParserDelegate 来完成的，从而得到 bean 的定义信息，这些信息在 Spring 中使用 BeanDefinition 对象来表示 - 这个名字可以让我们想到loadBeanDefinition,RegisterBeanDefinition 这些相关的方法 - 他们都是为处理 BeanDefinitin 服务的
- 容器解析得到 BeanDefinition 以后，需要把它在 IOC 容器中注册，这由 IOC 实现 BeanDefinitionRegistry 接口来实现。注册过程就是在 IOC 容器内部维护的一个HashMap 来保存得到的 BeanDefinition 的过程。这个 HashMap 是 IoC 容器持有 bean 信息的场所，以后对 bean 的操作都是围绕这个HashMap 来实现的.

然后我们就可以通过 BeanFactory 和 ApplicationContext 来享受到 Spring IOC 的服务了,在使用 IOC 容器的时候，我们注意到除了少量粘合代码，绝大多数以正确 IoC 风格编写的应用程序代码完全不用关心如何到达工厂，因为容器将把这些对象与容器管理的其他对象钩在一起。基本的策略是把工厂放到已知的地方，最好是放在对预期使用的上下文有意义的地方，以及代码将实际需要访问工厂的地方。 Spring 本身提供了对声明式载入 web 应用程序用法的应用程序上下文,并将其存储在ServletContext 中的框架实现。

### Spring进阶- Spring IOC实现原理详解之Bean实例化(生命周期,循环依赖等)

本文主要研究如何从IOC容器已有的BeanDefinition信息，实例化出Bean对象；这里还会包括三块重点内容：

- BeanFactory中getBean的主体思路
- Spring如何解决循环依赖问题
- Spring中Bean的生命周期

![img](https://www.pdai.tech/images/spring/springframework/spring-framework-ioc-source-74.png)

上文我们已经分析了IoC初始化的流程，最终的将Bean的定义即BeanDefinition放到beanDefinitionMap中，本质上是一个`ConcurrentHashMap`<String, Object>；并且BeanDefinition接口中包含了这个类的Class信息以及是否是单例等；

这样我们初步有了实现`Object getBean(String name)`这个方法的思路：

- 从beanDefinitionMap通过beanName获得BeanDefinition
- 从BeanDefinition中获得beanClassName
- 通过反射初始化beanClassName的实例instance 
  - 构造函数从BeanDefinition的getConstructorArgumentValues()方法获取
  - 属性值从BeanDefinition的getPropertyValues()方法获取
- 返回beanName的实例instance

#### Spring中getBean的主体思路

BeanFactory实现getBean方法在AbstractBeanFactory中，这个方法重载都是调用doGetBean方法进行实现的：

```java
public Object getBean(String name) throws BeansException {
  return doGetBean(name, null, null, false);
}
public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
  return doGetBean(name, requiredType, null, false);
}
public Object getBean(String name, Object... args) throws BeansException {
  return doGetBean(name, null, args, false);
}
public <T> T getBean(String name, @Nullable Class<T> requiredType, @Nullable Object... args)
    throws BeansException {
  return doGetBean(name, requiredType, args, false);
}
```

我们来看下doGetBean方法(这个方法很长，我们主要看它的整体思路和设计要点）：

```java
// 参数typeCheckOnly：bean实例是否包含一个类型检查
protected <T> T doGetBean(
			String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)
			throws BeansException {

  // 解析bean的真正name，如果bean是工厂类，name前缀会加&，需要去掉
  String beanName = transformedBeanName(name);
  Object beanInstance;

  // Eagerly check singleton cache for manually registered singletons.
  Object sharedInstance = getSingleton(beanName);
  if (sharedInstance != null && args == null) {
    // 无参单例从缓存中获取
    beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, null);
  }

  else {
    // 如果bean实例还在创建中，则直接抛出异常
    if (isPrototypeCurrentlyInCreation(beanName)) {
      throw new BeanCurrentlyInCreationException(beanName);
    }

    // 如果 bean definition 存在于父的bean工厂中，委派给父Bean工厂获取
    BeanFactory parentBeanFactory = getParentBeanFactory();
    if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
      // Not found -> check parent.
      String nameToLookup = originalBeanName(name);
      if (parentBeanFactory instanceof AbstractBeanFactory) {
        return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
            nameToLookup, requiredType, args, typeCheckOnly);
      }
      else if (args != null) {
        // Delegation to parent with explicit args.
        return (T) parentBeanFactory.getBean(nameToLookup, args);
      }
      else if (requiredType != null) {
        // No args -> delegate to standard getBean method.
        return parentBeanFactory.getBean(nameToLookup, requiredType);
      }
      else {
        return (T) parentBeanFactory.getBean(nameToLookup);
      }
    }

    if (!typeCheckOnly) {
      // 将当前bean实例放入alreadyCreated集合里，标识这个bean准备创建了
      markBeanAsCreated(beanName);
    }

    StartupStep beanCreation = this.applicationStartup.start("spring.beans.instantiate")
        .tag("beanName", name);
    try {
      if (requiredType != null) {
        beanCreation.tag("beanType", requiredType::toString);
      }
      RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
      checkMergedBeanDefinition(mbd, beanName, args);

      // 确保它的依赖也被初始化了.
      String[] dependsOn = mbd.getDependsOn();
      if (dependsOn != null) {
        for (String dep : dependsOn) {
          if (isDependent(beanName, dep)) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
          }
          registerDependentBean(dep, beanName);
          try {
            getBean(dep); // 初始化它依赖的Bean
          }
          catch (NoSuchBeanDefinitionException ex) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
          }
        }
      }

      // 创建Bean实例：单例
      if (mbd.isSingleton()) {
        sharedInstance = getSingleton(beanName, () -> {
          try {
            // 真正创建bean的方法
            return createBean(beanName, mbd, args);
          }
          catch (BeansException ex) {
            // Explicitly remove instance from singleton cache: It might have been put there
            // eagerly by the creation process, to allow for circular reference resolution.
            // Also remove any beans that received a temporary reference to the bean.
            destroySingleton(beanName);
            throw ex;
          }
        });
        beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
      }
      // 创建Bean实例：原型
      else if (mbd.isPrototype()) {
        // It's a prototype -> create a new instance.
        Object prototypeInstance = null;
        try {
          beforePrototypeCreation(beanName);
          prototypeInstance = createBean(beanName, mbd, args);
        }
        finally {
          afterPrototypeCreation(beanName);
        }
        beanInstance = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
      }
      // 创建Bean实例：根据bean的scope创建
      else {
        String scopeName = mbd.getScope();
        if (!StringUtils.hasLength(scopeName)) {
          throw new IllegalStateException("No scope name defined for bean ´" + beanName + "'");
        }
        Scope scope = this.scopes.get(scopeName);
        if (scope == null) {
          throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
        }
        try {
          Object scopedInstance = scope.get(beanName, () -> {
            beforePrototypeCreation(beanName);
            try {
              return createBean(beanName, mbd, args);
            }
            finally {
              afterPrototypeCreation(beanName);
            }
          });
          beanInstance = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
        }
        catch (IllegalStateException ex) {
          throw new ScopeNotActiveException(beanName, scopeName, ex);
        }
      }
    }
    catch (BeansException ex) {
      beanCreation.tag("exception", ex.getClass().toString());
      beanCreation.tag("message", String.valueOf(ex.getMessage()));
      cleanupAfterBeanCreationFailure(beanName);
      throw ex;
    }
    finally {
      beanCreation.end();
    }
  }

  return adaptBeanInstance(name, beanInstance, requiredType);
}
```

#### 重点：Spring如何解决循环依赖问题

**spring 解决三级依赖：三级缓存**

先来看下这三级缓存

```java
/** Cache of singleton objects: bean name --> bean instance */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
 
/** Cache of early singleton objects: bean name --> bean instance */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

/** Cache of singleton factories: bean name --> ObjectFactory */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
```

**第一层缓存（singletonObjects）**：单例对象缓存池，已经实例化并且属性赋值，这里的对象是**成熟对象**；

**第二层缓存（earlySingletonObjects）**：单例对象缓存池，已经实例化但尚未属性赋值，这里的对象是**半成品对象**；

**第三层缓存（singletonFactories）**: 单例工厂的缓存

如下是获取单例中

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
  // Spring首先从singletonObjects（一级缓存）中尝试获取
  Object singletonObject = this.singletonObjects.get(beanName);
  // 若是获取不到而且对象在建立中，则尝试从earlySingletonObjects(二级缓存)中获取
  if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
    synchronized (this.singletonObjects) {
        singletonObject = this.earlySingletonObjects.get(beanName);
        if (singletonObject == null && allowEarlyReference) {
          ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
          if (singletonFactory != null) {
            //若是仍是获取不到而且容许从singletonFactories经过getObject获取，则经过singletonFactory.getObject()(三级缓存)获取
              singletonObject = singletonFactory.getObject();
              //若是获取到了则将singletonObject放入到earlySingletonObjects,也就是将三级缓存提高到二级缓存中
              this.earlySingletonObjects.put(beanName, singletonObject);
              this.singletonFactories.remove(beanName);
          }
        }
    }
  }
  return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
```

从上面三级缓存的分析，咱们能够知道，Spring**解决循环依赖的诀窍就在于singletonFactories这个三级cache**。这个cache的类型是ObjectFactory，定义以下：

```java
public interface ObjectFactory<T> {
    T getObject() throws BeansException;
}
```

在bean建立过程当中，有两处比较重要的匿名内部类实现了该接口。一处是Spring利用其建立bean的时候，另外一处就是:

```java
addSingletonFactory(beanName, new ObjectFactory<Object>() {
   @Override   public Object getObject() throws BeansException {
      return getEarlyBeanReference(beanName, mbd, bean);
   }});
```

此处就是解决循环依赖的关键，这段代码发生在createBeanInstance以后，也就是说单例对象此时已经被建立出来的。这个对象已经被生产出来了，虽然还不完美（尚未进行初始化的第二步和第三步），可是已经能被人认出来了（根据对象引用能定位到堆中的对象），因此Spring此时将这个对象提早曝光出来让你们认识，让你们使用。

**Spring为什么不能解决构造器的循环依赖？**

构造器注入形成的循环依赖： 也就是beanB需要在beanA的构造函数中完成初始化，beanA也需要在beanB的构造函数中完成初始化，这种情况的结果就是两个bean都不能完成初始化，循环依赖难以解决。

Spring解决循环依赖主要是依赖三级缓存，但是的**在调用构造方法之前还未将其放入三级缓存之中**，因此后续的依赖调用构造方法的时候并不能从三级缓存中获取到依赖的Bean，因此不能解决。

**[#](#spring为什么不能解决prototype作用域循环依赖) Spring为什么不能解决prototype作用域循环依赖？**

这种循环依赖同样无法解决，因为spring不会缓存‘prototype’作用域的bean，而spring中循环依赖的解决正是通过缓存来实现的。

**[#](#spring为什么不能解决多例的循环依赖) Spring为什么不能解决多例的循环依赖？**

多实例Bean是每次调用一次getBean都会执行一次构造方法并且给属性赋值，根本没有三级缓存，因此不能解决循环依赖。

**其他循环依赖如何解决？**

- **生成代理对象产生的循环依赖**

这类循环依赖问题解决方法很多，主要有：

1. 使用@Lazy注解，延迟加载
2. 使用@DependsOn注解，指定加载先后关系
3. 修改文件名称，改变循环依赖类的加载顺序

- **使用@DependsOn产生的循环依赖**

这类循环依赖问题要找到@DependsOn注解循环依赖的地方，迫使它不循环依赖就可以解决问题。

- **多例循环依赖**

这类循环依赖问题可以通过把bean改成单例的解决。

- **构造器循环依赖**

这类循环依赖问题可以通过使用@Lazy注解解决。

#### 重点：Spring中Bean的生命周期

Spring 只帮我们管理**单例模式** Bean 的**完整**生命周期，对于 prototype 的 bean ，Spring 在创建好交给使用者之后则不会再管理后续的生命周期。

**Spring 容器中 Bean 的生命周期流程**

![img](https://www.pdai.tech/images/spring/springframework/spring-springframework-aop-4.png)

- 如果 BeanFactoryPostProcessor 和 Bean 关联, 则调用postProcessBeanFactory方法.(即首**先尝试从Bean工厂中获取Bean**)

- 如果 InstantiationAwareBeanPostProcessor 和 Bean 关联，则调用postProcessBeforeInstantiation方法

- 根据配置情况调用 Bean 构造方法**实例化 Bean**。

- 利用依赖注入完成 Bean 中所有**属性值的配置注入**。

- 如果 InstantiationAwareBeanPostProcessor 和 Bean 关联，则调用postProcessAfterInstantiation方法和postProcessProperties

- 调用xxxAware接口

   (上图只是给了几个例子) 

  - 第一类Aware接口
    - 如果 Bean 实现了 BeanNameAware 接口，则 Spring 调用 Bean 的 setBeanName() 方法传入当前 Bean 的 id 值。
    - 如果 Bean 实现了 BeanClassLoaderAware 接口，则 Spring 调用 setBeanClassLoader() 方法传入classLoader的引用。
    - 如果 Bean 实现了 BeanFactoryAware 接口，则 Spring 调用 setBeanFactory() 方法传入当前工厂实例的引用。
  - 第二类Aware接口
    - 如果 Bean 实现了 EnvironmentAware 接口，则 Spring 调用 setEnvironment() 方法传入当前 Environment 实例的引用。
    - 如果 Bean 实现了 EmbeddedValueResolverAware 接口，则 Spring 调用 setEmbeddedValueResolver() 方法传入当前 StringValueResolver 实例的引用。
    - 如果 Bean 实现了 ApplicationContextAware 接口，则 Spring 调用 setApplicationContext() 方法传入当前 ApplicationContext 实例的引用。
    - ...

- 如果 BeanPostProcessor 和 Bean 关联，则 Spring 将调用该接口的预初始化方法 postProcessBeforeInitialzation() 对 Bean 进行加工操作，此处非常重要，Spring 的 AOP 就是利用它实现的。

- 如果 Bean 实现了 InitializingBean 接口，则 Spring 将调用 afterPropertiesSet() 方法。(或者有执行@PostConstruct注解的方法)

- 如果在配置文件中通过 **init-method** 属性指定了初始化方法，则调用该初始化方法。

- 如果 BeanPostProcessor 和 Bean 关联，则 Spring 将调用该接口的初始化方法 postProcessAfterInitialization()。此时，Bean 已经可以被应用系统使用了。

- 如果在 `` 中指定了该 Bean 的作用范围为 scope="singleton"，则将该 Bean 放入 Spring IoC 的缓存池中，将触发 Spring 对该 Bean 的生命周期管理；如果在 `` 中指定了该 Bean 的作用范围为 scope="prototype"，则将该 Bean 交给调用者，调用者管理该 Bean 的生命周期，Spring 不再管理该 Bean。

- 如果 Bean 实现了 DisposableBean 接口，则 Spring 会调用 destory() 方法将 Spring 中的 Bean 销毁；(或者有执行@PreDestroy注解的方法)

- 如果在配置文件中通过 **destory-method** 属性指定了 Bean 的销毁方法，则 Spring 将调用该方法对 Bean 进行销毁。

**Bean的完整生命周期经历了各种方法调用，这些方法可以划分为以下几类**：(结合上图，需要有如下顶层思维)

- **Bean自身的方法**： 这个包括了Bean本身调用的方法和通过配置文件中`<bean>`的init-method和destroy-method指定的方法
- **Bean级生命周期接口方法**： 这个包括了BeanNameAware、BeanFactoryAware、ApplicationContextAware；当然也包括InitializingBean和DiposableBean这些接口的方法（可以被@PostConstruct和@PreDestroy注解替代)
- **容器级生命周期接口方法**： 这个包括了InstantiationAwareBeanPostProcessor 和 BeanPostProcessor 这两个接口实现，一般称它们的实现类为“后处理器”。
- **工厂后处理器接口方法**： 这个包括了AspectJWeavingEnabler, ConfigurationClassPostProcessor, CustomAutowireConfigurer等等非常有用的工厂后处理器接口的方法。工厂后处理器也是容器级的。在应用上下文装配配置文件之后立即调用。

### Spring进阶 - Spring AOP实现原理详解之AOP切面的实现

1. 根据其XML配置我们不难发现**AOP是基于IOC的Bean加载来实现的**；这便使我们的主要入口

![img](https://www.pdai.tech/images/spring/springframework/spring-springframework-aop-1.png)

所以理解Spring AOP的初始化必须要先理解[Spring IOC的初始化]()。

1. 然后我们就能找到如下**初始化的流程和aop对应的handler**类

由**IOC Bean加载**方法栈中找到parseCustomElement方法，找到parse `aop:aspectj-autoproxy`的handler(org.springframework.aop.config.AopNamespaceHandler)

![img](https://www.pdai.tech/images/spring/springframework/spring-springframework-aop-2.png)

#### aop配置标签的解析

> 上文中，我们找到了AopNamespaceHandler，其实就是注册BeanDefinition的解析器BeanDefinitionParser，将`aop:xxxxxx`配置标签交给指定的parser来处理。

```java
public class AopNamespaceHandler extends NamespaceHandlerSupport {

	/**
	 * Register the {@link BeanDefinitionParser BeanDefinitionParsers} for the
	 * '{@code config}', '{@code spring-configured}', '{@code aspectj-autoproxy}'
	 * and '{@code scoped-proxy}' tags.
	 */
	@Override
	public void init() {
		// In 2.0 XSD as well as in 2.5+ XSDs
        // 注册解析<aop:config> 配置
		registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
        // 注册解析<aop:aspectj-autoproxy> 配置
		registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
		registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());

		// Only in 2.0 XSD: moved to context namespace in 2.5+
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
	}

}
```

![img](https://www.pdai.tech/images/spring/springframework/spring-springframework-aop-4.png)

我们就可以定位到核心的初始化方法肯定在postProcessBeforeInstantiation和postProcessAfterInitialization中。

**总结AOP初始化过程**

**AOP的创建**工作是交给AnnotationAwareAspectJAutoProxyCreator来完成的

**AopNamespaceHandler**注册了`<aop:aspectj-autoproxy/>`的解析类是AspectJAutoProxyBeanDefinitionParser

**AspectJAutoProxyBeanDefinitionParser**的parse 方法 通过AspectJAwareAdvisorAutoProxyCreator类去创建

**AspectJAwareAdvisorAutoProxyCreator**实现了两类接口，BeanFactoryAware和BeanPostProcessor；根据Bean生命周期方法找到两个核心方法：postProcessBeforeInstantiation和postProcessAfterInitialization 

1. **postProcessBeforeInstantiation**：主要是**处理使用了@Aspect注解的切面类**，然后将切面类的所有切面方法根据使用的注解生成对应Advice，并将Advice连同切入点匹配器和切面类等信息一并封装到Advisor
2. **postProcessAfterInitialization**：主要负责将Advisor注入到合适的位置，**创建代理（cglib或jdk)**，为后面给代理进行增强实现做准备。

### Spring进阶 - Spring AOP实现原理详解之AOP代理的创建

Spring默认在**目标类实现接口时是通过JDK代理**实现的，只有非接口的是通过Cglib代理实现的。当设置proxy-target-class为**true时在目标类不是接口或者代理类时优先使用cglib代理**实现。

### Spring进阶 - Spring AOP实现原理详解之Cglib代理实现

#### 动态代理要解决什么问题？

**代理模式**(Proxy pattern): 为另一个对象提供**一个替身或占位符**以控制对这个对象的访问

![img](https://www.pdai.tech/images/pics/a6c20f60-5eba-427d-9413-352ada4b40fe.png)

举个简单的例子：

我(client)如果要买(doOperation)房，可以找中介(proxy)买房，中介直接和卖方(target)买房。中介和卖方都实现买卖(doOperation)的操作。中介就是代理(proxy)。

**什么是动态代理？**动态代理就是，在程序运行期，创建目标对象的代理对象，并对目标对象中的方法进行功能性增强的一种技术。

![img](https://www.pdai.tech/images/spring/springframework/spring-springframework-aop-61.png)

#### 什么是Cglib? SpringAOP和Cglib是什么关系？

Cglib是一个强大的、高性能的代码生成包，它广泛被许多AOP框架使用，为他们**提供方法的拦截**。

![img](https://www.pdai.tech/images/spring/springframework/spring-springframework-aop-62.png)

#### cglib代理案例

cglib代理类，需要实现MethodInterceptor接口，并指定代理目标类target

**代理**

```java
package tech.pdai.springframework.proxy;

import java.lang.reflect.Method;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

/**
 * This class is for proxy demo.
 *
 * @author pdai
 */
public class UserLogProxy implements MethodInterceptor {

    /**
     * 业务类对象，供代理方法中进行真正的业务方法调用
     */
    private Object target;

    public Object getUserLogProxy(Object target) {
        //给业务对象赋值
        this.target = target;
        //创建加强器，用来创建动态代理类
        Enhancer enhancer = new Enhancer();
        //为加强器指定要代理的业务类（即：为下面生成的代理类指定父类）
        enhancer.setSuperclass(this.target.getClass());
        //设置回调：对于代理类上所有方法的调用，都会调用CallBack，而Callback则需要实现intercept()方法进行拦
        enhancer.setCallback(this);
        // 创建动态代理类对象并返回
        return enhancer.create();
    }

    // 实现回调方法
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        // log - before method
        System.out.println("[before] execute method: " + method.getName());

        // call method
        Object result = proxy.invokeSuper(obj, args);

        // log - after method
        System.out.println("[after] execute method: " + method.getName() + ", return value: " + result);
        return null;
    }
}
```

**[#](#使用代理) 使用代理**

启动类中指定代理目标并执行。

```java
package tech.pdai.springframework;

import tech.pdai.springframework.proxy.UserLogProxy;
import tech.pdai.springframework.service.UserServiceImpl;

/**
 * Cglib proxy demo.
 *
 * @author pdai
 */
public class ProxyDemo {

    /**
     * main interface.
     *
     * @param args args
     */
    public static void main(String[] args) {
        // proxy
        UserServiceImpl userService = (UserServiceImpl) new UserLogProxy().getUserLogProxy(new UserServiceImpl());

        // call methods
        userService.findUserList();
        userService.addUser();
    }
}
```

#### Cglib代理的流程

![img](https://www.pdai.tech/images/spring/springframework/spring-springframework-aop-63.png)



### Spring进阶 - Spring AOP实现原理详解之JDK代理实现

**什么是JDK代理?**

JDK动态代理是有JDK提供的工具类Proxy实现的，动态代理类是在运行时生成指定接口的代理类，每个代理实例（实现需要代理的接口）都有一个关联的调用处理程序对象，此对象实现了InvocationHandler，最终的业务逻辑是在InvocationHandler实现类的invoke方法上。

#### JDK代理案例

**JDK代理类**

代理类如下：

```java
package tech.pdai.springframework.proxy;

import tech.pdai.springframework.service.IUserService;
import tech.pdai.springframework.service.UserServiceImpl;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

/**
 * This class is for proxy demo.
 *
 * @author pdai
 */
public class UserLogProxy {

    /**
     * proxy target
     */
    private IUserService target;

    /**
     * init.
     *
     * @param target target
     */
    public UserLogProxy(UserServiceImpl target) {
        super();
        this.target = target;
    }

    /**
     * get proxy.
     *
     * @return proxy target
     */
    public IUserService getLoggingProxy() {
        IUserService proxy;
        ClassLoader loader = target.getClass().getClassLoader();
        Class[] interfaces = new Class[]{IUserService.class};
        InvocationHandler h = new InvocationHandler() {
            /**
             * proxy: 代理对象。 一般不使用该对象 method: 正在被调用的方法 args: 调用方法传入的参数
             */
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                String methodName = method.getName();
                // log - before method
                System.out.println("[before] execute method: " + methodName);

                // call method
                Object result = null;
                try {
                    // 前置通知
                    result = method.invoke(target, args);
                    // 返回通知, 可以访问到方法的返回值
                } catch (NullPointerException e) {
                    e.printStackTrace();
                    // 异常通知, 可以访问到方法出现的异常
                }
                // 后置通知. 因为方法可以能会出异常, 所以访问不到方法的返回值

                // log - after method
                System.out.println("[after] execute method: " + methodName + ", return value: " + result);
                return result;
            }
        };
        /**
         * loader: 代理对象使用的类加载器.
         * interfaces: 指定代理对象的类型. 即代理代理对象中可以有哪些方法.
         * h: 当具体调用代理对象的方法时, 应该如何进行响应, 实际上就是调用 InvocationHandler 的 invoke 方法
         */
        proxy = (IUserService) Proxy.newProxyInstance(loader, interfaces, h);
        return proxy;
    }

}
```

**[#](#使用代理) 使用代理**

启动类中指定代理目标并执行。

```java
package tech.pdai.springframework;

import tech.pdai.springframework.proxy.UserLogProxy;
import tech.pdai.springframework.service.IUserService;
import tech.pdai.springframework.service.UserServiceImpl;

/**
 * Jdk proxy demo.
 *
 * @author pdai
 */
public class ProxyDemo {

    /**
     * main interface.
     *
     * @param args args
     */
    public static void main(String[] args) {
        // proxy
        IUserService userService = new UserLogProxy(new UserServiceImpl()).getLoggingProxy();

        // call methods
        userService.findUserList();
        userService.addUser();
    }
}
```

#### JDK代理的流程

> JDK代理自动生成的class是由sun.misc.ProxyGenerator来生成的。

一共三个步骤（**把大象装进冰箱分几步**？）：

- 第一步：（把冰箱门打开）准备工作，将所有方法包装成ProxyMethod对象，包括Object类中hashCode、equals、toString方法，以及被代理的接口中的方法
- 第二步：（把大象装进去）为代理类组装字段，构造函数，方法，static初始化块等
- 第三步：（把冰箱门带上）写入class文件

### Spring进阶 - SpringMVC实现原理之DispatcherServlet的初始化过程

#### DispatcherServlet和ApplicationContext有何关系？

DispatcherServlet 作为一个 Servlet，需要根据 Servlet 规范使用 Java 配置或 web.xml 声明和映射。反过来，DispatcherServlet 使用 Spring 配置来发现请求映射、视图解析、异常处理等等所需的委托组件。

DispatcherServlet 需要 WebApplicationContext（继承自 ApplicationContext） 来配置。WebApplicationContext 可以链接到ServletContext 和 Servlet。因为绑定了 ServletContext，这样应用程序就可以在需要的时候使用 RequestContextUtils 的静态方法访问 WebApplicationContext。

大多数应用程序只有一个WebApplicationContext，除此之外也可以一个Root WebApplicationContext 被多个 Servlet实例，然后各自拥有自己的Servlet WebApplicationContext 配置。

Root WebApplicationContext 包含需要共享给多个 Servlet 实例的数据源和业务服务基础 Bean。这些 Bean 可以在 Servlet 特定的范围被继承或覆盖。

![img](https://www.pdai.tech/images/spring/springframework/spring-springframework-mvc-13.png)



#### DispatcherServlet是如何初始化的？

DispatcherServlet的类结构关系：

![img](https://www.pdai.tech/images/spring/springframework/spring-springframework-mvc-11.png)

**init**：init()方法位于HttpServletBean, 主要读取web.xml中servlet参数配置，并将交给子类方法initServletBean()继续初始化。

**initWebApplicationContext**：initWebApplicationContext用来初始化和刷新WebApplicationContext。

webApplicationContext只会初始化一次，依次尝试构造函数初始化，没有则通过contextAttribute初始化，仍没有则创建新的。

configureAndRefreshWebApplicationContext方法初始化设置Spring环境

**refresh**

有了webApplicationContext后，就开始刷新了（onRefresh()方法），这个方法是FrameworkServlet提供的模板方法，由子类DispatcherServlet来实现的。

```java
/**
  * This implementation calls {@link #initStrategies}.
  */
@Override
protected void onRefresh(ApplicationContext context) {
  initStrategies(context);
}
```

刷新主要是调用initStrategies(context)方法对DispatcherServlet中的组件进行初始化，这些组件就是在SpringMVC请求流程中包的主要组件。

```java
/**
  * Initialize the strategy objects that this servlet uses.
  * <p>May be overridden in subclasses in order to initialize further strategy objects.
  */
protected void initStrategies(ApplicationContext context) {
  initMultipartResolver(context);
  initLocaleResolver(context);
  initThemeResolver(context);

  // 主要看如下三个方法
  initHandlerMappings(context);
  initHandlerAdapters(context);
  initHandlerExceptionResolvers(context);

  initRequestToViewNameTranslator(context);
  initViewResolvers(context);
  initFlashMapManager(context);
}
```

**initHanlderxxx**

我们主要看initHandlerXXX相关的方法，它们之间的关系可以看SpringMVC的请求流程

![img](https://www.pdai.tech/images/spring/springframework/spring-springframework-mvc-26.png)

HandlerMapping是映射处理器

HandlerAdpter是**处理适配器**，它用来找到你的Controller中的处理方法

HandlerExceptionResolver是当遇到处理异常时的异常解析器



### Spring进阶 - SpringMVC实现原理之DispatcherServlet处理请求的过程

#### doGet入口

我们知道servlet处理get请求是doGet方法，所以我们去找DispatcherServlet类结构中的doGet方法。

本质上就是调用doService方法，由DispatchServlet类实现

#### 请求分发

doDispatch方法是真正处理请求的核心方法

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
  HttpServletRequest processedRequest = request;
  HandlerExecutionChain mappedHandler = null;
  boolean multipartRequestParsed = false;

  WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

  try {
    ModelAndView mv = null;
    Exception dispatchException = null;

    try {
      // 判断是不是文件上传类型的request
      processedRequest = checkMultipart(request);
      multipartRequestParsed = (processedRequest != request);

      // 根据request获取匹配的handler.
      mappedHandler = getHandler(processedRequest);
      if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return;
      }

      // 根据handler获取匹配的handlerAdapter
      HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

      // 如果handler支持last-modified头处理
      String method = request.getMethod();
      boolean isGet = HttpMethod.GET.matches(method);
      if (isGet || HttpMethod.HEAD.matches(method)) {
        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
        if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
          return;
        }
      }

      if (!mappedHandler.applyPreHandle(processedRequest, response)) {
        return;
      }

      // 真正handle处理，并返回modelAndView
      mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

      if (asyncManager.isConcurrentHandlingStarted()) {
        return;
      }

      // 通过视图的prefix和postfix获取完整的视图名
      applyDefaultViewName(processedRequest, mv);

      // 应用后置的拦截器
      mappedHandler.applyPostHandle(processedRequest, response, mv);
    }
    catch (Exception ex) {
      dispatchException = ex;
    }
    catch (Throwable err) {
      // As of 4.3, we're processing Errors thrown from handler methods as well,
      // making them available for @ExceptionHandler methods and other scenarios.
      dispatchException = new NestedServletException("Handler dispatch failed", err);
    }

    // 处理handler处理的结果，显然就是对ModelAndView 或者 出现的Excpetion处理
    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
  }
  catch (Exception ex) {
    triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
  }
  catch (Throwable err) {
    triggerAfterCompletion(processedRequest, response, mappedHandler,
        new NestedServletException("Handler processing failed", err));
  }
  finally {
    if (asyncManager.isConcurrentHandlingStarted()) {
      // Instead of postHandle and afterCompletion
      if (mappedHandler != null) {
        mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
      }
    }
    else {
      // Clean up any resources used by a multipart request.
      if (multipartRequestParsed) {
        cleanupMultipart(processedRequest);
      }
    }
  }
}
```

**映射和适配器处理**

对于真正的handle方法，我们看下其处理流程

```java
/**
  * This implementation expects the handler to be an {@link HandlerMethod}.
  */
@Override
@Nullable
public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
    throws Exception {

  return handleInternal(request, response, (HandlerMethod) handler);
}
```

交给handleInternal方法处理，以RequestMappingHandlerAdapter这个HandlerAdapter中的处理方法为例

然后执行invokeHandlerMethod这个方法，用来对RequestMapping（usercontroller中的list方法）进行处理

invokeAndHandle交给UserController中具体执行list方法执行

#### 视图渲染

接下来继续执行processDispatchResult方法，对视图和model（如果有异常则对异常处理）进行处理（显然就是渲染页面了）

接下来显然就是渲染视图render()方法了, spring在initStrategies方法中初始化的组件（LocaleResovler等）就派上用场了。



### SpringBoot入门 - SpringBoot简介

**为什么有了SpringFramework还会诞生SpringBoot？**

简单而言，因为虽然Spring的组件代码是轻量级的，但它的配置却是重量级的；所以SpringBoot的设计策略是通过**开箱即用**和**约定大于配置** 来解决配置重的问题的。

**SpringBoot的核心功能**

- **起步依赖** 起步依赖本质上是一个Maven项目对象模型（Project Object Model，POM），定义了对其他库的传递依赖，这些东西加在一起即支持某项功能。

简单的说，起步依赖就是将具备某种功能的坐标打包到一起，并提供一些默认的功能。

- **自动配置**

Spring Boot的自动配置是一个运行时（更准确地说，是应用程序启动时）的过程，考虑了众多因素，才决定Spring配置应该用哪个，不该用哪个。该过程是Spring自动完成的。

Spring Boot 推荐的基础 POM 文件

| 名称                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| spring-boot-starter              | 核心 POM，包含自动配置支持、日志库和对 YAML 配置文件的支持。 |
| spring-boot-starter-amqp         | 通过 spring-rabbit 支持 AMQP。                               |
| spring-boot-starter-aop          | 包含 spring-aop 和 AspectJ 来支持面向切面编程（AOP）。       |
| spring-boot-starter-batch        | 支持 Spring Batch，包含 HSQLDB。                             |
| spring-boot-starter-data-jpa     | 包含 spring-data-jpa、spring-orm 和 Hibernate 来支持 JPA。   |
| spring-boot-starter-data-mongodb | 包含 spring-data-mongodb 来支持 MongoDB。                    |
| spring-boot-starter-data-rest    | 通过 spring-data-rest-webmvc 支持以 REST 方式暴露 Spring Data 仓库。 |
| spring-boot-starter-jdbc         | 支持使用 JDBC 访问数据库。                                   |
| spring-boot-starter-security     | 包含 spring-security。                                       |
| spring-boot-starter-test         | 包含常用的测试所需的依赖，如 JUnit、Hamcrest、Mockito 和 spring-test 等。 |
| spring-boot-starter-velocity     | 支持使用 Velocity 作为模板引擎。                             |
| spring-boot-starter-web          | 支持 Web 应用开发，包含 Tomcat 和 spring-mvc。               |
| spring-boot-starter-websocket    | 支持使用 Tomcat 开发 WebSocket 应用。                        |
| spring-boot-starter-ws           | 支持 Spring Web Services。                                   |
| spring-boot-starter-actuator     | 添加适用于生产环境的功能，如性能指标和监测等功能。           |
| spring-boot-starter-remote-shell | 添加远程 SSH 支持。                                          |
| spring-boot-starter-jetty        | 使用 Jetty 而不是默认的 Tomcat 作为应用服务器。              |
| spring-boot-starter-log4j        | 添加 Log4j 的支持。                                          |
| spring-boot-starter-logging      | 使用 Spring Boot 默认的日志框架 Logback。                    |
| spring-boot-starter-tomcat       | 使用 Spring Boot 默认的 Tomcat 作为应用服务器。              |

### SpringBoot入门 - 对Hello world进行MVC分层



**经典MVC三层架构**

三层架构(3-tier application) 通常意义上的三层架构就是将整个业务应用划分为：表现层（UI）、业务逻辑层（BLL）、数据访问层（DAL）。区分层次的目的即为了“高内聚，低耦合”的思想。

1、表现层（UI）：通俗讲就是展现给用户的界面，即用户在使用一个系统的时候他的所见所得。

2、业务逻辑层（BLL）：针对具体问题的操作，也可以说是对数据层的操作，对数据业务逻辑处理。

3、数据访问层（DAL）：该层所做事务直接操作数据库，针对数据的增添、删除、修改、更新、查找等。

![img](https://www.pdai.tech/images/spring/springframework/spring-framework-helloworld-2.png)

![img](https://www.pdai.tech/images/spring/springboot/springboot-hello-mvc-1.png)

### SpringBoot入门 - 添加内存数据库H2

**什么是H2内存数据库**

> H2是一个用Java开发的**嵌入式数据库**，它本身只是一个类库，可以直接嵌入到应用项目中。

**有哪些用途**？

- H2最大的用途在于可以**同应用程序打包在一起发布**，这样可以非常方便地存储少量结构化数据。
- 它的另一个用途是**用于单元测试**。启动速度快，而且可以关闭持久化功能，每一个用例执行完随即还原到初始状态。
- H2的第三个用处是**作为缓存**，作为NoSQL的一个补充。当某些场景下数据模型必须为关系型，可以拿它当Memcached使，作为后端MySQL/Oracle的一个缓冲层，缓存一些不经常变化但需要频繁访问的数据，比如字典表、权限表。不过这样系统架构就会比较复杂了。

**H2的产品优势**?

- 纯Java编写，不受平台的限制；
- 只有一个jar文件，适合作为嵌入式数据库使用；
- h2提供了一个十分方便的web控制台用于操作和管理数据库内容；
- 功能完整，支持标准SQL和JDBC。麻雀虽小五脏俱全；
- 支持内嵌模式、服务器模式和集群。

**[#](#什么是jpa-和jdbc是什么关系) 什么是JPA，和JDBC是什么关系**

- **什么是JDBC**

JDBC（JavaDataBase Connectivity）就是Java数据库连接，说白了就是用Java语言来操作数据库。

![img](https://www.pdai.tech/images/project/project-b-4.png)

- **什么是ORM**

对象关系映射（Object Relational Mapping，简称ORM）， 简单的说，**ORM是通过使用描述对象和数据库之间映射的元数据，将java程序中的对象自动持久化到关系数据库中**。本质上就是将数据从一种形式转换到另外一种形式，具体如下：

![img](https://www.pdai.tech/images/project/project-b-3.png)

具体映射：

1. 数据库的表（table） --> 类（class）
2. 记录（record，行数据）--> 对象（object）
3. 字段（field）--> 对象的属性（attribute）

- **什么是JPA**

JPA是Spring提供的一种ORM



### SpringBoot入门 - 配置热部署devtools工具

**什么是热部署和热加载？**

热部署和热加载是在应用正在运行的时候，**自动更新（重新加载或者替换class等）应用**的一种能力。

**热部署**

- 在服务器运行时重新部署项目
- 它是直接重新加载整个应用，这种方式会释放内存，比热加载更加干净彻底，但同时也更费时间。

**热加载**

- 在在运行时重新加载class，从而升级应用。
- 热加载的实现原理主要依赖[java的类加载机制]()，在实现方式可以概括为在容器启动的时候起一条后台线程，定时的检测类文件的时间戳变化，如果类的时间戳变掉了，则将类重新载入。
- 对比反射机制，反射是在运行时获取类信息，通过动态的调用来改变程序行为； 热加载则是在运行时通过重新加载改变类信息，直接改变程序行为。

**实现热部署**

1. pom配置：添加spring-boot-devtools的依赖。

2. idea配置：**无任何配置时，手动触发重启更新（Ctrl+F9）**或（**mvn compile**）；**IDEA需开启运行时编译，自动重启更新**

3. application.yml配置：

   ```
   spring:
     devtools:
       restart:
         enabled: true  #设置开启热部署
         additional-paths: src/main/java #重启目录
         exclude: WEB-INF/**
     thymeleaf:
       cache: false #使用Thymeleaf模板引擎，关闭缓存
   ```

4. 使用LiveReload



devtools原理：spring-boot-devtools使用了两个类加载器ClassLoader，一个ClassLoader**加载不会发生更改的类**（第三方jar包），另一个ClassLoader（restart ClassLoader）**加载会更改的类**（自定义的类）。

后台启动一个**文件监听线程（File Watcher）**，**监测的目录中的文件发生变动时， 原来的restart ClassLoader被丢弃，将会重新加载新的restart ClassLoader**。

因为文件变动后，**第三方jar包不再重新加载，只加载自定义的类**，加载的类比较少，所以重启比较快。

这也是为什么，同样是重启应用，为什么不手动重启，建议使用spring-boot-devtools进行热部署重启。

### SpringBoot入门 - 开发中还有哪些常用注解

**@SpringBootApplication**

定义在main方法入口类处，用于启动sping boot应用项目

**@EnableAutoConfiguration**

让spring boot根据类路径中的jar包依赖当前项目进行自动配置

在src/main/resources的META-INF/spring.factories

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration

若有多个自动配置，用“，”隔开
```

**@ImportResource**

加载xml配置，一般是放在启动main类上

```java
@ImportResource("classpath*:/spring/*.xml")  单个

@ImportResource({"classpath*:/spring/1.xml","classpath*:/spring/2.xml"})   多个
```

**@Value**

application.properties定义属性，直接使用@Value注入即可

```java
public class A{
	 @Value("${push.start:0}")    如果缺失，默认值为0
     private Long  id;
}
```

**@ConfigurationProperties(prefix="person")**

可以新建一个properties文件，ConfigurationProperties的属性prefix指定properties的配置的前缀，通过location指定properties文件的位置

```java
@ConfigurationProperties(prefix="person")
public class PersonProperties {
	
	private String name ;
	private int age;
}
```

**@EnableConfigurationProperties**

用 @EnableConfigurationProperties注解使 @ConfigurationProperties生效，并从IOC容器中获取bean。

https://blog.csdn.net/u010502101/article/details/78758330

**@RestController**

组合@Controller和@ResponseBody，当你开发一个和页面交互数据的控制时，比如bbs-web的api接口需要此注解

**@RequestMapping("/api2/copper")**

用来映射web请求(访问路径和参数)、处理类和方法，可以注解在类或方法上。注解在方法上的路径会继承注解在类上的路径。

produces属性: 定制返回的response的媒体类型和字符集，或需返回值是json对象

```java
@RequestMapping(value="/api2/copper",produces="application/json;charset=UTF-8",method = RequestMethod.POST)
```

**@RequestParam**

获取request请求的参数值

```java
 public List<CopperVO> getOpList(HttpServletRequest request,
                                    @RequestParam(value = "pageIndex", required = false) Integer pageIndex,
                                    @RequestParam(value = "pageSize", required = false) Integer pageSize) {
 
  }
```

**@ResponseBody**

**支持将返回值放在response体内，而不是返回一个页面**。比如Ajax接口，可以用此注解返回数据而不是页面。此注解可以放置在返回值前或方法前。

**@Bean**

@Bean(name="bean的名字",initMethod="初始化时调用方法名字",destroyMethod="close")

定义在方法上，在容器内初始化一个bean实例类。

```java
@Bean(destroyMethod="close")
@ConditionalOnMissingBean
public PersonService registryService() {
		return new PersonService();
	}
```

**@Service**

用于标注业务层组件

**@Controller**

用于标注控制层组件(如struts中的action)

**@Repository**

用于标注数据访问组件，即DAO组件

**@Component**

泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。

**@PostConstruct**

spring容器初始化时，要执行该方法

```java
@PostConstruct  
public void init() {   
}   
```

**@PathVariable**

用来获得请求url中的动态参数

```java
@Controller  
public class TestController {  

     @RequestMapping(value="/user/{userId}/roles/{roleId}",method = RequestMethod.GET)  
     public String getLogin(@PathVariable("userId") String userId,  
         @PathVariable("roleId") String roleId){
           
         System.out.println("User Id : " + userId);  
         System.out.println("Role Id : " + roleId);  
         return "hello";  
     
     }  
}
```

**@ComponentScan**

注解会告知Spring扫描指定的包来初始化Spring

```java
@ComponentScan(basePackages = "com.bbs.xx")
```

**@EnableZuulProxy**

路由网关的主要目的是为了让所有的微服务对外只有一个接口，我们只需访问一个网关地址，即可由网关将所有的请求代理到不同的服务中。Spring Cloud是通过Zuul来实现的，支持自动路由映射到在Eureka Server上注册的服务。Spring Cloud提供了注解@EnableZuulProxy来启用路由代理。

**@Autowired**

在默认情况下使用 @Autowired 注释进行自动注入时，Spring 容器中匹配的候选 Bean 数目必须有且仅有一个。当找不到一个匹配的 Bean 时，Spring 容器将抛出 BeanCreationException 异常，并指出必须至少拥有一个匹配的 Bean。

当不能确定 Spring 容器中一定拥有某个类的 Bean 时，可以在需要自动注入该类 Bean 的地方可以使用 @Autowired(required = false)，这等于告诉 Spring: 在找不到匹配 Bean 时也不报错

[@Autowired注解注入map、list与@Qualifier在新窗口打开](https://blog.csdn.net/ethunsex/article/details/66475792)

**@Configuration**

```java
@Configuration("name")//表示这是一个配置信息类,可以给这个配置类也起一个名称
@ComponentScan("spring4")//类似于xml中的<context:component-scan base-package="spring4"/>
public class Config {

    @Autowired//自动注入，如果容器中有多个符合的bean时，需要进一步明确
    @Qualifier("compent")//进一步指明注入bean名称为compent的bean
    private Compent compent;

    @Bean//类似于xml中的<bean id="newbean" class="spring4.Compent"/>
    public Compent newbean(){
        return new Compent();
    }   
}
```

**@Import(Config1.class)**

导入Config1配置类里实例化的bean

```java
@Configuration
public class CDConfig {

    @Bean   // 将SgtPeppers注册为 SpringContext中的bean
    public CompactDisc compactDisc() {
        return new CompactDisc();  // CompactDisc类型的
    }
}

@Configuration
@Import(CDConfig.class)  //导入CDConfig的配置
public class CDPlayerConfig {

    @Bean(name = "cDPlayer")
    public CDPlayer cdPlayer(CompactDisc compactDisc) {  
         // 这里会注入CompactDisc类型的bean
         // 这里注入的这个bean是CDConfig.class中的CompactDisc类型的那个bean
        return new CDPlayer(compactDisc);
    }
}
```

**@Order**

@Order(1)，值越小优先级超高，越先运行

**@ConditionalOnExpression**

```java
@Configuration
@ConditionalOnExpression("${enabled:false}")
public class BigpipeConfiguration {
    @Bean
    public OrderMessageMonitor orderMessageMonitor(ConfigContext configContext) {
        return new OrderMessageMonitor(configContext);
    }
}
```

开关为true的时候才实例化bean

**@ConditionalOnProperty**

这个注解能够控制某个 @Configuration 是否生效。具体操作是通过其两个属性name以及havingValue来实现的，其中name用来从application.properties中读取某个属性值，如果该值为空，则返回false;如果值不为空，则将该值与havingValue指定的值进行比较，如果一样则返回true;否则返回false。如果返回值为false，则该configuration不生效；为true则生效。

https://blog.csdn.net/dalangzhonghangxing/article/details/78420057

**@ConditionalOnClass**

该注解的参数对应的类必须存在，否则不解析该注解修饰的配置类

```java
@Configuration
@ConditionalOnClass({Gson.class})
public class GsonAutoConfiguration {
    public GsonAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean
    public Gson gson() {
        return new Gson();
    }
}
```

**@ConditionalOnMisssingClass({ApplicationManager.class})**

如果存在它修饰的类的bean，则不需要再创建这个bean；

**@ConditionOnMissingBean(name = "example")**

表示如果name为“example”的bean存在，该注解修饰的代码块不执行。

### SpringBoot接口 - 如何统一接口封装

#### RESTful API接口

- **什么是 REST**？

Representational State Transfer，翻译是“表现层状态转化”。可以总结为一句话：**REST 是所有 Web 应用都应该遵守的架构设计指导原则**。

- **什么是 RESTful API**？

**符合 REST 设计标准的 API**，即 RESTful API。REST 架构设计，遵循的各项标准和准则，就是 HTTP 协议的表现，换句话说，HTTP 协议就是属于 REST 架构的设计模式。比如，无状态，请求-响应。

#### 为什么要统一封装接口

现在大多数项目采用前后分离的模式进行开发，统一返回方便前端进行开发和封装，以及出现时给出响应编码和信息。

以查询某个用户接口而言，如果没有封装, 返回结果如下

```json
{
  "userId": 1,
  "userName": "赵一"
}
```

如果封装了，返回正常的结果如下：

```json
{
  "timestamp": 11111111111,
  "status": 200,
  "message": "success",
  "data": {
    "userId": 1,
    "userName": "赵一"
  }
}
```

异常返回结果如下：

```json
{
  "timestamp": 11111111111,
  "status": 10001,
  "message": "User not exist",
  "data": null
}
```

### SpringBoot接口 - 如何对参数进行校验

**请求参数封装**

单一职责，所以将查询用户的参数封装到UserParam中， 而不是User（数据库实体）本身。

对每个参数字段添加validation注解约束和message。

**Controller中获取参数绑定结果**

使用@Valid或者@Validated注解，参数校验的值放在BindingResult中

**@Validated和@Valid什么区别？**

- **分组**

@Validated：提供了一个分组功能，可以在入参验证时，根据不同的分组采用不同的验证机制，这个网上也有资料，不详述。@Valid：作为标准JSR-303规范，还没有吸收分组的功能。

- **注解地方**

@Validated：可以用在类型、方法和方法参数上。但是不能用在成员属性（字段）上

@Valid：可以用在方法、构造函数、方法参数和成员属性（字段）上

- **嵌套类型**

比如本文例子中的address是user的一个嵌套属性, 只能用@Valid

### SpringBoot接口 - 如何参数校验国际化

**参数校验国际化**

1. 在Resources下添加resource bundle,填写名称和资源语言类型,添加中英文对应的message
2. **中英文切换拦截：**由于默认是拦截request参数获取locale参数来实现的切换语言，这里我们可以改下，优先从header中获取，如果没有获取到再从request参数中获取。

### SpringBoot接口 - 如何统一异常处理

#### **实现案例**

> 简单展示通过@ControllerAdvice进行统一异常处理。

**@ControllerAdvice异常统一处理**

对于400参数错误异常

```java
/**
 * Global exception handler.
 *
 * @author pdai
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    /**
     * exception handler for bad request.
     *
     * @param e
     *            exception
     * @return ResponseResult
     */
    @ResponseBody
    @ResponseStatus(code = HttpStatus.BAD_REQUEST)
    @ExceptionHandler(value = { BindException.class, ValidationException.class, MethodArgumentNotValidException.class })
    public ResponseResult<ExceptionData> handleParameterVerificationException(@NonNull Exception e) {
        ExceptionData.ExceptionDataBuilder exceptionDataBuilder = ExceptionData.builder();
        log.warn("Exception: {}", e.getMessage());
        if (e instanceof BindException) {
            BindingResult bindingResult = ((MethodArgumentNotValidException) e).getBindingResult();
            bindingResult.getAllErrors().stream().map(DefaultMessageSourceResolvable::getDefaultMessage)
                    .forEach(exceptionDataBuilder::error);
        } else if (e instanceof ConstraintViolationException) {
            if (e.getMessage() != null) {
                exceptionDataBuilder.error(e.getMessage());
            }
        } else {
            exceptionDataBuilder.error("invalid parameter");
        }
        return ResponseResultEntity.fail(exceptionDataBuilder.build(), "invalid parameter");
    }

}
```

对于自定义异常

```java
/**
 * handle business exception.
 *
 * @param businessException
 *            business exception
 * @return ResponseResult
 */
@ResponseBody
@ExceptionHandler(BusinessException.class)
public ResponseResult<BusinessException> processBusinessException(BusinessException businessException) {
    log.error(businessException.getLocalizedMessage(), businessException);
    // 这里可以屏蔽掉后台的异常栈信息，直接返回"business error"
    return ResponseResultEntity.fail(businessException, businessException.getLocalizedMessage());
}
```

对于其它异常

```java
/**
 * handle other exception.
 *
 * @param exception
 *            exception
 * @return ResponseResult
 */
@ResponseBody
@ExceptionHandler(Exception.class)
public ResponseResult<Exception> processException(Exception exception) {
    log.error(exception.getLocalizedMessage(), exception);
    // 这里可以屏蔽掉后台的异常栈信息，直接返回"server error"
    return ResponseResultEntity.fail(exception, exception.getLocalizedMessage());
}
```

**Controller接口**

（接口中无需处理异常）

```java
@Slf4j
@Api(value = "User Interfaces", tags = "User Interfaces")
@RestController
@RequestMapping("/user")
public class UserController {

    /**
     * http://localhost:8080/user/add .
     *
     * @param userParam user param
     * @return user
     */
    @ApiOperation("Add User")
    @ApiImplicitParam(name = "userParam", type = "body", dataTypeClass = UserParam.class, required = true)
    @PostMapping("add")
    public ResponseEntity<UserParam> add(@Valid @RequestBody UserParam userParam) {
        return ResponseEntity.ok(userParam);
    }
}
```

#### @ControllerAdvice还可以怎么用？

**除了通过@ExceptionHandler注解用于全局异常的处理之外**，@ControllerAdvice还有两个用法：

- **@InitBinder注解**

**用于请求中注册自定义参数的解析，从而达到自定义请求参数格式的目的；**

比如，在@ControllerAdvice注解的类中添加如下方法，来统一处理日期格式的格式化

```java
@InitBinder
public void handleInitBinder(WebDataBinder dataBinder){
    dataBinder.registerCustomEditor(Date.class,
            new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"), false));
}
```

Controller中传入参数（string类型）自动转化为Date类型

```java
@GetMapping("testDate")
public Date processApi(Date date) {
    return date;
}
```

- **@ModelAttribute注解**

**用来预设全局参数**，比如最典型的使用Spring Security时将添加当前登录的用户信息（UserDetails)作为参数。

```java
@ModelAttribute("currentUser")
public UserDetails modelAttribute() {
    return (UserDetails) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
}
```

所有controller类中requestMapping方法都可以直接获取并使用currentUser

```java
@PostMapping("saveSomething")
public ResponseEntity<String> saveSomeObj(@ModelAttribute("currentUser") UserDetails operator) {
    // 保存操作，并设置当前操作人员的ID（从UserDetails中获得）
    return ResponseEntity.success("ok");
}
```



**@ControllerAdvice是如何起作用的（原理）？**

DispatcherServlet中onRefresh方法是初始化ApplicationContext后的回调方法，它会调用initStrategies方法，主要更新一些servlet需要使用的对象，包括国际化处理，requestMapping，视图解析等等。

```
protected void initStrategies(ApplicationContext context) {
    initMultipartResolver(context); // 文件上传
    initLocaleResolver(context); // i18n国际化
    initThemeResolver(context); // 主题
    initHandlerMappings(context); // requestMapping
    initHandlerAdapters(context); // adapters
    initHandlerExceptionResolvers(context); // 异常处理
    initRequestToViewNameTranslator(context);
    initViewResolvers(context);
    initFlashMapManager(context);
}
```

**@ModelAttribute和@InitBinder**处理在**RequestMappingHandlerAdapter**中的afterPropertiesSet去处理advice

**@ExceptionHandler**显然是在上述initHandlerExceptionResolvers(context)方法中。ExceptionHandlerExceptionResolver中的在afterPropertiesSet去处理advice

### SpringBoot接口 - 如何提供多个版本接口

**有哪些控制接口多版本的方式？**

- 相同URL，用**不同的版本参数**区分
  - `api.pdai.tech/user?version=v1` 表示 v1版本的接口, 保持原有接口不动
  - `api.pdai.tech/user?version=v2` 表示 v2版本的接口，更新新的接口
- 区分**不同的接口域名**，不同的版本有不同的子域名, 路由到不同的实例:
  - `v1.api.pdai.tech/user` 表示 v1版本的接口, 保持原有接口不动, 路由到instance1
  - `v2.api.pdai.tech/user` 表示 v2版本的接口，更新新的接口, 路由到instance2
- 网关路由不同子目录到**不同的实例**（不同package也可以）
  - `api.pdai.tech/v1/user` 表示 v1版本的接口, 保持原有接口不动, 路由到instance1
  - `api.pdai.tech/v2/user` 表示 v2版本的接口，更新新的接口, 路由到instance2
- **同一实例**，用注解隔离不同版本控制
  - `api.pdai.tech/v1/user` 表示 v1版本的接口, 保持原有接口不动，匹配@ApiVersion("1")的handlerMapping
  - `api.pdai.tech/v2/user` 表示 v2版本的接口，更新新的接口，匹配@ApiVersion("2")的handlerMapping

**实现步骤**

1. **自定义@ApiVersion注解**
2. **定义版本匹配RequestCondition**：版本匹配支持三层版本 v1.1.1 （大版本.小版本.补丁版本）v1.1 (等同于v1.1.0)v1 （等同于v1.0.0)
3. **定义HandlerMapping**
4. **配置注册HandlerMapping或者实现WebMvcRegistrations的接口**

### SpringBoot接口 - 如何生成接口文档之Swagger技术栈

**什么是OpenAPI规范（OAS)？**

[OpenAPI 规范（OAS）在新窗口打开](https://fishead.gitbook.io/openapi-specification-zhcn-translation/3.0.0.zhcn#revisionHistory)定义了一个标准的、语言无关的 **RESTful API 接口规范**，它可以同时允许开发人员和操作系统查看并理解某个服务的功能，而无需访问源代码，文档或网络流量检查（既方便人类学习和阅读，也方便机器阅读）。正确定义 OAS 后，开发者可以使用最少的实现逻辑来理解远程服务并与之交互。

此外，文档生成工具可以使用 OpenAPI 规范来生成 API 文档，代码生成工具可以生成各种编程语言下的服务端和客户端代码，测试代码和其他用例。

官方GitHub地址： [OpenAPI-Specification在新窗口打开](https://github.com/OAI/OpenAPI-Specification)

**[#](#什么是swagger) 什么是Swagger？**

Swagger 是一个用于生成、描述和调用 RESTful 接口的 Web 服务。通俗的来讲，Swagger 就是将**项目中所有（想要暴露的）接口展现在页面上，并且可以进行接口调用和测试的服务**。

从上述 Swagger 定义我们不难看出 Swagger 有以下 3 个重要的作用：

- 将项目中所有的接口**展现在页面**上，这样后端程序员就不需要专门为前端使用者编写专门的接口文档；
- 当接口更新之后，只需要修改代码中的 Swagger 描述就可以**实时生成**新的接口文档了，从而规避了接口文档老旧不能使用的问题；
- 通过 Swagger 页面，我们可以直接进行**接口调用**，降低了项目开发阶段的调试成本。

Swagger3完全遵循了 OpenAPI 规范。Swagger 官网地址：[https://swagger.io/在新窗口打开](https://swagger.io/)。

**[#](#swagger和springfox有啥关系) Swagger和SpringFox有啥关系？**

Swagger 可以看作是一个遵循了 OpenAPI 规范的一项技术，而 springfox 则是这项技术的具体实现。 就好比 Spring 中的 IOC 和 DI 的关系 一样，前者是思想，而后者是实现。

**[#](#什么是knife4j-和swagger什么关系) 什么是Knife4J? 和Swagger什么关系？**

> **本质是Swagger的增强解决方案，前身只是一个SwaggerUI（swagger-bootstrap-ui）**

Knife4j是为Java MVC框架集成Swagger生成Api文档的增强解决方案, 前身是swagger-bootstrap-ui,取名kni4j是希望她能像一把匕首一样小巧,轻量,并且功能强悍!

Knife4j的前身是swagger-bootstrap-ui，为了契合微服务的架构发展,由于原来swagger-bootstrap-ui采用的是后端Java代码+前端Ui混合打包的方式,在微服务架构下显的很臃肿,因此项目正式更名为knife4j

更名后主要专注的方面

- 前后端Java代码以及前端Ui模块进行分离,在微服务架构下使用更加灵活
- 提供**专注于Swagger的增强解决方案**,不同于只是改善增强前端Ui部分

### SpringBoot接口 - 如何生成接口文档之集成Smart-Doc

上文我们看到可以通过Swagger系列可以快速生成API文档， 但是这种API文档生成是需要在接口上**添加注解**等，这表明这是一种**侵入式**方式； 

既然有了Swagger， 为何还会产生Smart-Doc这类工具呢？ 本质上是**Swagger侵入性和依赖性**。

### SpringBoot接口 - 如何访问外部接口

**访问外部接口场景：**调用其它模块的API，或者其它三方服务，比如调用外部的地图API或者天气API等。

#### 访问外部接口的常见方案

##### **方案一: 采用原生的Http请求**

##### **方案二: 采用Feign进行消费**

1、在maven项目中添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <version>1.2.2.RELEASE</version>
</dependency>
```

2、编写接口，放置在service层

这里的decisionEngine.url 是配置在properties中的 是ip地址和端口号 decisionEngine.url=http://10.2.1.148:3333/decision/person 是接口名字

```java
@FeignClient(url = "${decisionEngine.url}",name="engine")
public interface DecisionEngineService {
　　@RequestMapping(value="/decision/person",method= RequestMethod.POST)
　　public JSONObject getEngineMesasge(@RequestParam("uid") String uid,@RequestParam("productCode") String productCode);

}
```

3、在Java的启动类上加上@EnableFeignClients

```java
@EnableFeignClients //参见此处
@EnableDiscoveryClient
@SpringBootApplication
@EnableResourceServer
public class Application   implements CommandLineRunner {
    private static final Logger LOGGER = LoggerFactory.getLogger(Application.class);
    @Autowired
    private AppMetricsExporter appMetricsExporter;

    @Autowired
    private AddMonitorUnitService addMonitorUnitService;

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }    
}
```

4、在代码中调用接口即可

```java
@Autowired
private DecisionEngineService decisionEngineService ;
// ...
decisionEngineService.getEngineMesasge("uid" ,  "productCode");
```

##### **方案三: 采用RestTemplate方法**

> 在Spring-Boot开发中，RestTemplate同样提供了对外访问的接口API，这里主要介绍Get和Post方法的使用。Get请求提供了两种方式的接口getForObject 和 getForEntity，getForEntity提供如下三种方法的实现。

**[#](#get请求之——getforentity-stringurl-class-responsetype-object-urlvariables) Get请求之——getForEntity(Stringurl,Class responseType,Object…urlVariables)**

该方法提供了三个参数，其中url为请求的地址，responseType为请求响应body的包装类型，urlVariables为url中的参数绑定，该方法的参考调用如下:

```java
// http://USER-SERVICE/user?name={name)
RestTemplate restTemplate=new RestTemplate();
Map<String,String> params=new HashMap<>();
params.put("name","dada");  //
ResponseEntity<String> responseEntity=restTemplate.getForEntity("http://USERSERVICE/user?name={name}",String.class,params);
```

**[#](#get请求之——getforentity-uri-url-class-responsetype) Get请求之——getForEntity(URI url,Class responseType)**

该方法使用URI对象来替代之前的url和urlVariables参数来指定访问地址和参数绑定。URI是JDK java.net包下的一个类，表示一个统一资源标识符(Uniform Resource Identifier)引用。参考如下:

```java
RestTemplate restTemplate=new RestTemplate();
UriComponents uriComponents=UriComponentsBuilder.fromUriString("http://USER-SERVICE/user?name={name}")
    .build()
    .expand("dodo")
    .encode();
URI uri=uriComponents.toUri();
ResponseEntity<String> responseEntity=restTemplate.getForEntity(uri,String.class).getBody();
```

**[#](#get请求之——getforobject) Get请求之——getForObject**

getForObject方法可以理解为对getForEntity的进一步封装，它通过HttpMessageConverterExtractor对HTTP的请求响应体body内容进行对象转换，实现请求直接返回包装好的对象内容。getForObject方法有如下:

```java
getForObject(String url,Class responseType,Object...urlVariables)
getForObject(String url,Class responseType,Map urlVariables)
getForObject(URI url,Class responseType)
```

**[#](#post-请求) Post 请求**

Post请求提供有三种方法，postForEntity、postForObject和postForLocation。其中每种方法都存在三种方法，postForEntity方法使用如下:

```java
RestTemplate restTemplate=new RestTemplate();
User user=newUser("didi",30);
ResponseEntity<String> responseEntity=restTemplate.postForEntity("http://USER-SERVICE/user",user,String.class); //提交的body内容为user对象，请求的返回的body类型为String
String body=responseEntity.getBody();
```

postForEntity存在如下三种方法的重载

```java
postForEntity(String url,Object request,Class responseType,Object... uriVariables)
postForEntity(String url,Object request,Class responseType,Map uriVariables)
postForEntity(URI url,Object request，Class responseType)
```

postForEntity中的其它参数和getForEntity的参数大体相同在此不做介绍。

#### 在接口调用中需要注意什么?

需要注意两点：

1. 需要设置超时时间
2. 需要自行处理异常

### SpringBoot接口 - 如何保证接口幂等

#### 什么是幂等？

**幂等接口：**以相同的请求调用这个接口一次和调用这个接口多次，对系统产生的影响是相同的。

- **接口幂等和防止重复提交是一回事吗**？

严格来说，并不是。

1. **幂等**: 更多的是在重复请求已经发生，或是无法避免的情况下，采取一定的技术手段让这些重复请求不给系统带来副作用。
2. **防止重复**: 提交更多的是不让用户发起多次一样的请求。

#### 什么是接口幂等？

它描述了一次和多次请求某一个资源对于资源本身应该具有同样的结果（网络超时等问题除外），即第一次请求的时候对资源产生了副作用，但是以后的多次请求都不会再对资源产生副作用。

- **对哪些类型的接口需要保证接口幂等**？（增删改查）

我们看下标准的restful请求，幂等情况是怎么样的：

1. **SELECT查询操作**
   1. GET：只是获取资源，对资源本身没有任何副作用，天然的幂等性。
   2. HEAD：本质上和GET一样，获取头信息，主要是探活的作用，具有幂等性。
   3. OPTIONS：获取当前URL所支持的方法，因此也是具有幂等性的。
2. **DELETE删除操作**
   1. 删除的操作，如果从删除的一次和删除多次的角度看，数据并不会变化，这个角度看它是幂等的
   2. 但是如果，从另外一个角度，删除数据一般是返回受影响的行数，删除一次和多次删除返回的受影响行数是不一样的，所以从这个角度它需要保证幂等。（折中而言DELETE操作通常也会被纳入保证接口幂等的要求）
3. **ADD/EDIT操作**
   1. PUT：用于更新资源，有副作用，但是它应该满足幂等性，比如根据id更新数据，调用多次和N次的作用是相同的（根据业务需求而变）。
   2. POST：用于添加资源，多次提交很可能产生副作用，比如订单提交，多次提交很可能产生多笔订单。

#### 常见的保证幂等的方式？

##### 数据库层面

**悲观锁**：典型的数据库悲观锁：`for update`

```
select * from t_order where order_id = trade_no for update;
```

**唯一ID/索引**：针对**插入**操作

数据库唯一主键的实现主要是利用数据库中主键唯一约束的特性，一般来说唯一主键比较适用于“插入”时的幂等性，其能保证一张表中只能存在一条带该唯一主键的记录。

使用数据库唯一主键完成幂等性时需要注意的是，该主键一般来说并不是使用数据库中自增主键，而是使用分布式 ID 充当主键，这样才能保证在分布式环境下 ID 的**全局唯一性**。

- **去重表**

去重表本质上也是一种**唯一索引**方案。

这种方法适用于在业务中有唯一标的插入场景中，比如在以上的支付场景中，如果一个订单只会支付一次，所以订单ID可以作为唯一标识。这时，我们就可以建一张去重表，并且把唯一标识作为唯一索引，在我们实现时，把创建支付单据和写入去去重表，放在一个事务中，如果重复创建，数据库会抛出唯一约束异常，操作就会回滚。

**乐观锁（基于版本号或者时间戳）**：针对**更新**操作。

- **状态机**

本质上也是乐观锁，这种方法适合在有状态机流转的情况下，比如就会订单的创建和付款，订单的付款肯定是在之前，这时我们可以通过在设计状态字段时，使用int类型，并且通过值类型的大小来做幂等，比如订单的创建为0，付款成功为100。付款失败为99

##### 分布式锁

分布式锁实现幂等性的逻辑是，在每次执行方法之前判断，是否可以获取到分布式锁，如果可以，则表示为第一次执行方法，否则直接舍弃请求即可。

需要注意的是分布式锁的key必须为业务的唯一标识，通常用redis分布式锁或者zookeeper来实现分布式锁。

##### token机制

### SpringBoot接口 - 如何对接口进行签名

#### 常见的保证接口安全的方式？

##### AccessKey&SecretKey

> 这种设计一般用在开发接口的安全，以确保是一个**合法的开发者**。

- AccessKey： 开发者唯一标识
- SecretKey: 开发者密钥

##### 认证和授权

从两个视角去看

- 第一: **认证和授权**，认证是访问者的合法性，授权是访问者的权限分级；
- 第二: 其中认证包括**对客户端的认证**以及**对用户的认证**；

- **对于客户端的认证**

典型的是AppKey&AppSecret，或者ClientId&ClientSecret等

比如oauth2协议的client cridential模式

```bash
https://api.xxxx.com/token?grant_type=client_credentials&client_id=CLIENT_ID&client_secret=CLIENT_SECRET
```

grant_type参数等于client_credentials表示client credentials方式，client_id是客户端id，client_secret是客户端密钥。

返回token后，通过token访问其它接口。

- **对于用户的认证和授权**

比如oauth2协议的授权码模式(authorization code)和密码模式(resource owner password credentials)

```bash
https://api.xxxx.com/token?grant_type=password&username=USERNAME&password=PASSWORD&client_id=CLIENT_ID&scope=read
```

grant_type参数等于password表示密码方式，client_id是客户端id，username是用户名，password是密码。

(PS：password模式只有在授权码模式(authorization code)不可用时才会采用，这里只是举个例子而已)

可选参数scope表示申请的权限范围。（相关开发框架可以参考spring security, Apache Shiro，[SA-Token在新窗口打开](https://sa-token.dev33.cn/doc/index.html)等）

##### https

> 从接口传输安全的角度，防止接口数据**明文传输**

HTTP 有以下安全性问题:

- 使用明文进行通信，内容可能会被窃听；
- 不验证通信方的身份，通信方的身份有可能遭遇伪装；
- 无法证明报文的完整性，报文有可能遭篡改。

**HTTPs 并不是新协议，而是让 HTTP 先和 SSL(Secure Sockets Layer)通信，再由 SSL 和 TCP 通信，也就是说 HTTPs 使用了隧道进行通信。**

**通过使用 SSL，HTTPs 具有了加密(防窃听)、认证(防伪装)和完整性保护(防篡改)。**

##### 接口签名（加密）

接口签名（加密），主要**防止请求参数被篡改**。特别是安全要求比较高的接口，比如支付领域的接口。

- **签名的主要流程**

首先我们需要分配给客户端一个私钥用于URL签名加密，一般的签名算法如下：

1、首先对请求参数按key进行字母排序放入有序集合中（其它参数请参看后续补充部分）；

2、对排序完的数组键值对用&进行连接，形成用于加密的参数字符串；

3、在加密的参数字符串前面或者后面加上私钥，然后用加密算法进行加密，得到sign，然后随着请求接口一起传给服务器。

例如： https://api.xxxx.com/token?key=value&timetamp=xxxx&sign=xxxx-xxx-xxx-xxxx

服务器端接收到请求后，用同样的算法获得服务器的sign，对比客户端的sign是否一致，如果一致请求有效；如果不一致返回指定的错误信息。

- **补充：对什么签名？**

1. 主要包括请求参数，这是最主要的部分，**签名的目的要防止参数被篡改，就要对可能被篡改的参数签名**；
2. 同时考虑到请求参数的来源可能是请求路径path中，请求header中，请求body中。
3. 如果对客户端分配了AppKey&AppSecret，也可加入签名计算；
4. 考虑到其它幂等，token失效等，也会将涉及的参数一并加入签名，比如timestamp，流水号nonce等（这些参数可能来源于header）

- **补充: 签名算法？**

一般涉及这块，主要包含三点：密钥，签名算法，签名规则

1. **密钥secret**： 前后端约定的secret，这里要注意前端可能无法妥善保存好secret，比如SPA单页应用；
2. **签名算法**：也不一定要是对称加密算法，对称是反过来解析sign，这里是用同样的算法和规则计算出sign，并对比前端传过来的sign是否一致。
3. **签名规则**：比如多次加盐加密等；

- **补充：签名和加密是不是一回事？**

严格来说不是一回事：

1. **签名**是通过对参数按照指定的算法、规则计算出sign，最后前后端通过同样的算法计算出sign是否一致来防止参数篡改的，所以你可以看到参数是明文的，只是多加了一个计算出的sign。
2. **加密**是对请求的参数加密，后端进行解密；同时有些情况下，也会对返回的response进行加密，前端进行解密；这里存在加密和解密的过程，所以思路上必然是对称加密的形式+时间戳接口时效性等。

- **补充：签名放在哪里？**

签名可以放在请求参数中（path中，body中等），更为优雅的可以放在HEADER中，比如X-Sign（通常第三方的header参数以X-开头）

### SpringBoot接口 - 如何实现接口限流之单实例

#### 为什么要限流

每个系统都有服务的上线，所以当流量超过服务极限能力时，系统可能会出现卡死、崩溃的情况，所以就有了降级和限流。限流其实就是：当高并发或者瞬时高并发时，为了保证系统的稳定性、可用性，系统以牺牲部分请求为代价或者延迟处理请求为代价，保证系统整体服务可用。

### [#](#限流有哪些常见思路) 限流有哪些常见思路？

- **从算法上看**

令牌桶(Token Bucket)、漏桶(leaky bucket)和计数器算法是最常用的三种限流的算法。

- **单实例**

应用级限流方式只是单应用内的请求限流，不能进行全局限流。

1. 限流总资源数
2. 限流总并发/连接/请求数
3. 限流某个接口的总并发/请求数
4. 限流某个接口的时间窗请求数
5. 平滑限流某个接口的请求数
6. Guava RateLimiter

- **分布式**

我们需要**分布式限流**和**接入层限流**来进行全局限流。

1. redis+lua实现中的lua脚本
2. 使用Nginx+Lua实现的Lua脚本
3. 使用 OpenResty 开源的限流方案
4. 限流框架，比如Sentinel实现降级限流熔断

### SpringBoot集成MySQL - MyBatis XML方式

#### 什么是MyBatis？

> MyBatis是一款优秀的基于java的持久层框架，它内部封装了jdbc，使开发者只需要关注sql语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程。

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。

- mybatis是一个优秀的基于java的持久层框架，它内部封装了jdbc，使开发者只需要关注sql语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程。
- mybatis通过xml或注解的方式将要执行的各种statement配置起来，并通过java对象和statement中sql的动态参数进行映射生成最终执行的sql语句，最后由mybatis框架执行sql并将结果映射为java对象并返回。

**MyBatis的主要设计目**的就是让我们**对执行SQL语句时对输入输出的数据管理更加方便**，所以方便地写出SQL和方便地获取SQL的执行结果才是MyBatis的核心竞争力。

**Mybatis的功能架构分为三层**：

- **API接口层**：提供给外部使用的接口API，开发人员通过这些本地API来操纵数据库。接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理。
- **数据处理层**：负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。
- **基础支撑层**：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。

更多介绍可以参考：[MyBatis3 官方网站在新窗口打开](https://mybatis.org/mybatis-3/)

#### [#](#为什么说mybatis是半自动orm) 为什么说MyBatis是半自动ORM？

> 为什么说MyBatis是半自动ORM？

- **什么是全自动ORM**？

ORM框架可以根据对象关系模型直接获取，查询关联对象或者关联集合对象，简单而言使用全自动的ORM框架查询时可以不再写SQL。典型的框架如Hibernate； 因为Spring-data-jpa很多代码也是Hibernate团队贡献的，所以spring-data-jpa也是全自动ORM框架。

- **MyBatis是半自动ORM**？

Mybatis 在查询关联对象或关联集合对象时，需要手动编写 sql 来完成，所以，称之为半自动ORM 映射工具。

#### XML方式

**Mapper文件**

mapper文件定义在配置的路径中（`classpath:mybatis/mapper/*.xml`）

UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="tech.pdai.springboot.mysql57.mybatis.xml.dao.IUserDao">

	<resultMap type="tech.pdai.springboot.mysql57.mybatis.xml.entity.User" id="UserResult">
		<id     property="id"       	column="id"      		/>
		<result property="userName"     column="user_name"    	/>
		<result property="password"     column="password"    	/>
		<result property="email"        column="email"        	/>
		<result property="phoneNumber"  column="phone_number"  	/>
		<result property="description"  column="description"  	/>
		<result property="createTime"   column="create_time"  	/>
		<result property="updateTime"   column="update_time"  	/>
		<collection property="roles" ofType="tech.pdai.springboot.mysql57.mybatis.xml.entity.Role">
			<result property="id" column="id"  />
			<result property="name" column="name"  />
			<result property="roleKey" column="role_key"  />
			<result property="description" column="description"  />
			<result property="createTime"   column="create_time"  	/>
			<result property="updateTime"   column="update_time"  	/>
		</collection>
	</resultMap>
	
	<sql id="selectUserSql">
        select u.id, u.password, u.user_name, u.email, u.phone_number, u.description, u.create_time, u.update_time, r.name, r.role_key, r.description, r.create_time, r.update_time
		from tb_user u
		left join tb_user_role ur on u.id=ur.user_id
		inner join tb_role r on ur.role_id=r.id
    </sql>
	
	<select id="findList" parameterType="tech.pdai.springboot.mysql57.mybatis.xml.entity.query.UserQueryBean" resultMap="UserResult">
		<include refid="selectUserSql"/>
		where u.id != 0
		<if test="userName != null and userName != ''">
			AND u.user_name like concat('%', #{user_name}, '%')
		</if>
		<if test="description != null and description != ''">
			AND u.description like concat('%', #{description}, '%')
		</if>
		<if test="phoneNumber != null and phoneNumber != ''">
			AND u.phone_number like concat('%', #{phoneNumber}, '%')
		</if>
		<if test="email != null and email != ''">
			AND u.email like concat('%', #{email}, '%')
		</if>
	</select>
	
	<select id="findById" parameterType="Long" resultMap="UserResult">
		<include refid="selectUserSql"/>
		where u.id = #{id}
	</select>
	
	<delete id="deleteById" parameterType="Long">
 		delete from tb_user where id = #{id}
 	</delete>
 	
 	<delete id="deleteByIds" parameterType="Long">
		delete from tb_user where id in
 		<foreach collection="array" item="id" open="(" separator="," close=")">
 			#{id}
        </foreach> 
 	</delete>
 	
 	<update id="update" parameterType="tech.pdai.springboot.mysql57.mybatis.xml.entity.User">
 		update tb_user
 		<set>
 			<if test="userName != null and userName != ''">user_name = #{userName},</if>
 			<if test="email != null and email != ''">email = #{email},</if>
 			<if test="phoneNumber != null and phoneNumber != ''">phone_number = #{phoneNumber},</if>
			<if test="description != null and description != ''">description = #{description},</if>
 			update_time = sysdate()
 		</set>
 		where id = #{id}
	</update>

	<update id="updatePassword" parameterType="tech.pdai.springboot.mysql57.mybatis.xml.entity.User">
		update tb_user
		<set>
			password = #{password}, update_time = sysdate()
		</set>
		where id = #{id}
	</update>
 	
 	<insert id="save" parameterType="tech.pdai.springboot.mysql57.mybatis.xml.entity.User" useGeneratedKeys="true" keyProperty="id">
 		insert into tb_user(
 			<if test="userName != null and userName != ''">user_name,</if>
			<if test="password != null and password != ''">password,</if>
 			<if test="email != null and email != ''">email,</if>
			<if test="phoneNumber != null and phoneNumber != ''">phone_number,</if>
 			<if test="description != null and description != ''">description,</if>
 			create_time,
			update_time
 		)values(
 			<if test="userName != null and userName != ''">#{userName},</if>
			<if test="password != null and password != ''">#{password},</if>
 			<if test="email != null and email != ''">#{email},</if>
 			<if test="phoneNumber != null and phoneNumber != ''">#{phone_number},</if>
 			<if test="description != null and description != ''">#{description},</if>
 			sysdate(),
			sysdate()
 		)
	</insert>
	
</mapper> 
```

### SpringBoot集成MySQL - MyBatis 注解方式

**@Results和@Result注解**

> 对于xml配置查询时定义的ResultMap, 在注解中如何定义呢？

```xml
<resultMap type="tech.pdai.springboot.mysql57.mybatis.xml.entity.User" id="UserResult1">
  <id     property="id"       	column="id"      		/>
  <result property="userName"     column="user_name"    	/>
  <result property="password"     column="password"    	/>
  <result property="email"        column="email"        	/>
  <result property="phoneNumber"  column="phone_number"  	/>
  <result property="description"  column="description"  	/>
  <result property="createTime"   column="create_time"  	/>
  <result property="updateTime"   column="update_time"  	/>
</resultMap>
```

使用注解方式，用@Results注解对应

```java
@Results(
        id = "UserResult1",
        value = {
                @Result(id = true, property = "id", column = "id"),
                @Result(property = "userName", column = "user_name"),
                @Result(property = "password", column = "password"),
                @Result(property = "email", column = "email"),
                @Result(property = "phoneNumber", column = "phone_number"),
                @Result(property = "description", column = "description"),
                @Result(property = "createTime", column = "create_time"),
                @Result(property = "updateTime", column = "update_time")
        }
)
```

#### xml方式和注解方式融合

xml方式和注解方式是可以融合写的， 我们可以将复杂的SQL写在xml中

比如将resultMap定义在xml中

```xml
<resultMap type="tech.pdai.springboot.mysql57.mybatis.xml.entity.User" id="UserResult3">
  <id     property="id"       	column="id"      		/>
  <result property="userName"     column="user_name"    	/>
  <result property="password"     column="password"    	/>
  <result property="email"        column="email"        	/>
  <result property="phoneNumber"  column="phone_number"  	/>
  <result property="description"  column="description"  	/>
  <result property="createTime"   column="create_time"  	/>
  <result property="updateTime"   column="update_time"  	/>
  <collection property="roles" ofType="tech.pdai.springboot.mysql57.mybatis.xml.entity.Role">
    <result property="id" column="id"  />
    <result property="name" column="name"  />
    <result property="roleKey" column="role_key"  />
    <result property="description" column="description"  />
    <result property="createTime"   column="create_time"  	/>
    <result property="updateTime"   column="update_time"  	/>
  </collection>
</resultMap>
```

在方法中用@ResultMap

```java
@ResultMap("UserResult3")
@Select("select u.id, u.password, u.user_name, u.email, u.phone_number, u.description, u.create_time, u.update_time from tb_user u")
User findAll1();
```

### SpringBoot集成MySQL - MyBatis PageHelper分页

#### 为什么会出现PageHelper这类框架？

这便要从逻辑分页和物理分页开始说起：

- **逻辑分页**：从数据库将所有记录查询出来，存储到内存中，展示当前页，然后数据再直接从内存中获取（前台分页）
- **物理分页**：只从数据库中查询当前页的数据（后台分页）

由于**MyBatis默认实现中采用的是逻辑分页，所以才诞生了PageHelper一类的物理分页框架**。hibernate不要是因为hibernate采用的就是物理分页。

**PageHelper是如何实现物理分页的前提:MyBatis的插件机制**

Mybatis的分页功能很弱，它是基于内存的分页（查出所有记录再按偏移量和limit取结果），在大数据量的情况下这样的分页基本上是没有用的。本文基于插件，通过**拦截StatementHandler重写sql语句**，实现数据库的**物理分页**

#### PageHelper分页用法

**第一种：RowBounds方式的调用**

第一种，RowBounds方式的调用

```java
List<User> list = sqlSession.selectList("x.y.selectIf", null, new RowBounds(0, 10));
```

**[#](#第二种-mapper接口方式的调用startpage) 第二种：Mapper接口方式的调用startPage**

第二种，Mapper接口方式的调用，推荐这种使用方式。

```java
PageHelper.startPage(1, 10);
List<User> list = userMapper.selectIf(1);
```

**[#](#第三种-mapper接口方式的调用offsetpage) 第三种：Mapper接口方式的调用offsetPage**

第三种，Mapper接口方式的调用，推荐这种使用方式。

```java
PageHelper.offsetPage(1, 10);
List<User> list = userMapper.selectIf(1);
```

**[#](#第四种-参数方法调用) 第四种:参数方法调用**

第四种:参数方法调用

```java
//存在以下 Mapper 接口方法，你不需要在 xml 处理后两个参数
public interface CountryMapper {
    List<User> selectByPageNumSize(
            @Param("user") User user,
            @Param("pageNum") int pageNum, 
            @Param("pageSize") int pageSize);
}
//配置supportMethodsArguments=true
//在代码中直接调用：
List<User> list = userMapper.selectByPageNumSize(user, 1, 10);
```

**[#](#第五种-参数对象) 第五种:参数对象**

```java
//如果 pageNum 和 pageSize 存在于 User 对象中，只要参数有值，也会被分页
//有如下 User 对象
public class User {
    //其他fields
    //下面两个参数名和 params 配置的名字一致
    private Integer pageNum;
    private Integer pageSize;
}
//存在以下 Mapper 接口方法，你不需要在 xml 处理后两个参数
public interface CountryMapper {
    List<User> selectByPageNumSize(User user);
}
//当 user 中的 pageNum!= null && pageSize!= null 时，会自动分页
List<User> list = userMapper.selectByPageNumSize(user);
```

**[#](#第六种-iselect-接口方式) 第六种：ISelect 接口方式**

jdk6,7用法，创建接口

```java
Page<User> page = PageHelper.startPage(1, 10).doSelectPage(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectGroupBy();
    }
});
```

jdk8 lambda用法

```java
Page<User> page = PageHelper.startPage(1, 10).doSelectPage(()-> userMapper.selectGroupBy());
```

也可以直接返回PageInfo，注意doSelectPageInfo方法和doSelectPage

```java
pageInfo = PageHelper.startPage(1, 10).doSelectPageInfo(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectGroupBy();
    }
});
```

对应的lambda用法

```java
pageInfo = PageHelper.startPage(1, 10).doSelectPageInfo(() -> userMapper.selectGroupBy());
```

count查询，返回一个查询语句的count数

~~~java
long total = PageHelper.count(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectLike(user);
    }
});

对应的lambda用法

```java
total = PageHelper.count(()->userMapper.selectLike(user));
~~~

#### PageHelper是如何实现分页的？

理解它的原理，有两个点：

1. 第一，相对对于JDBC这种嵌入式的分页而言，PageHelper分页是独立的，能做到独立分页查询，那它**必然是通过某个拦截点进行了拦截，这样它才能够进行解耦分离出分页**。
2. 第二，我们通过`PageHelper.startPage(pageNum, pageSize, orderBy)`方法后的第一个select是具备分页能力的，那它**必然缓存了分页信息，同时结合线程知识，这里必然使用的是本地栈ThreadLocal**，即每个线程有一个本地缓存。

### SpringBoot集成MySQL - MyBatis 多个数据源

#### 什么场景会出现多个数据源？

> 一般而言有如下几种出现多数据源的场景。

- **场景一：不同的业务涉及的表位于不同的数据库**

随着业务的拓展，模块解耦，服务化的拆分等，不同的业务涉及的表会放在不同的数据库中。

- **场景二：主库和从库分离（读写分离）**

主从分离等相关知识请参考这篇文章： [MySQL - 主从复制与读写分离]()

- **场景三：数据库的分片**

数据库的分片相关知识和方案请参考：[SpringBoot集成MySQL - 分库分表ShardingJDBC]()

- **场景四：多租户隔离**

所有数据库表结构一致，只是不同客户的数据放在不同数据库中，通过数据库名对不同客户的数据隔离。这种场景有一个典型的叫法：多租户。

PS：除了这种多租户除了用不同的数据库隔离不同客户数据外，还会通过额外的表字段隔离（比如tenant_id字段，不同的tenant_id表示不同的客户），对应的实现方式和案例可以参考[SpringBoot集成MySQL - MyBatis-Plus基于字段隔离的多租户]()

#### 常见的多数据源的实现思路？

- **针对场景一：不同的业务涉及的表位于不同的数据库**

首先，出现这种场景且在一个模块中设计多数据源时，需要考虑当前架构的合理性，因为从设计的角度而言不同的业务拆分需要对应着不同的服务/模块。

其次，我们会**考虑不同的package去隔离，不同的数据源放在不同的包下的代码中**。

- **针对场景二：主库和从库分离（读写分离）**

这种场景下我们叫动态数据源，通常方式使用**AOP方式拦截+ThreadLocal**切换。（本文的示例主要针对这种场景）

#### 简单示例

https://blog.csdn.net/tuesdayma/article/details/81081666

### SpringBoot集成MySQL - MyBatis-Plus方式

MyBatis-Plus（简称 MP）是一个 MyBatis的增强工具，在 MyBatis 的基础上**只做增强不做改变**，为简化开发、提高效率而生。

#### 为什么会诞生MyBatis-Plus？

> 正如前文所述（[SpringBoot集成MySQL - MyBatis XML方式]()），为了更高的效率，出现了MyBatis-Plus这类工具，对MyBatis进行增强。

1. **考虑到MyBatis是半自动化ORM**，**MyBatis-Plus 启动即会自动注入基本 CURD**，性能基本无损耗，直接面向对象操作; 并且内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求；**总体上让其支持全自动化的使用方式（本质上借鉴了Hibernate思路）。**
2. **考虑到Java8 Lambda（函数式编程）开始流行**，MyBatis-Plus支持 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
3. **考虑到MyBatis还需要独立引入PageHelper分页插件**，MyBatis-Plus**支持了内置分页插件**，同PageHelper一样基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
4. **考虑到自动化代码生成方式**，MyBatis-Plus也支持了内置代码生成器，采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
5. **考虑到SQL性能优化等问题**，MyBatis-Plus内置性能分析插件, 可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
6. 其它还有解决一些常见开发问题，比如**支持主键自动生成**，支持4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题；以及**内置全局拦截插件**，提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

#### 比较好的实践

> 总结下开发的过程中比较好的实践

1. Mapper层：继承`BaseMapper`

```java
public interface IRoleDao extends BaseMapper<Role> {
}
```

1. Service层：继承`ServiceImpl`并实现对应接口

```java
public class RoleDoServiceImpl extends ServiceImpl<IRoleDao, Role> implements IRoleService {

}
```

1. Lambda函数式查询

```java
@Override
public List<Role> findList(RoleQueryBean roleQueryBean) {
    return lambdaQuery().like(StringUtils.isNotEmpty(roleQueryBean.getName()), Role::getName, roleQueryBean.getName())
            .like(StringUtils.isNotEmpty(roleQueryBean.getDescription()), Role::getDescription, roleQueryBean.getDescription())
            .like(StringUtils.isNotEmpty(roleQueryBean.getRoleKey()), Role::getRoleKey, roleQueryBean.getRoleKey())
            .list();
}
```

1. 分页采用内置MybatisPlusInterceptor

```java
/**
  * inject pagination interceptor.
  *
  * @return pagination
  */
@Bean
public PaginationInnerInterceptor paginationInnerInterceptor() {
    return new PaginationInnerInterceptor();
}

/**
  * add pagination interceptor.
  *
  * @return MybatisPlusInterceptor
  */
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
    MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
    mybatisPlusInterceptor.addInnerInterceptor(paginationInnerInterceptor());
    return mybatisPlusInterceptor;
}
```

1. 对于复杂的关联查询

可以配置原生xml方式, 在其中自定义ResultMap

#### 除了分页插件之外还提供了哪些插件？

插件都是基于拦截器实现的，MyBatis-Plus提供了如下[插件在新窗口打开](https://baomidou.com/pages/2976a3/#mybatisplusinterceptor)：

- 自动分页: PaginationInnerInterceptor
- 多租户: TenantLineInnerInterceptor
- 动态表名: DynamicTableNameInnerInterceptor
- 乐观锁: OptimisticLockerInnerInterceptor
- sql 性能规范: IllegalSQLInnerInterceptor
- 防止全表更新与删除: BlockAttackInnerInterceptor



### SpringBoot集成MySQL - MyBatis-Plus代码自动生成

#### 模版引擎

由于CRUD的工作占了普通开发很多工作，而这些工作是重复的，所以出现了此类的代码生成工具。这些工具通过模板引擎来生成代码，常见于三方集成工具，IDE插件等等。

**[#](#什么是模板引擎) 什么是模板引擎？**

模板引擎可以在代码生成过程中减少大量机械重复工作，大大提高开发效率，良好的设计使得代码重用，后期维护都降低成本。一个好的模板引擎的使用要考虑的方面无外乎：功能是否强大，使用是否简单，整合性、扩展性与灵活性，性能。

比如：

- Velocity
- FreeMarker
- Thymeleaf
- ...

![img](https://www.pdai.tech/images/spring/springboot/springboot-engine-1.png)

#### 代码生成配置

> mybatis-plus-generator 3.5.1 及其以上版本，对历史版本不兼容！3.5.1 以下的请参考[这里在新窗口打开](https://baomidou.com/pages/d357af/)

```java
import com.baomidou.mybatisplus.generator.FastAutoGenerator;

/**
 * This class is for xxxx.
 *
 * @author pdai
 */
public class TestGenCode {

    public static void main(String[] args) {
        FastAutoGenerator.create("jdbc:mysql://localhost:3306/test_db?useSSL=false&autoReconnect=true&characterEncoding=utf8", "test", "bfXa4Pt2lUUScy8jakXf")
                .globalConfig(builder ->
                        builder.author("pdai") // 设置作者
                                .enableSwagger() // 开启 swagger 模式
                )
                .packageConfig(builder ->
                        builder.parent("tech.pdai.springboot.mysql8.mybatisplus.anno") // 设置父包名
                                .moduleName("gencode") // 设置父包模块名
                )
                .strategyConfig(builder ->
                        builder.addInclude("tb_user", "tb_role", "tb_user_role")
                )
                .execute();
    }
}
```

配置的装载, FastAutoGenerator本质上就是通过builder注入各种配置，并将它交给代码生成主类：AutoGenerator

```java
public void execute() {
    new AutoGenerator(this.dataSourceConfigBuilder.build())
        // 全局配置
        .global(this.globalConfigBuilder.build())
        // 包配置
        .packageInfo(this.packageConfigBuilder.build())
        // 策略配置
        .strategy(this.strategyConfigBuilder.build())
        // 注入配置
        .injection(this.injectionConfigBuilder.build())
        // 模板配置
        .template(this.templateConfigBuilder.build())
        // 执行
        .execute(this.templateEngine);
}
```

### SpringBoot集成MySQL - MyBatis-Plus基于字段隔离的多租户

#### 什么是多租户

多租户技术（英语：multi-tenancy technology）或称多重租赁技术，是一种软件架构技术，它是在探讨与实现如何于多用户的环境下**共用相同的系统或程序组件**，并且仍可确保**各用户间数据的隔离性**。

多租户简单来说是指一个单独的实例可以为多个组织服务。多租户技术为共用的数据中心内如何以单一系统架构与服务提供多数客户端相同甚至可定制化的服务，并且仍然可以保障客户的数据隔离。一个支持多租户技术的系统需要在设计上对它的数据和配置进行虚拟分区，从而使系统的每个租户或称组织都能够使用一个单独的系统实例，并且每个租户都可以根据自己的需求对租用的系统实例进行个性化配置。

#### 多租户在数据存储上有哪些实现方式？

##### DB隔离：独立数据库

这是第一种方案，即**一个租户一个数据库**，这种方案的用户数据隔离级别最高，安全性最好，但成本也高。

- **优点**：

1. 为不同的租户提供独立的数据库，有助于简化数据模型的扩展设计，满足不同租户的独特需求；
2. 如果出现故障，恢复数据比较简单。

- **缺点**：

1. 增大了数据库的安装数量，随之带来维护成本和购置成本的增加。
2. 这种方案与传统的一个客户、一套数据、一套部署类似，差别只在于软件统一部署在运营商那里。如果面对的是银行、医院等需要非常高数据隔离级别的租户，可以选择这种模式，提高租用的定价。如果定价较低，产品走低价路线，这种方案一般对运营商来说是无法承受的。

##### Schema隔离：共享数据库，隔离数据架构

这是第二种方案，即多个或所有租户共享Database，但**一个租户（Tenant）一个Schema**。

- **优点**：

1. 为安全性要求较高的租户提供了一定程度的逻辑数据隔离，并不是完全隔离；每个数据库可以支持更多的租户数量。

- **缺点**：

1. 如果出现故障，数据恢复比较困难，因为恢复数据库将牵扯到其他租户的数据；
2. 如果需要跨租户统计数据，存在一定困难。

##### 字段隔离：共享数据库，共享数据架构

这是第三种方案，即租户共享同一个Database、同一个Schema，但在表中**通过TenantID区分租户的数据**。这是共享程度最高、隔离级别最低的模式。

- **优点**：

1. 三种方案比较，第三种方案的维护和购置成本最低，允许每个数据库支持的租户数量最多。

- **缺点**：

1. 隔离级别最低，安全性最低，需要在设计开发时加大对安全的开发量；
2. 数据备份和恢复最困难，需要逐表逐条备份和还原。
3. 如果希望以最少的服务器为最多的租户提供服务，并且租户接受以牺牲隔离级别换取降低成本，这种方案最适合。

#### 插件的顺序

> MyBatis-Plus使用多个功能插件需要注意顺序关系

MyBatis-Plus基于字段的多租户是通过插件机制拦截实现的，因为还有很多其它的拦截器，比如:

- 自动分页: PaginationInnerInterceptor
- 多租户: TenantLineInnerInterceptor
- - 动态表名: DynamicTableNameInnerInterceptor
- 乐观锁: OptimisticLockerInnerInterceptor
- sql 性能规范: IllegalSQLInnerInterceptor
- 防止全表更新与删除: BlockAttackInnerInterceptor

所以需要注意顺序: 使用多个功能需要注意顺序关系,建议使用如下顺序

- 多租户,动态表名
- 分页,乐观锁
- sql 性能规范,防止全表更新与删除

总结: 对 sql 进行单次改造的优先放入,不对 sql 进行改造的最后放入

### SpringBoot集成ShardingJDBC - Sharding-JDBC简介和基于MyBatis的单库分表

![image-20240328154440636](C:\Users\1\Desktop\JAVA全栈知识体系\JAVA 全栈知识体系.assets\image-20240328154440636.png)

### SpringBoot集成ShardingJDBC - 基于JPA的单库分表

MyBatis 是一个优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 消除了几乎所有的 JDBC 代码和参数的手工设置以及结果集的检索。

JPA（Java Persistence API）是Java EE的标准ORM框架，它简化了对象持久化的开发，提供了更多的灵活性和更少的样板代码。

单库分表通常是为了解决数据量大的表的性能问题。分表通常有两种方式：垂直分表（即将表中的列分散到不同的表中）和水平分表（即将表中的数据按照某种规则分布到不同的表中）。

MyBatis 和 JPA 在单库分表上有所不同，主要体现在以下几个方面：

1. MyBatis 需要手动编写 SQL 并处理分表逻辑。
2. JPA 可以通过继承`javax.persistence.Entity`类和使用`@Table`注解来实现单表的分表功能，但是它没有内置的分表策略，可能需要第三方框架如ShardingSphere来实现水平分表。

以下是一个简单的例子，演示如何使用MyBatis实现基于分表键的水平分表：

```
// MyBatis Mapper 接口
public interface OrderMapper {
    void insertOrder(@Param("tableName") String tableName, @Param("order") Order order);
}
 
// MyBatis Mapper XML
<insert id="insertOrder">
    INSERT INTO ${tableName} (order_id, customer_id) VALUES (#{order.id}, #{order.customerId})
</insert>
 
// Service 方法，用于根据分表键动态选择表
public void insertOrder(Order order) {
    String tableName = "order_" + order.getId() % 10; // 假设分表键是订单ID的最后一位
    orderMapper.insertOrder(tableName, order);
}
```

在使用JPA时，你可以使用ShardingSphere来实现水平分表，如下：

```
// Entity 类
@Entity
@Table(name = "t_order")
public class Order {
    @Id
    private Long orderId;
    private Long customerId;
    // ... 其他字段和getter/setter
}
 
// ShardingSphere配置
shardingRule.tableRules().add(TableRule.builder("t_order").actualTables(Arrays.asList("t_order_0", "t_order_1")).dataSourceRule(dataSourceRule).build());
```

在这个配置中，`t_order`是逻辑表，`t_order_0`和`t_order_1`是真实的物理表。ShardingSphere会根据`orderId`的值来决定将数据插入到哪个物理表中。

总结：MyBatis 需要手动处理分表逻辑，而JPA可以通过第三方框架如ShardingSphere来简化分表操作。选择哪种方案取决于你的具体需求和偏好。

### SpringBoot集成ShardingJDBC - 基于JPA的读写分离

#### 读写分离库的场景和设计目标？

透明化读写分离所带来的影响，**让使用方尽量像使用一个数据库一样使用主从数据库集群**，是ShardingSphere读写分离模块的主要设计目标。

面对日益增加的系统访问量，数据库的吞吐量面临着巨大瓶颈。 对于同一时刻有大量并发读操作和较少写操作类型的应用系统来说，将数据库拆分为主库和从库，**主库负责处理事务性的增删改操作，从库负责处理查询操作**，能够有效的避免由数据更新导致的行锁，使得整个系统的查询性能得到极大的改善。

![img](https://www.pdai.tech/images/spring/springboot/spring-sharding-11.png)

读写分离的数据节点中的数据内容是一致的，而水平分片的每个数据节点的数据内容却并不相同。将水平分片和读写分离联合使用，能够更加有效的提升系统性能。

读写分离虽然可以提升系统的吞吐量和可用性，但同时也带来了数据不一致的问题。 

#### 核心功能

- 提供一主多从的读写分离配置，可独立使用，也可配合分库分表使用。
- 独立使用读写分离支持SQL透传。
- 同一线程且同一数据库连接内，如有写入操作，以后的读操作均从主库读取，用于保证数据一致性。
- 基于Hint的强制主库路由。

#### shardingJDBC的主从分离解决不了什么问题？

- 主库和从库的数据同步。
- 主库和从库的数据同步延迟导致的数据不一致。
- 主库双写或多写。

### SpringBoot集成ShardingJDBC - 基于JPA的DB隔离多租户方案

- **逻辑表**

水平拆分的数据库（表）的相同逻辑和数据结构表的总称。例：订单数据根据主键尾数拆分为10张表，分别是t_order_0到t_order_9，他们的逻辑表名为t_order。

- **真实表**

在分片的数据库中真实存在的物理表。即上个示例中的t_order_0到t_order_9。

- **数据节点**

数据分片的最小单元。由数据源名称和数据表组成，例：ds_0.t_order_0。

- **绑定表**

指分片规则一致的主表和子表。例如：t_order表和t_order_item表，均按照order_id分片，则此两张表互为绑定表关系。绑定表之间的多表关联查询不会出现笛卡尔积关联，关联查询效率将大大提升。举例说明，如果SQL为：

```sql
SELECT i.* FROM t_order o JOIN t_order_item i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);
```

- **广播表**

指所有的分片数据源中都存在的表，表结构和表中的数据在每个数据库中均完全一致。适用于数据量不大且需要与海量数据的表进行关联查询的场景，例如：字典表。

#### **分片键**

用于分片的数据库字段，是将数据库(表)水平拆分的关键字段。例：将订单表中的订单主键的尾数取模分片，则订单主键为分片字段。 SQL中如果无分片字段，将执行全路由，性能较差。 除了对单分片字段的支持，ShardingSphere也支持根据多个字段进行分片。

#### **[#](#分片算法) 分片算法**

> 通过分片算法将数据分片，支持通过=、>=、<=、>、<、BETWEEN和IN分片。分片算法需要应用方开发者自行实现，可实现的灵活度非常高。

目前提供4种分片算法。由于分片算法和业务实现紧密相关，因此并未提供内置分片算法，而是通过分片策略将各种场景提炼出来，提供更高层级的抽象，并提供接口让应用开发者自行实现分片算法。

- **精确分片算法**

对应PreciseShardingAlgorithm，用于处理使用单一键作为分片键的=与IN进行分片的场景。需要配合StandardShardingStrategy使用。

- **范围分片算法**

对应RangeShardingAlgorithm，用于处理使用单一键作为分片键的`BETWEEN AND、>、<、>=、<=`进行分片的场景。需要配合StandardShardingStrategy使用。

- **复合分片算法**

对应ComplexKeysShardingAlgorithm，用于处理使用多键作为分片键进行分片的场景，包含多个分片键的逻辑较复杂，需要应用开发者自行处理其中的复杂度。需要配合ComplexShardingStrategy使用。

- **Hint分片算法**

对应HintShardingAlgorithm，用于处理使用Hint行分片的场景。需要配合HintShardingStrategy使用。

#### 分片策略

> 包含分片键和分片算法，由于分片算法的独立性，将其独立抽离。真正可用于分片操作的是**分片键 + 分片算法**，也就是分片策略。目前提供5种分片策略。

- **标准分片策略**

对应StandardShardingStrategy。提供对SQL语句中的`=, >, <, >=, <=, IN`和BETWEEN AND的分片操作支持。StandardShardingStrategy只支持单分片键，提供PreciseShardingAlgorithm和RangeShardingAlgorithm两个分片算法。PreciseShardingAlgorithm是必选的，用于处理=和IN的分片。RangeShardingAlgorithm是可选的，用于处理`BETWEEN AND, >, <, >=, <=`分片，如果不配置RangeShardingAlgorithm，SQL中的BETWEEN AND将按照全库路由处理。

- **复合分片策略**

对应ComplexShardingStrategy。复合分片策略。提供对SQL语句中的`=, >, <, >=, <=, IN`和BETWEEN AND的分片操作支持。ComplexShardingStrategy支持多分片键，由于多分片键之间的关系复杂，因此并未进行过多的封装，而是直接将分片键值组合以及分片操作符透传至分片算法，完全由应用开发者实现，提供最大的灵活度。

- **行表达式分片策略**

对应InlineShardingStrategy。使用Groovy的表达式，提供对SQL语句中的=和IN的分片操作支持，只支持单分片键。对于简单的分片算法，可以通过简单的配置使用，从而避免繁琐的Java代码开发，如: `t_user_$->{u_id % 8}` 表示t_user表根据u_id模8，而分成8张表，表名称为t_user_0到t_user_7。

- **Hint分片策略**

对应HintShardingStrategy。通过Hint指定分片值而非从SQL中提取分片值的方式进行分片的策略。

- **不分片策略**

对应NoneShardingStrategy。不分片的策略。

![img](https://www.pdai.tech/images/spring/springboot/spring-sharding-22.png)

### SpringBoot集成连接池 - 数据库连接池和默认连接池HikariCP

#### 什么是数据库连接池？

数据库连接池负责**分配、管理和释放**数据库连接，它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个；释放空闲时间超过最大空闲时间的数据库连接来避免因为没有释放数据库连接而引起的数据库连接遗漏。这项技术能明显提高对数据库操作的性能。

#### 数据库连接池基本原理？

连接池基本的思想是**在系统初始化的时候，将数据库连接作为对象存储在内存中，当用户需要访问数据库时，并非建立一个新的连接，而是从连接池中取出一个已建立的空闲连接对象**。

使用完毕后，用户也并非将连接关闭，而是将连接放回连接池中，以供下一个请求访问使用。

而连接的建立、断开都由连接池自身来管理。

同时，还可以通过设置连接池的参数来控制连接池中的初始连接数、连接的上下限数以及每个连接的最大使用次数、最大空闲时间等等。也可以通过其自身的管理机制来监视数据库连接的数量、使用情况等。

如下是常用的HikariCP的使用配置

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test_db?useSSL=false&autoReconnect=true&characterEncoding=utf8
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: bfXa4Pt2lUUScy8jakXf
    # 指定为HikariDataSource
    type: com.zaxxer.hikari.HikariDataSource
    # hikari连接池配置
    hikari:
      #连接池名
      pool-name: HikariCP
      #最小空闲连接数
      minimum-idle: 5
      # 空闲连接存活最大时间，默认10分钟
      idle-timeout: 600000
      # 连接池最大连接数，默认是10
      maximum-pool-size: 10
      # 此属性控制从池返回的连接的默认自动提交行为,默认值：true
      auto-commit: true
      # 此属性控制池中连接的最长生命周期，值0表示无限生命周期，默认30分钟
      max-lifetime: 1800000
      # 数据库连接超时时间,默认30秒
      connection-timeout: 30000
      # 连接测试query
      connection-test-query: SELECT 1
```

**Hikari是默认的数据源**

![img](https://www.pdai.tech/images/spring/springboot/springboot-hikari-3.png)

#### 为什么HikariCP会成为默认连接池？

- **字节码精简** ：优化代码，直到编译后的字节码最少，这样，CPU缓存可以加载更多的程序代码；
- **优化代理和拦截器**：减少代码，例如HikariCP的Statement proxy只有100行代码，只有BoneCP的十分之一；
- **自定义数组类型（FastStatementList）代替ArrayList**：避免每次get()调用都要进行range check，避免调用remove()时的从头到尾的扫描；
- **自定义集合类型（ConcurrentBag)**：提高并发读写的效率；
- **其它**：针对BoneCP缺陷的优化，比如对于耗时超过一个CPU时间片的方法调用的研究等。

### SpringBoot集成连接池 - 集成数据库Druid连接池

Druid连接池是阿里巴巴开源的数据库连接池项目。Druid连接池**为监控而生**，内置强大的监控功能，监控特性不影响性能。功能强大，能防SQL注入，内置Loging能诊断Hack应用行为。

### SpringBoot数据库管理 - 用Liquibase对数据库管理和迁移

Liquibase是一个用于**用于跟踪、管理和应用数据库变化的开源工具**，通过日志文件(changelog)的形式记录数据库的变更(changeset)，然后执行日志文件中的修改，将数据库更新或回滚(rollback)到一致的状态。它的目标是提供一种数据库类型无关的解决方案，通过执行schema类型的文件来达到迁移。

**为何会出现Liquibase这类工具呢**？

在实际上线的应用中，随着版本的迭代，经常会遇到**需要变更数据库表和字段**，必然会遇到需要对这些**变更进行记录和管理，以及回滚**等等；同时只有脚本化且版本可管理，才能在让数据库实现真正的DevOps（自动化执行 + 回滚等）。在这样的场景下Liquibase等工具的出现也就成为了必然。

#### Liquibase有哪些概念？是如何工作的？

> **工作流程**：将**SQL**变更记录到**changeset**，多个changeset变更组成了日志文件(**changelog**)，liquibase将changelog更新日志文件同步到指定的**RDBMS**中。

![img](https://www.pdai.tech/images/spring/springboot/springboot-liquibase-1.png)

日志文件(databaseChangeLog)支持多种格式，如XML, YAML, JSON, SQL; 我们以xml为例，看下相关配置

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<databaseChangeLog
	xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext"
	xmlns:pro="http://www.liquibase.org/xml/ns/pro"
	xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
		http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.9.0.xsd
		http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd
		http://www.liquibase.org/xml/ns/pro http://www.liquibase.org/xml/ns/pro/liquibase-pro-4.9.0.xsd">
    <changeSet id="1" author="bob">  
        <comment>A sample change log</comment>  
        <createTable/> 
    </changeSet>  
    <changeSet id="2" author="bob" runAlways="true">  
        <alterTable/>  
    </changeSet>  
    <changeSet id="3" author="alice" failOnError="false" dbms="oracle">
        <alterTable/>  
    </changeSet>  
    <changeSet id="4" author="alice" failOnError="false" dbms="!oracle">
        <alterTable/>  
    </changeSet>  
</databaseChangeLog>
```

### SpringBoot数据库管理 - 用flyway对数据库管理和迁移

#### 什么是Flyway? 要解决什么问题？

> Flyway是一款数据库迁移（migration）工具。简单点说，就是在你部署应用的时候，帮你执行数据库脚本的工具。Flyway支持SQL和Java两种类型的脚本，你可以将脚本打包到应用程序中，在应用程序启动时，由Flyway来管理这些脚本的执行，这些脚本被Flyway称之为migration。

*PS: 本质上和liquibase机制一致*。

按照verion的顺序（和数据库中的更新记录对比，找到未更新的），更新如下

![img](https://www.pdai.tech/images/spring/springboot/springboot-flyway-11.png)

#### Flyway中的变迁(migrations)

> 对于Flyway，对数据库的所有更改都称为变迁(migrations)，等同于liquibase中的changeset。

在Flyway中变迁(migrations)定义的更细，包含如下三种：

1. **版本变迁(Versioned Migrations)**: 每个版本执行一次，包含有版本、描述和校验和；常用于创建，修改，删除表；插入，修改数据等
2. **撤销变迁(Undo Migrations)**: 版本变迁(Versioned Migrations)的反操作。
3. **可重复变迁(Repeatable Migrations)**: 可以执行多次，包含描述和校验和（没有版本）；主要用于视图，存储过程，函数等

#### Flyway中常用命令

**Migrate**: 是Flyway工作流的核心。它将扫描文件系统或类路径以查找可用的Migrate。它将把它们与已应用于数据库的Migrate进行比较。如果发现任何差异则迁移数据。

**Clean**: 清除掉对应数据库Schema中所有的对象，包括表结构，视图，存储过程等，clean操作在dev 和 test阶段很好用；(PS：不能用在product环境)

**Info**: 用于打印所有的Migrations的详细和状态信息，也是通过MetaData和Migrations完成的，可以快速定位当前的数据库版本；

**Validate**: 验证以及apply的Migrations是否有变更，默认开启的；原理是对比MetaData表与本地Migrations的checkNum值，如果值相同则验证通过，否则失败。

**Undo**: Migrate的反操作, 即回滚操作，这是收费功能

**BaseLine**：对已经存在数据库Schema结构的数据库一种解决方案。实现在非空数据库新建MetaData表，并把Migrations应用到该数据库；也可以应用到已有表结构的数据库中也可以实现添加Metadata表。

**Repair**：repair操作能够修复metaData表，该操作在metadata出现错误时很有用

### SpringBoot集成Redis - 基于RedisTemplate+Jedis的数据操作

#### Redis基础和5种基础数据类型

首先对redis来说，所有的**key（键）都是字符串**。我们在谈基础数据结构时，讨论的是**存储值的数据类型**，主要包括常见的5种数据类型，分别是：**String、List、Set、Zset、Hash**。

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-1.jpeg)

| 结构类型         | 结构存储的值                               | 结构的读写能力                                               |
| ---------------- | ------------------------------------------ | ------------------------------------------------------------ |
| **String字符串** | 可以是字符串、整数或浮点数                 | 对整个字符串或字符串的一部分进行操作；对整数或浮点数进行自增或自减操作； |
| **List列表**     | 一个链表，链表上的每个节点都包含一个字符串 | 对链表的两端进行push和pop操作，读取单个或多个元素；根据值查找或删除元素； |
| **Set集合**      | 包含字符串的无序集合                       | 字符串的集合，包含基础的方法有看是否存在添加、获取、删除；还包含计算交集、并集、差集等 |
| **Hash散列**     | 包含键值对的无序散列表                     | 包含方法有添加、获取、删除单个元素                           |
| **Zset有序集合** | 和散列一样，用于存储键值对                 | 字符串成员与浮点数分数之间的有序映射；元素的排列顺序由分数的大小决定；包含方法有添加、获取、删除单个元素以及根据分值范围或成员来获取元素 |

#### 什么是Jedis

**Jedis是Redis的Java客户端**，在SpringBoot 1.x版本中也是默认的客户端。在SpringBoot 2.x版本中**默认客户端是Luttuce**。

#### Spring中的Template和RedisTemplate

Spring 通过模板方式（RedisTemplate）提供了对Redis的数据**查询和操作**功能。

**RedisTemplate对于Redis5种基础类型的操作**？

```java
redisTemplate.opsForValue(); // 操作字符串
redisTemplate.opsForHash(); // 操作hash
redisTemplate.opsForList(); // 操作list
redisTemplate.opsForSet(); // 操作set
redisTemplate.opsForZSet(); // 操作zset
```

**对HyperLogLogs（基数统计）类型的操作**?

相关内容请[参考: Redis特殊数据类型 HyperLogLogs]()

```java
redisTemplate.opsForHyperLogLog();
```

**对geospatial (地理位置)类型的操作**?

相关内容请[参考: Redis特殊数据类型 geospatial]()

```java
redisTemplate.opsForGeo();
```

**对于BitMap的操作**？也是在opsForValue()方法返回类型ValueOperations中

相关内容请[参考: Redis特殊数据类型 BitMap]()

```java
Boolean setBit(K key, long offset, boolean value);
Boolean getBit(K key, long offset);
```

**对于Stream的操作**？

相关内容请[参考: Redis Stream 详解]()

```java
redisTemplate.opsForStream();
```

### SpringBoot集成Redis - 基于RedisTemplate+Lettuce数据操作

在SpringBoot 2.x版本中Redis默认客户端是Lettuce

什么是Lettuce?

> Lettuce 是一个**可伸缩线程安全的 Redis 客户端**。多个线程可以共享同一个 RedisConnection。它利用优秀 netty NIO 框架来高效地管理多个连接。[Github官网在新窗口打开](https://github.com/lettuce-io/lettuce-core)

Lettuce的特性：

- 支持 同步、异步、响应式 的方式
- 支持 Redis Sentinel
- 支持 Redis Cluster
- 支持 SSL 和 Unix Domain Socket 连接
- 支持 Streaming API
- 支持 CDI 和 Spring 的集成
- 支持 Command Interfaces
- 兼容 Java 8+ 以上版本

#### 为何SpringBoot2.x中Lettuce会成为默认的客户端？

> 除了上述特性的支持性之外，最为重要的是Lettuce中使用了**Netty框架**，使其具备线程**共享和异步**的支持性。

- **线程共享**

Jedis 是直连模式，在多个线程间共享一个 Jedis 实例时是线程不安全的，如果想要在多线程环境下使用 Jedis，需要使用连接池，每个线程都去拿自己的 Jedis 实例，当连接数量增多时，物理连接成本就较高了。

Lettuce 是基于 netty 的，连接实例可以在多个线程间共享，所以，一个多线程的应用可以使用一个连接实例，而不用担心并发线程的数量。

- **异步和反应式**

Lettuce 从一开始就按照非阻塞式 IO 进行设计，是一个纯异步客户端，对异步和反应式 API 的支持都很全面。

即使是同步命令，底层的通信过程仍然是异步模型，只是通过阻塞调用线程来模拟出同步效果而已。

#### Lettuce的基本的API方式

依赖POM包

```xml
<dependency>
  <groupId>io.lettuce</groupId>
  <artifactId>lettuce-core</artifactId>
  <version>x.y.z.BUILD-SNAPSHOT</version>
</dependency>
```

- 基础用法

```java
RedisClient client = RedisClient.create("redis://localhost");
StatefulRedisConnection<String, String> connection = client.connect();
RedisStringCommands sync = connection.sync();
String value = sync.get("key");
```

- 异步方式

```java
StatefulRedisConnection<String, String> connection = client.connect();
RedisStringAsyncCommands<String, String> async = connection.async();
RedisFuture<String> set = async.set("key", "value")
RedisFuture<String> get = async.get("key")

async.awaitAll(set, get) == true

set.get() == "OK"
get.get() == "value"
```

- 响应式

```java
StatefulRedisConnection<String, String> connection = client.connect();
RedisStringReactiveCommands<String, String> reactive = connection.reactive();
Mono<String> set = reactive.set("key", "value");
Mono<String> get = reactive.get("key");

set.subscribe();

get.block() == "value"
```

#### RedisConfig配置

通过@Bean的方式配置RedisTemplate，主要是设置RedisConnectionFactory以及各种类型数据的Serializer。

```java
package tech.pdai.springboot.redis.lettuce.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

/**
 * Redis configuration.
 *
 * @author pdai
 */
@Configuration
public class RedisConfig {

    /**
     * redis template.
     *
     * @param factory factory
     * @return RedisTemplate
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.afterPropertiesSet();
        return template;
    }
}
```

#### [#](#redistemplate的使用) RedisTemplate的使用

我们以整个系列文章一致的UserController简单示例下

```java
package tech.pdai.springboot.redis.lettuce.controller;


import io.swagger.annotations.ApiOperation;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.*;
import tech.pdai.springboot.redis.lettuce.entity.User;
import tech.pdai.springboot.redis.lettuce.entity.response.ResponseResult;

import javax.annotation.Resource;

/**
 * @author pdai
 */
@RestController
@RequestMapping("/user")
public class UserController {

    // 注意：这里@Autowired是报错的，因为@Autowired按照类名注入的
    @Resource
    private RedisTemplate<String, User> redisTemplate;

    /**
     * @param user user param
     * @return user
     */
    @ApiOperation("Add")
    @PostMapping("add")
    public ResponseResult<User> add(User user) {
        redisTemplate.opsForValue().set(String.valueOf(user.getId()), user);
        return ResponseResult.success(redisTemplate.opsForValue().get(String.valueOf(user.getId())));
    }

    /**
     * @return user list
     */
    @ApiOperation("Find")
    @GetMapping("find/{userId}")
    public ResponseResult<User> edit(@PathVariable("userId") String userId) {
        return ResponseResult.success(redisTemplate.opsForValue().get(userId));
    }

}
```

### SpringBoot集成Redis - 基于RedisTemplate+Lettuce数据类封装

#### 有了RedisTemplate为什么还要进一步封装？

RedisTemplate中的操作和方法众多，为了程序**保持方法使用的一致性，屏蔽一些无关的方法以及对使用的方法进一步封装**。



#### RedisService封装

RedisService接口类

```java
package tech.pdai.springboot.redis.lettuce.enclosure.service;

import org.springframework.data.redis.core.RedisCallback;

import java.util.Collection;
import java.util.Set;

/**
 * Redis Service.
 *
 * @author pdai
 */
public interface IRedisService<T> {

    void set(String key, T value);

    void set(String key, T value, long time);

    T get(String key);

    void delete(String key);

    void delete(Collection<String> keys);

    boolean expire(String key, long time);

    Long getExpire(String key);

    boolean hasKey(String key);

    Long increment(String key, long delta);

    Long decrement(String key, long delta);

    void addSet(String key, T value);

    Set<T> getSet(String key);

    void deleteSet(String key, T value);

    T execute(RedisCallback<T> redisCallback);
}
```

RedisService的实现类

```java
package tech.pdai.springboot.redis.lettuce.enclosure.service.impl;

import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;
import tech.pdai.springboot.redis.lettuce.enclosure.service.IRedisService;

import javax.annotation.Resource;
import java.util.Collection;
import java.util.Set;
import java.util.concurrent.TimeUnit;

/**
 * @author pdai
 */
@Service
public class RedisServiceImpl<T> implements IRedisService<T> {

    @Resource
    private RedisTemplate<String, T> redisTemplate;

    @Override
    public void set(String key, T value, long time) {
        redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
    }

    @Override
    public void set(String key, T value) {
        redisTemplate.opsForValue().set(key, value);
    }

    @Override
    public T get(String key) {
        return redisTemplate.opsForValue().get(key);
    }

    @Override
    public void delete(String key) {
        redisTemplate.delete(key);
    }

    @Override
    public void delete(Collection<String> keys) {
        redisTemplate.delete(keys);
    }

    @Override
    public boolean expire(String key, long time) {
        return redisTemplate.expire(key, time, TimeUnit.SECONDS);
    }

    @Override
    public Long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }

    @Override
    public boolean hasKey(String key) {
        return redisTemplate.hasKey(key);
    }

    @Override
    public Long increment(String key, long delta) {
        return redisTemplate.opsForValue().increment(key, delta);
    }

    @Override
    public Long decrement(String key, long delta) {
        return redisTemplate.opsForValue().increment(key, -delta);
    }

    @Override
    public void addSet(String key, T value) {
        redisTemplate.opsForSet().add(key, value);
    }

    @Override
    public Set<T> getSet(String key) {
        return redisTemplate.opsForSet().members(key);
    }

    @Override
    public void deleteSet(String key, T value) {
        redisTemplate.opsForSet().remove(key, value);
    }

    @Override
    public T execute(RedisCallback<T> redisCallback) {
        return redisTemplate.execute(redisCallback);
    }

}
```

### 分布式系统 - 分布式锁及实现方案

```
https://www.pdai.tech/md/arch/arch-z-lock.html
```

#### 什么是分布式锁

> 要介绍分布式锁，首先要提到与分布式锁相对应的是线程锁、进程锁。

- **线程锁**：主要用来给方法、代码块加锁。当某个方法或代码使用锁，在同一时刻仅有一个线程执行该方法或该代码段。线程锁只在同一JVM中有效果，因为线程锁的实现在根本上是依靠线程之间共享内存实现的，比如synchronized是共享对象头，显示锁Lock是共享某个变量（state）。
- **进程锁**：为了控制同一操作系统中多个进程访问某个共享资源，因为进程具有独立性，各个进程无法访问其他进程的资源，因此无法通过synchronized等线程锁实现进程锁。
- **分布式锁**：当多个进程**不在同一个系统**中(比如分布式系统中控制共享资源访问)，用分布式锁控制多个进程对资源的访问。

#### [#](#分布式锁的设计原则) 分布式锁的设计原则

> 分布式锁的最小设计原则：**安全性**和**有效性**

[Redis的官网在新窗口打开](https://redis.io/docs/reference/patterns/distributed-locks/)上对使用分布式锁提出至少需要满足如下三个要求：

1. **互斥**（属于安全性）：在任何给定时刻，只有一个客户端可以持有锁。
2. **无死锁**（属于有效性）：即使锁定资源的客户端崩溃或被分区，也总是可以获得锁；通常通过超时机制实现。
3. **容错性**（属于有效性）：只要大多数 Redis 节点都启动，客户端就可以获取和释放锁。

除此之外，分布式锁的设计中还可以/需要考虑：

1. 加锁解锁的**同源性**：A加的锁，不能被B解锁
2. 获取锁是**非阻塞**的：如果获取不到锁，不能无限期等待；
3. **高性能**：加锁解锁是高性能的

#### [#](#分布式锁的实现方案) 分布式锁的实现方案

- **基于数据库实现分布式锁**
  - 基于数据库表（锁表，很少使用）
  - 乐观锁(基于版本号)
  - 悲观锁(基于排它锁)
- **基于 redis 实现分布式锁**
  - 单个Redis实例：setnx(key,当前时间+过期时间) + Lua
  - Redis集群模式：Redlock
- 基于 zookeeper实现分布式锁
  - 临时有序节点来实现的分布式锁,Curator
- **基于 Consul 实现分布式锁**

#### [#](#基于数据库如何实现分布式锁-有什么缺陷) 基于数据库如何实现分布式锁？有什么缺陷？

> 基于数据库如何实现分布式锁？有什么缺陷？

##### [#](#基于数据库表-锁表-很少使用) 基于数据库表（锁表，很少使用）

最简单的方式可能就是直接创建一张锁表，然后通过操作该表中的数据来实现了。当我们想要获得锁的时候，就可以在该表中增加一条记录，想要释放锁的时候就删除这条记录。

为了更好的演示，我们先创建一张数据库表，参考如下：

```sql
CREATE TABLE database_lock (
	`id` BIGINT NOT NULL AUTO_INCREMENT,
	`resource` int NOT NULL COMMENT '锁定的资源',
	`description` varchar(1024) NOT NULL DEFAULT "" COMMENT '描述',
	PRIMARY KEY (id),
	UNIQUE KEY uiq_idx_resource (resource)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='数据库分布式锁表';
```

当我们想要获得锁时，可以插入一条数据：

```sql
INSERT INTO database_lock(resource, description) VALUES (1, 'lock');
```

当需要释放锁的时，可以删除这条数据：

```sql
DELETE FROM database_lock WHERE resource=1;
```

##### [#](#基于悲观锁) 基于悲观锁

**悲观锁实现思路**？

1. 在对任意记录进行修改前，先尝试为该记录加上排他锁（exclusive locking）。
2. 如果加锁失败，说明该记录正在被修改，那么当前查询可能要等待或者抛出异常。 具体响应方式由开发者根据实际需要决定。
3. 如果成功加锁，那么就可以对记录做修改，事务完成后就会解锁了。
4. 其间如果有其他对该记录做修改或加排他锁的操作，都会等待我们解锁或直接抛出异常。

**以MySQL InnoDB中使用悲观锁为例**？

要使用悲观锁，我们必须关闭mysql数据库的自动提交属性，因为MySQL默认使用autocommit模式，也就是说，当你执行一个更新操作后，MySQL会立刻将结果进行提交。set autocommit=0;

```sql
//0.开始事务
begin;/begin work;/start transaction; (三者选一就可以)
//1.查询出商品信息
select status from t_goods where id=1 for update;
//2.根据商品信息生成订单
insert into t_orders (id,goods_id) values (null,1);
//3.修改商品status为2
update t_goods set status=2;
//4.提交事务
commit;/commit work;
```

上面的查询语句中，我们使用了**select…for update**的方式，这样就通过开启排他锁的方式实现了悲观锁。此时在t_goods表中，id为1的 那条数据就被我们锁定了，其它的事务必须等本次事务提交之后才能执行。这样我们可以保证当前的数据不会被其它事务修改。

上面我们提到，使用`select…for update`会把数据给锁住，不过我们需要注意一些锁的级别，MySQL InnoDB默认行级锁。行级锁都是基于索引的，如果一条SQL语句用不到索引是不会使用行级锁的，会使用表级锁把整张表锁住，这点需要注意。

##### [#](#基于乐观锁) 基于乐观锁

乐观并发控制（又名“乐观锁”，Optimistic Concurrency Control，缩写“OCC”）是一种并发控制的方法。它假设多用户并发的事务在处理时不会彼此互相影响，各事务能够在不产生锁的情况下处理各自影响的那部分数据。在提交数据更新之前，每个事务会先检查在该事务读取数据后，有没有其他事务又修改了该数据。如果其他事务有更新的话，正在提交的事务会进行回滚。

**以使用版本号实现乐观锁为例？**

使用版本号时，可以在数据初始化时指定一个版本号，每次对数据的更新操作都对版本号执行+1操作。并判断当前版本号是不是该数据的最新的版本号。

```sql
1.查询出商品信息
select (status,status,version) from t_goods where id=#{id}
2.根据商品信息生成订单
3.修改商品status为2
update t_goods 
set status=2,version=version+1
where id=#{id} and version=#{version};
```

- **缺陷**

对数据库依赖，开销问题，行锁变表锁问题，无法解决数据库单点和可重入的问题。

#### 基于redis如何实现分布式锁？有什么缺陷？

> 基于redis如何实现分布式锁？这里一定要看[Redis的官网在新窗口打开](https://redis.io/docs/reference/patterns/distributed-locks/)的分布式锁的实现这篇文章。

##### [#](#set-nx-px-lua) set NX PX + Lua

**加锁**： set NX PX + 重试 + 重试间隔

向Redis发起如下命令: `SET productId:lock 0xx9p03001 NX PX 30000` 其中，"productId"由自己定义，可以是与本次业务有关的id，"0xx9p03001"是一串随机值，必须保证全局唯一(原因在后文中会提到)，“NX"指的是当且仅当key(也就是案例中的"productId:lock”)在Redis中不存在时，返回执行成功，否则执行失败。"PX 30000"指的是在30秒后，key将被自动删除。执行命令后返回成功，表明服务成功的获得了锁。

```java
@Override
public boolean lock(String key, long expire, int retryTimes, long retryDuration) {
    // use JedisCommands instead of setIfAbsense
    boolean result = setRedis(key, expire);

    // retry if needed
    while ((!result) && retryTimes-- > 0) {
        try {
            log.debug("lock failed, retrying..." + retryTimes);
            Thread.sleep(retryDuration);
        } catch (Exception e) {
            return false;
        }

        // use JedisCommands instead of setIfAbsense
        result = setRedis(key, expire);
    }
    return result;
}

private boolean setRedis(String key, long expire) {
    try {
        RedisCallback<String> redisCallback = connection -> {
            JedisCommands commands = (JedisCommands) connection.getNativeConnection();
            String uuid = SnowIDUtil.uniqueStr();
            lockFlag.set(uuid);
            return commands.set(key, uuid, NX, PX, expire); // 看这里
        };
        String result = redisTemplate.execute(redisCallback);
        return !StringUtil.isEmpty(result);
    } catch (Exception e) {
        log.error("set redis occurred an exception", e);
    }
    return false;
}
```

**解锁**：采用lua脚本

在删除key之前，一定要判断服务A持有的value与Redis内存储的value是否一致。如果贸然使用服务A持有的key来删除锁，则会误将服务B的锁释放掉。

```lua
if redis.call("get", KEYS[1])==ARGV[1] then
	return redis.call("del", KEYS[1])
else
	return 0
end
```

##### [#](#基于redlock实现分布式锁) 基于RedLock实现分布式锁

> 这是Redis作者推荐的分布式集群情况下的方式，请看这篇文章[Is Redlock safe?在新窗口打开](http://antirez.com/news/101)

假设有两个服务A、B都希望获得锁，有一个包含了5个redis master的Redis Cluster，执行过程大致如下:

1. 客户端获取当前时间戳，单位: 毫秒
2. 服务A轮寻每个master节点，尝试创建锁。(这里锁的过期时间比较短，一般就几十毫秒) RedLock算法会尝试在大多数节点上分别创建锁，假如节点总数为n，那么大多数节点指的是n/2+1。
3. 客户端计算成功建立完锁的时间，如果建锁时间小于超时时间，就可以判定锁创建成功。如果锁创建失败，则依次(遍历master节点)删除锁。
4. 只要有其它服务创建过分布式锁，那么当前服务就必须轮寻尝试获取锁。

##### [#](#基于redis的客户端) 基于Redis的客户端

> 这里Redis的客户端（Jedis, Redisson, Lettuce等）都是基于上述两类形式来实现分布式锁的，只是两类形式的封装以及一些优化（比如Redisson的watch dog)。

以基于Redisson实现分布式锁为例（支持了 单实例、Redis哨兵、redis cluster、redis master-slave等各种部署架构）：

**特色**？

1. redisson所有指令都通过lua脚本执行，保证了操作的原子性
2. redisson设置了watchdog看门狗，“看门狗”的逻辑保证了没有死锁发生
3. redisson支持Redlock的实现方式。

**过程**？

1. 线程去获取锁，获取成功: 执行lua脚本，保存数据到redis数据库。
2. 线程去获取锁，获取失败: 订阅了解锁消息，然后再尝试获取锁，获取成功后，执行lua脚本，保存数据到redis数据库。

**互斥**？

如果这个时候客户端B来尝试加锁，执行了同样的一段lua脚本。第一个if判断会执行“exists myLock”，发现myLock这个锁key已经存在。接着第二个if判断，判断myLock锁key的hash数据结构中，是否包含客户端B的ID，但明显没有，那么客户端B会获取到pttl myLock返回的一个数字，代表myLock这个锁key的剩余生存时间。此时客户端B会进入一个while循环，不听的尝试加锁。

**watch dog自动延时机制**？

客户端A加锁的锁key默认生存时间只有30秒，如果超过了30秒，客户端A还想一直持有这把锁，怎么办？其实只要客户端A一旦加锁成功，就会启动一个watch dog看门狗，它是一个后台线程，会每隔10秒检查一下，如果客户端A还持有锁key，那么就会不断的延长锁key的生存时间。

**可重入**？

每次lock会调用incrby，每次unlock会减一。

##### [#](#进一步理解) 进一步理解

1. 借助Redis实现分布式锁时，有一个共同的缺陷: 当获取锁被拒绝后，需要不断的循环，重新发送获取锁(创建key)的请求，直到请求成功。这就造成空转，浪费宝贵的CPU资源。
2. RedLock算法本身有争议，具体看这篇文章[How to do distributed locking在新窗口打开](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html) 以及作者的回复[Is Redlock safe?在新窗口打开](http://antirez.com/news/101)

#### [#](#基于zookeeper如何实现分布式锁) 基于zookeeper如何实现分布式锁？

说几个核心点：

- **顺序节点**

创建一个用于发号的节点“/test/lock”，然后以它为父亲节点的前缀为“/test/lock/seq-”依次发号：

![img](https://www.pdai.tech/images/zk/zk-1.png)

- **获得最小号得锁**

由于序号的递增性，可以规定排号最小的那个获得锁。所以，每个线程在尝试占用锁之前，首先判断自己是排号是不是当前最小，如果是，则获取锁。

- **节点监听机制**

每个线程抢占锁之前，先抢号创建自己的ZNode。同样，释放锁的时候，就需要删除抢号的Znode。抢号成功后，如果不是排号最小的节点，就处于等待通知的状态。等谁的通知呢？不需要其他人，只需要等前一个Znode 的通知就可以了。当前一个Znode 删除的时候，就是轮到了自己占有锁的时候。第一个通知第二个、第二个通知第三个，击鼓传花似的依次向后。



### SpringBoot集成Redis - Redis分布式锁的实现之Jedis(setNXPX+Lua)

#### 实现案例

**定义Redis的分布式锁类**

**定义AOP拦截点**：定义RedisLock注解

**定义AOP切面**：定义AOP切面类RedisLockAspect，用来拦截@RedisLock注解方法，并调用RedisDistributedLock对方法加锁处理。

**切面使用**：只需要添加@RedisLock注解即可

### SpringBoot定时任务 - Timer实现方式

#### 什么样的场景会使用定时任务？

比如每天/每周/每月生成日志汇总，定时发送推送信息，定时生成数据表格等

#### [#](#定时任务有哪些实现方式) 定时任务有哪些实现方式？

- 定时任务基础
  - Cron表达式
  - Linux定时任务工具crontb
- JDK内置
  - Timer
  - ScheduleExecutorService
- Netty
  - HashedWheelTimer
- Spring
  - Spring自带Schedule
  - Spring集成Quartz
- 分布式集群
  - Quartz持久化JDBC方式
  - Elastic-job
  - xxl-job

#### schedule 和 scheduleAtFixedRate 有何区别？

- **schedule**：每次执行完当前任务后，然后间隔一个period的时间再执行下一个任务； 当某个任务执行周期大于时间间隔时，依然按照间隔时间执行下个任务，即它**保证了任务之间执行的间隔**。
- **scheduleAtFixedRate**：每次执行时间为上一次任务开始起向后推一个period间隔，也就是说下次执行时间相对于上一次任务开始的时间点；按照上述的例子，它**保证了总时间段内的任务的执行次数**。

#### 为什么几乎很少使用Timer这种方式？

Timer底层是**使用一个单线来实现多个Timer任务处理的，所有任务都是由同一个线程来调度，所有任务都是串行执行，意味着同一时间只能有一个任务得到执行，而前一个任务的延迟或者异常会影响到之后的任务**。

如果有一个定时任务在运行时，产生未处理的异常，那么当前这个线程就会停止，那么所有的定时任务都会停止，受到影响。

PS：在这点上你可以看到，定时任务Job中**异常**和**超时**等一般都是要自行处理的，以防止对其它任务的影响。

### SpringBoot定时任务 - ScheduleExecutorService实现方式

#### 为什么用ScheduledExecutorService 代替 Timer？

ScheduledExecutorService是基于**线程池**的，可以开启多个线程进行执行多个任务，每个任务开启一个线程； 这样任务的延迟和未处理异常就不会影响其它任务的执行了。

#### schedule 和 scheduleAtFixedRate和 scheduleWithFixedDelay有何区别？

- **schedule**：延迟执行一个任务。
- **scheduleAtFixedRate**：每次执行时间为上一次任务开始起向后推一个period间隔，也就是说下次执行时间相对于上一次任务开始的时间点；按照上述的例子，它**保证了总时间段内的任务的执行次数**
- **scheduleAtFixedDelay**：每次执行完当前任务后，然后间隔一个period的时间再执行下一个任务； 当某个任务执行周期大于时间间隔时，依然按照间隔时间执行下个任务，即它**保证了任务之间执行的间隔**。

（PS：和timer对比下，timer中没有scheduleAtFixedDelay，它的schedule等同于scheduleAtFixedDelay）

### SpringBoot定时任务 - Netty HashedWheelTimer方式

Timer和ScheduledExecutorService是JDK内置的定时任务方案，而业内还有一个经典的定时任务的设计叫时间轮(Timing Wheel)。

Netty内部基于时间轮实现了一个HashedWheelTimer来优化百万量级I/O超时的检测，它是一个高性能，低消耗的数据结构，它适合用非准实时，延迟的短平快任务，例如心跳检测。

#### 什么是时间轮(Timing Wheel)

时间轮(Timing Wheel)是一种环形的数据结构，就像一个时钟可以分成很多格子（Tick)，每个格子代表时间的间隔，它指向存储的具体任务（timerTask）的一个链表。

![以上述在论文中的图片例子，这里一个轮子包含8个格子（Tick), 每个tick是一秒钟；

**任务的添加**：如果一个任务要在17秒后执行，那么它需要转2轮，最终加到Tick=1位置的链表中。

**任务的执行**：在时钟转2Round到Tick=1的位置，开始执行这个位置指向的链表中的这个任务。（# 这里表示剩余需要转几轮再执行这个任务）

#### Netty的HashedWheelTimer要解决什么问题

> HashedWheelTimer是Netty根据时间轮(Timing Wheel)开发的工具类，它要解决什么问题呢？这里面有两个要点：**延迟任务 + 低时效性**。

在Netty中的一个典型应用场景是判断某个连接是否idle，如果idle（如客户端由于网络原因导致到服务器的心跳无法送达），则服务器会主动断开连接，释放资源。判断连接是否idle是通过定时任务完成的，但是Netty可能维持数百万级别的长连接，对每个连接去定义一个定时任务是不可行的，所以如何提升I/O超时调度的效率呢？

Netty根据时间轮(Timing Wheel)开发了HashedWheelTimer工具类，用来**优化I/O超时调度**(本质上是延迟任务）；之所以采用时间轮(Timing Wheel)的结构还有一个很重要的原因是I/O超时这种类型的任务对**时效性不需要非常精准**。

#### HashedWheelTimer的使用方式

> 在了解时间轮（Timing Wheel)和Netty的HashedWheelTimer要解决的问题后，我们看下HashedWheelTimer的使用方式

通过构造函数看主要参数

```java
public HashedWheelTimer(
        ThreadFactory threadFactory,
        long tickDuration, TimeUnit unit, int ticksPerWheel, boolean leakDetection,
        long maxPendingTimeouts, Executor taskExecutor) {

}
```

具体参数说明如下：

- `threadFactory`：线程工厂，用于创建工作线程， 默认是Executors.defaultThreadFactory()
- `tickDuration`：tick的周期，即多久tick一次
- `unit`: tick周期的单位
- `ticksPerWheel`：时间轮的长度，一圈下来有多少格
- `leakDetection`：是否开启内存泄漏检测，默认是true
- `maxPendingTimeouts`：最多执行的任务数，默认是-1，即不限制。在高并发量情况下才会设置这个参数。

#### [#](#实现案例) 实现案例

> 这里展示下HashedWheelTimer的基本使用案例。

**[#](#pom依赖) Pom依赖**

引入pom的依赖

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.77.Final</version>
</dependency>
```

**[#](#_2个简单例子) 2个简单例子**

例子1：5秒后执行TimerTask

```java
@SneakyThrows
public static void simpleHashedWheelTimer() {
    log.info("init task 1...");
    
    HashedWheelTimer timer = new HashedWheelTimer(1, TimeUnit.SECONDS, 8);

    // add a new timeout
    timer.newTimeout(timeout -> {
        log.info("running task 1...");
    }, 5, TimeUnit.SECONDS);
}
```

执行结果如下：

```java
23:32:21.364 [main] INFO tech.pdai.springboot.schedule.timer.netty.HashedWheelTimerTester - init task 1...
...
23:32:27.454 [pool-1-thread-1] INFO tech.pdai.springboot.schedule.timer.netty.HashedWheelTimerTester - running task 1...
```

例子2：任务失效后cancel并让它重新在3秒后执行。

```java
@SneakyThrows
public static void reScheduleHashedWheelTimer() {
    log.info("init task 2...");

    HashedWheelTimer timer = new HashedWheelTimer(1, TimeUnit.SECONDS, 8);

    Thread.sleep(5000);

    // add a new timeout
    Timeout tm = timer.newTimeout(timeout -> {
        log.info("running task 2...");
    }, 5, TimeUnit.SECONDS);

    // cancel
    if (!tm.isExpired()) {
        log.info("cancel task 2...");
        tm.cancel();
    }

    // reschedule
    timer.newTimeout(tm.task(), 3, TimeUnit.SECONDS);
}
23:28:36.408 [main] INFO tech.pdai.springboot.schedule.timer.netty.HashedWheelTimerTester - init task 2...
23:28:41.412 [main] INFO tech.pdai.springboot.schedule.timer.netty.HashedWheelTimerTester - cancel task 2...
23:28:45.414 [pool-2-thread-1] INFO tech.pdai.springboot.schedule.timer.netty.HashedWheelTimerTester - running task 2...
```

#### [#](#hashedwheeltimer是如何实现的) HashedWheelTimer是如何实现的?

> 简单看下HashedWheelTimer是如何实现的

![img](https://www.pdai.tech/images/spring/springboot/springboot-timer-timewheel-3.png)

`Worker`：worker工作线程主要负责任务调度触发，单线程运行。

`HashedWheelBucket`： 时间轮上面的格子，内部持有HashedWheelTimeout组成的链表结构的头尾节点，多个格子组成的时间轮形成一圈又一圈的任务环

`HashedWheelTimeout`： 往时间轮里面提交的任务会被封装成HashedWheelTimeout

#### 什么是多级Timing Wheel?

多级的时间轮是比较好理解的，时钟是有小时，分钟，秒的，秒转一圈(Round)分钟就转一个格（Tick）, 分钟转一圈(Round)小时就转一格（Tick）。

![img](https://www.pdai.tech/images/spring/springboot/springboot-timer-timewheel-2.png)

### SpringBoot定时任务 - Spring Schedule实现方式

#### CRON表达式 - CRON表达式介绍和使用

计划任务，是任务在约定的时间执行已经计划好的工作，这是表面的意思。在Linux中，我们经常用到 cron 服务器来完成这项工作。cron服务器可以根据配置文件约定的时间来执行特定的任务。

Cron表达式是一个字符串，字符串以5或6个空格隔开，分为6或7个域，每一个域代表一个含义，Cron有如下两种语法格式：

- Seconds Minutes Hours DayofMonth Month DayofWeek Year
- Seconds Minutes Hours DayofMonth Month DayofWeek

##### [#](#cron表达式的结构) Cron表达式的结构

Cron表达式是一个具有时间含义的字符串，字符串以5个空格隔开，分为6个域，格式为X X X X X X。其中X是一个域的占位符。单个域有多个取值时，使用半角逗号,隔开取值。每个域可以是确定的取值，也可以是具有逻辑意义的特殊字符。

##### [#](#域取值) 域取值

下表为Cron表达式中六个域能够取的值以及支持的特殊字符。

| 域              | 是否必需 | 取值范围                                                     | 特殊字符      |
| --------------- | -------- | ------------------------------------------------------------ | ------------- |
| 秒 Seconds      | 是       | [0, 59]                                                      | * , - /       |
| 分钟 Minutes    | 是       | [0, 59]                                                      | * , - /       |
| 小时 Hours      | 是       | [0, 23]                                                      | * , - /       |
| 日期 DayofMonth | 是       | [1, 31]                                                      | * , - / ? L W |
| 月份 Month      | 是       | [1, 12]或[JAN, DEC]                                          | * , - /       |
| 星期 DayofWeek  | 是       | [1, 7]或[MON, SUN]。若使用[1, 7]表达方式，1代表星期一，7代表星期日。 | * , - / ? L # |
| 年 Year         | 否       | 1970+                                                        | - * /         |

##### [#](#特殊字符) 特殊字符

每一个域都使用数字，但还可以出现如下特殊字符，它们的含义是：

- `*`：表示匹配该域的任意值。假如在Minutes域使用*, 即表示每分钟都会触发事件。
- `?`：只能用在DayofMonth和DayofWeek两个域。它也匹配域的任意值，但实际不会。因为DayofMonth和DayofWeek会相互影响。例如想在每月的20日触发调度，不管20日到底是星期几，则只能使用如下写法： 13 13 15 20 * ?, 其中最后一位只能用？，而不能使用*，如果使用*表示不管星期几都会触发，实际上并不是这样。
- `-`：表示范围。例如在Minutes域使用5-20，表示从5分到20分钟每分钟触发一次
- `/`：表示起始时间开始触发，然后每隔固定时间触发一次。例如在Minutes域使用5/20,则意味着5分钟触发一次，而25，45等分别触发一次.
- `,`：表示列出枚举值。例如：在Minutes域使用5,20，则意味着在5和20分每分钟触发一次。
- `L`：表示最后，只能出现在DayofWeek和DayofMonth域。如果在DayofWeek域使用5L,意味着在最后的一个星期四触发。
- `W`:表示有效工作日(周一到周五),只能出现在DayofMonth域，系统将在离指定日期的最近的有效工作日触发事件。例如：在 DayofMonth使用5W，如果5日是星期六，则将在最近的工作日：星期五，即4日触发。如果5日是星期天，则在6日(周一)触发；如果5日在星期一到星期五中的一天，则就在5日触发。另外一点，W的最近寻找不会跨过月份 。
- `LW`:这两个字符可以连用，表示在某个月最后一个工作日，即最后一个星期五。
- `#`:用于确定每个月第几个星期几，只能出现在DayofMonth域。例如在4#2，表示某月的第二个星期三。

##### [#](#常用表达式例子) 常用表达式例子

- `0 0 2 1 * ? *` 表示在每月的1日的凌晨2点调整任务
- `0 15 10 ? * MON-FRI` 表示周一到周五每天上午10:15执行作业
- `0 15 10 ? 6L 2002-2006` 表示2002-2006年的每个月的最后一个星期五上午10:15执行作
- `0 0 10,14,16 * * ?` 每天上午10点，下午2点，4点
- `0 0/30 9-17 * * ?` 朝九晚五工作时间内每半小时
- `0 0 12 ? * WED` 表示每个星期三中午12点
- `0 0 12 * * ?` 每天中午12点触发
- `0 15 10 ? * *` 每天上午10:15触发
- `0 15 10 * * ?` 每天上午10:15触发
- `0 15 10 * * ? *` 每天上午10:15触发
- `0 15 10 * * ? 2005` 2005年的每天上午10:15触发
- `0 * 14 * * ?` 在每天下午2点到下午2:59期间的每1分钟触发
- `0 0/5 14 * * ?` 在每天下午2点到下午2:55期间的每5分钟触发
- `0 0/5 14,18 * * ?` 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发
- `0 0-5 14 * * ?` 在每天下午2点到下午2:05期间的每1分钟触发
- `0 10,44 14 ? 3 WED` 每年三月的星期三的下午2:10和2:44触发
- `0 15 10 ? * MON-FRI` 周一至周五的上午10:15触发
- `0 15 10 15 * ?` 每月15日上午10:15触发
- `0 15 10 L * ?` 每月最后一日的上午10:15触发
- `0 15 10 ? * 6L` 每月的最后一个星期五上午10:15触发
- `0 15 10 ? * 6L 2002-2005` 2002年至2005年的每月的最后一个星期五上午10:15触发
- `0 15 10 ? * 6#3` 每月的第三个星期五上午10:15触发

注：

- **有些子表达式能包含一些范围或列表**

例如：子表达式（天（星期））可以为 “MON-FRI”，“MON，WED，FRI”，“MON-WED,SAT”

- **`\*`字符代表所有可能的值**

因此，`*`在子表达式（月）里表示每个月的含义，`*`在子表达式（天（星期））表示星期的每一天

- **`/`字符用来指定数值的增量**

例如：在子表达式（分钟）里的“0/15”表示从第0分钟开始，每15分钟

在子表达式（分钟）里的“3/20”表示从第3分钟开始，每20分钟（它和“3，23，43”）的含义一样

- **`？`字符仅被用于天（月）和天（星期）两个子表达式，表示不指定值**

当2个子表达式其中之一被指定了值以后，为了避免冲突，需要将另一个子表达式的值设为“？”

- **`L`字符仅被用于天（月）和天（星期）两个子表达式，它是单词“last”的缩写**

但是它在两个子表达式里的含义是不同的：

1. 在天（月）子表达式中，“L”表示一个月的最后一天
2. 在天（星期）自表达式中，“L”表示一个星期的最后一天，也就是SAT

如果在“L”前有具体的内容，它就具有其他的含义了

例如：“6L”表示这个月的倒数第６天，“FRIL”表示这个月的最一个星期五

注意：在使用“L”参数时，不要指定列表或范围，因为这会导致问题

#### CRON生成和校验工具

https://www.bejson.com/othertools/cron/

![img](https://www.pdai.tech/images/develop/cron/develop-cron-tool-1.png)

http://cron.ciding.cc/

![img](https://www.pdai.tech/images/develop/cron/develop-cron-tool-3.png)

#### 实现案例

##### @EnableScheduling+@Scheduled

- **通过@EnableScheduling启用定时任务，@Scheduled定义任务**

```java
@EnableScheduling
@Configuration
public class ScheduleDemo {

    /**
     * 每隔1分钟执行一次。
     */
    @Scheduled(fixedRate = 1000 * 60 * 1)
    public void runScheduleFixedRate() {
        log.info("runScheduleFixedRate: current DateTime, {}", LocalDateTime.now());
    }

    /**
     * 每个整点小时执行一次。
     */
    @Scheduled(cron = "0 0 */1 * * ?")
    public void runScheduleCron() {
        log.info("runScheduleCron: current DateTime, {}", LocalDateTime.now());
    }

}
```

- **@Scheduled所支持的参数**：

1. `cron`：cron表达式，指定任务在特定时间执行；
2. `fixedDelay`：表示上一次任务执行完成后多久再次执行，参数类型为long，单位ms；
3. `fixedDelayString`：与fixedDelay含义一样，只是参数类型变为String；
4. `fixedRate`：表示按一定的频率执行任务，参数类型为long，单位ms；
5. `fixedRateString`: 与fixedRate的含义一样，只是将参数类型变为String；
6. `initialDelay`：表示延迟多久再第一次执行任务，参数类型为long，单位ms；
7. `initialDelayString`：与initialDelay的含义一样，只是将参数类型变为String；
8. `zone`：时区，默认为当前时区，一般没有用到。

### SpringBoot定时任务 - 基础quartz实现方式

#### 什么是Quartz

Quartz是OpenSymphony开源组织在Job scheduling领域又一个开源项目，它可以与J2EE与J2SE应用程序相结合也可以单独使用。Quartz可以用来创建简单或为运行十个，百个，甚至是好几万个Jobs这样复杂的程序。Jobs可以做成标准的Java组件或 EJBs。

**它的特点如下**

- 纯java实现，可以作为独立的应用程序，也可以嵌入在另一个独立式应用程序运行
- 强大的调度功能，Spring默认的调度框架，灵活可配置；
- 作业持久化，调度环境持久化机制，可以保存并恢复调度现场。系统关闭数据不会丢失；灵活的应用方式，可以任意定义触发器的调度时间表，支持任务和调度各种组合，组件式监听器、各种插件、线程池等功能，多种存储方式等；
- 分布式和集群能力，可以被实例化，一个Quartz集群中的每个节点作为一个独立的Quartz使用，通过相同的数据库表来感知到另一个Quartz应用

#### [#](#quartz的体系结构) Quartz的体系结构

![img](https://www.pdai.tech/images/spring/springboot/springboot-job-quartz-1.png)

**Job** 表示一个工作，要执行的具体内容。

**JobDetail** 表示一个具体的可执行的调度程序，Job 是这个可执行程调度程序所要执行的内容，另外 JobDetail 还包含了这个任务调度的方案和策略。

**Trigger** 代表一个调度参数的配置，什么时候去调。

**Scheduler** 代表一个调度容器，一个调度容器中可以注册多个 JobDetail 和 Trigger。当 Trigger 与 JobDetail 组合，就可以被 Scheduler 容器调度了。

#### 实现案例

- 引入POM依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

- 定义Job

只需要继承QuartzJobBean，并重载executeInternal方法即可定义你自己的Job执行逻辑。

```java
@Slf4j
public class HelloJob extends QuartzJobBean {

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        // get parameters
        context.getJobDetail().getJobDataMap().forEach(
                (k, v) -> log.info("param, key:{}, value:{}", k, v)
        );
        // your logics
        log.info("Hello Job执行时间: " + new Date());
    }
}
```

- 配置Job

JobDetail, Trigger, Schedule(这里采用CronScheduleBuilder)

```java
/**
 * @author pdai
 */
@Configuration
public class QuartzConfig {

    @Bean("helloJob")
    public JobDetail helloJobDetail() {
        return JobBuilder.newJob(HelloJob.class)
                .withIdentity("DateTimeJob")
                .usingJobData("msg", "Hello Quartz")
                .storeDurably()//即使没有Trigger关联时，也不需要删除该JobDetail
                .build();
    }

    @Bean
    public Trigger printTimeJobTrigger() {
        // 每秒执行一次
        CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule("0/1 * * * * ?");
        return TriggerBuilder.newTrigger()
                .forJob(helloJobDetail())
                .withIdentity("quartzTaskService")
                .withSchedule(cronScheduleBuilder)
                .build();
    }
}
```



### SpringBoot定时任务 - 分布式quartz cluster方式

通常我们使用quartz只是实现job单实例运行，本例将展示quartz实现基于数据库的分布式任务管理，和控制job生命周期。

#### 什么是Quartz持久化

- **为什么要持久化**？

当程序突然被中断时，如断电，内存超出时，很有可能造成任务的丢失。 可以将调度信息存储到数据库里面，进行持久化，当程序被中断后，再次启动，仍然会保留中断之前的数据，继续执行，而并不是重新开始。

- **Quartz提供了两种持久化方式**

Quartz提供两种基本作业存储类型:

1. **RAMJobStore**

在默认情况下Quartz将任务调度的运行信息保存在**内存**中，这种方法提供了最佳的性能，因为内存中数据访问最快。不足之处是缺乏数据的持久性，当程序路途停止或系统崩溃时，所有运行的信息都会丢失。

1. **JobStoreTX**

所有的任务信息都会保存到**数据库**中，可以控制事物，还有就是如果应用服务器关闭或者重启，任务信息都不会丢失，并且可以恢复因服务器关闭或者重启而导致执行失败的任务。

### SpringBoot定时任务 - 分布式elastic-job方式

#### 什么是ElasticJob

> ElasticJob 是面向互联网生态和海量任务的分布式调度解决方案，由两个相互独立的子项目 ElasticJob-Lite 和 ElasticJob-Cloud 组成。 它通过弹性调度、资源管控、以及作业治理的功能，打造一个适用于互联网场景的分布式调度解决方案，并通过开放的架构设计，提供多元化的作业生态。 它的各个产品使用统一的作业 API，开发者仅需一次开发，即可随意部署。ElasticJob 已于 2020 年 5 月 28 日成为 Apache ShardingSphere 的子项目。

使用 ElasticJob 能够让开发工程师不再担心任务的线性吞吐量提升等非功能需求，使他们能够更加专注于面向业务编码设计； 同时，它也能够解放运维工程师，使他们不必再担心任务的可用性和相关管理需求，只通过轻松的增加服务节点即可达到自动化运维的目的。

**ElasticJob-Lite**: 定位为轻量级无中心化解决方案，使用 jar 的形式提供分布式任务的协调服务。

![img](https://www.pdai.tech/images/spring/springboot/springboot-elasticjob-lite-1.png)

**lasticJob-Cloud**： 采用自研 Mesos Framework 的解决方案，额外提供资源治理、应用分发以及进程隔离等功能。

![img](https://www.pdai.tech/images/spring/springboot/springboot-elasticjob-cloud-1.png)

**ElasticJob-Lite和ElasticJob-Cloud的区别**

|          | ElasticJob-Lite | ElasticJob-Cloud  |
| -------- | --------------- | ----------------- |
| 无中心化 | 是              | 否                |
| 资源分配 | 不支持          | 支持              |
| 作业模式 | 常驻            | 常驻 + 瞬时       |
| 部署依赖 | ZooKeeper       | ZooKeeper + Mesos |

### SpringBoot定时任务 - 分布式xxl-job方式

具体见：https://www.pdai.tech/md/spring/springboot/springboot-x-task-xxl-job-timer.html

#### 什么是xxl-job

> XXL-JOB是一个分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。

支持如下特性：

- 1、简单：支持通过Web页面对任务进行CRUD操作，操作简单，一分钟上手；
- 2、动态：支持动态修改任务状态、启动/停止任务，以及终止运行中任务，即时生效；
- 3、调度中心HA（中心式）：调度采用中心式设计，“调度中心”自研调度组件并支持集群部署，可保证调度中心HA；
- 4、执行器HA（分布式）：任务分布式执行，任务"执行器"支持集群部署，可保证任务执行HA；
- 5、注册中心: 执行器会周期性自动注册任务, 调度中心将会自动发现注册的任务并触发执行。同时，也支持手动录入执行器地址；
- 6、弹性扩容缩容：一旦有新执行器机器上线或者下线，下次调度时将会重新分配任务；
- 7、触发策略：提供丰富的任务触发策略，包括：Cron触发、固定间隔触发、固定延时触发、API（事件）触发、人工触发、父子任务触发；
- 8、调度过期策略：调度中心错过调度时间的补偿处理策略，包括：忽略、立即补偿触发一次等；
- 9、阻塞处理策略：调度过于密集执行器来不及处理时的处理策略，策略包括：单机串行（默认）、丢弃后续调度、覆盖之前调度；
- 10、任务超时控制：支持自定义任务超时时间，任务运行超时将会主动中断任务；
- 11、任务失败重试：支持自定义任务失败重试次数，当任务失败时将会按照预设的失败重试次数主动进行重试；其中分片任务支持分片粒度的失败重试；
- 12、任务失败告警；默认提供邮件方式失败告警，同时预留扩展接口，可方便的扩展短信、钉钉等告警方式；
- 13、路由策略：执行器集群部署时提供丰富的路由策略，包括：第一个、最后一个、轮询、随机、一致性HASH、最不经常使用、最近最久未使用、故障转移、忙碌转移等；
- 14、分片广播任务：执行器集群部署时，任务路由策略选择"分片广播"情况下，一次任务调度将会广播触发集群中所有执行器执行一次任务，可根据分片参数开发分片任务；
- 15、动态分片：分片广播任务以执行器为维度进行分片，支持动态扩容执行器集群从而动态增加分片数量，协同进行业务处理；在进行大数据量业务操作时可显著提升任务处理能力和速度。
- 16、故障转移：任务路由策略选择"故障转移"情况下，如果执行器集群中某一台机器故障，将会自动Failover切换到一台正常的执行器发送调度请求。
- 17、任务进度监控：支持实时监控任务进度；
- 18、Rolling实时日志：支持在线查看调度结果，并且支持以Rolling方式实时查看执行器输出的完整的执行日志；
- 19、GLUE：提供Web IDE，支持在线开发任务逻辑代码，动态发布，实时编译生效，省略部署上线的过程。支持30个版本的历史版本回溯。
- 20、脚本任务：支持以GLUE模式开发和运行脚本任务，包括Shell、Python、NodeJS、PHP、PowerShell等类型脚本;
- 21、命令行任务：原生提供通用命令行任务Handler（Bean任务，"CommandJobHandler"）；业务方只需要提供命令行即可；
- 22、任务依赖：支持配置子任务依赖，当父任务执行结束且执行成功后将会主动触发一次子任务的执行, 多个子任务用逗号分隔；
- 23、一致性：“调度中心”通过DB锁保证集群分布式调度的一致性, 一次任务调度只会触发一次执行；
- 24、自定义任务参数：支持在线配置调度任务入参，即时生效；
- 25、调度线程池：调度系统多线程触发调度运行，确保调度精确执行，不被堵塞；
- 26、数据加密：调度中心和执行器之间的通讯进行数据加密，提升调度信息安全性；
- 27、邮件报警：任务失败时支持邮件报警，支持配置多邮件地址群发报警邮件；
- 28、推送maven中央仓库: 将会把最新稳定版推送到maven中央仓库, 方便用户接入和使用;
- 29、运行报表：支持实时查看运行数据，如任务数量、调度次数、执行器数量等；以及调度报表，如调度日期分布图，调度成功分布图等；
- 30、全异步：任务调度流程全异步化设计实现，如异步调度、异步运行、异步回调等，有效对密集调度进行流量削峰，理论上支持任意时长任务的运行；
- 31、跨语言：调度中心与执行器提供语言无关的 RESTful API 服务，第三方任意语言可据此对接调度中心或者实现执行器。除此之外，还提供了 “多任务模式”和“httpJobHandler”等其他跨语言方案；
- 32、国际化：调度中心支持国际化设置，提供中文、英文两种可选语言，默认为中文；
- 33、容器化：提供官方docker镜像，并实时更新推送dockerhub，进一步实现产品开箱即用；
- 34、线程池隔离：调度线程池进行隔离拆分，慢任务自动降级进入"Slow"线程池，避免耗尽调度线程，提高系统稳定性；
- 35、用户管理：支持在线管理系统用户，存在管理员、普通用户两种角色；
- 36、权限控制：执行器维度进行权限控制，管理员拥有全量权限，普通用户需要分配执行器权限后才允许相关操作；

#### xxl-job的架构设计

##### [#](#设计思想) 设计思想

将调度行为抽象形成“**调度中心**”公共平台，而平台自身并不承担业务逻辑，“调度中心”负责**发起调度请求**。

将**任务**抽象成分散的**JobHandler**，交由“**执行器**”统一管理，“执行器”负责接收调度请求并执行对应的JobHandler中业务逻辑。

因此，“调度”和“任务”两部分可以相互解耦，提高系统整体稳定性和扩展性；

##### [#](#系统组成) 系统组成

1. 调度模块

   （调度中心） 

   1. 负责管理调度信息，按照调度配置发出调度请求，自身不承担业务代码。调度系统与任务解耦，提高了系统可用性和稳定性，同时调度系统性能不再受限于任务模块；
   2. 支持可视化、简单且动态的管理调度信息，包括任务新建，更新，删除，GLUE开发和任务报警等，所有上述操作都会实时生效，同时支持监控调度结果以及执行日志，支持执行器Failover。

2. 执行模块

   （执行器）： 

   1. 负责接收调度请求并执行任务逻辑。任务模块专注于任务的执行等操作，开发和维护更加简单和高效；
   2. 接收“调度中心”的执行请求、终止请求和日志请求等。

##### 架构图

![img](https://www.pdai.tech/images/spring/springboot/springboot-xxl-job-8.png)

#### 实现案例

##### Bean模式(基于方法)

> Bean模式任务，支持基于方法的开发方式，每个任务对应一个方法。基于方法开发的任务，底层会生成JobHandler代理，和基于类的方式一样，任务也会以JobHandler的形式存在于执行器任务容器中。

**优点**：

- 每个任务只需要开发一个方法，并添加”@XxlJob”注解即可，更加方便、快速。
- 支持自动扫描任务并注入到执行器容器。

**缺点**：要求Spring容器环境；

##### Bean模式(基于类)

> Bean模式任务，支持基于类的开发方式，每个任务对应一个Java类。

**优点**：不限制项目环境，兼容性好。即使是无框架项目，如main方法直接启动的项目也可以提供支持，可以参考示例项目 “xxl-job-executor-sample-frameless”；

**缺点**：

- 每个任务需要占用一个Java类，造成类的浪费；
- 不支持自动扫描任务并注入到执行器容器，需要手动注入。

##### GLUE模式

> 任务以源码方式维护在调度中心，支持通过Web IDE在线更新，实时编译和生效，因此不需要指定JobHandler。

### SpringBoot集成文件 - 基础的文件上传和下载

#### 实现案例

##### Pom依赖

引入spring-boot-starter-web即可

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

##### [#](#文件上传) 文件上传

上传一个文件，并保存到本地文件夹中

```java
@PostMapping("/upload")
public ResponseResult<String> upload(@RequestParam(value = "file", required = true) MultipartFile file) {
    try {
        // 本地文件保存位置
        String uploadPath = "/Users/pdai/uploadFile"; // 改这里
        File uploadDir = new File(uploadPath);
        if (!uploadDir.exists()) {
            uploadDir.mkdir();
        }
        log.info(uploadDir.getAbsolutePath());

        // 本地文件
        File localFile = new File(uploadPath + File.separator + file.getOriginalFilename());

        // transfer to local
        file.transferTo(localFile);
    } catch (Exception e) {
        e.printStackTrace();
        return ResponseResult.fail(e.getMessage());
    }
    return ResponseResult.success();
}
```

##### 文件下载

从本地文件夹中读取文件，并通过http下载

```java
@GetMapping("/download")
public void download(HttpServletResponse response) {
    response.reset();
    response.setContentType("application/octet-stream");
    response.setHeader("Content-disposition",
            "attachment;filename=file_" + System.currentTimeMillis() + ".hprof");

    // 从文件读到servlet response输出流中
    File file = new File("/Users/pdai/pdai_heap_dump_test.hprof"); // 改这里
    try (FileInputStream inputStream = new FileInputStream(file);) { // try-with-resources
        byte[] b = new byte[1024];
        int len;
        while ((len = inputStream.read(b)) > 0) {
            response.getOutputStream().write(b, 0, len);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

#### SpringBoot文件上传大小参数？

SpringBoot对上传的文件大小有限制，默认的最大每个文件配置最大为1MB，默认多个文件上传（上传目标文件夹）总大小是10MB。

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 1024MB # 单个文件大小
      max-request-size: 10240MB # 总文件大小（允许存储文件的文件夹大小）
```

更多其它的参数可以参看MultipartProperties类

```java
@ConfigurationProperties(prefix = "spring.servlet.multipart", ignoreUnknownFields = false)
public class MultipartProperties {

	/**
	 * Whether to enable support of multipart uploads.
	 */
	private boolean enabled = true;

	/**
	 * Intermediate location of uploaded files.
	 */
	private String location;

	/**
	 * Max file size.
	 */
	private DataSize maxFileSize = DataSize.ofMegabytes(1);

	/**
	 * Max request size.
	 */
	private DataSize maxRequestSize = DataSize.ofMegabytes(10);

	/**
	 * Threshold after which files are written to disk.
	 */
	private DataSize fileSizeThreshold = DataSize.ofBytes(0);

	/**
	 * Whether to resolve the multipart request lazily at the time of file or parameter
	 * access.
	 */
	private boolean resolveLazily = false;

}
```

#### [#](#多个文件上传) 多个文件上传？

Spring支持接收多个文件的，只需要用MultipartFile接收即可

```java
@PostMapping("/upload")
public ResponseResult<String> upload(MultipartFile[] files) {

}
```

### SpringBoot集成文件 - 大文件的上传(异步，分片，断点续传和秒传)

#### 基于分片的断点续传和秒传

> 当我们上传的文件很大时，对大文件的处理通常通过**分片上传**、**断点续传**和**秒传**技术手段实现。

- **分片上传**

分片上传就是将一个大文件**分成若干份大小相等的小块文件**，等所有小块文件上传成功后，再将文件进行合并成完整的原始文件。

- **断点续传**

断点续传就是重新上传文件时先判断哪些文件块已经上传过了（比如将分片的chuck md5等信息保存在DB中)，如果**上传过了则跳过这些块**，否则上传没有上传的块。

- **秒传**

当用户选择上传一个文件时，服务端**检测该文件之前是否已经被上传过**，如果服务器已经存有该文件（完全一样），就立马返回前端 “文件已上传成功”。前端随即将进度条更新至100%。 这样给用户的感觉就是 “秒传” 的感觉。

对于分片上传，**前后端分别需要怎么做呢**？

1. **前端**：需要将文件file.slice()成多个文件块，并计算每一块的md5值，每次请求上传都是传一个小文件块。
2. **后端**：需要接收每次上传的文件块并保存文件块的信息（比如md5), 如果已经上传则跳过；最后等所有文件上传完成之后，将所有的文件块合并成一个大文件。

#### 异步上传

> 当我们上传的文件很大，后台需要处理的时间很长，用户期望切换到其它页面继续工作，而不需要等待完成。这时候我们可以采用异步上传的方式。

在SpringBoot中将一个方法声明为异步方法非常简单，只需两个注解即可`@EnableAsync`和`@Async`。

1. `@EnableAsync`用于**开启SpringBoot支持异步的功能**，用在SpringBoot的**启动类**上。

```java
@SpringBootApplication
@EnableAsync // 这里
public class App {
 
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

1. `@Async`用于**方法**上，标记该方法为**异步处理方法**。

```java
@Service
public class UserServiceImpl {
 
    @Async // 这里带@Async的方法，会被当成一个子线程
    public void processFile(InputStream inputStream) {
        
    }
}
```

###  SpringBoot集成文件 - 集成POI之Excel导入导出、

#### 什么是POI

> Apache POI 是用Java编写的免费开源的跨平台的 **Java API**，Apache POI提供API给Java程序对Microsoft Office格式档案读和写的功能。POI为“Poor Obfuscation Implementation”的首字母缩写，意为“简洁版的模糊实现”。

Apache POI 是创建和维护操作各种符合Office Open XML（OOXML）标准和微软的OLE 2复合文档格式（OLE2）的Java API。用它可以使用Java读取和创建,修改MS Excel文件.而且,还可以使用Java读取和创建MS Word和MSPowerPoint文件。

#### POI中基础概念

生成xls和xlsx有什么区别呢？

| XLS                                   | XLSX                           |
| ------------------------------------- | ------------------------------ |
| 只能打开xls格式，无法直接打开xlsx格式 | 可以直接打开xls、xlsx格式      |
| 只有65536行、256列                    | 可以有1048576行、16384列       |
| 占用空间大                            | 占用空间小，运算速度也会快一点 |

POI对Excel中的对象的封装对应关系如下：

| Excel            | POI XLS              | POI XLSX(Excel 2007+) |
| ---------------- | -------------------- | --------------------- |
| Excel 文件       | HSSFWorkbook （xls） | XSSFWorkbook（xlsx）  |
| Excel 工作表     | HSSFSheet            | XSSFSheet             |
| Excel 行         | HSSFRow              | XSSFRow               |
| Excel 单元格     | HSSFCell             | XSSFCell              |
| Excel 单元格样式 | HSSFCellStyle        | HSSFCellStyle         |
| Excel 颜色       | HSSFColor            | XSSFColor             |
| Excel 字体       | HSSFFont             | XSSFFont              |

------

#### 实现案例

> 这里展示SpringBoot集成POI导出用户列表的和导入用户列表的例子。

##### [#](#pom依赖) Pom依赖

引入poi的依赖包

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.2</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.2</version>
</dependency>
```

##### [#](#导出excel) 导出Excel

UserController中导出的方法

```java
@ApiOperation("Download Excel")
@GetMapping("/excel/download")
public void download(HttpServletResponse response) {
    try {
        SXSSFWorkbook workbook = userService.generateExcelWorkbook();
        response.reset();
        response.setContentType("application/vnd.ms-excel");
        response.setHeader("Content-disposition",
                "attachment;filename=user_excel_" + System.currentTimeMillis() + ".xlsx");
        OutputStream os = response.getOutputStream();
        workbook.write(os);
        workbook.dispose();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

UserServiceImple中导出Excel的主方法

```java
private static final int POSITION_ROW = 1;
private static final int POSITION_COL = 1;

/**
  * @return SXSSFWorkbook
  */
@Override
public SXSSFWorkbook generateExcelWorkbook() {
    SXSSFWorkbook workbook = new SXSSFWorkbook();
    Sheet sheet = workbook.createSheet();

    int rows = POSITION_ROW;
    int cols = POSITION_COL;

    // 表头
    Row head = sheet.createRow(rows++);
    String[] columns = new String[]{"ID", "Name", "Email", "Phone", "Description"};
    int[] colWidths = new int[]{2000, 3000, 5000, 5000, 8000};
    CellStyle headStyle = getHeadCellStyle(workbook);
    for (int i = 0; i < columns.length; ++i) {
        sheet.setColumnWidth(cols, colWidths[i]);
        addCellWithStyle(head, cols++, headStyle).setCellValue(columns[i]);
    }

    // 表内容
    CellStyle bodyStyle = getBodyCellStyle(workbook);
    for (User user : getUserList()) {
        cols = POSITION_COL;
        Row row = sheet.createRow(rows++);
        addCellWithStyle(row, cols++, bodyStyle).setCellValue(user.getId());
        addCellWithStyle(row, cols++, bodyStyle).setCellValue(user.getUserName());
        addCellWithStyle(row, cols++, bodyStyle).setCellValue(user.getEmail());
        addCellWithStyle(row, cols++, bodyStyle).setCellValue(String.valueOf(user.getPhoneNumber()));
        addCellWithStyle(row, cols++, bodyStyle).setCellValue(user.getDescription());
    }
    return workbook;
}

private Cell addCellWithStyle(Row row, int colPosition, CellStyle cellStyle) {
    Cell cell = row.createCell(colPosition);
    cell.setCellStyle(cellStyle);
    return cell;
}

private List<User> getUserList() {
    return Collections.singletonList(User.builder()
            .id(1L).userName("pdai").email("pdai@pdai.tech").phoneNumber(121231231231L)
            .description("hello world")
            .build());
}

private CellStyle getHeadCellStyle(Workbook workbook) {
    CellStyle style = getBaseCellStyle(workbook);

    // fill
    style.setFillForegroundColor(IndexedColors.GREY_25_PERCENT.getIndex());
    style.setFillPattern(FillPatternType.SOLID_FOREGROUND);

    return style;
}

private CellStyle getBodyCellStyle(Workbook workbook) {
    return getBaseCellStyle(workbook);
}

private CellStyle getBaseCellStyle(Workbook workbook) {
    CellStyle style = workbook.createCellStyle();

    // font
    Font font = workbook.createFont();
    font.setBold(true);
    style.setFont(font);

    // align
    style.setAlignment(HorizontalAlignment.CENTER);
    style.setVerticalAlignment(VerticalAlignment.TOP);

    // border
    style.setBorderBottom(BorderStyle.THIN);
    style.setBottomBorderColor(IndexedColors.BLACK.getIndex());
    style.setBorderLeft(BorderStyle.THIN);
    style.setLeftBorderColor(IndexedColors.BLACK.getIndex());
    style.setBorderRight(BorderStyle.THIN);
    style.setRightBorderColor(IndexedColors.BLACK.getIndex());
    style.setBorderTop(BorderStyle.THIN);
    style.setTopBorderColor(IndexedColors.BLACK.getIndex());

    return style;
}
```

##### [#](#导入excel) 导入Excel

> 我们将上面导出的excel文件导入。

UserController中导入的方法

```java
@ApiOperation("Upload Excel")
@PostMapping("/excel/upload")
public ResponseResult<String> upload(@RequestParam(value = "file", required = true) MultipartFile file) {
    try {
        userService.upload(file.getInputStream());
    } catch (Exception e) {
        e.printStackTrace();
        return ResponseResult.fail(e.getMessage());
    }
    return ResponseResult.success();
}
```

UserServiceImple中导入Excel的主方法

```java
@Override
public void upload(InputStream inputStream) throws IOException {
    XSSFWorkbook book = new XSSFWorkbook(inputStream);
    XSSFSheet sheet = book.getSheetAt(0);
    // add some validation here

    // parse data
    int cols;
    for (int i = POSITION_ROW; i < sheet.getLastRowNum(); i++) {
        XSSFRow row = sheet.getRow(i + 1); // 表头不算
        cols = POSITION_COL;
        User user = User.builder()
                .id(getCellLongValue(row.getCell(cols++)))
                .userName(getCellStringValue(row.getCell(cols++)))
                .email(getCellStringValue(row.getCell(cols++)))
                .phoneNumber(Long.parseLong(getCellStringValue(row.getCell(cols++))))
                .description(getCellStringValue(row.getCell(cols++)))
                .build();
        log.info(user.toString());
    }

    book.close();
}

private String getCellStringValue(XSSFCell cell) {
    try {
        if (null!=cell) {
            return String.valueOf(cell.getStringCellValue());
        }
    } catch (Exception e) {
        return String.valueOf(getCellIntValue(cell));
    }
    return "";
}

private long getCellLongValue(XSSFCell cell) {
    try {
        if (null!=cell) {
            return Long.parseLong("" + (long) cell.getNumericCellValue());
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return 0L;
}

private int getCellIntValue(XSSFCell cell) {
    try {
        if (null!=cell) {
            return Integer.parseInt("" + (int) cell.getNumericCellValue());
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return 0;
}
```

### SpringBoot集成文件 - 集成EasyExcel之Excel导入导出

#### 什么是EasyExcel

> EasyExcel是**阿里开源的基于POI封装的**Excel处理工具，更多请参考[官方文档在新窗口打开](https://poi.apache.org/index.html)。

EasyExcel是一个**基于Java的、快速、简洁、解决大文件内存溢出**的Excel处理工具。它能让你在不用考虑性能、内存的等因素的情况下，快速完成Excel的读、写等功能。

#### [#](#easyexcel要解决poi什么问题) EasyExcel要解决POI什么问题？

> 因为EasyExcel是基于POI封装的，主要考虑的是易用性，封装性和性能问题。

Java解析、生成Excel比较有名的框架有Apache poi、jxl。但他们都存在一个严重的问题就是非常的**耗内存**，poi有一套SAX模式的API可以一定程度的解决一些内存溢出的问题，但POI还是有一些缺陷，比如07版Excel解压缩以及解压后存储都是在内存中完成的，内存消耗依然很大。easyexcel重写了poi对07版Excel的解析，一个**3M的excel用POI sax解析依然需要100M左右内存，改用easyexcel可以降低到几M**，并且再大的excel也不会出现内存溢出；03版依赖POI的sax模式，在上层做了模型转换的封装，让使用者更加简单方便。

#### 实现案例

##### 导出Excel

UserController中导出的方法

```java
@ApiOperation("Download Excel")
@GetMapping("/excel/download")
public void download(HttpServletResponse response) {
    try {
        response.reset();
        response.setContentType("application/vnd.ms-excel");
        response.setHeader("Content-disposition",
                "attachment;filename=user_excel_" + System.currentTimeMillis() + ".xlsx");
        userService.downloadExcel(response.getOutputStream());
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

UserServiceImple中导出Excel的主方法(是不是很简洁)

```java
@Override
public void downloadExcel(ServletOutputStream outputStream) {
    EasyExcelFactory.write(outputStream, User.class).sheet("User").doWrite(this::getUserList);
}
private List<User> getUserList() {
    return Collections.singletonList(User.builder()
            .id(1L).userName("pdai").email("pdai@pdai.tech").phoneNumber(121231231231L)
            .description("hello world")
            .build());
}
```

##### 导入Excel

> 我们将上面导出的excel文件导入。

UserController中导入的方法

```java
@ApiOperation("Upload Excel")
@PostMapping("/excel/upload")
public ResponseResult<String> upload(@RequestParam(value = "file", required = true) MultipartFile file) {
    try {
        userService.upload(file.getInputStream());
    } catch (Exception e) {
        e.printStackTrace();
        return ResponseResult.fail(e.getMessage());
    }
    return ResponseResult.success();
}
```

UserServiceImple中导入Excel的主方法

```java
@Override
public void upload(InputStream inputStream) throws IOException {
    // ReadListener不是必须的，它主要的设计是读取excel数据的后置处理(并考虑一次性读取到内存潜在的内存泄漏问题)
    EasyExcelFactory.read(inputStream, User.class, new ReadListener<User>() {

        @Override
        public void invoke(User user, AnalysisContext analysisContext) {
            cachedDataList.add(user);
        }

        @Override
        public void doAfterAllAnalysed(AnalysisContext analysisContext) {
            cachedDataList.forEach(user -> log.info(user.toString()));
        }
    }).sheet().doRead();
}
```

##### 填充Excel模板

我们先来准备一个excel模板，考虑了横向表和纵向列表，以及单一信息等，基本上能满足多数的应用场景。

![img](https://www.pdai.tech/images/spring/springboot/springboot-file-excel-easyexcel-4.png)

UserController中下载填充后的Excel方法

```java
@ApiOperation("Fill Excel Template")
@GetMapping("/excel/fill")
public void fillTemplate(HttpServletResponse response) {
    try {
        response.reset();
        response.setContentType("application/vnd.ms-excel");
        response.setHeader("Content-disposition",
                "attachment;filename=user_excel_template_" + System.currentTimeMillis() + ".xlsx");
        userService.fillExcelTemplate(response.getOutputStream());
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

UserServiceImpl中填充excel模板的方法

```java
// 模板注意 用{} 来表示你要用的变量 如果本来就有"{","}" 特殊字符 用"\{","\}"代替
// {} 代表普通变量 {.} 代表是list的变量 {前缀.} 前缀可以区分不同的list
@Override
public void fillExcelTemplate(ServletOutputStream outputStream) {

    // 确保文件可访问，这个例子的excel模板，放在根目录下面
    String templateFileName = "/Users/pdai/Downloads/user_excel_template.xlsx";

    // 方案1
    try (ExcelWriter excelWriter = EasyExcelFactory.write(outputStream).withTemplate(templateFileName).build()) {
        WriteSheet writeSheet = EasyExcelFactory.writerSheet().build();
        FillConfig fillConfig = FillConfig.builder().direction(WriteDirectionEnum.HORIZONTAL).build();
        // 如果有多个list 模板上必须有{前缀.} 这里的前缀就是 userList，然后多个list必须用 FillWrapper包裹
        excelWriter.fill(new FillWrapper("userList", getUserList()), fillConfig, writeSheet);
        excelWriter.fill(new FillWrapper("userList", getUserList()), fillConfig, writeSheet);

        excelWriter.fill(new FillWrapper("userList2", getUserList()), writeSheet);
        excelWriter.fill(new FillWrapper("userList2", getUserList()), writeSheet);

        Map<String, Object> map = new HashMap<>();
        map.put("user", "pdai");
        map.put("date", new Date());

        excelWriter.fill(map, writeSheet);
    }
}
```

访问http://localhost:8080/user/excel/fill 下载

![img](C:\Users\1\Desktop\JAVA全栈知识体系\JAVA 全栈知识体系.assets\springboot-file-excel-easyexcel-3.png)

### SpringBoot集成文件 - 集成EasyPOI之Excel导入导出

**不建议在稍复杂的项目中使用EasyPOI**。

#### 实现案例

##### 导出Excel

UserController中导出的方法

```java
@ApiOperation("Download Excel")
@GetMapping("/excel/download")
public void download(HttpServletResponse response) {
    try {
        response.reset();
        response.setContentType("application/vnd.ms-excel");
        response.setHeader("Content-disposition",
                "attachment;filename=user_excel_" + System.currentTimeMillis() + ".xlsx");
        userService.downloadExcel(response.getOutputStream());
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

UserServiceImple中导出Excel的主方法

```java
@Override
public void downloadExcel(ServletOutputStream outputStream) throws IOException {
    ExportParams exportParams = new ExportParams();
    exportParams.setTitle("User Table");
    exportParams.setSheetName("User Sheet");
    Workbook workbook = ExcelExportUtil.exportExcel(exportParams, User.class, getUserList());
    workbook.write(outputStream);
    workbook.close();
}
```

```java
private List<User> getUserList() {
    List<User> userList = new ArrayList<>();
    userList.add(User.builder()
            .id(1L).userName("pdai").email("pdai@pdai.tech").phoneNumber(121231231231L)
            .description("hello world")
            .build());
    return userList;
}
```

##### 导入Excel

UserController中导入的方法，（导入的文件在项目根目录）

```java
@ApiOperation("Upload Excel")
@PostMapping("/excel/upload")
public ResponseResult<String> upload(@RequestParam(value = "file", required = true) MultipartFile file) {
    try {
        userService.upload(file.getInputStream());
    } catch (Exception e) {
        e.printStackTrace();
        return ResponseResult.fail(e.getMessage());
    }
    return ResponseResult.success();
}
```

UserServiceImple中导入Excel的主方法

```java
@Override
public void upload(InputStream inputStream) throws Exception {
    ImportParams importParams = new ImportParams();
    List<User> userList = ExcelImportUtil.importExcel(inputStream, User.class, importParams);
    userList.stream().forEach(user -> log.info(user.toString()));
}
```

##### 填充Excel模板

UserController中下载填充后的Excel方法

```java
@ApiOperation("Fill Excel Template")
@GetMapping("/excel/fill")
public void fillTemplate(HttpServletResponse response) {
    try {
        response.reset();
        response.setContentType("application/vnd.ms-excel");
        response.setHeader("Content-disposition",
                "attachment;filename=user_excel_template_" + System.currentTimeMillis() + ".xlsx");
        userService.fillExcelTemplate(response.getOutputStream());
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

UserServiceImpl中填充excel模板的方法

```java
@Override
public void fillExcelTemplate(ServletOutputStream outputStream) throws IOException {
    // 确保文件可访问，这个例子的excel模板，放在根目录下面
    String templateFileName = "/Users/pdai/Downloads/user_excel_template_easypoi.xlsx";
    TemplateExportParams params = new TemplateExportParams(templateFileName);

    Map<String, Object> map = new HashMap<>();
    map.put("user", "pdai");
    map.put("date", new Date());
    map.put("userList", getUserList());
    Workbook workbook = ExcelExportUtil.exportExcel(params, map);
    workbook.write(outputStream);
    workbook.close();
}

private List<User> getUserList() {
    List<User> userList = new ArrayList<>();
    userList.add(User.builder()
            .id(1L).userName("pdai").email("pdai@pdai.tech").phoneNumber(121231231231L)
            .description("hello world")
            .build());
    userList.add(User.builder()
            .id(2L).userName("pdai2").email("pdai2@pdai.tech").phoneNumber(1212312312312L)
            .description("hello world2")
            .build());
    return userList;
}
```

### SpringBoot集成文件 - 集成POI之Word导出

#### 导出Word

UserController中导出的方法

```java
package tech.pdai.springboot.file.word.poi.controller;


import java.io.OutputStream;

import javax.servlet.http.HttpServletResponse;

import io.swagger.annotations.ApiOperation;
import org.apache.poi.xwpf.usermodel.XWPFDocument;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import tech.pdai.springboot.file.word.poi.service.IUserService;

/**
 * @author pdai
 */
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private IUserService userService;

    @ApiOperation("Download Word")
    @GetMapping("/word/download")
    public void download(HttpServletResponse response) {
        try {
            XWPFDocument document = userService.generateWordXWPFDocument();
            response.reset();
            response.setContentType("application/vnd.ms-excel");
            response.setHeader("Content-disposition",
                    "attachment;filename=user_world_" + System.currentTimeMillis() + ".docx");
            OutputStream os = response.getOutputStream();
            document.write(os);
            os.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

UserServiceImple中导出Word方法

```java
package tech.pdai.springboot.file.word.poi.service.impl;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.math.BigInteger;
import java.util.ArrayList;
import java.util.List;

import lombok.extern.slf4j.Slf4j;
import org.apache.poi.openxml4j.exceptions.InvalidFormatException;
import org.apache.poi.util.Units;
import org.apache.poi.xwpf.usermodel.BreakType;
import org.apache.poi.xwpf.usermodel.Document;
import org.apache.poi.xwpf.usermodel.ParagraphAlignment;
import org.apache.poi.xwpf.usermodel.XWPFDocument;
import org.apache.poi.xwpf.usermodel.XWPFParagraph;
import org.apache.poi.xwpf.usermodel.XWPFRun;
import org.apache.poi.xwpf.usermodel.XWPFTable;
import org.apache.poi.xwpf.usermodel.XWPFTableCell;
import org.openxmlformats.schemas.wordprocessingml.x2006.main.CTTblPr;
import org.openxmlformats.schemas.wordprocessingml.x2006.main.CTTblWidth;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Service;
import tech.pdai.springboot.file.word.poi.entity.User;
import tech.pdai.springboot.file.word.poi.service.IUserService;

/**
 * @author pdai
 */
@Slf4j
@Service
public class UserServiceImpl implements IUserService {

    @Override
    public XWPFDocument generateWordXWPFDocument() {
        XWPFDocument doc = new XWPFDocument();

        // Title
        createTitle(doc, "Java 全栈知识体系");

        // Chapter 1
        createChapterH1(doc, "1. 知识准备");
        createChapterH2(doc, "1.1 什么是POI");
        createParagraph(doc, "Apache POI 是创建和维护操作各种符合Office Open XML（OOXML）标准和微软的OLE 2复合文档格式（OLE2）的Java API。用它可以使用Java读取和创建,修改MS Excel文件.而且,还可以使用Java读取和创建MS Word和MSPowerPoint文件。更多请参考[官方文档](https://poi.apache.org/index.html)");
        createChapterH2(doc, "1.2 POI中基础概念");
        createParagraph(doc, "生成xls和xlsx有什么区别？POI对Excel中的对象的封装对应关系？");

        // Chapter 2
        createChapterH1(doc, "2. 实现案例");
        createChapterH2(doc, "2.1 用户列表示例");
        createParagraph(doc, "以导出用户列表为例");

        // 表格
        List<User> userList = getUserList();
        XWPFParagraph paragraph = doc.createParagraph();
        XWPFTable table = paragraph.getDocument().createTable(userList.size(), 5);
        table.setWidth(500);
        table.setCellMargins(20, 20, 20, 20);

        //表格属性
        CTTblPr tablePr = table.getCTTbl().addNewTblPr();
        //表格宽度
        CTTblWidth width = tablePr.addNewTblW();
        width.setW(BigInteger.valueOf(8000));

        for(int i = 0; i< userList.size(); i++) {
            List<XWPFTableCell> tableCells = table.getRow(i).getTableCells();
            tableCells.get(0).setText(userList.get(i).getId()+"");
            tableCells.get(1).setText(userList.get(i).getUserName());
            tableCells.get(2).setText(userList.get(i).getEmail());
            tableCells.get(3).setText(userList.get(i).getPhoneNumber()+"");
            tableCells.get(4).setText(userList.get(i).getDescription());
        }

        createChapterH2(doc, "2.2 图片导出示例");
        createParagraph(doc, "以导出图片为例");
        // 图片
        InputStream stream = null;
        try {
            XWPFParagraph paragraph2 = doc.createParagraph();
            Resource resource = new ClassPathResource("pdai-guli.png");
            stream = new FileInputStream(resource.getFile());
            XWPFRun run = paragraph2.createRun();
            run.addPicture(stream, Document.PICTURE_TYPE_PNG, "Generated", Units.toEMU(256), Units.toEMU(256));
        } catch (IOException | InvalidFormatException e) {
            e.printStackTrace();
        }

        return doc;
    }

    private void createTitle(XWPFDocument doc, String content) {
        XWPFParagraph title = doc.createParagraph();
        title.setAlignment(ParagraphAlignment.CENTER);
        XWPFRun r1 = title.createRun();
        r1.setBold(true);
        r1.setFontFamily("宋体");
        r1.setText(content);
        r1.setFontSize(22);
    }

    private void createChapterH1(XWPFDocument doc, String content) {
        XWPFParagraph actTheme = doc.createParagraph();
        actTheme.setAlignment(ParagraphAlignment.LEFT);
        XWPFRun runText1 = actTheme.createRun();
        runText1.setBold(true);
        runText1.setText(content);
        runText1.setFontSize(18);
    }

    private void createChapterH2(XWPFDocument doc, String content) {
        XWPFParagraph actType = doc.createParagraph();
        XWPFRun runText2 = actType.createRun();
        runText2.setBold(true);
        runText2.setText(content);
        runText2.setFontSize(15);
    }

    private void createParagraph(XWPFDocument doc, String content) {
        XWPFParagraph actType = doc.createParagraph();
        XWPFRun runText2 = actType.createRun();
        runText2.setText(content);
        runText2.setFontSize(11);
    }

    private List<User> getUserList() {
        List<User> userList = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            userList.add(User.builder()
                    .id(Long.parseLong(i + "")).userName("pdai" + i).email("pdai@pdai.tech" + i).phoneNumber(121231231231L)
                    .description("hello world" + i)
                    .build());
        }
        return userList;
    }

}
```

### SpringBoot集成文件 - 集成POI-tl之基于模板的Word导出

poi-tl（poi template language）是**Word模板引擎**，使用Word模板和数据创建很棒的Word文档。

#### poi-tl的TDO模式

> TDO模式：Template + data-model = output

以官网的例子为例：

```java
XWPFTemplate template = XWPFTemplate.compile("template.docx").render(
  new HashMap<String, Object>(){{
    put("title", "Hi, poi-tl Word模板引擎");
}});  
template.writeAndClose(new FileOutputStream("output.docx")); 
```

- compile 编译模板 - Template
- render 渲染数据 - data-model
- write 输出到流 - output

#### [#](#template-模板) Template：模板

模板是Docx格式的Word文档，你可以使用Microsoft office、WPS Office、Pages等任何你喜欢的软件制作模板，也可以使用Apache POI代码来生成模板。

所有的标签都是以`{{`开头，以`}}`结尾，标签可以出现在任何位置，包括页眉，页脚，表格内部，文本框等，表格布局可以设计出很多优秀专业的文档，推荐使用表格布局。

poi-tl模板遵循“所见即所得”的设计，模板和标签的样式会被完全保留。

#### [#](#data-model-数据) Data-model：数据

数据类似于哈希或者字典，可以是Map结构（key是标签名称）：

```java
Map<String, Object> data = new HashMap<>();
data.put("name", "Sayi");
data.put("start_time", "2019-08-04");
```

可以是对象（属性名是标签名称）：

```java
public class Data {
  private String name;
  private String startTime;
  private Author author;
}
```

数据可以是树结构，每级之间用点来分隔开，比如`{ {author.name} }`标签对应的数据是author对象的name属性值。

**Word模板不是由简单的文本表示，所以在渲染图片、表格等元素时提供了数据模型，它们都实现了接口RenderData**，比如图片数据模型PictureRenderData包含图片路径、宽、高三个属性。

#### [#](#output-输出) Output：输出

以流的方式进行输出：

```java
template.write(OutputStream stream);
```

可以写到任意输出流中，比如文件流：

```java
template.write(new FileOutputStream("output.docx"));
```

比如网络流：

```java
response.setContentType("application/octet-stream");
response.setHeader("Content-disposition","attachment;filename=\""+"out_template.docx"+"\"");

// HttpServletResponse response
OutputStream out = response.getOutputStream();
BufferedOutputStream bos = new BufferedOutputStream(out);
template.write(bos);
bos.flush();
out.flush();
PoitlIOUtils.closeQuietlyMulti(template, bos, out); // 最后不要忘记关闭这些流。
```

### SpringBoot集成文件 - 集成itextpdf之导出PDF

见：https://www.pdai.tech/md/spring/springboot/springboot-x-file-pdf-itext.html

### SpringBoot应用部署 - 打包成jar部署

**通过Maven打包成jar**

![img](https://www.pdai.tech/images/spring/springboot/springboot-x-jar-8.png)

**通过Idea打包成jar**

1. File栏选择Project Structure
2. 在Artifacts下添加jar
3. 选择module，main class等
4. 确认编译后的jar输出位置
5. 进行build
6. 点击build/rebuild
7. 编译后的jar

### SpringBoot应用部署 - 使用第三方JAR包

#### 方案一：安装到Maven仓库

#### 方案二：使用systemPath属性

### Redis知识体系详解

![img](https://www.pdai.tech/images/db/redis/db-redis-overview.png)

### Redis入门 - Redis概念和基础

#### 什么是Redis

Redis是一款**内存高速缓存数据库**。Redis全称为：**Remote Dictionary Server**（远程数据服务），使用C语言编写，Redis是一个key-value存储系统（键值存储系统），支持丰富的数据类型，如：String、list、set、zset、hash。

Redis是一种支持key-value等多种数据结构的存储系统。可用于缓存，事件发布或订阅，高速队列等场景。支持网络，提供字符串，哈希，列表，队列，集合结构直接存取，基于内存，可持久化。

#### 为什么要使用Redis

**读写性能优异**

- Redis能读的速度是110000次/s,写的速度是81000次/s （测试条件见下一节）。

**数据类型丰富**

- Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。

**原子性**

- Redis的所有操作都是原子性的，同时Redis还支持对几个操作全并后的原子性执行。

**丰富的特性**

- Redis支持 publish/subscribe, 通知, key 过期等特性。

**持久化**

- Redis支持RDB, AOF等持久化方式

**发布订阅**

- Redis支持发布/订阅模式

**分布式**

- Redis Cluster

#### Redis的使用场景

##### 热点数据的缓存

缓存是Redis最常见的应用场景，之所有这么使用，主要是因为Redis读写性能优异。而且逐渐有取代memcached，成为首选服务端缓存的组件。而且，Redis内部是支持事务的，在使用时候能有效保证数据的一致性。

作为缓存使用时，一般有两种方式保存数据：

- **读取前，先去读Redis，如果没有数据，读取数据库，将数据拉入Redis。**
- **插入数据时，同时写入Redis。**

方案一：实施起来简单，但是有两个需要注意的地方：

- 避免缓存击穿。（数据库没有就需要命中的数据，导致Redis一直没有数据，而一直命中数据库。）
- 数据的实时性相对会差一点。

方案二：数据实时性强，但是开发时不便于统一处理。

当然，两种方式根据实际情况来适用。如：方案一适用于对于数据实时性要求不是特别高的场景。方案二适用于字典表、数据量不大的数据存储。

##### [#](#限时业务的运用) 限时业务的运用

redis中可以使用expire命令设置一个键的生存时间，到时间后redis会删除它。利用这一特性可以运用在**限时的优惠活动信息、手机验证码**等业务场景。

##### [#](#计数器相关问题) 计数器相关问题

redis由于incrby命令可以实现原子性的递增，所以可以运用于**高并发的秒杀活动、分布式序列号的生成、具体业务还体现在比如限制一个手机号发多少条短信、一个接口一分钟限制多少请求、一个接口一天限制调用多少次**等等。

##### [#](#分布式锁) 分布式锁

这个主要利用redis的setnx命令进行，setnx："set if not exists"就是如果不存在则成功设置缓存同时返回1，否则返回0 ，这个特性在很多后台中都有所运用，因为我们服务器是集群的，定时任务可能在两台机器上都会运行，所以在定时任务中首先 通过setnx设置一个lock， 如果成功设置则执行，如果没有成功设置，则表明该定时任务已执行。 当然结合具体业务，我们可以给这个lock加一个过期时间，比如说30分钟执行一次的定时任务，那么这个过期时间设置为小于30分钟的一个时间就可以，这个与定时任务的周期以及定时任务执行消耗时间相关。

在分布式锁的场景中，主要用在比如秒杀系统等。

##### [#](#延时操作) 延时操作

比如在订单生产后我们占用了库存，10分钟后去检验用户是否真正购买，如果没有购买将该单据设置无效，同时还原库存。 由于redis自2.8.0之后版本提供Keyspace Notifications功能，允许客户订阅Pub/Sub频道，以便以某种方式接收影响Redis数据集的事件。 所以我们对于上面的需求就可以用以下解决方案，我们在订单生产时，设置一个key，同时设置10分钟后过期， 我们在后台实现一个监听器，监听key的实效，监听到key失效时将后续逻辑加上。

当然我们也可以利用rabbitmq、activemq等消息中间件的延迟队列服务实现该需求。

##### [#](#排行榜相关问题) 排行榜相关问题

关系型数据库在排行榜方面查询速度普遍偏慢，所以可以借助redis的SortedSet进行热点数据的排序。

比如点赞排行榜，做一个SortedSet, 然后以用户的openid作为上面的username, 以用户的点赞数作为上面的score, 然后针对每个用户做一个hash, 通过zrangebyscore就可以按照点赞数获取排行榜，然后再根据username获取用户的hash信息，这个当时在实际运用中性能体验也蛮不错的。

##### [#](#点赞、好友等相互关系的存储) 点赞、好友等相互关系的存储

Redis 利用集合的一些命令，比如求交集、并集、差集等。

在微博应用中，每个用户关注的人存在一个集合中，就很容易实现求两个人的共同好友功能。

##### [#](#简单队列) 简单队列

由于Redis有list push和list pop这样的命令，所以能够很方便的执行队列操作。

### Redis入门 - 数据类型：5种基础数据类型详解



#### Redis数据结构简介

首先对redis来说，所有的**key（键）都是字符串**。我们在谈基础数据结构时，讨论的是**存储值的数据类型**，主要包括常见的5种数据类型，分别是：String、List、Set、Zset、Hash。

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-1.jpeg)

| 结构类型         | 结构存储的值                               | 结构的读写能力                                               |
| ---------------- | ------------------------------------------ | ------------------------------------------------------------ |
| **String字符串** | 可以是字符串、整数或浮点数                 | 对整个字符串或字符串的一部分进行操作；对整数或浮点数进行自增或自减操作； |
| **List列表**     | 一个链表，链表上的每个节点都包含一个字符串 | 对链表的两端进行push和pop操作，读取单个或多个元素；根据值查找或删除元素； |
| **Set集合**      | 包含字符串的无序集合                       | 字符串的集合，包含基础的方法有看是否存在添加、获取、删除；还包含计算交集、并集、差集等 |
| **Hash散列**     | 包含键值对的无序散列表                     | 包含方法有添加、获取、删除单个元素                           |
| **Zset有序集合** | 和散列一样，用于存储键值对                 | 字符串成员与浮点数分数之间的有序映射；元素的排列顺序由分数的大小决定；包含方法有添加、获取、删除单个元素以及根据分值范围或成员来获取元素 |

#### [#](#基础数据结构详解) 基础数据结构详解

##### String字符串

> String是redis中最基本的数据类型，**一个key对应一个value**。

String类型是二进制安全的，意思是 redis 的 string 可以包含任何数据。如数字，字符串，jpg图片或者序列化的对象。

- **图例**

下图是一个String类型的实例，其中键为hello，值为world

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-3.png)

- **命令使用**

| 命令   | 简述                   | 使用              |
| ------ | ---------------------- | ----------------- |
| GET    | 获取存储在给定键中的值 | GET name          |
| SET    | 设置存储在给定键中的值 | SET name value    |
| DEL    | 删除存储在给定键中的值 | DEL name          |
| INCR   | 将键存储的值加1        | INCR key          |
| DECR   | 将键存储的值减1        | DECR key          |
| INCRBY | 将键存储的值加上整数   | INCRBY key amount |
| DECRBY | 将键存储的值减去整数   | DECRBY key amount |

- **命令执行**

```bash
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> del hello
(integer) 1
127.0.0.1:6379> get hello
(nil)
127.0.0.1:6379> set counter 2
OK
127.0.0.1:6379> get counter
"2"
127.0.0.1:6379> incr counter
(integer) 3
127.0.0.1:6379> get counter
"3"
127.0.0.1:6379> incrby counter 100
(integer) 103
127.0.0.1:6379> get counter
"103"
127.0.0.1:6379> decr counter
(integer) 102
127.0.0.1:6379> get counter
"102"
```

- 实战场景
  - **缓存**： 经典使用场景，把常用信息，字符串，图片或者视频等信息放到redis中，redis作为缓存层，mysql做持久化层，降低mysql的读写压力。
  - **计数器**：redis是单线程模型，一个命令执行完才会执行下一个，同时数据可以一步落地到其他的数据源。
  - **session**：常见方案spring session + redis实现session共享，

##### List列表

> Redis中的List其实就是**链表**（Redis用双端链表实现List）。

使用List结构，我们可以轻松地实现最新消息排队功能（比如新浪微博的TimeLine）。List的另一个应用就是消息队列，可以利用List的 PUSH 操作，将任务存放在List中，然后工作线程再用 POP 操作将任务取出进行执行。

- **图例**

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-5.png)

- **命令使用**

| 命令   | 简述                                                         | 使用             |
| ------ | ------------------------------------------------------------ | ---------------- |
| RPUSH  | 将给定值推入到列表右端                                       | RPUSH key value  |
| LPUSH  | 将给定值推入到列表左端                                       | LPUSH key value  |
| RPOP   | 从列表的右端弹出一个值，并返回被弹出的值                     | RPOP key         |
| LPOP   | 从列表的左端弹出一个值，并返回被弹出的值                     | LPOP key         |
| LRANGE | 获取列表在给定范围上的所有值                                 | LRANGE key 0 -1  |
| LINDEX | 通过索引获取列表中的元素。你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。 | LINDEX key index |

- 使用列表的技巧
  - lpush+lpop=Stack(栈)
  - lpush+rpop=Queue（队列）
  - lpush+ltrim=Capped Collection（有限集合）
  - lpush+brpop=Message Queue（消息队列）
- **命令执行**

```bash
127.0.0.1:6379> lpush mylist 1 2 ll ls mem
(integer) 5
127.0.0.1:6379> lrange mylist 0 -1
1) "mem"
2) "ls"
3) "ll"
4) "2"
5) "1"
127.0.0.1:6379> lindex mylist -1
"1"
127.0.0.1:6379> lindex mylist 10        # index不在 mylist 的区间范围内
(nil)
```

- 实战场景
  - **微博TimeLine**: 有人发布微博，用lpush加入时间轴，展示新的列表信息。
  - **消息队列**

##### Set集合

> Redis 的 Set 是 **String 类型的无序集合**。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

Redis 中集合是通过**哈希表**实现的，所以添加，删除，查找的复杂度都是 O(1)。

- **图例**

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-7.png)

- **命令使用**

| 命令      | 简述                                  | 使用                 |
| --------- | ------------------------------------- | -------------------- |
| SADD      | 向集合添加一个或多个成员              | SADD key value       |
| SCARD     | 获取集合的成员数                      | SCARD key            |
| SMEMBERS  | 返回集合中的所有成员                  | SMEMBERS key member  |
| SISMEMBER | 判断 member 元素是否是集合 key 的成员 | SISMEMBER key member |

其它一些集合操作，请参考这里https://www.runoob.com/redis/redis-sets.html

- **命令执行**

```bash
127.0.0.1:6379> sadd myset hao hao1 xiaohao hao
(integer) 3
127.0.0.1:6379> smembers myset
1) "xiaohao"
2) "hao1"
3) "hao"
127.0.0.1:6379> sismember myset hao
(integer) 1
```

- 实战场景
  - **标签**（tag）,给用户添加标签，或者用户给消息添加标签，这样有同一标签或者类似标签的可以给推荐关注的事或者关注的人。
  - **点赞，或点踩，收藏等**，可以放到set中实现

##### Hash散列

> Redis hash 是一个 string 类型的 field（字段） 和 value（值） 的映射表，hash 特别适合用于存储对象。

- **图例**

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-4.png)

- **命令使用**

| 命令    | 简述                                     | 使用                          |
| ------- | ---------------------------------------- | ----------------------------- |
| HSET    | 添加键值对                               | HSET hash-key sub-key1 value1 |
| HGET    | 获取指定散列键的值                       | HGET hash-key key1            |
| HGETALL | 获取散列中包含的所有键值对               | HGETALL hash-key              |
| HDEL    | 如果给定键存在于散列中，那么就移除这个键 | HDEL hash-key sub-key1        |

- **命令执行**

```bash
127.0.0.1:6379> hset user name1 hao
(integer) 1
127.0.0.1:6379> hset user email1 hao@163.com
(integer) 1
127.0.0.1:6379> hgetall user
1) "name1"
2) "hao"
3) "email1"
4) "hao@163.com"
127.0.0.1:6379> hget user user
(nil)
127.0.0.1:6379> hget user name1
"hao"
127.0.0.1:6379> hset user name2 xiaohao
(integer) 1
127.0.0.1:6379> hset user email2 xiaohao@163.com
(integer) 1
127.0.0.1:6379> hgetall user
1) "name1"
2) "hao"
3) "email1"
4) "hao@163.com"
5) "name2"
6) "xiaohao"
7) "email2"
8) "xiaohao@163.com"
```

- 实战场景
  - **缓存**： 能直观，相比string更节省空间，的维护缓存信息，如用户信息，视频信息等。

##### Zset有序集合

> Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的, 但分数(score)却可以重复。有序集合是通过两种数据结构实现：

1. **压缩列表(ziplist)**: ziplist是为了提高存储效率而设计的一种特殊编码的双向链表。它可以存储字符串或者整数，存储整数时是采用整数的二进制而不是字符串形式存储。它能在O(1)的时间复杂度下完成list两端的push和pop操作。但是因为每次操作都需要重新分配ziplist的内存，所以实际复杂度和ziplist的内存使用量相关
2. **跳跃表（zSkiplist)**: 跳跃表的性能可以保证在查找，删除，添加等操作的时候在对数期望时间内完成，这个性能是可以和平衡树来相比较的，而且在实现方面比平衡树要优雅，这是采用跳跃表的主要原因。跳跃表的复杂度是O(log(n))。

- **图例**

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-8.png)

- **命令使用**

| 命令   | 简述                                                     | 使用                           |
| ------ | -------------------------------------------------------- | ------------------------------ |
| ZADD   | 将一个带有给定分值的成员添加到有序集合里面               | ZADD zset-key 178 member1      |
| ZRANGE | 根据元素在有序集合中所处的位置，从有序集合中获取多个元素 | ZRANGE zset-key 0-1 withccores |
| ZREM   | 如果给定元素成员存在于有序集合中，那么就移除这个元素     | ZREM zset-key member1          |

更多命令请参考这里 https://www.runoob.com/redis/redis-sorted-sets.html

- **命令执行**

```bash
127.0.0.1:6379> zadd myscoreset 100 hao 90 xiaohao
(integer) 2
127.0.0.1:6379> ZRANGE myscoreset 0 -1
1) "xiaohao"
2) "hao"
127.0.0.1:6379> ZSCORE myscoreset hao
"100"
```

- 实战场景
  - **排行榜**：有序集合经典使用场景。例如小说视频等网站需要对用户上传的小说视频做排行榜，榜单可以按照用户关注数，更新时间，字数等打分，做排行。

### Redis入门 - 数据类型：3种特殊类型详解

三种特殊的数据类型，分别是 **HyperLogLogs**（基数统计）， **Bitmaps** (位图) 和 **geospatial** （地理位置）。

#### HyperLogLogs（基数统计）

- **什么是基数？**

举个例子，A = {1, 2, 3, 4, 5}， B = {3, 5, 6, 7, 9}；那么基数（**不重复的元素**）= 1, 2, 4, 6, 7, 9； （允许容错，即可以接受一定误差）

- **HyperLogLogs 基数统计用来解决什么问题**？

这个结构可以非常**省内存**的去**统计各种计数**，比如注册 IP 数、每日访问 IP 数、页面实时UV、在线用户数，共同好友数等。

- **它的优势体现在哪**？

一个大型的网站，每天 IP 比如有 100 万，粗算一个 IP 消耗 15 字节，那么 100 万个 IP 就是 15M。而 HyperLogLog 在 Redis 中每个键占用的内容都是 12K，理论存储近似接近 2^64 个值，不管存储的内容是什么，它一个基于基数估算的算法，只能比较准确的估算出基数，可以使用**少量固定的内存去存储并识别集合中的唯一元素**。而且这个估算的基数并不一定准确，是一个带有 0.81% 标准错误的近似值（对于可以接受一定容错的业务场景，比如IP数统计，UV等，是可以忽略不计的）。

- **相关命令使用**

```bash
127.0.0.1:6379> pfadd key1 a b c d e f g h i	# 创建第一组元素
(integer) 1
127.0.0.1:6379> pfcount key1					# 统计元素的基数数量
(integer) 9
127.0.0.1:6379> pfadd key2 c j k l m e g a		# 创建第二组元素
(integer) 1
127.0.0.1:6379> pfcount key2
(integer) 8
127.0.0.1:6379> pfmerge key3 key1 key2			# 合并两组：key1 key2 -> key3 并集
OK
127.0.0.1:6379> pfcount key3
(integer) 13
```

#### Bitmap （位存储）

> Bitmap 即位图数据结构，都是操作二进制位来进行记录，只有**0 和 1 两个状态**。

- **用来解决什么问题**？

比如：统计用户信息，活跃，不活跃！ 登录，未登录！ 打卡，不打卡！ **两个状态的，都可以使用 Bitmaps**！

如果存储一年的打卡状态需要多少内存呢？ 365 天 = 365 bit 1字节 = 8bit 46 个字节左右！

- **相关命令使用**

使用bitmap 来记录 周一到周日的打卡！ 周一：1 周二：0 周三：0 周四：1 ......

```bash
127.0.0.1:6379> setbit sign 0 1
(integer) 0
127.0.0.1:6379> setbit sign 1 1
(integer) 0
127.0.0.1:6379> setbit sign 2 0
(integer) 0
127.0.0.1:6379> setbit sign 3 1
(integer) 0
127.0.0.1:6379> setbit sign 4 0
(integer) 0
127.0.0.1:6379> setbit sign 5 0
(integer) 0
127.0.0.1:6379> setbit sign 6 1
(integer) 0
```

查看某一天是否有打卡！

```bash
127.0.0.1:6379> getbit sign 3
(integer) 1
127.0.0.1:6379> getbit sign 5
(integer) 0
```

统计操作，统计 打卡的天数！

```bash
127.0.0.1:6379> bitcount sign # 统计这周的打卡记录，就可以看到是否有全勤！
(integer) 3
```

#### geospatial (地理位置)

> Redis 的 Geo 在 Redis 3.2 版本就推出了! 这个功能可以推算地理位置的信息: 两地之间的距离, 方圆几里的人

**[#](#geoadd) geoadd：添加地理位置**

```bash
127.0.0.1:6379> geoadd china:city 118.76 32.04 manjing 112.55 37.86 taiyuan 123.43 41.80 shenyang
(integer) 3
127.0.0.1:6379> geoadd china:city 144.05 22.52 shengzhen 120.16 30.24 hangzhou 108.96 34.26 xian
(integer) 3
```

**规则**

两级无法直接添加，我们一般会下载城市数据(这个网址可以查询 GEO： http://www.jsons.cn/lngcode)！

- 有效的经度从-180度到180度。
- 有效的纬度从-85.05112878度到85.05112878度。

```bash
# 当坐标位置超出上述指定范围时，该命令将会返回一个错误。
127.0.0.1:6379> geoadd china:city 39.90 116.40 beijin
(error) ERR invalid longitude,latitude pair 39.900000,116.400000
```

[#](#geopos) **geopos：获取指定的成员的经度和纬度**

```bash
127.0.0.1:6379> geopos china:city taiyuan manjing
1) 1) "112.54999905824661255"
   1) "37.86000073876942196"
2) 1) "118.75999957323074341"
   1) "32.03999960287850968"
```

获得当前定位, 一定是一个坐标值!

**[#](#geodist) geodist：如果不存在, 返回空**

单位如下

- m
- km
- mi 英里
- ft 英尺

```bash
127.0.0.1:6379> geodist china:city taiyuan shenyang m
"1026439.1070"
127.0.0.1:6379> geodist china:city taiyuan shenyang km
"1026.4391"
```

**[#](#georadius) georadius：附近的人 ==> 获得所有附近的人的地址, 定位, 通过半径来查询**

获得指定数量的人

```bash
127.0.0.1:6379> georadius china:city 110 30 1000 km			以 100,30 这个坐标为中心, 寻找半径为1000km的城市
1) "xian"
2) "hangzhou"
3) "manjing"
4) "taiyuan"
127.0.0.1:6379> georadius china:city 110 30 500 km
1) "xian"
127.0.0.1:6379> georadius china:city 110 30 500 km withdist
1) 1) "xian"
   2) "483.8340"
127.0.0.1:6379> georadius china:city 110 30 1000 km withcoord withdist count 2
1) 1) "xian"
   2) "483.8340"
   3) 1) "108.96000176668167114"
      2) "34.25999964418929977"
2) 1) "manjing"
   2) "864.9816"
   3) 1) "118.75999957323074341"
      2) "32.03999960287850968"
```

参数 key 经度 纬度 半径 单位 [显示结果的经度和纬度] [显示结果的距离] [显示的结果的数量]

**[#](#georadiusbymember) georadiusbymember：显示与指定成员一定半径范围内的其他成员**

```bash
127.0.0.1:6379> georadiusbymember china:city taiyuan 1000 km
1) "manjing"
2) "taiyuan"
3) "xian"
127.0.0.1:6379> georadiusbymember china:city taiyuan 1000 km withcoord withdist count 2
1) 1) "taiyuan"
   2) "0.0000"
   3) 1) "112.54999905824661255"
      2) "37.86000073876942196"
2) 1) "xian"
   2) "514.2264"
   3) 1) "108.96000176668167114"
      2) "34.25999964418929977"
```

参数与 georadius 一样

**[#](#geohash-较少使用) geohash(较少使用)：该命令返回11个字符的hash字符串**

```bash
127.0.0.1:6379> geohash china:city taiyuan shenyang
1) "ww8p3hhqmp0"
2) "wxrvb9qyxk0"
```

将二维的经纬度转换为一维的字符串, 如果两个字符串越接近, 则距离越近

**[#](#底层) 底层**

> geo底层的实现原理实际上就是Zset, 我们可以通过Zset命令来操作geo

```bash
127.0.0.1:6379> type china:city
zset
```

查看全部元素 删除指定的元素

```bash
127.0.0.1:6379> zrange china:city 0 -1 withscores
 1) "xian"
 2) "4040115445396757"
 3) "hangzhou"
 4) "4054133997236782"
 5) "manjing"
 6) "4066006694128997"
 7) "taiyuan"
 8) "4068216047500484"
 9) "shenyang"
1)  "4072519231994779"
2)  "shengzhen"
3)  "4154606886655324"
127.0.0.1:6379> zrem china:city manjing
(integer) 1
127.0.0.1:6379> zrange china:city 0 -1
1) "xian"
2) "hangzhou"
3) "taiyuan"
4) "shenyang"
5) "shengzhen"
```

### Redis入门 - 数据类型：Stream详解

Redis Stream也是一种超轻量MQ并没有完全实现消息队列所有设计要点，这决定着它适用的场景。

#### Stream的结构

每个 Stream 都有唯一的名称，它就是 Redis 的 key，在我们首次使用 xadd 指令追加消息时自动创建。

![img](https://www.pdai.tech/images/db/redis/db-redis-stream-2.png)

上图解析：

- `Consumer Group` ：**消费组**，使用 XGROUP CREATE 命令创建，一个消费组有多个消费者(Consumer), 这些消费者之间是**竞争关系**。
- `last_delivered_id` ：**游标**，每个消费组会有个游标 last_delivered_id，任意一个消费者**读取了消息**都会使游标 last_delivered_id **往前移动**。
- `pending_ids` ：**消费者(Consumer)的状态变量，作用是维护消费者的未确认的 id**。 pending_ids 记录了当前已经被客户端读取的消息，但是还没有 `ack` (Acknowledge character：确认字符）。如果客户端没有ack，这个变量里面的消息ID会越来越多，一旦某个消息被ack，它就开始减少。这个pending_ids变量在Redis官方被称之为PEL，也就是Pending Entries List，这是一个很**核心**的数据结构，它用来**确保客户端至少消费了消息一次，而不会在网络传输的中途丢失了没处理**。

此外我们还需要理解两点：

- `消息ID`: 消息ID的形式是timestampInMillis-sequence，例如1527846880572-5，它**表示当前的消息在毫米时间戳1527846880572时产生，并且是该毫秒内产生的第5条消息**。消息ID可以由服务器自动生成，也可以由客户端自己指定，但是形式必须是**整数-整数**，而且必须是后面加入的消息的ID要大于前面的消息ID。
- `消息内容`: 消息内容就是**键值对**，形如hash结构的键值对，这没什么特别之处。

#### 增删改查

消息队列相关命令：

- XADD - 添加消息到末尾
- XTRIM - 对流进行修剪，限制长度
- XDEL - 删除消息
- XLEN - 获取流包含的元素数量，即消息长度
- XRANGE - 获取消息列表，会自动过滤已经删除的消息
- XREVRANGE - 反向获取消息列表，ID 从大到小
- XREAD - 以阻塞或非阻塞方式获取消息列表

```bash
# *号表示服务器自动生成ID，后面顺序跟着一堆key/value
127.0.0.1:6379> xadd codehole * name laoqian age 30  #  名字叫laoqian，年龄30岁
1527849609889-0  # 生成的消息ID
127.0.0.1:6379> xadd codehole * name xiaoyu age 29
1527849629172-0
127.0.0.1:6379> xadd codehole * name xiaoqian age 1
1527849637634-0
127.0.0.1:6379> xlen codehole
(integer) 3
127.0.0.1:6379> xrange codehole - +  # -表示最小值, +表示最大值
127.0.0.1:6379> xrange codehole - +
1) 1) 1527849609889-0
   1) 1) "name"
      1) "laoqian"
      2) "age"
      3) "30"
2) 1) 1527849629172-0
   1) 1) "name"
      1) "xiaoyu"
      2) "age"
      3) "29"
3) 1) 1527849637634-0
   1) 1) "name"
      1) "xiaoqian"
      2) "age"
      3) "1"
127.0.0.1:6379> xrange codehole 1527849629172-0 +  # 指定最小消息ID的列表
1) 1) 1527849629172-0
   2) 1) "name"
      2) "xiaoyu"
      3) "age"
      4) "29"
2) 1) 1527849637634-0
   2) 1) "name"
      2) "xiaoqian"
      3) "age"
      4) "1"
127.0.0.1:6379> xrange codehole - 1527849629172-0  # 指定最大消息ID的列表
1) 1) 1527849609889-0
   2) 1) "name"
      2) "laoqian"
      3) "age"
      4) "30"
2) 1) 1527849629172-0
   2) 1) "name"
      2) "xiaoyu"
      3) "age"
      4) "29"
127.0.0.1:6379> xdel codehole 1527849609889-0
(integer) 1
127.0.0.1:6379> xlen codehole  # 长度不受影响
(integer) 3
127.0.0.1:6379> xrange codehole - +  # 被删除的消息没了
1) 1) 1527849629172-0
   2) 1) "name"
      2) "xiaoyu"
      3) "age"
      4) "29"
2) 1) 1527849637634-0
   2) 1) "name"
      2) "xiaoqian"
      3) "age"
      4) "1"
127.0.0.1:6379> del codehole  # 删除整个Stream
(integer) 1
```

#### 独立消费

我们可以在**不定义消费组**的情况下进行Stream消息的独立消费，当Stream没有新消息时，甚至可以阻塞等待。Redis设计了一个**单独的消费指令xread**，可以将Stream当成普通的消息队列(list)来使用。使用xread时，我们可以完全忽略消费组(Consumer Group)的存在，就好比Stream就是一个普通的列表(list)。

```bash
# 从Stream头部读取两条消息
127.0.0.1:6379> xread count 2 streams codehole 0-0
1) 1) "codehole"
   2) 1) 1) 1527851486781-0
         2) 1) "name"
            2) "laoqian"
            3) "age"
            4) "30"
      2) 1) 1527851493405-0
         2) 1) "name"
            2) "yurui"
            3) "age"
            4) "29"
# 从Stream尾部读取一条消息，毫无疑问，这里不会返回任何消息
127.0.0.1:6379> xread count 1 streams codehole $
(nil)
# 从尾部阻塞等待新消息到来，下面的指令会堵住，直到新消息到来
127.0.0.1:6379> xread block 0 count 1 streams codehole $
# 我们从新打开一个窗口，在这个窗口往Stream里塞消息
127.0.0.1:6379> xadd codehole * name youming age 60
1527852774092-0
# 再切换到前面的窗口，我们可以看到阻塞解除了，返回了新的消息内容
# 而且还显示了一个等待时间，这里我们等待了93s
127.0.0.1:6379> xread block 0 count 1 streams codehole $
1) 1) "codehole"
   2) 1) 1) 1527852774092-0
         2) 1) "name"
            2) "youming"
            3) "age"
            4) "60"
(93.11s)
```

客户端如果想要使用xread进行**顺序消费**，一定要记住当前消费到哪里了，也就是返回的消息ID。下次继续调用xread时，将上次返回的**最后一个消息ID作为参数传递进去**，就可以继续消费后续的消息。

**block 0表示永远阻塞**，直到消息到来，**block 1000表示阻塞1s**，如果1s内没有任何消息到来，就返回nil

```bash
127.0.0.1:6379> xread block 1000 count 1 streams codehole $
(nil)
(1.07s)
```

#### 消费组消费

- **消费组消费图**

![img](https://www.pdai.tech/images/db/redis/db-redis-stream-3.png)

- 相关命令：
  - XGROUP CREATE - 创建消费者组
  - XREADGROUP GROUP - 读取消费者组中的消息
  - XACK - 将消息标记为"已处理"
  - XGROUP SETID - 为消费者组设置新的最后递送消息ID
  - XGROUP DELCONSUMER - 删除消费者
  - XGROUP DESTROY - 删除消费者组
  - XPENDING - 显示待处理消息的相关信息
  - XCLAIM - 转移消息的归属权
  - XINFO - 查看流和消费者组的相关信息；
  - XINFO GROUPS - 打印消费者组的信息；
  - XINFO STREAM - 打印流信息
- **创建消费组**

Stream通过**xgroup create**指令创建消费组(Consumer Group)，需要传递起始消息ID参数用来初始化last_delivered_id变量。

```bash
127.0.0.1:6379> xgroup create codehole cg1 0-0  #  表示从头开始消费
OK
# $表示从尾部开始消费，只接受新消息，当前Stream消息会全部忽略
127.0.0.1:6379> xgroup create codehole cg2 $
OK
127.0.0.1:6379> xinfo stream codehole  # 获取Stream信息
 1) length
 2) (integer) 3  # 共3个消息
 3) radix-tree-keys
 4) (integer) 1
 5) radix-tree-nodes
 6) (integer) 2
 7) groups
 8) (integer) 2  # 两个消费组
 9) first-entry  # 第一个消息
10) 1) 1527851486781-0
    2) 1) "name"
       2) "laoqian"
       3) "age"
       4) "30"
11) last-entry  # 最后一个消息
12) 1) 1527851498956-0
    2) 1) "name"
       2) "xiaoqian"
       3) "age"
       4) "1"
127.0.0.1:6379> xinfo groups codehole  # 获取Stream的消费组信息
1) 1) name
   2) "cg1"
   3) consumers
   4) (integer) 0  # 该消费组还没有消费者
   5) pending
   6) (integer) 0  # 该消费组没有正在处理的消息
2) 1) name
   2) "cg2"
   3) consumers  # 该消费组还没有消费者
   4) (integer) 0
   5) pending
   6) (integer) 0  # 该消费组没有正在处理的消息
```

- **消费组消费**

Stream提供了**xreadgroup**指令可以进行消费组的**组内消费**，需要提供消费组名称、消费者名称和起始消息ID。它同xread一样，也可以阻塞等待新消息。

读到新消息后，对应的消息ID就会**进入消费者的PEL(正在处理的消息)结构**里，客户端处理完毕后使用**xack**指令**通知服务器**，本条消息已经**处理完毕**，该消息ID就会从**PEL中移除**。

```bash
# >号表示从当前消费组的last_delivered_id后面开始读
# 每当消费者读取一条消息，last_delivered_id变量就会前进
127.0.0.1:6379> xreadgroup GROUP cg1 c1 count 1 streams codehole >
1) 1) "codehole"
   2) 1) 1) 1527851486781-0
         2) 1) "name"
            2) "laoqian"
            3) "age"
            4) "30"
127.0.0.1:6379> xreadgroup GROUP cg1 c1 count 1 streams codehole >
1) 1) "codehole"
   2) 1) 1) 1527851493405-0
         2) 1) "name"
            2) "yurui"
            3) "age"
            4) "29"
127.0.0.1:6379> xreadgroup GROUP cg1 c1 count 2 streams codehole >
1) 1) "codehole"
   2) 1) 1) 1527851498956-0
         2) 1) "name"
            2) "xiaoqian"
            3) "age"
            4) "1"
      2) 1) 1527852774092-0
         2) 1) "name"
            2) "youming"
            3) "age"
            4) "60"
# 再继续读取，就没有新消息了
127.0.0.1:6379> xreadgroup GROUP cg1 c1 count 1 streams codehole >
(nil)
# 那就阻塞等待吧
127.0.0.1:6379> xreadgroup GROUP cg1 c1 block 0 count 1 streams codehole >
# 开启另一个窗口，往里塞消息
127.0.0.1:6379> xadd codehole * name lanying age 61
1527854062442-0
# 回到前一个窗口，发现阻塞解除，收到新消息了
127.0.0.1:6379> xreadgroup GROUP cg1 c1 block 0 count 1 streams codehole >
1) 1) "codehole"
   2) 1) 1) 1527854062442-0
         2) 1) "name"
            2) "lanying"
            3) "age"
            4) "61"
(36.54s)
127.0.0.1:6379> xinfo groups codehole  # 观察消费组信息
1) 1) name
   2) "cg1"
   3) consumers
   4) (integer) 1  # 一个消费者
   5) pending
   6) (integer) 5  # 共5条正在处理的信息还有没有ack
2) 1) name
   2) "cg2"
   3) consumers
   4) (integer) 0  # 消费组cg2没有任何变化，因为前面我们一直在操纵cg1
   5) pending
   6) (integer) 0
# 如果同一个消费组有多个消费者，我们可以通过xinfo consumers指令观察每个消费者的状态
127.0.0.1:6379> xinfo consumers codehole cg1  # 目前还有1个消费者
1) 1) name
   2) "c1"
   3) pending
   4) (integer) 5  # 共5条待处理消息
   5) idle
   6) (integer) 418715  # 空闲了多长时间ms没有读取消息了
# 接下来我们ack一条消息
127.0.0.1:6379> xack codehole cg1 1527851486781-0
(integer) 1
127.0.0.1:6379> xinfo consumers codehole cg1
1) 1) name
   2) "c1"
   3) pending
   4) (integer) 4  # 变成了5条
   5) idle
   6) (integer) 668504
# 下面ack所有消息
127.0.0.1:6379> xack codehole cg1 1527851493405-0 1527851498956-0 1527852774092-0 1527854062442-0
(integer) 4
127.0.0.1:6379> xinfo consumers codehole cg1
1) 1) name
   2) "c1"
   3) pending
   4) (integer) 0  # pel空了
   5) idle
   6) (integer) 745505
```

#### 信息监控

Stream提供了XINFO来实现对服务器信息的监控，可以查询：

- 查看队列信息

```bash
127.0.0.1:6379> Xinfo stream mq
 1) "length"
 2) (integer) 7
 3) "radix-tree-keys"
 4) (integer) 1
 5) "radix-tree-nodes"
 6) (integer) 2
 7) "groups"
 8) (integer) 1
 9) "last-generated-id"
10) "1553585533795-9"
11) "first-entry"
12) 1) "1553585533795-3"
    2) 1) "msg"
       2) "4"
13) "last-entry"
14) 1) "1553585533795-9"
    2) 1) "msg"
       2) "10"
```

- 消费组信息

```bash
127.0.0.1:6379> Xinfo groups mq
1) 1) "name"
   2) "mqGroup"
   3) "consumers"
   4) (integer) 3
   5) "pending"
   6) (integer) 3
   7) "last-delivered-id"
   8) "1553585533795-4"
```

- 消费者组成员信息

```bash
127.0.0.1:6379> XINFO CONSUMERS mq mqGroup
1) 1) "name"
   2) "consumerA"
   3) "pending"
   4) (integer) 1
   5) "idle"
   6) (integer) 18949894
2) 1) "name"
   2) "consumerB"
   3) "pending"
   4) (integer) 1
   5) "idle"
   6) (integer) 3092719
3) 1) "name"
   2) "consumerC"
   3) "pending"
   4) (integer) 1
   5) "idle"
   6) (integer) 23683256
```

#### Stream用在什么样场景

可用作**实时通信，大数据分析，异地数据备份**等

![img](https://www.pdai.tech/images/db/redis/db-redis-stream-4.png)

客户端可以**平滑扩展，提高处理能力**

![img](https://www.pdai.tech/images/db/redis/db-redis-stream-5.png)

#### 消息ID的设计是否考虑了时间回拨的问题？

考虑了。

XADD生成的1553439850328-0，就是Redis生成的消息ID，由两部分组成:**时间戳-序号**。时间戳是毫秒级单位，是生成消息的Redis服务器时间，它是个64位整型（int64）。序号是在这个毫秒时间点内的消息序号，它也是个64位整型。

为了保证消息是有序的，因此Redis生成的ID是**单调递增有序**的。由于ID中包含时间戳部分，为了避免服务器时间错误而带来的问题（例如服务器时间延后了），Redis的每个Stream类型数据都维护一个latest_generated_id属性，用于记录最后一个消息的ID。**若发现当前时间戳退后（小于latest_generated_id所记录的），则采用时间戳不变而序号递增的方案来作为新消息ID**（这也是序号为什么使用int64的原因，保证有足够多的的序号），从而保证ID的单调递增性质。

#### 消费者崩溃带来的会不会消息丢失问题?

不会。

为了解决组内消息读取但处理期间消费者崩溃带来的消息丢失问题，STREAM 设计了 **Pending 列表，用于记录读取但并未处理完毕的消息**。命令XPENDIING 用来获消费组或消费内消费者的未处理完毕的消息。

```
127.0.0.1:6379> XPENDING mq mqGroup - + 10 # 使用 start end count 选项可以获取详细信息
1) 1) "1553585533795-0" # 消息ID
   2) "consumerA" # 消费者
   3) (integer) 1654355 # 从读取到现在经历了1654355ms，IDLE
   4) (integer) 5 # 消息被读取了5次，delivery counter
2) 1) "1553585533795-1"
   2) "consumerA"
   3) (integer) 1654355
   4) (integer) 4
# 共5个，余下3个省略 ...
```



#### 消费者彻底宕机后如何转移给其它消费者处理？

> 还有一个问题，就是若某个消费者宕机之后，没有办法再上线了，那么就需要将该消费者Pending的消息，转义给其他的消费者处理，就是**消息转移**。

消息转移的操作时将某个消息转移到自己的Pending列表中。使用语法**XCLAIM**来实现，需要设置组、转移的目标消费者和消息ID，同时需要提供**IDLE（已被读取时长）**，只有超过这个时长，才能被转移。演示如下：

```bash
# 当前属于消费者A的消息1553585533795-1，已经15907,787ms未处理了
127.0.0.1:6379> XPENDING mq mqGroup - + 10
1) 1) "1553585533795-1"
   2) "consumerA"
   3) (integer) 15907787
   4) (integer) 4

# 转移超过3600s的消息1553585533795-1到消费者B的Pending列表
127.0.0.1:6379> XCLAIM mq mqGroup consumerB 3600000 1553585533795-1
1) 1) "1553585533795-1"
   2) 1) "msg"
      2) "2"

# 消息1553585533795-1已经转移到消费者B的Pending中。
127.0.0.1:6379> XPENDING mq mqGroup - + 10
1) 1) "1553585533795-1"
   2) "consumerB"
   3) (integer) 84404 # 注意IDLE，被重置了
   4) (integer) 5 # 注意，读取次数也累加了1次
```

#### 坏消息问题，Dead Letter，死信问题

正如上面所说，如果某个消息，不能被消费者处理，也就是不能被XACK，这是要长时间处于Pending列表中，即使被反复的转移给各个消费者也是如此。此时该消息的**delivery counter就会累加**（上一节的例子可以看到），当累加到某个我们预设的**临界值**时，我们就认为是**坏消息（也叫死信，DeadLetter，无法投递的消息）**，由于有了判定条件，我们将坏消息处理掉即可，删除即可。删除一个消息，使用XDEL语法，演示如下：

```bash
# 删除队列中的消息
127.0.0.1:6379> XDEL mq 1553585533795-1
(integer) 1
# 查看队列中再无此消息
127.0.0.1:6379> XRANGE mq - +
1) 1) "1553585533795-0"
   2) 1) "msg"
      2) "1"
2) 1) "1553585533795-2"
   2) 1) "msg"
      2) "3"
```

注意本例中，并没有删除Pending中的消息因此你查看Pending，消息还会在。可以执行XACK标识其处理完毕！

### Redis进阶 - 数据结构：对象机制详解

Redis的每种对象其实都由**对象结构(redisObject)** 与 **对应编码的数据结构**组合而成, 本文主要介绍**对象结构(redisObject)** 部分。

#### 底层设计

![img](https://www.pdai.tech/images/db/redis/db-redis-object-2-2.png)

它反映了Redis的每种对象其实都由**对象结构(redisObject)** 与 **对应编码的数据结构**组合而成，而每种对象类型对应若干编码方式，不同的编码方式所对应的底层数据结构是不同的。

所以，我们需要从几个个角度来着手底层研究：

- **对象设计机制**: 对象结构(redisObject)
- **编码类型和底层数据结构**: 对应编码的数据结构

#### 为什么Redis会设计redisObject对象

> 为什么Redis会设计redisObject对象？

在redis的命令中，用于对键进行处理的命令占了很大一部分，而对于键所保存的值的类型（键的类型），键能执行的命令又各不相同。但是要正确实现这些命令, 必须为不同类型的键设置不同的处理方式: 比如说, 删除一个列表键和删除一个字符串键的操作过程就不太一样。

以上的描述说明, **Redis 必须让每个键都带有类型信息, 使得程序可以检查键的类型, 并为它选择合适的处理方式**.

比如说， 集合类型就可以由字典和整数集合两种不同的数据结构实现， 但是， 当用户执行 ZADD 命令时， 他/她应该不必关心集合使用的是什么编码， 只要 Redis 能按照 ZADD 命令的指示， 将新元素添加到集合就可以了。

这说明, **操作数据类型的命令除了要对键的类型进行检查之外, 还需要根据数据类型的不同编码进行多态处理**.

为了解决以上问题, **Redis 构建了自己的类型系统**, 这个系统的主要功能包括:

- redisObject 对象.
- 基于 redisObject 对象的类型检查.
- 基于 redisObject 对象的显式多态函数.
- 对 redisObject 进行分配、共享和销毁的机制.

#### redisObject数据结构

**redisObject 是 Redis 类型系统的核心****, 数据库中的每个键、值, 以及 Redis 本身处理的参数, 都表示为这种数据类型.**

```c
/*
 * Redis 对象
 */
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 编码方式
    unsigned encoding:4;

    // LRU - 24位, 记录最末一次访问时间（相对于lru_clock）; 或者 LFU（最少使用的数据：8位频率，16位访问时间）
    unsigned lru:LRU_BITS; // LRU_BITS: 24

    // 引用计数
    int refcount;

    // 指向底层数据结构实例
    void *ptr;

} robj;
```

下图对应上面的结构

![img](https://www.pdai.tech/images/db/redis/db-redis-object-1.png)

**其中type、encoding和ptr是最重要的三个属性**。

- **type记录了对象所保存的值的类型**，它的值可能是以下常量中的一个：

```c
/*
* 对象类型
*/
#define OBJ_STRING 0 // 字符串
#define OBJ_LIST 1 // 列表
#define OBJ_SET 2 // 集合
#define OBJ_ZSET 3 // 有序集
#define OBJ_HASH 4 // 哈希表
```

- **encoding记录了对象所保存的值的编码**，它的值可能是以下常量中的一个：

```c
/*
* 对象编码
*/
#define OBJ_ENCODING_RAW 0     /* Raw representation */
#define OBJ_ENCODING_INT 1     /* Encoded as integer */
#define OBJ_ENCODING_HT 2      /* Encoded as hash table */
#define OBJ_ENCODING_ZIPMAP 3  /* 注意：版本2.6后不再使用. */
#define OBJ_ENCODING_LINKEDLIST 4 /* 注意：不再使用了，旧版本2.x中String的底层之一. */
#define OBJ_ENCODING_ZIPLIST 5 /* Encoded as ziplist */
#define OBJ_ENCODING_INTSET 6  /* Encoded as intset */
#define OBJ_ENCODING_SKIPLIST 7  /* Encoded as skiplist */
#define OBJ_ENCODING_EMBSTR 8  /* Embedded sds string encoding */
#define OBJ_ENCODING_QUICKLIST 9 /* Encoded as linked list of ziplists */
#define OBJ_ENCODING_STREAM 10 /* Encoded as a radix tree of listpacks */
```

- **ptr是一个指针，指向实际保存值的数据结构**，这个数据结构由type和encoding属性决定。举个例子， 如果一个redisObject 的type 属性为`OBJ_LIST` ， encoding 属性为`OBJ_ENCODING_QUICKLIST` ，那么这个对象就是一个Redis 列表（List)，它的值保存在一个QuickList的数据结构内，而ptr 指针就指向quicklist的对象；

- **lru属性: 记录了对象最后一次被命令程序访问的时间**

**空转时长**：**当前时间减去键的值对象的lru时间**，就是该键的空转时长。Object idletime命令可以打印出给定键的空转时长

如果服务器打开了maxmemory选项，并且服务器用于回收内存的算法为volatile-lru或者allkeys-lru，那么当服务器占用的内存数超过了maxmemory选项所设置的上限值时，**空转时长较高的那部分键会优先被服务器释放，从而回收内存**。

#### 命令的类型检查和多态

> 那么Redis是如何处理一条命令的呢？

**当执行一个处理数据类型命令的时候，redis执行以下步骤**

- 根据给定的**key**，在数据库字典中查找和他相对应的**redisObject**，如果没找到，就返回NULL；
- 检查redisObject的**type**属性和执行命令所需的**类型**是否相符，如果不相符，返回类型错误；
- 根据redisObject的**encoding**属性所指定的**编码**，选择合适的操作函数来处理底层的数据结构；
- 返回数据结构的操作结果作为命令的**返回值**。

比如现在执行LPOP命令：

![img](https://www.pdai.tech/images/db/redis/db-redis-object-3.png)

#### 对象共享

> redis一般会把一些常见的值放到一个共享对象中，这样可使程序**避免了重复分配**的麻烦，也**节约了一些CPU时间**。

**redis预分配的值对象如下**：

- **各种命令的返回值**，比如成功时返回的OK，错误时返回的ERROR，命令入队事务时返回的QUEUE，等等
- **包括0 在内，小于REDIS_SHARED_INTEGERS的所有整数**（REDIS_SHARED_INTEGERS的默认值是10000）

![img](https://www.pdai.tech/images/db/redis/db-redis-object-4.png)

> 注意：共享对象只能被**字典和双向链表**这类能带有指针的数据结构使用。像整数集合和压缩列表这些只能保存字符串、整数等自勉之的内存数据结构

**为什么redis不共享列表对象、哈希对象、集合对象、有序集合对象，只共享字符串对象**？

- 列表对象、哈希对象、集合对象、有序集合对象，本身可以包含字符串对象，复杂度较高。
- 如果共享对象是保存字符串对象，那么验证操作的复杂度为O(1)
- 如果共享对象是保存字符串值的字符串对象，那么验证操作的复杂度为O(N)
- 如果共享对象是包含多个值的对象，其中值本身又是字符串对象，即其它对象中嵌套了字符串对象，比如列表对象、哈希对象，那么验证操作的复杂度将会是O(N的平方)

如果对复杂度较高的对象创建共享对象，需要**消耗很大的CPU**，用这种消耗去换取内存空间，是不合适的

#### [#](#引用计数以及对象的消毁) 引用计数以及对象的消毁

> redisObject中有**refcount**属性，是对象的**引用计数**，显然**计数0**那么就是可以**回收**。

- 每个redisObject结构都带有一个refcount属性，指示这个对象被引用了多少次；
- 当新创建一个对象时，它的refcount属性被设置为1；
- 当对一个对象进行共享时，redis将这个对象的refcount加一；
- 当使用完一个对象后，或者消除对一个对象的引用之后，程序将对象的refcount减一；
- 当对象的refcount降至0 时，这个RedisObject结构，以及它引用的数据结构的内存都会被释放。

#### [#](#小结) 小结

- redis使用自己实现的对象机制（redisObject)来实现**类型判断、命令多态和基于引用次数的垃圾回收**；
- redis会预分配一些常用的数据对象，并通过**共享这些对象来减少内存占用**，和**避免频繁的为小对象分配内存**。

### Redis进阶 - 数据结构：底层数据结构详解

- 简单动态字符串 - sds
- 压缩列表 - ZipList
- 快表 - QuickList
- 字典/哈希表 - Dict
- 整数集 - IntSet
- 跳表 - ZSkipList

#### 简单动态字符串 - sds

> 这是一种用于存储二进制数据的一种结构, 具有动态扩容的特点. 其实现位于src/sds.h与src/sds.c中。

- **SDS的总体概览**如下图:

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-3.png)

其中**`sdshdr`是头部, `buf`是真实存储用户数据的地方**. 另外注意, 从命名上能看出来, 这个数据结构除了能存储二进制数据, 显然是用于设计作为字符串使用的, 所以在buf中, 用户数据后总跟着一个\0. 即图中 **`"数据" + "\0" `是为所谓的buf**。

通过上图我们可以看到，SDS有五种不同的头部. 其中sdshdr5实际并未使用到. 所以实际上有四种不同的头部, 分别如下:

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-2.png)

其中：

- `len` 保存了SDS保存字符串的长度
- `buf[]` 数组用来保存字符串的每个元素
- `alloc`分别以uint8, uint16, uint32, uint64表示整个SDS, 除过头部与末尾的\0, 剩余的字节数.
- `flags` 始终为一字节, 以低三位标示着头部的类型, 高5位未使用.

##### [#](#为什么使用sds) 为什么使用SDS

- **常数复杂度获取字符串长度**

由于 len 属性的存在，我们获取 SDS 字符串的长度只需要读取 len 属性，时间复杂度为 **O(1)**。而对于 C 语言，获取字符串的长度通常是经过遍历计数来实现的，时间复杂度为 **O(n)**。通过 `strlen key` 命令可以获取 key 的字符串长度。

- **杜绝缓冲区溢出**

我们知道在 C 语言中使用 `strcat` 函数来进行两个字符串的拼接，一旦没有分配足够长度的内存空间，就会造成缓冲区溢出。而对于 SDS 数据类型，在进行字符修改的时候，**会首先根据记录的 len 属性检查内存空间是否满足需求**，如果不满足，会进行相应的空间扩展，然后在进行修改操作，所以不会出现缓冲区溢出。

- **减少修改字符串的内存重新分配次数**

C语言由于不记录字符串的长度，所以如果要修改字符串，必须要重新分配内存（先释放再申请），因为如果没有重新分配，字符串长度增大时会造成内存缓冲区溢出，字符串长度减小时会造成内存泄露。

而对于SDS，由于`len`属性和`alloc`属性的存在，对于修改字符串SDS实现了**空间预分配**和**惰性空间释放**两种策略：

1、`空间预分配`：对字符串进行空间扩展的时候，扩展的内存比实际需要的多，这样可以减少连续执行字符串增长操作所需的内存重分配次数。

2、`惰性空间释放`：对字符串进行缩短操作时，程序不立即使用内存重新分配来回收缩短后多余的字节，而是使用 `alloc` 属性将这些字节的数量记录下来，等待后续使用。（当然SDS也提供了相应的API，当我们有需要时，也可以手动释放这些未使用的空间。）

- **二进制安全**

因为C字符串以空字符作为字符串结束的标识，而对于一些二进制文件（如图片等），内容可能包括空字符串，因此C字符串无法正确存取；而所有 SDS 的API 都是以处理二进制的方式来处理 `buf` 里面的元素，并且 SDS 不是以空字符串来判断是否结束，而是以 **len 属性表示的长度来判断字符串是否结束**。

- **兼容部分 C 字符串函数**

虽然 SDS 是二进制安全的，但是一样遵从每个字符串都是以空字符串结尾的惯例，这样可以**重用 C 语言库`` 中的一部分函数。**

##### 小结

redis的字符串表示为sds，而不是C字符串（以\0结尾的char*）， 它是Redis 底层所使用的字符串表示，它被用在几乎所有的Redis 模块中。可以看如下对比：

![img](https://www.pdai.tech/images/db/redis/redis-ds-2.png)

#### 压缩列表 - ZipList

> ziplist是为了**提高存储效率**而设计的一种特殊编码的双向链表。它可以**存储字符串或者整数**，**存储整数**时是采用**整数的二进制而不是字符串形式存储**。它能在O(1)的时间复杂度下完成list两端的push和pop操作。但是因为每次操作都需要**重新分配ziplist的内存**，所以实际复杂度和ziplist的内存使用量相关。

##### [#](#ziplist结构) ziplist结构

整个ziplist在内存中的存储格式如下：

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-6.png)

- `zlbytes`字段的类型是uint32_t, 这个字段中存储的是整个ziplist所**占用的内存的字节数**
- `zltail`字段的类型是uint32_t, 它指的是ziplist中最后一个entry的偏移量. 用于**快速定位最后一个entry**, 以快速完成pop等操作
- `zllen`字段的类型是uint16_t, 它指的是整个ziplit中entry的数量. 这个值只占2bytes（16位）: 如果ziplist中entry的数目**小于65535(2的16次方)**, 那么该字段中存储的就是**实际entry的值**. 若**等于或超过65535,** 那么该字段的值固定为65535, 但实际数量需要**一个个entry的去遍历**所有entry才能得到.
- `zlend`是一个**终止字节**, 其值为全F, 即0xff. ziplist保证任何情况下, 一个entry的首字节都不会是255

##### Entry结构

**第一种情况**：一般结构 <prevlen> <encoding> <entry-data>`  `

`prevlen`：前一个entry的大小，编码方式见下文；

`encoding`：不同的情况下值不同，用于表示**当前entry的类型和长度**；

`entry-data`：真是用于存储entry表示的数据；

**第二种情况**：此时entry结构：<prevlen> <encoding>

在entry中存储的是int类型时，encoding和entry-data会合并在encoding中表示，此时没有entry-data字段；

redis中，在存储数据时，会先尝试将string转换成int存储，节省空间；

##### 为什么ZipList特别省内存

- ziplist节省内存是**相对于普通的list**来说的，如果是普通的数组，那么它每个元素占用的内存是一样的且取决于最大的那个元素（很明显它是需要预留空间的）；
- 所以ziplist在设计时就很容易想到要尽量让每个元素按照实际的内容大小存储，**所以增加encoding字段**，针对不同的encoding来细化存储大小；
- 这时候还需要解决的一个问题是遍历元素时如何定位下一个元素呢？在普通数组中每个元素定长，所以不需要考虑这个问题；但是ziplist中每个data占据的内存不一样，所以为了解决遍历，需要增加记录上一个元素的length，**所以增加了prelen字段**。

**为什么我们去研究ziplist特别节省内存的数据结构**？ 在实际应用中，大量存储字符串的优化是需要你对底层的数据结构有一定的理解的，而ziplist在场景优化的时候也被考虑采用的首选。

##### ziplist的缺点

最后我们再看看它的一些缺点：

- ziplist也不预留内存空间, 并且在移除结点后, 也是立即缩容, 这代表**每次写操作都会进行内存分配操作**.
- 结点如果扩容, 导致结点占用的内存增长, 并且超过254字节的话, 可能会导致链式反应: 其后一个结点的entry.prevlen需要从一字节扩容至五字节. **最坏情况下, 第一个结点的扩容, 会导致整个ziplist表中的后续所有结点的entry.prevlen字段扩容**. 虽然这个内存重分配的操作依然只会发生一次, 但代码中的时间复杂度是o(N)级别, 因为链式扩容只能一步一步的计算. 但这种情况的概率十分的小, 一般情况下链式扩容能连锁反映五六次就很不幸了. 之所以说这是一个蛋疼问题, 是因为, 这样的坏场景下, 其实时间复杂度并不高: 依次计算每个entry新的空间占用, 也就是o(N), 总体占用计算出来后, 只执行一次内存重分配, 与对应的memmove操作, 就可以了

#### 快表 - QuickList

它是一种以**ziplist为结点**的**双端链表**结构. 宏观上, quicklist是一个链表, 微观上, 链表中的每个结点都是一个ziplist。

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-4.png)

#### 字典/哈希表 - Dict

hash结构定义：

```c
typedef struct dictht{
    //哈希表数组
    dictEntry **table;
    //哈希表大小
    unsigned long size;
    //哈希表大小掩码，用于计算索引值
    //总是等于 size-1
    unsigned long sizemask;
    //该哈希表已有节点的数量
    unsigned long used;
 
}dictht
```

哈希表是由数组 table 组成，table 中每个元素都是指向 dict.h/dictEntry 结构，dictEntry 结构定义如下

```c
typedef struct dictEntry{
     //键
     void *key;
     //值
     union{
          void *val;
          uint64_tu64;
          int64_ts64;
     }v;
 
     //指向下一个哈希表节点，形成链表
     struct dictEntry *next;
}dictEntry
```

key 用来保存键，val 属性用来保存值，值可以是一个指针，也可以是uint64_t整数，也可以是int64_t整数。

链地址法解决hash 冲突

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-13.png)



##### 一些要点

- **哈希算法**：Redis计算哈希值和索引值方法如下：

```bash
#1、使用字典设置的哈希函数，计算键 key 的哈希值
hash = dict->type->hashFunction(key);

#2、使用哈希表的sizemask属性和第一步得到的哈希值，计算索引值
index = hash & dict->ht[x].sizemask;
```

- **解决哈希冲突**：这个问题上面我们介绍了，方法是**链地址法**。通过字典里面的 *next 指针指向下一个具有相同索引值的哈希表节点。
- **扩容和收缩**：当哈希表保存的键值对太多或者太少时，就要通过 **rerehash(重新散列）**来对哈希表进行相应的扩展或者收缩。具体步骤：

1、如果执行扩展操作，会基于原哈希表创建一个大小等于 ht[0].used*2n 的哈希表（也就是每次扩展都是根据原哈希表已使用的空间扩大一倍创建另一个哈希表）。相反如果执行的是收缩操作，每次收缩是根据已使用空间缩小一倍创建一个新的哈希表。

2、重新利用上面的哈希算法，计算索引值，然后将键值对放到新的哈希表位置上。

3、所有键值对都迁徙完毕后，释放原哈希表的内存空间。

- **触发扩容的条件**：

1、服务器目前没有执行 BGSAVE 命令或者 BGREWRITEAOF 命令，并且负载因子大于等于1。

2、服务器目前正在执行 BGSAVE 命令或者 BGREWRITEAOF 命令，并且负载因子大于等于5。

ps：负载因子 = 哈希表已保存节点数量 / 哈希表大小。

- **渐近式 rehash**

什么叫渐进式 rehash？也就是说扩容和收缩操作不是一次性、集中式完成的，而是分多次、渐进式完成的。如果保存在Redis中的键值对只有几个几十个，那么 rehash 操作可以瞬间完成，但是如果键值对有几百万，几千万甚至几亿，那么要一次性的进行 rehash，势必会造成Redis一段时间内不能进行别的操作。所以Redis采用渐进式 rehash,这样在进行渐进式rehash期间，字典的删除查找更新等操作可能会在两个哈希表上进行，第一个哈希表没有找到，就会去第二个哈希表上进行查找。但是进行 增加操作，一定是在新的哈希表上进行的。

#### 整数集 - IntSet

整数集合（intset）是集合类型的底层实现之一，当一个集合只包含整数值元素，并且这个集合的元素数量不多时，Redis 就会使用整数集合作为集合键的底层实现。

##### intset结构

首先看源码结构

```c
typedef struct intset {
    uint32_t encoding;
    uint32_t length;
    int8_t contents[];
} intset;
```

`encoding` 表示**编码方式**，的取值有三个：INTSET_ENC_INT16, INTSET_ENC_INT32, INTSET_ENC_INT64

`length` 代表其中存储的**整数的个数**

`contents` 指向实际存储数值的连续内存区域, 就是一个数组；整数集合的每个元素都是 contents 数组的一个数组项（item），各个项在数组中按值得大小**从小到大有序排序**，且数组中**不包含任何重复项**。（虽然 intset 结构将 contents 属性声明为 int8_t 类型的数组，但实际上 contents 数组并不保存任何 int8_t 类型的值，contents 数组的真正类型取决于 encoding 属性的值）

##### 内存布局图

其内存布局如下图所示

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-8.png)

我们可以看到，content数组里面每个元素的数据类型是由encoding来决定的，那么如果原来的数据类型是int16, 当我们再插入一个int32类型的数据时怎么办呢？这就是下面要说的intset的升级。

**[#](#整数集合的升级) 整数集合的升级**

当在一个int16类型的整数集合中插入一个int32类型的值，整个集合的所有元素**都会转换成32类型**。 整个过程有三步：

- 根据新元素的类型（比如int32），扩展整数集合底层数组的空间大小，并为新元素分配空间。
- 将底层数组现有的所有元素都转换成与新元素相同的类型， 并将类型转换后的元素放置到正确的位上， 而且在放置元素的过程中， 需要继续维持底层数组的有序性质不变。
- 最后改变encoding的值，length+1。

**那么如果我们删除掉刚加入的int32类型时，会不会做一个降级操作呢**？

**不会**。主要还是减少开销的权衡。

#### 跳表 - ZSkipList

> 跳跃表结构在 Redis 中的运用场景只有一个，那就是作为**有序列表 (Zset)** 的使用。跳跃表的性能可以保证在查找，删除，添加等操作的时候在**对数期望时间内完成**，这个性能是可以和平衡树来相比较的，而且在实现方面比平衡树要优雅，这就是跳跃表的长处。跳跃表的缺点就是需要的存储空间比较大，属于利用**空间来换取时间**的数据结构。

##### 什么是跳跃表

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-10.png)



内存布局结构：

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-11.png)

**zskiplist的核心设计要点**

- **头节点**不持有任何数据, 且其level[]的长度为32
- 每个结点
  - `ele`字段，持有数据，是**sds类型**
  - `score`字段, 其标示着结点的**得分**, 结点之间凭借得分来判断先后顺序, 跳跃表中的结点按结点的得分升序排列.
  - `backward`指针, 这是原版跳跃表中所没有的. 该指针指向结点的前一个紧邻结点.
  - level字段, 用以记录所有结点(除过头节点外)；每个结点中最多持有32个zskiplistLevel结构. 实际数量在结点创建时, 按幂次定律随机生成(不超过32). 每个zskiplistLevel中有两个字段 
    - `forward`字段指向比自己得分高的某个结点(不一定是紧邻的), 并且, 若当前zskiplistLevel实例在level[]中的索引为X, 则其forward字段指向的结点, 其level[]字段的容量至少是X+1. 这也是上图中, 为什么forward指针总是画的水平的原因.
    - `span`字段代表forward字段指向的结点, 距离当前结点的距离. 紧邻的两个结点之间的距离定义为1.

- **kiplist与平衡树、哈希表的比较**

来源于：https://www.jianshu.com/p/8ac45fd01548

skiplist和各种平衡树（如AVL、红黑树等）的元素是**有序排列**的，而哈希表不是有序的。

在做范围查找的时候，平衡树比skiplist**操作要复杂**。

平衡树的插入和删除操作可能引发子树的调整，**逻辑复杂**，而skiplist的插入和删除只需要修改相邻节点的指针，操作简单又快速。

从内存占用上来说，skiplist比平衡树更**灵活**一些。

从算法实现难度上来比较，skiplist比平衡树要简单得多。

### Redis进阶 - 数据结构：redis对象与编码(底层结构)对应关系详解

#### 字符串对象

注意字符串的长度不能超过512M。

- **编码**

字符串对象的编码可以是int，raw或者embstr。

- `int 编码`：保存的是可以用 long 类型表示的整数值。
- `embstr 编码`：保存长度小于44字节的字符串（redis3.2版本之前是39字节，之后是44字节）。
- `raw 编码`：保存长度大于44字节的字符串（redis3.2版本之前是39字节，之后是44字节）。

**int 编码是用来保存整数值，而embstr是用来保存短字符串，raw编码是用来保存长字符串。**

- **内存布局**

字符串对象支持三种编码方式: RAW, INT, EMBSTR, 三种方式的内存布局分别如下:

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-21.png)

- **raw 和 embstr 的区别**

其实 embstr 编码是专门用来保存短字符串的一种优化编码，raw 和 embstr 的区别：

embstr与raw都使用redisObject和sds保存数据，区别在于，**embstr的使用只分配一次内存空间**（因此redisObject和sds是连续的），而r**aw需要分配两次内存空间**（分别为redisObject和sds分配空间）。因此与raw相比，embstr的好处在于创建时少分配一次空间，删除时少释放一次空间，以及对象的所有数据连在一起，寻找方便。而embstr的坏处也很明显，如果字符串的长度增加需要重新分配内存时，整个redisObject和sds都需要重新分配空间，因此redis中的**embstr实现为只读**。

ps：**Redis中对于浮点数类型也是作为字符串保存的，在需要的时候再将其转换成浮点数类型**。

- **编码的转换**

当 **int 编码**保存的值不再是整数，或大小超过了long的范围时，**自动转化为raw**。

对于 embstr 编码，由于 Redis 没有对其编写任何的修改程序（embstr 是只读的），在对embstr对象进行修改时，都会先转化为raw再进行修改，因此，**只要是修改embstr对象，修改后的对象一定是raw的**，无论是否达到了44个字节。

#### 列表对象

> list 列表，它是简单的字符串列表，按照插入顺序排序，你可以添加一个元素到列表的头部（左边）或者尾部（右边），它的底层实际上是个**链表结构**。

- **编码**

列表对象的编码是**quicklist**。

- **内存布局**

列表对象的内存布局如下图所示:

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-22.png)

#### 哈希对象

> 哈希对象的键是一个字符串类型，值是一个键值对集合。

- **编码**

哈希对象的编码可以是 **ziplist 或者 hashtable**；对应的底层实现有两种, 一种是**ziplist**, 一种是**dict**。

两种编码**内存布局**分别如下:

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-23.png)

如果使用ziplist，profile 存储如下：

![img](https://www.pdai.tech/images/db/redis/db-redis-x-object-9.png)

当使用 hashtable 编码时，上面命令存储如下：

![img](https://www.pdai.tech/images/db/redis/db-redis-x-object-10.png)

hashtable 编码的哈希表对象底层使用字典数据结构，哈希对象中的每个键值对都使用一个字典键值对。

- **编码转换**

和上面列表对象使用 ziplist 编码一样，当同时满足下面两个条件时，使用ziplist（压缩列表）编码：

1、列表保存元素个数小于512个

2、每个元素长度小于64字节

不能满足这两个条件的时候使用 hashtable 编码。以上两个条件也可以通过Redis配置文件`zset-max-ziplist-entries` 选项和 `zset-max-ziplist-value` 进行修改。

#### 集合对象

> 集合对象 set 是 **string 类型**（整数也会转换成string类型进行存储）的**无序集合**。注意集合和列表的区别：集合中的元素是无序的，因此不能通过索引来操作元素；集合中的元素不能有重复。

- **编码**

集合对象的编码可以是 **intset 或者 hashtable**; 底层实现有两种, 分别是intset和dict。 

显然当使用**intset**作为底层实现的数据结构时, 集合中存储的只能是数值数据, 且**必须是整数**; 

而当使用**dict**作为集合对象的底层实现时, 是将数据**全部存储于dict的键中, 值字段闲置不用**.

集合对象的内存布局如下图所示:

<img src="https://www.pdai.tech/images/db/redis/db-redis-ds-x-24.png" alt="img" style="zoom:150%;" />

- **举例说明**

```bash
SADD numbers 1 3 5
```

![img](https://www.pdai.tech/images/db/redis/db-redis-x-object-11.png)

```bash
SADD Dfruits "apple" "banana" "cherry"
```

![img](https://www.pdai.tech/images/db/redis/db-redis-x-object-12.png)

- **编码转换**

当集合同时满足以下两个条件时，使用 intset 编码：

1、集合对象中所有元素都是整数

2、集合对象所有元素数量不超过512

不能满足这两个条件的就使用 hashtable 编码。第二个条件可以通过配置文件的 `set-max-intset-entries` 进行配置。

#### 有序集合对象

> 和上面的集合对象相比，有序集合对象是**有序的**。与列表使用索引下标作为排序依据不同，有序集合为每个元素设置一个分数（score）作为排序依据。

- **编码**

有序集合的底层实现依然有两种, 一种是使用**ziplist作为底层实现**, 另外一种比较特殊, 底层使用了两种数据结构: **dict与skiplist**. 前者对应的编码值宏为**ZIPLIST**, 后者对应的编码值宏为**SKIPLIST**

使用**ziplist**来实现在序集合很容易理解, 只需要在ziplist这个数据结构的基础上做好**排序与去重**就可以了.

 使用**zskiplist**来实现有序集合也很容易理解, Redis中实现的这个跳跃表似乎天然就是为了实现有序集合对象而实现的, 那么为什么还要辅助一个dict实例呢? 我们先看来有序集合对象在这两种编码方式下的内存布局, 然后再做解释:

首先是编码为ZIPLIST时, 有序集合的内存布局如下:

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-25.png)

然后是编码为SKIPLIST时, 有序集合的内存布局如下:

![img](https://www.pdai.tech/images/db/redis/db-redis-ds-x-26.png)

说明：其实有序集合单独使用字典或跳跃表其中一种数据结构都可以实现，但是这里使用两种数据结构组合起来，原因是假如我们单独使用 字典，虽然能以 O(1) 的时间复杂度查找成员的分值，但是因为字典是以无序的方式来保存集合元素，所以每次进行范围操作的时候都要进行排序；假如我们单独使用跳跃表来实现，虽然能执行范围操作，但是查找操作有 O(1)的复杂度变为了O(logN)。因此Redis使用了两种数据结构来**共同实现有序集合**。

- **编码转换**

当有序集合对象同时满足以下两个条件时，对象使用 ziplist 编码：

1、保存的元素数量小于128；

2、保存的所有元素长度都小于64字节。

不能满足上面两个条件的使用 skiplist 编码。以上两个条件也可以通过Redis配置文件`zset-max-ziplist-entries` 选项和 `zset-max-ziplist-value` 进行修改。

### Redis进阶 - 持久化：RDB和AOF机制详解

为了防止数据丢失以及服务重启时能够恢复数据，Redis支持数据的持久化，主要分为两种方式，分别是**RDB和AOF**; 当然实际场景下还会使用这两种的混合模式。

#### RDB 持久化

> RDB 就是 Redis DataBase 的缩写，中文名为**快照/内存快照**，RDB持久化是把**当前进程数据生成快照保存到磁盘上的过程**，由于是某一时刻的快照，那么快照中的值要早于或者等于内存中的值。

##### 触发方式

> 触发rdb持久化的方式有2种，分别是**手动触发**和**自动触发**。

###### **手动触发**

> 手动触发分别对应save和bgsave命令

- **save命令**：阻塞当前Redis服务器，直到RDB过程完成为止，对于内存 比较大的实例会造成长时间**阻塞**，线上环境不建议使用
- **bgsave命令**：Redis进程执行fork操作创建子进程，RDB持久化过程由子 进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短

bgsave流程图如下所示

![img](https://www.pdai.tech/images/db/redis/redis-x-rdb-1.png)

具体流程如下：

- redis客户端执行bgsave命令或者自动触发bgsave命令；
- 主进程判断当前是否已经存在正在执行的子进程，如果存在，那么主进程直接返回；
- 如果不存在正在执行的子进程，那么就fork一个新的子进程进行持久化数据，fork过程是阻塞的，fork操作完成后主进程即可执行其他操作；
- 子进程先将数据写入到临时的rdb文件中，待快照数据写入完成后再原子替换旧的rdb文件；
- 同时发送信号给主进程，通知主进程rdb持久化完成，主进程更新相关的统计信息（info Persitence下的rdb_*相关选项）。

###### 自动触发

> 在以下4种情况时会自动触发

- redis.conf中**配置`save m n`**，即在m秒内有n次修改时，自动触发bgsave生成rdb文件；
- **主从复制**时，从节点要从主节点进行全量复制时也会触发bgsave操作，生成当时的快照发送到从节点；
- 执行**debug reload命令重新加载**redis时也会触发bgsave操作；
- 默认情况下执行**shutdown命令**时，如果没有开启aof持久化，那么也会触发bgsave操作；

##### RDB 更深入理解

**由于生产环境中我们为Redis开辟的内存区域都比较大（例如6GB），那么将内存中的数据同步到硬盘的过程可能就会持续比较长的时间，而实际情况是这段时间Redis服务一般都会收到数据写操作请求。那么如何保证数据一致性呢**？

RDB中的核心思路是**Copy-on-Write**，来保证在进行快照操作的这段时间，需要压缩写入磁盘上的数据在内存中不会发生变化。

在正常的快照操作中，一方面Redis主进程会fork一个新的快照进程专门来做这个事情，这样保证了Redis服务不会停止对客户端包括写请求在内的任何响应。

另一方面这段时间发生的数据变化会以**副本**的方式存放在另一个新的内存区域，待快照操作结束后才会同步到原来的内存区域。

举个例子：如果主线程对这些数据也都是读操作（例如图中的键值对 A），那么，主线程和 bgsave 子进程相互不影响。但是，如果主线程要修改一块数据（例如图中的键值对 C），那么，这块数据就会被复制一份，生成该数据的副本。然后，bgsave 子进程会把这个副本数据写入 RDB 文件，而在这个过程中，主线程仍然可以直接修改原来的数据。

![img](https://www.pdai.tech/images/db/redis/redis-x-aof-42.jpg)

- **在进行快照操作的这段时间，如果发生服务崩溃怎么办**？

很简单，在没有将数据全部写入到磁盘前，这次快照操作都不算成功。如果出现了服务崩溃的情况，**将以上一次完整的RDB快照文件作为恢复内存数据的参考**。也就是说，在快照操作过程中不能影响上一次的备份数据。Redis服务会在磁盘上创建一个临时文件进行数据操作，待操作成功后才会用这个临时文件替换掉上一次的备份。

- **可以每秒做一次快照吗**？

这种想法其实是错误的。虽然 bgsave 执行时不阻塞主线程，但是，**如果频繁地执行全量快照，也会带来两方面的开销**：

- 一方面，频繁将全量数据写入磁盘，会给磁盘带来很大压力，**多个快照竞争有限的磁盘带宽**，前一个快照还没有做完，后一个又开始做了，容易造成恶性循环。
- 另一方面，bgsave 子进程需要通过 fork 操作从主线程创建出来。虽然，子进程在创建后不会再阻塞主线程，但是，**fork 这个创建过程本身会阻塞主线程**，而且主线程的内存越大，阻塞时间越长。如果频繁 fork 出 bgsave 子进程，这就会频繁**阻塞主线程**了。

##### RDB优缺点

- **优点**
  - RDB文件是某个时间节点的快照，默认使用LZF压缩算法进行压缩，压缩后的文件体积远远小于内存大小，**适用于备份、全量复制**等场景；
  - Redis加载RDB文件恢复数据要**远远快于AOF方式**；
- **缺点**
  - RDB方式**实时性不够**，无法做到秒级的持久化；
  - 每次调用bgsave都需要**fork子进程**，fork子进程属于重量级操作，**频繁执行成本较高**；
  - RDB文件是二进制的，**没有可读性**，AOF文件在了解其结构的情况下可以手动修改或者补全；
  - **版本兼容RDB文件**问题；

针对RDB**不适合实时持久化**的问题，Redis提供了AOF持久化方式来解决

#### AOF 持久化

AOF日志采用写后日志，即**先写内存，后写日志**。

![img](https://www.pdai.tech/images/db/redis/redis-x-aof-41.jpg)

**为什么采用写后日志**？

Redis要求高性能，采用写日志有两方面好处：

- **避免额外的检查开销**：Redis 在向 AOF 里面记录日志的时候，并不会先去对这些命令进行语法检查。所以，**如果先记日志再执行命令的话，日志中就有可能记录了错误的命令**，Redis 在使用日志恢复数据时，就可能会出错。
- **不会阻塞当前的写操作**

但这种方式存在潜在风险：

- 如果命令执行完成，写日志之前**宕机了，会丢失数据**。
- 主线程**写磁盘压力大**，导致写盘慢，阻塞后续操作。

##### 如何实现AOF

AOF日志记录Redis的每个写命令，步骤分为：**命令追加（append）、文件写入（write）和文件同步（sync）**。

- **命令追加** 当AOF持久化功能打开了，服务器在执行完一个写命令之后，会以协议格式将被执行的写命令追加到服务器的 aof_buf 缓冲区。
- **文件写入和同步** 关于何时将 aof_buf 缓冲区的内容写入AOF文件中，Redis提供了三种写回策略：

![img](https://www.pdai.tech/images/db/redis/redis-x-aof-4.jpg)

`Always`，同步写回：每个写命令执行完，**立马同步地将日志写回磁盘**；

`Everysec`，每秒写回：每个写命令执行完，只是**先把日志写到AOF文件的内存缓冲区，每隔一秒把缓冲区中的内容写入磁盘**；

`No`，操作系统控制的写回：每个写命令执行完，只是先把日志写到AOF文件的内存缓冲区，由**操作系统决定何时将缓冲区内容写回磁盘**。

- **三种写回策略的优缺点**

上面的三种写回策略体现了一个重要原则：**trade-off，取舍，指在性能和可靠性保证之间做取舍**。

##### 深入理解AOF重写

为了解决AOF文件体积膨胀的问题，Redis**提供AOF文件重写机制**来对AOF文件进行“**瘦身**”。

- **图例解释AOF重写**

Redis通过创建一个新的AOF文件来替换现有的AOF，新旧两个AOF文件保存的数据相同，但新AOF文件没有了冗余命令。

![img](https://www.pdai.tech/images/db/redis/redis-x-aof-1.jpg)



- **AOF重写会阻塞吗**？

AOF重写过程是由后台进程bgrewriteaof来完成的。主线程fork出后台的bgrewriteaof子进程，fork会把主线程的内存拷贝一份给bgrewriteaof子进程，这里面就包含了数据库的最新数据。然后，bgrewriteaof子进程就可以在不影响主线程的情况下，逐一把拷贝的数据写成操作，记入重写日志。

所以aof在重写时，在**fork进程时是会阻塞**住主线程的。

- **AOF日志何时会重写**？

有两个配置项控制AOF重写的触发：

`auto-aof-rewrite-min-size`:表示运行AOF重写时文件的最小大小，默认为64MB。

`auto-aof-rewrite-percentage`:当前aof文件比上一次重写后aof文件的增量大小，和上一次重写后aof文件大小的比值。

- **重写日志时，有新数据写入咋整**？

重写过程总结为：“**一个拷贝，两处日志**”。在fork出子进程时的拷贝，以及在重写时，如果有新数据写入，主线程就会将命令记录到两个aof日志内存缓冲区中。如果AOF写回策略配置的是always，则直接将命令写回旧的日志文件，并且保存一份命令至AOF重写缓冲区，这些操作对新的日志文件是不存在影响的。（旧的日志文件：主线程使用的日志文件，新的日志文件：bgrewriteaof进程使用的日志文件）

而在bgrewriteaof子进程完成会日志文件的重写操作后，会提示主线程已经完成重写操作，主线程会将AOF重写缓冲中的命令追加到新的日志文件后面。这时候在高并发的情况下，AOF重写缓冲区积累可能会很大，这样就会造成阻塞，Redis后来通过Linux管道技术让aof重写期间就能同时进行回放，这样aof重写结束后只需回放少量剩余的数据即可。

最后通过修改文件名的方式，保证文件切换的原子性。

在AOF重写日志期间发生宕机的话，因为日志文件还没切换，所以恢复数据时，用的还是旧的日志文件。

**总结操作**：

- 主线程fork出子进程**重写aof日志**
- 子进程重写日志完成后，主线程**追加aof日志缓冲**
- **替换日志文件**

![img](https://www.pdai.tech/images/db/redis/redis-x-aof-2.jpg)

- **主线程fork出子进程的是如何复制内存数据的**？

fork采用操作系统提供的写时复制（copy on write）机制，就是为了避免一次性拷贝大量内存数据给子进程造成阻塞。fork子进程时，子进程时会拷贝父进程的页表，即**虚实映射关系（虚拟内存和物理内存的映射索引表），而不会拷贝物理内存**。这个拷贝会消耗大量cpu资源，并且拷贝完成前会阻塞主线程，阻塞时间取决于内存中的数据量，数据量越大，则内存页表越大。拷贝完成后，父子进程使用相同的内存地址空间。

但主进程是可以有**数据写入**的，这时候就会**拷贝物理内存中的数据**。如下图（进程1看做是主进程，进程2看做是子进程）：

![img](https://www.pdai.tech/images/db/redis/redis-x-aof-3.png)

在主进程有数据写入时，而这个数据刚好在页c中，操作系统会创建这个页面的副本（页c的副本），即拷贝当前页的物理数据，将其映射到主进程中，而子进程还是使用原来的的页c。

- **在重写日志整个过程时，主线程有哪些地方会被阻塞**？

1. fork子进程时，需要**拷贝虚拟页表**，会对主线程阻塞。
2. 主进程有**bigkey写入**时，操作系统会创建页面的副本，并拷贝原有的数据，会对主线程阻塞。
3. 子进程重写日志完成后，主进程**追加aof重写缓冲区时可能**会对主线程阻塞。

- **为什么AOF重写不复用原AOF日志**？

两方面原因：

1. 父子进程**写同一个文件会产生竞争**问题，影响父进程的性能。
2. 如果AOF重写过程中**失败**了，相当于**污染了原本的AOF文件，无法做恢复数据使用**。

#### RDB和AOF混合方式（4.0版本)

内存快照以一定的频率执行，在两次快照之间，使用 AOF 日志记录这期间的所有命令操作。

这样一来，快照不用很频繁地执行，这就避免了频繁 fork 对主线程的影响。而且，AOF 日志也只用记录两次快照间的操作，也就是说，不需要记录所有操作了，因此，就不会出现文件过大的情况了，也可以避免重写开销。

如下图所示，T1 和 T2 时刻的修改，用 AOF 日志记录，等到第二次做全量快照时，就可以清空 AOF 日志，因为此时的修改都已经记录到快照中了，恢复时就不再用日志了。

![img](https://www.pdai.tech/images/db/redis/redis-x-rdb-4.jpg)

#### 从持久化中恢复数据

其实想要从这些文件中恢复数据，只需要**重新启动Redis**即可。我们还是通过图来了解这个流程：

![img](https://www.pdai.tech/images/db/redis/redis-x-aof-5.png)

#### 性能与实践

通过上面的分析，我们都知道RDB的快照、AOF的重写都需要fork，这是一个重量级操作，会对Redis造成阻塞。因此为了不影响Redis主进程响应，我们需要尽可能降低阻塞。

- 降低fork的频率，比如可以手动来触发RDB生成快照、与AOF重写；
- 控制Redis最大使用内存，防止fork耗时过长；
- 使用更牛逼的硬件；
- 合理配置Linux的内存分配策略，避免因为物理内存不足导致fork失败。

在线上我们到底该怎么做？我提供一些自己的实践经验。

- 如果Redis中的数据并不是特别敏感或者可以通过其它方式重写生成数据，可以关闭持久化，如果丢失数据可以通过其它途径补回；
- 自己制定策略定期检查Redis的情况，然后可以手动触发备份、重写数据；
- 单机如果部署多个实例，要防止多个机器同时运行持久化、重写操作，防止出现内存、CPU、IO资源竞争，让持久化变为串行；
- 可以加入主从机器，利用一台从机器进行备份处理，其它机器正常响应客户端的命令；
- RDB持久化与AOF持久化可以同时存在，配合使用。

### Redis进阶 - 消息传递：发布订阅模式详解

#### Redis发布订阅简介

> Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

Redis 的 SUBSCRIBE 命令可以让客户端订阅任意数量的频道， 每当有新信息发送到被订阅的频道时， 信息就会被发送给所有订阅指定频道的客户端。

作为例子， 下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![img](https://www.pdai.tech/images/db/redis/db-redis-sub-1.svg)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![img](https://www.pdai.tech/images/db/redis/db-redis-sub-2.svg)

#### 发布/订阅使用

> Redis有两种发布/订阅模式：
>
> - 基于**频道(Channel)**的发布/订阅
> - 基于**模式(pattern)**的发布/订阅

##### 基于频道(Channel)的发布/订阅

"发布/订阅"模式包含两种角色，分别是发布者和订阅者。发布者可以向指定的频道(channel)发送消息; 订阅者可以订阅一个或者多个频道(channel),所有订阅此频道的订阅者都会收到此消息。

![img](https://www.pdai.tech/images/db/redis/db-redis-sub-8.png)

- **发布者发布消息**

发布者发布消息的命令是 publish,用法是 publish channel message，如向 channel1.1说一声hi

```bash
127.0.0.1:6379> publish channel:1 hi
(integer) 1
```

这样消息就发出去了。返回值表示接收这条消息的订阅者数量。发出去的消息不会被持久化，也就是有客户端订阅channel:1后只能接收到后续发布到该频道的消息，之前的就接收不到了。

- **订阅者订阅频道**

订阅频道的命令是 subscribe，可以同时订阅多个频道，用法是 subscribe channel1 [channel2 ...],例如新开一个客户端订阅上面频道:(不会收到消息，因为不会收到订阅之前就发布到该频道的消息)

```bash
127.0.0.1:6379> subscribe channel:1
Reading messages... (press Ctrl-C to quit)
1) "subscribe" // 消息类型
2) "channel:1" // 频道
3) "hi" // 消息内容
```

执行上面命令客户端会进入订阅状态，处于此状态下客户端不能使用除`subscribe`、`unsubscribe`、`psubscribe`和`punsubscribe`这四个属于"发布/订阅"之外的命令，否则会报错。

##### 基于模式(pattern)的发布/订阅

如果有某个/某些模式和这个频道匹配的话，那么所有订阅这个/这些频道的客户端也同样会收到信息。

- **用图例解释什么是基于模式的发布订阅**

下图展示了一个带有频道和模式的例子， 其中 tweet.shop.* 模式匹配了 tweet.shop.kindle 频道和 tweet.shop.ipad 频道， 并且有不同的客户端分别订阅它们三个：

![img](https://www.pdai.tech/images/db/redis/db-redis-sub-5.svg)当有信息发送到 tweet.shop.kindle 频道时， 信息除了发送给 clientX 和 clientY 之外， 还会发送给订阅 tweet.shop.* 模式的 client123 和 client256 ：

![img](https://www.pdai.tech/images/db/redis/db-redis-sub-6.svg)

- **基于模式的例子**

通配符中?表示1个占位符，*表示任意个占位符(包括0)，?*表示1个以上占位符。

publish发布

```bash
127.0.0.1:6379> publish c m1
(integer) 0
127.0.0.1:6379> publish c1 m1
(integer) 1
127.0.0.1:6379> publish c11 m1
(integer) 0
```

psubscribe订阅

```bash
127.0.0.1:6379> psubscribe c? b* d?*
```

- **注意点**

(1)使用psubscribe命令可以**重复订阅同一个频道**，如客户端执行了`psubscribe c? c?*`。这时向c1发布消息客户端会接受到两条消息，而同时publish命令的返回值是2而不是1。同样的，如果有另一个客户端执行了`subscribe c1` 和`psubscribe c?*`的话，向c1发送一条消息该客户顿也会受到两条消息(但是是两种类型:message和pmessage)，同时publish命令也返回2.

(2)punsubscribe命令可以**退订指定的规则**，用法是: `punsubscribe [pattern [pattern ...]]`,如果没有参数则会退订所有规则。

(3)**使用punsubscribe只能退订通过psubscribe命令订阅的规则，不会影响直接通过subscribe命令订阅的频道**；**同样unsubscribe命令也不会影响通过psubscribe命令订阅的规则。**另外需要注意punsubscribe命令退订某个规则时不会将其中的通配符展开，而是进行**严格的字符串匹配**，所以`punsubscribe *` 无法退订`c*`规则，而是必须使用`punsubscribe c*`才可以退订。（它们是相互独立的，后文可以看到数据结构上看也是两种实现）

#### 深入理解

##### 基于频道(Channel)的发布/订阅如何实现的？

底层是通过字典（图中的pubsub_channels）实现的，这个字典就用于保存订阅频道的信息：字典的键为正在被订阅的频道， 而字典的值则是一个链表， 链表中保存了所有订阅这个频道的客户端。

- **数据结构**

比如说，在下图展示的这个 pubsub_channels 示例中， client2 、 client5 和 client1 就订阅了 channel1 ， 而其他频道也分别被别的客户端所订阅：

![img](https://www.pdai.tech/images/db/redis/db-redis-sub-3.svg)

- **订阅**

当客户端调用 SUBSCRIBE 命令时， 程序就将客户端和要订阅的频道在 pubsub_channels 字典中关联起来。

举个例子，如果客户端 client10086 执行命令 `SUBSCRIBE channel1 channel2 channel3` ，那么前面展示的 pubsub_channels 将变成下面这个样子：

![img](https://www.pdai.tech/images/db/redis/db-redis-sub-4.svg)

- **发布**

当调用 `PUBLISH channel message` 命令， 程序首先根据 channel 定位到字典的键， 然后将信息发送给字典值链表中的所有客户端。

比如说，对于以下这个 pubsub_channels 实例， 如果某个客户端执行命令 `PUBLISH channel1 "hello moto"` ，那么 client2 、 client5 和 client1 三个客户端都将接收到 "hello moto" 信息：

- **退订**

使用 UNSUBSCRIBE 命令可以退订指定的频道， 这个命令执行的是订阅的反操作： 它从 pubsub_channels 字典的给定频道（键）中， 删除关于当前客户端的信息， 这样被退订频道的信息就不会再发送给这个客户端。

##### 基于模式(Pattern)的发布/订阅如何实现的？

底层是pubsubPattern节点的链表。

- **数据结构** redisServer.pubsub_patterns 属性是一个**链表**，链表中保存着所有和模式相关的信息：

```c
struct redisServer {
    // ...
    list *pubsub_patterns;
    // ...
};
```

链表中的每个节点都包含一个 redis.h/pubsubPattern 结构：

```c
typedef struct pubsubPattern {
    redisClient *client;
    robj *pattern;
} pubsubPattern;
```

**client 属性保存着订阅模式的客户端，而 pattern 属性则保存着被订阅的模式**。

每当调用 PSUBSCRIBE 命令订阅一个模式时， 程序就创建一个包含客户端信息和被订阅模式的 pubsubPattern 结构， 并将该结构添加到 redisServer.pubsub_patterns 链表中。

作为例子，下图展示了一个包含两个模式的 pubsub_patterns 链表， 其中 client123 和 client256 都正在订阅 tweet.shop.* 模式：

![img](https://www.pdai.tech/images/db/redis/db-redis-sub-9.svg)

- **订阅**

如果这时客户端 client10086 执行 `PSUBSCRIBE broadcast.list.*` ， 那么 pubsub_patterns 链表将被更新成这样：

![img](https://www.pdai.tech/images/db/redis/db-redis-sub-10.svg)

通过遍历整个 pubsub_patterns 链表，程序可以检查所有正在被订阅的模式，以及订阅这些模式的客户端。

- **发布**

发送信息到模式的工作也是由 PUBLISH 命令进行的, 显然就是匹配模式获得Channels，然后再把消息发给客户端。

- **退订**

使用 PUNSUBSCRIBE 命令可以退订指定的模式， 这个命令执行的是订阅模式的反操作： 程序会删除 redisServer.pubsub_patterns 链表中， 所有和被退订模式相关联的 pubsubPattern 结构， 这样客户端就不会再收到和模式相匹配的频道发来的信息。

### Redis进阶 - 事件：Redis事件机制详解

具体见：https://www.pdai.tech/md/db/nosql-redis/db-redis-x-event.html

#### 事件机制

> Redis中的事件驱动库只关注网络IO，以及定时器。

该事件库处理下面两类事件：

- **文件事件**(file event)：用于处理 Redis 服务器和客户端之间的网络IO。
- **时间事件**(time eveat)：Redis 服务器中的一些操作（比如serverCron函数）需要在给定的时间点执行，而时间事件就是处理这类定时操作的。

事件驱动库的代码主要是在`src/ae.c`中实现的，其示意图如下所示。

![img](https://www.pdai.tech/images/db/redis/db-redis-event-1.png)

**`aeEventLoop`是整个事件驱动的核心**，它管理着文件事件表和时间事件列表，不断地循环处理着就绪的文件事件和到期的时间事件。

#### 文件事件

> Redis基于**Reactor模式**开发了自己的网络事件处理器，也就是文件事件处理器。文件事件处理器使用**IO多路复用技术**（建议先看下 [Java IO多路复用详解]() ），同时监听多个套接字，并为套接字关联不同的事件处理函数。当套接字的可读或者可写事件触发时，就会调用相应的事件处理函数。

### Redis进阶 - 事务：Redis事务详解

#### 什么是Redis事务

Redis 事务的本质是**一组命令的集合**。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

总结说：**redis事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令。**

#### Redis事务相关命令和使用

> MULTI 、 EXEC 、 DISCARD 和 WATCH 是 Redis 事务相关的命令。

- MULTI ：**开启事务**，redis会将后续的命令逐个放入队列中，然后使用EXEC命令来原子化执行这个命令系列。
- EXEC：**执行事务**中的所有操作命令。
- DISCARD：**取消事务**，放弃执行事务块中的所有命令。
- WATCH：**监视**一个或多个key,如果事务在执行前，这个key(或多个key)被其他命令修改，则事务被中断，不会执行事务中的任何命令。
- UNWATCH：**取消WATCH**对所有key的监视。

##### 标准的事务执行

给k1、k2分别赋值，在事务中修改k1、k2，执行事务后，查看k1、k2值都被修改。

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> set k2 22
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) OK
127.0.0.1:6379> get k1
"11"
127.0.0.1:6379> get k2
"22"
127.0.0.1:6379>
```

##### [#](#事务取消) 事务取消

```bash
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 33
QUEUED
127.0.0.1:6379> set k2 34
QUEUED
127.0.0.1:6379> DISCARD
OK
```

##### [#](#事务出现错误的处理) 事务出现错误的处理

- **语法错误（编译器错误）**

在开启事务后，修改k1值为11，k2值为22，但k2语法错误，最终导致事务提交失败，k1、k2保留原值。

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> sets k2 22
(error) ERR unknown command `sets`, with args beginning with: `k2`, `22`, 
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379>
```

- **Redis类型错误（运行时错误）**

在开启事务后，修改k1值为11，k2值为22，但将k2的类型作为List，在运行时检测类型错误，最终导致事务提交失败，此时事务并没有回滚，而是跳过错误命令继续执行， 结果k1值改变、k2保留原值

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k1 v2
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> lpush k2 22
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> get k1
"11"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379>
```

##### CAS操作实现乐观锁

> **WATCH 命令**可以为 Redis 事务提供 check-and-set （CAS）行为。

```bash
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
```

使用上面的代码， 如果在 WATCH 执行之后， EXEC 执行之前， 有其他客户端修改了 mykey 的值， 那么当前客户端的事务就会失败。 程序需要做的， 就是不断重试这个操作， 直到没有发生碰撞为止。

- **watch是如何监视实现的呢**？

![img](https://www.pdai.tech/images/db/redis/db-redis-trans-2.png)

#### Redis事务执行步骤

通过上文命令执行，很显然Redis事务执行是三个阶段：

- **开启**：以MULTI开始一个事务
- **入队**：将多个命令入队到事务中，接到这些命令并不会立即执行，而是放到等待执行的事务队列里面
- **执行**：由EXEC命令触发事务

![img](https://www.pdai.tech/images/db/redis/db-redis-trans-1.png)

#### 为什么 Redis 不支持回滚？

以下是这种做法的优点：

- **Redis 命令只会因为错误的语法而失败**（并且这些问题不能在入队时发现），**或是命令用在了错误类型的键上面**：这也就是说，从实用性的角度来说，**失败的命令是由编程错误造成的**，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
- 因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

有种观点认为 Redis 处理事务的做法会产生 bug ， 然而需要注意的是， 在通常情况下， **回滚并不能解决编程错误带来的问题**。 

#### 如何理解Redis与事务的ACID？

一般来说，事务有四个性质称为ACID，分别是**原子性，一致性，隔离性和持久性**。

- **原子性atomicity**

首先通过上文知道 运行期的错误是不会回滚的，很多文章由此说Redis事务违背原子性的；而官方文档认为是**遵从原子性**的。

Redis官方文档给的理解是，**Redis的事务是原子性的：所有的命令，要么全部执行，要么全部不执行**。而不是完全成功。

- **一致性consistency**

redis事务可以保证命令失败的情况下得以回滚，数据能恢复到没有执行之前的样子，是**保证一致性**的，除非redis进程意外终结。

- **隔离性Isolation**

redis事务是**严格遵守隔离性**的，原因是redis是**单进程单线程**模式(v6.0之前），可以保证命令执行过程中不会被其他客户端命令打断。

但是，Redis不像其它结构化数据库有隔离级别这种设计。

- **持久性Durability**

**redis事务是不保证持久性的**，这是因为redis持久化策略中不管是RDB还是AOF都是**异步执行**的，不保证持久性是出于对性能的考虑。

#### Redis事务其它实现

- **基于Lua脚本**，Redis可以保证脚本内的命令一次性、按顺序地执行，其同时也不提供事务运行错误的回滚，执行过程中如果部分命令运行错误，剩下的命令还是会继续运行完
- **基于中间标记变量**，通过另外的标记变量来标识事务是否执行完成，读取数据时先读取该标记变量判断是否事务执行完成。但这样会需要额外写代码实现，比较繁琐

### Redis进阶 - 高可用：主从复制详解

#### 主从复制概述

> 主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave)；数据的复制是单向的，只能由主节点到从节点。

**主从复制的作用**主要包括：

- **数据冗余**：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
- **故障恢复**：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
- **负载均衡**：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
- **高可用基石**：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

主从库之间采用的是**读写分离**的方式。

- 读操作：主库、从库都可以接收；
- 写操作：首先到主库执行，然后，主库将写操作同步给从库。

![img](https://www.pdai.tech/images/db/redis/db-redis-copy-1.png)



#### 主从复制原理

- `全量（同步）复制`：比如第一次同步时
- `增量（同步）复制`：只会把主从库网络断连期间主库收到的命令，同步给从库

##### 全量复制

> 当我们启动多个 Redis 实例的时候，它们相互之间就可以通过 **replicaof**（Redis 5.0 之前使用 slaveof）命令**形成主库和从库的关系**，之后会按照**三个阶段**完成数据的第一次同步。

- **确立主从关系**

例如，现在有实例 1（ip：172.16.19.3）和实例 2（ip：172.16.19.5），我们在实例 2 上执行以下这个命令后，实例 2 就变成了实例 1 的从库，并从实例 1 上复制数据：

```bash
replicaof 172.16.19.3 6379
```

- **全量复制的三个阶段**

你可以先看一下下面这张图，有个整体感知，接下来我再具体介绍。

![img](https://www.pdai.tech/images/db/redis/db-redis-copy-2.jpg)

**第一阶段是主从库间建立连接、协商同步的过程**，主要是为全量复制做准备。在这一步，从库和主库建立起连接，并告诉主库即将进行同步，主库确认回复后，主从库间就可以开始同步了。

具体来说，从库给主库发送 psync 命令，表示要进行数据同步，主库根据这个命令的参数来启动复制。psync 命令包含了主库的 runID 和复制进度 offset 两个参数。runID，是每个 Redis 实例启动时都会自动生成的一个随机 ID，用来唯一标记这个实例。当从库和主库第一次复制时，因为不知道主库的 runID，所以将 runID 设为“？”。offset，此时设为 -1，表示第一次复制。主库收到 psync 命令后，会用 FULLRESYNC 响应命令带上两个参数：主库 runID 和主库目前的复制进度 offset，返回给从库。从库收到响应后，会记录下这两个参数。这里有个地方需要注意，**FULLRESYNC 响应表示第一次复制采用的全量复制**，也就是说，主库会把当前所有的数据都复制给从库。

**第二阶段，主库将所有数据同步给从库**。从库收到数据后，在本地完成数据加载。这个过程依赖于内存快照生成的 RDB 文件。

具体来说，主库执行 bgsave 命令，生成 RDB 文件，接着将文件发给从库。从库接收到 RDB 文件后，会先清空当前数据库，然后加载 RDB 文件。这是因为从库在通过 replicaof 命令开始和主库同步前，可能保存了其他数据。为了避免之前数据的影响，从库需要先把当前数据库清空。在主库将数据同步给从库的过程中，主库不会被阻塞，仍然可以正常接收请求。否则，Redis 的服务就被中断了。但是，这些请求中的写操作并没有记录到刚刚生成的 RDB 文件中。为了保证主从库的数据一致性，主库会在内存中用专门的 replication buffer，记录 RDB 文件生成后收到的所有写操作。

**第三个阶段，主库会把第二阶段执行过程中新收到的写命令，再发送给从库**。具体的操作是，当主库完成 RDB 文件发送后，就会把此时 replication buffer 中的修改操作发给从库，从库再重新执行这些操作。这样一来，主从库就实现同步了。

##### 增量复制

> 在 Redis 2.8 版本引入了增量复制。

- **为什么会设计增量复制**？

如果主从库在命令传播时出现了网络闪断，那么，从库就会和主库重新进行一次全量复制，**开销非常大**。从 Redis 2.8 开始，网络断了之后，主从库会采用增量复制的方式继续同步。

- **增量复制的流程**

你可以先看一下下面这张图，有个整体感知，接下来我再具体介绍。

![img](https://www.pdai.tech/images/db/redis/db-redis-copy-3.jpg)

先看两个概念： `replication buffer` 和 `repl_backlog_buffer`

`repl_backlog_buffer`：它是为了从库断开之后，**如何找到主从差异数据**而设计的环形缓冲区，从而避免全量复制带来的性能开销。如果从库断开时间太久，repl_backlog_buffer环形缓冲区被主库的写命令覆盖了，那么从库连上主库后只能乖乖地进行一次全量复制，所以**repl_backlog_buffer配置尽量大一些，可以降低主从断开后全量复制的概率**。而在repl_backlog_buffer中找主从差异的数据后，如何发给从库呢？这就用到了replication buffer。

`replication buffer`：Redis和客户端通信也好，和从库通信也好，Redis都需要给分配一个 内存buffer进行数据交互，客户端是一个client，从库也是一个client，我们每个client连上Redis后，Redis都会分配一个client buffer，所有数据交互都是通过这个buffer进行的：Redis先把数据写到这个buffer中，然后再把buffer中的数据发到client socket中再通过网络发送出去，这样就完成了数据交互。**所以主从在增量同步时，从库作为一个client，也会分配一个buffer，只不过这个buffer专门用来传播用户的写命令到从库，保证主从数据一致**，我们通常把它叫做replication buffer。

- **如果在网络断开期间，repl_backlog_size环形缓冲区写满之后，从库是会丢失掉那部分被覆盖掉的数据，还是直接进行全量复制呢**？

对于这个问题来说，有两个关键点：

1. **一个从库如果和主库断连时间过长**，造成它在主库repl_backlog_buffer的slave_repl_offset位置上的**数据已经被覆盖掉了**，此时从库和主库间将进行**全量复制**。
2. 每个从库会记录自己的slave_repl_offset，每个从库的复制进度也不一定相同。在和主库重连进行恢复时，从库会通过psync命令把自己记录的slave_repl_offset发给主库，**主库会根据从库各自的复制进度**，来决定这个从库可以进行增量复制，还是全量复制。

#### 更深入理解

##### 当主服务器不进行持久化时复制的安全性

**为什么不持久化的主服务器自动重启非常危险呢**？为了更好的理解这个问题，看下面这个失败的例子，其中主服务器和从服务器中数据库都被删除了。

- 我们设置节点A为主服务器，关闭持久化，节点B和C从节点A复制数据。
- 这时出现了一个崩溃，但Redis具有自动重启系统，重启了进程，因为关闭了持久化，节点重启后只有一个空的数据集。
- 节点B和C从节点A进行复制，**现在节点A是空的，所以节点B和C上的复制数据也会被删除**。
- 当在高可用系统中使用Redis Sentinel，关闭了主服务器的持久化，并且允许自动重启，这种情况是很危险的。比如主服务器可能在很短的时间就完成了重启，以至于Sentinel都无法检测到这次失败，那么上面说的这种失败的情况就发生了。

如果数据比较重要，并且在使用主从复制时关闭了主服务器持久化功能的场景中，都应该禁止实例自动重启。

##### 为什么主从全量复制使用RDB而不使用AOF？

1、**RDB文件内容是经过压缩的二进制数据（不同数据类型数据做了针对性优化），文件很小**。而AOF文件记录的是每一次写操作的命令，写操作越多文件会变得很大，其中还包括很多对同一个key的多次冗余操作。在主从全量数据同步时，传输RDB文件可以尽量降低对主库机器网络带宽的消耗，从库在加载RDB文件时，一是文件小，读取整个文件的速度会很快，二是因为RDB文件存储的都是二进制数据，从库直接按照RDB协议解析还原数据即可，速度会非常快，而AOF需要依次重放每个写命令，这个过程会经历冗长的处理逻辑，恢复速度相比RDB会慢得多，所以使用RDB进行主从全量复制的成本最低。

2、假设要使用AOF做全量复制，意味着必须打开AOF功能，**打开AOF就要选择文件刷盘的策略，选择不当会严重影响Redis性能**。而RDB只有在需要定时备份和主从全量复制数据时才会触发生成一次快照。而在很多丢失数据不敏感的业务场景，其实是不需要开启AOF的。

##### 为什么还有无磁盘复制模式？

Redis 默认是磁盘复制，但是**如果使用比较低速的磁盘，这种操作会给主服务器带来较大的压力**。Redis从2.8.18版本开始尝试支持无磁盘的复制。使用这种设置时，**子进程直接将RDB通过网络发送给从服务器，不使用磁盘作为中间存储**。

**无磁盘复制模式**：master创建一个新进程直接dump RDB到slave的socket，不经过主进程，不经过硬盘。**适用于disk较慢，并且网络较快的时候**。

使用`repl-diskless-sync`配置参数来启动无磁盘复制。

使用`repl-diskless-sync-delay` 参数来配置传输开始的延迟时间；master等待一个`repl-diskless-sync-delay`的秒数，如果没slave来的话，就直接传，后来的得排队等了; 否则就可以一起传。

##### 为什么还会有从库的从库的设计？

通过分析主从库间第一次数据同步的过程，你可以看到，一次全量复制中，对于主库来说，需要完成两个耗时的操作：**生成 RDB 文件和传输 RDB 文件**。

如果从库数量很多，而且都要和主库进行全量复制的话，就会导致**主库忙于 fork 子进程生成 RDB 文件**，进行数据全量复制。fork 这个操作会阻塞主线程处理正常请求，从而**导致主库响应应用程序的请求速度变慢**。此外，**传输 RDB 文件也会占用主库的网络带宽**，同样会给主库的资源使用带来压力。那么，有没有好的解决方法可以分担主库压力呢？

其实是有的，这就是**“主 - 从 - 从”模式**。

在刚才介绍的主从库模式中，所有的从库都是和主库连接，所有的全量复制也都是和主库进行的。现在，我们可以通过“主 - 从 - 从”模式**将主库生成 RDB 和传输 RDB 的压力，以级联的方式分散到从库上**。

简单来说，我们在部署主从集群的时候，可以手动选择一个从库（比如选择内存资源配置较高的从库），用于级联其他的从库。然后，我们可以再选择一些从库（例如三分之一的从库），在这些从库上执行如下命令，让它们和刚才所选的从库，建立起主从关系。

```bash
replicaof 所选从库的IP 6379
```

这样一来，这些从库就会知道，在进行同步时，不用再和主库进行交互了，只要和级联的从库进行写操作同步就行了，这就可以减轻主库上的压力，如下图所示：

![img](https://www.pdai.tech/images/db/redis/db-redis-copy-4.jpg)

##### 读写分离及其中的问题

- **延迟与不一致问题**

前面已经讲到，由于主从复制的命令传播是异步的，延迟与数据的不一致不可避免。如果应用对数据不一致的接受程度程度较低，可能的优化措施包括：**优化主从节点之间的网络环境（如在同机房部署）；监控主从节点延迟（通过offset）判断，如果从节点延迟过大，通知应用不再通过该从节点读取数据；使用集群同时扩展写负载和读负载等。**

- **数据过期问题**

在单机版Redis中，存在两种删除策略：

- `惰性删除`：服务器不会主动删除数据，只有当客户端**查询**某个数据时，服务器**判断**该数据是否过期，如果过期则删除。
- `定期删除`：服务器执行**定时任务**删除过期数据，但是考虑到内存和CPU的折中（删除会释放内存，但是频繁的删除操作对CPU不友好），该删除的频率和执行时间都受到了限制。

在主从复制场景下，为了主从节点的数据一致性，从节点不会主动删除数据，而是由主节点控制从节点中过期数据的删除。由于主节点的惰性删除和定期删除策略，都不能保证主节点及时对过期数据执行删除操作，因此，当客户端通过Redis从节点读取数据时，很容易读取到已经过期的数据。

**Redis 3.2中，从节点在读取数据时，增加了对数据是否过期的判断**：如果该数据已过期，则不返回给客户端；将Redis升级到3.2可以解决数据过期问题。

- **故障切换问题**

在没有使用哨兵的读写分离场景下，应用针对读和写分别连接不同的Redis节点；当主节点或从节点出现问题而发生更改时，需要及时修改应用程序读写Redis数据的连接；连接的切换可以手动进行，或者自己写监控程序进行切换，但前者响应慢、容易出错，后者实现复杂，成本都不算低。

### Redis进阶 - 高可用：哨兵机制（Redis Sentinel）详解

#### 哨兵机制（Redis Sentinel）

> Redis Sentinel，即Redis哨兵，在Redis 2.8版本开始引入。哨兵的核心功能是**主节点的自动故障转移**。

下图是一个典型的哨兵集群监控的逻辑图：

![img](https://www.pdai.tech/images/db/redis/db-redis-sen-1.png)

哨兵实现了什么功能呢？下面是Redis官方文档的描述：

- **监控（Monitoring）**：哨兵会不断地检查主节点和从节点是否运作正常。
- **自动故障转移（Automatic failover）**：当主节点不能正常工作时，哨兵会开始自动故障转移操作，它会将失效主节点的其中一个从节点升级为新的主节点，并让其他从节点改为复制新的主节点。
- **配置提供者（Configuration provider）**：客户端在初始化时，通过连接哨兵来获得当前Redis服务的主节点地址。
- **通知（Notification）**：哨兵可以将故障转移的结果发送给客户端。

其中，监控和自动故障转移功能，使得哨兵可以及时发现主节点故障并完成转移；而配置提供者和通知功能，则需要在与客户端的交互中才能体现。

#### 哨兵集群的组建

> 上图中哨兵集群是如何组建的呢？哨兵实例之间可以相互发现，要归功于 Redis 提供的 pub/sub 机制，也就是**发布 / 订阅机制**。

在主从集群中，主库上有一个名为`__sentinel__:hello`的频道，不同哨兵就是通过它来相互发现，实现互相通信的。在下图中，哨兵 1 把自己的 IP（172.16.19.3）和端口（26579）发布到`__sentinel__:hello`频道上，哨兵 2 和 3 订阅了该频道。那么此时，哨兵 2 和 3 就可以从这个频道直接获取哨兵 1 的 IP 地址和端口号。然后，哨兵 2、3 可以和哨兵 1 建立网络连接。

![img](https://www.pdai.tech/images/db/redis/db-redis-sen-6.jpg)

#### 哨兵监控Redis库

这是由**哨兵向主库发送 INFO 命令**来完成的。就像下图所示，哨兵 2 给主库发送 INFO 命令，**主库接受到这个命令后，就会把从库列表返回给哨兵**。接着，哨兵就可以根据从库列表中的连接信息，和每个从库**建立连接**，并在这个连接上持续地对从库进行监控。哨兵 1 和 3 可以通过相同的方法和从库建立连接。

![img](https://www.pdai.tech/images/db/redis/db-redis-sen-7.jpg)

#### 主库下线的判定

首先要理解两个概念：**主观下线**和**客观下线**

- **主观下线**：**任何**一个哨兵都是可以监控探测，并**作出**Redis节点下线的**判断**；
- **客观下线**：有哨兵集群**共同决定**Redis节点是否下线；

当某个哨兵（如下图中的哨兵2）判断主库“主观下线”后，就会给其他哨兵发送 `is-master-down-by-addr` 命令。接着，其他哨兵会根据自己和主库的连接情况，做出 Y 或 N 的响应，Y 相当于赞成票，N 相当于反对票。

![img](https://www.pdai.tech/images/db/redis/db-redis-sen-2.jpg)

如果赞成票数（这里是2）是大于等于哨兵配置文件中的 `quorum` 配置项（比如这里如果是quorum=2）, 则可以判定**主库客观下线**了。

#### 哨兵集群的选举

- **哨兵的选举机制是什么样的**？

哨兵的选举机制其实很简单，就是一个Raft选举算法： **选举的票数大于等于num(sentinels)/2+1时，将成为领导者，如果没有超过，继续选举**

- 任何一个想成为 Leader 的哨兵，要满足两个条件：
  - 第一，拿到半数以上的赞成票；
  - 第二，拿到的票数同时还需要大于等于哨兵配置文件中的 quorum 值。

#### 新主库的选出

- 过滤掉不健康的（下线或断线），没有回复过哨兵ping响应的从节点
- 选择`salve-priority`从节点优先级最高（redis.conf）的
- 选择复制偏移量最大，只复制最完整的从节点

![img](https://www.pdai.tech/images/db/redis/db-redis-sen-3.jpg)

#### 故障的转移

**故障转移流程如下**：

![img](https://www.pdai.tech/images/db/redis/db-redis-sen-4.png)

- 将slave-1脱离原从节点（PS: 5.0 中应该是`replicaof no one`)，升级主节点，
- 将从节点slave-2指向新的主节点
- 通知客户端主节点已更换
- 将原主节点（oldMaster）变成从节点，指向新的主节点

**转移之后**

![img](https://www.pdai.tech/images/db/redis/db-redis-sen-5.png)

### Redis进阶 - 高可拓展：分片技术（Redis Cluster）详解

前面两篇文章，[主从复制]()和[哨兵机制]()保障了高可用，就读写分离而言虽然slave节点扩展了主从的读并发能力，但是**写能力**和**存储能力**是无法进行扩展，就只能是master节点能够承载的上限。如果面对海量数据那么必然需要构建master（主节点分片)之间的集群，同时必然需要吸收高可用（主从复制和哨兵机制）能力，即每个master分片节点还需要有slave节点，这是分布式系统中典型的纵向扩展（集群的分片技术）的体现；

![img](https://pic4.zhimg.com/v2-5dfa3ff3667282699e4509243fa63ebf_r.jpg)

具体见：https://www.pdai.tech/md/db/nosql-redis/db-redis-x-cluster.html





Cluster模式是Redis的一种高级集群模式，它通过数据分片和分布式存储实现了负载均衡和高可用性。在Cluster模式下，Redis将所有的键值对数据分散在多个节点上。每个节点负责一部分数据，称为槽位。通过对数据的分片，Cluster模式可以突破单节点的内存限制，实现更大规模的数据存储。

**数据分片与槽位**

Redis Cluster将数据分为16384个槽位，每个节点负责管理一部分槽位。当客户端向Redis Cluster发送请求时，Cluster会根据键的哈希值将请求路由到相应的节点。具体来说，Redis Cluster使用CRC16算法计算键的哈希值，然后对16384取模，得到槽位编号。

**优点**：

1. 数据分片，实现大规模数据存储。
2. 负载均衡，提高系统性能。
3. 自动故障转移，提高高可用性。

**缺点**：

1. 配置和管理较复杂。
2. 一些复杂的多键操作可能受到限制。

### Redis进阶 - 缓存问题：一致性, 穿击, 穿透, 雪崩, 污染等

#### 为什么要理解Redis缓存问题

在高并发的业务场景下，数据库大多数情况都是用户并发访问最薄弱的环节。所以，就需要使用redis做一个缓冲操作，让请求先访问到redis，而不是直接访问Mysql等数据库。这样可以大大缓解数据库的压力。

当缓存库出现时，必须要考虑如下问题：

- 缓存穿透
- 缓存击穿
- 缓存雪崩
- 缓存污染（或者满了）
- 缓存和数据库一致性

#### 缓存穿透

- **问题来源**

缓存穿透是指**缓存和数据库中都没有的数据**，而用户不断发起请求。由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。

在流量大时，可能DB就挂掉了，要是有人利用不存在的key频繁攻击我们的应用，这就是漏洞。

如发起为id为“-1”的数据或id为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。

- **解决方案**

1. 接口层**增加校验**，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
2. 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为**key-null**，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击
3. **布隆过滤器**。bloomfilter就类似于一个hash set，用于快速判某个元素是否存在于集合中，其典型的应用场景就是快速判断一个key是否存在于某容器，不存在就直接返回。布隆过滤器的关键就在于hash算法和容器大小，

#### 缓存击穿

- **问题来源**

缓存击穿是指**缓存中没有但数据库中有的数据**（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。

- **解决方案**

1、设置热点数据永远不过期。

2、接口限流与熔断，降级。重要的接口一定要做好限流策略，防止用户恶意刷接口，同时要降级准备，当接口中的某些 服务 不可用时候，进行熔断，失败快速返回机制。

3、加互斥锁

#### 缓存雪崩

- **问题来源**

缓存雪崩是指缓存中**数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机**。和缓存击穿不同的是，**缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了**，很多数据都查不到从而查数据库。

- **解决方案**

1. 缓存数据的**过期时间设置随机**，防止同一时间大量数据过期现象发生。
2. 如果缓存数据库是**分布式部署**，将热点数据均匀分布在不同的缓存数据库中。
3. 设置**热点数据永远不过期**。

#### 缓存污染（或满了）

缓存污染问题说的是缓存中一些只会被**访问一次或者几次的的数据**，被访问完后，再也不会被访问到，但这部分数据依然留存在缓存中，消耗缓存空间。

缓存污染会随着数据的持续增加而逐渐显露，随着服务的不断运行，缓存中会存在大量的永远不会再次被访问的数据。缓存空间是有限的，如果**缓存空间满了**，再往缓存里写数据时就会有额外开销，影响Redis性能。这部分额外开销主要是指写的时候判断淘汰策略，**根据淘汰策略去选择要淘汰的数据，然后进行删除操作**。

**最大缓存设置多大**：**建议把缓存容量设置为总数据量的 15% 到 30%，兼顾访问性能和内存空间开销**

##### 缓存淘汰策略

Redis共支持八种淘汰策略，分别是noeviction、volatile-random、volatile-ttl、volatile-lru、volatile-lfu、allkeys-lru、allkeys-random 和 allkeys-lfu 策略。

**怎么理解呢**？主要看分三类看：

- 不淘汰 
  - **noeviction** （v4.0后默认的）：该策略是Redis的默认策略。
- 对设置了过期时间的数据中进行淘汰 
  - 随机：**volatile-random**：在设置了过期时间的键值对中，进行随机删除。因为是**随机删除**，无法把不再访问的数据筛选出来，所以可能依然会存在缓存污染现象，**无法解决缓存污染问题**。
  - ttl：**volatile-ttl**：这种算法判断淘汰数据时参考的指标比随机删除时多进行一步过期时间的排序。Redis在筛选需删除的数据时，**越早过期的数据越优先被选择**。
  - lru：**volatile-lru**：LRU算法：LRU 算法的全称是 Least Recently Used，按照**最近最少使用**的原则来筛选数据。这种模式下会使用 LRU 算法筛选设置了过期时间的键值对。（**只淘汰最近最少访问、并设置了过期时间的 key**）
  - lfu：**volatile-lfu**：会使用 LFU 算法选择设置了过期时间的键值对。当 LFU 策略筛选数据时，Redis 会在候选集合中，根据数据 lru 字段的后 8bit 选择**访问次数最少**的数据进行淘汰。当访问次数相同时，再根据 lru 字段的前 16bit 值大小，选择**访问时间最久远**的数据进行淘汰。（**只淘汰访问频率最低、并设置了过期时间 key**）
- 全部数据进行淘汰 
  - 随机：**allkeys-random**
  - lru：**allkeys-lru**
  - lfu：**allkeys-lfu**

**LFU 算法**：LFU 缓存策略是在 LRU 策略基础上，为每个数据增加了一个计数器，来统计这个数据的访问次数。当使用 LFU 策略筛选淘汰数据时，首先会根据数据的访问次数进行筛选，把访问次数最低的数据淘汰出缓存。如果两个数据的访问次数相同，LFU 策略再比较这两个数据的访问时效性，把距离上一次访问时间更久的数据淘汰出缓存。

#### 数据库和缓存一致性

![img](https://www.pdai.tech/images/db/redis/db-redis-cache-1.jpg)

读取缓存步骤一般没有什么问题，但是一旦涉及到数据更新：数据库和缓存更新，就容易出现缓存(Redis)和数据库（MySQL）间的数据一致性问题。

**不管是先写MySQL数据库，再删除Redis缓存；还是先删除缓存，再写库，都有可能出现数据不一致的情况**。举一个例子：

1.如果删除了缓存Redis，还没有来得及写库MySQL，另一个线程就来读取，发现缓存为空，则去数据库中读取数据写入缓存，此时缓存中为脏数据。

2.如果先写了库，在删除缓存前，写库的线程宕机了，没有删除掉缓存，则也会出现数据不一致情况。

因为写和读是并发的，没法保证顺序,就会出现缓存和数据库的数据不一致的问题。

##### 4种相关模式

更新缓存的的Design Pattern有四种：Cache aside, Read through, Write through, Write behind caching; 

**节选最最常用的Cache Aside Pattern, 总结来说就是**

- **读的时候**，先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回响应。
- **更新的时候**，先更新数据库，然后再删除缓存。

其具体逻辑如下：

- **失效**：应用程序先从cache取数据，没有得到，则从数据库中取数据，成功后，放到缓存中。
- **命中**：应用程序从cache中取数据，取到后返回。
- **更新**：先把数据存到数据库中，成功后，再让缓存失效。

一个是查询操作，一个是更新操作的并发，首先，没有了删除cache数据的操作了，而是先更新了数据库中的数据，此时，缓存依然有效，所以，并发的查询操作拿的是没有更新的数据，但是，更新操作马上让缓存的失效了，后续的查询操作再把数据从数据库中拉出来。而不会像文章开头的那个逻辑产生的问题，后续的查询操作一直都在取老的数据。

##### 方案：队列 + 重试机制

![img](https://www.pdai.tech/images/db/redis/db-redis-cache-4.png)

流程如下所示

- 更新数据库数据；
- 缓存因为种种问题删除失败
- 将需要删除的key发送至消息队列
- 自己消费消息，获得需要删除的key
- 继续重试删除操作，直到成功

然而，该方案有一个缺点，对业务线代码造成大量的侵入。于是有了方案二，在方案二中，启动一个订阅程序去订阅数据库的binlog，获得需要操作的数据。在应用程序中，另起一段程序，获得这个订阅程序传来的信息，进行删除缓存操作。

##### 方案：异步更新缓存(基于订阅binlog的同步机制)

![img](https://www.pdai.tech/images/db/redis/db-redis-cache-5.png)

1. **技术整体思路**：

MySQL binlog增量订阅消费+消息队列+增量数据更新到redis

1）读Redis：热数据基本都在Redis

2）写MySQL: 增删改都是操作MySQL

3）更新Redis数据：MySQ的数据操作binlog，来更新到Redis

1. **Redis更新**

1）**数据操作**主要分为两大块：

- 一个是全量(将全部数据一次写入到redis)
- 一个是增量（实时更新）

这里说的是增量,指的是mysql的update、insert、delate变更数据。

2）**读取binlog后分析 ，利用消息队列,推送更新各台的redis缓存数据**。

这样一旦MySQL中产生了新的写入、更新、删除等操作，就可以把binlog相关的消息推送至Redis，Redis再根据binlog中的记录，对Redis进行更新。

当然，这里的消息推送工具你也可以采用别的第三方：kafka、rabbitMQ等来实现推送更新Redis。

### Redis进阶 - 运维监控：Redis的监控详解

#### Redis自身状态及命令

**状态信息 - info**

Redis提供的INFO命令不仅能够查看实时的吞吐量(ops/sec)，还能看到一些有用的运行时信息。

- info查看所有状态信息

```bash
[root@redis_test_vm ~]# redis-cli -h 127.0.0.1
127.0.0.1:6379> auth xxxxx
OK
127.0.0.1:6379> info
```

- 查看某个section的信息

```bash
127.0.0.1:6379> info memory
```

**监控执行命令 - monitor**

monitor用来监视服务端收到的命令。

```bash
127.0.0.1:6379> monitor
```

**监控延迟**

- **监控延迟 - latency**

```bash
[root@redis_test_vm ~]# redis-cli --latency -h 127.0.0.1
min: 0, max: 1, avg: 0.21
```

如果我们故意用DEBUG命令制造延迟，就能看到一些输出上的变化：

```bash
[root@redis_test_vm ~]# redis-cli -h 127.0.0.1
127.0.0.1:6379> debug sleep 2
OK
(2.00s)
```

- **服务端 - 内部机制**

服务端内部的延迟监控稍微麻烦一些，因为延迟记录的默认阈值是0。尽管空间和时间耗费很小，Redis为了高性能还是默认关闭了它。所以首先我们要开启它，设置一个合理的阈值，例如下面命令中设置的100ms：

```bash
127.0.0.1:6379> CONFIG SET latency-monitor-threshold 100
OK
```

下面就用LATENCY的各种子命令来查看延迟记录：

- `LATEST`：四列分别表示事件名、最近延迟的Unix时间戳、最近的延迟、最大延迟。
- `HISTORY`：延迟的时间序列。可用来产生图形化显示或报表。
- `GRAPH`：以图形化的方式显示。最下面以竖行显示的是指延迟在多久以前发生。
- `RESET`：清除延迟记录。

```bash
127.0.0.1:6379> latency latest
1) 1) "command"
   1) (integer) 1616058778
   2) (integer) 500
   3) (integer) 2000

127.0.0.1:6379> latency history command
1) 1) (integer) 1616058773
   2) (integer) 2000
2) 1) (integer) 1616058776
   2) (integer) 150
3) 1) (integer) 1616058778
   2) (integer) 500

127.0.0.1:6379> latency graph command
command - high 2000 ms, low 150 ms (all time high 2000 ms)
--------------------------------------------------------------------------------
#  
|  
|  
|_#

666
mmm
```

还有一个子命令DOCTOR，它能列出一些指导建议，例如开启慢日志进一步追查问题原因，查看是否有大对象被踢出或过期，以及操作系统的配置建议等。

```bash
127.0.0.1:6379> latency doctor
```

- **度量延迟Baseline - intrinsic-latency**

延迟中的一部分是来自环境的，比如操作系统内核、虚拟化环境等等。Redis提供了让我们度量这一部分延迟基线(Baseline)的方法：

```bash
[root@redis_test_vm ~]# redis-cli --intrinsic-latency 100 -h 127.0.0.1
```

#### Redis可视化监控工具

> 在谈Redis可视化监控工具时，要分清工具到底是**仅仅指标的可视化**，还是**可以融入监控体系**(比如包含可视化，监控，报警等; 这是生产环境长期监控生态的基础）

- **只能可视化指标不能监控**： `redis-stat`、`RedisLive`、`redmon` 等工具
- **用于生产环境**： 基于`redis_exporter`以及`grafana`可以做到指标可视化，持久化，监控以及报警等

**redis-stat**

> [redis-stat在新窗口打开](https://github.com/junegunn/redis-stat)是一个比较有名的redis指标可视化的监控工具，采用ruby开发，基于redis的info和monitor命令来统计，不影响redis性能。

它提供了命令行彩色控制台展示模式

![img](https://www.pdai.tech/images/db/redis/redis-stat-1.png)

和web模式

![img](https://www.pdai.tech/images/db/redis/redis-stat-2.png)

**RedisLive**

> [RedisLive在新窗口打开](https://github.com/nkrode/RedisLive)是采用python开发的redis的可视化及查询分析工具

docker运行

```bash
docker run --name redis-live -p 8888:8888 -d snakeliwei/redislive
```

运行实例图

![img](https://www.pdai.tech/images/db/redis/redis-live-1.png)

**redmon**

> [redmon在新窗口打开](https://github.com/steelThread/redmon)提供了cli、admin的web界面，同时也能够实时监控redis

docker运行

```bash
docker run -p 4567:4567 -d  vieux/redmon -r redis://192.168.99.100:6379
```

![img](https://www.pdai.tech/images/db/redis/redis-redmon-1.png)

**redis_exporter**

> [redis_exporter在新窗口打开](https://github.com/oliver006/redis_exporter)为Prometheus提供了redis指标的exporter，支持Redis 2.x, 3.x, 4.x, 5.x, and 6.x，配合Prometheus以及grafana的Prometheus Redis插件，可以在grafana进行可视化及监控

运行实例图

![img](https://www.pdai.tech/images/db/redis/redis-export-1.png)

#### Redis监控体系

##### 什么样的场景会谈到redis监控体系？

一个大型系统引入了Redis作为缓存中间件，具体描述如下：

- 部署架构采用Redis-Cluster模式;
- 后台应用系统有几十个，应用实例数超过二百个;
- 所有应用系统共用一套缓存集群;
- 集群节点数几十个，加上容灾备用环境，节点数量翻倍;
- 集群节点内存配置较高。

![img](https://www.pdai.tech/images/db/redis/db-redis-monitor-2.png)

**问题描述**

系统刚开始关于Redis的一切都很正常，随着应用系统接入越来越多，应用系统子模块接入也越来越多，开始出现一些问题，应用系统有感知，集群服务端也有感知，如下描述：

- 集群节点崩溃;
- 集群节点假死;
- 某些后端应用访问集群响应特别慢。

其实问题的根源都是架构运维层面的欠缺，对于Redis集群服务端的运行监控其实很好做，上文也介绍了很多直接的命令方式，但只能看到服务端的一些常用指标信息，无法深入分析，治标不治本，对于Redis的内部运行一无所知，特别是对于业务应用如何使用Redis集群一无所知：

- Redis集群使用的热度问题?
- 哪些应用占用的Redis内存资源多?
- 哪些应用占用Redis访问数最高?
- 哪些应用使用Redis类型不合理?
- 应用系统模块使用Redis资源分布怎么样?
- 应用使用Redis集群的热点问题?

##### 构建Redis监控体系具备什么价值？

Redis监控告警的价值对每个角色都不同，重要的几个方面：

- redis故障快速通知，定位故障点；
- 分析redis故障的Root cause（任何一个故障和性能问题，其根本“诱因”往往只有一个，称为这个故障的Root cause。）
- redis容量规划和性能管理
- redis硬件资源利用率和成本

##### 监控体系化包含哪些维度？

监控的目的不仅仅是监控Redis本身，而是为了更好的使用Redis。传统的监控一般比较单一化，没有系统化，但对于Redis来说，个人认为至少包括：**一是服务端，二是应用端，三是服务端与应用端联合分析**。

**服务端**

服务端首先是操作系统层面，常用的CPU、内存、网络IO，磁盘IO，服务端运行的进程信息等；

Redis运行进程信息，包括服务端运行信息、客户端连接数、内存消耗、持久化信息 、键值数量、主从同步、命令统计、集群信息等；

Redis运行日志，日志中会记录一些重要的操作进程，如运行持久化时，可以有效帮助分析崩溃假死的程序。

**[#](#应用端) 应用端**

应用端、获取应用端使用Redis的一些行为，具体哪些应用哪些模块最占用 Redis资源、哪些应用哪些模块最消耗Redis资源、哪些应用哪些模块用法有误等。

**[#](#联合分析) 联合分析**

联合分析结合服务端的运行与应用端使用的行为，如：一些造成服务端突然阻塞的原因，可能是应用端设置了一个很大的缓存键值，或者使用的键值列表，数据量超大造成阻塞。

##### 具体的监控指标有哪些呢？

redis监控的数据采集，数据采集1分钟一次，分为下面几个方面：

- 服务器系统数据采集
- Redis Server数据采集
- Redis响应时间数据采集
- Redis监控Screen

##### 常用哪些成熟方案呢？

指标**采集**，即采集redis提供的metric指标，所以方案中有时候会出现Agent，比如metricBeat；

监控的数据**持久化**，只有将监控数据放到数据库，才能对比和长期监控；

**时序化**，因为很多场景都会按照时间序列去展示 - 所以通常是用时序库或者针对时间列优化；

**可视化**，比如常见的kibana，grafana等

按条件**报警**，因为运维不可能盯着看，只有引入报警配置，触发报警条件时即发出报警，比如短息提醒等；基于不同报警方式，平台可以提供插件支持等；

**ELK Stack**

> 经典的ELK

- 采集agent: metricBeat
- 收集管道：logstash
- DB: elasticSearch
- view和告警: kibana及插件

**[#](#fluent-prometheus-grafana) fluent + Prometheus + Grafana**

> 推荐使用这种

- 采集指标来源: redis-export
- 收集管道：fluentd
- DB: Prometheus
- view和告警: Grafana及插件

### Redis进阶 - 性能调优：Redis性能调优详解

具体见：https://pdai.tech/md/db/nosql-redis/db-redis-x-performance.html

![img](https://www.pdai.tech/images/db/redis/redis-performance-8.png)

从资源使用角度来看，包含的知识点如下：

- **CPU 相关**：使用复杂度过高命令、数据的持久化，都与耗费过多的 CPU 资源有关
- **内存相关**：bigkey 内存的申请和释放、数据过期、数据淘汰、碎片整理、内存大页、内存写时复制都与内存息息相关
- **磁盘相关**：数据持久化、AOF 刷盘策略，也会受到磁盘的影响
- **网络相关**：短连接、实例流量过载、网络流量过载，也会降低 Redis 性能
- **计算机系统**：CPU 结构、内存分配，都属于最基础的计算机系统知识
- **操作系统**：写时复制、内存大页、Swap、CPU 绑定，都属于操作系统层面的知识



## MYSQL

主要有 CHAR 和 VARCHAR 两种类型，一种是定长的，一种是变长的。

VARCHAR 这种变长类型能够节省空间，因为只需要存储必要的内容。

#### **InnoDB和MyISAM比较**

- 事务: InnoDB 是事务型的，可以使用 Commit 和 Rollback 语句。
- 并发: MyISAM 只支持表级锁，而 InnoDB 还支持行级锁。
- 外键: InnoDB 支持外键。
- 备份: InnoDB 支持在线热备份。
- 崩溃恢复: MyISAM 崩溃后发生损坏的概率比 InnoDB 高很多，而且恢复的速度也更慢。
- 其它特性: MyISAM 支持压缩表和空间数据索引。

#### **优化数据访问**

**[#](#_1-减少请求的数据量) 1. 减少请求的数据量**

- 只返回必要的列: 最好不要使用 SELECT * 语句。
- 只返回必要的行: 使用 LIMIT 语句来限制返回的数据。
- 缓存重复查询的数据: 使用缓存可以避免在数据库中进行查询，特别在要查询的数据经常被重复查询时，缓存带来的查询性能提升将会是非常明显的。

**[#](#_2-减少服务器端扫描的行数) 2. 减少服务器端扫描的行数**

最有效的方式是使用索引来覆盖查询。

**切分大查询，分解大查询（单表查询）**

#### 建索引的几大原则

1. **最左前缀匹配原则**，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(`>`、`<`、`between`、`like`)就停止匹配，比如`a = 1 and b = 2 and c > 3 and d = 4` 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。
2. **=和in可以乱序**，比如`a = 1 and b = 2 and c = 3` 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式。
3. **尽量选择区分度高的列作为索引**，区分度的公式是`count(distinct col)/count(*)`，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，那可能有人会问，这个比例有什么经验值吗？使用场景不同，这个值也很难确定，一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录。
4. **索引列不能参与计算，保持列“干净”**，比如`from_unixtime(create_time) = ’2014-05-29’`就不能使用到索引，原因很简单，b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。所以语句应该写成create_time = unix_timestamp(’2014-05-29’)。
5. **尽量的扩展索引，不要新建索引**。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

#### 数据库的三大范式包括：

- 第一范式(1NF)。确保每个数据列都是不可再分的原子值，即每个单元格中只包含一个值。（属性不可分。）
- 第二范式(2NF)。建立在第一范式的基础上，要求表中的**每个非主键列完全依赖于主键列**，而不是依赖于其他非主键列。（每个非主属性完全函数依赖于键码。）
- 第三范式(3NF)。进一步建立在第二范式的基础上，要求表中的**每个非主键列之间不应该存在传递依赖关系**，也就是说非主键列之间不应该相互依赖，而是直接依赖于主键列。（非主属性不传递函数依赖于键码。）

#### ACID

##### **[#](#_1-原子性-atomicity) 1. 原子性(Atomicity)**

事务被视为不可分割的最小单元，事务的所有操作要么全部提交成功，要么全部失败回滚。

回滚可以用日志来实现，日志记录着事务所执行的修改操作，在回滚时反向执行这些修改操作即可。

##### **[#](#_2-一致性-consistency) 2. 一致性(Consistency)**

数据库在事务执行前后都保持一致性状态。在一致性状态下，所有事务对一个数据的读取结果都是相同的。

##### **[#](#_3-隔离性-isolation) 3. 隔离性(Isolation)**

一个事务所做的修改在最终提交以前，对其它事务是不可见的。

##### **[#](#_4-持久性-durability) 4. 持久性(Durability)**

一旦事务提交，则其所做的修改将会永远保存到数据库中。即使系统发生崩溃，事务执行的结果也不能丢失。

可以通过数据库备份和恢复来实现，在系统发生崩溃时，使用备份的数据库进行数据恢复。

#### 并发一致性问题

##### 丢失修改

T1 和 T2 两个事务都对一个数据进行修改，T1 先修改，T2 随后修改，T2 的修改覆盖了 T1 的修改。

##### 读脏数据

T1 修改一个数据，T2 随后读取这个数据。如果 T1 撤销了这次修改，那么 T2 读取的数据是脏数据。

##### 不可重复读

T2 读取一个数据，T1 对该数据做了修改。如果 T2 再次读取这个数据，此时读取的结果和第一次读取的结果不同。

##### 幻影读

T1 读取某个范围的数据，T2 在这个范围内插入新的数据，T1 再次读取这个范围的数据，此时读取的结果和和第一次读取的结果不同。

#### 隔离级别

##### 未提交读(READ UNCOMMITTED)

事务中的修改，即使没有提交，对其它事务也是可见的。

##### [#](#提交读-read-committed) 提交读(READ COMMITTED)

一个事务只能读取已经提交的事务所做的修改。换句话说，一个事务所做的修改在提交之前对其它事务是不可见的。

##### [#](#可重复读-repeatable-read) 可重复读(REPEATABLE READ)

保证在同一个事务中多次读取同样数据的结果是一样的。

##### [#](#可串行化-serializable) 可串行化(SERIALIZABLE)

强制事务串行执行。

------

| 隔离级别 | 脏读 | 不可重复读 | 幻影读 |
| :------: | :--: | :--------: | :----: |
| 未提交读 |  √   |     √      |   √    |
|  提交读  |  ×   |     √      |   √    |
| 可重复读 |  ×   |     ×      |   √    |
| 可串行化 |  ×   |     ×      |   ×    |

#### SQL语句优化

**负向查询不能使用索引**

```sql
select name from user where id not in (1,3,4);
```

**前导模糊查询不能使用索引**

如:

```sql
select name from user where name like '%zhangsan'
```

**数据区分不明显的不建议创建索引**

如 user 表中的性别字段，可以明显区分的才建议创建索引，如身份证等字段。

**字段的默认值不要为 null**

这样会带来和预期不一致的查询结果。

**在字段上进行计算不能命中索引**

```sql
select name from user where FROM_UNIXTIME(create_time) < CURDATE();
```

应该修改为:

```sql
select name from user where create_time < FROM_UNIXTIME(CURDATE());
```

**最左前缀问题**

如果给 user 表中的 username pwd 字段创建了复合索引那么使用以下SQL 都是可以命中索引:

```sql
select username from user where username='zhangsan' and pwd ='axsedf1sd'

select username from user where pwd ='axsedf1sd' and username='zhangsan'

select username from user where username='zhangsan'
```

但是使用

```sql
select username from user where pwd ='axsedf1sd'
```

是不能命中索引的。

**如果明确知道只有一条记录返回**

```sql
select name from user where username='zhangsan' limit 1
```

可以提高效率，可以让数据库停止游标移动。

**[#](#不要让数据库帮我们做强制类型转换) 不要让数据库帮我们做强制类型转换**

```sql
select name from user where telno=18722222222
```

这样虽然可以查出数据，但是会导致全表扫描。

需要修改为

```text
select name from user where telno='18722222222'
```

**[#](#如果需要进行-join-的字段两表的字段类型要相同) 如果需要进行 join 的字段两表的字段类型要相同**

不然也不会命中索引。

## Tomcat

#### 什么是Servlet？

> 所谓Servlet，其实就是Sun为了让Java能实现动态可交互的网页，从而进入Web编程领域而制定的一套标准！

一个Servlet主要做下面三件事情：

- 创建并填充Request对象，包括：URI、参数、method、请求头信息、请求体信息等
- 创建Response对象
- 执行业务逻辑，将结果通过Response的输出流输出到客户端

**Servlet没有main方法，所以，如果要执行，则需要在一个容器里面才能执行，这个容器就是为了支持Servlet的功能而存在，Tomcat其实就是一个Servlet容器的实现**。

#### 架构图

![img](https://www.pdai.tech/images/tomcat/tomcat-x-design-2-1.jpeg)

##### 从组件的角度看

- **Server**: 表示服务器，它提供了一种优雅的方式来启动和停止整个系统，不必单独启停连接器和容器；它是Tomcat构成的顶级构成元素，所有一切均包含在Server中；

- **Service**: 表示服务，Server可以运行多个服务。比如一个Tomcat里面可运行订单服务、支付服务、用户服务等等；Server的实现类StandardServer可以包含一个到多个Services, Service的实现类为StandardService调用了容器(Container)接口，其实是调用了Servlet Engine(引擎)，而且StandardService类中也指明了该Service归属的Server;

- **Container**: 表示容器，可以看做Servlet容器；引擎(Engine)、主机(Host)、上下文(Context)和Wraper均继承自Container接口，所以它们都是容器。

  - Engine -- 引擎
  - Host -- 主机
  - Context -- 上下文
  - Wrapper -- 包装器

- **Connector**: 表示连接器, **它将Service和Container连接起来**，首先它需要注册到一个Service，它的作用就是把来自客户端的请求转发到Container(容器)，这就是它为什么称作连接器, 它支持的协议如下：

  - 支持AJP协议
  - 支持Http协议
  - 支持Https协议

- **Service内部**还有各种支撑组件，下面简单罗列一下这些组件

  - Manager -- 管理器，用于管理会话Session
  - Logger -- 日志器，用于管理日志
  - Loader -- 加载器，和类加载有关，只会开放给Context所使用
  - Pipeline -- 管道组件，配合Valve实现过滤器功能
  - Valve -- 阀门组件，配合Pipeline实现过滤器功能
  - Realm -- 认证授权组件

  ## 高并发

  **高并发实现的三板斧：缓存，限流和降级**。

  在高并发环境下，**服务之间的依赖关系导致调用失败，解决的方式通常是: 限流->熔断->隔离->降级, 其目的是防止雪崩效应**。

  #### 基本的容错模式

  > 常见的容错模式主要包含以下几种方式

  - 主动超时：Http请求主动设置一个超时时间，超时就直接返回，不会造成服务堆积
  - 限流：限制最大并发数
  - 熔断：当错误数超过阈值时快速失败，不调用后端服务，同时隔一定时间放几个请求去重试后端服务是否能正常调用，如果成功则关闭熔断状态，失败则继续快速失败，直接返回。（此处有个重试，重试就是弹性恢复的能力）
  - 隔离：把每个依赖或调用的服务都隔离开来，防止级联失败引起整体服务不可用
  - 降级：服务失败或异常后，返回指定的默认信息

  ![img](https://www.pdai.tech/images/arch/arch-x-reduce-2.png)

  

  ### 限流

  限流其实就是：当**高并发或者瞬时高并发**时，为了保证系统的稳定性、可用性，系统以牺牲部分请求为代价或者延迟处理请求为代价，保证系统整体服务可用。

  #### 算法

  令牌桶(Token Bucket)、漏桶(leaky bucket)和计数器算法是最常用的三种限流的算法

  #### 方案一：令牌桶方式(Token Bucket)

  > 令牌桶算法是网络流量整形（Traffic Shaping）和速率限制（Rate Limiting）中最常使用的一种算法。先有一个木桶，系统按照固定速度，往桶里加入Token，如果桶已经满了就不再添加。当有请求到来时，会各自拿走一个Token，取到Token 才能继续进行请求处理，没有Token 就拒绝服务。

  ![img](https://www.pdai.tech/images/arch/arch-xianliu-1.png)

  这里如果一段时间没有请求时，桶内就会积累一些Token，下次一旦有突发流量，只要Token足够，也能一次处理，所以令牌桶算法的特点是*允许突发流量*。

  #### 方案二：漏桶方式

  水(请求)先进入到漏桶里,漏桶以一定的速度出水(接口有响应速率),当水流入速度过大会直接溢出（访问频率超过接口响应速率),然后就拒绝请求,可以看出漏桶算法能强行限制数据的传输速率。

  ![img](https://www.pdai.tech/images/arch/arch-xianliu-2.png)

  可见这里有两个变量,一个是桶的大小,支持流量突发增多时可以存多少的水(burst),另一个是水桶漏洞的大小(rate)。

  因为漏桶的漏出速率是固定的参数,所以,即使网络中不存在资源冲突(没有发生拥塞),漏桶算法也不能使流突发(burst)到端口速率.因此,漏桶算法对于存在突发特性的流量来说缺乏效率.

  #### 令牌桶和漏桶对比

  - 令牌桶是按照固定速率往桶中添加令牌，请求是否被处理需要看桶中令牌是否足够，当令牌数减为零时则拒绝新的请求；
  - 漏桶则是按照常量固定速率流出请求，流入请求速率任意，当流入的请求数累积到漏桶容量时，则新流入的请求被拒绝；
  - 令牌桶限制的是平均流入速率（允许突发请求，只要有令牌就可以处理，支持一次拿3个令牌，4个令牌），并允许一定程度突发流量；
  - 漏桶限制的是常量流出速率（即流出速率是一个固定常量值，比如都是1的速率流出，而不能一次是1，下次又是2），从而平滑突发流入速率；
  - 令牌桶允许一定程度的突发，而漏桶主要目的是平滑流入速率；
  - 两个算法实现可以一样，但是方向是相反的，对于相同的参数得到的限流效果是一样的。

  #### 方案三：计数器方式

  计数器限流算法也是比较常用的，主要用来限制总并发数，比如数据库连接池大小、线程池大小、程序访问并发数等都是使用计数器算法。也是最简单粗暴的算法。

  ##### [#](#采用atomicinteger) 采用AtomicInteger

  使用AomicInteger来进行统计当前正在并发执行的次数，如果超过域值就简单粗暴的直接响应给用户，说明系统繁忙，请稍后再试或其它跟业务相关的信息。

  弊端：使用 AomicInteger 简单粗暴超过域值就拒绝请求，可能只是瞬时的请求量高，也会拒绝请求。

  ##### [#](#采用令牌semaphore) 采用令牌Semaphore

  使用Semaphore信号量来控制并发执行的次数，如果超过域值信号量，则进入阻塞队列中排队等待获取信号量进行执行。如果阻塞队列中排队的请求过多超出系统处理能力，则可以在拒绝请求。

  相对Atomic优点：如果是瞬时的高并发，可以使请求在阻塞队列中排队，而不是马上拒绝请求，从而达到一个流量削峰的目的。

  ##### [#](#采用threadpoolexecutor-java线程池) 采用ThreadPoolExecutor java线程池

  固定线程池大小,超出固定先线程池和最大的线程数,拒绝线程请求;

  ### 服务降级

  > 由于爆炸性的流量冲击，对一些服务进行有策略的放弃，以此缓解系统压力，保证目前主要业务的正常运行。它主要是针对非正常情况下的应急服务措施：当此时一些业务服务无法执行时，给出一个统一的返回结果。

  #### 降级方式

  - 延迟服务：比如发表了评论，重要服务，比如在文章中显示正常，但是延迟给用户增加积分，只是放到一个缓存中，等服务平稳之后再执行。
  - 在粒度范围内关闭服务（片段降级或服务功能降级）：比如关闭相关文章的推荐，直接关闭推荐区
  - 页面异步请求降级：比如商品详情页上有推荐信息/配送至等异步加载的请求，如果这些信息响应慢或者后端服务有问题，可以进行降级；
  - 页面跳转（页面降级）：比如可以有相关文章推荐，但是更多的页面则直接跳转到某一个地址
  - 写降级：比如秒杀抢购，我们可以只进行Cache的更新，然后异步同步扣减库存到DB，保证最终一致性即可，此时可以将DB降级为Cache。
  - 读降级：比如多级缓存模式，如果后端服务有问题，可以降级为只读缓存，这种方式适用于对读一致性要求不高的场景。

  #### 自动降级分类

  - 超时降级：主要配置好超时时间和超时重试次数和机制，并使用异步机制探测回复情况
  - 失败次数降级：主要是一些不稳定的api，当失败调用次数达到一定阀值自动降级，同样要使用异步机制探测回复情况
  - 故障降级：比如要调用的远程服务挂掉了（网络故障、DNS故障、http服务返回错误的状态码、rpc服务抛出异常），则可以直接降级。降级后的处理方案有：默认值（比如库存服务挂了，返回默认现货）、兜底数据（比如广告挂了，返回提前准备好的一些静态页面）、缓存（之前暂存的一些缓存数据）
  - 限流降级: 当我们去秒杀或者抢购一些限购商品时，此时可能会因为访问量太大而导致系统崩溃，此时开发者会使用限流来进行限制访问量，当达到限流阀值，后续请求会被降级；降级后的处理方案可以是：排队页面（将用户导流到排队页面等一会重试）、无货（直接告知用户没货了）、错误页（如活动太火爆了，稍后重试）

  ### 服务熔断

  > 熔断这一概念来源于电子工程中的断路器（Circuit Breaker）。在互联网系统中，当下游服务因访问压力过大而响应变慢或失败，上游服务为了保护系统整体的可用性，可以暂时切断对下游服务的调用。

  #### 服务熔断与服务降级比较

  服务熔断对服务提供了proxy，防止服务不可能时，出现串联故障（cascading failure），导致雪崩效应。

  **服务熔断一般是某个服务（下游服务）故障引起，而服务降级一般是从整体负荷考虑**。

  - 共性： 
    - 目的 -> 都是从可用性、可靠性出发，提高系统的容错能力。
    - 最终表现->使某一些应用不可达或不可用，来保证整体系统稳定。
    - 粒度 -> 一般都是服务级别，但也有细粒度的层面：如做到数据持久层、只许查询不许增删改等。
    - 自治 -> 对其自治性要求很高。都要求具有较高的自动处理机制。
  - 区别： 
    - 触发原因 -> 服务熔断通常是下级服务故障引起；服务降级通常为整体系统而考虑。
    - 管理目标 -> 熔断是每个微服务都需要的，是一个框架级的处理；而服务降级一般是关注业务，对业务进行考虑，抓住业务的层级，从而决定在哪一层上进行处理：比如在IO层，业务逻辑层，还是在外围进行处理。
    - 实现方式 -> 代码实现中的差异。

  #### Hystrix特性

  Spring Cloud Netflix Hystrix就是**隔离措施**的一种实现,可以设置在某种超时或者失败情形下断开依赖调用或者返回指定逻辑,从而提高分布式系统的稳定性.

  - **请求熔断**： 当Hystrix Command请求后端服务失败数量超过一定比例(默认50%), 断路器会切换到开路状态(Open). 这时所有请求会直接失败而不会发送到后端服务. 断路器保持在开路状态一段时间后(默认5秒), 自动切换到半开路状态(HALF-OPEN).

  这时会判断下一次请求的返回情况, 如果请求成功, 断路器切回闭路状态(CLOSED), 否则重新切换到开路状态(OPEN). Hystrix的断路器就像我们家庭电路中的保险丝, 一旦后端服务不可用, 断路器会直接切断请求链, 避免发送大量无效请求影响系统吞吐量, 并且断路器有自我检测并恢复的能力.

  - **服务降级**：Fallback相当于是降级操作. 对于查询操作, 我们可以实现一个fallback方法, 当请求后端服务出现异常的时候, 可以使用fallback方法返回的值. fallback方法的返回值一般是设置的默认值或者来自缓存.告知后面的请求服务不可用了，不要再来了。
  - **依赖隔离**(采用舱壁模式，Docker就是舱壁模式的一种)：在Hystrix中, 主要通过线程池来实现资源隔离. 通常在使用的时候我们会根据调用的远程服务划分出多个线程池.比如说，一个服务调用两外两个服务，你如果调用两个服务都用一个线程池，那么如果一个服务卡在哪里，资源没被释放

  后面的请求又来了，导致后面的请求都卡在哪里等待，导致你依赖的A服务把你卡在哪里，耗尽了资源，也导致了你另外一个B服务也不可用了。这时如果依赖隔离，某一个服务调用A B两个服务，如果这时我有100个线程可用，我给A服务分配50个，给B服务分配50个，这样就算A服务挂了，我的B服务依然可以用。

  - **请求缓存**：比如一个请求过来请求我userId=1的数据，你后面的请求也过来请求同样的数据，这时我不会继续走原来的那条请求链路了，而是把第一次请求缓存过了，把第一次的请求结果返回给后面的请求。
  - **请求合并**：我依赖于某一个服务，我要调用N次，比如说查数据库的时候，我发了N条请求发了N条SQL然后拿到一堆结果，这时候我们可以把多个请求合并成一个请求，发送一个查询多条数据的SQL的请求，这样我们只需查询一次数据库，提升了效率

  

  **容灾技术**是系统的高可用性技术的一个组成部分，容灾系统更加强调处理外界环境对系统的影响，特别是灾难性事件对整个IT节点的影响，提供节点级别的系统恢复功能。

  **故障转移（failover）**，即当活动的服务或应用意外终止时，快速启用**冗余**或备用的服务器、系统、硬件或者网络接替它们工作。故障恢复是在计划内或计划外中断解决后**切换回主站点**的过程。

  

  **什么是分布式系统？**

  **一群独立计算机集合共同对外提供服务，但是对于系统的用户来说，就像是一台计算机在提供服务一样**。

  

  ### 	Zookeeper

  https://www.cnblogs.com/monkey-xuan/p/15884952.html

  ### Zookeeper分布式锁

  https://zhuanlan.zhihu.com/p/639756647

  ### 分布式系统-全局唯一ID

  

  https://pdai.tech/md/arch/arch-z-id.html

  https://zhuanlan.zhihu.com/p/455695292

  

  ### RPC实现原理

  https://www.jianshu.com/p/78f72ccf0377

  ### Zookeeper原理

  https://www.cnblogs.com/monkey-xuan/p/15884952.html

  ### 消息队列MQ

  https://mikechen.cc/7319.html

  ### RabbitMQ详解

  https://blog.csdn.net/weixin_42039228/article/details/123493937

  ### Kafka原理

  https://cloud.tencent.com/developer/article/2156633

  https://zhuanlan.zhihu.com/p/543312272

  ### RocketMQ详解

  https://mikechen.cc/25317.html

  ### ActiveMQ详解

  https://blog.csdn.net/weixin_37626416/article/details/80930568

  

  

  

