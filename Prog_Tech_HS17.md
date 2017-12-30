# Programming Techniques for scientific simulation

- 01a: Info:
- 01b: Version Control/Git: Local file, Staging area, Repository, Push/Pull, Branching/Merging,
- 02a: Preprocessing/Compiling/Linking: Include guards,
- 02b: Make:
- 03a: CMake: compilation, libraries, configuration
- 03b: Templates & Generic Programming: function overloading,
- 04 : Classes, Operators, Function Objects: Constructors, Destructors, copy & assignment, const/mutable/friends/this, inlining member functions, use typedefs, Function objects,
- 05 : Operators & Templates, Traits,
- 06 : Templates, RNGs, Timing, Exceptions, Monte Carlo Integration
- 07 : Algorithms & Data structures in C++: Complexity/Big O, STL, Iterators, Containers & Sequences, Generic Algorithms,
- 08 : Inheritance: Derived classes, abstract base classes, virtual functions v templates, Programming Styles: (procedural, modular, obj. oriented, generic) ,
- 09 : Hardware: CPUs, Machine code & Assembly languages, CISC/RISC, Pipelining, Moore's Law, Parallelisation, SIMD, GPUs, Shared Memory, RAM, Caches,
- 10 : Optimisation & Numerical Libraries: Timing, Profiling, Data structures, Algorithms, Assembly, Compiler intrinsics, Optimisation flags, In-cache, BLAS & LAPACK
- 11a: Optimisation in C++: Timing, Profiling, Template Meta Programming
- 11b: I/O
- 12a: Python: ...
- 13a: Python II: NumPy, SciPy, Matplotlib, h5py
- 13b: C++ 11: auto, lambda expressions, nullptr, decltype, suffix return type, for(:), usw.
- 14: C++ repetition

___
## C++ compiling

### Define preprocessor macros
```c++
#define SUM(A,B) A+B
     std::cout << SUM(3,4);
//u Gets converted to std::cout << 3+4;
```
One could use macros to implement a generic functions. However, you should always avoid this.  Short version:  

* No type safety
* Clumsy for longer functions
* Unexpected side effects
* Problems with debugger and other tools  

Macros are very important in C but have far fewer uses in C++. **The first rule about macros is: don’t use them unless you have to.** Almost every macro demonstrates a flaw in the programming language, in the program, or in the programmer. Because they rearrange the program text before the compiler proper sees it, macros are also a major problem for many programming support tools. So when you use macros, you should expect inferior service from tools such as debuggers, cross- reference tools, and profilers. If you must use macros, please read the reference manual for your own implementation of the C++ preprocessor carefully and try not to be too clever. Also, to warn readers, follow the convention to name macros using lots of capital letters.

___
### Undefine macros
```c++
#define XXX “Hello”
std::cout << XXX;
#undef XXX std::cout << “XXX”;

//Gets converted to
std::cout << “Hello”;
std::cout << “XXX”;
```
___
### Running preprocessor only
```txt
c++ -E
```
___
### Conditional compilation

#### \#ifdef  
```c++
#ifdef SYMBOL
	something
#else somethingelse
#endif
```
Becomes, if SYMBOL is defined:
something  
Otherwise it becomes:
somethingelse

#### Other comlex instructions
```c++
#if !defined (__GNUC__)	std::cout << “ A non-GNU compiler”;#elif __GNUC__<=2 && _GNUC_MINOR < 95	std::cout << “gcc before 2.95”;
#elif __GNUC__==2	std::cout << “gcc after 2.95”;
#elif __GNUC__>=3	std::cout << “gcc version 3 or higher”;
#endif
```

```c++
#if !defined(__GNUC__)#error This program requires the GNU compilers
#else
```
___
### Segmenting programs
Programs can be split into several files, compiled separately and finally linked together.  
Example:

file “square.hpp”  
```c++
double square(double);```

file “square.cpp”
```c++
#include “square.hpp”
double square(double x) {	return x*x;
}```

file “main.cpp”
```c++
#include <iostream>
#include “square.hpp”
int main() {     std::cout << square(5.);}
```
___
#### Compile and link files
Compile the file square.cpp, with the -c option (no linking)  
`$ c++ -c square.cpp`
Compile the file main.cpp, with the -c option (no linking)  
`$ c++ -c main.cpp`
Link the object files  `$ c++ main.o square.o`
Link the object files and name it square.exe  
`$ c++ main.o square.o –o square.exe`
___
### Include guards
Problem:  
Consider file _grandfather.h_:  
```c++ struct foo {   int member;};
```  

and file _father.h_:  `#include “grandfather.h”;`
And finally _child.cpp_:  
`#include "grandfather.h”`
`#include "father.h”`

Q: What happens when we run child.cpp?  
`$ c++ -c child.cpp`

A: We include _grandfather.h_ twice!

Change the file _grandfather.h_ to:

```c++
#ifndef GRANDFATHER_H
#define GRANDFATHER_H
struct foo{   int member;};#endif /* GRANDFATHER_H */
```
___
### Creating (static) libraries

**ar** creates an archive, more than one object file can be specified. The name must be libsomething.a  **ranlib** adds a table of contents (not needed on some platforms)  **-I** specifes the directory where the header file is located  **-L** specifes the directory where the library is located  
**-lsomething** specifies looking in the library libsomething.a  
*Note that the order of libraries is important! If liba.a calls a function in libb.a, you need to link in the right order: -la -lb*

After a while this gets tedious. What we do is we use a build system, e.g. make, cmake, SCons, …
___
___

## CMake

### CMakeLists.txt
Example of a basic CMakeLists.txt file.

```makefile
# Require certain version
cmake_minimum_required(VERSION 2.8)

# Create a new project and give it the name "Square" project(Square)

#just compile the square program from source filesadd_executable(square.exe main.cpp square.cpp)


#install the programs into the bin subdirectoryinstall(TARGETS square.exe DESTINATION bin)
```
Additional things that can be added to the file:

```makefile
# configure a header file to pass some of the CMake settings
# to the source code
configure_file (
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
include_directories("${PROJECT_BINARY_DIR}")
```

___
### Running CMake

```makefile
# Run CMake in build directory
cd build-directory
cmake source-directory
```

```makefile
# Specify where files should be installed
cmake –DCMAKE_INSTALL_PREFIX=install-path source-directory
```
___
### Building CMake  
Build all: `make`  Build a specific target: `make square.exe`  
Cleaning the build: `make clean`  
Installing: `make install`
___

### Static libraries with CMake
```makefile
# Make a static library for the square functionadd_library(square STATIC square.cpp)
```
```makefile
# Now compile the main function and link it against the static libraryadd_executable(square.exe main.cpp) target_link_libraries(square.exe square)
```

```makefile
# Install the library, header, and programsinstall(TARGETS square.exe DESTINATION bin)
install(TARGETS square	ARCHIVE DESTINATION lib	LIBRARY DESTINATION lib	)
       install(FILES square.hpp DESTINATION include)
```
___
### Dynamic libraries with CMake

```makefile
# Make a dynamic library for the square functionadd_library(square SHARED square.cpp)
```

```makefile
# Now compile the main function and link it against the static libraryadd_executable(square.exe main.cpp) target_link_libraries(square.exe square)
```

```makefile
# Install the library, header, and programsinstall(TARGETS square.exe DESTINATION bin)
install(TARGETS square	ARCHIVE DESTINATION lib	LIBRARY DESTINATION lib	RUNTIME DESTINATION bin	)
       install(FILES square.hpp DESTINATION include)
```
### Choose between static and dynamic libraries

```makefile
# Allow the user to set an option whether to build static or dynamic  
option(BUILD_SQUARE_SHARED "Build the square library shared." ON)if(BUILD_SQUARE_SHARED)
	set(SQUARE_LIBRARY_TYPE SHARED)else(BUILD_SQUARE_SHARED)
	set(SQUARE_LIBRARY_TYPE STATIC)endif(BUILD_SQUARE_SHARED)
```

```makefile
# Make a static library for the square functionadd_library(square ${SQUARE_LIBRARY_TYPE} square.cpp)
```
___
### Configuring files
CMake variables can be inserted into files, to configure the sources before building them.  

**1)** First create an “input” file containing placeholders:

```c++
//file version.cpp.in#include "square.hpp"#define SQUARE_VERSION_MAJOR "@Square_VERSION_MAJOR@"#define SQUARE_VERSION_MINOR "@Square_VERSION_MINOR@”std::string square::version(){	return SQUARE_VERSION_MAJOR"."SQUARE_VERSION_MINOR;}
```
**2)** Set variables in the CMakeLists.txt and configure the file:  

```makefile
# CMakeLists.txt...set(Square_VERSION_MAJOR 1)
set(Square_VERSION_MINOR 0)
# create the source file, substituting variablesconfigure_file (
	"${PROJECT_SOURCE_DIR}/version.cpp.in"
	"${PROJECT_BINARY_DIR}/version.cpp")
```
___
### Additional commands
```makefile
#Perform the commands in a subdirectoryadd_subdirectory( dir )
```

```makefile
#Set additional include directoriesinclude_directories( dir )
```

```makefile
#Set directories containing librarieslink_directories ( dir )
```

```makefile#Set compiler definitions
add_definitions ( -D...)
```
___
___
## Templates and generic programming

### Function overloading
We want to have a generic function (works for a lot of different types).  
In C++ we can use the same function name multiple times:

```c++
int min(int a, int b) { return a<b ? a : b;};
float min(float a, float b) { return a<b ? a : b;};

double min(double a, double b) { return a<b ? a : b;};
```  
The compiler chooses which one to use, depending on the variable types given to the function.

```c++
min(1,3); //calls min(int, int)  
min(1.0,3.0); //calls min(double, double)
```
There is a problem, however:  

```c++
min(1,3.1415927); // Problem! which type?
min(1.0,3.1415927); // OKmin(1,int(3.1415927)); // OK but does not make sense
//one could also define new function double min(int,double);
```
The reason why it works in C++ to have several functions with the same name:  
They don't really have the same name. The function arguments get appended to the function name.
___
### Generic algorithms with templates
We can implement the example above generically by using a template:  

```c++
template <typename T>T const& min (T x, T y){  return (x < y ? x : y);}
```

What happens if we want to use a mixed call in the example above `min(1,3.141)`?  
-> Now we need to specify the first argument since it cannot be deduced.

```c++
min<double>(1,3.141);min<int>(3,4);
```

Advantages of using templates are, that we get functions that:  

* work for many types T  * are as generic and abstract as the formal definition  
* are one-to-one translations of the abstract algorithm

___
___
## Classes

### Definition of a class (Data declarators)
Classes are collections of “members” representing one entity.  

Members can be:

* functions  
* data  
* typesThese members can be split into  

* `public` accessible interface to the outside.  Should not be modified later! Only representation-independent interface, accessible to all.* `private` hidden representation of the concept. Can be changed without breaking any program using the class. Representation-dependent functions and data members.
* `friend` declarators allow related classes access to representation.

Objects of this type can be modified only through these member functions -> localization of access, easier debugging.  
**The default in a class is always private.**  
In a struct the default is public.
___
### How to design classes
What are the logical entities (nouns)?  -> classes  What are the internal state variables ?  -> private data members  How will it be created/initialized and destroyed?  -> constructor and destructor  What are its properties (adjectives)?  -> public constant member functions u how can it be   manipulated (verbs)?  -> public operators and member functions  
___
### Example of a class
```txt
We want to create a class for a trafficlight:

Property	The state of the traffic light (green, orange or red)Operation
	Set the stateConstruction	Create a light in a default state (e.g. red)
	Create a light in a given stateDestruction	Nothing special needs to be doneInternal representation	Store the state in a variable	Alternative: connect via a network to a real traffic light
```

```c++
class Trafficlight  {	public: //access declaration		enum light { green, orange, red}; //type member

 		Trafficlight(); //default constructor
		Trafficlight(light = red); //constructor
		~Trafficlight(); //destructor		
		light state() const; //function member      	void set_state(light);	private: // this is hidden		light state_; //data member};

//Usage of the above class:
Trafficlight x(Trafficlight::green);
Trafficlight::light l;l = x.state();l = Trafficlight::green;
```
Another example  

```
We want to create a class to represent a point in the euclidian 2D space:

Internal state:	x- and y- coordinates	is one possible representationConstruction
	default: (0,0)	from x- and y- values
	same as another pointProperties:	distance to another point u x- and y- coordinates
	polar coordinatesOperations
	Inverting a point
	assignment
```

```c++
class Point {	private:  		double x_,y_;	public:		
    Point(); //default constructor, (0,0)  		Point(double, double); //constructor from two numbers  		Point(const Point&); //copy constructor		~Point();	// destructor  		double dist(const Point& )  		const;  		double x() const;  		double y() const;  		double abs() const;  		double angle() const;  		void invert() ;  		Point& operator=(const Point&);};

```

Note that variables that are private contain a `_` at the end of their name.  
Remember the rule of three: If there is any one of the three following, all three have to be defined:  

* Copy Constructor (is automatically generated (a.k.a. synthesized) as memberwise copy, unless otherwise specified)
* Destructor (normally empty and automatically generated)
* Copy Assignment

Nontrivial destructor only if resources (memory) allocated by the object. This usually also requires nontrivial copy constructor and assignment operator. (example: array class)

___
### Class terminology

#### Declaration
`class Point;`

#### Definition
```c++class Point {	private:		double x_,y_;   public:		Point(); //default constructor, (0,0)
		Point(double, double);		...
};
```
#### Implementation
```c++
double Point::abs() const {     return std::sqrt(x_*x_+y_*y_)}
```
#### Constructors
```c++
//preferred method with constructor initializer list
Point::Point(double x, double y) : x_(x), y_(y) {
	...
}

/** OR **/

Point::Point(double x, double y) { x_ = x; y_ = y;}
```
___
___
## References and const members
This does **not** work:

```c++
class A {  	private:		int& x;    	const int y;  	public:    	A(int& r, int s) {			x = r; //does not work 				 //what does x refer to ?
			y = s; //does not compile
				 //y is const!
		}
};
```  
Instead, use the initialization syntax:

```c++
class A {  	private:		int& x;    	const int y;  	public:    	A(int& r, int s) : x(r), y(s) {
    		//...
    	}
};
```
Stylistic advice: initialize all members in this way.

___
### Const
```c++
//variables or data members declared as const cannot be modified
int const a = 1;
a = 2; // error!struct A {int const a=3; } obj;
obj.a=4; // error!
```
```cpp
//member functions declared as const do not modify the object
double Point::abs() const;double Point::abs() const {...}
```
```c++
//only const member functions can be called for const objects
const Point P(5,6);
P.invert(); //error!
```
___
### Static
The keyword `static` has three applications in C++, but only the two listed are used. The third use is a holdover from C and is for global variables inside a file of a code.  

#### Static inside functions
It simply means that once the variable has been initialized, it remains in memory until the end of the program. You can think of it as saying that the variable sticks around, maintaining its value, until the program completely ends. For instance, you can use a static variable to record the number of times a function has been called simply by including the following lines inside the function.

```c++
foo() {
	static int count = 0;
	count++;
}
```
Because count is a static variable, the line `static int count = 0;` will only be executed once. Whenever the function is called, count will have the last value assigned to it.

You can use `static` to prevent a variable from being reinitialized inside a loop. For instance, in the following code, `number_of_times` comes out to be 100, even though the line `static int number_of_times = 0;` is inside the inner loop, where it would apparently be executed every time the program loops. The trick is that the keyword static prevents re-initialization of the variable. One feature of using a static keyword is that it happens to be initialized to zero automatically for you -- but don't rely on this behavior (it makes your intention unclear).

```c++
for(int x=0; x<10; x++)
{
	for(int y=0; y<10; y++)
  	{
   		static int number_of_times = 0;
    	number_of_times++;
   	}
}
```

#### Static inside classes
The second use of static is inside a class definition. While most variables declared inside a class occur on an instance-by-instance basis (which is to say that for each instance of a class, the variable can have a different value), a static member variable has the same value in any instance of the class and doesn't even require an instance of the class to exist.  

Importantly, it is good syntax to refer to static member functions through the use of a class name:  
`class_name::x;` rather than `instance_of_class.x;`.  
Doing so helps to remind the programmer that static member variables do not belong to a single instance of the class and that you don't need to have a single instance of a class to use a static member variable.

```cpp
class Genome  {	public:		
    Genome(); //constructor  		static const unsigned short gene_number=64; //static data member 		
 		Genome clone() const;  		static void set_mutation_rate (unsigned short);
  			private:   		unsigned long gene_;   		static unsigned short};// in source file:mutation_rate_;unsigned short Genome::mutation_rate_=2; //definition
```
___
### Mutable
Allows modification of members even in const objects.  

```cpp
class A {   	public:   		int func() const;	private:		mutable int cnt_;
};int A::func() const {
	cnt_++; //this is okay	return 42;}
```
___
### Friends
Allowing classes to have access to each others internal representation (private parts).

```cpp
class Vector;

class Point {
		//...	private:		double x,y,z;      	friend class Vector;};class Vector {   		//...
	private:   		double x,y,z;      	friend class Point;};
```
This also works for functions:

```cpp
friend Point Point::invert(...);
friend int func(...);
```
___
### \*this
`*this` is a pointer to the object itself. It is used to access the object from a member function.

```cpp
double Point::x() const {     return this->x_; //or (*this).x_ or simply x_}
```
```cpp
//array copy assignment

const Array& Array::operator=(const Array& arr) {	//copy the array
	return * this;     }
```
___
___

## Operators and functions

### Inlining
Use to improve the speed of a program.  
Avoid excessive inlining as it leads to code-bloat.  
Note that the keyword `inline` is only a suggestion to the compiler, it is up to the compiler if he actually inlines something.


```c++
class complex {	private:
		double re_, im_;
	public:       inline double real() const;       inline double imag() const;};double complex::real() const {	return re_;
}double complex::imag() const {
	return im_;
}
```
___
### Typedefs / using
From C++11 `using` replaces the old `typedef`.  
E.g.
```c++
using element_t = double;
```

Generally **don't avoid using typedefs**. Use them to allow easy modification of your code later on (you only need to change the typedef line and there is no need for refactoring).  
Note: the `_t` is used to denote types, like `uint64_t`.

```cpp
class Animal {
	public:		typedef unsigned short age_t;       age_t age() const;	private:
		age_t age_;};
```


### Overloading class functions

```cpp
class Point {
	private:
		double x_,y_;  	public:		const Point& operator+=(const Point& rhs) {
	    	x_ += rhs.x_;
	    	y_ += rhs.y_;
	    	return *this;		}
		Point operator+(Point x, const Point& y) {
			return x+= y;		}
		std::ostream& operator <<(std::ostream& out, const Point& p) {
			out<< “( “ <<p.x()<< “, “ <<p.y()<< “ )”;
			return out;
		}


};

//we can now use print:
Point p;std::cout << “The point is “ << p << std::endl;
```
```cpp
class Potential {	double operator()(double d) { return 1.0/d; }	//Don’t get confused by the two pairs of ()()
	//The first is the name of the operator
	//The second is the argument list};

//Potential V;double distance; std::cout << V(distance);

```
#### More about operators
```cpp
A a; ++a; //these two require:
const A& A::operator++();
//or
const A& operator++(A&);A a; a++; //requireA A::operator++(int);//or
A operator++(A&,int);//The additional int argument is just to distinguish the pre- and postincrement

A b; b=a;
//uses the assignmentconst A& A::operator=(const A&);A b=a; and A b(a);
//both use the copy constructor
A::A(const A&);
```
___
### Conversion operators
conversion of A -> B as in:

```cpp
A a; B b=B(a); //can be implemented in two ways:

//constructor
B::B(const A&);
//conversion operator
A::operator B();
```
Keep in mind the implicit type conversion order:  
**bool, char < int < unsigned int < float < double**  
And also: **short < int < long < long long**  
Avoid narrowing conversion (loss of information when converting from the right to the left).
___
### Function objects
```cpp
//Assume a function with parameters:
double func(double x, double lambda) {
	return exp(-lambda*x);}
```
```cpp
class MyFunc {	private:
		const double lambda; 	public:		MyFunc(double l) : lambda(l) {}		double operator() (double x) {
			return exp(-lambda*x);
		}};
MyFunc f(3.5);
integrate(f,0.,1.,1000);
//uses object of type MyFunc like a function!
```
___
___
## Traits

### Class templates
```cpp
template <typename T>
class A {	public:		typedef T T2; //alternate name for T       A(); //constructor       //...  
		T func1(T x) {return x; } //definition		T func2(T); //declaration only	private:
		//...};

//define a member outside of the class body:
template <typename T> T A::func2(T x) {	//...}
```
```cpp
template <typename T>
class sarray {
	public:		typedef std::size_t sz_t;  //size type		typedef T elem_t;          //element type         // ... as before	private:		sz_t size_;    //size of array		elem_t* elem_; //pointer to array};
```
`std::size_t` is simply an unsigned integer type that can store the maximum size of a theoretically possible object of any type.

If we want to create a fully compatible array class, we use typedef to create our generic types:

```cpp
template <typename T>class sarray {	public:		typedef std::size_t size_type; //size type		typedef T value_type; //value / element type		typedef T& reference; //reference type		typedef T const& const_reference; // const reference type         // ...	private:
		size_type size_;   //size of array		value_type* elem_; //pointer to array};
```
Now we need to add the keyword `typename` to our operator overloads, or the compiler won't know that the member is a typ and not a variable or function.

```cpp
// subscript operatortemplate <typename T>typename sarray<T>::reference sarray<T>::operator[](size_type index) {	assert(0 <= index && index < size());	return elem_[index];}// const subscript operatortemplate <typename T>typename sarray<T>::const_reference sarray<T>::operator[](size_type index) const {	assert(0 <= index && index < size());	return elem_[index];}
```
___
### Template specialization
Take as an example an array of bools. Internally it is stored as one byte for every entry. We want to specialize this such that every bool only takes up one bit of storage.

```cpp
template <class T>
class sarray {	//generic implementation	//...
};template <>class sarray<bool> {	//optimized version for bool 	...
};
```
___
### Traits
Recall the template for the min function

```cpp
template <typename T>T const& min(T const& x, T const& y) {	return x < y ? x : y;}
```
We now want to allow things like `min(1.0,2)` and `min(1,2.0)`

Now, instead of doing it manually with 3 different typenames and having to call the function with the return type `min<int>(1.0,2)` or `min<double>(1,2.0)`, we use traits.

```cpp
template <class T, class U>typename min_type<T,U>::type& min(T const& x, T const& y)
```
The keyword typename is needed here so that C++ knows the member is a type and not a variable or function. This is required to parse the program code correctly – it would not be able to check the syntax otherwise.

How to derive min_type:

```cpp
//empty template type to trigger error messages if usedtemplate <typename T, typename U>
struct min_type {};

//partially specialized valid templates:template <class T>
struct min_type<T,T> {typedef T type; };
//fully specialized valid templates:template <>
struct min_type<double,float> {typedef double type; };

template <>
struct min_type<float,double> {typedef double type; };

template <>
struct min_type<float,int> {typedef float type; };

template <>
struct min_type<int,float> {typedef float type; };

//add more specialized templates here
```

```cpp
template <typename T>
struct average_type {	typedef typename	helper1<T,std::numeric_limits<T>::is_specialized>::type type;};
//the first helper: template<typename T, bool F>
struct helper1 {typedef T type;
};//the first helper if numeric limits is specialized for T (a partial specialization)
template<class T>struct helper1<T,true> {	typedef typename	helper2<T,std::numeric_limits<T>::is_integer>::type type;};

//the second helper
template<class T, bool F>struct helper2 {
	typedef T type;
};//the second helper if T is an integer type (a partial specialization)
template<class T>struct helper2<T,true> {
	typedef double type;
};
```
___
### Concepts
A concept is a set of requirements on types:

* The operations the types must provide* Their semantics (i.e. the meaning of these operations)
* Their time/space complexityA type that satisfies the requirements is said to model the concept. A concept can extend the requirements of another concept, whichis called refinement. The standard defines few fundamental concepts, e.g.  
* CopyConstructible
* Assignable
* EqualityComparable
* Destructible

___
## Exceptions
### Exception handling possibilites in C++  
* Global error variables (Flags)
* Functions returning error codes
* Objects that store error codes
* Exceptions (try - throw - catch)

**Global variables:** Typical for older C-code.  
Exception handling done however it pleases the programmer.  
This typically fills the code with a lot of seemingly random checks.  
**Functions retunring error code:** Every function call has to be confirmed by returning a value.  
Typical for big APIs (OS level programming).  
Caller can verify the return value of a function in order to check if the run was correct.  
**Exceptions:** Change the usual control flow.  
Exceptions work further than the normal boundaries of functions.

_How to use:_  
`throw` to signal an error which violates the PRE-condition of a function or makes the continuation of your progam impossible.  
`catch` only if it is clear if how to handle the error.  
Do **not** use throw for programming errors or violation of invariants (use `assert` instead).  
Do **not** use exceptions to change the control flow of your program!

```cpp
if(n<=0)	throw "n too small"; //throwing a char array
if(index >= size())	throw std::range_error("index"); //throwing a ...
```

Standard exceptions are in `<stdexcept>`, all derived from `std::exception`.  
Logic errors (base class std::logic_error)* domain\_error: value outside the domain of the variable
* invalid\_argument: argument is invalid
* length\_error: size too big
* out\_of\_range: argument has invalid value

Runtime errors (base class `std::runtime_error`)

* range\_error: an invalid value occurred as part of a calculation
* overflo\_error: a value got too large
* underflow\_error: a value got too smallAll take a string as argument in the constructor (the message). All have a member what which contains the message.

```cpp
int main() {	try {		std::cout << integrate(sin,0,10,1000);	}	catch (std::exception& e) {		std::cerr << "Error: " << e.what() << "\n";	}	catch(...) {
		// catch all other exceptions		std::cerr << "A fatal error occurred.\n";	}
}
```
___
___
## Timer

### Time (Timer)

The standard library contains:* Header `<ctime>`: date & time* Header `<chrono>` (since C++11): precision time measurements

_Benchmarking example:_

```c++
//requires #include <chrono>
std::chrono::time_point< std::chrono::high_resolution_clock > t_start, t_end;
t_start = std::chrono::high_resolution_clock::now();

//put code to benchmark here

t_end = std::chrono::high_resolution_clock::now();

std::cout << "It took: " << static_cast<std::chrono::duration<double> >(t_end - t_start).count()
		  << " seconds." << std::endl;
```

___
___
## Random Number Engines

### Random numbers
**Real random numbers:** are hard to obtain. It can be done from physical phenomena supposed to be random, e.g.

* Thermal noise* Atmospheric noise
* Cosmic radiation

**Pseudorandom numbers:**
* Get random numbers from an algorithm* Totally deterministic and therefore not random at all
* But maybe good enough for your application
* Never trust (just one) (pseudo) random number generator

Utilities for dealing with random numbers in standard library since C++11: Header `<random>`

Useful and good generators:

```cpp
#include <random>//Mersenne-twistersstd::mt19937 rng1;std::mt19937_64 rng2;//lagged Fibonacci generatorsstd::ranlux24_base rng3;std::ranlux48_base rng4;//linear congruential generatorsstd::minstd_rand0 rng5;
std::minstd_rand rng6;
```
```cpp
//set the seed of a RNG
#include <random>
//default random enginestd::default_random_engine e;e.seed(42); // set seed
```
#### Distributions
**Uniform distributions:**
* Integer: `std::uniform_int_distribution<int> dist1(a,b)`
* Floating point: `std::uniform_real_distribution<double> dist2(a,b)`

**Exponential distribution:**

* `std::exponential_distribution<double> dist3(lambda);`

**Normal distribution:**

* `std::normal_distribution<double> dist4(mu, sigma);`

___
___
## Data structures in C++

### Arrays:
Are consecutive range in memory. Fast arbitrary emelent access. Profits from cache effects. Constant time insertion and removal at the end. Searching in sorted array is O(ln N). Insertion and removal at arbitrary position in O(N).
* **C array**
* **vector**
* **valarray**
* **deque:**  
The deque is more complicated to implement, but yields constant time insertion and removal at the beginning.

___
### Linked lists:

* **list:**  
An linked list is a collection of objects linked by pointers into a one-dimensional sequence. Constant time insertion and removal anywhere (just reconnect the pointers). Does not profit from cache effects. Access to an arbitrary element is O(N), searching is also O(N).  
Functions:
	* `splice` joins lists without copying, moves elements from one to end of the other.
	* `sort` optimized sort, just relinks the list without copying elements
	* `merge` preserves order when “splicing” sorted lists
	* `remove(T x)`
	* `remove_if(criterion)` criterion is a function object or function, returning a bool and taking a const T& as argument.  
	Example:  		* `bool is_negative(const T& x) { return x<0; }`		* `list.remove_if(is_negative);`
___
### Trees:* **map*** **set:**  
Unordered container, entrys are unique.* **multimap:**  
Can contain more than one entry (e.g. phone number) per key.
* **multiset:**  
Unordered container, multiple entries possible
___### Queues and stacks:* **queue:**  
The queue works like a Mensa, FIFO (first in first out). In constant time you can push an element to the end of the queue, access the first and last element and remove the first element.  
Functions:
	* `void push(const T& x)` //inserts at end
	* `void pop()` //removes front
	* `T& front()`, `T& back()`, `const T& front()`, `const T& back()`
	* **priority_queue:**  
The priority_queue is like a Mensa, but professors are allowed to go to the head of the queue (not passing other professors!). The element with the highest priority is the first one to get out. For elements with equal priority, the first one in the queue is the first one out. Prioritizing is done with < operator.
* **stack:**  
The stack works like a pile of books, LIFO (last in first out). In constant time you can push an element to the top, access the top-most element and remove the top-most element.  
Functions:
	* `void push(const T& x)` //insert at top
	* `void pop()` //removes top	* `T& top()`	* `const T& top() const

___
### Generic traversal of containers
We want to traverse a vector and a list in the same way:

```cpp
for (iterator p = a.begin(); p != a.end(); ++p)	cout << *p;
```

#### Array implementation:
```cpp
template<class T>class Array {	public:		typedef T* iterator;		typedef unsigned size_type;		Array();		Array(size_type);  
  		iterator begin(){
  			return p_;
  		}  		iterator end() {
  			return p_+sz_;
  		}
  	private:  		T* p_;  		size_type sz_;};
```

#### Linked list implementation:

```cpp
template <class T>struct node_iterator {	Node<T>* p;  	node_iterator(Node<T>* q) : p(q) {}  	node_iterator<T>& operator++() {
  		p=p->next;
  		return *this;
  	}
  	T* operator ->() {
  		return &(p->value);
  	}
  	T& operator*() {
  		return p->value;
  	}
  	bool operator!=(const node_iterator<T>& x) {
  		return p!=x.p;
  	}	// more operators missing ...};

template<class T>class list {	private:
		Node<T>* first;	public:		typedef node_iterator<T> iterator;		iterator begin() {
			return iterator(first);
		}       iterator end() {
       	return iterator(0);
     	}};
```

___

___

## Useful algorithms overview

**Nonmodifying:**  

* for\_each
* find, find\_if, find\_first\_of
* adjacent\_find
* count, coun\_if
* mismatch
* equal
* search
* find\_end
* search\_n

**Modifying:**

* transform
* copy, copy\_backward
* swap, iter\_swap, swap\_ranges
* replace, replace\_if, replace\_copy, replace\_copy\_if
* fill, fill\_n
* generate, generate\_n
* remove, remove\_if, remove\_copy, remove\_copy\_if
* unique, unique\_copy
* reverse, reverse\_copy
* rotate, rotate\_copy
* random\_shuffle

**Sorted sequences:**

* sort,stable\_sort* partial\_sort, partial\_sort\_copy* nth\_element
* lower\_bound, upper\_bound
* equal\_range
* binary\_search
* merge, inplace\_merge
* partition, stable\_partition

**Permutations:**
* next\_permutation
* prev\_permutation

**Set Algorithms:**

* includes* set\_union
* set\_intersection
* set\_difference
* set\_symmetric\_difference**Minimum and Maximum:**

* min* max
* min\_element
* max\_element
* lexicographical\_compare

___
___
## Inheritance
Feature of object oriented programming.

```cpp
class PennaFishingSim : public PennaSim {
	public:		PennaFishingSim( ... whatever you had ...);    	void fish();};
```
___

### Abstract base classes
We want to have a function that can run a simulation and print some information:

```cpp
void perform(Simulation& s) {	std::cout << “Running the simulation “ << s.name() << “\n”; s.run(); //run it }
```
This class must have an name() and a run() member function
```cpp
class Simulation{
	public:		Simulation () {};		virtual std::string name() const = 0;
		virtual void run() = 0;};
```
`virtual` means that this function depends on concrete simulation, derived classes can change it.`= 0` means that in general we don’t know how to implement it. This function thus must be provided for any concrete simulation.


___

# Programming Styles

### Run-time v compile-time polymorphism
|Run-time|Compile-time|
|-|-|
|Uses **virtual functions**|    Uses **templates**
|Decision at run-time|      Decision at compile-time
|Works for objects derived from the common base|    Works for objects with the right members
|One function created for the base, class -> **saves space**| A new function is created for each class used -> **takes more space**
|Virtual function call **needs lookup in type table -> slower** | No virtual function call, **can be inlined -> faster**
|Extension possible using only definition of base class | Extension needs definitions and implementations of all functions
|Most useful for **application frameworks**, user interfaces, “big” functions| Useful for **small, low level constructs**, small fast functions and generic algorithms


### Abstract base classes, virtual function tables

### Comparison on programming Styles
The following chapter will show the different advantages and disadvantages of programming styles by means of the implementation of a stack.
### Procedural programming
Procedural programming is one of the more simple programming paradigms. It is to use in many different programming languages. *see simpson integration in Ex 1.6 for more details*
```c++

void push(double*& s, double v){
    *(s++) = v;
}
double pop(double *&s) {
    return *--s;
}
int main() {
    double stack[1000];
    double* p=stack;
    push(p,10.);
    std::cout << pop(p) << “\n”;
    std::cout << pop(p) << “\n”;
    // error of popping below
    // beginning goes undetected!
}
```


### Modular programming
The modular implementation of a stack allows transparent change in underlying data structure without
breaking the user’s program. *see simpson integration in Ex 2.2 for more details*

```c++
namespace Stack {
struct stack {
    double* s;
    double* p;
    int n;
};

void init(stack& s, int l) {
    s.s=new double[l];
    s.p=s.s;
    s.n=l;
}

void destroy(stack& s) {
    delete[] s.s;
}

void push(stack& s, double v) {
    if (s.p==s.s+s.n-1) throw
        std::runtime_error(“overflow”);
    *s.p++=v;
}

double pop(stack& s) {
    if (s.p==s.s) throw
        std::runtime_error(“underflow”);
    return *--s.p;
}
}

int main() {
Stack::stack s;
Stack::init(s,100); // must be called*
Stack::push(s,10.);
Stack::pop(s);
Stack::pop(s);      // throws error
Stack::destroy(s);  // must be called
}
```
### Object orientated programming

By implementing the stack as a class, one is able to encapsulate the data and make use of the automatic initialisation and cleanup of the class. *see simpson integration in Ex 4.2/5.3 for more details*

```c++
namespace Stack {
class stack {
    double* s;
    double* p;
    int n;
public:
    stack(int=1000); // like init
    ~stack(); // like destroy
    void push(double);
    double pop();
};

int main() {
    Stack::stack s(100);    // initialization done automatically
    s.push(10.);
    std::cout << s.pop();   // destruction done automatically
}
```

### Generic programming
By templating the class and implementing it generically you can ensure that the stack works for any data type. It also creates an efficient datatype because of the fact that the instantiation happens at compile time. *see simpson integration in Ex 8.1 for more details*

```c++
namespace Stack {
template <class T>
class stack {
    T* s;
    T* p;
    int n;
public:
    stack(int=1000); // like init
    ~stack(); // like destroy
    void push(T);
    T pop();
};

int main() {
    Stack::stack<double> s(100); // works for any type!
    s.push(1.3);
    std::cout << s.pop();
}
```

# Hardware

### CPUs
Basic components of the CPU:
- **Memory controller**: Manages loading from and storing to memory
- **Registers**: can store integers, floating point numbers, specified constants
- **Arithmetic and logical units** (ALU):
    - Perform arithmetic operations and comparisons
    - Operates on registers (very fast)
    - On some CPUs can directly operate on memory (slow)
- **Fetch and decode unit**: fetches instruction from memory, interprets then and sends them to the ALU or memory controller

##### CISC: Complex Instruction Set CPUs
CISC CPUs run on an instruction sets that implement many high level instructions (e.g sin, cos). They usually have high clock cycles but take many cycles to complete instructions. Examples: Intel IA-32 (i386), AMD x86_64.

##### RISC: Reduced Instruction Set CPUs
RISC CPUs are small simple CPUs that use low level instructs to compute. They execute instructions very quickly and can be pipelined well but need a large amount of assembly instructions to create programs. Examples are IBM Power or PowerPC.

##### Pipelining

Pipelining is the process of speeding up software by running consecutive instructions before the previous ones are finished.

<img src="graphics/pipeline.PNG" width="710"/>

##### Branch prediction
Pipelining is not possible if there are branches in the program (if-statements). CPUs circumvent this problem by  **predicting/guessing** the result of the branches and simply starting the execution of the most likely branch. If predicting is correct, then the pipeline continues as normal. Else if the prediction is wrong, the pipeline aborts and starts again at the right branch.

##### Increasing performance in CPUs
Until the early 2000s CPU performance was increased by simply increasing the number of **transistors** and the **clock speed**. Nowadays performance enhancement is reached with better **pipelining** and branch prediction. Another way that CPUs are getting faster is using caches more efficiently. I recent years almost CPUs have **multiple cores** and can run threads in **parallel**. In general CPUs are becoming **more and more complex**.  

##### SIMD: Single Instruction Multiple Data

SIMD vector operations perform the same operation on multiple values at the time. Typical SIMD operations are element multiplication or addition on vectors and matrices.

##### GPUs

GPUs are becoming increasingly important in high performance computing. They can perform massive amounts of floating point operations at very high speeds. GPUs are specifically **designed for performing SIMD instructions**. They are difficult to programs because they **contain many cores**, have inhomogeneous memory (many small caches that are spread out). E.g. the NVIDIA Tesla P100 GPU contains more than 3500 cores and can perform approximately 5*10^12 Floating point operations per second.

### Memory versus CPU speed

Although all computer components are getting faster and faster, they are not speeding up at a proportional rate. CPUs are doubling in speed every 2-2.5 years and transistor density increases every 18 months. Supercomputers are speeding up even faster by utilising massive clusters of CPUs and GPUs. One thing that hasn't increased much is RAM/Disk speed.

RAM Types (Random Access Memory):
There are many different types of varying storage size, price and speed.
- SRAM: static RAM, very fast, expensive, data stored on flip-flops
- DRAM: dynamic RAM, slower, much cheaper than SRAM, data stored on tiny capacitors
- SDRAM: synchronous dynamic RAM, synchronised to cache cycle, faster than DRAM
- DDR RAM: double data rate RAM, can send data twice per clock cycle
- Interleaved Memory Systems: uses multiple memory banks, can read from all simultaneously -> same latency, more band width

Modern computers use a combination of the different technologies. They usually have many GBs of DRAM and every decreasing amounts of every increasing fast cache memory. That means the less memory an algorithms needs, the faster it runs.🎉

### Caches

When the CPU requests a word (8 bytes), a full **cache line** (64 bytes) is loaded into the cache. The first word of the cache is then loaded into CPU. When the next word is requested then the CPU first checks if the word has already in the previous cache line. If so the data can quickly be sent to the CPU (**cache hit**). Else the cache fetches a new line from memory (**cache miss**). This means that algorithms are faster if the data it operates on is organised sequentially.

##### Types of caches
- Direct mapped: Each memory location can only be stored at **one** cache location.
- n-way associative: Each memory location can  be stored at **n** cache locations. Better performance, more expensive.
- Fully associative: Each memory location can be stored at **any** cache location.

<img src="graphics/cache.PNG" width="710"/>

##### Virtual memory

Virtual memory is the ability of an OS to temporarily map logical memory onto physical addresses. This means that the computer can store data used in the program directly on the hard disk. By using virtual memory the computer gains access to much larger amounts of very slow memory.


# Optimisation
Optimal approach to Optimisation:
1. Do not optimise
2. Use compiler flags
3. Find optimal algorithm, library
4. Find optimal data structure
5. Use profiling to determine slow parts of program
6. Optimise slow parts
7. Use parallelisation or vectorisation

### Timing & Profiling
- `time` is a simple shell command which returns the total & CPU runtime of a program. It is the simplest way to time your program
```sh
$ time program.exe
real    0m0.872s            #total time
user    0m0.813s            #cputime of the program
sys     0m0.016s            #cputime of other software whilst the program is running.
```

- **Manual instrumentation**: In c++11 the `<chrono>` header enables "manual instrumentation" or timing in a program. To make sure that your timings are accurate, make sure that your program times multiple repetitions. _See previous chapter for more information_
- **Profiling**  : `gprof` is widely used tool for linux which enables profiling of a program. To use it simply compile the program with `-pg`, run it and view the output with `gprof`. Gprof gives you information on how long different functions run for in your program.

```sh
$ g++ myProgram.cpp -pg -o program.exe
$ ./program.exe
$ gprof ./program.exe gmon.out > prof.txt
```

### Choice of data structures

When choosing data structures it is important to use **STL containers** wherever possible. Differing data structures have different strengths:
- Array: random access
- Linked lists: fast insertion in the middle
- Trees: middle ground, fastest if both features are needed

### Choice of algorithm

The way to find an efficient algorithm is to use a library. Many libraries are available at [Netlib.org](http://www.netlib.org/).

The main advantage of professional libraries are that they are:
- bug free, thoroughly tested
- optimised
- well documented (at least better documented then your code)
- support most architectures

### Assembly Calls and Compiler intrinsics (is this relevant?)


### Optimisation options

- `-DNDEBUG`: switch off all asserts and other tests
- `-O1`,...,`-O5`: -O flags are shortcuts to turn on many different compiler optimisation techniques. Levels 3 and higher increase the risk of wrong code being generated.
- `-Os`: optimises a program for size instead of speed
- `-O0 -g`: switch off all optimisations, for debugging
- `-Og:` enables optimisations that don't effect debugging (gcc >= 4.8)
- `-fopt-info`: gives you information on which optimisations have been done (gnu compilers)

### Common compiler optimisations

---

##### Subexpression elimination

<img src="graphics/Optimisation1.png" height="210" align="top">
<img src="graphics/Optimisation2.png" height="210" align="top">

The compiler can easily change simple expressions to eliminate unnecessary additions or loading of elements from arrays. The example below shows where the compiler has trouble. This is due to the **function call** in the expression. The compiler doesn't optimise the function call away because the function call may change after every call. *e.g. RNG function*

<img src="graphics/Optimisation3.png" height="230" align="top">
---
##### Strength reduction
The compile often changes expressions into ones that are easier to compute.

<img src="graphics/Optimisation4.png" height="230" align="top">
---
##### Loop invariant code motion
<img src="graphics/Optimisation5.png" height="230" align="top">

The compiler can recognise some expressions in loops that don't change and take them out. Once again the compiler cannot do this in the case of function calls.

<img src="graphics/Optimisation6.png" height="230" align="top">
---

##### Constant folding
Modern compilers create more efficient code by calculating some expressions at compile time if all necessary information is available. This feature is utilised by some template meta programs. *see next chapter*

<img src="graphics/Optimisation7.png" height="230" align="top">
---

##### Copy propagation
Modern compilers also use copy propagation to allow for better pipelining.

<img src="graphics/Optimisation8.png" height="230" align="top">
---

##### Dead code removal
The compiler can detect if a section of code is never executed and removes it from the program.

<img src="graphics/Optimisation9.png" height="230" align="top">
---

##### Variable renaming
Another way the compiler increases pipelining, is by renaming variable that are not effected by one another.

<img src="graphics/Optimisation10.png" height="220" align="top">
---

##### Induction variable simplification
<img src="graphics/Optimisation11.png" height="320" align="top">
---
<img src="graphics/Optimisation12.png" height="320" align="top">

---
##### Loop unrolling
Code for a scalar product:
```c++
double s=0.;

for (int i=0; i<3; ++i)
    s += x[i] * y[i];
```
or
```c++
double s = x[0] * y[0] + x[1] * y[1] + x[2] * y[2];
```


Loop unrolling creates faster code for two main reasons:
- No control statements in the loop
- Easier to pipeline

Partial loop unrolling is also possible:
```c++
for (int i=0; i<N; i+=4) {
 a[i] = b[i] * c[i];
 a[i+1] = b[i+1] * c[i+1];
 a[i+2] = b[i+2] * c[i+2];
 a[i+3] = b[i+3] * c[i+3];
}
```

### Storage Order and optimising cache

Multi dimensional arrays are generally stored linearly in memory. C/C++ uses **row-major** order and has stride-one access using the last index (A[ i ][ **j** ]).
Fortran, Matlab use **column-major** and have stride-one access using the first index (A[ **i** ][ j ]). The picture below shows the row major system and the red box represents one cache line.

<center>
<img src="graphics/rowMajor.png" height="320" align="top">
</center>

By aiming for stride-one access you can make your software faster because of the increase in cache hits. For **large matrices** you can use **block multiplication** to insure that the calculations can remain **in-cache**.


### Template Meta Programs

A template meta program is a program that creates your program at compile time using templates. The c++ templates can be used to optimise code by unrolling complex loops and deciding branches/ if-statements during compilation. The technique was first used effectively by Todd Veldhuisen in the blitz++ library.

The following code snippet shows a template meta program for calculating the factorial of a number at compile time. The templated class simply contains a enum used for storing the variable.

//__TODO c++14 TMP variables??__

```c++
template<int i>
class FACTOR{
  public:
      enum {RESULT = i * FACTOR<i-1>::RESULT};
};

class FACTOR<1>{
  public:
      enum {RESULT = 1};
};

     int a = FACTOR<5>::RESULT;
}
```

# LAPACK, BLAS & FORTRAN Subroutines
### FORTRAN Subroutines
Subroutines are functions written in FORTRAN which are generally used to
their effiecency.
#### In c++:
1. Write / find documentation for a subroutine suited to your need. E.g
some algorithm, operation.
2. declare function in your c++ code using the following syntax. It is
important to pass your variables as pointers.
*Note:* function names end with an underscore (except for on windows).

``` c++
extern 'C' func_ (int parameter1*, int* parameter2, double* returnvalue);
//now you can use the fortran function in your code!

double result;
int a = 2;
int b = 5;
func_(&a,&b,&result);
std::cout << result;
```

When compiling with fortran subroutines you can either link libraries for
compile your own fortran source files.

```sh
$ g++ main.cpp -lfortranlib
```

```sh
$ g++ -c -s main.cpp
$ gfortran -c -s mySubroutine.f
$ g++ main.o mySubroutine.o
```

# BLAS: Basic Linear Algebra Subprograms
BLAS is a group of Fortran Subroutines that perform LinAlg operations.
There are three different levels of BLAS:
 - Level 1: Scalar, Scalar/Vector & Vector/Vector operations
 - Level 2: Vector/Matrix operations
 - Level 3: Matrix/Matrix operations

There are also three different types of BLAS implementations:
  - Reference implementations: Work on all maschines but aren't optimised
=> slow
  - Vendor implementations: Created by computer & chip manufactures, very
fast, are proprietary and may not be free. *e.g Math Kernel Library
(Intel)*
  - Open source, (automatically) tuned implementations: fast, free to
use.
  *e.g ATLAS, OpenBLAS (recommened in the lecture).*

  # LAPACK: Linear Algebra Package
  LAPACK is a library of LinAlg Operations and Algorithms that is written
in FORTRAN and based heavily on BLAS. (if you are using an optimised
implementation of BLAS then LAPACK will also run quickly). It contains
many handy functions including *QR/SVD decompositions, Eigen value
problems & LSE Solvers*.
Note: To use LAPACK or BLAS functions add the **-lblas** or **-llapack**
flags whilst compiling.


#Input / Output
###stdin, stdout, stderr

###Pipelining & Redirection

Programs can be linked together using pipelining. `prog1.exe | prog2.exe` links the stdout of prog1.exe to the stdin of prog2.exe.

The output of a program can also be directed into a file:
`./simulation.exe > data.txt` writes the content of stdout to data.txt. You can also differentiate between stdout & stderr using the following three symbols:
 - `1>` stdout
 - `2>` stderr
 - `&>` error & output

 ###Formatting cout<<

`cout<<` transforms value into ascii text and outputs it through the stdout. `<iomanip>` & `<iostream>` contain formatting options for ostreams.
- `out.setw(x)` sets field width to x
- `out.setfill('#')` fills with #
- `out.left` `out.right.` `out.internal` sets the text alignment.
 - `out.setprecision` changes the floating point precision.

 ###File IO
```c++
// basic file operations
#include <iostream>
#include <fstream>
using namespace std;

int main () {
  ofstream myfile;
  myfile.open ("example.txt");
  myfile << "Writing this to a file.\n";
  myfile.close();
  return 0;
}
```

###HDF5
- For high volume data
- Has c/c++, python API
- Enables both serial & parallel I/O
- complicated


# Python
Python is a high-level interpreted programming language, designed to encourage simple, readable code.
- Python has  **untyped, dynamic** variables, meaning the type is not declared explicitly and the variables can change their type.
- Pyhton is an object oriented language. Almost all things in Python are **objects**. e.g. Functions, integers, usw.
- External code can be used by adding **modules** to your code. ```import myModule```
- Python has two main versions. **Python 2.x** is the legacy version which is no longer supported but contains some packages not included in **Python 3.x**. Python 3 is the current version of the language and has been around since 2008
- It contains many practical packages for scientific computing. *see next chapter*
- Python can run in a interactive shell. **IPython** is a common shell which contains syntax highlighting, tab completion and many handy functions.
- See the Python cheatsheet for more information

# Python II
### NumPy
NumPy is the fundamental package for scientific computing with Python. It is written in C and offers similar performance. NumPy provides:
- N-dimensional arrays which are very efficient. Also element operations are done element wise.
- Linear Algebra objects, FFTs & Maths functions.
- RNGs

See the [NumPy_Python_Cheat_Sheet](https://gitlab.ethz.ch/toml/RW_Studygroup/raw/2f90e798052d0976f649ea3da580a79cfeafabe5/Programming_Techniques/Numpy_Python_Cheat_Sheet.pdf) for more information.




### SciPy
SciPy is a group of algorithms and functions build on NumPy data structures. It is organised into subpackages which offer the following features:
- Integration, ODE Solvers
- Interpolation & Splines
- Linear algebra
- Linear optimisation algorithms
- Root-finding routines
- Sparse matrices
- FFTs



### Other packages:
- **Matplotlib:** is a 2D plotting library which produces high-quality plots and figures for a variety of platforms.
- **H5py:** Python interface for HDF5 file format. See *lecture/demos/week13* for examples on how to use this package.
- **Mpi4py:** Message Passing Interface. Allows parallel programming with Python.
- **Sympy:** Computer Algebra System for Python which enables symbolic mathematics.


### TODO
- mutable and immutable objects
- ``` with X as x``` context manager









# C++11
### Keyword Auto
Typ of a variable is inferred by the initialiser.  
E.g. ```c++
std::vector<double> v(5);  
auto i = v[3]; //double```  

___
### Range-based for loops
Executes a for loop over a range. Used as a more readable equivalent to the traditional for loop operating over a range of values, such as all elements in a container.

```c++
std::vector<double> v(5);
for (double x : v)
	std::cout << x; //00000
for (int x : {1, 2, 5})
	std::cout << x; //125
for (double& x : v)
	x = 5;
```
___
### Lambda Expressions
Form: `[value](int x) -> bool {return x > value; };`  
`[…]` is called caption  
`(…)` are the parameters  
`-> bool` is the return type  
`{…}` is the instruction to be done  

Capture list:  
* `[x]` Access to copied value of x  
* `[&x]` Access to reference of x  
* `[&x, y]` Access to reference of x and copied value of y  
* `[&]` Default reference access to all objects in the scope of the lambda expression  
* `[=]` Default value access to all objects in the scope of the lambda expression
___


### Keywords & c++ lingo:
- `struct/class`:
    - method: function in a class
    - member: data in a classes
    - `public`: can be accessed from outside the class
    - `private`: can only be accessed from inside the class
    - `protected`: same as private but can be accessed by inherited classes
    - destructor/constructor: initialises/ destroys objects once they go out of scope
        - `explicit`/implicit: If a constructor is defined with the `explicit` keyword, the compiler won't be able to convert the type of the input parameters (leading to compile errors if different types are used).  
    - `static`: used for variables in classes with one version per class. _see previous chapter_

- function:
    - signature: return type, parameters, constness of function
    - declaration: A declaration introduces a name into a program. For functions this includes the return type and input parameters. `void func(int);`
    - definition: Definitions actually implement the identifier created by the declaration. They can accur only once in a program.
    - operators: functions with syntactic sugar

- `const`:  
    - `const` variables: value can't be changed
    - `const` functions: assures that a function won't change input parameters
- pointers/reference:
    - saves copying
    - pointers are mutable
    - `int * const ptr;` -> pointer can't move around
    - `const int* ptr;`  -> pointer can move but the integer value must stay the same
    - references are `const`, need to be initialised
- `namespace`:
    - is a scope
    - used to organise functions and avoid naming conflicts

- cast-conversion: allows compiler to convert the type of a variable if it knows how to
    - static casting: explicit casting, always use static casting in exam!
- error handling:
    - `assert`: checks for problems, gives nicer errors, not in production code
    - Exceptions: change control flow of program. Are the recommended way of error handling.

- copy-constructor: = called on definition, e.g. `int a = 1;`
- assignment operator: = called after definition, e.g. `int a; a=2`
- encapsulation: give user access of interface but not to implementation. APIs usw.
- `template`:
    - used for generic programming
    - creates needed implementations of a function/ class at compile time
    - specialisation: specialise template from specific types

- instantiation: code generation, happens compile-time
- compiler:
    - preprocessor: handles marcos (`#include` `#DEFINE`), dumb text replacer, has difficult errors.
    - linker: links together different .o files and libraries into a finished executable
    - library: used for distribution, compilation unit, can be precompiled
    - interpreter: "compiles and runs program at the same time. e.g. python"
- type:
    - type of object
    - c++ is strongly typed (type is always defined, can't be changed) -> high performance
- polymorphism:
    - static polymorphism: Templates, compile-time, performance
    - dynamic polymorphism: Inheritance, runtime, shorter compile time
        - `virtual`: the virtual keyword is used to define functions that can be redefined in derived classes.
        - abstract function: if a virtual function is defined as = 0
- concept: precondition for classes / templates
    - syntax: how it looks
    - semantics: what it means
