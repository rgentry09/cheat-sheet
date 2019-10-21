# C++ Syntax Cheat Sheet

## Preface
Since the C++ language varies so heavily between versions (e.g. C++0x, C++11, C++17, etc.), I will preface this cheat sheet by saying that the majority of the examples here target C++0x or c++11, as those are the versions that I am most familiar with. I come from the aerospace industry (embedded flight software) in which we purposefully don't use the latest technologies for safety reasons, so most of the code I write is in C++0x and sometimes C++11. Nevertheless, the basic concepts of C++ and object oriented programming still generally apply to both past and future versions of the language.

## Table of Contents

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:0 orderedList:0 -->

- [C++ Syntax Cheat Sheet](#c-syntax-cheat-sheet)
	- [Table of Contents](#table-of-contents)
	- [1.0 C++ Classes](#10-c-classes)
		- [1.1 Class Syntax](#11-class-syntax)
			- [1.1.1 Class Declaration (`.h` file)](#111-class-declaration-h-file)
			- [1.1.2 Class Definition (`.cpp` file)](#112-class-definition-cpp-file)
			- [1.1.3 Class Utilization (Another `.cpp` file)](#113-class-utilization-another-cpp-file)
			- [1.1.4 Getters and Setters](#114-getters-and-setters)
		- [1.2 Inheritance](#12-inheritance)
			- [1.2.1 `Rectangle` Declaration (`.h` file)](#121-rectangle-declaration-h-file)
			- [1.2.2 `Rectangle` Definition (`.cpp` file)](#122-rectangle-definition-cpp-file)
			- [1.2.3 `Rectangle` Utilization (Another `.cpp` file)](#123-rectangle-utilization-another-cpp-file)
		- [1.3 Class Polymorphism](#13-class-polymorphism)
			- [1.3.1 Motivation](#131-motivation)
			- [1.3.2 Virtual Methods](#132-virtual-methods)
		- [1.4 Special Methods (Constructor, Destructor, ...)](#14-special-methods)
			- [1.4.1 Constructor and Destructor](#141-constructor-and-destructor)
				- [1.4.1.1 Use of `explicit` in Constructors](#1411-use-of-explicit-in-constructors)
				- [1.4.1.2 Member Initializer List](#1412-member-initializer-list)
			- [1.4.2. `new` and `delete`](#142-new-and-delete)
			- [1.4.3. Copy Constructor and Copy Assignment](#143-copy-constructor-and-copy-assignment)
			- [1.4.4. Move Constructor and Move Assignment](#144-move-constructor-and-move-assignment)
		- [1.5 Operator Overloading](#15-operator-overloading)
		- [1.6 Templates](#16-templates)
	- [2.0 General C++ Syntax](#20-general-c-syntax)
		- [2.1 Namespaces](#21-namespaces)
		- [2.2 References/Pointers](#22-references-and-pointers)
		- [2.3 Keywords](#23-keywords)
			- [2.3.1 General keywords](#231-general-keywords)
			- [2.3.2 Storage class specifiers](#232-storage-class-specifiers)
			- [2.3.3  `const` and `dynamic` Cast Conversion](#233-const-and-dynamic-cast-conversion)
		- [2.4 Preprocessor Tokens](#24-preprocessor-tokens)
		- [2.5 Strings ](#25-strings-stdstring)
		- [2.6 Iterators](#26-iterators-stditerator)
		- [2.7 Exceptions](#27-exceptions)
			- [2.7.1 Motivation](#271-motivation)
			- [2.7.2 Standard exception hierarchy](#272-stdexception)
		- [2.8 Lambdas](#28-lambdas)

<!-- /TOC -->


## 1.0 C++ Classes
### 1.1 Class Syntax
#### 1.1.1 Class Declaration (`.h` file)
Here's a simple class representing a polygon, a shape with any number of sides.

The class *declaration* typically goes in the header file, which has the extension `.h` (or, less commonly, `.hpp` to distinguish from C headers). The *declaration* gives the class name, any classes it may extend, declares the members and methods, and declares which members/methods are public, private, or protected. You can think of the declaration as sort of saying: "there will be a thing and here's how it will look like". The declaration is used to inform the compiler about the future essence and use of a particular symbol. 

```c++
// File: polygon.h

#include <string>

class Polygon {

// Private members and methods are only accessible via methods in the class definition
private:
    int num_sides;    	// Number of sides

// Protected members and methods are only accessible in the class definition or by classes who extend this class
protected:
    std::string name;   // Name of the polygon

// Public members and methods are accessible to anyone who creates an instance of the class
public:
    // Constructors
    Polygon(const int num_sides, const std::string & name); // <--- This constructor takes the number of sides and name as arguments

    // Getters and Setters
    int GetNumSides(void) const;
    void SetNumSides(const int num_sides);

    std::string & GetName(void) const;
    void SetName(const std::string & name);
    
}; // <--- Don't forget the semicolon!
```

#### 1.1.2 Class Definition (`.cpp` file)
The class *definition* typically goes in the `.cpp` file. The *definition* extends the declaration by providing an actual implementation of whatever it is that you're building. Continuing the example from the declaration, the definition can be thought of as saying: "Right, that thing I told you briefly about earlier? Here's how it actually functions". The definition thus provides the compileable implementation.

```c++
// File: polygon.cpp

#include <string>	// <--- Required for std::string

#include "polygon.h"    // <--- Obtains the class declaration

// Constructor
// You must scope the method definitions with the class name (Polygon::)
// Also, see the section on the 'explicit' keyword for a warning about constructors with exactly one argument
Polygon::Polygon(const int num_sides, const std::string & name) {
    this->num_sides = num_sides;	// 'this' is a pointer to the instance of the class. Members are accessed via the -> operator
    this->name = name;			// In this case you need to use 'this->...' to avoid shadowing the member variable since the argument shares the same name
}

// Get the number of sides
int Polygon::GetNumSides(void) const {	// The 'const' here tells the compiler that you guarantee that you won't modify the object when this function is called. This allows it to perform optimizations that it otherwise may not be able to do
    return this->num_sides;
}

// Set the number of sides
void Polygon::SetNumSides(const int num_sides) {
    this->num_sides = num_sides;
}

// Get the polygon name
std::string & Polygon::GetName(void) const {
    return this->name;
}

// Set the polygon name
void Polygon::SetName(const std::string & name) {
    this->name = name;
}
```

Regarding the use of `this->` in a class definition, there are places where it's strictly necessary for readability, e.g. when your method parameter shares the exact same name as a member variable, you use `this->` to avoid what's called shadowing. However, some prefer to always use `this->` explicitly regardless of whether it's necessary.

#### 1.1.3 Class Utilization (Another `.cpp` file)
```c++
// File: main.cpp

#include <string>
#include <iostream>

#include "Polygon.h"    // <--- Obtains the class declaration

int main(int argc, char * argv[]) {
    // Create a polygon with 4 sides and the name "Rectangle"
    Polygon polygon = Polygon(4, "Rectangle");

    // Check number of sides -- Prints "Rectangle has 4 sides"
    std::cout << polygon.GetName() << " has " << polygon.GetNumSides() << " sides"<< std::endl;

    // Change number of sides to 3 and rename to "Triangle"
    polygon.SetNumSides(3);
    polygon.SetName("Triangle");
}
```

#### 1.1.4 Getters and Setters
A shortcut often used for Getters/Setters is to define them in the class declaration (`.h`) file as follows:
```c++
// File: car.h

#include <string>

class Car {
private:
	int year;
	std::string make;

public:
	int GetYear(void) const { return this->year; }
	void SetYear(const int year) { this->year = year; }
	std::string & GetMake(void) const { return this->make; }
	void SetMake(const std::string & make) { this->make = make; }
};
```

This is often used for very basic getters and setters, and also for basic constructors. In contrast, you'll nearly always find more complex methods defined in the `.cpp` file. One exception to this is with class templates, in which the entire templated class declaration and definition must reside in the header file.

Another important consideration: If you have getters and setters for all of your members, you may want to reconsider the design of your class. Sometimes having getters and setters for every member is indicative of poor planning of the class design and interface. In particular, setters should be used more thoughtfully. Could a variable be set once in the constructor and left constant thereafter? Does it need to be modified at all? Is it set somewhere else in another method, perhaps even indirectly?

### 1.2 Inheritance
A class can extend another class, meaning that the new class inherits all of the data from the other class, and can also override its methods, add new members, etc. Inheritance is the key feature required for [polymorphism](#13-class-polymorphism).

It is important to note that this feature is often overused by beginners and sometimes unnecessary hierarchies are created, adding to the overally complexity. There are some good alternatives such as [composition](https://en.wikipedia.org/wiki/Composition_over_inheritance) and [aggregation](https://stackoverflow.com/a/269535), although, of course, sometimes inheritance is exactly what is needed.

**Example:** the class `Rectangle` can inherit from the class `Polygon`. You would then say that a `Rectangle` extends from a `Polygon`, or that class `Rectangle` is a sub-class of `Polygon`. In plain English, this means that a `Rectangle` is a more specialized version of a `Polygon`. Thus, all rectangles are polygons, but not all polygons are rectangles.

#### 1.2.1 `Rectangle` Declaration (`.h` file)
```c++
// File: rectangle.h

#include <string>       // <--- Explicitly include the string header, even though polygon.h also includes it

#include "polygon.h"	// <--- You must include the declaration in order to extend the class

// We extend from Polygon by using the colon (:) and specifying which type of inheritance
// will be used (public inheritance, in this case)

class Rectangle : public Polygon {
private:
	int length;
	int width;
    
    // <--- NOTE: The member variables 'num_sides' and 'name' are already inherited from Polygon
    //            it's as if we sort of get them for free, since we are a sub-class

public:
	// Constructors
	explicit Rectangle(const std::string &name);
	Rectangle(const std::string &name, const int length, const int width);

	// Getters and Setters
	const int GetLength(void) const { return this->length; }
	void SetLength(const int) { this->length = length; }

	const int GetWidth(void) const { return this->width; }
	void SetWidth(const int) { this->width = width; }
    
    // <--- NOTE: Again, the getters/setters for 'num_sides' and 'name' are already inherited from Polygon

	// Other Methods
	const int Area(void) const;
};
```

> NOTE: The inheritance access specifier (`public`, `protected`, or `private`) is used to determine the [type of inheritance](https://www.tutorialspoint.com/cplusplus/cpp_inheritance.htm). If this is omitted then `private` inheritance is used by default. **Public inheritance is by far the most common type of inheritance**.

#### 1.2.2 `Rectangle` Definition (`.cpp` file)
```c++
// File: rectangle.cpp
			
#include "rectangle.h"	// <--- Only need to include 'Rectangle', since 'Polygon' is included in 'rectangle.h'

// This constructor calls the superclass (Polygon) constructor and sets the name and number of sides to '4', and then sets the length and width
Rectangle::Rectangle(const std::string &name, const int length, const int width) : Polygon(4, name) {
	this->length = length;
	this->width = width;
}

// This constructor calls the superclass (Polygon) constructor, but sets the length and width to a constant value
// The explicit keyword is used to restrict the use of the constructor. See section below for more detail
explicit Rectangle::Rectangle(const std::string &name) : Polygon(4, name) {
	this->length = 1;
	this->width = 1;
}

// Compute the area of the rectangle
int Rectangle::Area(void) const {
	return length * width;		// <--- Note that you don't explicitly need 'this->', you can directly use the member variables
}
```

#### 1.2.3 `Rectangle` Utilization (Another `.cpp` file)
```c++
// File: main.cpp

#include <iostream>

#include "Rectangle.h"

int main(int argc, char *argv[]) {
	Rectangle rectangle = Rectangle("Square", 6, 6);

	// Prints "Square has 4 sides, and an area of 36"
	std::cout << rectangle.GetName() << " has " << rectangle.GetNumSides() << " sides, and an area of " << rectangle.Area() << std::endl;
}
```

### 1.3 Class Polymorphism
Polymorphism describes a system in which a common interface is used to manipulate objects of different types. In essence various classes can inherit from a common interface through which they make certain guarantees about which methods/variables are available for use. By adhering to this common interface, one can use a pointer to an object of the base interface type to call the methods of any number of extending classes. Using polymorphism one can say "I don't care what type this really is; I know it implements `Foo()` and `Bar()` because it inherits from this interface", which is a pretty nifty feature.

The `virtual` keyword is used to ensure runtime polymorphism for class methods. Additionally, an overriding method can be forced by the compiler by not providing a default implementation in the interface, which is done by setting the method to `= 0`, as will be shown later.

#### 1.3.1 Motivation
Let's consider a similar class hierarchy using shapes as previously discussed. Considering a shape to be any 3 or more sided polygon from which we can compute certain attributes (like the shape's area), let's extend from it to create a rectangle class from which we can set the length/width and a circle class in which you can set the radius. **In both cases, we want to be able to compute the area of the shape.** This is a key observation that we will expand upon later.

For now, this (poorly implemented) shape class will suffice:

```c++
// File: shape.h

#include <cmath> 	// needed for M_PI constant

class Shape {
    // We'll leave Shape empty for now... not very interesting yet
};

class Rectangle : public Shape {
private:
    double length;
    double width;
    
public:
    // Constructor using a member initializer list instead of assignment in the method body
	Rectangle(const double w, const double l) : width(w), length(l) {} 
    
    // Compute the area of a rectangle
    double Area(void) const {
        return length * width;	
    }
};

class Circle : public Shape {
private:
    double radius;
    
public:
    explicit Circle(double r) : radius(r) {}
	
    // Compute the area of a circle
    double Area(void) const {
        return M_PI * radius * radius;  // pi*r^2
    } 
};
```

> NOTE: As shown here, you can put multiple classes in a single header, although in practice unless you have a good reason for doing so it's probably best to use a separate header file per class.

> NOTE: I'm not using default value initialization for member variables (i.e. `double length = 0;`) and I'm using parentheses `()` instead of braces `{}` for the initializer list since older compilers (pre-C++11) may not support the new syntax.

So, we have our two classes, `Rectangle` and `Circle`, but in this case inheriting from `Shape` isn't really buying us anything. To make use of polymorphism we need to pull the common `Area()` method into the base class as follows, by using virtual methods.

#### 1.3.2 Virtual Methods
Imagine you want to have a pointer to a shape with which you want to compute the area of that shape. For example, maybe you want to hold shapes in some sort of data structure, but you don't want to limit yourself to just rectangles or just circles; you want to support all objects that call themselves a 'Shape'. Something like:

```c++
Rectangle rectangle(2.0, 5.0);
Circle circle(1.0);

// Point to the rectangle
Shape * unknown_shape = &rectangle; // Could point to *any* shape, Rectangle, Circle, Triangle, Dodecagon, etc.

unknown_shape->Area();  // Returns 10.0

// Point to the circle
unknown_shape = &circle;
unknown-shape->Area();  // Returns 3.14...
```

The way to achieve this is to use the `virtual` keyword on the base class methods, which specifies that when a pointer to a base class invokes the method of an object that it points to, it should determine, at runtime, the correct method to invoke. That is, when `unknown_shape` points to a `Rectangle` it invokes `Rectangle::Area()` and if `unknown_shape` points to a `Circle` it invokes `Circle::Area()`.

Virtual methods are employed as follows:

```c++
#include <cmath> 

class Shape {
public:
    // Virtual destructor (VERY IMPORTANT, SEE NOTE BELOW)
    virtual ~Area() {}

    // Virtual area method
    virtual double Area() const {
        return 0.0;
    }
};

class Rectangle : public Shape {
private:
    double length;
    double width;
    
public:
    Rectangle(double w, double l) : width(w), length(l) {}

    // Override the Shape::Area() method with an implementation specific to Rectangle
    double Area() const override {
        return length * width;	
    }
};

class Circle : public Shape {
private:
    double radius;
    
public:
    explicit Circle(double t) : radius(r) {}

    // Override the Shape::Area() method with an implementation specific to Circle
    //
    // NOTE: there is an 'override' keyword that was introduced in C++11 and is optional: it is used
    // to enforce that the method is indeed an overriding method of a virtual base method at compile time
    // and is used as follows:
    double Area() const override {
        return M_PI * radius * radius; // pi*r^2
    }
};
```

> NOTE: It is very important that a default virtual destructor was included after adding the virtual `Area()` method to the base class. Whenever a base class includes even a single virtual method, it must include a virtual destructor so that the correct destructor(s) are called in the correct order when the object is eventually deleted.

This is called runtime polymorphism because the decision of which implementation of the `Area()` method to use is determined during program execution based on the type that the base is pointing at. It is implemented using [the virtual table](https://www.learncpp.com/cpp-tutorial/125-the-virtual-table/) mechanism. In a nutshell: it is a little more expensive to use but it can be immensely useful. There is also compile-time polymorphism. Here is more on the [differences between them](https://www.geeksforgeeks.org/polymorphism-in-c/).

In the example above, if a class extends from `Shape` but does not include an override of `Area()` then calling the `Area()` method will invoke the base class method which (in the implementation above) returns `0.0`.

In some cases, you may want to **enforce** that sub-classes implement this method. This is done by not providing a default implementation, thus making it what is called a *pure virtual* method.

```
class Shape {
public:
    virtual ~Area() {}
    virtual double Area() const = 0;
};
```

In general a class with only pure virtual methods and a virtual destructor is called an *abstract class* or *interface* and is typically named as such (e.g. `ButtonInterface`, or similar). An interface class guarantees that all extending classes implement a specific method with a specific method signature.

### 1.4 Special methods
All this methods are used to manage class lifetime.
- **Constructor** performs creation of an object from given data. Establishes [class invariant](https://softwareengineering.stackexchange.com/a/32755).
- **Destructor** performs deletion of an object. Removes class invariant.

Briefly **class invariant** is a statement that holds true from creation to deletion of an object.
Other methods like: **copy constructor, move constructor, copy assignment, move assignment** are used for specific reasons, explained later in this item. 
#### 1.4.1 Constructor and Destructor
Meaning of this methods was explained above, here are some syntax:
##### 1.4.1.1 Use of `explicit` in Constructors
The keyword `explicit` should be used in single-argument constructors to avoid the following situation. Consider the class `Array`:
```c++
class Array {
	int size;
public:
	Array(int size) {	// constructor
		this->size = size;
	}
	
	~Array() {		// destructor
		// do some cleanup
		// Note: destructors only needed(basically)
		// to clean something that was allocated
		// from the heap with new operator
		// (new explained later)
	}
};
```

The following is now legal but ambiguous:
```c++
Array array = 12345;
```

It ends up being the equivalent of this:
```c++
Array array = Array(12345);
```

That's fine, one would suppose, but what about the following:
```c++
// Method PrintArray is defined as: Array::Print(const Array &array)
array.Print(12345);
```

Uh-oh. That's now legal, compilable code, but what does it mean? It is extremely unclear to the user.

To fix this, declare the single-argument `Array` constructor as `explicit`:
```c++
class Array {
	int size;
public:
	explicit Array(int size) {
		this->size = size;
	}
	
	~Array() { 
		//... 
	}
};
```

Now you can only use the print method as follows:
```c++
array.Print(Array(12345));
```
##### 1.4.1.2 Member initializer list
In previous example, we could've used simpler notation for initializing members:
```c++
class MemberInitializedArray
{
	int size1;
	int size2;
public:
	explicit Array(int s1, int s2)
	: size1{s1}, size2{s2}	// init members
	{
	
	}
};
```
#### 1.4.2 `new` and `delete`
Before moving on, we should consider two special "functions" in c++. 
When you declare a variable
```c++
int a = 3;
```
it uses memory, memory of your pc, your RAM. But there are two types of standard memory(in program): heap and stack.
`a` is allocated on the stack, program decides how many and where on the stack allocate this memory. 
But what if we want to decide when to allocate memory manually? There are some great flexibility features come with this decision. So here it goes:
```c++
int* a = new int {3};	// allocation
```
But with great power comes great responsibility. **We need to deallocate it manually**:
```c++
delete a;		// manual deallocation
```
More on [this](https://www.geeksforgeeks.org/new-and-delete-operators-in-cpp-for-dynamic-memory/) topic.
There are also `new[]` and `delete[]` for arrays, explained in the link.
#### 1.4.3 Copy constructor and copy assignment
Sometimes there are a need for such statements:
```c++
MyClass a {1};
MyClass b {a};	// first
MyClass c = a;	// second
```
It could be accomplished with **copy constructor**(first) and **copy assignment operator**(second).
```c++
class MyClass {
	int data;
public:
	MyClass(int i) :i{i} {}
	
	MyClass(const MyClass& mc)	// first
	: i{mc.i} {}
	
	MyClass& opeartor=(const MyClass& mc)	// second
	{
		i = mc.i;
	}
};
```
We used `operator` notation here, it will be explained soon. Now you could just see the result and how to implement it.
#### 1.4.4 Move constructor and move assignment
And sometimes we have a situation when class that we initializing from(recall **copy constructor**) won't be used in the future. All data it maintains can be moved from it to our current object.
```c++
class Movable {
	SomeClass* data_ptr;
public:
	Movable(SomeClass data)
	:data_ptr{new SomeClass {data}} {}
	
	Movable(Movable&& mvbl) {	// first
	
		data_ptr = mvbl.data_ptr;	// just assign allocated memory, no need to copy
						// mvbl will be deleted after this call
						
		mvbl.data_ptr = nullptr;	// and assign it to nullptr
						// so that it can't acces "our" data anymore
	}
	
	Movable& operator=(Movable&& mvbl) {	//second
		data_ptr = mvbl.data_ptr;
		mvbl.data_ptr = nullptr;
	}
	
	~Movable() {
		delete data_ptr;	// non-empty destructor example
	}
};
```
It is called **move constructor**(first) and **move assignment**(second).
And usecases are:
```c++
Movable some_func() {
	// ...
}

int main()
{
	Movable b {some_func()};		// first
	Movable a = some_func();		// second
}
```
It invokes because `some_func` returns object that won't be used anywhere else, it is deleted right after the call, so we use this opportunity and take it's data for ourselves.


P. S. Idiom called [`copy-and-swap`](https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom), it is better to know it, as it is proved to be very useful through the years.
### 1.5 Operator Overloading
It is pretty essential for human to use this notation for arithmetic operations:
```c++
int a = 6;
int b = 3;

int c = a + b;
int d = a - b;
int e = a * b;
int f = a / b;
```
the other version of this is:
```c++
int a = 6, b = 3;

int c = add(a, b);
int d = subtract(a, b);
int e = multiply(a, b);
int f = divide(a, b);
```
Which do you like better? First version is more intuitive as for me. The same logic stands behind `operator`s in c++. One of the mottos in c++ is "Create your classes to function as harmoniously as the built-in types". So let's create harmonious
class complex with support for basic arithmethic.
```c++
class Complex {
	double re {}; // real part, default is 0.0 
	double im {}; // imaginary part, default is 0.0
public:
	Complex(double re, double im) :re{re}, im{im} {}
	
	// just declarations
	friend Complex operator+(const Complex& fst, const Complex& snd);	// friend is a function that could reach
	friend Complex operator-(const Complex& fst, const Complex& snd);	// private members of a class
	friend Complex operator*(const Complex& fst, const Complex& snd);
	friend Complex operator/(const Complex& fst, const Complex& snd);
};

// and now definitions
Complex operator+(const Complex& fst, const Complex& snd)
{
	return {fst.re + snd.re, fst.im + snd.im};
}

Complex operator-(const Complex& fst, const Complex& snd)
{
	return {fst.re - snd.re, fst.im - snd.im};
}

Complex operator*(const Complex& fst, const Complex& snd)
{
	return {fst.re * snd.re - fst.im * snd.im, 	// just formula
		fst.re * snd.im + fst.im * snd.re};
}

Complex operator/(const Complex& fst, const Complex& snd)
{
	double denominator = snd.re * snd.re + snd.im * snd.im;		// another formula
	return {(fst.re * snd.re + fst.im * snd.im) / denominator, 
		(fst.im * snd.re - fst.re * snd.im) / denominator};		
}

// usage
int main()
{
	Complex a {1, 2};
	Complex b {5, 3};
	
	Complex c = a + b;	// 6 + 5i
	Complex d = a - b;	// -4 - 1i
	Complex e = a * b;	// -1 + 13i
	Complex f = a / b;	// 0.323529 + 0.205882i
}
```
Oh, that's good, but what about output, how could we output it?
It turns out that output is an operator too. Remember `std::cout << something` ? That's a left shift operator, but it is used to denote the output, and `std::cout` is just the global instance of class `std::ostream`. So we could just provide this operator for our own class:
```c++
#include <iostream>

class Complex {
	// ...
public:
	// ...
	
	friend std::ostream& operator<<( std::ostream& os, const Complex& cpx);
};

// implementation
std::ostream& operator<<( std::ostream& os, const Complex& cpx)
{
	os << cpx.re;
	( cpx.im < 0 ? os : os << '+' ) << cpx.im << 'i' << '\n';

	return os;
}

int main()
{
	Complex a {1, 2};
	Complex b {5, 3};
	
	std::cout << a + b;	
	std::cout << a - b;
	std::cout << a * b;
	std::cout << a / b;
}
```
There are also right shift operator denoting input, recall `std::cin >> something;`. About different kinds of operators, you can read [here](http://en.cppreference.com/w/cpp/language/operators).

### 1.6 Templates
Let's try to write `add` function from previous item.
```c++
double add(double fst, double snd)
{
	return fst + snd;
}
```
Cool, but this won't work with `int`, and many other types! Templates is a mechanism that allows generalizing of a function, that works with every type that supports `operator+`, here is the notation:
```c++
template <typename T>		// T is the name of a type
T add(const T& fst, const T& snd)	
// using const reference, because it is more efficient for large objects
{
	return fst + snd;	// if T don't support +, it will fail
}

// usage
int main()
{
	add(3, 5);		// int version
	add(3.45f, 5.0f);	// float version
	
	Complex a {1, 2};	// class from previous item
	Complex b {5, 3};
	add(a, b);	// works because of support of operator+
}
```
So this how we do generic code in c++. It saves a lot of time, try it! This approach could be used almost everywhere, for example:
```c++
template <typename T>
class Storage {
	T field;
public:
	Storage( const T& field) :field{field} {}
};

// usage
int main()
{
	Storage<int> st {3};
	Storage<Storage<int>> stst {st};
}
```
So this what happens when you use:
```c++
std::vector<int> vec;
```
There are a lot of usefulness in templates. It is one of the main technics of modern c++ development. Read more, like [this](https://www.geeksforgeeks.org/templates-cpp/) and [that](http://en.cppreference.com/w/cpp/language/templates).

Also the new feature is coming to c++20, named [concepts](https://cppdepend.com/blog/?p=524).

## 2.0 General C++ Syntax
### 2.1 Namespaces
In big projects, there are thousands of variables, and each needs it's own name. Let's imagine that there are library with linked list structure, and we need to somehow represent single node of a list:
```c++
// file named "LinkedList.h"
template <typename T>	// use of templates
struct Node { // pretty and short name
	Node* next;
	Node* prev;
	
	T data;	
};
```
And there are library with binary search tree structure:
```c++
// file named "BST.h"
template <typename T>
struct Node {
	Node* left;
	Node * right;
	
	T data;
};
```
We want to create our project that uses both of this libraries:
```c++
#include "LinkedList.h"
#include "BST.h"

int main()
{
	Node<int> * a = new Node<int> {nullptr, nullptr, 3};	// oops
};
```
How do compiler know what Node we are about to use? `namespace` comes to the rescue:
```c++
// "LinkedList.h"
namespace lst {
template <typename T>
struct Node {
	// ...
};
};
```
and
```c++
// "BST.h"
namespace bst {
template <typename T>
struct Node {
	// ...
};
};
```
and usage is:
```c++
#include "LinkedList.h"
#include "BST.h"

int main()
{
	lst::Node<int> * a = new lst::Node<int> {nullptr, nullptr, 3};	// it is linked list node
	bst::Node<int> * a = new bst::Node<int> {nullptr, nullptr, 3};	// it is bst node
};
```
So now you could specialize, and don't be afraid of name collisions, just use simple, pretty names.
By the way remember `std::cout`? It is object of `namsepaces std`, it is recommended to use full namespace names almost everywhere, because you are sure what structure you are using. But if it is your small(very small) project you could try:
```c++
#include <iostream>

using namespace std;	// everything inside namespace std becomes visible

int main()
{
	cout << "Hello, namespace!"
	
	return 0;
}
```


### 2.2 References and Pointers
are used to store the address of the varibale/object in memory. So having the pointer or reference, you could do the same operations as that of object being pointed to.
```c++
int a = 3;
int b = 5;

int* aptr = &a;		// & operator gets the address of variable a
int* bptr = &b;		// int * is a type of pointer to int

int c = *aptr + *bptr;	// * opeartor used to get the value of the object that 
			// is being pointed to
int d = a + b;		// c and d are equal at the end of the day
```
Think of pointer as the abstraction that knows where to find an object, but don't own it.
The references are about the same, but they use more convenient syntax, but some [limitations](https://stackoverflow.com/questions/57483/what-are-the-differences-between-a-pointer-variable-and-a-reference-variable-in) apply. The main difference is that refernce can't be reassigned and must be assigned at initialization:
```c++
int a = 3;
int b = 5;

int& aref = a;		// & means reference type
int& bref = b;	

int c = aref + bref;	// simple addition syntax 

int d = a + b;		// c and d are equal at the end of the day
```
Also pointers are used in arrays like that:
```c++
#include <iostream>

int arr[] = { 9, 5, 8, 2 };		// array storage is contiguous
int size = 4;

for(int i = 0; i < size; i++) std::cout << arr[i] << ' ';	// 9 5 8 2
std::cout << '\n';

int* ptr = arr;		// ptr points to first element of an array
for(int i = 0; i < size; i++) std::cout << *(ptr + i) << ' ';	// 9 5 8 2
std::cout << '\n'

ptr = arr;		// reassignment of pointer, can't be preformed with reference
for(int i = 0; i < size; i++) std::cout << *(ptr++) << ' ';	// 9 5 8 2
std::cout << '\n'
```
This loops are equivalent, moreover first-second are implemented in the same way. First loop is just convenience syntax for second. Notice, that pointers can be incremented. Also there are a special pointer, called `nullptr`.
```c++
double* ptr = nullptr;	// means it points to nowhere

double a = *ptr;	// this is a huge error, DON'T DO THIS
```

### 2.3 Keywords
[Reference](http://en.cppreference.com/w/cpp/keyword)

#### 2.3.1 General Keywords
[`asm`](http://en.cppreference.com/w/cpp/language/asm)
[`auto`](http://en.cppreference.com/w/cpp/language/auto)
[`const`](http://en.cppreference.com/w/cpp/language/cv)
[`constexpr` (*since C++11*)](http://en.cppreference.com/w/cpp/language/constexpr)
[`explicit`](http://en.cppreference.com/w/cpp/language/explicit)
[`export` (*until C++11*)](http://en.cppreference.com/w/cpp/keyword/export)
[`extern` (*language linkage*)](http://en.cppreference.com/w/cpp/language/language_linkage)
[`friend`](http://en.cppreference.com/w/cpp/language/friend)
[`inline`](http://en.cppreference.com/w/cpp/language/inline)
[`mutable`](http://en.cppreference.com/w/cpp/language/cv)
[`noexcept` (*operator*)](http://en.cppreference.com/w/cpp/language/noexcept)
[`noexcept` (*function specifier*)](http://en.cppreference.com/w/cpp/language/noexcept_spec)
[`nullptr`](http://en.cppreference.com/w/cpp/language/nullptr)
[`override`](http://en.cppreference.com/w/cpp/language/override)
[`static` (*class member specifier*)](http://en.cppreference.com/w/cpp/language/static)
[`template`](http://en.cppreference.com/w/cpp/language/templates)
[`this`](http://en.cppreference.com/w/cpp/language/this)
[`virtual` (*function specifier*)](http://en.cppreference.com/w/cpp/language/virtual)
[`virtual` (*base class specifier*)](http://en.cppreference.com/w/cpp/language/derived_class)
[`volatile`](http://en.cppreference.com/w/cpp/language/cv)

#### 2.3.2 Storage Class Specifiers
[Reference](http://en.cppreference.com/w/cpp/language/storage_duration)
* `auto` (*until C++11*)
* `register` (*until C++17*)
* `static`
* `extern`
* `thread_local` (*since C++11*)

#### 2.3.3 `const` and `dynamic` Cast Conversion
* [`const_cast`](http://en.cppreference.com/w/cpp/language/const_cast)
* [`dynamic_cast`](http://en.cppreference.com/w/cpp/language/dynamic_cast)

### 2.4 Preprocessor Tokens
* `#if`: Preprocessor version of `if(...)`
* `#elif`: Preprocessor version of `else if(...)`
* `#else`: Preprocessor version of `else`
* `#endif`: Used to end an `#if`, `#ifdef`, or `#ifndef`
* `defined()`: Returns true if the macro is defined
* `#ifdef`: Same as `#if defined(...)`
* `#ifndef`: Same as `#if !defined(...)`
* `#define`: Defines a text macro. See [here](http://en.cppreference.com/w/cpp/preprocessor/replace) for full explanation, including macro functions and predefined macros.
* `#undef`: Un-defines a text macro
* `#include`: Includes a source file
* `#line`: Changes the current file name and line number in the preprocessor
* `#error`: Prints an error message and stops compilation
* `#pragma`: Non-standard, used instead of header guards (`#ifndef HEADER_H` ...)

### 2.5 Strings (`std::string`)
[Reference](http://en.cppreference.com/w/cpp/string/basic_string)

### 2.6 Iterators (`std::iterator<...>`)
[Reference](http://en.cppreference.com/w/cpp/concept/Iterator)

### 2.7 Exceptions
#### 2.7.1 Motivation
What if you want to tell that this behaviour is unacceptable in current situation? Then you should use exceptions:
```c++
#include <iostream>

int divide(int a, int b)
{
	if( b == 0 ) throw "Can't divide by zero";	// throwing an exception
	return a / b;
}

// usage
int main()
{
	int a = 3, b = 0;
	
	try {				// try-catch close
		divide(a, b);		// we are trying to do division
	} catch( const char* err_msg) {	// if exception being thrown, we are trying to catch it
		std::cout << "Error is: " << err_msg << '\n';	// print error on the output
	}
	
	return 0;	// and finish work
}
```
#### 2.7.2 `std::exception`
[Here](http://en.cppreference.com/w/cpp/error/exception) are all standard exceptions, you may use them as follows:
```c++
#include <iostream>
#include <exception>	// for std::logic_error

class Complex { 	// class from previous examples
	double re {};
	double im {};
public:
	// ... constructor, plus/minus/multiply operators ...
	
	class ZeroDivisionError : public std::logic_error {		// class for zero division error
		const* char what() const { return "Can't divide by zero"; }
	};
	
	friend Complex operator/(const Complex& fst, const Complex& snd)
	{
		if(snd.re == 0 && snd.im == 0) throw ZeroDivisionError {};	// throwing by value
		
		// ... other code for division ...
	}
};

// usage
int main()
{
	Complex a{1, 2};
	Complex b{0, 0};
	
	try {
		a / b;
	} catch( const Complex::ZeroDivisionError& e) {	// catching by const reference
		std::cout << e.what() << '\n';
	}
	
	return 0; // and happily finish program
}
```
`what` is a standard name for a function that tells what's happened in the exception class.
### 2.8 Lambdas
Very common pattern is to write such a code:
```c++
struct AddTo3 {
	int operator()(int addto) const { return 3 + addto; }
};

int add_to_3(int addto) { return 3 + addto; }
// basically the same, but it is not enough in some cases

// usage 
int main()
{
	(AddTo3 {})(5);	// 8
	add_to_3(5);	// 8
}
```
It is so common, that language designers decided to add it to the standard and call it `lambda function`:
```c++
int main()
{
	auto add_to_3 {					// much, much simpler, just declare variable add_to_3
		[] (int addto) { return 3 + addto; }
	};
	
	add_to_3(5);
}
```
Now, what this syntax means:
```c++
int main()
{
	auto add_to_3 {
		[] // 1
		/**
		 * Capture variables, from outer scope, by reference(&),
		 * or by value(=).
		 * [&] - captures all variables by reference(uses them inside lambda)
		 * [=] - captures all variables by value(copies them inside lambda)
		 * [&var] - capture only var by reference
		 * [=var] - capture only var by value
		 * [=, &var] capture var by ref, and others by value
		 */
		(int addto) // 2
		/**
		 * Argument list, just like in ordinary functions
		 */
		{ return 3 + addto; } // 3
		/**
		* Lambda body, just like in ordinary functions
		*/
	};
	
	add_to_3(5);	// 8
}
```
So you could use it in advanced scenarios. Read more, like [this](https://stackoverflow.com/questions/7627098/what-is-a-lambda-expression-in-c11), and [that](https://en.cppreference.com/w/cpp/language/lambda).
