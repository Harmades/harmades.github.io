---
title: "C# Interface"
date: 2017-10-15T00:41:08+02:00
draft: true
---

<!-- Preview -->

What makes C# interfaces unique in their own way, from C# 1 to C# 8.

<!-- Post -->

In an object-oriented language, interface defines a contract that implementing classes follow.

Java and C# have them, but C# gives you more restrictions about the way you implement interface.

Let's take the cloneable example. If you choose to implement a Cloneable contract, it provides you a clone method, which returns a copy of the object instance.

So you come up with a Cloneable interface, which specifies the clone method. It returns a Cloneable, because you don't know what type the implementing class will be.

In Java, it is possible to implement the interface, changing the return type to some more specific type. It is safe to do so, as you do not break the contract ! This is called return type covariance :

```java
public interface Cloneable {
    Cloneable clone();
}

public class ACloneable implements Cloneable {
    public ACloneable clone() {return null;}
}

/* Note : I know this is not the way Cloneable are implemented in Java, but let's forget about that for now ;) (you can read more about cloneable elements here) */
```

However, in C#, you're not allowed to do it :

```csharp
public interface Cloneable
{
    ICloneable Clone();
}

public class ACloneable : Cloneable
{
    public ACloneable Clone() {return null;} // Compiler error : ACloneable does not implement interface member ICloneable.Clone(), because it does not have the matching return type of ICloneable.
}
```

```
// It allows you to do this (Java)
ACloneable clone = cloneableInstance.clone();

// Instead of this (C#)
ACloneable clone = (ACloneable)cloneableInstance.Clone();
```

The Java code looks a lot nicer to me, because I don't like having to tell the compiler that I know more things (downcasting).

C# interface specifies that method signatures must match, otherwise the compiler throws an error to your face. But the Java compiler is happy, which means I've successfully implemented the interface with my ```ACloneable``` class.

I'm mainly programming in C# right now. Am I stuck with downcasting every single time I want to clone, or encounter similar cases ? No. There's a workaround with explicit interface implementation.

For the same reason, a similar case exist with method arguments, and it's called method arguments contravariance. Java and C# don't allow this, neither C++, as it creates [overloading issues](http://yosefk.com/c++fqa/inheritance-virtual.html#fqa-20.8).