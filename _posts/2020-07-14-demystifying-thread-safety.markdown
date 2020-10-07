---
layout: post
title:  "Demystifying Thread Safety"
date:   2020-07-14 00:00:00 +0
categories: java
---

_* The content of this post was originally published at [Parser Digital Community](https://parserdigital.com/demystifying-thread-safety-2/) *_

Modern programming languages support easy multi-threading out of the box and multitasking has become synonymous with efficiency. Multi-threading grants the ability to simultaneously run or execute multiple tasks, represented as Threads - and it can take different shapes. Multiple patterns were identified, different techniques were conceived and mechanisms have been exhaustively enhanced to simplify and increase the performance of the code in multi-threaded scenarios.

Although it is a powerful feature, multi-threading comes with some common disadvantages. The first disadvantage is the inner **difficulty and complexity** of the subject. As a sensitive topic, it requires maximum attention to detail - is very easy to get wrong and this may lead to **unpredictable and incorrect results**. In worse cases, issues like the well known _deadlock_ can be happen and its misuse can have negative impact on software systems. Another disadvantage id the **performance overhead** than can bring - the creation of native threads have a natural system cost.

As a Software Engineer, one of the most intriguing things that can happen, is the inability and incapacity of accepting the reality without being able to understand, replicate and validate it. For this reason, I consider the **complex to debug and test** the major threat that multi-threading carries.

## The Mystery

In single and distributed systems, multi-threading issues happen often. This usually occurs when these systems share mutable data - an this can come in many different shapes. Accessing databases, orchestrating process between distributed instances... Sometimes the simple absence of [good locking systems](https://docs.hazelcast.org/docs/3.2/manual/html/lock.html) spread the occurrence of race conditions. Database systems already incorporate fine grained concurrency lock based control mechanisms that keep consistency of data. However, most of the times that doesn't avoid getting locking exceptions that some of us seem to struggle to understand.

### Multi-threading in Action

For the sake of simplicity lets analyses the well known Single Pattern implemented in Java.

```java
class MySingleton {

    private static MySingleton mySingleton;

    public static synchronized MySingleton getMySingleton() {
        if (mySingleton == null) {
            mySingleton = new MySingleton();
        }
        return mySingleton;
    }

}
```

At the end, this is not the most accurate example in terms of performance, although, this is considered a way of obtaining a reasonable _thread-safe_ singleton implementation. It uses the lightweight synchronization mechanism on the _getter_ method to guarantee synchronized access to the singleton instance. Have you ever tried to prove this is true?

### Keeping it Simple

In the _Java Concurrent Utilities Framework_ ecosystem, threads can be created by extending the `Thread` class or implementing the `Runnable` interface. The last lacks the ability to make the thread being able to return a value. For supporting this feature, the _Callable_ interface exists. So, consider the following _native_ thread abstration that simply gets the instance of the singleton class.

```java
public class MyThread implements Callable<MySingleton> {

    private final CountDownLatch latch;

    public MyThread(CountDownLatch latch) {
        this.latch = latch;
    }

    @Override
    public MySingleton call() throws InterruptedException {
        latch.await();
        return MySingleton.getMySingleton();
    }

}
```

Please note the parameter `latch`. Restating the source Java-docs, a `CountDownLatch` is _a synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes_. This isn't obviously necessary to get the singleton instance, but it help to will provide the necessary synchronization in the next steps.

The execution method `call` simply invokes and returns the singleton instance.

## The Truth is Worth It

Consider the following unit test. It recreates a multi-threading scenario where 2 threads instances are created and submitted to a _Thread Pool_.

```java
@Test
public void testMySingleton() throws ExecutionException, InterruptedException {

    ExecutorService executorService = Executors.newFixedThreadPool(2);

    CountDownLatch latch = new CountDownLatch(1);

    Future<MySingleton> singleton1 = executorService.submit(new MyThread(latch));
    Future<MySingleton> singleton2 = executorService.submit(new MyThread(latch));

    latch.countDown();

    assertEquals(singleton1.get().hashCode(), singleton2.get().hashCode());

}
```

The `CountDownLatch` mentioned before is initialized with 1 will hold the execution and of both threads, that will be simultaneously released when the `countDown()` is invoked. This strategy is _deffered_ - it implies capturing the value returned by both threads encapsulated in a _Future_. Assertion is based on the object _hashCode_. Same _hashCode_ implies same object instance.

Executing this test doesn't lead to surprising results. The singleton instantiated twice is _thread-safe_ - the first thread will acquire the lock in the object constructor and instantiate the static field. The second thread will simply return the instance.

### Silver Lining

From the beginning, a _thread-safe_ scenario was considered. Removing the `synchronised` keyword from the singleton _getter_ method should, hence, put in cause thread safety. Executing the same test with the suggested change leads to nondeterministic failures because the Singleton is created twice.

```java
java.lang.AssertionError: 
Expected :125993742
Actual   :1192108080
```

This is one of the reasons why the Singleton pattern is already considered an _anti-pattern_. Without proper synchronization, the implementation ridiculously nullifies its own purpose.

## Conclusion

Thread safety is a hard subject but it doesn't necessarily have to be mystery for any developer. While avoiding multi-threading issues at all is a big challenge, being able to understand them _a priori_ brings value. As shown possible issues and their impact can be mitigated during development leading to more solid and scalable systems. You might think that the scenario presented is `oversimplified and optimistic` , however when applied to more complex tasks the same rules apply. I always keep this pieces of code close.