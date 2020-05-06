---
layout: post
title: "Effective C++"
date: 2019-12-08
excerpt: "笔记"
tags: [C++]
comments: false
---
**1.Accustoming Yourself to C++**

01.View C++ as a federation  of languages

***multi-paradigm:***

- procedural
- object-oriented
- functional
- generic
- meta

***multi-sublanguages:***  
 
- C
- Object-Oriented C++
- Template C++
- STL


**02.Prefer const, enum, and inline to #define**

const usages: 

- pointer const
- const point to
- point to const
- ***class static const***
	
	keep one copy in all class objects
		
	init value in *.cpp if not supported in *.h

- ***enum hack***

	no memory address
	
	not advance compiler alloc memory for const
	
	base to template meta programming, not a hack

- ***template inline***

	replace #define function, safe to use

**03.Use const whenever possible**

const stl::iterator : const point to

stl::const_iterator : point to const

const member function

- easy to understand
- good support for pass by reference-to-const

bitwise(physical) constness

- can’t modify any member variable(not static)        

logical constness

- In some ways, we need to modify cache in const member function
- mutable(Hi compiler, it always can be modified)
- let non-const member function use a copy of const member function

**04.Make sure that objects are initialized before they are used**

***init built-in variables***

- constructor : member initialization list

***non-built-in parameter’s default constructor occurs before entering constructor***

- copy “nothing” (wanna use default constructor to init)

	    ABEntry::ABEntry()
	    :theName(),
	    theAddress()
	    { }

***multi-constructors***

private assignment function to replace, but it’s pseudo-initialization

***init order problem(non-local static object)***

singleton

**05.Know what functions C++ silently writes and calls.**

    // what happens?
    class Empty
    {
	    public:
	    Empty() {…}; // default ctor
	    Empty(const Empty& rhs) {…}; // copy ctor
	    ~Empty() {…}; // dtor
	    
	    Empty& operator= (const Empty& rhs) {…}; // copy assignment operator
    };

default ctor

create silently when you don’t apply ctor

destructor

non-virtual silently

copy ctor(copy assignment operator)

copy member variable in order silently

member variable is reference/const

compiler refuse to generate silently

it’s derived class but its base class’s copy ctor is private

**06.Explicitly disallow the use of compiler-generated functions you do not want
avoid copy ctor(copy assignment operator)**

declare private

member/friend function can call it, not safe

So, Do not apply implement!

Linkage error will give an notation to users

We can give no name to parameter

Not perfect, we detect error(use copy ctor) in link period

    class Uncopyable
    {
	    protected:
	    Uncopyable() {}
	    ~Uncopyable() {}
	    private:
	    Uncopyable(const Uncopyable&);
	    Uncopyable& operator=(const Uncopyable&);
    }
The Uncopyable class is good to derive your class
It can detect the error use of copy ctor in compile period

class HomeForSale: private Uncopyable
{ };

Boost class : noncopyable class

07.Declare destructors virtual in polymorphic base classes
don’t derive from a base class that has non-virtual dtor
don’t declare virtual dtor for a non-base class(extra memory burden)
final is usable in C++11, it’s good to declare for non-base class

08.Prevent exceptions from leaving destructors
Throw exception in normal member function
Give users the opportunity to decide abort or continue

09.Never call virtual functions during constructions or destructions
constructions:
In base class ctor, virtual function is unable to find derived function
Derived class ctor is not inited.

destructions:
member variable is destroyed before calling function, it’s harmful.

10.Have assignment operators return a references to *this
It’s convenience for us to do something like “x = y = z = 15;”

11.Handle assignment to self in operator=
consider that “parameter object’s address == this” in operator=
identity test
exception will not do something harmful
copy and swap

12.Copy all parts of an object
all member variables
call base class’s copy function

3.Resource Management
13.Use objects to manage resources
RAII : Resources Acquisition Is Initialization
Smart Pointer
call delete automatically in deconstructor but not delete[]
auto_ptr
prev auto_ptr will be null when copy new auto_ptr
std::auto_ptr<Investment> pInv1(createInvestment()); // Init
std::auto_ptr<Investment> pInv2(pInv1); // pInv2 valid, pInv1 is null
pInv1 = pInv2 // pInv1 valid, pInv2 is null

RCSP(Reference-Counter Smart Pointer)
similar to Garbage Collection
disadvantage : Cycles Of References
shared_ptr

14.Think carefully about copying behavior in resource-managing classes
Do not allow copying behaviors(private derived from Uncopyable class)
Allow copying behaviors
RCSP
Deep copying resources
Transfer the control to new RAII copy object(auto_ptr)

15.Provide access to raw resources in resource-managing classes
member function “get()”
pointer dereferencing
operator->
operator*
Do not apply operator transfer silently
convenience but unsafe

16.Use the same form in corresponding uses of new and delete
new -> delete
new [] -> delete []

It’s not recommended that use typedef to define array
For example:
typedef std::string AddressLine[4];
std::string* pal = new AddressLine;
delete pal; // harmful!
delete[] pal; // good

17.Store newed objects in smart pointers in standalone statements
processWidget(std::shared_ptr<Widget>(new Widget), priority());
1.priority()
2.new Widget
3.std::shared_ptr
But 1.priority may be insert into 2. and 3. order.(Optimize or …)
It’s harmful if priority() throw exception and abort.

So, it’s better to write as:
std::shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority());

4.Design and Declarations
18.Make interfaces easy to use correctly and hard to use incorrectly
shared_ptr
custom deleter : resolution of “cross DLL’s problem”

19.Treat class design as type design
emmm…try to think out these problems before designing new class

20.Prefer pass-by-reference-to-const to pass-by-value
avoid object slicing
performance optimize
implement of reference is pointer
a built-in parameter is reasonable to pass-by-value

21.Don’t try to return a reference when you must return a object

22.Declare data members private

23.Prefer non-member non-friend functions to member functions

24.Declare non-member functions when type conversions should apply to all parameters
class Rational
{
public:
Rational(int numerator = 0, int denominator = 1);
int getNumerator() const;
int getDenominator() const;

const Rational operator* (const Rational& lhs, const Rational& rhs);
private:
…
};

Rational result;
Rational a(1, 8);
result = a * 2; // Good
result = 2 * a;    // Bad

Rational constructor is not explicit declare, so in “a * 2”:
const Rational temp(2);
result = a * temp; 
explicit is necessary to avoid transfer int to Rational silently.

Perfect version : non-member
const Rational operator* (const Rational& lhs, const Rational& rhs)
{
return Rational(lhs.getNumerator() * rhs.getNumerator(), 
lhs.getDenominator() * rhs * getDenominator());
}

Is this needs to be a friend to Rational class?
No, it can use getXXX public method to implement.
It has no reason to make a friend with Rational.

25.Consider support for a non-throwing swap
namespace std
{
template<typename T>
void swap(T& a, T& b)
{
T temp(a);
a = b;
b = temp;
}
}
standard swap : performance problem to objects(too many copy times)
pimpl(pointer to implementation)
class WidgetImpl
{
public:
…
private:
int a, b, c;
…
};    

class Widget
{
public:
Widget(const Widget& rhs);
Widget& operator=(const Widget& rhs)
{
…
*pImpl = *(rhs.pImpl);
…
}
private:
WidgetImpl* pImpl;
};

Then let std::swap know it!
Specialization version:
namespace std
{
…
template<>
void swap<Widget>(Widget& a, Widget& b)
{
swap(a.pImpl, b.pImpl);
}
}

Still failure, because pImpl is private, Good Implementation:
class Widget
{
public:
…
void swap(Widget& other)
{
using std::swap;
swap(pImpl, other.pImpl);
}
…
};

namespace std
{
template<>
void Swap<Widget>(Widget& a, Widget& b)
{
a.swap(b);
}
}

Suppose that class Widget and WidgetImpl is template class…
template<typename T>
class WidgetImpl { … };
template<typename T>
class Widget { … };

namespace std
{
template<typename T>
void swap< Widget<T> >(Widget& a, Widget& b)
{
a.swap(b);
}
}

Failure, C++ doesn’t support function template(std::swap), partially specialize
It’s OK to use function reload:
namespace std
{
template<typename T>
void swap(Widget<T>& a, Widget<T>& b)
{
a.swap(b);
}
}

Failure, C++ std doesn’t recommend us to add new templates to std…
Resolution : apply WidgetStuff::swap(name lookup rules will find it, when use swap)
namespace WidgetStuff
{
…
template<typename T>
class Widget { … };
…
template<typename T>
void swap(Widget<T>& a, Widget<T>& b)
{
a.swap(b);
}
}

But something goes uncertain:
when user wants to define a function template:
template<typename T>
void doSomething(T& lhs, T& rhs)
{
// using std::swap; // it’s necessary to tell compiler if no specialization T swap
…
swap(lhs, rhs);
…
}

5.Implementations
define variable too early : performance
too many casts : performance, readable
ignore exception : resource leak, data blocked
use inline too much : code scope larger
coupling : build time larger

26.Postpone variable definitions as long as possible
Outside loop:
Widget w;
for(int i = 0; i < n; ++i)
{
w = …;
…
}

Inside loop:
for(int i = 0; i < n; ++i)
{
Widget w(…);
…
}

Outside loop cost : 1 constructor + 1 destructor + n assignment operator
Inside loop cost : n constructor + n destructor

For performance-sensitive part, we must consider it.

27.Minimize casting
try to hide casting inside function if it truly needs to do
use new C++ style cast:
const_cast : cast away the constness

dynamic_cast : safe downcasting, performance cost(multi-times strcmp call)
try to store derived class pointer in STL container
apply base class a empty implementation member function(For performance)
avoid cascading dynamic_casts

reinterpret_cast : low level cast, “point to int” -> “int”

static_cast : implicit conversions

28.Avoid returning “handles” to object internals
references, pointers, iterators

29.Strive for exception-safe code


