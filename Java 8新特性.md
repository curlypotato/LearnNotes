# Java 8新特性

## Java 8 有哪些新特性
##### 1、函数式编程
##### 2、Lambda表达式
##### 3、函数式接口
##### 4、内置的函数式接口
##### 5、Stream
##### 6、Parallel-Streams
##### 7、Map集合

## Java 8 新特性好处
* 速度更快
* 代码更少（增加了新的语法 Lambda 表达式）
* 强大的Stream API
* 便于并行
* 最大化减少空指针异常 Optional

## Java 8 新特性详细介绍

### 1、函数式编程

面向对象编程：面向对象的语言，一切皆对象，如果 想要调用一个函数，函数必须属于一个类或对旬，然后在使用类或对象进行调用。面向对象编程可能需要多写很多重复的代码行。

``` java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("do something...");
    }
}
```
函数式编程：在某些编程语言中，如 js、c++，我们可以直接写一个函数，然后在需要的时候进行调用，即函数式编程。

### 2、Lambda 表达式

在Java 8 以前，使用 `Collections` 的 sort 方法对字符串排序的写法：

``` java
List<String> names = Arrays.asList("dabin","tyson","sophia");

Collections.sort(names, new Comparator<String>() {
    @Overrid
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
```

Java 8 推荐使用 lambda 表达式，简化这种写法，如下代码：

``` java
List<String> names = Arrays.asList("dabin","tyson","sophia");

// 简化写法一：
Collections.sort(names, (String a, String b) -> b.compareTo(a));
// 简化写法二，省略入参类型，Java 编译器能够根据类型推断机制判断出参数类型
names.sort((a, b) -> b.compareTo(a));
```

可以看到使用 Lambda 表达式之后，代码变得很简短并且易于阅读。

### 3、函数式接口

Functional Interface：函数式接口，只包含一个抽象方法的接口。只有函数式接口才能缩写 Lambda 表达式。@FunctionalInterface 定义类为一个函数式接口，如果添加了第二个抽象方法，编译器会立刻抛出错误提示。

``` java
@FunctionalInterface
interface Concerter<F, T> {
    T convert(F from);
}

public class FunctionalInterfaceTest {
    public static void main(String[] args) {
        Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
        Integer converted = converter.convert("666");
        System.out.println(converted);
    }
}
```

```
输出结果
666
```

### 4、内置的函数式接口

Comparater 和 Runnable，Java 8 为他们都添加了 @FunctionalInterface 注解，以用来支持 Lambda 表达式。

#### Predicate 断言

指定入参类型，并返回 boolean 值的函数式接口。用来组合一个复杂的逻辑判断。

``` java
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("dabin"); // true
```


#### Comparator

Java 8 将 Comparator 升级成函数式接口，可以使用 Lambda 表达式简化代码。

``` java
public class ComparatorTest {
    
    public static void main(String[] args) {
        Comparator<Person> comparator = Comparator.comparing(p -> p.firstName);

        Person p1 = new Person("dabin", "wang");
        Person p2 = new Person("xiaobin", "wang");

        // 打印 -20
        System.out.println(comparator.compare(p1, p2));
        // 打印 20
        System.out.println(comparator.reversed().campare(p1, p2));
    }

}

class Person {

    public String firstName;
    public String lastNam;

    public Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

}
```

#### Consumer

Consumer 接口接收一个泛型参数，然后调用 accept，对这个参数做一系列消费操作。

Consumer 源码：

``` java
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }

}
```

示例1：

``` java
public class ConsumerTest {

    public static void main(String[] args) {
        Consumer<Integer> consumer = x -> {
            int a = x + 6;
            System.out.println(a);
            System.out.println("示例" + a);
        };
        consumer.accept(660);
    } 

}
```
```
 输出结果
 666
 示例666
```

示例2：在stream里，对入参做一些操作，主要是用于forEach，对传入的参数，做一系列的业务操作。

``` java
// CopyOnWriteArrayList
public void forEach(Consumer<? super E> action) {
    if (action == null) throw new NullPointerException();
    Object[] elements = getArray();
    int len = elements.length;
    for (int i = 0; i < len; ++i>) {
        @SuppressWarnings("unchecked") E e = (E) elements[i];
        action.accept(e);
    }
}

CopyOnWriteArrayList<Integer> list = new CopyOnWriteArrayList<>();
list.add(1);
list.add(2);
// forEach 需要传入Consumer 参数
list.stream().forEach(System.out::println);
list.forEach(System.out::println);
```

示例3：addThen 方法使用。

``` java
6    public class ConsumersTest {
7
8       public static void main(String[] args) {
9           Consumer<Integer> consumer1 = x -> System.out.println("first x : " + x);
10          Consumer<Integer> consumer2 = x -> {
11              System.out.println("second x : " + x);
12              throw new NullPointerException("throw exception second");
13          };
14          Consumer<Integer> consumer3 = x -> System.out.println("third x : " + x);
15
16          consumer1.andThen(consumer2).andThen(consumer3).accpet(1);
17      }
18
19    }
```
```
 输出结果
 first x : 1
 second x : 1
 Exception in thread "main" java.lang.NullPointerException: throw exception second
  at com.dabin.java8.ConsumersTest.lambda$main$1(ConsumersTest.java:16)
```

### 5、Stream

使用 `java.util.Stream` 对一个包含一个或多个元素的集合做各种操作，原集合不变，返回新集合。只能对实现了 `java.util.Collection` 接口的类做流的操作。`Map` 不支持 `Stream` 流。`Stream` 流支持同步执行，也支持并发执行。

#### Filter 过滤

Filer 的入参是一个 Predicate，用于筛选出我们需要的集合元素。原集合不变，Filter集合过滤不符合特定条件的，下面的代码会过滤掉 `nameList` 中不以大彬开头的字符串。

``` java
6    public class StreamTest {
7
8       public static void main(String[] args) {
9           List<String> nameList = new ArrayList<>();
10          nameList.add("示例1");
11          nameList.add("示例2");
12          nameList.add("aaa");
13          nameList.add("bbb");
14          
15          nameList.stream().filter((s) -> s.startsWith("示例")).forEach(System.out::println);          
16      }
17
18   }
```
```
 输出结果
 示例1
 示例2
 ```

 #### Sorted 排序

 自然排序，不改变原集合，返回排序后的集合。
 
 ``` java
 public class StreamTest1 {
    public static void mian(String[] args) {
        List<String> nameList = new ArrayList<>();
        nameList.add("示例3");
        nameList.add("示例1");
        nameList.add("示例2");
        nameList.add("aaa");
        nameList.add("bbb");

        nameList.stream().filter((s) -> s.startsWith("示例")).sorted().forEach(System.out::println);
    }
 }
 ```
 ```
输出结果
示例1
示例2
示例3
 ```

 逆序排序：

``` java
 nameList.stream()
        .sorted(Comparator.reverseOrder());
 ```

对元素某个字段排序：

``` java
 list.stream().sorted(Comparator.comparing(Student::getAge).reversed());
 list.stream().sorted(Comparator.comparing(Student::getAge);
 ```

 #### Map 转换

 将每个字符串转为大写

``` java
 public class StreamTest2 {
    public static void mian(String[] args) {
        List<String> nameList = new ArrayList<>();
        nameList.add("aaa");
        nameList.add("bbb");

        nameList.stream()
                .map(String::toUpperCase)
                .forEach(System.out::println);
    }
 }
 ```
```
 输出结果
  AAA
  BBB
 ```

#### Match 匹配

验证 `nameList` 中的字符串是否有以 `示例` 开头的。 

``` java
 public class StreamTest3 {
    public static void mian(String[] args) {
        List<String> nameList = new ArrayList<>();
        nameList.add("示例1");
        nameList.add("示例2");

        boolean startWithDabin = nameList.stream()
                                         .map(String::toUpperCase)
                                         .anyMatch((s) -> s.startsWith("示例"));

        System.out.println(startWithDabin);
    }
 }
 ```

```
 输出结果
  true
  ```

#### Count 计数

统计 `stream` 流中的元素总数，返回值是 `long` 类型。 

``` java
 public class StreamTest4 {
    public static void mian(String[] args) {
        List<String> nameList = new ArrayList<>();
        nameList.add("示例1");
        nameList.add("示例2");
        nameList.add("aaa");

        long count = nameList.stream()
                             .map(String::toUpperCase)
                             .filter((s) -> s.startsWith("示例"))
                             .count();

        System.out.println(count);
    }
 }
 ```

```
 输出结果
  2
  ```

#### Reduce

类似拼接，可以实现将 `list` 归约成一个值。它的返回类型是 `Optional` 类型。 

``` java
 public class StreamTest5 {
    public static void mian(String[] args) {
        List<String> nameList = new ArrayList<>();
        nameList.add("示例1");
        nameList.add("示例2");

        Optional<String> reduced = nameList.stream()
                                           .sorted()
                                           .reduce((s1, s2) -> s1 + "#" + s2);
                             

        reduced.ifPresent(System.out::println);
    }
 }
 ```

```
 输出结果
 示例1#示例2
  ```

#### flatMap

flatMap 用于将多个Stream连接成一个Stream。

下面的例子，把几个小的list转换到一个大的list。

``` java
 public class StreamTest6 {
    public static void mian(String[] args) {
        List<String> team1 = Arrays.asList("示例1","示例2","示例3");
        List<String> team2 = Arrays.asList("示例4","示例5");

        List<List<String>> players = new ArrayList<>();
        players.add(team1);
        players.add(team2);

        List<String> flatMapList = players.stream()
                                          .flatMap(pList -> pList.stream())
                                          .collect(Collectors.toList());
        
        System.out.println(flatMapList);                                 
    }
 }
 ```

```
 输出结果
 [示例1, 示例2, 示例3, 示例4, 示例5]
  ```

下面的例子中，将words数组中的元素按照字符拆分，然后对字符去重。

``` java
 public class StreamTest7 {
    public static void mian(String[] args) {
        List<String> words = new ArrayList<String>();
        words.add("示例123");
        words.add("示例4");

        // 将words数组中的元素按照字符拆分，然后对字符去重
        List<String> stringList = words.stream()
                                       .flatMap(word -> Arrays.stream(word.split("")))
                                       .distinct()
                                       .collect(Collectors.toList());
        
        stringList.forEach(e -> System.out.print(e + ", "));                                 
    }
 }
 ```

```
 输出结果
  示, 例, 1, 2, 3, 4,
  ```

### 6、Parallel-Streams

并行流，`stream` 流是支持**顺序**和**并行**的。顺序流操作是单线程操作，串行化的流无法带来性能上的提升，通常我们会使用多线程来并行执行任务，处理速度更快。

``` java
 public class StreamTest7 {
    public static void mian(String[] args) {
        int max = 100;
        List<String> strs = new ArrayList<>(max);
        for (int i = 0; i < max; i++>) {
            UNID uuid = UUID.randomUUID();
            strs.add(uuid.toString());
        }

        List<String> sortedStrs = strs.stream().sorted().collect(Collectors.toList());
        System.out.println(sortedStrs);
    }
 }
 ```
```
 输出结果
[06a61070-58b3-44e1-885a-0ff7ab1a10bf, 0703e7e7-8639-4ff4-9815-c03ef72bb630, 086de47d-c23a-47ef-bbac-47d88b85931c, 0beaa66b-5fe4-4b56-ad1f-8a96b00b08ec, 0eeb31ba-e8f8-43e3-adc6-a94b6c97b6c2, 18c438a8-3d72-476e-ac08-aae6a89f7b21, 19e7702f-6e75-4dc0-8257-6adf636144fb, 1f493d6e-fdbe-4008-84d4-a859990aeb8f, 228a18ad-58c9-4b80-87da-a74df0ab5843, 24b8c494-5465-47d4-9265-3de38323a812, 26a579b4-4289-4dee-8a4a-3b3a44176df4, 2c230010-4323-41f1-9cc4-20b94f378065, 2d55011f-70fa-41d8-9b01-3e7aab6957ee, 2ebe318b-1eb5-4a23-920f-fe8bf6ccadf6, 333b626a-a92e-4f3b-b7df-31ee178bfd0b, 3ab89ada-dc83-42cd-b83b-0449ff2d227a, 40536ee3-bfba-448c-85d1-2d84d2bdcf85, 492e0c03-e392-4697-99f8-c9fd46769f27, 50ab8dc6-e99b-4159-8c11-1a907895c018, 510a9e2c-6f32-491e-a918-433e422c6968, 544ca1fb-099a-44a9-841f-2a0c0e643368, 559af1b3-2150-47a4-8fc0-fd18a46d65c1, 57596d25-f69b-47d8-9c35-cd8f2bc5083e, 5dea157a-b093-436c-a139-f79659d0ba84, 5f13281e-f887-4826-9022-64c29a2a194c, 6067b399-b10a-4be9-87b2-d577cfb21cc0, 63856996-5a5f-4cf7-83ae-9b2c6d1f8d2b, 643d63e8-7b88-4abb-a4ad-9163a547f241, 64e5543b-a2a7-48cb-ad0c-63713a6055c3, 69e2acce-57c7-46d4-b864-7663b15f139e, 6eabfc3f-a7c3-4695-b9ea-33a1e7bf785b, 6ff2aa1d-2d14-465a-8ca9-38958049fcd1, 71161e1b-4b0d-4be8-a25f-bc82ea10d66e, 72500c23-2356-4523-bf75-93f504235d03, 741a74d3-66eb-40ba-9709-415105289fa7, 75a6aeb5-b58c-4558-9bf6-5a1b91dd7ff2, 75f8a709-70ca-4169-83ec-52edbcd12a1c, 7612cabf-d30a-46aa-9ad4-20783ec121db, 76ee5173-75ed-4df1-932e-22a7b9badd2c, 77721e9a-a1cc-4bff-b062-a72dcf955ea8, 79eb09fd-cb29-44a3-a3ef-fbf2d13ef8ca, 7caf7bc4-db1b-40a8-a7fd-1d59091217f7, 7edec94a-66d9-4cf9-8b29-a8c7a78f8171, 7f1a4111-47fa-4bb2-bcd9-3ba42d3ebe22, 81838cb3-3045-4e75-a143-38cc15b6c02b, 81ac90a1-0f66-4333-9e11-0236c1eea002, 82c68d85-aa51-42c1-b2d3-9681250a6fe4, 83a4128a-a072-4702-8c06-caa5b36b3b8e, 863cbaa5-6591-40f1-95de-722dc9c69874, 87aea49f-cf58-4f5b-8e70-3d13b9278b51, 8d39edcf-998a-4869-b697-4fba21bc3a1e, 8ea75cfe-8cf8-4060-b353-1537edbf29ca, 8f468dae-a461-44c8-99b7-c55721abf237, 8fbdeda8-1aea-42c8-be98-eb7abaf35504, 9757a867-646d-4de8-a447-86e36a7a1d13, 97a1e758-19c4-4ff8-82ca-da300a86d7ef, 9b319b28-5928-436b-87f2-29807b9b4498, 9c787424-1f5b-4f49-a43a-1324c722f10c, 9d330c89-0bf1-47fd-86f1-26eda6f95730, 9dade012-d454-42f1-a5fd-c2d7cea50fcd, a2c1b0b3-8277-4c42-8bae-307500561bb8, a42d4154-fa69-45b1-8e19-e24520286f33, a9adf9e1-cc57-4a22-b5ff-d461cd0b5f7d, b0ae2a45-3bc6-43e1-a610-3f9e7143132b, b0c3c937-1708-4930-881a-add20adb2d5c, b114bfef-d9f6-4a09-b72c-63a72f8e165c, b3a29fba-8b53-4e00-a65d-bf87ff63e726, b44aed83-e2c1-4d03-a072-24a8856dcd2d, b470711b-02b4-4551-8eb6-3a06a81e6611, b72015fa-d7c2-4cd0-bf41-dafcbbb9b8e4, b8709c97-fcdb-4c1e-a494-dd2d7bc2731d, b9095961-0d1b-4b73-b558-3d716c2a4ad4, ba64cfde-f7dd-4ee7-8a62-0d8dc4366989, bcee6bc3-a71b-4bdb-a4ba-cd1810d663ca, bfcfc7d0-9474-4373-a98b-cb942a0bc5d3, c1432057-f623-4e07-ab4c-14aff064662a, c3052bef-391a-4099-b9e0-b2ebf385db39, c40ba8f5-682b-4a19-9a71-66f01ba41821, c8453213-e886-4142-83de-dbf401d96ce9, cbf38f9c-bc41-4fbe-9b40-d7e3b9f7116b, d18ae78e-8b30-45e4-af38-d27cce52fd0d, d3957f18-1a3c-40fe-968f-0fc25b4ef7a0, d6c9a74b-49f1-40e3-afca-7b60369a3d01, daced4ef-9885-470b-ab38-919e9e040ee1, dc9e62ba-305d-46a7-8e24-7dddbcc277ef, dde91131-52ed-469f-80e4-f85f051f7719, e1796080-bef5-46d0-9a81-be1bf5dce016, e203c65a-8b23-4a27-9908-fa1875e2f943, e31cd2ef-840d-4be8-8955-4cc676fce2e9, e59aae13-9a50-4ab3-88a7-6cf8ec512967, e9769fb9-25bc-474a-8309-1f8d8270f1d8, ea21507d-5caa-440d-aa67-2cb326fbd6a3, eaa33f56-cc03-44e4-9a92-6c3059a011a3, eb56ff80-4bd7-4572-8f4d-e0bbd935d0e6, ecdc2ed9-4abd-44ca-bd7d-e24f98f5209e, ecddc4ef-1e6b-4918-bb6c-effc42d4c679, eed50d5c-fd38-4832-b731-c89a77ad3a08, f132141f-e731-457a-9b7f-e1765c8efd10, fab528c7-e74c-416f-aaa1-4844451c2dac, fc4245a0-50e6-4ea1-84ed-020ac932384c]
  ```


### 7、Map集合

Java 8 针对 map 操作增加了一些方法，非常方便。

1) 删除元素使用 `removeIf()` 方法。

``` java
package com.curly.admin;

import java.util.*;
import java.util.stream.Collectors;

/**
 * @author broWsJle
 * @date 2023/2/17 16:13
 */
public class MapTest {

    public static void main(String[] args) {

        Map<Integer, String> map = new HashMap<>();
        map.put(1, "dabin1");
        map.put(2, "dabin2");

        // 删除value没有含有1的键值对
        map.values().removeIf(value -> !value.contains("1"));

        System.out.println(map);
    }
}
 ```
 ```
 输出结果：
 {1=dabin1}
  ```

2) `putIfAbsent(key, value)` 如果指定的key不存在，则put进去。

``` java
package com.curly.admin;

import java.util.*;
import java.util.stream.Collectors;

/**
 * @author broWsJle
 * @date 2023/2/17 16:13
 */
public class MapTest2 {

    public static void main(String[] args) {
        Map<Integer, String> map = new HashMap<>();
        map.put(1, "示例1");

        for (int i = 0; i < 3; i++) {
            map.putIfAbsent(i, "示例" + i);
        }
        map.forEach((id, val) -> System.out.println(val + ", "));
    }
}
```
```
输出结果：
示例0, 
示例1, 
示例2, 
```

3) map转换

``` java
package com.curly.admin;

import java.util.*;
import java.util.stream.Collectors;

/**
 * @author broWsJle
 * @date 2023/2/17 16:13
 */
public class MapTest3 {

    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        map.put("1", 1);
        map.put("2", 2);

        Map<String, String> newMap = map.entrySet().stream()
                .collect(Collectors.toMap(e -> e.getKey(), e -> "示例" + String.valueOf(e.getValue())));

        newMap.forEach((key, val) -> System.out.println(val + ", "));
    }
}
```
```
输出结果：
示例1, 
示例2,
```

4) map遍历

``` java
/**
 * @author broWsJle
 * @date 2023/2/17 16:13
 */
public class TestJava8New {

    public static void main(String[] args) {
        Map<Integer, String> map = new HashMap<>();
        map.put(1, "示例1");
        map.put(2, "示例2");

        System.out.println("方式一：");
        map.keySet().forEach(k -> {
            System.out.print(map.get(k) + ", ");
        });
        System.out.println();

        System.out.println("方式二：");
        map.entrySet().iterator()
                .forEachRemaining(e -> System.out.print(e.getValue() + ","));
        System.out.println();

        System.out.println("方式三：");
        map.entrySet().forEach(entry -> {
            System.out.print(entry.getValue() + ", ");
        });
        System.out.println();

        System.out.println("方式四：");
        map.values().forEach(v -> {
            System.out.print(v + ", ");
        });
    }
    
}
```
```
输出结果：
方式一：
示例1, 示例2, 
方式二：
示例1,示例2,
方式三：
示例1, 示例2, 
方式四：
示例1, 示例2, 
```