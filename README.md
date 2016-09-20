# RxJava2Extensions

<a href='https://travis-ci.org/akarnokd/RxJava2Extensions/builds'><img src='https://travis-ci.org/akarnokd/RxJava2Extensions.svg?branch=master'></a>
[![codecov.io](http://codecov.io/github/akarnokd/RxJava2Extensions/coverage.svg?branch=master)](http://codecov.io/github/akarnokd/RxJava2Extensions?branch=master)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.akarnokd/rxjava2-extensions/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.akarnokd/rxjava2-extensions)

Extra sources, operators and components and ports of many 1.x companion libraries for RxJava 2.x.

# Features

## Mathematical operations over numerical sequences

Although most of the operations can be performed with `reduce`, these operators have lower overhead
as they cut out the reboxing of primitive intermediate values.

The following operations are available in `MathFlowable` for `Flowable` sequences and `MathObservable` in `Observable`
sequences:

  - `averageDouble()`
  - `averageFloat()`
  - `max()`
  - `min()`
  - `sumDouble()`
  - `sumFloat()`
  - `sumInt()`
  - `sumLong()`
  
Example

```java
MathFlowable.averageDouble(Flowable.range(1, 10))
.test()
.assertResult(5.5);

Flowable.just(5, 1, 3, 2, 4)
.to(MathFlowable::min)
.test()
.assertResult(1);
```
  

## Parallel operations

RxJava is sequential by nature and you can achieve parallelism by having parallel sequences. Forking out
and joining a single sequence can become complicated and incurs quite an overhead. By using the `ParallelFlowable`
API, the forking and joining of parallel computations started in and ending in a Flowable is more efficient than
the classical methods (groupBy/flatMap, window/flatMap, etc).

The main entry point is `ParallelFlowable.from(Publisher, int)` where given a `Flowable` and the parallelism level, the source
sequence is dispatched into N parallel 'rails' which are fed in a round-robin fashion from the source `Flowable`.

The `from()` method only defines how many rails there will be but it won't actually run these rails on different threads. As with other RxJava, asynchrony is optional and introduced via an operator `runOn` which takes the usual `Scheduler` of your chosing. Note that the parallelism level of the `Scheduler` and the `ParallelFlowable` need not to match (but it is recommended).

Not all sequential operations make sense in the parallel world. These are the currently supported operations:

  - `map`, 
  - `filter`
  - `flatMap`, 
  - `concatMap`
  - `reduce`
  - `collect`
  - `sort`
  - `toSortedList`
  - `compose`
  - `doOnCancel`, `doOnError`, `doOnComplete`, `doOnNext`, `doOnSubscribe`, `doAfterTerminate`, `doOnRequest`

Example:

```java
ParallelFlowable.from(Flowable.range(1, 1000))
.runOn(Schedulers.computation())
.map(v -> v * v)
.filter(v -> (v & 1) != 0)
.sequential()
.subscribe(System.out::println)
```

## String operations

Currently, only the `StringFlowable` and `StringObservable` support streaming the characters of a `CharSequence`:

```java
StringFlowable.characters("Hello world")
.map(v -> Characters.toLower((char)v))
.subscribe(System.out::print, Throwable::printStackTrace, System.out::println);
```

## Asynchronous jumpstarting a sequence

TBD: conversion of RxJavaAsyncUtil

## Computational expressions

The operators on `StatementFlowable` and `StatementObservable` allow taking different branches at subscription time:

### ifThen - conditionally chose a source to subscribe to

This is similar to the imperative `if` statement but with reactive flows:

```java
if ((System.currentTimeMillis() & 1) != 0) {
    System.out.println("An odd millisecond");
} else {
    System.out.println("An even millisecond");
}


```java
Flowable<String> source = StatementFlowable.ifThen(
    () -> (System.currentTimeMillis() & 1) != 0,
    Flowable.just("An odd millisecond"),
    Flowable.just("An even millisecond")
);
```

source
.delay(1, TimeUnit.MILLISECONDS)
.repeat(1000)
.subscribe(System.out::println);
```

### switchCase - calculate a key and pick a source from a Map

This is similar to the imperative `switch` statement:

```java
switch ((int)(System.currentTimeMillis() & 7)) {
case 1: System.out.println("one"); break;
case 2: System.out.println("two"); break;
case 3: System.out.println("three"); break;
default: System.out.println("Something else");
}
```

```java
Map<Integer, Flowable<String>> map = new HashMap<>();

map.put(1, Flowable.just("one"));
map.put(2, Flowable.just("two"));
map.put(3, Flowable.just("three"));

Flowable<String> source = StatementFlowable.switchCase(
    () -> (int)(System.currentTimeMillis() & 7),
    map,
    Flowable.just("Something else")
);

source
.delay(1, TimeUnit.MILLISECONDS)
.repeat(1000)
.subscribe(System.out::println);
```

### doWhile - resubscribe if a condition is true after

This is similar to the imperative `do-while` loop (executing the loop body at least once):

```java
long start = System.currentTimeMillis();
do {
    Thread.sleep(1);
    System.out.println("Working...");
while (start + 100 > System.currentTimeMillis());
```

```java
long start = System.currentTimeMillis();

Flowable<String> source = StatementFlowable.doWhile(
    Flowable.just("Working...").delay(1, TimeUnit.MILLISECONDS),
    () -> start + 100 > System.currentTimeMillis()
);

source.subscribe(System.out::println);
```


### whileDo - subscribe and resubscribe if a condition is true

This is similar to the imperative `while` loop (where the loop body may not execute if the condition is false
to begin with):

```java

while ((System.currentTimeMillis() & 1) != 0) {
    System.out.println("What an odd millisecond!");
}
```

```java
Flowable<String> source = StatementFlowable.whileDo(
    Flowable.just("What an odd millisecond!"),
    () -> (System.currentTimeMillis() & 1) != 0
);

source.subscribe(System.out::println);
```

## Join patterns

(Conversion done)

TBD: examples

## Debug support

TBD: some of RxJavaDebug and the assembly tracking feature of RxJava 1.x moved here.

# Releases

**gradle**

```
dependencies {
    compile "com.github.akarnokd:rxjava2-extensions:0.2.1"
}
```


Maven search:

[http://search.maven.org](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.github.akarnokd%22)
