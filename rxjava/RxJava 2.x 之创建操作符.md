最近几天想把rxjava2的操作符都整理一下，看到网上的很多文章都总结的很好，但是时间久了依然会忘记。

## 创建操作符

- [just操作符]()
- [fromArray操作符]()
- [empty操作符]()
- [error操作符]()
- [never操作符]()
- [fromIterable操作符]()
- [interval操作符]()
- [intervalRange操作符]()
- [range/rangeLong操作符]()
- [defer操作符]()

### just
`just操作符一次可以发送多个数据，最多支持10个数据元素`
```java
//最多支持10个数据元素
Observable.just(1, 2, 3, 4, 5)
          .subscribeWith(new CommonObserver<>());
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

### fromArray
`fromArray操作符接收任意长度的数据数组`
```java
Observable.fromArray(1, 2, 3, 4, 5, 6, 7)
          .subscribeWith(new CommonObserver<>());
```
`输出`:
```java
onNext 1
onNext 2
onNext 3
onNext 4
onNext 5
onNext 6
onNext 7
onComplete

Process finished with exit code 0
```

### empty
`empty操作符不会发送任何数据，而是直接发送onComplete事件`
```java
Observable.empty()
          .subscribeWith(new CommonObserver<>());
```
`输出`:
```java
onComplete

Process finished with exit code 0
```
### error
`error操作符不会发送任何数据，而是直接发送onError事件`
```java
Observable.error(new NullPointerException())
          .subscribeWith(new CommonObserver<>());
```
```java
onError java.lang.NullPointerException

Process finished with exit code 0
```
### never
`never操作符什么都不会发送的，也不会触发观察者任何的回调`
```java
Observable.never()
          .subscribeWith(new CommonObserver<>());
```
### fromIterable
`fromIterable操作符可以遍历可迭代数据集合`
```java
Observable.fromIterable(Arrays.asList(1, 2, 3, 4, 5))
          .subscribeWith(new CommonObserver<>());
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
### timer
`timer操作符用来指定时间间隔触发回调`
```java
Observable.timer(1, TimeUnit.SECONDS).subscribeWith(new CommonObserver<>());
```
输出:
```java
onNext 0
onComplete
```
### interval
`interval操作符会不断地发送数据`
```java
Observable.interval(1, TimeUnit.SECONDS).subscribeWith(new CommonObserver<>());
```
输出:
```java
onNext 0
onNext 1
onNext 2
onNext 3
......
```
### intervalRange
`intervalRange操作符指定发送数据的范围和时间间隔`
```java
Observable.intervalRange(0, 5, 0, 1, TimeUnit.SECONDS).subscribeWith(new CommonObserver<>());
```
输出:
```java
onNext 0
onNext 1
onNext 2
onNext 3
onNext 4
onComplete

Process finished with exit code 0
```
`注意:`
当范围内的数据发送完毕就会回调`onComplete`方法
### range / rangeLong
`range和rangeLong操作符都是指定范围发送数据`
```java
Observable.range(0, 5).subscribeWith(new CommonObserver<>());
```
输出:
```java
onNext 0
onNext 1
onNext 2
onNext 3
onNext 4
onComplete

Process finished with exit code 0
```
### defer
`defer操作符可以使一个被观察者订阅多个观察者`
```java
Observable<String> observable = Observable.defer(new Callable<ObservableSource<String>>() {
            @Override
            public ObservableSource<String> call() throws Exception {
                return Observable.just("hello", "world");
            }
        });
//订阅第一个观察者
observable.subscribeWith(new CommonObserver<>());
//订阅第二个观察者
observable.subscribeWith(new CommonObserver<>());
```
输出:
```java
onNext hello
onNext world
onComplete
onNext hello
onNext world
onComplete

Process finished with exit code 0
```
`注意:`
只有当第一个观察者执行完后才回去创建第二个被观察者然后订阅观察者

复习文档
https://github.com/byhook/rxjava2-study

参考：
https://maxwell-nc.github.io/android/rxjava2-2.html
