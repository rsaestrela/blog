---
layout: post
title:  "How interesting was CMS GC"
date:   2020-09-01 00:00:00 +0
categories: java
---

In the last few years, a lot of changes happened regarding Garbage Collection for the HotSpot Java VM. The garbage collector G1 became popular and production-ready in JDK 8u40, being adopted as the [default collector in Java 9](http://openjdk.java.net/jeps/248). Some more collectors were made available for an experimental period. Just to name a few: [ZGC](https://openjdk.java.net/jeps/333), [Shenandoah](https://openjdk.java.net/jeps/189) or [Epsilon](https://openjdk.java.net/jeps/318) - which is a collector designed for testing purposes only that doesn't collect garbage \(zero-effort\). 

With so many names popping I've decided to recap a bit my knowledge on garbage collection theory and review how the old Java collectors used work compared to the new ones. During my research I found some interesting details I didn't know about, **particularly related with CMS \(Concurrent Mark and Sweep\)**. It's known that even before Java 8, collectors were concurrent. In order to reduce STW \(stop-the-world\), these collectors use all available system threads to execute their tasks. A good example is ParallelGC that makes use of [Weak Generational Hypothesis](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/generations.html) by touching only live objects and it uses all cores as much as possible to shorten the STW pause time. As an evacuating collector, ParallelGC promotes objects that have survived GC to Tenured space and passes that responsibility for ParallelOld. 

ParallelOld is a compacting collector with only a single contiguous memory space which makes it very efficient in its use of memory \(avoiding fragmentation\). By default the size of old generation is 7 times bigger than young, and due to the fact that the marking time is proportional to the number of live objects in this region \(considering long-lived objects as well\) the STW time scales almost linearly with the size of the heap. As a result, ParallelOld starts to scale badly in terms of a STW time.

### CMS

As the default collector for the Tenured space in Java 7, CMS was designed to make a different usage of concurrency. In order to reduce pause time, CMS does as much work as possible while application threads are still running. It's a fact that CMS uses more CPU and memory and it doesn't compact heap, so Tenured can become fragmented. In a nutshell, these are its phases:

1. Initial Mark \(STW\) - Provides a stable set of starting points for GC that are within the region; these are known as internal pointers and provide a equivalent set to the GC roots for the purpose of collection cycle
2. Concurrent Mark \(STW\) - Runs the tri-color marking algorithm on the heap and keeps track of any changes that might might later require fix
3. Concurrent Preclean
4. Remark Phase
5. Concurrent Sweep
6. Concurrent Reset

Until now, all good. Now the interesting stuff commences.

#### CMS coordination with Young GC 

What happens if Eden fills up while CMS is running? Because application threads cannot continue, they pause and a STW on the young runs while GC is running for the old generation. This young GC run will usually take longer than in the case of parallel collectors, because it only has half the cores available for the young generation \(the other half of the cores are running CMS\). At the end of this young collection, these objects need to be moved into Tenured while CMS is still running, which requires some coordination between the two collectors. **This is why CMS requires a slightly different young collector. CMS cannot be used with Parallel GC. By default, HotSpot sets ParNew to perform GC on the young generation.**

#### Fallback to ParallelOld

An interesting thing that may happen is called _Concurrent Mode Failure_ \(CMF\). When there is a very high premature promotion in the young collection, the JVM has no choice but to fallback to ParallelOld \(which is 100% STW\). Effectively, the allocation pressure can be so high that CMS doesn't have time to finish processing the old generation before all the space to accommodate newly promoted objects. In order to prevent this issue, by default CMS starts a collection at 75% of it's capacity.

#### Fallback to ParallelGC

Another common issue that leads to CMF is the heap fragmentation. CMS does not support compact Tenured and this means that after a collection, the free space in Tenured is not a single contiguous block, and objects that are promoted have to be filed into the gaps between existing threads. If this is not possible, in this case, there's a fallback to ParallelGC. Known for being good at avoiding memory fragmentation, ParallelGC takes action and does a fully STW and compacting collection.

### Conclusion

At the time it was designed, CMS was the solution to overcome the long STW pauses that the previous algorithms implied. But as a highly and sophisticated algorithm, CMS caused a lot of complexities to the GC code base in JDK. It counts with around 72 configuration flags, on top of the common 50 which makes it hard to understand and tune. Oracle's answer for this problem was G1, that is much easier to tune and avoids the problems described before. CMS was [removed](https://openjdk.java.net/jeps/363) from JDK on release 14. As the default algorithm since Java 9, G1 it's [constantly improving](https://openjdk.java.net/jeps/307) and more than ever, Oracle is pushing new solutions for garbage collection. Some of the new GC solutions mentioned in the introduction, don't really represent a better solution or a replacement for G1, but a different view or approach that might work better for different and very specific use cases. At the time of this writing, Oracle is preparing the release of JDK 15 bundled with [ZGC](https://openjdk.java.net/jeps/377) and [Shenandoah](https://openjdk.java.net/jeps/379) production-ready. It's case to say, the future of Java is now!

**References**

* [Java Platform, Standard Edition Documentation](https://docs.oracle.com/en/java/javase/index.html) \(Java Virtual Machine Guide & Garbage Collection Tuning\)
* Benjamin J Evans, James Gough & Chris Newland \(2018\) Optimizing Java: Practical techniques for improving JVM application performance