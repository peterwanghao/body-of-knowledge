## 1. Lambda表达式

Lambda 表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性。可以把Lambda表达式理解为简洁地表示可传递的匿名函数的一种方式：它没有名称，但它有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表。

### Lambda的基本语法是 

(parameters) -> { statements; }

java中，引入了一个新的操作符“->”，该操作符在很多资料中，称为箭头操作符，或者lambda操作符；箭头操作符将lambda分成了两个部分：

  **1.  左侧：lambda表达式的参数列表**
  **2.  右侧：lambda表达式中所需要执行的功能，即lambda函数体**

以下是lambda表达式的重要特征:

- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：**如果<u>主体只有一个表达式返回值则编译器会自动返回值</u>，大括号需要指定表达式返回了一个数值。

```java
public class LamdbaDemo {
	public static void main(String args[]) {
		LamdbaDemo tester = new LamdbaDemo();

		// 类型声明
		MathOperation addition = (int a, int b) -> a + b;

		// 不用类型声明
		MathOperation subtraction = (a, b) -> a - b;

		// 大括号中的返回语句
		MathOperation multiplication = (int a, int b) -> {
			return a * b;
		};

		// 没有大括号及返回语句
		MathOperation division = (int a, int b) -> a / b;

		System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
		System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
		System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
		System.out.println("10 / 5 = " + tester.operate(10, 5, division));

		// 不用括号
		GreetingService greetService1 = message -> System.out.println("Hello " + message);

		// 用括号
		GreetingService greetService2 = (message) -> System.out.println("Hello " + message);

		greetService1.sayMessage("Runoob");
		greetService2.sayMessage("Google");
	}

	interface MathOperation {
		int operation(int a, int b);
	}

	interface GreetingService {
		void sayMessage(String message);
	}

	private int operate(int a, int b, MathOperation mathOperation) {
		return mathOperation.operation(a, b);
	}
}
```

### 变量作用域

lambda 表达式只能引用标记了 final 的外层局部变量，这就是说不能在 lambda 内部**修改定义在域外的局部变量**，否则会编译错误。

也可以直接在 lambda 表达式中访问外层的局部变量，lambda 表达式的局部变量可以不用声明为 final，但是必须不可被后面的代码修改（即隐性的具有 final 的语义）。

在 Lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量。

```java
public class LamdbaDemo {
	final static String salutation = "Hello! ";
	
	public static void main(String args[]) {
		LamdbaDemo tester = new LamdbaDemo();

		GreetingService greetService1 = message -> System.out.println(salutation + message);

		String salutation = "Hello ";
		GreetingService greetService2 = (message) -> System.out.println(salutation + message);

		greetService1.sayMessage("Runoob");
		greetService2.sayMessage("Google");
	}

	interface GreetingService {
		void sayMessage(String message);
	}

}
```

**lamabd表达式中，需要有函数式接口的支持；**



## 2. 函数式接口

**函数式接口就是只定义一个抽象方法的接口。**

可以使用@FunctionalInterface注解修饰，对该接口做检查；如果接口里，有多个抽象方法，使用该注解，会有语法错误。

用函数式接口可以干什么呢？Lambda表达式允许你直接以内联的形式为函数式接口的抽象方法提供实现，并把整个表达式作为函数式接口的实例（具体说来，是函数式接口一个具体实现的实例）。你用匿名内部类也可以完成同样的事情，只不过比较笨拙：需要提供一个实现，然后再直接内联将它实例化。

### 方法引用

方法引用可以被看做仅仅调用特定方法的Lambda的一种快捷写法。

方法引用使用“::”操作符，“::”前是引用的方法存在的类名，后面就是引用的方法名称。编译器自动会判断如何使用引用，如何传递参数等。

方法的引用的语法，主要有四类：

1.指向静态方法的方法引用，例如Integer的parseInt方法 ，可以写成Integer::parseInt

```
     类：：静态方法名
```

2.指向任意类型实例方法的方法引用，例如String的length方法，写成String::length；
```
    类：：实例方法名
```

3.指向现有对象的实例方法的方法引用
```
    对象：：实例方法名
```

4.构造器的引用：对于一个现有构造函数，你可以利用它的名称和关键字new来创建它的一个引

```
    ClassName::new
```

```java
public class MethodReference {
	public static void main(String args[]) {
		/*************** 方法的引用 ****************/
        // 类::静态方法名
		Comparator<Integer> aa = (x, y) -> Integer.compare(x, y); //Lambda表达式
        System.out.println(aa.compare(3, 2));
        Comparator<Integer> bb = Integer::compare; //方法引用
        System.out.println(bb.compare(3, 2));
        
 
        Comparator<Integer> cc = (x, y) -> x.compareTo(y);
        System.out.println(cc.compare(3, 2));
        Comparator<Integer> dd = Integer::compareTo;
        System.out.println(dd.compare(3, 2));
        
        // 类::实例方法名
        BiPredicate<String, String> bp = (x, y) -> x.equals(y); //Lambda表达式
        System.out.println(bp.test("a", "b"));
        BiPredicate<String, String> bp1 = String::equals; //方法引用
        System.out.println(bp1.test("a", "b"));
 
        // 对象::实例方法名
        Consumer<String> con1 = x -> System.out.println(x); //Lambda表达式
        con1.accept("abc");
        Consumer<String> con = System.out::println; //方法引用
        con.accept("abc");
        
        Emp emp = new Emp("上海", "xiaoMIng", 18);
        Consumer<String> setter = x -> emp.setAddress(x);
        setter.accept("ddd");
        Supplier<String> supper = () -> emp.getAddress();
        System.out.println(supper.get());
        Consumer<String> setter1 = emp::setAddress;
        setter1.accept("eee");
        Supplier<String> supper1 = emp::getAddress;
        System.out.println(supper1.get());
        
        /*************** 构造器的引用 ****************/
        // 无参构造函数，创建实例
        Supplier<Emp> supper2 = () -> new Emp();
        Supplier<Emp> supper3 = Emp::new;
        Emp emp1 = supper3.get();
        Consumer<String> setter2 = emp1::setAddress;
        setter2.accept("eee");
        System.out.println(emp1);
        // 一个参数
        Function<String, Emp> fun = address -> new Emp(address);
        Function<String, Emp> fun1 = Emp::new;
        System.out.println(fun1.apply("beijing"));
        // 两个参数
        BiFunction<String, Integer, Emp> bFun = (name, age) -> new Emp(name, age);
        BiFunction<String, Integer, Emp> bFun1 = Emp::new;
        System.out.println(bFun1.apply("xiaohong", 18));
	}
	
}
```

### 内置函数接口

Java API中已经有了几个函数式接口，比如Comparable、Runnable和Callable。Java 8的库设计师帮你在**java.util.function**包中引入了几个新的函数式接口。包括Predicate <T>、Function<T,R>、Supplier<T>、Consumer<T>和BinaryOperator<T>等。

- Predicate：断言型接口，或者判断型的接口。定义了 boolean test(T t) 抽象方法，需要表示一个涉及类型T的布尔表达式时可以使用。接受一个参数T，返回boolean
- BiPredicate：定义了 boolean test(T t, U u) 抽象方法，接收 t 和 u参数，返回比较的接口，需要两个对象做比较可以使用
- Consumer：消费型接口。定义了 void accept(T t) 抽象方法，需要访问某对象并对其进行某些操作时可以使用。接受一个参数T，不返回结果
- BiConsumer：定义了 void accept(T t, U u) 抽象方法，我把它看成Consumer的升级版
- Function：函数型接口。定义了 R apply(T t) 抽象方法，它接受一个 泛型T的对象，并返回一个泛型R的对象。如果需要将接收对象转换成其它对象可以使用。接受一个参数T，返回结果R
- BiFunction：定义 R apply(T t, U u) 抽象方法，接收t和u参数，返回R对象，如果需要两个对象中的某些值来组装成另一个对象，可以使用。我把它看成Function的升级版
- Supplier：供给型接口。定义了 T get() 抽象方法，不接收参数返回Lambda表达式的值。不接受任何参数，返回结果T
- UnaryOperator：继承自Function，接受一个参数T，返回相同类型T的结果
- BinaryOperator：继承自BiFunction，接受两个相同类型T的参数，返回相同类型T的结果



Java类型要么是引用类型（比如Byte、Integer、Object、List），要么是原始类型（比如int、double、byte、char）。但是泛型（比如Consumer<T>中的T）只能绑定到引用类型。这是由泛型内部的实现方式造成的。因此，在Java里有一个将原始类型转换为对应的引用类型的机制。这个机制叫作装箱（boxing）。相反的操作，也就是将引用类型转换为对应的原始类型，叫作拆箱（unboxing）。Java还有一个自动装箱机制来帮助程序员执行这一任务：装箱和拆箱操作是自动完成的。但这在性能方面是要付出代价的。装箱后的值本质上就是把原始类型包裹起来，并保存在堆里。因此，装箱后的值需要更多的内存，并需要额外的内存搜索来获取被包裹的原始值。

为了避免装箱操作，对Predicate<T>和Function<T, R>等通用函数式接口的**原始类型特化**：IntPredicate、IntToLongFunction等。



JDK 1.8之前已有的函数式接口:
    java.lang.Runnable
    java.util.concurrent.Callable
    java.security.PrivilegedAction
    java.util.Comparator
    java.io.FileFilter
    java.nio.file.PathMatcher
    java.lang.reflect.InvocationHandler
    java.beans.PropertyChangeListener
    java.awt.event.ActionListener
    javax.swing.event.ChangeListener



## 3. 流Stream

流(Stream)是Java API的新成员，它允许你以声明性方式**处理数据集合**（通过查询语句来表达，而不是临时编写一个实现）。就现在来说，你可以把它们看成**遍历数据集的高级迭代器**。此外，流还可以透明地并行处理，你无需写任何多线程代码了！

### 创建流的几种方式

1. Arrays.stream，我们可以通过Arrays的静态方法，传入一个泛型数组，创建一个流
2. Stream.of，我们可以通过Stream的静态方法，传入一个泛型数组，或者多个参数，创建一个流，这个静态方法，也是调用了Arrays的stream静态方法
3. Collection.stream,可以用过集合的接口的默认方法，创建一个流；使用这个方法，包括继承Collection的接口，如：Set，List，Map，SortedSet 等等
4. Stream.iterate，是Stream接口下的一个静态方法，从名字也可以看出，这个静态方法，是以迭代器的形式，创建一个数据流
5. Stream.generate，也是stream中的一个静态方法

```java
String[] dd = { "a", "b", "c" };
// 方法1
Arrays.stream(dd).forEach(System.out::print);// abc
System.out.println();

// 方法2
Stream.of(dd).forEach(System.out::print);// abc
System.out.println();

// 方法3
Arrays.asList(dd).stream().forEach(System.out::print);// abc
System.out.println();

// 方法4
Stream.iterate(0, x -> x + 1).limit(10).forEach(System.out::print);// 0123456789
System.out.println();

// 方法5
Stream.generate(() -> "x").limit(10).forEach(System.out::print);// xxxxxxxxxx
System.out.println();
```



### 流的操作

java.util.stream.Stream中的Stream接口定义了许多操作。它们可以分为两大类。可以连接起来的流操作称为**中间操作**，关闭流的操作称为**终端操作**。

- 返回类型为接口本身的Stream<T>方法为中间操作
- 返回其他对象类型的称为终端操作

### 中间操作

- filter  该操作会接受一个谓词（一个返回boolean的函数）作为参数，并返回一个包括所有符合谓词的元素的流。
- map,mapToInt,mapToLong,mapToDouble   调用这个函数后，可以改变返回的类型
- flatMap,flatMapToInt,flatMapToLong,flatMapToDouble 这个接口，跟map一样，接收一个Fucntion的函数式接口，不同的是，Function接收的泛型参数，第二个参数是一个Stream流；方法，返回的也是泛型R，具体的作用是把两个流，变成一个流返回
- distinct  去重复
- sorted  排序
- Stream<T> sorted(Comparator<? super T> comparator);  根据属性排序
- peek  对对象进行操作
- limit  截断--取先maxSize个对象
- skip  截断--忽略前N个对象
- empty 构建
- of 构建
- iterate 构建
- generate 构建
- concat 构建

### 终端操作

- forEach  对集合的流，进行遍历操作。  在并行的程序中，如果对处理之后的数据，没有顺序的要求，使用forEach的效率，肯定是要更好的

- forEachOrdered  对集合的流，进行遍历操作。 使用并行流（parallelStream流）时严格按照顺序取数据

- toArray  返回Object[]

- reduce  reduce 是一种归约操作，将流归约成一个值的操作叫做归约操作，用函数式编程语言的术语来说，这种称为折叠（fold）

- collect  收集器操作，可以当做是一种更高级的归约操作。收集器 从Collectors工厂类中创建收集器 import static **java.util.stream.Collectors.***;

  收集器非常有用，因为用它可以简洁而灵活地定义collect用来生成结果集合的标准。更具体地说，对流调用collect方法将对流中的元素触发一个归约操作（由Collector来参数化）。

- min  min和max传入的是一个Comparator这个是一个对比接口，那么返回就是根据比较的结果，取到的集合里面，最小的值

- max  取到的集合里面，最大的值

- count  跟List接口的size一样，返回的都是这个集合流的元素的长度，不同的是，流是集合的一个高级工厂，中间操作是工厂里的每一道工序，我们对这个流操作完成后，可以进行元素的数量的和

- anyMatch  判断的条件里，任意一个元素成功，返回true

- allMatch  判断条件里的元素，所有的都是，返回true

- noneMatch  跟allMatch相反，判断条件里的元素，所有的都不是，返回true

- findFirst  返回集合的第一个对象

- findAny  返回这个集合中，取到的任何一个对象。在串行的流中，findAny和findFirst返回的，都是第一个对象；而在并行的流中，findAny返回的是最快处理完的那个线程的数据，所以说，在并行操作中，对数据没有顺序上的要求，那么findAny的效率会比findFirst要快的

### 并行数据处理与性能

Stream接口可以让你以声明性方式处理数据集。到目前为止，最重要的好处是可以对这些集合执行操作流水线，能够自动利用计算机上的多个内核。

可以通过对收集源调用parallelStream方法来把集合转换为并行流。并行流就是一个把内容分成多个数据块，并用不同的线程分别处理每个数据块的流。这样一来，你就可以自动把给定操作的工作负荷分配给多核处理器的所有内核，让它们都忙起来。

对顺序流调用parallel方法转换成并行流，类似地，你只需要对并行流调用sequential方法就可以把它变成顺序流。

并行流内部使用了默认的ForkJoinPool，它默认的线程数量就是你的处理器数量，这个值是由 Runtime.getRuntime().available-Processors()得到的。

但是你可以通过系统属性 java.util.concurrent.ForkJoinPool.common.parallelism来改变线程池大小。

这是一个全局设置，因此它将影响代码中所有的并行流。反过来说，目前还无法专为某个并行流指定这个值。一般而言，让ForkJoinPool的大小等于处理器数量是个不错的默认值，除非你有很好的理由，否则我们强烈建议你不要修改它。

分支/合并框架的目的是以递归方式将可以并行的任务拆分成更小的任务，然后将每个子任务的结果合并起来生成整体结果。它是ExecutorService接口的一个实现，它把子任务分配给线程池（称为ForkJoinPool）中的工作线程。

Spliterator是Java 8中加入的另一个新接口；这个名字代表“可分迭代器”（splitable iterator）。和Iterator一样，Spliterator也用于遍历数据源中的元素，但它是为了并行执行而设计的。

### **stream 的特点**

**① 只能遍历一次：**

数据流的从一头获取数据源，在流水线上依次对元素进行操作，当元素通过流水线，便无法再对其进行操作，可以重新在数据源获取一个新的数据流进行操作；

**② 采用内部迭代的方式：**

对Collection进行处理，一般会使用 **Iterator** 遍历器的遍历方式，这是一种**外部迭代；**

而对于处理Stream，只要申明处理方式，处理过程由流对象自行完成，这是一种**内部迭代**，对于大量数据的迭代处理中，内部迭代比外部迭代要更加高效；

### **stream 相对于 Collection 的优点**

**无存储**：

流并不存储值；

流的元素源自数据源（可能是某个数据结构、生成函数或I/O通道等等），通过一系列计算步骤得到；

**函数式风格**：

对流的操作会产生一个结果，但流的数据源不会被修改；

**惰性求值**：

多数流操作（包括过滤、映射、排序以及去重）都可以以惰性方式实现。

这使得我们可以用一遍遍历完成整个流水线操作，并可以用短路操作提供更高效的实现；

**无需上界**：

不少问题都可以被表达为无限流（infinite stream）：

用户不停地读取流直到满意的结果出现为止（比如说，枚举 完美数 这个操作可以被表达为在所有整数上进行过滤）；

集合是有限的，但流可以表达为无限流；

**代码简练**：

对于一些collection的迭代处理操作，使用 stream 编写可以十分简洁，如果使用传统的 collection 迭代操作，代码可能十分啰嗦，可读性也会比较糟糕；

### stream 和 iterator 迭代的效率比较

**传统 iterator (for-loop) 比 stream(JDK8) 迭代性能要高，尤其在小数据量的情况下；**

在少低数据量的处理场景中（size<=1000），stream 的处理效率是不如传统的 iterator 外部迭代器处理速度快的，但是实际上这些处理任务本身运行时间都低于毫秒，这点效率的差距对普通业务几乎没有影响，反而 stream 可以使得代码更加简洁；

**在多核情景下，对于大数据量的处理，parallel stream 可以有比 iterator 更高的迭代处理效率；**

在大数据量（szie>10000）时，stream 的处理效率会高于 iterator，特别是使用了并行流，在cpu恰好将线程分配到多个核心的条件下（当然parallel stream 底层使用的是 JVM 的 ForkJoinPool，这东西分配线程本身就很玄学），可以达到一个很高的运行效率，然而实际普通业务一般不会有需要迭代高于10000次的计算；

Parallel Stream 受 CPU 环境影响很大，当没分配到多个cpu核心时，加上引用 forkJoinPool 的开销，运行效率可能还不如普通的 Stream；

stream 中含有装箱类型，在进行中间操作之前，最好**转成对应的数值流**，减少由于频繁的拆箱、装箱造成的性能损失；



## 4. 组合式异步编程CompletableFuture

JDK5新增了Future接口，用于描述一个异步计算的结果。虽然 Future 以及相关使用方法提供了异步执行任务的能力，但是对于结果的获取却是很不方便，只能通过阻塞方式`get()`或者轮询的方式`isDone()`得到任务的结果。阻塞的方式显然和我们的异步编程的初衷相违背，轮询的方式又会耗费无谓的 CPU 资源，而且也不能及时地得到计算结果。

从Java 8开始引入了`CompletableFuture`，它针对`Future`做了改进，可以传入回调对象，当异步任务完成或者发生异常时，自动调用回调对象的回调方法。

CompletableFuture实现了**CompletionStage接口**和**Future接口**，前者是对后者的一个扩展，增加了异步回调、流式处理、多个Future组合处理的能力，使Java在处理多任务的协同工作时更加顺畅便利。

CompletionStage接口

- CompletionStage代表异步计算过程中的某一个阶段，一个阶段完成以后可能会触发另外一个阶段
- 一个阶段的计算执行可以是一个Function，Consumer或者Runnable。比如：stage.thenApply(x -> square(x)).thenAccept(x -> System.out.print(x)).thenRun(() -> System.out.println())
- 一个阶段的执行可能是被单个阶段的完成触发，也可能是由多个阶段一起触发



### 创建异步操作

java.util.concurrent.CompletableFuture 提供了四个静态方法来创建一个异步操作。

```java
public static CompletableFuture<Void> runAsync(Runnable runnable)
public static CompletableFuture<Void> runAsync(Runnable runnable,Executor executor)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```

没有指定Executor的方法会使用全局的 `ForkJoinPool.commonPool()`中获得一个线程执行这些任务。如果指定线程池，则使用指定的线程池运行。以下所有的方法都类同:

- runAsync方法不支持返回值。
- supplyAsync可以支持返回值。

### 计算完成时的回调方法

当CompletableFuture的计算结果完成，或者抛出异常的时候，可以执行特定的Action。

**whenComplete 和 whenCompleteAsync** 区别：方法不以Async结尾，意味着Action使用相同的线程执行，而Async可能会使用其他线程执行（如果是使用相同的线程池，也可能会被同一个线程选中执行）

**handle** 是执行**任务完成时**对结果的处理。handle 是在任务完成后再执行，还可以处理异常的任务。

whenComplete接收的是BiConsumer，handler接收的是BiFunction；BiConsumer没有返回值，而BiFunction是有的。一个是返回传进去的值,一个是返回处理返回的值。

### 转换和运行 线程串行化方法

可以使用 `thenApply()`, `thenAccept()` 和`thenRun()`方法附上一个回调给CompletableFuture。

可以使用 `thenApply()` 处理和改变CompletableFuture的结果。持有一个`Function<R,T>`作为参数。`Function<R,T>`是一个简单的函数式接口，接受一个T类型的参数，产出一个R类型的结果。

如果你不想从你的回调函数中返回任何东西，仅仅想在Future完成后运行一些代码片段，你可以使用`thenAccept() `和 `thenRun()`方法，这些方法经常在调用链的最末端的最后一个回调函数中使用。
`CompletableFuture.thenAccept() `持有一个`Consumer<T> `，返回一个`CompletableFuture<Void>`。它可以访问`CompletableFuture`的结果。

虽然`thenAccept()`可以访问CompletableFuture的结果，但`thenRun()`不能访Future的结果，它持有一个Runnable返回CompletableFuture<Void>：

## 组合

**使用 `thenCompose() `组合两个独立的future**

如果你的回调函数返回一个CompletableFuture，但是你想从CompletableFuture链中获取一个直接合并后的结果，这时候你可以使用`thenCompose()`。

**使用`thenCombine()`组合两个独立的 future**

虽然`thenCompose()`被用于当一个future依赖另外一个future的时候用来组合两个future。`thenCombine()`被用来当两个独立的`Future`都完成的时候，用来做一些事情。

**组合多个CompletableFuture**

```java
static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs)
static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs)
```

`CompletableFuture.allOf`的使用场景是当你一个列表的独立future，并且你想在它们都完成后并行的做一些事情。

`CompletableFuture.anyOf()`和其名字介绍的一样，当任何一个CompletableFuture完成的时候【相同的结果类型】，返回一个新的CompletableFuture。

### 异常处理

**1. 使用 exceptionally() 回调处理异常**
`exceptionally()`回调给你一个从原始Future中生成的错误恢复的机会。你可以在这里记录这个异常并返回一个默认值。

**2. 使用 handle() 方法处理异常**
API提供了一个更通用的方法 - `handle()`从异常恢复，无论一个异常是否发生它都会被调用。




可见`CompletableFuture`的优点是：

- 异步任务结束时，会自动回调某个对象的方法；
- 异步任务出错时，会自动回调某个对象的方法；
- 主线程设置好回调后，不再关心异步任务的执行。



## 5. Optional静态类

几乎所有的Java程序员碰到NullPointerException时的第一冲动就是添加一个if语句，在调用方法使用该变量之前检查它的值是否为null，快速地搞定问题。

Java 8引入了一个名为java.util.Optional<T>的新的类。变量存在时，Optional类只是对类简单封装。变量不存在时，缺失的值会被建模成一个“空”的Optional对象，由方法Optional.empty()返回。Optional是一个容器对象，它可能包含空值，也可能包含非空值。当属性value被设置时，isPesent()方法将返回true，并且get()方法将返回这个值。

该类支持泛型，即其属性value可以是任何对象的实例。

### Optional类的方法

| **序号** | **方法**                                                     | **方法说明**                                                 |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **1**    | `private Optional()`                                         | 无参构造，构造一个空Optional                                 |
| **2**    | `private Optional(T value)`                                  | 根据传入的非空value构建Optional                              |
| **3**    | `public static<T> Optional<T> empty()`                       | 返回一个空的Optional，该实例的value为空                      |
| **4**    | `public static <T> Optional<T> of(T value)`                  | 根据传入的非空value构建Optional，与Optional(T value)方法作用相同 |
| **5**    | `public static <T> Optional<T> ofNullable(T value)`          | 与of(T value)方法不同的是，ofNullable(T value)允许你传入一个空的value，当传入的是空值时其创建一个空Optional，当传入的value非空时，与of()作用相同 |
| **6**    | `public T get()`                                             | 返回Optional的值，如果容器为空，则抛出NoSuchElementException异常 |
| **7**    | `public boolean isPresent()`                                 | 判断当家Optional是否已设置了值                               |
| **8**    | `public void ifPresent(Consumer<? super T> consumer)`        | 判断当家Optional是否已设置了值，如果有值，则调用Consumer函数式接口进行处理 |
| **9**    | `public Optional<T> filter(Predicate<? super T> predicate)`  | 如果设置了值，且满足Predicate的判断条件，则返回该Optional，否则返回一个空的Optional |
| **10**   | `public<U> Optional<U> map(Function<? super T, ? extends U> mapper)` | 如果Optional设置了value，则调用Function对值进行处理，并返回包含处理后值的Optional，否则返回空Optional |
| **11**   | `public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper)` | 与map()方法类型，不同的是它的mapper结果已经是一个Optional，不需要再对结果进行包装 |
| **12**   | `public T orElse(T other)`                                   | 如果Optional值不为空，则返回该值，否则返回other              |
| **13**   | `public T orElseGet(Supplier<? extends T> other)`            | 如果Optional值不为空，则返回该值，否则根据other另外生成一个  |
| **14**   | `public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier)throws X` | 如果Optional值不为空，则返回该值，否则通过supplier抛出一个异常 |

### 使用Optional避免空指针

在我们日常开发过程中不可避免地会遇到空指针问题，在以前，出现空指针问题，我们通常需要进行调试等方式才能最终定位到具体位置，尤其是在分布式系统服务之间的调用，问题更难定位。在使用Optional后，我们可以将接受到的参数对象进行包装，比如，订单服务要调用商品服务的一个接口，并将商品信息通过参数传入，这时候，传入的商品参数可能直接传入的就是null，这时，商品方法可以使用Optional.of(T)对传入的对象进行包装，如果T为空，则会直接抛出空指针异常，我们看到异常信息就能立即知道发生空指针的原因是参数T为空；或者，当传入的参数为空时，我们可以使用Optional.orElse()或Optional.orElseGet()方法生成一个默认的实例，再进行后续的操作。

下面再看个具体例子：在User类中有个Address类，在Address类中有个Street类,Street类中有streetName属性，现在的需求是：根据传入的User实例，获取对应的streetName，如果User为null或Address为null或Street为null，返回“nothing found”，否则返回对应的streetName。

```java
@Data
public class User {
	private String name;
	private Integer age;
	private Address address;
}

@Data
public class Address {
	private Street street;
}

@Data
public class Street {
	private String streetName;
	private Integer streetNo;
}
```

```java
public String getUserSteetName(User user) {

		if (null != user) {
			Address address = user.getAddress();
			if (null != address) {
				Street street = address.getStreet();
				if (null != street) {
					return street.getStreetName();
				}
			}
		}

		return "nothing found";
	}
```

```java
public String getUserSteetNameBetter(User user) {

		Optional<User> userOptional = Optional.ofNullable(user);
		final String streetName = Optional.ofNullable(
				Optional.ofNullable(userOptional
				.orElse(new User()).getAddress())
				.orElse(new Address()).getStreet())
				.orElse(new Street()).getStreetName();
		return StringUtils.isEmpty(streetName) ? "nothing found" : streetName;
	}
```



## 6. 新的日期和时间API

java8引入了一套全新的时间日期API，新的时间及日期API位于java.time包中整合了很多Joda-Time的特性，java.time包中的类是不可变且线程安全的。

下面是一些关键类

- Instant——它代表的是时间戳
- LocalDate——不包含具体时间的日期，比如2014-01-14。它可以用来存储生日，周年纪念日，入职日期等。
- LocalTime——它代表的是不含日期的时间
- LocalDateTime——它包含了日期及时间，不过还是没有偏移信息或者说时区。
- ZonedDateTime——这是一个包含时区的完整的日期时间，偏移量是以UTC/格林威治时间为基准的。
- Duration —— Duration类表示秒或纳秒时间间隔，适合处理较短的时间，需要更高的精确性。我们能使用between()方法比较两个瞬间的差。
- Period —— 计算两个日期之间包含多少天、周、月、年。

```jav
// 获取当前时间戳
// Instant类由一个静态的工厂方法now()可以返回当前时间戳
Instant timestamp = Instant.now();
log.info("当前的时间戳是：" + timestamp);

// 获取当前时间毫秒数
Date date = new Date();
log.info("" + date.getTime());
log.info("" + Calendar.getInstance().getTimeInMillis());
Instant timestamp1 = Instant.now();
log.info("" + timestamp1.toEpochMilli());

// 获取当前日期
LocalDate today = LocalDate.now();
log.info("今天的日期：" + today);

// 获取当前的年月日
int year = today.getYear();
int month = today.getMonthValue();
int day = today.getDayOfMonth();
log.info("年：" + year +" 月："+ month +" 日："+ day);

// 获取某个特定的日期
// 可以创建出任意一个日期，它接受年月日的参数，然后返回一个等价的LocalDate实例。
LocalDate dateOfBirth = LocalDate.of(2020, 11, 23);
log.info("生日：" + dateOfBirth);

// 检查两个日期是否相等
// LocalDate重写了equals方法来进行日期的比较
log.info("是否相等：" + dateOfBirth.equals(today));

// 使用MonthDay类。这个类由月日组合，不包含年信息，可以用来代表每年重复出现的一些日期或其他组合
MonthDay a = MonthDay.of(dateOfBirth.getMonth(),dateOfBirth.getDayOfMonth());
MonthDay b = MonthDay.from(today);
log.info("是否相等：" + a.equals(b));

// 如何判断某个日期在另一个日期的前面还是后面或者相等，
// 在java8中，LocalDate类中使用isBefore()、isAfter()、equals()方法来比较两个日期。
// 如果调用方法的那个日期比给定的日期要早的话，isBefore()方法会返回true。
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
log.info("是否在之前：" + tomorrow.isBefore(today));
log.info("是否在之后：" + tomorrow.isAfter(today));


// 获取1周后的日期
LocalDate oneToday = today.plus(1, ChronoUnit.WEEKS);
log.info("一周后的日期：" + oneToday);

// 获取1年前后的日期
LocalDate preYear = today.minus(1, ChronoUnit.YEARS);
log.info("一年前的日期：" + preYear);
LocalDate nextYear = today.plus(1, ChronoUnit.YEARS);
log.info("一年后的日期：" + nextYear);

// 获取当前时间 这里用的是LocalTime类，默认的格式是hh:mm:ss:nnn
LocalTime now = LocalTime.now();
log.info("现在的时间：" + now);

// 增加时间里面的小时数
LocalTime two = now.plusHours(2);
log.info("两小时后的时间：" + two);

// java8自带了Clock类，可以用来获取某个时区下（所以对时区是敏感的）当前的瞬时时间、日期。
// 用来代替System.currentTimelnMillis()与TimeZone.getDefault()方法
Clock clock = Clock.systemUTC();
log.info("clock：" + clock);
Clock clock1 = Clock.systemDefaultZone();
log.info("clock：" + clock1);

// java8中不仅将日期和时间进行了分离，同时还有时区。
// 比如ZonId代表的是某个特定时区，ZonedDateTime代表带时区的时间，等同于以前的GregorianCalendar类。
// 使用该类，可以将本地时间转换成另一个时区中的对应时间。
LocalDateTime localDateTime = LocalDateTime.now();
ZoneId zone = ZoneId.of(ZoneId.SHORT_IDS.get("ACT"));
ZonedDateTime dateAndTimeInNewYork = ZonedDateTime.of(localDateTime, zone);
log.info("现在时区的时间和在特定时区的时间：" + dateAndTimeInNewYork);

// 检查闰年
// LocalDate类有一个isLeapYear()方法来返回当前LocalDate对应的那年是否是闰年
log.info("{} 是否是闰年：{}", today,today.isLeapYear());

// 两个日期之间包含多少天，多少月
LocalDate date1 = LocalDate.of(2021, 11, 12);
LocalDate date2 = LocalDate.of(2021, 11, 23);
Period dValue = Period.between(date1, date2);
log.info("日期{}和日期{}相差{}个月", date1, date2, dValue.getMonths());
log.info("日期{}和日期{}相差{}天", date1, date2, dValue.get(ChronoUnit.DAYS));

LocalDate startDate = LocalDate.of(2015, 2, 20);
LocalDate endDate = LocalDate.of(2017, 2, 15);
Period period = Period.between(startDate, endDate);
log.info("Years:" + period.getYears() + 
		  " months:" + period.getMonths() + 
		  " days:"+period.getDays());

// 取两个日期直接相差多少天
// 方法1
long daysBetween = ChronoUnit.DAYS.between(date1, date2);
log.info("日期{}和日期{}相差{}天", date1, date2, daysBetween);
// 方法2
long minusDay = date2.toEpochDay() - date1.toEpochDay();
log.info("日期{}和日期{}相差{}天", date1, date2, minusDay);
long minusDay2 = endDate.toEpochDay() - startDate.toEpochDay();
log.info("日期{}和日期{}相差{}天", startDate, endDate, minusDay2);

// 两个日期之间相差多少秒，多少纳秒
Instant start = Instant.parse("2017-10-03T09:15:30.00Z");
Instant end = Instant.parse("2017-10-03T10:16:30.00Z");
Duration duration = Duration.between(start, end);
log.info("日期{}和日期{}相差{}秒",start, end, duration.getSeconds());

// 如何在java8中使用预定义的格式器来对日期进行解析/格式化
// 在java8之前，时间日期的格式化非常麻烦，经常使用SimpleDateFormat来进行格式化，但是SimpleDateFormat并不是线程安全的。
// 在java8中，引入了一个全新的线程安全的日期与时间格式器。并且预定义好了格式。
String dateString = "20211123";
LocalDate formatted = LocalDate.parse(dateString, DateTimeFormatter.BASIC_ISO_DATE);
log.info("字符串{}格式化后的日期格式是{}", dateString, formatted);

// 使用自定义的格式器来解析日期
String holidayString = "2021-11-23 11:11:11";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime holiday = LocalDateTime.parse(holidayString, formatter);
log.info("字符串{}格式化后的日期格式是{}", holidayString, holiday);

// 对日期进行格式化，转换成字符串
LocalDateTime arriveDate = LocalDateTime.now();
DateTimeFormatter formatter1 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String landing = arriveDate.format(formatter1);
log.info("时间{}格式化后的字符串是{}", arriveDate, landing);
```



### java8中日期与时间API的几个关键点

经过上面的例子，我们已经对java8的时间日期有了一定的了解，现在回顾一下

- 它提供了javax.time.ZoneId用来处理时区。
- 它提供了LocalDate与LocalTime类
- Java 8中新的时间与日期API中的所有类都是不可变且线程安全的，这与之前的Date与Calendar API中的恰好相反，那里面像java.util.Date以及SimpleDateFormat这些关键的类都不是线程安全的。
- 新的时间与日期API中很重要的一点是它定义清楚了基本的时间与日期的概念，比方说，瞬时时间，持续时间，日期，时间，时区以及时间段。它们都是基于ISO日历体系的。

### 每个Java开发人员都应该至少了解这套新的API中的这五个类：

- Instant 它代表的是时间戳，比如2016-04-14T14:20:13.592Z，这可以从java.time.Clock类中获取，像这样： Instant current = Clock.system(ZoneId.of("Asia/Tokyo")).instant();
- LocalDate 它表示的是不带时间的日期，比如2016-04-14。它可以用来存储生日，周年纪念日，入职日期等。
- LocalTime - 它表示的是不带日期的时间
- LocalDateTime - 它包含了时间与日期，不过没有带时区的偏移量
- ZonedDateTime - 这是一个带时区的完整时间，它根据UTC/格林威治时间来进行时区调整
- 这个库的主包是java.time，里面包含了代表日期，时间，瞬时以及持续时间的类。它有两个子package，一个是java.time.foramt，这个是什么用途就很明显了，还有一个是java.time.temporal，它能从更低层面对各个字段进行访问。
- 时区指的是地球上共享同一标准时间的地区。每个时区都有一个唯一标识符，同时还有一个地区/城市(Asia/Tokyo)的格式以及从格林威治时间开始的一个偏移时间。比如说，东京的偏移时间就是+09:00。
- OffsetDateTime类实际上包含了LocalDateTime与ZoneOffset。它用来表示一个包含格林威治时间偏移量（+/-小时：分，比如+06:00或者 -08：00）的完整的日期（年月日）及时间（时分秒，纳秒）。
- DateTimeFormatter类用于在Java中进行日期的格式化与解析。与SimpleDateFormat不同，它是不可变且线程安全的，如果需要的话，可以赋值给一个静态变量。DateTimeFormatter类提供了许多预定义的格式器，你也可以自定义自己想要的格式。当然了，根据约定，它还有一个parse()方法是用于将字符串转换成日期的，如果转换期间出现任何错误，它会抛出DateTimeParseException异常。类似的，DateFormatter类也有一个用于格式化日期的format()方法，它出错的话则会抛出DateTimeException异常。
- 再说一句，“MMM d yyyy”与“MMm dd yyyy”这两个日期格式也略有不同，前者能识别出"Jan 2 2014"与"Jan 14 2014"这两个串，而后者如果传进来的是"Jan 2 2014"则会报错，因为它期望月份处传进来的是两个字符。为了解决这个问题，在天为个位数的情况下，你得在前面补0，比如"Jan 2 2014"应该改为"Jan 02 2014"。