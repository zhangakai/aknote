#简单spring实现

-----------------------------

IOC实现   

    创建注解
    @target 作用在类上 @retention 生命周期为运行时 用在controller层
    实现service：同上 用在service 层
    实现repository:同上 用在dao层
    实现component：通用注解 可以代替上面三个 
    针对不同的场景实现不同的逻辑

    根据package获取类集合 ---使用类加载器获取资源信息
    为什么不传入绝对路径进行扫描获取？
    因为1.不同机器之间的路径可能不相同 比如windows和Linux目录符号是反着的
    2.如果打包的是jar或者是War包 根本找不到路径
    步骤
    public static Set<Class<?>> extractPackageClass(String packageName) {
        ClassLoader classLoader = getClassLoader();
        URL packageUrl = classLoader.getResource(packageName.replace('.', '/'));
        if (packageUrl == null) {
            log.warn("unable to get anything from package" + packageName);
            return null;
        }

        Set<Class<?>> classSet = null;
        if (packageUrl.getProtocol().equalsIgnoreCase("file")) {
            classSet = new HashSet<>();
            File packageDirectory = new File(packageUrl.getPath());
            extractClassFile(classSet, packageDirectory, packageName);
        }
        return classSet;

    }

    private static void extractClassFile(Set<Class<?>> classSet, File fileSource, String packageName) {
        if (!fileSource.isDirectory()) {
            return;
        }
        File[] files = fileSource.listFiles(new FileFilter() {
            @Override
            public boolean accept(File pathname) {
                if (pathname.isDirectory()) {
                    return true;
                } else {
                    String absFilePath = pathname.getAbsolutePath();
                    if (absFilePath.endsWith(".class")) {
                        addToClassSet(absFilePath);
                    }
                }
                return false;
            }

            private void addToClassSet(String absFilePath)  {
                absFilePath = absFilePath.replace(File.separatorChar, '.');
                String className = absFilePath.substring(absFilePath.indexOf(packageName));
                className = className.substring(0, className.lastIndexOf('.'));'o'
                Class targetClass = null;
                try {
                    targetClass = Class.forName(className);
                } catch (ClassNotFoundException e) {
                    e.printStackTrace();
                }
                classSet.add(targetClass);
            }
        });
        if (files != null) {
            for (File f : files) {
                extractClassFile(classSet, f, packageName);
            }
        }

    }

    实现单例容器

    public enum BeanContainer {
    CONTAINER;

    public static BeanContainer getContainer(){
        return CONTAINER;
    }

    private final Map<Class<?>, Object> beanMap = new ConcurrentHashMap<>();

    private static final List<Class<? extends Annotation>> BEAN_ANNOTATION =
            Arrays.asList(Component.class, Service.class, Repository.class, Controller.class);

    public void loadBeans(String packageName) {
        Set<Class<?>> classSet = ClassUtil.extractPackageClass(packageName);
        if (classSet == null || classSet.isEmpty()) {
            return;
        }
        for (Class<?> clazz : classSet) {
            for (Class<? extends Annotation> annotationClass : BEAN_ANNOTATION) {
                if (clazz.isAnnotationPresent(annotationClass)) {
                    try {
                        beanMap.put(clazz, clazz.getDeclaredConstructor().newInstance());
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    public Object abbBean(Class<?> clazz, Object bean) {
        return beanMap.put(clazz, bean);
    }

    public Object removeBean(Class<?> clazz) {
        return beanMap.remove(clazz);
    }

    public Set<Class<?>> getClasses(){
        return beanMap.keySet();
    }

    public Object getBean(Class<?> clazz){
        return beanMap.get(clazz);
    }

    public Set<Object> getBeans(){
        return new HashSet<>(beanMap.values());
    }

    public Set<Class<?>> getClassByAnnotation(Class<? extends Annotation> annotation) {
        Set<Class<?>> keySet = getClasses();
        if (keySet == null || keySet.isEmpty()) {
            log.warn("nothing in map");
            return null;
        }
        Set<Class<?>> classSet = new HashSet<>();
        for (Class<?> aClass : keySet) {
            if (aClass.isAnnotationPresent(annotation)) {
                classSet.add(aClass);
            }
        }
        return classSet;
    }

    public Set<Class<?>> getClassBySuper(Class<?> interfaceOrClass) {
        Set<Class<?>> keySet = getClasses();
        if (keySet == null || keySet.isEmpty()) {
            log.warn("nothing in map");
            return null;
        }
        Set<Class<?>> classSet = new HashSet<>();
        for (Class<?> aClass : keySet) {
            if (interfaceOrClass.isAssignableFrom(aClass)) {
                classSet.add(aClass);
            }
        }
        return classSet;
    }



    public static ClassLoader getClassLoader(){
        return Thread.currentThread().getContextClassLoader();
    }


    

AOP

    首先创建注解
    public @interface Aspect {
    //需要被织入横切逻辑的注解标签
    Class<? extends Annotation> value();
    }
    public @interface Order {
    按值的大小进行排序
    int value();
    }
    创建 用于封装顺序和对应的Aspect对象
    public class AspectInfo {
    private int orderIndex;
    private DefaultAspect aspectObject;
    }

    二 
    实现Aspect横切逻辑 以及被代理方法的定序执行
    创建MethodInterceptor的实现类 定义成员变量 targetClass aspectInfoList
    按照order对aspect进行排序 实现定序执行
    AspectListExecutor implements MethodInterceptor
    
    重写intercept(Object target, Method method, Object[] args, MethodProxy methodProxy
    按照oder的顺序升序执行完aspect的before方法
    执行被代理类的方法 returnValue = methodProxy.invoke(target, args);
    如果被代理方法正常返回 则按照oder的顺序降序执行完afterReturning方法
    如果被代理方法异常返回 则。。。。。。。。。执行完afterThrowing方法
    最后返回returnValue

    实现AOP class AspectWeaver 
    1.获取所有的切面类(用@Aspect修饰的类)
    2.将切面类按照不同的织入目标进行切分 Map<Class<? extends Annotation>, List<AspectInfo>>
    key是@controller @service等四个注解 value 是aspectInfo列表对象
    3.按照不同的织入目标进行织入Aspect的逻辑
    4.获取被代理类的集合
    5.遍历被代理类 分别为每个代理类生成动态代理实例
    Object proxy = Enhancer.enhance(targetClass, aspectListExecutor);
    6.将动态代理对象实例添加到容器中

    AspectJ框架
    编译时织入 编译后织入 类加载器织入(利用java agent在 类加载的时候织入切面逻辑 在被代理类中加入字节码)
    改写@Aspect string pointcut();
    创建PointCutLocator类 用于解析Aspect表达式并且定位被织入的目标
    内部有两个成员变量 一个是Poincut解析器 一个是解析出来的表达式 根据解析出来的表达式可以判断一个类是不是符合要求
    遍历所有类 所有方法 匹配则织入类 生成代理对象
    



MVC

    实现DispatcherServlet extends HttpServlet用于
    解析请求路径和请求方法
    依赖容器 建立并维护Controller方法与请求的映射 
    用合适的Controller方法去处理特定的请求
    在配置文件中写上path路径用来将请求转发给dispatchservlet

    实现责任链
    public class RequestProcessorChain {
    private Iterator<RequestProcessor> requestProcessorIterator;

    private HttpServletRequest request;

    private HttpServletResponse response;

    private  String requestMethod;

    private  String requestPath;

    private  int responseCode;

    private ResultRender resultRender;}
    方法有两个 
    /*以责任链的模式执行请求处理*/
    public void doRequestProcessChain() {
        try {
            while (requestProcessorIterator.hasNext()) {
                if (!requestProcessorIterator.next().process(this)) {
                    break;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void doRender(){
        if (this.resultRender == null) {
            this.resultRender = new DefaultResultRender();
        }
        try {
            this.resultRender.render(this);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    实现interface RequestProcessor 里面有个process方法 方法参数为
    processChain对象
    四个实现类分别是Pre Static Jsp Controller 
    pre用于请求预处理 包括编码以及路径处理---统一设置为UTF-8 
    以及将请求末尾的/除掉
    
    题外话tomcat管理servlet的是context 将servlet实例包装成wrapper 负责servleet的生命周期
    
    static 静态资源请求处理 包括但不限于图片 css等
    字段RequestDispatcher dispatcher;
    dispatcher = context.getNamedDispatcher("default");
    process方法里面:4
    通过请求路径是否是请求的静态资源(写死了 startsWith("/static/"))
    如果是 则转发给dispathcer处理 并且返回false 这样chain就停止了
    
    jspRequestProcessor同上
    只是获得startsWith("/templates/")) 变成了context.getNamedDispatcher("jsp");
    前缀判读变成了startsWith("/templates/")) 


    ControllerRequestProcessor负责的最多 
    主要功能为针对特定请求 选择匹配的Controller方法进行处理
    解析请求里的参数以及对应的值 并赋值给Controller方法里面的参数
    选择合适的render为后续请求处理结果的渲染做准备
    首先实现注解
    1.@interface RequestMapping 作用在方法及类上
    String value() default ""; 用于表明请求路径
    RequestMethod method() default RequestMethod.GET;表明请求方法
    2.public @interface RequestParam {
    String value() default "";请求的方法参数名称
    boolean required() default true;该参数是否必须
    }
    3.public @interface ResponseBody /*用于标记对返回值进行json处理*/

    其次创建两个封装类
    一个是public class ControllerMethod {
    private Class<?> controllerClass;
    private Method  invokeMethod;
    private Map<String,Class<?>> methodParameters;
    }
    一个是public class RequestPathInfo {
    private String httpMethod;//post get
    private String httpPath;
    }

    初始化映射关系
    1.遍历所有的被@ResquestMapping标记的类 获得注解值作为一级路径
    2.遍历类里面所有方法被@RequestMapping标记的方法 获得注解值作为二级路径
    3.解析方法里被@RequestPrarm标记的参数 获取该注解的值 作为参数名 获取被标记的参数类型 建立参数名和参数类型的映射
    4.将获得的信息封装为以上两个对象
    5.放在Map里
    
    process方法里面:
    解析请求方法 请求路径 获得对应的ControllerMethod实例 解析参数 传递给对应的ControllerMethod实例执行
    


----------------------------------------------------------


spring boot 启动过程

    SpringAppliction 负责
    创建一个合适的ApplictionContext实例
    注册一个CommandLinePropertySource负责将命令行参数转化为spring proprty
    刷新application context 加载所有的单例bean
    激活所有的CommandLineRunner(一个接口,表示要run的bean) bean
    Create an appropriate ApplicationContext instance (depending on your classpath)
    Register a CommandLinePropertySource to expose command line arguments as Spring properties
    Refresh the application context, loading all singleton beans
    Trigger any CommandLineRunner beans

    l Spring是提供基本功能的核心
    
    l SpringMVC是基于Spring的MVC框架
    
    l SpringBoot是一个快速开发集成包，用于简化Spring配置；
    
    l SpringCloud是基于SpringBoot的服务管理框架。





    
    
