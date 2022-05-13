# comparable 和 comparator 的区别

Comparable & Comparator 都是用来实现集合中元素的比较、排序的;
Comparable 是在集合内部定义的方法实现的排序，
Comparator 是在集合外部实现的排序，
所以，如想实现排序，就需要在集合外定义 Comparator 接口的方法或在集合内实现 Comparable 接口的方法。

## Comparable 定义
```java
package java.lang;
import java.util.*;

public interface Comparable<T> {
    public int compareTo(T o);
}
```

假设我们通过 x.compareTo(y) 来“比较x和y的大小”。若返回“负数”，意味着“x比y小”；返回“零”，意味着“x等于y”；返回“正数”，意味着“x大于y”。

## comparator

Comparator 策略模式（strategy design pattern）
用 Comparator 是 策略模式（strategy design pattern），就是不改变对象自身，而用一个策略对象（strategy object）来改变它的行为。

Comparator 定义
```java
package java.util;

public interface Comparator<T> {

    int compare(T o1, T o2);

    boolean equals(Object obj);
}
```

1、 若一个类要实现Comparator接口：它一定要实现compare(T o1, T o2) 函数，但可以不实现 equals(Object obj) 函数。

为什么可以不实现 equals(Object obj) 函数呢？ 因为任何类，默认都是已经实现了equals(Object obj)的。 Java中的一切类都是继承于java.lang.Object，在Object.java中实现了equals(Object obj)函数；所以，其它所有的类也相当于都实现了该函数。

2、 int compare(T o1, T o2) 是“比较o1和o2的大小”。返回“负数”，意味着“o1比o2小”；返回“零”，意味着“o1等于o2”；返回“正数”，意味着“o1大于o2”。

## 举例
Comparable 例子
```java
package thread.comparable;

import java.util.ArrayList;
import java.util.Collections;

public class Person implements Comparable {
    private int age;
    private String name;

    public Person(String name,int age) {
        this.age = age;
        this.name = name;
    }

    @Override
    public int compareTo(Object o) {
        if (o instanceof Person) {
            Person person = (Person) o;
            int result;
            result = getAge() - person.getAge();
            //result =person.getAge() -  getAge();
            return result;
        }
        return 0;
    }

    @Override
    public String toString() {
        return "Person{" +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }


    public static void main(String[] args) {
        ArrayList<Person> list = new ArrayList<Person>();
        list.add(new Person("ccc", 20));
        list.add(new Person("AAA", 30));
        list.add(new Person("bbb", 10));
        list.add(new Person("ddd", 40));
        // 打印list的原始序列
        System.out.printf("Original sort, list:%s\n", list);
        Collections.sort(list);
        System.out.printf("Original sort, list:%s\n", list);
    }
}
```


Comparator 例子

```java
import java.util.ArrayList;
import java.util.Comparator;

public class Person {
    private int age;
    private String name;

    public Person(String name, int age) {
        this.age = age;
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }

}

class AseAgePerson implements Comparator<Person> {


    @Override
    public int compare(Person o1, Person o2) {
        int result;
        result = o1.getAge() - o2.getAge();
        return result;
    }


}

class DescAgePerson implements Comparator<Person> {


    @Override
    public int compare(Person o1, Person o2) {
        int result;
        result = o2.getAge() - o1.getAge();
        return result;
    }


}

class Test {

    public static void main(String[] args) {
        ArrayList<Person> list = new ArrayList<>();
        list.add(new Person("ccc", 20));
        list.add(new Person("AAA", 30));
        list.add(new Person("bbb", 10));
        list.add(new Person("ddd", 40));
        // 打印list的原始序列
        System.out.println(list);
        list.sort(new AseAgePerson());
        //list.sort(new DescAgePerson());
        System.out.println(list);
    }
}
```
