
## 变换操作符

- [map操作符]()
- [flatMap操作符]()
- [flatMapIterable操作符]()
- [concatMap操作符]()
- [switchMap操作符]()
- [cast操作符]()
- [scan操作符]()
- [buffer操作符]()
- [toList操作符]()
- [groupBy操作符]()
- [toMap操作符]()

### map
`map操作符作为基本的转换操作符，可以把每一个元素转换成新的元素发射`
```java
Observable.just(0, 1, 2, 3, 4, 5)
          .map(new Function<Integer, String>() {
              @Override
              public String apply(Integer integer) throws Exception {
                  return "apply " + integer;
              }
          })
          .subscribe(new CommonComsumer<>());
```
输出:
```java
accept apply 0
accept apply 1
accept apply 2
accept apply 3
accept apply 4
accept apply 5

Process finished with exit code 0
```
### flatMap
`flatMap操作符将每个元素转换成被观察者,每个被观察者发送的元素会合并成新的被观察者,顺序输出`
`Student类`
```java
public class Student {

    public int age;

    public String name;

    public List<Integer> courses;

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

    public List<Integer> getCourses() {
        return courses;
    }

    public void setCourses(List<Integer> courses) {
        this.courses = courses;
    }

    public static List<Student> create() {
        List<Student> students = new ArrayList<>();
        Student student;
        for (int i = 0; i < 5; i++) {
            student = new Student();
            student.setAge(i);
            student.setName("name " + i);
            student.setCourses(Arrays.asList(1, 2, 3));
            students.add(student);
        }
        return students;
    }

}
```
```java
List<Student> students = Student.create();
        Observable.fromIterable(students)
                .flatMap(value -> {
                    return Observable.fromIterable(value.courses);
                }).subscribeWith(new CommonObserver<>());
```
输出:
```java
onNext 1
onNext 2
onNext 3
onNext 1
onNext 2
onNext 3
onNext 1
onNext 2
onNext 3
onNext 1
onNext 2
onNext 3
onNext 1
onNext 2
onNext 3
onComplete

Process finished with exit code 0
```
### flatMapIterable
`flatMapIterable操作符与flatMap类似,但它可以将每个元素转化成iterable`
```java
List<Student> students = Student.create();
        Observable.fromIterable(students)
                .flatMapIterable(new Function<Student, Iterable<Integer>>() {
                    @Override
                    public Iterable<Integer> apply(Student student) throws Exception {
                        return student.courses;
                    }
                }).subscribeWith(new CommonObserver<>());
```
输出:
```java
onNext 1
onNext 2
onNext 3
onNext 1
onNext 2
onNext 3
onNext 1
onNext 2
onNext 3
onNext 1
onNext 2
onNext 3
onNext 1
onNext 2
onNext 3
onComplete

Process finished with exit code 0
```
### concatMap
`concatMap操作符和flatMap类似,flatMap内部通过merge合并元素,顺序可能会错乱,但是concatMap内部安装concat合并元素,按照顺序发射`
```java
List<Student> students = Student.create();
        Observable.fromIterable(students)
                .concatMap(value -> {
                    return Observable.fromIterable(value.courses);
                }).subscribeWith(new CommonObserver<>());
```
输出:
```java
onNext 1
onNext 2
onNext 3
onNext 1
onNext 2
onNext 3
onNext 1
onNext 2
onNext 3
onNext 1
onNext 2
onNext 3
onNext 1
onNext 2
onNext 3
onComplete

Process finished with exit code 0
```
### switchMap
`switchMap操作符每次转换出来新的数据都会取代前一个被观察者`
```java
Observable.just(0, 1, 2)
                .switchMap(value -> {
                    return Observable.timer(1, TimeUnit.SECONDS).map(longValue -> value);
                }).subscribeWith(new CommonObserver<>());

```
输出:
```java
onNext 2
onComplete

Process finished with exit code 0
```
### cast
`cast操作符主要用来强制转换每一个元素的类型`
```java
Observable.just(1, 2, 3, 4, 5)
          //把每个元素都转换成Number类型,然后再发射
          .cast(Number.class).subscribeWith(new CommonObserver<>());
```
输出:
```java
onNext 1
onNext 2
onNext 3
onNext 4
onNext 5
onComplete

Process finished with exit code 0
```
### scan
`scan操作符扫描每一个元素,忽略第一个元素,从第二个元素开始进行变换后返回`
```java
Observable.just(1, 2, 3, 4, 5)
          .scan(new BiFunction<Integer, Integer, Integer>() {
              @Override
              public Integer apply(Integer value1, Integer value2) throws Exception {
                  return value2 + 1;
              }
          }).subscribeWith(new CommonObserver<>());
```
输出:
```java
onNext 1
onNext 3
onNext 4
onNext 5
onNext 6
onComplete

Process finished with exit code 0
```
### buffer
`buffer操作符主要用来把多个元素打包成一个元素一次过发送数据`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7, 8)
          .buffer(3).subscribeWith(new CommonObserver<>());
```
输出:
```java
onNext [1, 2, 3]
onNext [4, 5, 6]
onNext [7, 8]
onComplete

Process finished with exit code 0
```
### toList
`toList操作符主要用来把所有元素转换成一个List一次过发送出去`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7, 8)
          .toList().subscribe(new CommonComsumer<>());
```
输出:
```java
accept [1, 2, 3, 4, 5, 6, 7, 8]

Process finished with exit code 0
```
### groupBy
`groupBy操作符会将接收到的数据分组key`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7, 8)
          .groupBy(new Function<Integer, String>() {
              @Override
              public String apply(Integer integer) throws Exception {
                  return integer >= 5 ? "A组" : "B组";
              }
          }).subscribe(new Consumer<GroupedObservable<String, Integer>>() {
            @Override
            public void accept(GroupedObservable<String, Integer> observable) throws Exception {
                observable.subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        String key = observable.getKey();
                        System.out.println(key + ":" + String.valueOf(integer));
                    }
                });
            }
        });
```
输出:
```java
B组:1
B组:2
B组:3
B组:4
A组:5
A组:6
A组:7
A组:8

Process finished with exit code 0
```
### toMap
`toMap操作符主要用来将元素转换成key,value对应的map`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7, 8)
          .toMap(new Function<Integer, String>() {
              @Override
              public String apply(Integer integer) throws Exception {
                  return integer >= 5 ? "A组" : "B组";
              }
          }).subscribe(new CommonComsumer<>());
```
输出:
```java
accept {B组=4, A组=8}

Process finished with exit code 0
```
### map
`map操作符作为基本的转换操作符，可以把每一个元素转换成新的元素发射`
```java

```
输出:
```java

```
### map
`map操作符作为基本的转换操作符，可以把每一个元素转换成新的元素发射`
```java

```
输出:
```java

```

复习文档
https://github.com/byhook/rxjava2-study
参考:
https://maxwell-nc.github.io/android/rxjava2-6.html
