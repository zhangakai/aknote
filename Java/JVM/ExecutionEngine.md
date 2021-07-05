#虚拟机字节码执行引擎

-----------------------------------------

执行引擎是Java虚拟机核心的组成部分之一。“虚拟机”是一个相对于“物理机”的概念，这两种机
器都有代码执行能力，其区别是物理机的执行引擎是直接建立在处理器、缓存、指令集和操作系统层
面上的，而虚拟机的执行引擎则是由软件自行实现的，因此可以不受物理条件制约地定制指令集与执
行引擎的结构体系，能够执行那些不被硬件直接支持的指令集格式。

在不同的虚拟机实现中，执行引擎在执行字节码的 时候，通常会有解释执行（通过解释器执行）
和编译执行（通过即时编译器产生本地代码执行）两种选择。

------------------------------------------------

一.运行时栈帧结构

    1.局部变量表 
    存放方法参数和局部变量 以变量槽为单位 一个变量槽32或64位 变量槽的个数在编译时就确定了
    2.操作数栈 
    最大深度同上编译时确定 存在Code属性的max_stacks数据项
    两个不同栈帧作为不同方法的虚拟机栈的元素 是完全相互独立的
    但是在大多虚拟机的实现里都会进行一些优化处理 令两个栈帧出现一部分重叠
    重叠的就是操作数栈的一部分和局部变量表的一部分
    3.动态连接
    每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用 持有这个引用是为了支持方
    法调用过程中的动态连接(符号引用一部分会在类加载阶段或者第一次使用的时候就被转化为直接引用
    这种转化被称为静态解析 另外一部分将在每一次运行期间都转化为直接引用 这部分就称为动态连接)
    4.方法返回地址
    5.附件信息
    可能会有调试 性能收集相关的信息
    

二.方法调用

    方法调用阶段唯一的任务就是确定被调用方法的版本 即调用哪一个方法
    暂时还未涉及方法内部的具体运行过程

    1.解析
    所有方法调用的目标方法在Class文件里面都是一个常量池中的符号引用
    在类加载的解析阶段 会将其中的一部分符号引用转化为直接引用  这种解析能够成立的前
    提是:方法在程序真正运行之前就有一个可确定的调用版本 并且这个方法的调用版本在运行期是不
    可改变的 这种方法的调用被称为解析

    包括静态方法和私有方法 实例构造器 父类方法 以及 final修饰的方法
    静态方法与类型关联 私有方法外部不可访问 因此他们都不可能被重写 所以适合解析调用

    调用方法的指令有五种
    invokestatic-调用静态方法 invokespecial-调用init 私有方法 父类方法
    invokevirtual-虚方法 invokeinterface-接口方法
    invokedynamic-先在运行时动态解析出调用点限定符所引用的方法 然后再执行该方法

    2.分派

    静态分派 : 所有依赖静态类型来决定方法执行版本的分派动作 都称为静态分派
    首先两个概念:静态类型和实际类型 Person p = new Student()
    对于变量p 他的静态类型是Person 实际类型是Student
    静态类型总能在编译期可知 实际类型到运行时才知道 比如 Person p = x>0? new Student():new Teacher()
    
    应用:方法重载

    -----------------------------------------------------------------

    动态分派 运行时确定执行方法的类型  通过invokevirtual实现
    invokevirtual指令的运行时解析过程大致分为以下几步：
    1）找到操作数栈顶的第一个元素所指向的对象的实际类型，记作C。
    2）如果在类型C中找到与常量中的描述符和简单名称都相符的方法，则进行访问权限校验，如果
    通过则返回这个方法的直接引用，查找过程结束；不通过则返回java.lang.IllegalAccessError异常。
    3）否则，按照继承关系从下往上依次对C的各个父类进行第二步的搜索和验证过程。
    4）如果始终没有找到合适的方法，则抛出java.lang.AbstractMethodError异常

    优化:
    动态分派是执行非常频繁的动作，而且动态分派的方法版本选择过程需要运行时在接收者类型的
    方法元数据中搜索合适的目标方法，因此，Java虚拟机实现基于执行性能的考虑，真正运行时一般不
    会如此频繁地去反复搜索类型元数据。面对这种情况，一种基础而且常见的优化手段是为类型在方法
    区中建立一个虚方法表
    虚方法表中存放着各个方法的实际入口地址。如果某个方法在子类中没有被重写，那子类的虚方
    法表中的地址入口和父类相同方法的地址入口是一致的，都指向父类的实现入口。如果子类中重写了
    这个方法，子类虚方法表中的地址也会被替换为指向子类实现版本的入口地址。


三.动态类型语言支持

首先两个概念:静态类型语言和动态类型语言 
静态类型语言能够在编译期确定变量类型，最显著的好处是编译器可以提供全面严谨的
类型检查，这样与数据类型相关的潜在问题就能在编码时被及时发现，利于稳定性及让项目容易达到
更大的规模。而动态类型语言在运行期才确定类型，这可以为开发人员提供极大的灵活性，某些在静
态类型语言中要花大量臃肿代码来实现的功能，由动态类型语言去做可能会很清晰简洁，清晰简洁通
常也就意味着开发效率的提升。

Java最初是一门静态类型语言 但是后面虚拟机加入了对动态类型语言的支持

有点复杂 后面理解了写


四.基于栈的执行引擎

    1.基于栈的指令集与基于寄存器的指令集
    
    
        
    