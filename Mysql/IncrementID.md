#自增ID

---------------------------------

    AUTO_INCREMENT 属性可用于为新行生成唯一标识。当向表中插入新记录，并且 auto_increment 字段为 NULL 或 DEFAULT 时，该值将自动增加。

    AUTO_INCREMENT 列默认从 1 开始。自动生成的值永远不能低于 0。
    
    每个表只能有一个 AUTO_INCREMENT 列。它必须定义为一个键（不一定是 PRIMARY KEY 或 UNIQUE 键）。在一些存储引擎中（包括默认的 InnoDB），
    
    如果 key 由多列组成，则 AUTO_INCREMENT 列必须是第一列。


    在5.几版本时代 auto_increment存放在内存中 每次重启会从1开始 
    后面将此值持久化了。

    可以为 AUTO_INCREMENT 列指定一个值。如果键是主键或唯一键，则键中不得存在该值。

    如果新值高于当前最大值，则更新 AUTO_INCREMENT 值，因此下一个值会更高。如果新值低于当前最大值，则 AUTO_INCREMENT 值保持不变。

    例如
    CREATE TABLE t (id INTEGER UNSIGNED AUTO_INCREMENT PRIMARY KEY) ENGINE = InnoDB;

    INSERT INTO t VALUES (NULL);
    SELECT id FROM t;
    +----+
    | id |
    +----+
    |  1 |
    +----+
    
    INSERT INTO t VALUES (10); -- higher value
    SELECT id FROM t;
    +----+
    | id |
    +----+
    |  1 |
    | 10 |
    +----+
    
    INSERT INTO t VALUES (2); -- lower value
    INSERT INTO t VALUES (NULL); -- auto value
    SELECT id FROM t;
    +----+
    | id |
    +----+
    |  1 |
    |  2 |
    | 10 |
    | 11 |
    +----+



---------------------------------------


    AUTO_INCREMENT 列通常有缺失值。发生这种情况是因为如果删除了一行，或者显式更新了 AUTO_INCREMENT 值，
    则永远不会重新使用旧值。 REPLACE 语句也删除了一行，它的值被浪费了。使用 InnoDB，值可以由事务保留；
    但如果事务失败（例如，由于 ROLLBACK），保留值将丢失