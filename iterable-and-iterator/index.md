# Iterable 인터페이스와 Iterator 인터페이스


## Iterable 인터페이스

![image](https://user-images.githubusercontent.com/46465928/158289000-134f2963-c98e-4f62-b164-26fa83f6c397.png)

Collection 인터페이스의 상위 인터페이스는 Iterable 이다.

```java
package java.lang;

public interface Iterable<T> {
    /**
     * Returns an iterator over elements of type {@code T}.
     *
     * @return an Iterator.
     */
    Iterator<T> iterator();

    /**
     * Performs the given action for each element of the {@code Iterable}
     * until all elements have been processed or the action throws an
     * exception.  Actions are performed in the order of iteration, if that
     * order is specified.  Exceptions thrown by the action are relayed to the
     * caller.
     * <p>
     * The behavior of this method is unspecified if the action performs
     * side-effects that modify the underlying source of elements, unless an
     * overriding class has specified a concurrent modification policy.
     *
     * @implSpec
     * <p>The default implementation behaves as if:
     * <pre>{@code
     *     for (T t : this)
     *         action.accept(t);
     * }</pre>
     *
     * @param action The action to be performed for each element
     * @throws NullPointerException if the specified action is null
     * @since 1.8
     */
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
}
```

Iterable 인터페이스 안에는 iterator 메소드가 추상메소드로 선언이 되어있다. 이렇게 때문에 Collection 인터페이스 계층구조에서 List, Set, Queue를 구현하는 클래스들은 다 iterator 메소드를 가지고 있다. 따라서 **Iterable 인터페이스는 iterator() 메소드를 하위 클래스에서 구현하도록 강제하기 위해 사용한다.**

## Iterator 인터페이스

![image](https://user-images.githubusercontent.com/46465928/156331538-22bba21a-94fd-47fb-8e73-dd7ef422b1b8.png)

Iterator 인터페이스는 Collection과는 별개로 존재하는 인터페이스이다.

```java
package java.util;

public interface Iterator<E> {
    /**
     * Returns {@code true} if the iteration has more elements.
     * (In other words, returns {@code true} if {@link #next} would
     * return an element rather than throwing an exception.)
     *
     * @return {@code true} if the iteration has more elements
     */
    boolean hasNext();

    /**
     * Returns the next element in the iteration.
     *
     * @return the next element in the iteration
     * @throws NoSuchElementException if the iteration has no more elements
     */
    E next();

    /**
     * Removes from the underlying collection the last element returned
     * by this iterator (optional operation).  This method can be called
     * only once per call to {@link #next}.
     * <p>
     * The behavior of an iterator is unspecified if the underlying collection
     * is modified while the iteration is in progress in any way other than by
     * calling this method, unless an overriding class has specified a
     * concurrent modification policy.
     * <p>
     * The behavior of an iterator is unspecified if this method is called
     * after a call to the {@link #forEachRemaining forEachRemaining} method.
     *
     * @implSpec
     * The default implementation throws an instance of
     * {@link UnsupportedOperationException} and performs no other action.
     *
     * @throws UnsupportedOperationException if the {@code remove}
     *         operation is not supported by this iterator
     *
     * @throws IllegalStateException if the {@code next} method has not
     *         yet been called, or the {@code remove} method has already
     *         been called after the last call to the {@code next}
     *         method
     */
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
}
```

Iterator 인터페이스를 구현함으로써 hasNext(), next(), remove() 등의 메소드를 이용할 수 있다. 따라서 **Iterable 인터페이스는 hasNext()와 next() 메소드를 하위 클래스에서 구현하도록 강제하기 위해 사용한다.**

```java
import java.util.Iterator;
import java.util.LinkedList;
 
public class Test {
    public static void main(String[] args) {
        LinkedList<String> list = new LinkedList<>();
        list.add("Lee"); list.add("ekk"); list.add("eww");
        Iterator it = list.iterator();
 
        while (it.hasNext()) {
            System.out.print(it.next() + " ");
        }
 
    }
}
```

이처럼 인터페이스를 사용하면 표준을 정의하고 구현을 강제함으로써 코드의 일관성을 유지할 수 있다.

## 참고
https://devlog-wjdrbs96.tistory.com/84

