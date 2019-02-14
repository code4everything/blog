## Java利用Comparator实现分组排序

在数据库中我们可以使用 `order by` 和 `group by` 轻松实现分组和排序的功能，那么在Java中我们又该如何实现呢？下面我们一起来研究一番

### `Comparator` 与 `Comparable`

- `Comparable` 是一个排序接口，实现了该接口的类，表示该类支持排序功能，重写 `compareTo` 方法可使程序按照我们的意愿对数组或列表进行排序

- `Comparator` 是一个比较器接口，如果我们需要对类进行排序，而该类并没有实现 `Comparable` 接口，那么我们可以通过实现 `Comparator` 接口来保持类的排序能力

### 简单排序

1. 创建一个 `Bean` 类，我们将对该类进行排序

    ``` java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class Student {

       /**
       * 班级名称
       */
       private String className;

       /**
       * 学生姓名
       */
       private String name;

       /**
       * 分数
       */
       private Integer score;
    }
    ```
    
2. 准备一些数据，用于测试

    ``` java
    // 准备一些数据，用于测试
    final String classOne = "一班";
    final String classTwo = "二班";
    final String classThree = "三班";
    List<Student> students = new ArrayList<>();
    students.add(new Student(classOne, "李一", 43));
    students.add(new Student(classOne, "王二", 75));
    students.add(new Student(classOne, "张三", 91));
    students.add(new Student(classTwo, "郭子", 59));
    students.add(new Student(classTwo, "潘子", 88));
    students.add(new Student(classTwo, "刘子", 97));
    students.add(new Student(classThree, "陈六", 66));
    students.add(new Student(classThree, "唐七", 77));
    students.add(new Student(classThree, "周八", 89));
    ```
    
3. 创建一个 `Comparator` 对象

    如果我们将第一参数与第二个参数进行比较，那么表示这是一个升序序列（自然排序），否则表示一个降序序列。
    当然这样说并不完全准确，主要还是看返回的整数值是大于零、小于零还是等于零，只有当大于零时对象 `o2` 才会置于对象 `o1` 前面

    ``` java
    // 按分数进行降序排序
    Comparator<Student> comparator = (o1, o2) -> o2.getScore().compareTo(o1.getScore());
    ```
    
4. 测试排序结果

    ``` java
    Collections.sort(students, comparator);
    students.forEach(System.out::println);
    ```
    
    打印结果，可以看出列表是按分数进行排序的，接下来我们将对班级名称进行分组并按分数排序
    
    ``` text
    Student(className=二班, name=刘子, score=97)
    Student(className=一班, name=张三, score=91)
    Student(className=三班, name=周八, score=89)
    Student(className=二班, name=潘子, score=88)
    Student(className=三班, name=唐七, score=77)
    Student(className=一班, name=王二, score=75)
    Student(className=三班, name=陈六, score=66)
    Student(className=二班, name=郭子, score=59)
    Student(className=一班, name=李一, score=43)
    ```
    
### 分组排序

1. 创建一个分组排序的 `Comparator` 对象

    由于我们需要按班级名称进行分组排序，所以我们必须进行两次计较，第一次比较是否是相同的班级名称，如果不是直接返回比较的结果值即可，否则返回分数的比较结果

    ``` java
    Comparator<Student> groupComparator = (o1, o2) -> {
        int diff = o1.getClassName().compareTo(o2.getClassName());
        return diff == 0 ? o2.getScore().compareTo(o1.getScore()) : diff;
    };
    ```
    
2. 测试排序结果

    ``` java
    Collections.sort(students, groupComparator);
    students.forEach(System.out::println);
    ```
    
    打印结果，可以看出确实时按班级分组排序的，但是如果我们想要班级名称也是按顺序排的呢，怎么做？
    
    ``` text
    Student(className=一班, name=张三, score=91)
    Student(className=一班, name=王二, score=75)
    Student(className=一班, name=李一, score=43)
    Student(className=三班, name=周八, score=89)
    Student(className=三班, name=唐七, score=77)
    Student(className=三班, name=陈六, score=66)
    Student(className=二班, name=刘子, score=97)
    Student(className=二班, name=潘子, score=88)
    Student(className=二班, name=郭子, score=59)
    ```
    
### 有序的分组排序

如何让分组的名称按照我们想要的顺序出现？简单，请看下面

1. 定义一个分组顺序

    分组顺序用来判断该分组所在位置，这样我们就可以自定义分组的位置，从而实现有序的分组
    
    ``` java
    Map<String, Integer> groupOrder = new HashMap<>(4);
    groupOrder.put(classOne, 1);
    groupOrder.put(classTwo, 2);
    groupOrder.put(classThree, 3);
    ```
    
2. 创建一个有序的分组排序 `Comparator` 对象

    通过自定义的班级顺序来决定班级名称出现的位置

    ``` java
    Comparator<Student> orderlyGroupComparator = (o1, o2) -> {
        int diff = groupOrder.get(o1.getClassName()).compareTo(groupOrder.get(o2.getClassName()));
        return diff == 0 ? o2.getScore().compareTo(o1.getScore()) : diff;
    };
    ```
    
3. 测试排序结果

    ``` java
    Collections.sort(students, orderlyGroupComparator);
    students.forEach(System.out::println);
    ```
    
    打印结果
    
    ``` text
    Student(className=一班, name=张三, score=91)
    Student(className=一班, name=王二, score=75)
    Student(className=一班, name=李一, score=43)
    Student(className=二班, name=刘子, score=97)
    Student(className=二班, name=潘子, score=88)
    Student(className=二班, name=郭子, score=59)
    Student(className=三班, name=周八, score=89)
    Student(className=三班, name=唐七, score=77)
    Student(className=三班, name=陈六, score=66)
    ```
    
### 总结

现在回过去看，原来分组排序如此的简单呀，不过这完全得益于 `Comparator` 的强大（你也完全可以使用 `Comparable` 接口来达到相同的效果）

[本示例源代码参考](/code4everything/demo/blob/master/demo-test/src/main/java/com/zhazhapan/demo/test/SortTest.java)

> 由于本人水平有限，有不正确的地方，还望指正，谢谢
