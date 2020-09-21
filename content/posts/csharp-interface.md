+++
title = "Interface in C#"
date = 2017-10-15
tags = ["C#", "Java", ".NET", "Interface"]
+++

# C# vs Java interface

An interface defines a contract that implementing classes must follow. Java and C# of course have them, but C# gives you more (unnecessary) restrictions about the way you implement interface.

Let's take an example : the ```Cloneable``` example. Here's the corresponding interface, in Java, with its method signature:
```java
public interface Cloneable {
    Cloneable clone();
}
```

In Java, the compiler lets you change the return type of the ```clone``` method to some more specific type. It is completely safe to do so, as you do not break the contract ! This is called return type covariance :

```java
public interface Cloneable {
    Cloneable clone();
}

public class ACloneable implements Cloneable {
    public ACloneable clone() { return new ACloneable(); }
}
// Usage
Cloneable cloneableInstance = new ACloneable();
ACloneable clone = cloneableInstance.clone();
```

However, in C#, you're not allowed to do it :

```csharp
public interface ICloneable
{
    ICloneable Clone();
}

public class ACloneable : ICloneable
{
    // Compiler error : ACloneable does not implement interface member ICloneable.Clone(),
    // because it does not have the matching return type of ICloneable.
    public ACloneable Clone() => new ACloneable();

    // Legal implementation.
    public ICloneable Clone() => new ACloneable();
}
// Usage
ICloneable cloneableInstance = new ACloneable();
ACloneable clone = (ACloneable)cloneableInstance.Clone();
```

The Java code looks a lot nicer to me, because I don't like having to tell the compiler that I know more things (downcasting).

C# interface specifies that method signatures must match, otherwise the compiler throws an error to your face. This behaviour is explained in the C# specification. If you've never seen it, go [check it out](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/) ! If you're curious about the language, it really is an interesting document. Interface method implementations behaviour are described as following :

> For purposes of interface mapping, a class member A matches an interface member B when:
> 
> * A and B are methods, and the name, type, and formal parameter lists of A and B are identical.

But the Java compiler is happy, which means I've successfully implemented the interface with my ```ACloneable``` class. As opposed to the C# specification, the Java specification allows it :

> An instance method mC declared in or inherited by class C, overrides from C another method mI declared in interface I, if all of the following are true:
> 
> * I is a superinterface of C.
> * mI is not static.
> * C does not inherit mI.
> * The signature of mC is a subsignature (§8.4.2) of the signature of mI.
> * mI is public

# Are C# programmers restricted by the lack of return type covariance ?

I'm mainly programming in C# right now. Am I stuck with downcasting every single time I encounter the ```ICloneable``` case ? No. There's a workaround : explicit interface implementation.

First, let's see what they are ! Here's an extract of the actual implementation of ```List<T>``` in .NET Core, with the ```Add``` methods. One method comes from the non-generic interface ```IList```, the other one from ```IList<T>```.

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

Explicit interface implementation allows the concrete ```List<T>``` to implement ```IList```, thus ensuring retrocompatibility to the old non-generic interface, while hiding all the methods from the ```IList``` interface. This prevents non-type safe usage of the ```List<T>``` interface with an instance of ```List<T>```. You can still access all the implemented methods, provided that you can the instance to the interface with the hidden methods.

# How does explicit interface implementation solve the ```ICloneable``` problem ?

Well, you can implement ```ICloneable``` explictly, hiding the method from direct usage through your class. Then, define a new method with the correct return type !

```csharp
public interface ICloneable
{
    ICloneable Clone();
}

public class ACloneable : ICloneable
{
    // Consumers will call this method instead, and get ACloneable instead of ICloneable !
    public ACloneable Clone() => new ACloneable();
    
    ICloneable.Clone() => Clone();
}
```

Explicitly implementing interfaces hides the method from direct use with a ```ACloneable``` instance, but not through ```ICloneable```.

# Final words

In C#, you must indeed implement all methods defined by an interface, but it's up to you if you want to expose them directly to the caller. There's no equivalent of this in Java ... both have different point of views regarding what type of contract interfaces define. However, I'd really like to see covariant return type in C# one day (maybe [soon](https://github.com/dotnet/csharplang/blob/master/proposals/covariant-returns.md) ?), because the explicit interface implementation clearly is a workaround. Leave your opinion in the comments !


## Contravariant method argument type

There is a similar case, dealing with method argument. You can read more about that on [wikipedia](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)#Contravariant_method_argument_type).

Java and C# don't allow this, neither C++, as it creates [overloading issues](http://yosefk.com/c++fqa/inheritance-virtual.html#fqa-20.8).