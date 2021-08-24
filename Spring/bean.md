#bean

------------------------------------------

Bean Scope

    首先Scope是什么意思呢?
    答:指的是bean的生命周期,这意味着 Bean 的对象将在何时被实例化,该对象存在多长时间以及在整个过程中将为该 Bean 创建多少个对象
    新版本
    singleton
    (Default) Scopes a single bean definition to a single object instance per Spring IoC container.
    
    prototype //spring只负责生产 prototype的生命周期spring不管
    Scopes a single bean definition to any number of object instances.
    
    request
    Scopes a single bean definition to the lifecycle of a single HTTP request; that is, each HTTP request has its own
    instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring ApplicationContext.
    
    session
    Scopes a single bean definition to the lifecycle of an HTTP Session. 
    Only valid in the context of a web-aware Spring ApplicationContext.
    
    application
    Scopes a single bean definition to the lifecycle of a ServletContext. Only valid in the context of a web-aware Spring ApplicationContext.
    
    websocket
    Scopes a single bean definition to the lifecycle of a WebSocket. Only valid in the context of a web-aware Spring ApplicationContext

    老版本:
    Singleton: Only one instance will be created for a single bean definition per Spring IoC container and the same object will be shared for each request made for that bean.
    Prototype: A new instance will be created for a single bean definition every time a request is made for that bean.
    Request: A new instance will be created for a single bean definition every time an HTTP request is made for that bean. But Only valid in the context of a web-aware Spring ApplicationContext.
    Session: Scopes a single bean definition to the lifecycle of an HTTP Session. But Only valid in the context of a web-aware Spring ApplicationContext.
    Global-Session: Scopes a single bean definition to the lifecycle of a global HTTP Session. It is also only valid in the context of a web-aware Spring ApplicationContext.


    