---
layout: post
title:  "Threading for Performance"
date:   2020-06-20 00:00:00 +0
categories: java
---

In the _Information Age_, we all want answers from the web as quick as possible. Compared to the past software systems nowadays need to process a huge amount of data, in very different hardware contexts where cloud based systems are a reality. Now, the market demands responsive, loosely coupled, scalable, resilient and message-driven systems.

When Architecting a system for performance, multitasking is a must. The obsolete single-threaded applications are not an option anymore and modern systems need to implement multi-threading mechanisms as the answer to the market performance needs. Although it might sound a great idea, traditional multi-threading comes at a cost. It carries natural and inner performance disadvantages usually referred as _performance overhead_.

The performance overhead associated with multi-threading, regards mainly to the fact that, the CPU has to switch the execution between threads. When switching, it has to save the register state of a thread and load the state of a different thread. Modern processors are very good at this as they implement high performance caches. But caches are finite. When caches are full, processors must evict data to make room for new data. Thus software threads tend to evict each other's data, and the cache fighting from too many threads can hurt performance. As with caches, time slicing accessing Virtual Memory causes threads to fight each other for real memory for its stack and private data structures. In extreme cases, there can be so many threads that the program even can run out of virtual memory.

While the hardware industry is making remarkable progress improving hardware systems, the software industry is in constant adaptation, visioning and implementing different approaches to take the best out of hardware. It might sound confusing, but this has shown to be the symbiosis of success.

## Reactive

Reactive is a _buzzword_, you should have heard about it - and you might think this is something new. The term has been in use since the paper, _The Reactive Engine_ was made available by Alan Kay in 1969. A few years later, in 1997 Conal Elliot and Paul Hudak came up with _Function Reactive Animation_. But it was only in 2014 when this topic was rekindled with the _Reactive Manifesto_.

Through constant change, organizations had to reinvent their software products.

> _These changes are happening because application requirements have changed dramatically in recent years. Only a few years ago a large application had tens of servers, seconds of response time, hours of offline maintenance and gigabytes of data. Today applications are deployed on everything from mobile devices to cloud-based clusters running thousands of multi-core processors. Users expect millisecond response times and 100% up-time. Data is measured in Petabytes. Today's demands are simply not met by yesterday’s software architectures._
>
> [https://www.reactivemanifesto.org/](https://www.reactivemanifesto.org/)

Reactive applications focuses on the systems that react to the events, the varying loads, and multiple users. It also react effectively to all the conditions, whether it's successful or has failed to process requests. The Reactive Manifesto came as attempt to define a new industry standard. It defines four characteristics to design Reactive Systems.

> _We believe that a coherent approach to systems architecture is needed, and we believe that all necessary aspects are already recognized individually: we want systems that are Responsive, Resilient, Elastic and Message Driven. We call these Reactive Systems._
>
> [https://www.reactivemanifesto.org/](https://www.reactivemanifesto.org/)

### Reactive Programming

With the advent of Reactive Systems, the industry of software conceived a new paradigm called Reactive Programming.

> _RP designs code in such a way that the problem is divided into many small steps, where each step or task can be executed asynchronously without blocking each other. Once each task is done , it's composed together to produce a complete flow without getting bound in the input or output._
>
> **Tejaswini Mandar Jog** - Reactive Programming With Java 9

In another words, Reactive Programming disseminates the idea of instead of creating a thread for each blocking concurrent task, a dedicated thread commonly called _event loop_ looks to all the tasks that are assigned to threads in a non-reactive model, and processes each of them on the same process. This new paradigm reduced the problem of context switch, by imposing an asynchronous and non-blocking mindset. As a result, the performance and the utilization of resources on a multi-core is increased and it provided a more maintainable approach to deal with asynchronism.

But writing high-performance, efficient, scalable and correct Reactive software is hard. With the rise of Functional Programming a new paradigm was born - Functional Reactive Programming. An abstraction on top of imperative systems that allows to program asynchronous and event-driven functional programs avoiding the pitfalls \(like the well known _callback hell_\) of reactive imperative code. It established a concrete functional approach to model programs behavior and how behavior changes according to events.

It was the beginning of a new movement called _Reactive Extensions_. An API for asynchronous programming with observable streams of events. Reactor, RxJava or RxNet just to name a few are new set of tools created under the name of Reactive Extensions or ReactiveX. Combining ideas from the _Observer_ pattern, _Iterator_ pattern and Functional programming , reactive extensions allow _composing asynchronous and event-based programs by using observable sequences_.

In 2017, Java 9 introduced the Flow API, a _Reactive Streams_ specification as a standard to build Stream-oriented libraries for reactive and non-blocking JVM-based systems. The library RxJava was adapted to depend on this this new specification.

## Coroutines

Although Reactive is a very powerful concept when applied to programming , libraries didn't provide the right solution as a general approach to manage asynchronous processing. While frameworks are still embracing, reinventing mechanisms and providing better reactive integrations, some natural pitfalls such as complexity, natural performance overhead and code readability forced the Industry to come up with a more practical and simple approach. _Jetbrains_ and the community reinvented an old concept and the result was - Coroutines.

Coroutines are commonly described as lightweight threads. It's a fact that they are conceptually very similar to threads, however they do not mirror the traditional threading that involves native system calls. They run on native threads and they do not depend on any synchronization mechanism such as mutexes or semaphores. The natural performance overheard is then nullified and this makes them highly performance and convenient to implement multitasking on applications.

Although in a total different context, Kotlin Coroutines can be seen like a very well designed and sophisticated variation of existing solutions - such as the well know _Promise_ \(ES6\) , 2 years latter improved with ECMAScript 2017 with the _async_ and _await_ mechanisms.

Coroutines are great as they provide a very solid multitasking context that allows _structured concurrency_, _scopes_, _cancellation & timeouts_, _composition_ and _asynchronous flows_ just to name a few treats - allied with a notable syntax and a fast learning curve.

```kotlin
fun main(args: Array<String>) = runBlocking {
    launch {
        printWorld()
    }
    println("Hello,")
}

private suspend fun printWorld() {
    delay(1000L)
    println("World!")
}
```

This increased incredibly the appetite to migrate existing Reactive solutions to Coroutines. Specially in Android development where the existing approach was complex and a simpler and more convenient way to make applications flow asynchronous was necessary to improve the performance of the applications.

Please note the code block above. As the _suspend_ function is asynchronous, `runBlocking` scope blocks the current thread for waiting the asynchronous code to complete. There’s no magic, and a synchronous, blocking I/O or mutex call will remain thread-blocking outside of the coroutine scope. So, what's next?

## Project Loom

The current JVM model, creates an operating system thread per Java thread. Project Loom is a proposal to enhance the JVM and the Java Library in order to support the ability to create lightweight threads that run on top of conventional threads.

The lightweight threads will be called **Fibers** and the coroutine logic will be called **Continuation**. Similar to Kotlin `suspend` functions, Continuations will have an entry point and a _yield_ point that corresponds to the point were the execution can be suspended. If the caller resumes the continuation, the control returns to the last point. When a Fiber encounters a blocking call, it won't unblock the native thread - but the Fiber is suspended until this action to complete switching the execution to a different Fiber. Fibers run in a thread pool, yielding cooperatively. Switching execution between Fibers is a cheap operation.

Reactive and asynchronous programming usually lead to cognitive complexity, lost of context and control. In opposition, Loom's approach is not based in a high level asynchronous mechanism. It will give the simplicity of synchronous operations with the performance cost of asynchronous. Blocking executions executed in Fibers will use non-blocking I/O under the hood. For the developer, it will be much easier to develop and maintain high performance systems since the complexity will "reside" at JVM level, leveraging this functionality consistently to different platforms.

I really recommend to read the [Loom Proposal](https://cr.openjdk.java.net/~rpressler/loom/Loom-Proposal.html).

OpenJDK already gave a preview early access to project Loom. Some [tests already performed](https://dzone.com/articles/a-new-java-with-a-stronger-fiber) accuse that with Loom it will be possible to create 1000 times more fibers than threads, and creating them is 70 times faster.

## Conclusion

World Economy requires more than ever performance at a low cost. In a _Age_ where data and computing is in exponential grown, Architecting software systems with Performance in mind is a must.

We usually say that _Technology is Eating the World_. In a World more aware of sustainability, optimization was never so important - this is happening in many different industries. And software is not an exception to the rule. We have to improve, otherwise the World will eat us.