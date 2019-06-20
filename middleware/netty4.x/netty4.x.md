### NettyChannel
![](resources/middleware/netty/netty_channel.png)

### Selector & SelectionKey
```java
// netty 实现了 SelectedSelectionKeySet 替换了 SelectorImpl#selectedKeys 和 SelectorImpl#publicSelectedKeys
public abstract class SelectorImpl extends AbstractSelector {
    protected Set<SelectionKey> selectedKeys = new HashSet();
    protected HashSet<SelectionKey> keys = new HashSet();
    private Set<SelectionKey> publicKeys;
    private Set<SelectionKey> publicSelectedKeys;
    
     /**
      *  省略部分代码
      */
}
```
SelectedSelectionKeySet 以数组的方式实现，将时间复杂度降为O(1)。  
```java
final class SelectedSelectionKeySet extends AbstractSet<SelectionKey> {

    SelectionKey[] keys;
    int size;

    SelectedSelectionKeySet() {
        keys = new SelectionKey[1024];
    }

    @Override
    public boolean add(SelectionKey o) {
        if (o == null) {
            return false;
        }

        keys[size++] = o;
        if (size == keys.length) {
            increaseCapacity();
        }

        return true;
    }

    @Override
    public int size() {
        return size;
    }
    
    /**
     *  省略部分代码
     */

    void reset() {
        reset(0);
    }

    void reset(int start) {
        Arrays.fill(keys, start, size, null);
        size = 0;
    }

    private void increaseCapacity() {
        SelectionKey[] newKeys = new SelectionKey[keys.length << 1];
        System.arraycopy(keys, 0, newKeys, 0, size);
        keys = newKeys;
    }
}
```
netty 还装饰了 Selector。原生 NIO api 遍历 selector.selectedKeys()时，每次处理一个事件都需要remove，而 netty 装饰的 SelectedSelectionKeySetSelector每次select之前就会清空上次的事件重新填充。
```java
final class SelectedSelectionKeySetSelector extends Selector {
    private final SelectedSelectionKeySet selectionKeys;
    private final Selector delegate;

    SelectedSelectionKeySetSelector(Selector delegate, SelectedSelectionKeySet selectionKeys) {
        this.delegate = delegate;
        this.selectionKeys = selectionKeys;
    }

    /**
     *  省略部分代码
     */

    @Override
    public int selectNow() throws IOException {
        selectionKeys.reset();
        return delegate.selectNow();
    }

    @Override
    public int select(long timeout) throws IOException {
        selectionKeys.reset();
        return delegate.select(timeout);
    }

    @Override
    public int select() throws IOException {
        selectionKeys.reset();
        return delegate.select();
    }

    /**
     *  省略部分代码
     */
}
```