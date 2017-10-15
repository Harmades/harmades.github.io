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

In Java, it is possible to implement the interface, changing the return type to some more specific type. This is called return type covariance :

```java
public interface Cloneable {
    Cloneable clone();
}

public class ACloneable implements Cloneable {
    public ACloneable clone() {return null;}
}

/* Note : I know this is not the way Cloneable are implemented in Java, but let's forget about that for now ;) (you can read more about cloneable elements here) */
```

However, in C#, we're not allowed to do it :

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

C# interface specifies that method signatures must match, otherwise the compiler throws an error to your face. But the Java compiler is happy, which means I've successfully implemented the interface with my ```ACloneable``` class.

For the same reason, a similar case exist with method parameters. It is called input type contravariance.