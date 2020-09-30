[toc]
实现了Queue接口，并没有实现BlockingQueue接口，所以它不是阻塞队列，也不能用于线程池中，但是它是线程安全的，可用于多线程环境中。无界队列

###成员变量
```java
private transient volatile Node<E> head;//头结点
private transient volatile Node<E> tail;//尾节点，指向最后一个节点或者倒数第二个节点
private static final sun.misc.Unsafe UNSAFE;//操作cas的类
private static final long headOffset;
private static final long tailOffset;
private static class Node<E> {
    volatile E item;
    volatile Node<E> next;//单链表
}
```
###构造器
```java
public ConcurrentLinkedQueue() {    
    head = tail = new Node<E>(null);// 初始化头尾节点,指向同一个对象
}
public ConcurrentLinkedQueue(Collection<? extends E> c) {
    Node<E> h = null, t = null;    
    for (E e : c) {// 遍历c，并把它元素全部添加到单链表中
        checkNotNull(e);
        Node<E> newNode = new Node<E>(e);
        if (h == null)
            h = t = newNode;
        else {
            t.lazySetNext(newNode);//不保证可见性的方式提升性能
            t = newNode;
        }
    }
    if (h == null)
        h = t = new Node<E>(null);
    head = h;
    tail = t;
}
```
###入队
```java
public boolean add(E e) {
    return offer(e);
}
// 入队到链表尾
public boolean offer(E e) {
    checkNotNull(e);// 不能添加空元素
    final Node<E> newNode = new Node<E>(e); // 创建新节点   
    for (Node<E> t = tail, p = t;;) { //定义了两个变量  t，p，自旋
        Node<E> q = p.next;        
        if (q == null) { // 如果没有next，说明到链表尾部了（tail指向最后一个元素），就入队                        
334         if (p.casNext(null, newNode)) {//CAS更新p.next为新节点,如果成功了,就返回true,如果不成功就自旋
                //如果p不等于t，说明有其它线程先一步更新tail, 也就不会走到q==null这个分支了,p取到的可能是t后面的值
                if (p != t) 
                    casTail(t, newNode);  // 把tail原子更新为新节点,失败也ok（指向倒数第二）
340             return true;// 返回入队成功
            }
        }
        else if (p == q)     
            p = (t != (t = tail)) ? t : head;// 如果p的next等于p，说明p已经被删除了（已经出队了）,重新设置p的值
        else 
            p = (p != t && t != (t = tail)) ? t : q;// t后面还有值，重新设置p的值
    }
}
```
变量t代表的是tail节点，p用来表示队列的尾节点，初始情况下等于tail节点，q指向p的next判断是不是尾节点的依据是next是不是null

第一次添加元素：p，t 都指向tail，q为null，执行334行代码，cas成功则返回true，此时p\==t 直接执行340行，循环终止，返回为false,则说明被其他线程抢先添加了节点，此时进入下一轮循环：p，t 的值都未变，指向tail，但q不为null而是真正的尾节点，p != q，进入else的逻辑，此时p\==t（三目表达式为false），p赋值为q，p指向真正的尾节点，进入下一轮循环，q为null，重复上面第一步的逻辑，一直到cas成功返回true为止，
第二次添加元素：此时tail指向倒数第二个元素，p，t都指向tail，q指向最后一个节点，执行else逻辑，p\==t（三目表达式为false），p赋值为q，p指向真正的尾节点，进入下一轮循环，q为null，cas成功：则执行336行 此时p != t（t执行tail，p指向最后一个节点），执行casTail，将 t 指向新节点（tail也指向新节点），casTail执行失败也没关系（说明有另外一个线程设置了tail节点），并结束循环。 tail指向最后一个节点，cas失败：则说明被其他线程抢先添加了节点，此时进入下一轮循环，q指向最后一个节点（p指向倒数第二个），进入else，p !=t成立，t !=tail不成立（如果成立说明tail节点已经指向了新节点，p赋值为tail），p被赋值为q。p都指向最后一个节点p==q  ： (这里其实又是另外一种场景)说明p指向的节点的next也指向它自己，这种节点称之为哨兵节点，这种节点在队列中存在的价值不大，一般表示为要删除的节点或者是空节点tail没有一直指向尾节点的原因：固定指向尾节点的话，每次放入元素都需要移动tail，需要一次cas操作。不固定指向的话，根据tail找到尾节点，可以节省移动tail操作，1.8版本的jdk为offer两次移动一次tail（1.7版本为一个常量值，可配），根据tail遍历找到尾节点比cas操作效率要高
###出队
```java
public E remove() {
    E x = poll();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}
public E poll() {
    restartFromHead:
    for (;;) {
        for (Node<E> h = head, p = h, q;;) {
            E item = p.item;// 尝试弹出链表的头节点     
            if (item != null && p.casItem(item, null)) {// 如果节点的值不为空，并且将其更新为null成功了
                // 如果头节点变了，则不会走到这个分支
                // 会先走下面的分支拿到新的头节点
                // 这时候p就不等于h了，就更新头节点
                // 在updateHead()中会把head更新为新节点
                // 并让head的next指向其自己
                if (p != h) // hop two nodes at a time
                    updateHead(h, ((q = p.next) != null) ? q : p);
                // 上面的casItem()成功，就可以返回出队的元素了
                return item;
            }
            // 下面三个分支说明头节点变了
            // 且p的item肯定为null
            else if ((q = p.next) == null) {
                // 如果p的next为空，说明队列中没有元素了
                // 更新h为p，也就是空元素的节点
                updateHead(h, p);
                // 返回null
                return null;
            }
            else if (p == q)
                // 如果p等于p的next，说明p已经出队了，重试
                continue restartFromHead;
            else
                // 将p设置为p的next
                p = q;
        }
    }
}
// 更新头节点的方法
final void updateHead(Node<E> h, Node<E> p) {
    // 原子更新h为p成功后，延迟更新h的next为它自己
    // 这里用延迟更新是安全的，因为head节点已经变了
    // 只要入队出队的时候检查head有没有变化就行了，跟它的next关系不大
    if (h != p && casHead(h, p))
        h.lazySetNext(h);
}
```
出队的整个逻辑比较清晰：
（1）定位到头节点，尝试更新其值为null；
（2）如果成功了，就成功出队；
（3）如果失败或者头节点变化了，就重新寻找头节点，并重试；
（4）整个出队过程没有一点阻塞相关的代码，所以出队的时候不会阻塞线程，没找到元素就返回null；
###ConcurrentLinkedQueue与LinkedBlockingQueue对比
（1）两者都是线程安全的队列；
（2）两者都可以实现取元素时队列为空直接返回null，后者的poll()方法可以实现此功能；
（3）前者全程无锁，后者全部都是使用重入锁控制的；
（4）前者效率较高，后者效率较低；
（5）前者无法实现如果队列为空等待元素到来的操作；
（6）前者是非阻塞队列，后者是阻塞队列；
（7）前者无法用在线程池中，后者可以；