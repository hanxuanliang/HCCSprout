# ArrayList/Vector 的底层分析

## ArrayList

我们先看看源码的顶部解释可以总结一下几点：

- **底层**：ArrayList是List接口的大小可变数组的实现。（`length <= capacity`）
- **Null**：ArrayList允许null元素。
- **时间复杂度**：`size、isEmpty、get、set、iterator和listIterator` 为O(1)。`add，remove` 需要O(n)。与用于LinkedList实现的常数因子相比，此实现的常数因子较低。
- **容量**：ArrayList的容量可以自动增长。（有增长策略）
- **同步**：ArrayList不是同步的。（Vector是同步的）
- **迭代器**：ArrayList的iterator和listIterator方法返回的迭代器是fail-fast的。

>如果多个线程同时访问一个ArrayList实例，而其中至少一个线程从结构上修改了列表，那么它必须保持外部同步。（结构上的修改是指任何添加或删除一个或多个元素的操作，或者显式调整底层数组的大小；仅仅设置元素的值不是结构上的修改。）这一般通过对自然封装该列表的对象进行同步操作来完成。如果不存在这样的对象，则应该使用 `Collections.synchronizedList` 方法将该列表 `“包装”` 起来。这最好在创建时完成，以防止意外对列表进行不同步的访问：
>
>`List list = Collections.synchronizedList(new ArrayList(…));`

> 此类的iterator和listIterator方法返回的迭代器是 `快速失败` 的：在创建迭代器之后，除非通过迭代器自身的remove或add方法从结构上对列表进行修改，否则在任何时间以任何方式对列表进行修改，迭代器都会抛出`ConcurrentModificationException`。因此，面对并发的修改，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

### 定义

```java
public class ArrayList<E> extends AbstractList<E>
	implements List<E>, RandomAccess, Cloneable, java.io.Serializable {}
```

- `implements RandomAccess` ：表明ArrayList支持快速（通常是固定时间）随机访问。此接口的主要目的是允许一般的算法更改其行为，从而在将其应用到随机或连续访问列表时能提供良好的性能。
- `implements Cloneable`：表明其可以调用clone()方法来返回实例的field-for-field拷贝。
- `implements java.io.Serializable`：表明该类具有序列化功能。

### 属性

```java
/**
 * 初始化默认容量
 */
private static final int DEFAULT_CAPACITY = 10;

/**
 * 指定该ArrayList容量为0时，返回该空数组
 */
private static final Object[] EMPTY_ELEMENTDATA = {};

/**
 * 当调用无参构造方法，返回的是该数组。刚创建一个ArrayList 时，其内数据量为0。
 * 它与EMPTY_ELEMENTDATA的区别就是：该数组是默认返回的（调用无参构造方法），
 * 而后者是在用户指定容量为0时返回
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 保存添加到ArrayList中的元素。ArrayList的容量就是该数组的长度。
 * 该值为DEFAULTCAPACITY_EMPTY_ELEMENTDATA 时，当第一次添加元素进入ArrayList中时，
 * 数组将扩容值DEFAULT_CAPACITY。
 * 被标记为transient，在对象被序列化的时候不会被序列化。
 */
transient Object[] elementData; // non-private to simplify nested class access

/**
 * ArrayList的实际大小（数组包含的元素个数）
 * @serial
 */
private int size;
```

但是**elementData被标记为transient，那么它的序列化和反序列化是如何实现的呢？**

### 构造方法

#### **ArrayList( int initialCapacity)**

```java
 /**
  * 构造一个指定初始化容量为capacity的空ArrayList。
  *
  * @param  initialCapacity  ArrayList的指定初始化容量
  * @throws IllegalArgumentException  如果ArrayList的指定初始化容量为负。
  */
 public ArrayList(int initialCapacity) {
	if (initialCapacity > 0) {
		this.elementData = new Object[initialCapacity];
	} else if (initialCapacity == 0) {
		this.elementData = EMPTY_ELEMENTDATA;
	} else {
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
	}
  }
```

#### **ArrayList()**

```java
 /**
  * 构造一个初始容量为 10 的空列表。
  */
 public ArrayList() {
	this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
 }
```

#### **ArrayList(Collection<? extends E> c)**

```java
/**
 * 构造一个包含指定 collection 的元素的列表，这些元素是按照该 collection 的迭代器返回它们的顺序排列的。
 *
 * @param c 其元素将放置在此列表中的 collection
 * @throws NullPointerException 如果指定的 collection 为 null
 */
 public ArrayList(Collection<? extends E> c) {
	elementData = c.toArray();
	if ((size = elementData.length) != 0) {
		// c.toArray might (incorrectly) not return Object[] (see 6260652)
		if (elementData.getClass() != Object[].class)
			elementData = Arrays.copyOf(elementData, size, Object[].class);
	} else {
		// replace with empty array.
		this.elementData = EMPTY_ELEMENTDATA;
	}
 }
```



## Vector