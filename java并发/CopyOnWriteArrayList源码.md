[toc]
#1.特性
```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
```
特性基本与arrayList一致，底层也是数组结构

#2.基本属性
```java
private static final long serialVersionUID = 8673264195747942595L;//序列化版本号
//全局锁
final transient ReentrantLock lock = new ReentrantLock();
//存储数据的数组
private transient volatile Object[] array;
```

#3.构造器
```java
public CopyOnWriteArrayList() {
    setArray(new Object[0]);//创建一个大小为0的Object数组作为array初始值
}
public CopyOnWriteArrayList(E[] toCopyIn) {
    //创建一个list，其内部元素是toCopyIn的的副本
    setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
}
//将传入参数集合中的元素复制到本list中
public CopyOnWriteArrayList(Collection<? extends E> c) {
    Object[] elements;
    if (c.getClass() == CopyOnWriteArrayList.class)
        elements = ((CopyOnWriteArrayList<?>)c).getArray();
    else {
        elements = c.toArray();
        // c.toArray可能不是Object[]（比如：继承ArrayList，重写toArray方法返回String[]，
        //只有ArrayList的toArray方法实现是Arrays.copyOf，因此在jdk8中，此处改为了ArrayList.class）
        if (elements.getClass() != Object[].class)
            elements = Arrays.copyOf(elements, elements.length, Object[].class);
    }
    setArray(elements);
}
```

#4.添加元素
```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();//先加锁
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);//复制到新数组中，长度+1
        newElements[len] = e;//在新数组中添加元素
        setArray(newElements);//将新数组设置给array
        return true;
    } finally {
        lock.unlock();
    }
}
```

#5.指定位置添加元素
```java
public void add(int index, E element) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        // 获取旧数组
        Object[] elements = getArray();
        int len = elements.length;
        // 检查是否越界, 可以等于len
        if (index > len || index < 0)
            throw new IndexOutOfBoundsException("Index: "+index+
                                                ", Size: "+len);
        Object[] newElements;
        int numMoved = len - index;
        if (numMoved == 0)
            // 如果插入的位置是最后一位
            // 那么拷贝一个n+1的数组, 其前n个元素与旧数组一致
            newElements = Arrays.copyOf(elements, len + 1);
        else {
            // 如果插入的位置不是最后一位
            // 那么新建一个n+1的数组
            newElements = new Object[len + 1];
            // 拷贝旧数组前index的元素到新数组中
            System.arraycopy(elements, 0, newElements, 0, index);
            // 将index及其之后的元素往后挪一位拷贝到新数组中
            // 这样正好index位置是空出来的
            System.arraycopy(elements, index, newElements, index + 1,
                             numMoved);
        }
        // 将元素放置在index处
        newElements[index] = element;
        setArray(newElements);
    } finally {
        // 释放锁
        lock.unlock();
    }
}
addIfAbsent
//添加一个不存在于集合中的元素。
public boolean addIfAbsent(E e) {
    // 获取元素数组
    Object[] snapshot = getArray();
    //已存在返回false，否则添加
    return indexOf(e, snapshot, 0, snapshot.length) >= 0 ? false :
        addIfAbsent(e, snapshot);
}

private boolean addIfAbsent(E e, Object[] snapshot) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 重新获取旧数组
        Object[] current = getArray();
        int len = current.length;
        // 如果快照与刚获取的数组不一致，说明有修改
        if (snapshot != current) {
            // 重新检查元素是否在刚获取的数组里，减少indexOf的对比次数
            int common = Math.min(snapshot.length, len);
            for (int i = 0; i < common; i++)
                //判断是否有线程指定下标添加了元素
                if (current[i] != snapshot[i] && eq(e, current[i]))
                    return false;
            if (indexOf(e, current, common, len) >= 0)
                    return false;
        }
        // 拷贝一份n+1的数组
        Object[] newElements = Arrays.copyOf(current, len + 1);
        // 将元素放在最后一位
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```
#6.获取指定位置元素
```java
public E get(int index) {
    return get(getArray(), index);
}
final Object[] getArray() {
    return array;
}
//私有方法
private E get(Object[] a, int index) {
    return (E) a[index];
}
```
这个方法是线程不安全的，因为这个分成了两步，分别是获取数组和获取元素，而且中间过程没有加锁。假设当前线程在获取数组（执行getArray()）后，其他线程修改了这个CopyOnWriteArrayList，那么它里面的元素就会改变，但此时当前线程返回的仍然是旧的数组，所以返回的元素就不是最新的了，这就是写时复制策略产生的弱一致性问题。

#7.修改指定位置元素
```java
public E set(int index, E element) {
    final ReentrantLock lock = this.lock;
    lock.lock();    
    try {
        Object[] elements = getArray();
        E oldValue = get(elements, index);//先获取要修改的旧值
        if (oldValue != element) {//值确实需要修改
            int len = elements.length;
            //将array复制到新数组
            Object[] newElements = Arrays.copyOf(elements, len);            
            newElements[index] = element;//修改元素
            setArray(newElements);//设置array为新数组
        } else {
            // 虽然值不需要改，但要保证volatile语义，需重新设置array
            setArray(elements);
        }
        return oldValue;
    } finally {
        lock.unlock();
    }
}
```
#8.删除元素
```java
public E remove(int index) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        E oldValue = get(elements, index);//获取要删除的元素
        int numMoved = len - index - 1;
        if (numMoved == 0)//删除的是最后一个元素
            setArray(Arrays.copyOf(elements, len - 1));
        else {
            //将元素分两次复制到新数组中
            Object[] newElements = new Object[len - 1];
            System.arraycopy(elements, 0, newElements, 0, index);//拷贝index前面的元素
            System.arraycopy(elements, index + 1, newElements, index,numMoved);//拷贝index后面的元素
            setArray(newElements);
        }
        return oldValue;
    } finally {
        lock.unlock();
    }
}
```
如果在返回迭代器后没有对里面的数组array进行修改，则这两个变量指向的确实是同一个数组；但是若修改了，则根据前面所讲，它是会新建一个数组，然后将修改后的数组复制到新建的数组，而老的数组就会被“丢弃”，所以如果修改了数组，则此时snapshot指向的还是原来的数组，而array变量已经指向了新的修改后的数组了。这也就说明获取迭代器后，使用迭代器元素时，其他线程对该list的增删改不可见，因为他们操作的是两个不同的数组，这就是弱一致性。

CopyOnWriteArrayList使用写时复制策略保证list的一致性，而获取–修改–写入三个步骤不是原子性，所以需要一个独占锁保证修改数据时只有一个线程能够进行。另外，CopyOnWriteArrayList提供了弱一致性的迭代器，从而保证在获取迭代器后，其他线程对list的修改是不可见的，迭代器遍历的数组是一个快照。

**使用场景及优点**
并发容器用于读多写少的并发场景。比如白名单，黑名单等场景。

读操作可能会远远多于写操作的场景。比如，有些系统级别的信息，往往只需要加载或者修改很少的次数，但是会被系统内所有模块频繁的访问。对于这种场景，我们最希望看到的就是读操作可以尽可能的快，而写即使慢一些也没关系。

CopyOnWriteArrayList 的思想比读写锁的思想更进一步。为了将读取的性能发挥到极致，CopyOnWriteArrayList 读取是完全不用加锁的，更厉害的是，写入也不会阻塞读取操作，也就是说你可以在写入的同时进行读取，只有写入和写入之间需要进行同步，也就是不允许多个写入同时发生，但是在写入发生时允许读取同时发生。这样一来，读操作的性能就会大幅度提升。

**读写分离**

**缺点**
内存占用，弱一致性