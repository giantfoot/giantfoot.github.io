#### Stack 解析

Stack，栈，特点是先进后出(FILO, First In Last Out)。

继承Vector，并额外提供了push、pop、peek、empty、search这几个方法。

search(Object)
```
/**
 * 栈底向栈顶方向遍历，查找指定对象o在栈中的位置。
 * @param   o   指定对象
 * @return  o的索引，如果没找到，返回-1
 */
public synchronized int search(Object o) {
    int i = lastIndexOf(o);

    if (i >= 0) {
        return size() - i;
    }
    return -1;
}
```
