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


    

    