#### LinkedList 解析

LinkedList添加了可以使其用作栈、队列或双端队列的方法。

LinkedList是List接口和Deque接口的双向链表实现。LinkedList实现了所有的列表操作，允许所有的元素（包括空元素）。

```
public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

在Java中，LinkedList的迭代器有三种：Iterator、ListIterator、DescendingIterator。其中DescendingIterator与Iterator只能单向遍历，遍历的方向相反。而ListIterator可以双向遍历。
```
/**
    * Adapter to provide descending iterators via ListItr.previous
    */
   private class DescendingIterator implements Iterator<E> {
       private final ListItr itr = new ListItr(size());
       public boolean hasNext() {
           return itr.hasPrevious();
       }
       public E next() {
           return itr.previous();
       }
       public void remove() {
           itr.remove();
       }
   }
```

Node
```
private static class Node<E> {
       E item;
       Node<E> next;
       Node<E> prev;

       Node(Node<E> prev, E element, Node<E> next) {
           this.item = element;
           this.next = next;
           this.prev = prev;
       }
   }
```
