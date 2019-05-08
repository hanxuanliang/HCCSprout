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
 * Default initial capacity.
 */
private static final int DEFAULT_CAPACITY = 10;

/**
 * Shared empty array instance used for empty instances.
 */
private static final Object[] EMPTY_ELEMENTDATA = {};

/**
 * Shared empty array instance used for default sized empty instances. We
 * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
 * first element is added.
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * The array buffer into which the elements of the ArrayList are stored.
 * The capacity of the ArrayList is the length of this array buffer. Any
 * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
 * will be expanded to DEFAULT_CAPACITY when the first element is added.
 */
transient Object[] elementData; // non-private to simplify nested class access

/**
 * The size of the ArrayList (the number of elements it contains).
 *
 * @serial
 */
private int size;
```



## Vector