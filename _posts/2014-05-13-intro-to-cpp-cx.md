---
layout: post
title: An Introduction to C++/CX
---

C++/CX is a set of language extensions to C++ that was created to make it easier to consume Microsoft's most recent form of the Window's API.
This API is exposed as a set WinRT classes.
These classes can be used from within a variety of languages.
While there are some C++/CX features to support consuming WinRT classes, it turns out that consuming them in C++ can be handled mostly the same way as consuming other COM classes.
But creating new C++ classes that can be consumed by other languages is harder.
These classes are COM classes under the covers, so all the reasons that make implementing regular COM implementations hard still apply: writing IDL, requiring MIDL, implementing IUknown, class factories, and registration.
Plus, there are new restrictions on interface definitions, three new methods that every class has to implement (IInspectable), and new activation requirements.
C++/CX hides all the COM complexity from the class implementer.
The developer can write a WinRT class almost as easily as writing a regular C++ class and, just by designating it as a ref class, the compiler generates everything necessary to consume the class in any other supported language.

## When to Use C++/CX

When you are writing a view model for XAML in C++, you need to use C++/CX.
You can use any C++/CX features in the view model.
The only other times you are required write a WinRT class is when you are writing a class in C++ and consuming it from a store app written in a different language or you are a dev on the Windows team writing a Windows API class.
You might also consider it for exposing an interface from a view to a view model.
For WinRT authoring scenarios it is recommended to use WRL instead of C++/CX.
Also use WRL to consume WinRT classes in all cases except view models.

## Consuming WinRT Classes win C++/CX

The main new CX features that help in consuming WinRT classes, also known as activatable classes, are built-in ref-counted smart pointers using the ^ (hat) syntax, and "ref new," the mechanism for creating the objects they point to.
For example the following line of code creates a smart pointer to a Calendar object.

```cpp
Windows::Globalization::Calendar^ calendar = ref new Windows::Globalization::Calendar;
```

The calendar object is created in a way that is similar to calling CoCreateInstance() for a COM object.
The calendar pointer manages the ref count as if it were a CComPtr.
Except note that when you pass calendar to a function, the ref count is not increased.
This is an optimization over CComPtr that is only possible because the smart pointer type is part of the language.

Note that you can also use WinRT classes as automatic variables on the stack and as data members: you aren't required to use a heap variable and a ^.

WinRT classes throw exceptions of type Platform::Exception, so be prepared to catch them.

Note that the above is possible because the compiler is automatically reading the metadata from the .winmd.

## WinRT Classes Available to Consume

The kinds of WinRT classes that are available to C++/CX code are the fundamental types in the default namespace, Platform types, Platform collections, and the Windows API.

## Fundamental Types in the Default Namespace

These are the WinRT fundamental numeric types, plus char16 to represent wchar_t characters.

```cpp
uint16
uint32
uint64
int16
int32
int64
float32
float64
char16
```

These are available in the default namespace.
WinRT classes can use these types in their public interfaces.
You can call ToString() on them.
Note that String and Boolean are absent from this list; they are in the Platform namespace.

See <http://msdn.microsoft.com/en-us/library/windows/apps/hh700121.aspx>.

## Platform Namespace

The following are some of the important classes in the Platform namespace.

```cpp
Platform::String
Platform::Boolean
Platform::Array
Platform::Delegate
Platform::Exception
Platform::Object
Platform::WeakReference
```
etc…

For the complete list, see: <http://msdn.microsoft.com/en-US/library/windows/apps/hh710417.aspx>.

## Windows Namespace

The Windows namespace contains the Windows API.
It's best to just point to the API reference.

See <http://msdn.microsoft.com/en-US/library/windows/apps/br211377.aspx>.

## Platform::Collections Namespace

Windows::Foundation::Collections defines a set of interfaces for collections.
Platform::Collections contains concrete implementations of those.
You can use instances of the concrete classes in your code, and pass them via the interfaces in WinRT classes.
You get two kinds of collections:

```cpp
Platform::Collections::Vector
Platform::Collections::Map
```

See <http://msdn.microsoft.com/en-US/library/windows/apps/hh710418.aspx>.

## Authoring WinRT Classes in C++/CX

If need to write a WinRT class, consider using WRL.
It’s more work, but there will be fewer surprises.
Having said that, writing a WinRT class using C++/CX is a straight-forward proposition.
You don't need to create an IDL file or worry about IUnknown or IInspectable.
You create a WinRT class as a "ref class".
When you compile the code, you get a .winmd metadata file along with your dll.
Those are the only files required to consume the class.

A WinRT class can be defined simply as this:

```cpp
namespace HelloDll
{
  public ref class Mood sealed
  {
  public:
    Platform::String^ Target();
    Platform::String^ Current();
  };
}
```

The "ref class" designates the class as a WinRT class.
The class itself is marked "public" and must be in the namespace that has the same name as the .winmd file.
The public interface must only use the WinRT types discussed above.

Note that the methods return the results of the function call, not an HRESULT.
The implementations of the methods should throw exceptions on failure.
Specifically, they should throw some type of Platform::Exception, not arbitrary C++ exceptions.
If your implementation would normally return an HRESULT instead of throwing an exception, you must convert the HRESULT to a Platform::Exception by throwing the results of a call to Exception::CreateException(hr).

```cpp
String^ Mood::Current()
{
  std::wstring retval;
  HRESULT hr = DoTheWorkToGetCurrent(retval);

  if (FAILED(hr))
  {
    throw Platform::Exception::CreateException(hr);
  }

  return ref new String(retval.c_str());
}
```

The compiler will emit code to catch this exception, turn it back into an HRESULT to send across the ABI boundary, then rethrow the exception on the other side.
To do this, it has to also convert your original return result into an out param.
You can avoid this overhead on the WinRT class side of the boundary by using WRL instead of C++/CX because you author your methods to return HRESULTs and use an out param from the start.

See <http://msdn.microsoft.com/en-us/library/hh699870.aspx>.

## Partial Classes

Partial classes are included to support XAML so that tools can generate part of a class definition in one file and a developer can write the rest of it in another.

See <http://msdn.microsoft.com/en-us/library/hh755808.aspx>.

## Properties

Properties are accessor methods of a WinRT class that appear as public data members to the class consumer.
These can have implementations or map directly to data members.

See <http://msdn.microsoft.com/en-us/library/hh755807.aspx>.

## Delegates

The "delegate" keyword is used to declare a type to describe a function signature.
You use this when you want to pass function pointers to WinRT classes for event handler callbacks.

See <http://msdn.microsoft.com/en-us/library/hh755798.aspx>.

## Events

The "event" keyword describes a public data member on a WinRT class that holds one or more event handlers, which are of delegate type.
These event handlers are functions supplied by the consumer of the class using the "+=" operator, which adds them to the classes event handler collection for that event.
The class itself "fires the event" which triggers calls to each event handler in the collection of event handlers for that event.
The "-=" operator is used to remove event handlers.
Events can be customized using add(), remove(), and raise().

See <http://msdn.microsoft.com/en-us/library/hh755799.aspx>.

## Interfaces

A WinRT interface is declared using "interface class" instead of "ref class."
An interface can have public properties, methods, and events, but no data members, private members, or implementation.
It can derive from other interfaces.
Other WinRT classes and structs may derive from and implement an interface.
Generic interfaces are similar to template classes.
Generic interfaces must be private, but specializations may be published as WinRT classes.

See <http://msdn.microsoft.com/en-us/library/hh755792.aspx>.

## Implicit Casting

In C++/CX, you can cast from a ^ pointer of a WinRT class to a ^ pointer of any other interface it implements.
The compiler emits code to call QueryInterface() for this.

See <http://msdn.microsoft.com/en-us/library/hh755802.aspx>.

## More Info

If you are new to C++/CX, watch this video first: Under the covers with C++ for Metro style apps <http://channel9.msdn.com/Events/BUILD/BUILD2011/TOOL-690C>.

C++/CX language reference: <http://msdn.microsoft.com/en-us/library/windows/apps/hh699871.aspx>.

## PS

You can use the undocumented /d1ZWtokens compiler switch to see approximately what your C++/CX code would look like in WRL.

You can use ildasm.exe to browse .winmd files.
