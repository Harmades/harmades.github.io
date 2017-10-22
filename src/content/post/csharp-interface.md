+++
title = "Interface in C#"
date = 2017-10-15T00:41:08+02:00
+++

What makes C# interfaces unique in their own way, from C# 1 to C# 8.

<!--more-->

# C# vs Java interface

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
    public ACloneable clone() { return new ACloneable(); }
}

/* Note : I know this is not the way Cloneable are implemented in Java, but let's forget about that for now ;) (you can read more about cloneable elements here) */
```

However, in C#, you're not allowed to do it :

```csharp
public interface ICloneable
{
    ICloneable Clone();
}

public class ACloneable : ICloneable
{
    public ACloneable Clone() { return new ACloneable(); } // Compiler error : ACloneable does not implement interface member ICloneable.Clone(), because it does not have the matching return type of ICloneable.
}
```

```
// It allows you to do this (Java)
ACloneable clone = cloneableInstance.clone();

// Instead of this (C#)
ACloneable clone = (ACloneable)cloneableInstance.Clone();
```

The Java code looks a lot nicer to me, because I don't like having to tell the compiler that I know more things (downcasting).

C# interface specifies that method signatures must match, otherwise the compiler throws an error to your face. This behaviour is explained in the C# specification. If you've never seen it, go check it out ! If you're curious about the language, it really is an interesting document. Interface method implementations behaviour are described as following :

> For purposes of interface mapping, a class member A matches an interface member B when:
> * A and B are methods, and the name, type, and formal parameter lists of A and B are identical.

But the Java compiler is happy, which means I've successfully implemented the interface with my ```ACloneable``` class. As opposed to the C# specification, the Java specification allows it :

> An instance method mC declared in or inherited by class C, overrides from C another method mI declared in interface I, if all of the following are true:
> * I is a superinterface of C.
> * mI is not static.
> * C does not inherit mI.
> * The signature of mC is a subsignature (§8.4.2) of the signature of mI.
> * mI is public

# Are C# programmers restricted by the lack of return type covariance ?

I'm mainly programming in C# right now. Am I stuck with downcasting every single time I encounter the ICloneable case ? No. There's a workaround : explicit interface implementation.

First, let's see what they are ! Here's an extract of the actual implementation of List<T> in .NET Core, with the Add methods. One method comes from the non-generic interface IList, the other one from IList<T>.

```csharp
    public class List<T> : IList<T>, System.Collections.IList, IReadOnlyList<T>
    {
        int System.Collections.IList.Add(Object item) {...}
        int Add(T item) {...}
    }

    var list = new List<string>();
    list.Add("OK");
    list.Add((object)"NOK"); // Compile-time error.
    var nonGenericList = (IList)list;
    nonGenericList.Add(3); // Runtime error, but compiles !
```

Explicit interface implementation allows the concrete List<T> to implement IList, thus ensuring retrocompatibility to the old non-generic interface, while hiding all the methods from the IList interface. This prevents non-type safe usage of the List<T> interface with an instance of List<T>. You can still access all the implemented methods, provided that you can the instance to the interface with the hidden methods.

# How does explicit interface implementation solve the ICloneable problem ?

Well, you can implement ICloneable explictly, hiding the method from direct usage through your class. Then, define a new method with the correct return type !

```csharp
public interface ICloneable
{
    ICloneable Clone();
}

public class ACloneable : ICloneable
{
    public ACloneable Clone() { return new ACloneable(); }
    
    ICloneable.Clone() { return new ACloneable(); }
}
```

For the same reason, a similar case exist with method arguments, and it's called method arguments contravariance. Java and C# don't allow this, neither C++, as it creates [overloading issues](http://yosefk.com/c++fqa/inheritance-virtual.html#fqa-20.8).