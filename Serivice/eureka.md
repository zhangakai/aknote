#注册中心模块

----------------------------------------------

1.实现

    1.在pom文件中引入依赖
    <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>

    2.sever配置
    server:
        port: 7001
    eureka:
        instance:
        hostname: 127.0.0.1
    client:
        register-with-eureka: false   #是否将自己注册到eureka中
        fetch-registry: false         #是否从eureka中获取信息
        service-url:
            defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

    加注解@EnableEurekaServer

    3.client配置
    <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>

    eureka:
        client:
        service-url:
            defaultZone: http://127.0.0.1:7001/eureka
        instance:
            prefer-ip-address: true

    4.高可用
    服务中心相互注册 Eureka server的高可用实际上就是将自己作为服务向其他服务注册中心注册自己,
    这样就会形成一组互相注册的服务注册中心,进而实现服务清单的互相同步,往注册中心A上注册的服务,
    可以被复制同步到注册中心B上,所以从任何一台注册中心上都能查询到已经注册的服务,从而达到高可用的效果。
    在sever的service-url:中写入另一个注册中心的地址
    在client中的service-url中写入sever的所有地址
    
    5.一些参数
    eureka.client.registry-fetch-interval-seconds 多久从服务端获取一次服务信息
    lease-renewal-interval-in-seconds: 心跳
    

2.原理

    1.三个角色
    Eureka Server：提供服务注册和发现，多个Eureka Server之间会同步数据，做到状态一致（最终一致性）
    Service Provider：服务提供方，将自身服务注册到Eureka，从而使服务消费方能够找到
    Service Consumer：服务消费方，从Eureka获取注册服务列表，从而能够消费服务

    2.自我保护机制
    假设在某种特定的情况下（如网络故障）, Eureka Client和Eureka Server无法进行通信，
    此时Eureka Client无法向Eureka Server发起注册和续约请求，Eureka Server中就可能
    因注册表中的服务实例租约出现大量过期而面临被剔除的危险，然而此时的Eureka Client可能
    是处于健康状态的（可接受服务访问），如果直接将注册表中大量过期的服务实例租约剔除显然
    是不合理的，自我保护机制提高了eureka的服务可用性。
    当自我保护机制触发时，Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务，
    仍能查询服务信息并且接受新服务注册请求，也就是其他功能是正常的。

    3.数据同步
    采用异步数据同步 保持最终一致性 提供的功能包括：服务注册、接收服务心跳、服务剔除、服务下线等。
    
    4.优缺点
    缺点:eureka consumer本身有缓存，服务状态更新滞后，最常见的状况就是，服务下线了但是服务消费者
    还未及时感知，此时调用到已下线服务会导致请求失败，只能依靠consumer端的容错机制来保证。
    优点:Eureka 的设计原则是 AP，即可用性和分区容错性。他保证了注册中心的可用性，但舍弃了数据一致性，
    各节点上的数据有可能是不一致的（会最终一致）。
    
3.待续。。。。