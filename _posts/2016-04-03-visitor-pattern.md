---
layout: post
title: The Visitor Design Pattern
---

## Introduction

If you come across an instance of the Visitor pattern in somebody's code and you aren't familiar with it, you might find it hard to decipher.
You might even question the sanity of the implementer.
What I want to do here is present the problem the Visitor pattern is designed to solve, and then invent the pattern from scratch in order to solve it.
Hopefully this will help you understand why the pattern exists as well as how it works and how to use it in your own code.

## What problem does the Visitor pattern solve?

The Visitor pattern is often used to visit each element in a heterogeneous collection in order to perform some behavior without mixing up that behavior with the collection or the elements in the collection.
Typically, the following three conditions indicate that the Visitor pattern would be useful:

1. We have a collection of heterogeneous elements.
Usually, the collection is a composite.
Examples include XML, JSON, and HTML trees, ASTs, and UI layout trees.
2. We want some behavior on the elements of that collection where the specific behavior differs depending on the element type.
3. We want the behavior to be external to the collection and elements, not commingled.

## C++ Example: database connections with interface

The Visitor pattern is not unique to C++, but that's how I'll explain it here.

Below, I introduce a simple fictitious interface named `DatabaseConnection` that is used to manage database connections.

```cpp
#include <iostream>
#include <vector>

class DatabaseConnection {
  public:
    virtual ~DatabaseConnection() = default;

    virtual void connect() = 0;
    virtual void disconnect() = 0;
};
```
Next, we have two concrete classes that implement that interface.
```cpp
class SqlServerConnection final : public DatabaseConnection {
  public:

    void connect() override { std::cout << "Connecting to SqlServer\n"; }
    void disconnect() override { std::cout << "Disconnecting from SqlServer\n"; }
};

class AccessConnection final : public DatabaseConnection {
  public:

    void connect() override { std::cout << "Connecting to Access\n"; }
    void disconnect() override { std::cout << "Disconnecting from Access\n"; }
};
```

## Requirement: Disconnect all

Assume we have a container of shared pointers to the `DatabaseConnection` interface.
Our first requirement is for a function that calls `disconnect()` on all of the  elements.
In this case, the desired behavior is internal to the elements and exposed via the interface.
We know how to do this: call a virtual function on each element.

In this case, the desired behavior is internal and exposed via the interface.
We know how to do this: call a virtual function on each element.

```cpp
// Simple when the behavior is internal to the hierarchy.
void disconnectAll(const std::vector<std::shared_ptr<DatabaseConnection>>& connections)
{
  for (const auto& connection : connections) {
    connection->disconnect();
  }
}

```

## Requirement: Count the number of each connection type

Now we have a new requirement: we need to count the number of each connection type.
How should we implement this?

```cpp
// What do we do?
void countConnectionTypes(const std::vector<std::shared_ptr<DatabaseConnection>>& connections)
{
  for (const auto& connection : connections) {
    // ?
  }
}
```

Here are some options:

1. Switch on type: we can add a virtual function that returns the type of the object and then call that in a switch statement to tally up the count of each type.
2. Use `dynamic_cast<>` and test for `nullptr` as a form of reflection and use this to count each type.
3. Implement new virtual functions in the elements that increment counts of each type, probably passing a struct of accumulated counts as an in/out parameter.

We try to avoid any kind of switching on type because it is brittle.
It is usually a code smell that indicates some sort of polymorphism is in order.
Therefore, let's rule out options 1 and 2.
We can also rule out option 3 because we said we don't want to add the new behavior to the concrete `DatabaseConnection` classes.

## How about function overloading?

Will the following code work?

```cpp
class ConnectionTypeCounter {
  public:
    void count(const SqlServerConnection&) {m_sqlServer++;}
    void count(const AccessConnection&) {m_access++;}

  private:
    int m_sqlServer = 0;
    int m_access = 0;
};

void countConnectionTypes(const std::vector<std::shared_ptr<DatabaseConnection>>& connections)
{
  ConnectionTypeCounter counter;
  for (const auto& connection : connections) {
    counter.count(*connection);
  }
}
```

## It doesn't work that way...

It looks like it should, but the compiler can't resolve a call to `ConnectionTypeCounter::count()` with a parameter of type `DatabaseConnection`.

    willthiswork.cpp:41:13: error: no matching member function for call to 'count'
        counter.count(*connection);
        ~~~~~~~~^~~~~
    willthiswork.cpp:29:10: note: candidate function not viable: no known conversion from
      'DatabaseConnection' to 'const SqlServerConnection' for 1st argument
        void count(const SqlServerConnection&) {m_sqlServer++;}
             ^
    willthiswork.cpp:30:10: note: candidate function not viable: no known conversion from
      'DatabaseConnection' to 'const AccessConnection' for 1st argument
        void count(const AccessConnection&) {m_access++;}

`ConnectionTypeCounter` only has overloads for the concrete classes `SqlServerConnection` and `AccessConnection`.
Remember that *overridden* functions are resolved at runtime, and *overloaded* functions are resolved at compile time.
Overloading is just a fancy way of naming functions: think name-mangling.

## ...but we'd like it to work that way

Now let's pass the `ConnectionTypeCounter` to a virtual function in the `DatabaseConnection` hierarchy.
*That* function can then call the overloaded function, passing itself as an argument.
Now the correct overloaded function will be called because it was resolved at compile time based on the argument type.

```cpp
class SqlServerConnection final : public DatabaseConnection {
  public:

    void connect() override { std::cout << "Connecting to SqlServer\n"; }
    void disconnect() override { std::cout << "Disconnecting from SqlServer\n"; }

    void someFunction(const ConnectionTypeCounter& connectionTypeCounter) override {
      connectionTypeCounter.count(*this);
    }
};
```
We need to do this for `AccessConnection` as well.
```cpp
class AccessConnection final : public DatabaseConnection {
  public:

    void connect() override { std::cout << "Connecting to AccessServer\n"; }
    void disconnect() override { std::cout << "Disconnecting from AccessServer\n"; }

    void someFunction(const ConnectionTypeCounter& connectionTypeCounter) override {
      connectionTypeCounter.count(*this);
    }
};
```
And of course we need this new function in the `DatabaseConnection` interface as well.

## Generalize what we just did to get the Visitor pattern

The problem with this approach so far is that we are cluttering the `DatabaseConnection` interface with new methods for each external behavior.
The Visitor pattern is a way to generalize this idea in such a way that a single virtual function can allow for the use of multiple external behaviors packaged as visitors.

To evolve this solution into the Visitor pattern, we follow these steps:

1. Create a visitor interface for the behaviors.
2. Change the `ConnectionTypeCounter` class to a class with "visitor" in the name and implement the visitor interface.
3. Add a new virtual method `accept()` to the `DatabaseConnection` interface to accept a visitor object.
4. Implement `accept()` in each class derived from the `DatabaseConnection` interface.

## Visitor interface

Usually we use "visitor" in the name of this interface.
We need to implement overloaded `visit()` methods for each class in the visitable hierarchy.
The `visit()` method takes a concrete class from the visitable hierarchy as an argument.

```cpp
class DatabaseConnectionVisitor {
  public:
    virtual void visit(const SqlServerConnection&) = 0;
    virtual void visit(const AccessConnection&) = 0;
};
```
Alternatively, we could use a unique function name for each concrete class parameter instead of overloading.
```cpp
class DatabaseConnectionVisitor {
  public:
    virtual void visitSqlServerConnection(const SqlServerConnection&) = 0;
    virtual void visitAccessConnection(const AccessConnection&) = 0;
};
```
This alternative naming convention is common, but I use overloading in this discussion.

## ConnectionTypeCounter repackaged as a visitor

This is what we started with:
```cpp
class ConnectionTypeCounter {
  public:
    void count(const SqlServerConnection&) {m_sqlServer++;}
    void count(const AccessConnection&) {m_access++;}

  private:
    int m_sqlServer = 0;
    int m_access = 0;
};
```
This is our new visitor class:
```cpp
class CountConnectionTypeVisitor final : public DatabaseConnectionVisitor {
  public:
    void visit(const SqlServerConnection&) override {m_sqlServer++;}
    void visit(const AccessConnection&) override {m_access++;}

  private:
    int m_sqlServer = 0;
    int m_access = 0;
};
```
Although most `visit()` implementations would need to use the parameter, this behavior (counting the connection types) doesn't.
Those that do need it are limited to the parameter's public interface.

## Need a new virtual function in the original class hierarchy to accept the visitor

The function that takes a visitor object is named `accept()`.
```cpp
class DatabaseConnection {
  public:
    virtual ~DatabaseConnection() = default;

    virtual void connect() = 0;
    virtual void disconnect() = 0;

    virtual void accept(DatabaseConnectionVisitor& visitor) = 0;
};

```

## Each concrete classes implements `accept()` the same way

```cpp
class SqlServerConnection final : public DatabaseConnection {
  public:

    void connect() override { std::cout << "Connecting to SqlServer\n"; }
    void disconnect() override { std::cout << "Disconnecting from SqlServer\n"; }

    void accept(DatabaseConnectionVisitor& visitor) override { visitor.visit(*this); }; 
};

class AccessConnection final : public DatabaseConnection {
  public:

    void connect() override { std::cout << "Connecting to Access\n"; }
    void disconnect() override { std::cout << "Disconnecting from Access\n"; }

    void accept(DatabaseConnectionVisitor& visitor) override { visitor.visit(*this); }; 
};
```
This hierarchy is now "visitable."

## Last step: pass a concrete visitor to each element in the collection

Create an instance of the visitor and pass it to each element of the collection.
```cpp
void countConnectionTypes(const std::vector<std::shared_ptr<DatabaseConnection>>& connections)
{
  CountConnectionTypeVisitor counterVisitor;
  for (const auto& connection : connections) {
    connection->accept(counterVisitor);
  }
}
```
Sometimes, for example with composites, the iteration is done internally to the collection instead of externally via iterators, as we did above.
In the internal case, we often pass the visitor to the head of the tree, and it is passed to each child element recursively.

Other times, the iteration is done internally by the visitor if different behaviors require different traversal orders.

## All the code so far

Here is code under discussion in one block for easy reading.

```cpp
#include <iostream>
#include <vector>

class SqlServerConnection;
class AccessConnection;

class DatabaseConnectionVisitor {
  public:
    virtual void visit(const SqlServerConnection&) = 0;
    virtual void visit(const AccessConnection&) = 0;
};

class CountConnectionTypeVisitor final : public DatabaseConnectionVisitor {
  public:
    void visit(const SqlServerConnection&) override {m_sqlServer++;}
    void visit(const AccessConnection&) override {m_access++;}

  private:
    int m_sqlServer = 0;
    int m_access = 0;
};

class DatabaseConnection {
  public:
    virtual ~DatabaseConnection() = default;

    virtual void connect() = 0;
    virtual void disconnect() = 0;

    virtual void accept(DatabaseConnectionVisitor& visitor) = 0;
};

class SqlServerConnection final : public DatabaseConnection {
  public:

    void connect() override { std::cout << "Connecting to SqlServer\n"; }
    void disconnect() override { std::cout << "Disconnecting from SqlServer\n"; }

    void accept(DatabaseConnectionVisitor& visitor) override { visitor.visit(*this); }; 
};

class AccessConnection final : public DatabaseConnection {
  public:

    void connect() override { std::cout << "Connecting to Access\n"; }
    void disconnect() override { std::cout << "Disconnecting from Access\n"; }

    void accept(DatabaseConnectionVisitor& visitor) override { visitor.visit(*this); }; 
};

void countConnectionTypes(const std::vector<std::shared_ptr<DatabaseConnection>>& connections)
{
  CountConnectionTypeVisitor counterVisitor;
  for (const auto& connection : connections) {
    connection->accept(counterVisitor);
  }
}

int main()
{
  std::vector<std::shared_ptr<DatabaseConnection>> connections {
      std::shared_ptr<DatabaseConnection>(new SqlServerConnection()),
      std::shared_ptr<DatabaseConnection>(new AccessConnection()) };

  countConnectionTypes(connections);
}
```

## Requirement: dump the type of each element

Adding a new behavior is easy.
Let's say we have a new requirement to dump the type of each element to standard out.
All we need to do is implement the visitor interface again with the new behavior.

```cpp
class DumpConnectionTypeVisitor final : public DatabaseConnectionVisitor {
  public:
    void visit(const SqlServerConnection&) {std::cout << "SqlServer\n"; }
    void visit(const AccessConnection&) {std::cout << "Access\n"; }
};

void dumpConnectionTypes(const std::vector<std::shared_ptr<DatabaseConnection>>& connections)
{
  DumpConnectionTypeVisitor counterVisitor;
  for (const auto& connection : connections) {
    connection->accept(counterVisitor);
  }
}
```

## Advantages of the Visitor pattern

### Clean separation of concerns

The Visitor pattern allows us to create a clean visitable hierarchy with a limited interface and then extend it with loosely-coupled external behaviors, which themselves are not coupled to each other.

### Reduced interface/class size

This separation of concerns leaves us with smaller classes and smaller interfaces.

### Adding new behaviors is safe and easy

The Visitor pattern embraces the open/closed principle.
It is open to extension by allowing new behavior classes to be derived from the visitor interface without making any changes to existing code.
It is closed to modification because no change to existing code is required to add new behavior.
This eliminates the risk of breaking existing code.

### Can accumulate state

Visitor implementations can have member variables to accumulate state without cluttering up the visitable hierarchy.
We saw this in the ``CountConnectionTypeVisitor`` class.

## Problems with the Visitor pattern

### Visitor code is hard to understand

This is the biggest drawback.
Even the function names `accept()` and `visit()` do not express intent well on first reading.
But once we have a working knowledge the pattern, it's okay.

### If the visitable hierarchy changes, the visitors must also change

If we have any visitors implemented and we add a new class to the visitable hierarchy, every visitor must be changed to support it.
The compiler will help us find all the visitors, and all that behavior probably needs to change to support the new class anyway, so this isn't as bad as it seems.

### Requires a robust public interface on the elements

Since the visitable hierarchy passes instances of classes to the visitor, the visitor only has access to its public data members.
If the behavior were to be implemented as member functions instead, it would have access to private members as well.
I'm not sure this is a disadvantage: the public interface should already be robust enough for unit testing.

### Requires an implementation of `accept()` in every class in the visitable hierarchy

It's a minor problem, but each implementation of `accept()` is exactly the same if we use overloading, or almost exactly the same if we use unique function names instead.
This boilerplate code is necessary but adds a small amount of noise to the hierarchy.

### An existing hierarchy can't be made visitable without modification

We must add `accept()` the first time we add a visitor.
If we don't own the hierarchy, we can't retrofit it to accept visitors.

## Conclusion

While the Visitor pattern can be difficult to grok, it is sometimes the right tool for the job.
It pays to know the pattern both for understanding existing code and for solving a particular design problem.
Use it when you must, but as is true for any pattern, don't go visitor-crazy.
