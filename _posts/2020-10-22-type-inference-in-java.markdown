---
layout: post
title:  "Type Inference in Java"
date:   2020-10-22 00:00:00 +0
categories: java
---

Type inference is a technique used by statically typed languages, where the compiler infers the types of variables using the context where they are declared. Programming languages with sophisticated type systems tend to rely more on type inference to improve the readability of the code. For some, this proved to produce more concise code and increase productivity by adding some fun and less typing. Others might feel it's worse because it removes useful information from the code and that makes it less maintainable. There are multiple opinions about it, but the truth is that writing without explicitly specifying types is trending up. There are some good examples of JVM languages with improved type systems that do this amazingly well \(Groovy, Scala and more recently Kotlin\), but it's worth to note how type inference has been evolving in Java and to give an update on the latest features.

### Java's Type Inference System

As a traditional programming language, the Java type inference system has been evolving quite slowly in the last years. Although, Java always provided a very basic form type inference since the first version expressed by one of its main OOP features: inheritance. The subclassing mechanism is natively supported by a subtyping inference on classes.

```java
Object sport = "football"; // sport is a String
```

This very basic form of type inference was considerably improved in Java 5, when generic methods were introduced. The type inference system was then capable of inferring parameterized classes \(type constructors\), subtyping extension and wildcards. For example, instead of:

```java
List<String> sports = Arrays.<String>asList("football", "tennis");
```

Since `asList` implements generics and returns a generic type `List<T>`, this type declaration can be easily inferred be the compiler:

```java
List<String> sports = Arrays.asList("football", "tennis");
```

In Java 7, the scope of this mechanism was slightly extended to infer type parameters of generic constructor invocations with `<>` \(known as _diamond_\). Later in 2014, Java 8 was shipped with type inference for lambda expressions. More recently, project Amber brought in Java 10 & 11 groundbreaking type inference improvements, this time for method local and lambda expression parameters using the keyword _var._ 

```java
var sport = "football"; // sport is a String
```

In my opinion Java is evolving this aspect quite interestingly for a few reasons. It's worth to note some of them.

#### Java Type Inference is not forced

Java does not enforce type inference. This is giving the programmer options and not obligations on when to specify types or let the compiler do that work. It may look a minor detail, but it's an important point in language design when it comes to backward compatibility \(one of Java greatest strengths\) and gives some freedom and flexibility for the developer to decide the syntax that best suits the best.

```java
List<String> winners = Arrays.<String>asList("first", "second"); //type manifest
List<String> winners = Arrays.asList("first", "second"); // inferred type
```

```java
List<String> losers = new ArrayList<String>(); // type manifest
List<String> losers = new ArrayList<>(); // inferred type (diamond operator)
```

```java
List<String> teams = new ArrayList<>(); // inferred type (diamond operator)
var teams = new ArrayList<String>(); // inferred type (local type inference)
```

#### Java Type Inference is nothing else but local

In opposition to other languages, type inference in Java is only local and only does one exact thing: local constraint solving. Instead of a more global approach, Java restricts this mechanism to a method, expression or statement - gathering constraints on unknown types and solving them at some point. There are multiple strategies \(aka algorithms\) to perform these operations in order to cover all edge cases. By design, applying this mechanism to local scopes can be a little bit restricting. On the other hand making types manifest mandatory outside local scopes appears to be a better solution in terms of maintainability \(directly proportional to readability\).

```java
private void resumeMatch(Squad squad) {
    sendScores(new ArrayList<String>());
    publishSquad(squad);
}

private void sendScores(ArrayList<?> scores) {
    List<String> results = scores; // DOES NOT COMPILE
}

private void publishSquad(var squad) { // DOES NOT COMPILE
    squad.publish();
}
```

#### Local Variable Type Inference promotes non-null type initialization

Local variable type inference delivered in Java 10 \(improved in version 11 for lambda expression parameters\) avoids null type initialization by design. And there's a good reason for this - for local variables declared with _var_, the type inference system first computes the type of the initializer, which will be rejected if it is _null_. Hopefully this will contribute to more reliable code.

```java
// DONT DO THIS
public File getFileContent() {
    File file = null;
    try {
        file = readFile("file.json");
    } catch (FileNotFoundException e) {
        // SOME
    }
    return file;
}
```

```java
public File getFileContent() {
    //var file = null; // DOES NOT COMPILE
    try {
        return readFile("file.json");
    } catch (FileNotFoundException e) {
        throw new IllegalStateException("Failed reading file");
    }
}
```

#### Local Variable Type Inference is less restrictive on non-denotable types

When inferring local variables with _var_ , the type can be rejected or special inference rules apply. This happens for non-denotable types. Non-denotable types are those types that can exist within the program, although there's no way to explicitly write out the name for that type. A good example of rejection in _null_ \(mentioned before\), but there are interesting cases of non-denotable types where the usage of _var_ comes with advantages - anonymous classes. When assigning variables with _var_ the type inference system will infer the type differently giving access to members that wouldn't be available using the traditional notation.

```java
var player = new Object() {
    final String name = "John";
    final int goals = 20;
};
System.out.printf("Player %s scored %s goals%n", player.name, player.goals);
```

```java
var ball = new Ball();
var striker = new Striker() {
    @Override
    public Goal shoot(Ball ball) {
        return new Goal(ball);
    }
    public Ball returnBall() {
        return ball;
    }
};
var ballReturned = striker.returnBall();
```

### Conclusion

Compared to other features Java released in the last years, the latest improvements made in the type inference system were one of the most controversial. Local type inference in Java 10 proved this to be true as most people seemed reluctant and described this feature as something just to make Java more popular and trendy \(C\#, Scala or later Kotlin already had this feature\) or as something that could encourage laziness and produce unreadable code. Some years after the release of this feature, the scenario is clear - like anything in programming, local variable inference requires good measure and adoption of [style guidelines](https://openjdk.java.net/projects/amber/LVTIstyle.html).  In the new Java release cycle, Project Amber quickly brought to Java something that the community was asking for a long time. As GraalVM \(and Truffle!\) is gaining popularity, I believe that some of the constraints at interpreter and compiler level will be easier to overcome and Java will bring new type system improvements, faster than ever.

