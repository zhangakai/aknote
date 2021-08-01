#事务

------------------------------------------------------------------

事务的抽象
    
    spring的事务是逻辑事务，其底层还是依靠数据库事务特性（物理事务）。
    Spring事务由事务管理器、事务定义、事务本身组成。
    事务管理器接口 org.springframework.transaction.PlatformTransactionManager ：
    public interface PlatformTransactionManager {

    TransactionStatus getTransaction(
            TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
    }

    TransactionDefinition 接口定义了事务的状态:
    Isolation:5种 分别是默认(默认数据库用的啥 spring事务就用啥) 读提交 读未提交 可重复读 串行化
    Propagation:7种 事务的传播特性 
    Timeout:此事务在超时和被底层事务（数据库事务）自动回滚之前运行的时间
    Read-only Status:读取但不修改数据时，可以使用只读事务

    TransactionStatus 接口为事务代码提供了一种简单的方式来控制事务执行和查询事务状态。
    public interface TransactionStatus extends SavepointManager {

    boolean isNewTransaction();

    boolean hasSavepoint();

    void setRollbackOnly();

    boolean isRollbackOnly();

    void flush();

    boolean isCompleted();

    }




事务失效场景

    1.当没有使用基于接口的代理时，@Transactional用于接口或者接口方法时。--解决方案 proxyTargetClass=false(会启用基于
    2.在代理模式下（默认），只有通过代理对象进去的方法调用才会被拦截。用this.调用方法不会被拦截，因此会失效。
    假设一个类为user，它的代理类为userProxy，调用userProxy.update()才会被拦截，在外面包一层事务
    （ wrapped with transactions）。而用user.update不会被拦截。
    解决方案：设置mode=AspectJ mode
    3.抛出非RuntimeException的异常
    解决方案:@Transactional(rollbackFor = Exception.class)
    4.异常被提前catch住 （这不是废话吗）
    5.数据库不支持事务
    6.存在多个事务管理器 未明确指定使用哪个时，可能会有错误（比如应该用2 结果用了1） 


@Transactional

    默认规则:
    Propagation setting is PROPAGATION_REQUIRED.
    Isolation level is ISOLATION_DEFAULT.
    Transaction is read/write.
    Transaction timeout defaults to the default timeout of the underlying transaction system, or to none if timeouts are not supported.
    Any RuntimeException triggers rollback, and any checked Exception does not.

    propagation 有7种规则
    Required: 加入到当前事务，没有则新创建
    Required_new: 新创建事务，如果当前已存在事务，暂停此事务（通过保留对当前事务的引用，然后在当前线程上下文中清除此事务实现）
    Supports: 有事务就用，没有就不用
    Not_Supports: 有事务就暂停事务，
    Mandatory: 强制加入当前事务，没有就抛异常
    Never:有事务就报错
    Nested: 对于 NESTED 传播，Spring 检查事务是否存在，
    如果存在，则标记一个保存点。这意味着如果
    业务逻辑执行抛出异常，那么事务将回滚到这个保存点。
    如果没有活动事务，它的工作方式类似于 REQUIRED

    Isolation:5种 主要还是通过设置底层数据库中事务的隔离方式来实现
    

编程式使用

    1.Using the TransactionTemplate.
    public class SimpleService implements Service {

    private final TransactionTemplate transactionTemplate;

    public SimpleService(PlatformTransactionManager transactionManager) {
        Assert.notNull(transactionManager, "The 'transactionManager' argument must not be null.");
        this.transactionTemplate = new TransactionTemplate(transactionManager);

        // the transaction settings can be set here explicitly if so desired
        this.transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_UNCOMMITTED);
        this.transactionTemplate.setTimeout(30); // 30 seconds
        // and so forth...
    }
    }

    public Object someServiceMethod() {
        return transactionTemplate.execute(new TransactionCallback() {
            // the code in this method executes in a transactional context
            public Object doInTransaction(TransactionStatus status) {
                updateOperation1();
                return resultOfUpdateOperation2();
            }
        });
    }
    2.Using a PlatformTransactionManager implementation directly.
    DefaultTransactionDefinition def = new DefaultTransactionDefinition();
    // explicitly setting the transaction name is something that can only be done programmatically
    def.setName("SomeTxName");
    def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
    
    TransactionStatus status = txManager.getTransaction(def);
    try {
    // execute your business logic here
    }
    catch (MyException ex) {
    txManager.rollback(status);
    throw ex;
    }
    txManager.commit(status);

    
    
    