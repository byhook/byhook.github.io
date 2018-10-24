最近几天想把rxjava2的操作符都整理一下，看到网上的很多文章都总结的很好，但是时间久了依然会忘记。

## 过滤操作符

- [filter操作符]()

### filter
`filter操作符，可以自己设定任意的规则来过滤数据`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7)
          .filter(new Predicate<Integer>() {
                @Override
                public boolean test(Integer value) throws Exception {
                    return value % 2 == 0;
                }
          }).subscribeWith(new CommonObserver<>());
```
输出:
```java
onNext 2
onNext 4
onNext 6
onComplete

Process finished with exit code 0
```

### take
`take操作符主要用来限制发射的数量`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9)
          .take(5)
          .subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 1
onNext 2
onNext 3
onNext 4
onNext 5
onComplete

Process finished with exit code 0
```
### takeLast
`takeLast操作符主要用来筛选最后的几个元素`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9)
          .takeLast(5)
          .subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 5
onNext 6
onNext 7
onNext 8
onNext 9
onComplete

Process finished with exit code 0
```
### firstElement/lastElement
`firstElement和lastElement操作符分别用来过滤第一个和最后一个元素`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7)
          .firstElement().subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer value) throws Exception {
                System.out.println("accept: " + value);
            }
        });
```
`输出:`
```java
accept: 1

Process finished with exit code 0
```
同理`lastElement操作符`会输出最后一个元素`7`
### first/last
`first和last操作符分别用来过滤第一个和最后一个元素,但它可以指定一个默认元素的参数`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7)
          .first(5).subscribe(new CommonComsumer<>());
```
`输出:`
```java
accept 1

Process finished with exit code 0
```
同理`last操作符`会输出最后一个元素`7`,这里要注意的是如果前面的使用的`empty操作符`,这里输出的就是默认的`5`这个元素
### firstOrError/lastOrError
`firstOrError/lastOrError操作符主要用于取第一个元素和最后一个元素如果为空,则会抛出异常`
```java
Observable.empty()
          .firstOrError().subscribe(new CommonComsumer<>());
```
`输出:`
```java
Exception in thread "main" io.reactivex.exceptions.OnErrorNotImplementedException
	at io.reactivex.internal.functions.Functions$OnErrorMissingConsumer.accept(Functions.java:704)
	at io.reactivex.internal.functions.Functions$OnErrorMissingConsumer.accept(Functions.java:701)
	at io.reactivex.internal.observers.ConsumerSingleObserver.onError(ConsumerSingleObserver.java:47)
	at io.reactivex.internal.operators.observable.ObservableElementAtSingle$ElementAtObserver.onComplete(ObservableElementAtSingle.java:117)
	at io.reactivex.internal.disposables.EmptyDisposable.complete(EmptyDisposable.java:53)
	at io.reactivex.internal.operators.observable.ObservableEmpty.subscribeActual(ObservableEmpty.java:28)
	at io.reactivex.Observable.subscribe(Observable.java:12030)
	at io.reactivex.internal.operators.observable.ObservableElementAtSingle.subscribeActual(ObservableElementAtSingle.java:37)
	at io.reactivex.Single.subscribe(Single.java:3394)
	at io.reactivex.Single.subscribe(Single.java:3380)
	at io.reactivex.Single.subscribe(Single.java:3351)
	at com.onzhou.rxjava2.filter.SampleFirstOrError.invoke(SampleFirstOrError.java:18)
	at com.onzhou.rxjava2.Main.main(Main.java:38)
Caused by: java.util.NoSuchElementException
	... 10 more

Process finished with exit code 0
```
### elementAt/elementAtOrError
`elementAt操作符主要用来指定发射第几个元素支持越界,而elementAtOrError也是指定发射第几个元素,但是越界会抛出异常`
`elementAt操作符:`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7)
          .elementAt(3).subscribe(new CommonComsumer<>());
```
`输出:`
```java
accept 4

Process finished with exit code 0
```
`elementAtOrError操作符:`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7)
          .elementAtOrError(16).subscribe(new CommonComsumer<>());
```
`输出:`
```java
Exception in thread "main" io.reactivex.exceptions.OnErrorNotImplementedException
	at io.reactivex.internal.functions.Functions$OnErrorMissingConsumer.accept(Functions.java:704)
	at io.reactivex.internal.functions.Functions$OnErrorMissingConsumer.accept(Functions.java:701)
	at io.reactivex.internal.observers.ConsumerSingleObserver.onError(ConsumerSingleObserver.java:47)
	at io.reactivex.internal.operators.observable.ObservableElementAtSingle$ElementAtObserver.onComplete(ObservableElementAtSingle.java:117)
	at io.reactivex.internal.operators.observable.ObservableFromArray$FromArrayDisposable.run(ObservableFromArray.java:110)
	at io.reactivex.internal.operators.observable.ObservableFromArray.subscribeActual(ObservableFromArray.java:36)
	at io.reactivex.Observable.subscribe(Observable.java:12030)
	at io.reactivex.internal.operators.observable.ObservableElementAtSingle.subscribeActual(ObservableElementAtSingle.java:37)
	at io.reactivex.Single.subscribe(Single.java:3394)
	at io.reactivex.Single.subscribe(Single.java:3380)
	at io.reactivex.Single.subscribe(Single.java:3351)
	at com.onzhou.rxjava2.filter.SampleElementAtOrError.invoke(SampleElementAtOrError.java:18)
	at com.onzhou.rxjava2.Main.main(Main.java:40)
Caused by: java.util.NoSuchElementException
	... 10 more

Process finished with exit code 0
```
### ofType
`ofType操作符主要用来筛选特定类型的数据`
```java
Observable.just(1, "name", true)
          .ofType(Integer.class).subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 1
onComplete

Process finished with exit code 0
```
### skip/skipLast
`skip或者skipLast操作符,可以跳过若干个元素，或者跳过一段时间`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9)
          .skip(5).subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 6
onNext 7
onNext 8
onNext 9
onComplete

Process finished with exit code 0
```
`skipLast操作符`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9)
          .skipLast(5).subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 1
onNext 2
onNext 3
onNext 4
onComplete

Process finished with exit code 0
```
### ignoreElements
`ignoreElements操作符会忽略所有的元素,回调onError或者onComplete`
```java
Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9)
          .ignoreElements().subscribe(new Action() {
            @Override
            public void run() throws Exception {
                System.out.println("onComplete");
            }
        });
```
`输出:`
```java
onComplete

Process finished with exit code 0
```
### distinct/distinctUntilChanged
`distinct操作符主要用来过滤重复的元素`
```java
Observable.just(1, 2, 3, 1, 3, 1, 2)
          .distinct().subscribe(new CommonComsumer<>());
```
`输出:`
```java
accept 1
accept 2
accept 3

Process finished with exit code 0
```
`distinctUntilChanged操作符主要用来过滤连续重复的元素`
```java
Observable.just(1, 2, 2, 1, 1, 6, 7)
          .distinctUntilChanged().subscribe(new CommonComsumer<>());
```
`输出:`
```java
accept 1
accept 2
accept 1
accept 6
accept 7

Process finished with exit code 0
```
### timeout
`timeout操作符主要用来进行超时过滤操作`
```java
Observable.intervalRange(0, 10, 0, 2, TimeUnit.SECONDS)
          .timeout(1, TimeUnit.SECONDS).subscribeWith(new CommonObserver<>());
```
执行1秒后超时抛出异常
`输出:`
```java
onNext 0
onError java.util.concurrent.TimeoutException

Process finished with exit code 0
```
### throttleFirst
`throttleFirst操作符主要用来连续时间段内响应一次操作`
```java
Observable.intervalRange(0, 6, 0, 1, TimeUnit.SECONDS)
          //每秒中只处理第一个数据
          .throttleFirst(1, TimeUnit.SECONDS).subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 0
onNext 2
onNext 4
onComplete

Process finished with exit code 0
```
### throttleLast/sample
`throttleLast和sample操作符类似都是隔一段时间采集数据`
```java
Observable.intervalRange(0, 8, 0, 1, TimeUnit.SECONDS)
          //每秒中只处理第一个数据
          //.sample(2, TimeUnit.SECONDS).subscribeWith(new CommonObserver<>());
          .throttleLast(2, TimeUnit.SECONDS).subscribeWith(new CommonObserver<>());
```

`输出:`
```java
onNext 1
onNext 3
onNext 5
onComplete

Process finished with exit code 0
```
### throttleWithTimeout/debounce
`throttleWithTimeout/debounce操作符主要用来处理一段时间内不响应才输出结果`
```java
Observable.intervalRange(0, 10, 0, 1, TimeUnit.SECONDS)
          //2秒内有新数据则抛弃旧数据
          //.debounce(2, TimeUnit.SECONDS);
         .throttleWithTimeout(2, TimeUnit.SECONDS).subscribeWith(new CommonObserver<>());
```
`输出:`
```java
onNext 9
onComplete

Process finished with exit code 0
```
