## 重要属性

基于jdk 1.8

```java
/**
 * The default initial capacity - MUST be a power of two.
 * 默认初始容量
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
* 最大容量 2^30 次方
*/
    static final int MAXIMUM_CAPACITY = 1 << 30;
/**
* 默认加载因子
*/  
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
/**
*  变树长度
*/ 
    static final int TREEIFY_THRESHOLD = 8;
/**
*  由树变链表长度
*/
    static final int UNTREEIFY_THRESHOLD = 6;
/**
*  变树的最小容量
*/
    static final int MIN_TREEIFY_CAPACITY = 64;
/**
*  节点数组
*/
    transient Node<K,V>[] table;
/**
*  hashmap大小
*/
    transient int size;
/**
*  hashmap修改次数，在遍历Itrator时做fast-fail，ConcurrentModificationException
*/
    transient int modCount;
/**
*   扩容阈值
*/
    int threshold;

```



## 重要方法

```java
/**
*  key 的hashcode 异或 右移16位，使高16位参与到hash算法中 ，其中1.7版本实现较为复杂，需要进行多次的移位和异或操作。
*/
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```





```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```



```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))  //key 先比较==  在比较equals
            e = p;
        else if (p instanceof TreeNode)                               // treenode 走红黑树的put方法
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);  
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st   到达8个进行树化操作
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key    存在对象根据条件判断是否重新赋值，赋值的话替换老的valvue即可
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```





```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {  // 超过最大容量 ，不报错，不扩容，返回原来的table数组
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) { // 肯定hash值小于容量，扩容之后位置不变
                            if (loTail == null)    //位置不变的分成一个数组
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        } 
                        else {                      // 位置变得分成一个数组
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {        //  不变的在新数组原来的位置
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {       // 变得数组位置加 老的数组容量
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```



## 1.7和1.8区别 



jdk 1.8   先放数据，然后判断是否需要扩容。逻辑上个简单

jdk1.7  每次放数据之前先判断是否需要扩容，先扩容，在存放数据。



## 要注意的点

key原则：String Integer 比较好，不可变对象，hashcode 不会变化。string 对hashcode有缓存，效率更高。



## 引出的问题

hashcode equals 方法是否需要重写。