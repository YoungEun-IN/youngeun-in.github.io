# Comparable 인터페이스와 Comparator 인터페이스의 차이


## Comparable 인터페이스

JAVA에서는 아래의 인터페이스 구현을 통해 **정렬의 기준을 프로그래머가 직접 정의**할 것을 요구하고 있다.

```java
public interface Comparable<T> {
  int compareTo(T obj);
}
```

사용 예시는 다음과 같다.
```java
import java.util.Iterator;
import java.util.TreeSet;
class Person implements Comparable<Person> {
    String name;
    int age;
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public void showData() {
        System.out.printf("%s %d \n", name, age);
    }
    public int compareTo(Person p) {
        if (age > p.age)
            return 1;
        else if (age < p.age)
            return -1;
        else
            return 0;
    }
}
class ComparablePerson {
    public static void main(String[] args) {
        TreeSet<Person> sTree = new TreeSet<Person>();
        sTree.add(new Person("Lee", 24));
        sTree.add(new Person("Hong", 29));
        sTree.add(new Person("Choi", 21));
        Iterator<Person> itr = sTree.iterator();
        while (itr.hasNext())
            itr.next().showData();
    }
}
```

## Comparator 인터페이스

String 클래스는 이미 compareTo 메소드를 구현하고 있다. 이 때 정령기준을 변경하기 위해서 String 클래스를 상속하는 클래스를 새로 정의하는 것은 번거롭다
따라서 **새로운 정렬 기준을 파라미터로 넘길 수 있도록 정의**된 것이 Comparator<T> 인터페이스이다.
```java
public interface Comparator<T> {
  int compare(T o1, T o2);
}
```  

사용 예시는 다음과 같다.
```java
import java.util.TreeSet;
import java.util.Iterator;
import java.util.Comparator;
class StrLenComparator implements Comparator<String> {
    public int compare(String str1, String str2) {
        if (str1.length() > str2.length())
            return 1;
        else if (str1.length() < str2.length())
            return -1;
        else
            return 0;
        /*
         * return str1.length()-str2.length();
         */
    }
}
class IntroComparator {
    public static void main(String[] args) {
        TreeSet<String> tSet = new TreeSet<String>(new StrLenComparator());
        tSet.add("Orange");
        tSet.add("Apple");
        tSet.add("Dog");
        tSet.add("Individual");
        Iterator<String> itr = tSet.iterator();
        while (itr.hasNext())
            System.out.println(itr.next());
    }
}
```

