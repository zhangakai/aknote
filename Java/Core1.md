#核心卷1笔记

    尽量不要使用char类型，最好将字符串类型作为抽象数据类型处理。

    java中if（a=0) 编译不通过 整形和boolean型不能互换。
    
    不提倡一行使用多个变量，提高程序可读性。
    
    string s = null 和string s = ""不一样。
    
    java 采用值引用 对值进行拷贝。方法得到的是对象引用的拷贝，对象引用及其他的拷贝同时引用同一个对象。
    
     为了保证包名的绝对唯一性， Sun 公司建议将公司的因特网域名（这显然是独一无二的）以逆序的形式作为包 名，并且对于不同的项目使用不同的子包。例如， horstmann.com 是本书作者之一注册的域 名。逆序形式为 com.horstmann。这个包还可以被进一步地划分成子包， 如 com.horstmann. corejava。
    
    方法的名字和参数列表称为方法的签名。例如，f(int) 和 f(String) 是两个具有相同名字， 不同签名的方法。如果在子类中定义了一个与超类签名相同的方 法， 那么子类中的这个方法就覆盖了超类中的这个相同签名的方法。 不过，返回类型不是签名的一部分， 因此，在覆盖方法时， 一定要保证返回类型 的兼容性。 允许子类将覆盖方法的返回类型定义为原返回类型的子类型。例如， 假设 Employee 类有 public Employee getBuddyO { . . . } 经理不会想找这种地位低下的员工。为了反映这一点， 在后面的子类 Manager 中， 可以按照如下所示的方式覆盖这个方法 public Manager getBuddyO { . . . } // OK to change return type 我们说，这两个 getBuddy 方法具有可协变的返回类型。
    
    在 Java 中，子类数组的引用可以转换成超类数组的引用， 而不需要采用强制类型 转换。例如， 下面是一个经理数组 Manager[] managers = new Manager[10];
    将它转换成 Employee[] 数组完全是合法的： EmployeeQ staff = managers; // OK 这样做肯定不会有问题， 请思考一下其中的缘由。毕竟， 如果 manager[i] 是一个 Manager, 也一定是一个 Employee。然而， 实际上，将会发生一些令人惊讶的事情。要 切记 managers 和 staff 引用的是同一个数组。现在看一下这条语句： staff[0] = new Employee("Harry Hacker", . . .); 编译器竟然接纳了这个赋值操作。但在这里， stafflO] 与 manager[0] 引用的是同一个 对象， 似乎我们把一个普通雇员擅自归入经理行列中了。这是一种很忌伟发生的情形， 当调用 managers[0].setBonus(U)00) 的时候， 将会导致调用一个不存在的实例域， 进而搅 乱相邻存储空间的内容。 为了确保不发生这类错误， 所有数组都要牢记创建它们的元素类型， 并负责监督仅 将类型兼容的引用存储到数组中。例如， 使用 new managers[10] 创建的数组是一个经理 数组。 如果试图存储一个 Employee 类型的引用就会引发 ArrayStoreException 异常。
    
    如果隐式和显式的参数不属于同一个类， equals 方法将如何处理呢？ 这是一个很有争议
    的问题。 在前面的例子中， 如果发现类不匹配，equals方法就返冋 false: 但是，许多程序员 却喜欢使用 instanceof进行检测： if (KotherObject instanceof Employee)) return false; 这样做不但没有解决 otherObject 是子类的情况，并且还有可能会招致一些麻烦。这就是建议 不要使用这种处理方式的原因所在。Java语言规范要求 equals 方法具有下面的特性： 1 ) 自反性：对于任何非空引用 x, x.equals(?0应该返回 truec 2 ) 对称性: 对于任何引用 x 和 y, 当且仅当 y.equals(x) 返回 true, x.equals(y) 也应该返 回 true。 3 ) 传递性： 对于任何引用 x、 y 和 z, 如果 x.equals(y) 返N true， y.equals(z)返回 true, x.equals(z) 也应该返回 true。 4 ) 一致性： 如果 x 和 y引用的对象没有发生变化，反复调用 x.eqimIS(y) 应该返回同样 的结果。 5 ) 对于任意非空引用 x, x.equals(null) 应该返回 false, 这些规则十分合乎情理，从而避免了类库实现者在数据结构中定位一个元素时还要考虑 调用 x.equals(y), 还是调用 y.equals(x) 的问题。 然而， 就对称性来说， 当参数不属于同一个类的时候需要仔细地思考一下。请看下面这 个调用： e.equals(in) 这里的 e 是一个 Employee 对象，m 是一个 Manager 对象， 并且两个对象具有相同的姓名、 薪水和雇佣日期。 如果在 Employee.equals 中用 instanceof进行检测， 则返回 true,. 然而这意 味着反过来调用： m.equals(e) 也需要返回 true、 对称性不允许这个方法调用返回 false，或者抛出异常： 这就使得 Manager 类受到了束缚。 这个类的 equals方法必须能够用自己与任何一 个 Employee 对象进行比较， 而不考虑经理拥有的那部分特有信息！ 猛然间会让人感觉 instanceof测试并不是完美无瑕, 某些书的作者认为不应该利用getClass 检测， 因为这样不符合置换原则有一个应用 AbstractSet 类的 equals 方法的典型例子，它将检测两个集合是否有相同的元素。AbstractSet 类有两个具体子类：TreeSet 和 HashSet, 它们分别使用不同的算法实现查找集合元素的操作。 无论集合采用何种方式实现，都需要拥有对任意两个集合进行比较的功能 然而， 集合是相当特殊的一个例子， 应该将 AbstractSetequals 声明为 final, 这是因为没 有任何一个子类需要重定义集合是否相等的语义（事实上，这个方法并没有被声明为 final。 这样做， 可以让子类选择更加有效的算法对集合进行是否相等的检测） 下面可以从两个截然不同的情况看一下这个问题： •如果子类能够拥有自己的相等概念， 则对称性需求将强制采用 getClass 进行检测
    •如果由超类决定相等的概念，那么就可以使用 imtanceof进行检测， 这样可以在不同 子类的对象之间进行相等的比较。 在雇员和经理的例子中， 只要对应的域相等， 就认为两个对象相等。如果两个 Manager 对象所对应的姓名、 薪水和雇佣日期均相等， 而奖金不相等， 就认为它们是不相同的， 因 此，可以使用 getClass 检测。 但是，假设使用雇员的 ID 作为相等的检测标准，并且这个相等的概念适用于所有的子 类， 就可以使用 instanceof进行检测， 并应该将 Employee.equals 声明为 final。
     
    下面给出编写一个完美的 equals方法的建议： 
    1 ) 显式参数命名为 otherObject, 稍后需要将它转换成另一个叫做 other 的变量。 
    2 ) 检测 this 与 otherObject 是否引用同一个对象： if (this = otherObject) return true; 这条语句只是一个优化。实际上，这是一种经常采用的形式。因为计算这个等式要比一 个一个地比较类中的域所付出的代价小得多。
    
     3 ) 检测 otherObject 是否为 null, 如果为 null, 返 回 false。这项检测是很必要的。 if (otherObject = null) return false; 4 ) 比较 this 与 otherObject 是否属于同一个类。如果 equals 的语义在每个子类中有所改 变，就使用 getClass 检测： if (getClass() != otherObject.getCIassO) return false; 如果所有的子类都拥有统一的语义，就使用 instanceof检测： if (!(otherObject instanceof ClassName)) return false;
     5 ) 将 otherObject 转换为相应的类类型变量： ClassName other = (ClassName) otherObject 
    6 ) 现在开始对所有需要比较的域进行比较了。使用 = 比较基本类型域，使用 equals 比 较对象域。如果所有的域都匹配， 就返回 true; 否 则 返 回 false。 return fieldl == other.field && Objects.equa1s(fie1d2, other.field2)
    如果在子类中重新定义 equals, 就要在其中包含调用super.equals(other)。 
    
     令人烦恼的是， 数组继承了 object 类的 toString 方法，数组类型将按照旧的格式 打印。例如： int[] luckyNumbers = { 2, 3, 5, 7S llf 13 }; String s = "" + luckyNumbers; 生成字符串“ [I@la46e30”（前缀 [I 表明是一个整型数组） 。修正的方式是调用静态 方法 Arrays.toString。代码： String s = Arrays.toString(luckyNumbers); 将生成字符串“ [2,3,5,7，11，13]”。 要想打印多维数组（即， 数组的数组）则需要调用 Arrays.deepToString 方法。
    
    分配数组列表， 如下所示： new ArrayListo(lOO) // capacity is 100 它与为新数组分配空间有所不同： new Employee[100] // size is 100 数组列表的容量与数组的大小有一个非常重要的区别。如果为数组分配 100 个元素 的存储空间，数组就有 100 个空位置可以使用。而容量为 100 个元素的数组列表只是拥 有保存 100 个元素的潜力（实际上， 重新分配空间的话，将会超过丨00 ), 但是在最初， 甚至完成初始化构造之后，数组列表根本就不含有任何元素。 
    
    ArrayList 中的size是指实际存储元素的个数。
    
    由于每个值分别包装在对象中， 所以 ArrayList<lnteger> 的效率远远低于 int[ ] 数 组。 因此， 应该用它构造小型集合，其原因是此时程序员操作的方便性要比执行效率更 加重要。 
    
    在比较两个枚举类型的值时， 永远不需要调用 equals, 而直接使用“ = =” 就 可以了。
    
    现在来考虑另一种情况，一个类扩展了一个超类， 同时实现了一个接口，并从超类和接口继承了相同的方法。例如， 假设 Person 是一个类， Student 定义为： class Student extends Person implements Named {. . . } 在这种情况下， 只会考虑超类方法，接口的所有默认方法都会被忽略。在我们的例子 中， Student 从 Person继承了 getName 方法， Named 接口是否为 getName 提供了默认实现并 不会带来什么区别。这正是“ 类优先” 规则。 
    
     在 JavaSE 8 中， 接口可以声明非抽象方法。 
    
    Java 有一个限制，无法构造泛型类型 T 的数组。数组构造器引用对于克服这个限制很有 用。表达式 new T[n]会产生错误，因为这会改为 new Object[n]。对于开发类库的人来说，这 是一个问题。例如，假设我们需要一个 Person 对象数组。Stream 接口有一个 toArray方法可 以返回 Object 数组： Object[] people = stream.toArrayO； 不过，这并不让人满意。用户希望得到一个 Person 引用数组，而不是 Object 引用数组。 流库利用构造器引用解决了这个问题。可以把 Person[]::new?人 toArray方法： Person口 people = stream.toArray(PersonD::new): toArray方法调用这个构造器来得到一个正确类型的数组。然后填充这个数组并返回。
    
     关于代码块以及自由变量值有一个术语： 闭包（c/o«/re)。如果有人吹嘘他们的语 言有闭包，现在你也可以自信地说 Java 也有闭包。在 Java 中， lambda 表达式就是闭包。 
    
    可以看到，lambda 表达式可以捕获外围作用域中变量的值。在 Java 中， 要确保所捕获 的值是明确定义的，这里有一个重要的限制。在 lambda 表达式中， 只能引用值不会改变的 变量。 
    
    lambda 表达式的体与嵌套块有相同的作用域。这里同样适用命名冲突和遮蔽的有关规 则。在 lambda 表达式中声明与一个局部变量同名的参数或局部变量是不合法的。 Path first = Paths.get(7usr/Mn"); Couparator<String> comp = (first, second) -> first.length() - second.lengthO; II Error: Variable first already defined 在方法中，不能有两个同名的局部变量， 因此， lambda 表达式中同样也不能有同名的局 部变量。 
    
    在一个 lambda 表达式中使用 this 关键字时， 是指创建这个 lambda 表达式的方法的 this 参数。例如，考虑下面的代码：
     public class ApplicationO {
     public void init() {
     ActionListener listener * event -> { System.out.print n(this.toStringO);
    } 表达式 this.toStringO 会调用 Application 对象的 toString方法， 而不是 ActionListener 实 例的方法。在 lambda 表达式中， this 的使用并没有任何特殊之处。lambda 表达式的作用域嵌 套在 init 方法中，与出现在这个方法中的其他位置一样， lambda 表达式中 this 的含义并没有 变化。
    
    
    内部类（inner class) 是定义在另一个类中的类。为什么需要使用内部类呢？ 其主要原因
    第 6章 接 口、 lambda 表?式与?部? 243
    有以下三点： •内部类方法可以访问该类定义所在的作用域中的数据， 包括私有的数据。 •内部类可以对同一个包中的其他类隐藏起来。 •当想要定义一个回调函数且不想编写大量代码时，使用匿名（anonymous) 内部类比较 便捷。 
    
    内部类（inner class) 是定义在另一个类中的类。为什么需要使用内部类呢？ 其主要原因有以下三点： •内部类方法可以访问该类定义所在的作用域中的数据， 包括私有的数据。 •内部类可以对同一个包中的其他类隐藏起来。 •当想要定义一个回调函数且不想编写大量代码时，使用匿名（anonymous) 内部类比较 便捷。 
    
    
    Error类层次结构描述了 Java 运行时系统的内部错误和资源耗尽错误。应用程序不应该 抛出这种类型的对象。 如果出现了这样的内部错误， 除了通告给用户，并尽力使程序安全地 终止之外， 再也无能为力了。这种情况很少出现。 
    
    在设计 Java 程序时， 需要关注 Exception 层次结构。这个层次结构又分解为两个分支： 一个分支派生于 RuntimeException ; 另一个分支包含其他异常。划分两个分支的规则是： 由 程序错误导致的异常属于 RuntimeException ; 而程序本身没有问题， 但由于像 I/O 错误这类 问题导致的异常属于其他异常: 派生于 RuntimeException 的异常包含下面几种情况： •错误的类型转换。 •数组访问越界 i •访问 null 指针 不是派生于 RuntimeException 的异常包括： •试图在文件尾部后面读取数据。 •试图打开一个不存在的文件。 •试图根据给定的字符串查找 Class 对象， 而这个字符串表示的类并不存在,， “ 如果出现 RuntimeException 异常， 那么就一定是你的问题” 是一条相当有道理的规则。 应该通过检测数组下标是否越界来避免 ArraylndexOutOfBoundsException 异常；应该通过在 使用变量之前检测是否为 null 来杜绝 NullPointerException 异常的发生。
    
    Java语 言 规 范 将 派 生 于 Error 类 或 RuntimeException类的所有异常称为非受查 ( unchecked) 异常，所有其他的异常称为受查（checked) 异常。这是两个很有用的术语，在 后面还会用到。编译器将核查是否为所有的受査异常提供了异常处理器
    
    在自己编写方法时， 不必将所有可能抛出的异常都进行声明。至于什么时候需要在方法 中用 throws 子句声明异常， 什么异常必须使用 throws 子句声明， 需要记住在遇到下面 4 种 情况时应该抛出异常： 1 ) 调用一个抛出受査异常的方法， 例如， FilelnputStream 构造器。 2 ) 程序运行过程中发现错误，并且利用 throw语句抛出一个受查异常（下一节将详细地 介绍 throw 语句)。 3 ) 程序出现错误， 例如，a[-l]=0 会抛出一个 ArraylndexOutOffloundsException 这样的 非受查异常。
    4 ) Java 虚拟机和运行时库出现的内部错误。 如果出现前两种情况之一， 则必须告诉调用这个方法的程序员有可能抛出异常。为什 么？ 因为任何一个抛出异常的方法都有可能是一个死亡陷阱。 如果没有处理器捕获这个异 常，当前执行的线程就会结束。
    
    不需要声明 Java 的内部错误，即从 Error 继承的错误。任何程序代码都具有抛出那些 异常的潜能， 而我们对其没有任何控制能力。 同样，也不应该声明从 RuntimeException 继承的那些非受查异常。
    
    一个方法必须声明所有可能抛出的受查异常， 而非受查异常要么不可控制（Error), 要么就应该避免发生（ RuntimeException)。如果方法没有声明所有可能发生的受查异常， 编 译器就会发出一个错误消息。 
    
    
    特别需要说明的是， 如果超类方法没有抛出任何受查异常， 子类也不能抛 出任何受查异常。
    
    在程序中，可能会遇到任何标准异常类都没有能够充分地描述清楚的问题。 在这种情 况下，创建自己的异常类就是一件顺理成章的事情了。我们需要做的只是定义一个派生于 Exception 的类，或者派生于 Exception 子类的类。例如， 定义一个派生于 IOException 的类。 习惯上， 定义的类应该包含两个构造器， 一个是默认的构造器；另一个是带有详细描述信息 的构造器（超类 Throwable 的 toString 方法将会打印出这些详细信息， 这在调试中非常有用)。 
    
    如果在 try语句块中的任何代码抛出了一个在 catch子句中说明的异常类，那么 1 ) 程序将跳过 try语句块的其余代码。
    2 ) 程序将执行 catch 子句中的处理器代码。 如果在 try 语句块中的代码没有拋出任何异常，那么程序将跳过 catch 子句。 如果方法中的任何代码拋出了一个在 catch 子句中没有声明的异常类型，那么这个方法 就会立刻退出（希望调用者为这种类型的异常设tKT catch 子句)。
    
     请记住， 编译器严格地执行 throws说明符。 如果调用了一个抛出受查异常的方法，就必 须对它进行处理， 或者继续传递。 哪种方法更好呢？ 通常， 应该捕获那些知道如何处理的异常， 而将那些不知道怎样处理 的异常继续进行传递。 如果想传递一个异常， 就必须在方法的首部添加一个 throws说明符， 以便告知调用者这 个方法可能会抛出异常
    
    虚拟机中的对象总有一个特定的非泛型类型。因此，所有的类型查询只产生原始类型。 例如： if (a instanceof Pair<String>) // Error 实际上仅仅测试 a 是否是任意类型的一个 Pair。下面的测试同样如此： if (a instanceof Pair<T>) // Error 或强制类型转换： Pair<St「ing> p = (Pair<String>) a; // Warning-can only test that a is a Pair 为提醒这一风险， 试图查询一个对象是否属于某个泛型类型时，倘若使用 instanceof 会 得到一个编译器错误， 如果使用强制类型转换会得到一个警告。 同样的道理， getClass 方法总是返回原始类型。例如： Pair<String> stringPair = . . Pai「<Employee〉employeePair = . . if (stringPair.getClassO == employeePair.getClassO) // they are equal 其比较的结果是 true, 这是因为两次调用 getClass 都将返回 Pair.class。
    
    
    8.6.3 不能创建参数化类型的数组
    不能实例化参数化类型的数组， 例如： Pair<String>[] table = new Pair<String>[10]; // Error 这有什么问题呢？ 擦除之后， table 的类型是 Pair[]。可以把它转换为 Object□: Object[] objarray = table; 数组会记住它的元素类型， 如果试图存储其他类型的元素， 就会抛出一个 ArrayStoreException 异常： objarray[0] = "Hello"; // Error component type is Pair 不过对于泛型类型， 擦除会使这种机制无效。以下赋值： objarray[0] = new Pair<Employee>0； 能够通过数组存储检査， 不过仍会导致一个类型错误。出于这个原因， 不允许创建参数 化类型的数组。 需要说明的是， 只是不允许创建这些数组， 而声明类型为 Pair<String>[] 的变量仍是合法 的。不过不能用 new Pair<String>[10] 初始化这个变量。 0 注释： 可以声明通配类型的数组， 然后进行类型转换： Pair<String>[] table = (Pair<String>[]) new Pair<?>[10];
    结果将是不安全的。如果在 table[0] 中存储一个 Pair<Employee>, 然后对 table[0]. getFirst() 调用一个 String 方法， 会得到一个 ClassCastException 异常。 提不：如果需要收集参数化类型对象， 只有一种安全而有效的方法： 使用 ArrayList:Arra yList<Pair<String»
    
    
     可以使用 @SafeVarargs 标注来消除创建泛型数组的有关限制， 方法如下： @SafeVarargs static <E> EQ array(E... array) { return array; } 现在可以调用： Pair<String>[] table = array(pairl，pai「2); 这看起很方便，不过隐藏着危险。以下代码： Object□ objarray = table; objarray[0] = new Pair<Employee>(); 能顺利运行而不会出现 ArrayStoreException 异常（因为数组存储只会检查擦除的类 型)，但在处理 table[0] 时你会在别处得到一个异常。
    
    
     
    
