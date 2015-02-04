Uniform Initialization and std::initializer_list
===================
[TOC]

Initialization. How it was earlier, in a "medieval" C++98 standard?
-------------

C++98 offers multiple initialization forms. We can do it (initialize) on a various style:

```cpp
// copy initialization:
double mySuperValue = 42.0;

//direct initialization:
int thisValueIsTheMostImportant(42);

//brace initialization:
struct Target 
{ 
	int x, y, z;
};
Target target = {42, 4, 2};

//function call - ctor
class Target2 
{
private:
    double x;
    double y;
public:
   Target2(double _x, double _y):x(_x),y(_y)
   {};
};
const Target2 target2(4.0, 2.0);

```

Few words about containers?
```cpp
//C-style array
int array[] = { 0,1,2,3,4,5,6,7,8,9 };

//STL containers
std::vector<int> v1;
for(int i=0; i<10; i++) 
	v1.push_back(array[i]); 

std::set<int> s;
for(int i=0; i<10; i++)
	s.insert(array[i]);

//containers with "handwork" initialization
std::vector<int> v2;
v2.push_back(88);
v2.push_back(69);
v2.push_back(33);
v2.push_back(42);

std::map<int, std::string> mm;
mm[5] = "five";
mm[10] = "ten";
mm[15] = "fifteen";

//we can print it. It is so simple and pretty ;)
for (std::map<int,std::string>::iterator it= mm.begin(); it != mm.end(); ++it)
    std::cout << it->first << " = " << it->second << std::endl;
```

Hail to the king, babe! C++11 with **uniform** initialization is comming!
-------------

Uniform initialization and initializer lists together provide a new common syntax for initialization in C++11.

```
int empty{};     				// value-initialization (to zero)
int mySuperValue {42};			// direct-list-initialization
const double thisValueIsTheMostImportant{42.0}; //as above
std::string word{'w', 'o', 'r', 'd'}; // initializer-list constructor call
int n2 = {1}; 					// copy-list-initialization

//struct Target
Target target{42, 4, 2};

struct Foo {
    std::vector<int> member;
    Foo() : member{-1, -2, -3} // list-initialization of a member in constructor
    {} 
};

//class Target2 
const Target2 target2{4.0, 2.0};  //<-- calls Target2 ctor

//containers:
int array[] 				{ 1,2,3,4,5 };
std::vector<int> v1 		{ 6,7,8,9,10 };
std::set<int> s 			{ 11,12,13,14,15 };
std::map<int,std::string> m // nested list-initialization
{ 
	{5,"five"}, 
	{10,"ten"}, 
	{15,"fifteen"} 
};

```

A few more examples. 
-------------
```cpp
using vectorInt = std::vector<int>;

int myIncredibleArray[] = {4, 2, 42};
int answerToTheUltimateQuestionOfLifeTheUniverseAndEverything{42};

vectorInt cv { myIncredibleArray[0], 20, answerToTheUltimateQuestionOfLifeTheUniverseAndEverything };

for( auto& value: cv)
        std::cout << value << std::endl;
```
Console output:
>4
20
42

Another example, with map container:
```cpp 
using mySexiMap = std::map<int,std::string>;
mySexiMap m 
{ 
	{5,"you"}, 
	{10,"like"}, 
	{15,"it!"}
};
for ( auto& value : m)
    std::cout << value.first << " " << value.second << std::endl;
```
Consol output:
>5 you
10 like
15 it!

```

using pairOfStrings = std::pair<std::string, std::string>;

auto twoStrings(pairOfStrings p) -> pairOfStrings
{
    return {p.first, p.second}; // list-initialization in return statement
}
[...]
void f(std::vector<int>& v); // func. declaration
f({ val1, val2, 10, 20, 30 }); // function argument
```

Consider one thing

    std::vector<int> myVector1 ( 10 );


 
  What is initialized? One single element that contains value 10, or a vector with size 10?
 
And now?

     std::vector<int> myVector2 { 10 };

Semantics differ for aggregates and non aggregates.
---------------------------------------------------

 - Aggregates - definition from C++11 standard:
 *“An aggregate is an array or a class with no user provided constructors, no*
*[default] initializers for non static data members, no private or protected non static data*
*members, no base classes, and no virtual functions.“*
```
struct Aggregate
{
    int x;
    double y;
};

//C++98
Aggregate object1;                  //init using dafault ctor
Aggregate object2();                //Whoops! declarated a function!
Aggregate object3 = { 42, 42.0 };
...
object1.x = 1;
object2.x = 7;    //error!
std::cout << object3.y << std::endl;
```

 - Non-aggregates
```
class NonAggregate
{
private:
    int x;
    double y;
public:
    NonAggregate(int _x, double _y):x(_x),y(_y)
    {}
    void print()
    {
        std::cout << "x = " << x << ", y = " << y << std::endl;
    }
};

//Old style with function call initialization for ctor.
NonAggregate object4(42, 42.0); 
...
object4.print();
```
Now, thanks to the uniform initialization we can use a  initializer list syntax which allows us to fully uniform type initialization that works on any objects - now there is no distinction between initialization of aggregate, or non-aggregate. 
We don't need to choose between ( ), { }, or initializing with default ctr. Now we can use braces in both cases (aggregate and non-aggregate).

```
//C++11
Aggregate object3 { 42, 42.0 }; 
NonAggregate object5 {42, 42.0}; //Taadaaam!
...
object5.print();
```

union - special case of aggregates.
----------------------------------------------
*"Uniform initialization syntax can be used with unions, but only the first member of the*
*union may be so initialized"*  - **Scott Meyers**
```
union u { int a; char* b; };
u a = { 1 }; // okay
u d = { 0, "asdf" }; // error
u e = { "asdf" }; // error (can’t initialize an int with a char array)
```
Why it works only for a first member?
We are not sure, but we think that explanation is:
*"A braced-init-list is not an expression and therefore has no type"* 
Impossible in C++98, but you can do it in C++11 using uniform initialization
-------

```
//C++98
class Foo {
public:
    Foo(): data(/*??dont know how??*/) 
    {}
private:
    int data[5]; // can't be initialized
};

const float * pData = new const float[4]; //error: uninitialized const in 'new' of 'const float'

//C++11
class Foo2 {
public:
    Foo2(): data{1, 2, 3} // list-initialization of a member in ctor.
    {}
private:
    int data[3];
};

const float * pData = new const float[4] { 1.5, 42.0, 4.2, 65.0 };
```

...but some things are forbidden.  For example C99 designated initializers :
```
struct Forbidden {
    int x, y, z;
};
Forbidden dont { .x = 5, .z = 8 }; // error!  non-trivial designated initializers not supported
```



Brace initialization and Implicit Narrowing
-------
```
//old style
int integer1 = 4.2;   //integer1 = 4

//new style
int integer2 {4.2};  //error: floating-point to integer conversion!
double real { integer1 }; // error!
int integer3 = {7.3};	// error, narrowing!
double real2 = 7;
int integer4{real2 };		// error, narrowing!
vector<int> vectorInt = { 1, 2.3, 4, 5.6 };	// error: double to int narrowing

int integer5 {static_cast<int>(4.2)};	//OK
char c{3};		// OK. Even though 3 is an int, this is not narrowing.
```


Be careful during initializing auto variables with braced initializers!
-------------------------------------------------------------------
```
auto i = { 2, 4, 6, 8 }; // i is std::initializer_list<int>

auto i1 = 10; // i1 is int
auto i2(10); // i2 is int
auto i3 {10}; // i3 is std::initializer_list<int>
...
std::vector<int> v1(i1); // v1.size() == 10, values == 0
std::vector<int> v2(i2); // v1.size() == 10, values == 0
std::vector<int> v3(i3); // v1.size() == 1, value == 10
...
template<typename T>
void f(T param) {}
f({ 2, 4, 6, 8}); // error! no type deduced for
                  // “{ 2, 4, 6, 8 }”
```

**Note:** 
*"Templates deduce no type for braced initializers. 
...but this is not true for std::initializer_list:
```
template<typename T>
void f1(std::initializer_list<T> param);
f1({2, 4, 6, 8}); // fine, T deduced as int
```


std::initializer_list
---------------------
C++11 gives us special template concept, which is called std::initializer_list. 


std::initializer_list brings possibility for constructors and other functions to take initializer-lists as parameters:


A quick look at Standard library header `<initializer_list>`
```
namespace std {
    template<class E> class initializer_list {
    public:
        typedef E value_type;
        typedef const E& reference;
        typedef const E& const_reference;
        typedef size_t size_type;
        typedef const E* iterator;
        typedef const E* const_iterator;
        initializer_list() noexcept;
        size_t size() const noexcept; // number of elements
        const E* begin() const noexcept; // first element
        const E* end() const noexcept; // one past the last element
    };
    // initializer list range access
    template<class E> const E* begin(initializer_list<E> il) noexcept;
    template<class E> const E* end(initializer_list<E> il) noexcept;

	//In C++14 standard will be added:
	template<class E> const E* rbegin(initializer_list<E> il) noexcept;
    template<class E> const E* rend(initializer_list<E> il) noexcept;
  


}
```

Very short using example:
```
#include <iostream>
#include <vector>

template <class T>
class LotsOfFun
{
public:
    LotsOfFun(std::initializer_list<T> list) : funnyVector(list)
    {
        std::cout << "Let's go!" << std::endl;
    }
    void print() 
    {
        for (const auto& value : funnyVector)
            std::cout << value << std::endl;
        std::cout << "\n";
    }
private:
    std::vector<T> funnyVector;
};

int main()
{
    LotsOfFun<std::string> teams {
            "Gornik Polkowice",
            "Tur Turek",
            "Izolator Boguchwala",
            "Milan Milanowek",
            "Niewinni Wronki"
    };

    teams.print();

    LotsOfFun<int> numbers { 4, 2, 42 };
    numbers.print();

    return 0;
}
```
Consol output:
>Gornik Polkowice
Tur Turek
Izolator Boguchwala
Milan Milanowek
Niewinni Wronki

>4
2
42

Another example -  std::initializer_list + map 

```
#include <iostream>
#include <map>
#include <set>

template<typename T1, typename T2>
class Example
{
public:
    Example(std::initializer_list<std::pair<const T1, T2>> input) : container(input) {
        std::cout << "Polish Football Premierligue:" << std::endl;
    }

    void print()
    {
        for(const auto& value : container)
            std::cout << value.first << " " << value.second << std::endl;
    }
private:
    std::map<T1, T2> container;
};

template <typename T>
void awesomeFunction(T param)
{
    std::cout << "Another example - function:" << std::endl;
    for(const auto& value : param)
        std::cout << value << std::endl;

}

int main()
{
    Example<int, std::string> letsTry {
            {1, "Gornik Polkowice"},
            {2, "Tur Turek"},
            {3, "Ina Goleniow"},
            {4, "Niewinni Wronki"}
    };
    letsTry.print();

    awesomeFunction<std::initializer_list<int>>({1, 2, 3});
    awesomeFunction<std::set<std::string>>({"Ania", "Basia", "Hermenegilda"});
    //but we can't
    //awesomeFunction({1, 2, 3});        // compiler error! "{1, 2, 3}" is not an expression,
                                    // it has no type, and so T cannot be deduced

}
```
Console output:
>Polish Football Premierligue:
1 Gornik Polkowice
2 Tur Turek
3 Ina Goleniow
4 Niewinni Wronki
Another example - function:
1
2
3
Another example - function:
Ania
Basia
Hermenegilda

std::initializer_list and overload resolution:
```
struct Foo {
    Foo(int x, int y)
    {
        std::cout << "Foo(int x, int y) called" << std::endl;
    }
    Foo(std::initializer_list<int> x) 
    {
        std::cout << "Foo(std::initializer_list) called" << std::endl;
    }
};
...
auto a = 7;
auto b = 42;

Foo foo1 { a, b }; // calls #2
Foo foo2 (b, a); // calls #1
```
Console output:
>Foo(std::initializer_list) called
Foo(int x, int y) called



Thank you,
Paweł Kowalczyk

