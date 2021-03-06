#Proxy

----------------------------------

动态代理实现:

    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        ......    //做一些检查
        
        /*
        *创建代理类 ->如果此代理类已存在,直接返回已存在的
        *否则调用ProxyClassFactory生成所需代理类。
        *ProxyClassFactory->生成代理类的字节码文件,然后加载到JVM
        */

        //使用软引用指向 生成的代理类的Class对象 (猜测:方便类卸载)
        Class<?> cl = getProxyClass0(loader, intfs);

        //生成代理对象
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }

            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            
            //传入自定义的InvocationHandler对象的引用
            return cons.newInstance(new Object[]{h});
        .........
    }

例子:

    
    Service service = new ServiceImpl();
    InvocationHandler invocationHandler = new InvocationHandler() {
            @Override
            // proxy为代理类的一个实例对象 method是被代理方法，args是传到方法的参数列表
            // service是被代理对象 proxy除了用来输出一些调试信息外，没啥用了
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("before connect");
                return method.invoke(service,args);
            };
        };
    Class<?> serviceImplClass = ServiceImpl.class;
    Service proxyService = (Service) Proxy.newProxyInstance
                (serviceImplClass.getClassLoader(),
                serviceImplClass.getInterfaces(), invocationHandler);
    proxyService.connectDB();


CGlib代理:

    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(PersonService.class);
    //obj为代理对象，method为原方法 args为参数列表 proxy为代理方法
    enhancer.setCallback((MethodInterceptor) (obj, method, args, proxy) -> {
    //可以做一定的过滤操作
    if (method.getDeclaringClass() != Object.class && method.getReturnType() == String.class) {
        return "Hello Tom!";
    } else {
        //调用代理方法
        return proxy.invokeSuper(obj, args);
        }
    });
    
    PersonService proxy = (PersonService) enhancer.create();


    