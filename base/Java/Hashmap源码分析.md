#Hashmap 源码分析

HashMap

HashMap 继承于AbstractMap，实现了 Map、Cloneable、java.io.Serializable 接口。

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    ......
}
```

HashMap 的结构是数组+链表形式组成。JDK 1.8 之后新增了红黑树的组成结构。因为红黑树具有快速增删改查的特点。

所以，HashMap 的结构是数组+链表/红黑树(链表大于 8 并且容量大于 64，是红黑树结构)。

其中数组就是 Node[] table ,哈希桶数组，是一个Node 的数组。

```java
static class Node<K,V> implements Map.Entry<K,V> {
   final int hash;
   final K key;
   V value;
   Node<K,V> next;
   Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
   }

   public final K getKey()        { return key; }
   public final V getValue()      { return value; }
   public final String toString() { return key + "=" + value; }

   public final int hashCode() {
      return Objects.hashCode(key) ^ Objects.hashCode(value);
   }

   public final V setValue(V newValue) {
       V oldValue = value;
       value = newValue;
       return oldValue;
   }
   ... ...
}
```

Node 就是HashMap 的一个内部类，本质就是键值对的映射，即每个键值对是存放在 Node 里的。Node 中的 hash 值，就是用来定位数组索引的位置。HashMap 的 key 取其 hash 值得到数组下标，然后把数据放在对应下标元素的链表上。

## 为什么使用链表+数组？

- 数组存储的值是 key 的 hash 值对数组长度取模。这样可以快速得到数组下标，然后得到哈希桶 Node[] ,即链表 。键值对存储在链表中。整个过程，对于数组来说，效率是最高的。

- 如果不用数组而使用 LinkedList 也是可以，但是查询速率还是数组的比较快。

- 如果使用 ArrayList ，底层也是数组，查找也很快，但是扩容时，数组扩容刚好是2的次幂，ArrayList的扩容机制是1.5倍扩容

## 若 Hash 冲突如何查询

当遇到 hash 冲突时，需要通过判断 key 值是否相等，即链表中每个 Node 的key 进行判断是否相等，相等就是我们要查询的，然后返回。可以看源代码中的实现：

```java
    final Node<K,V> getNode(Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n, hash; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & (hash = hash(key))]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

## 扩容机制

## HashMap 的并发问题

参考资料：

- https://cloud.tencent.com/developer/article/1491634

- https://tech.meituan.com/2016/06/24/java-hashmap.html

- https://blog.csdn.net/u014532901/article/details/78936283
