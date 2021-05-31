#ConcurrentHashMap

---------------------------------

字段

    最大容量
    private static final int MAXIMUM_CAPACITY = 1 << 30;

    默认容量
    private static final int DEFAULT_CAPACITY = 16;

    负载因子
    private static final float LOAD_FACTOR = 0.75f;

    树化门槛
    static final int TREEIFY_THRESHOLD = 8;

    树变链表门槛
    static final int UNTREEIFY_THRESHOLD = 6;

    容量到达此值才树化(即capacity>64且一个桶里面的链表长度大于8)
    static final int MIN_TREEIFY_CAPACITY = 64;

    扩容搬数据的时候一个线程至少负责多少桶 例如现在table[64] 扩容时每个线程取16个桶搬
    private static final int MIN_TRANSFER_STRIDE = 16;

    sizeCtl的邮戳
    private static int RESIZE_STAMP_BITS = 16;

    协程resize的最大线程数
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

    邮戳移位数
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

    /*
     * Encodings for Node hash fields. See above for explanation.
     */
    static final int MOVED     = -1; // hash for forwarding nodes
    static final int TREEBIN   = -2; // hash for roots of trees
    static final int RESERVED  = -3; // hash for transient reservations
    static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash

    /** Number of CPUS, to place bounds on some sizings */
    static final int NCPU = Runtime.getRuntime().availableProcessors();


结构


     static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }