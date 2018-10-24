## 条件操作符

- [all操作符]()
- [ambArray操作符]()
- [contains操作符]()
- [any操作符]()
- [isEmpty操作符]()
- [defaultIfEmpty操作符]()
- [switchIfEmpty操作符]()
- [sequenceEqual操作符]()
- [takeUntil操作符]()
- [takeWhile操作符]()
- [skipUntil操作符]()
- [skipWhile操作符]()

### all
`all操作符主要用来判断所有元素是否满足某个条件`
```java
Observable.just(0, 1, 2, 3, 4, 5)
          .all(new Predicate<Integer>() {
                @Override
                public boolean test(Integer integer) throws Exception {
                    //判断是否所有元素都大于等于0
                    return integer >= 0;
                }
          })
          .subscribe(new CommonComsumer<>());
```
输出:
```java
accept true

Process finished with exit code 0
```
### ambArray
`ambArray操作符可以从多个被观察者中选择第一个发射元素的被观察者进行处理，其他被观察者则抛弃`
```java
Observable.ambArray(
          Observable.timer(1, TimeUnit.SECONDS),
          //仅处理第一个发射元素的被观察者
          Observable.just(3, 4, 5))
          .subscribe(new CommonComsumer<>());
```

输出:
```java
accept 3
accept 4
accept 5

Process finished with exit code 0
```
### contains
`contains操作符判断数据源中是否包含某个元素`
```java
Observable.just(1, 2, 3)
          .contains(2)
          .subscribe(new CommonComsumer<>());
```
输出:
```java
accept true

Process finished with exit code 0
```
### any
`contains操作符内部实际上调用的是any操作符`
```java
Observable.just(0, 1, 2, 3, 4, 5)
          .any(new Predicate<Integer>() {
              @Override
              public boolean test(Integer integer) throws Exception {
                  //判断是否有元素等于0
                  return integer == 3;
              }
          })
          .subscribe(new CommonComsumer<>());
```
输出:
```java
accept true

Process finished with exit code 0
```
### isEmpty
`isEmpty操作符主要用来判断被观察者是否发射过元素,内部调用的all操作符`
```java
Observable.just(0, 1, 2, 3, 4, 5)
          .isEmpty()
          .subscribe(new CommonComsumer<>());
```
输出:
```java
accept false

Process finished with exit code 0
```
### defaultIfEmpty
`defaultIfEmpty操作符主要用于被观察者不发松数据的时候发送一个默认元素`
```java
Observable.empty()
          .defaultIfEmpty(10)
          .subscribe(new CommonComsumer<>());
```
输出:
```java
accept 10

Process finished with exit code 0
```
### switchIfEmpty
`switchIfEmpty操作符主要用于被观察者不发送数据的时候发送多个默认数据`
```java
Observable.empty()
          .switchIfEmpty(Observable.just(1, 2, 3))
          .subscribe(new CommonComsumer<>());
```
输出:
```java
accept 1
accept 2
accept 3

Process finished with exit code 0
```
### sequenceEqual
`sequenceEqual操作符主要用于对比两个被观察者发送的元素队列,它只关心两个发送队列的元素,元素发射顺序和最终状态`
```java
Observable.sequenceEqual(
          Observable.just(0L, 1L, 2L),
          Observable.intervalRange(0, 3, 0, 1, TimeUnit.SECONDS))
          .subscribe(new CommonComsumer<>());
```
输出:
```java
accept true

Process finished with exit code 0
```
### takeUntil
`takeUntil操作符主要用来达到某个条件就停止`
```java
Observable.just(0, 1, 2, 3, 4, 5, 6, 7)
          //当元素大于2时停止
          .takeUntil(integer -> integer == 2)
          .subscribe(new CommonComsumer<>());
```
输出:
```java
accept 0
accept 1
accept 2

Process finished with exit code 0
```
### takeWhile
`takeWhile操作符判断条件为true才执行被观察者的事件`
```java
Observable.just(0, 1, 2, 3, 4, 5, 6, 7)
          .takeWhile(integer -> integer != 5)
          .subscribe(new CommonComsumer<>());
```
输出:
```java
accept 0
accept 1
accept 2
accept 3
accept 4

Process finished with exit code 0
```
### skipUntil
`skipUntil操作符主要用来接收一个被观察者直到被观察者发送事件之前,第一个被观察者所有发送的元素都会被抛弃`
```java
Observable.intervalRange(0, 10, 0, 1, TimeUnit.SECONDS)
          .skipUntil(Observable.timer(3, TimeUnit.SECONDS))
          .subscribe(new CommonComsumer<>());
```
`前三秒发送的数据都被抛弃`
输出:
```java
accept 3
accept 4
accept 5
accept 6
accept 7
accept 8
accept 9

Process finished with exit code 0
```
### skipWhile
`skipWhile操作符主要用来跳过开始的一段数据`
```java
Observable.intervalRange(0, 10, 0, 1, TimeUnit.SECONDS)
          //跳过开始一段数据
          .skipWhile(value -> value < 5)
          .subscribe(new CommonComsumer<>());
```
输出:
```java
accept 5
accept 6
accept 7
accept 8
accept 9

Process finished with exit code 0
```

复习文档
https://github.com/byhook/rxjava2-study
参考:
https://maxwell-nc.github.io/android/rxjava2-5.html
