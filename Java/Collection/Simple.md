#简单集合

----------------------------------------------

ArrayList

    ArrayList实现了List, RandomAccess, Cloneable, java.io.Serializable等接口。

    ArrayList实现了List，提供了基础的添加、删除、遍历等操作。
    
    ArrayList实现了RandomAccess，提供了随机访问的能力。
    
    ArrayList实现了Cloneable，可以被克隆。
    
    ArrayList实现了Serializable，可以被序列化。

    默认容量为10，也就是通过new ArrayList()创建时的默认容量。
    transient Object[] elementData;

    为什么用transient?
    答:elementData定义为transient的优势，自己根据size序列化真实的元素，而不是根据数组的长度序列化元素，减少了空间占用。

    如何扩容?
    新容量是老容量的1.5倍（oldCapacity + (oldCapacity >> 1)）
    如果加了这么多容量发现比需要的容量还小 则以需要的容量为准；



CopyOnWriteArrayList

    CopyOnWriteArrayList是ArrayList的线程安全版本，内部也是通过数组实现
    每次对数组的修改都完全拷贝一份新的数组来修改  修改完了再替换掉老数组
    这样保证了只阻塞写操作 不阻塞读操作 实现读写分离。

    //修改时使用
    final transient ReentrantLock lock = new ReentrantLock();
    
     public void add(int index, E element) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            if (index > len || index < 0)
                throw new IndexOutOfBoundsException("Index: "+index+
                                                    ", Size: "+len);
            Object[] newElements;
            int numMoved = len - index;
            if (numMoved == 0)
                newElements = Arrays.copyOf(elements, len + 1);
            else {
                newElements = new Object[len + 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index, newElements, index + 1,
                                 numMoved);
            }
            newElements[index] = element;
            setArray(newElements);
        } finally {
            lock.unlock();
        }
    }


HashMap

    （1）容量
    容量为数组的长度，亦即桶的个数，默认为16，最大为2的30次方，当容量达到64时才可以树化。
    （2）装载因子
    装载因子用来计算容量达到多少时才进行扩容，默认装载因子为0.75。
    （3）树化
    树化，当容量达到64且链表的长度达到8时进行树化，当链表的长度小于6时反树化。



LinkedHashMap

    LinkedHashMap内部维护了一个双向链表，能保证元素按插入的顺序访问，
    也能以访问顺序访问，可以用来实现LRU缓存策略。
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
    
    /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;

    开启按顺序访问后:
    末尾为最新访问的元素 ,头部为最久未访问的


WeakHashMap

    WeakHashMap 没有红黑树了.
    WeakHashMap是一种弱引用map，内部的key会存储为弱引用，当jvm gc的时候，
    如果这些key没有强引用存在的话，会被gc回收掉，下一次当操作map的时候会把对应的Entry整个删除掉，
    基于这种特性，WeakHashMap特别适用于缓存处理。
    
    WeakHashMap的key是“弱键”，是WeakReference类型的；ReferenceQueue是一个队列，它会保存被GC回收的“弱键”。

    ReferenceQueue<Object> queue 用于存失效的Entry

    存储结构:数组加链表 
    
    节点:
    private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
        V value;
        final int hash;
        Entry<K,V> next;

        /**
         * Creates new entry.
         */
        Entry(Object key, V value,
              ReferenceQueue<Object> queue,
              int hash, Entry<K,V> next) {
            super(key, queue);
            this.value = value;
            this.hash  = hash;
            this.next  = next;
        }

ConcurrentHashMap

    并发安全


TreeMap

    纯红黑树

ConcurrentSkipListMap

    基于跳表
    数据结构: 
    （1）Node，数据节点，存储数据的节点，典型的单链表结构；
    （2）Index，索引节点，存储着对应的node值，及向下和向右的索引指针；
    （3）HeadIndex，头索引节点，继承自Index，并扩展一个level字段，用于记录索引的层级
    
HashSet

    基于HashMap value 放的是new Object();
    key可以有一个Null

LinkedHashSet
    
    基于LinkedHashMap

TreeSet 

    基于NavigableMap(跟TreeMap实现差不多)

CopyOnWriteArraySet

    CopyOnWriteArraySet底层是使用CopyOnWriteArrayList存储元素的
    添加元素时调用了CopyOnWriteArrayList的addIfAbsent()方法来保证元素不重复
    先检查有没有 没有再添加


ConcurrentSkipListSet
    
    ConcurrentSkipListSet底层是通过ConcurrentNavigableMap来实现的
    它是一个有序的线程安全的集合

    ConcurrentSkipListSet有序的 基于元素的自然排序或者通过比较器确定的顺序


LinkedList

    双向链表


PriorityQueue

    transient Object[] queue

    PriorityQueue是一个小顶堆 
    1）当数组比较小（小于64）的时候每次扩容容量翻倍；
    2）当数组比较大的时候每次扩容只增加一半的容量；

    poll:
    弹出堆顶 将堆尾数据挪到上面 向下调整
    offer:
    插入到尾部 向上调整

