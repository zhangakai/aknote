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



    