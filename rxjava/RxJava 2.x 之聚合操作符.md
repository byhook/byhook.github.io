
## 聚合操作符

- [startWith操作符]()
- [startWithArray操作符]()
- [concat/concatArray操作符]()
- [merge/mergeArray操作符]()
- [concatDelayError/mergeDelayError操作符]()
- [zip操作符]()
- [combineLatest操作符]()
- [combineLatestDelayError操作符]()
- [reduce操作符]()
- [count操作符]()
- [collect操作符]()

### startWith
`startWith操作符主要用来在发送元素之前追加数据`
```java
Observable.just(1, 2, 3, 4, 5)
          .startWith(Observable.just(7, 8, 9))
          .subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 7
onNext 8
onNext 9
onNext 1
onNext 2
onNext 3
onNext 4
onNext 5
onComplete

Process finished with exit code 0
```
### startWithArray
`startWithArray操作符主要用来发送元素之前追加多个元素`
```java
Observable.just(1, 2, 3, 4, 5)
          .startWithArray(7, 8, 9)
          .subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 7
onNext 8
onNext 9
onNext 1
onNext 2
onNext 3
onNext 4
onNext 5
onComplete

Process finished with exit code 0
```
### concat/concatArray
`concat和concatArray操作符串行执行`
```java
Observable.concat(
          Observable.just(1, 2),
          Observable.just(3, 4, 5),
          Observable.just(7, 8, 9))
          .subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 1
onNext 2
onNext 3
onNext 4
onNext 5
onNext 7
onNext 8
onNext 9
onComplete

Process finished with exit code 0
```
`注意: concat最多连接4个,而concatArray可以连接多个`
### merge/mergeArray
`merge和mergeArray操作符主要用于合并多个被观察者并行执行`
```java
Observable.merge(
          Observable.intervalRange(0, 3, 1, 1, TimeUnit.SECONDS),
          Observable.intervalRange(3, 3, 1, 1, TimeUnit.SECONDS)
        ).subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 0
onNext 3
onNext 1
onNext 4
onNext 2
onNext 5
onComplete

Process finished with exit code 0
```
`注意: merge最多连接4个,而mergeArray可以连接多个`
### concatDelayError/mergeDelayError
`使用concat和merge操作符时，如果遇到其中一个被观察者发出onError事件则会马上终止其他被观察者的事件，如果希望onError事件推迟到其他被观察者都结束后才触发，可以使用对应的concatDelayError或者mergeDelayError操作符`

### zip
`zip操作符主要用来将多个观察者压缩成一个单独的操作`
```java
Observable.zip(
          Observable.just(1, 2),
          Observable.just(7, 9),
          (int1, int2) -> int1 + int2)
          .subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 8
onNext 11
onComplete

Process finished with exit code 0
```
### combineLatest
`combineLatest操作符主要用于对同一个时间线上合并最后的元素`
```java
Observable.combineLatest(
          Observable.just(1, 2, 3),
          Observable.intervalRange(0, 5, 1, 1, TimeUnit.SECONDS),
          (int1, int2) -> int1 + int2)
          .subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 3
onNext 4
onNext 5
onNext 6
onNext 7
onComplete

Process finished with exit code 0
```
### combineLatestDelayError
`combineLatestDelayError操作符主要处理delayError的问题`

### reduce
`reduce操作符主要用来将所有元素聚合到一起`
```java
Observable.just(1, 2, 3)
          .reduce((last, item) -> {
          //先执行1+2，然后用1+2的结果和3相加，最后输出6，相当于把三个元素聚合在一起
          return last + item;
          })
          .subscribe(new CommonComsumer<>());
```
`输出:`
```java
accept 6

Process finished with exit code 0
```
### count
`count操作符主要用来统计被观察者发送了多少个元素`
```java
Observable.just(1, 2, 3)
          .count()
          .subscribe(new CommonComsumer<>());
```
`输出:`
```java
accept 3

Process finished with exit code 0
```
### collect
`collect和reduce操作符类似，不过它是需要自己定义收集的容器和收集逻辑`
```java
Observable.just(1, 2, 3)
          .collect(new Callable<ArrayList<Integer>>() {
                @Override
                public ArrayList<Integer> call() throws Exception {
                    return new ArrayList<>();
                }
          }, new BiConsumer<ArrayList<Integer>, Integer>() {
                @Override
                public void accept(ArrayList<Integer> list, Integer value) throws Exception {
                    System.out.println("add element " + value);
                    list.add(value);
                }
          })
          .subscribe(new CommonComsumer<>());
```
`输出:`
```java
add element 1
add element 2
add element 3
accept [1, 2, 3]

Process finished with exit code 0
```
### count
```java

```
`输出:`
```java

```


### count
```java

```
`输出:`
```java

```
