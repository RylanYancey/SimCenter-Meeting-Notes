# Templates in C++

Templates are best thought of as _compile-time generated Typing_.
Effectively, Templating is using _types as parameters_. 

When using Templates, we do not specify the type, but give it an alias. Here, we have called it "T" (which is standard).\
Consider this snippet:
```cpp
// Figure 1.1:
template<typename T>
T max(T x, T y) {
     // if x > y, return x, else, y 
     return (x > y)? x : y;
}
```
Here, we want to compare two variables, x and y, and return a variable of the same type as x and y.\
If we wanted to use this function, we would call it this way:
```cpp
// Figure 1.2:
int main () {
     int a   = max<int>(5, 9); // returns 9 as int
     float b = max<float>(3.0, 10.0); // returns 10.0 as float
}
```
When we `make` this project, the compiler will see that we called max with an int one time, and float as another. We specified the type to use with `<int>` and `<float>`. It will create these two functions:
```cpp
// Figure 1.3:
int max(int x, int y) { ...

float max(float x, float y) { ...
```
You can see here that T was replaced with whatever was inside the < >. Some common examples of Templates types include `vectors` and `smart pointers`. 

## Issues with Templates
  
The keen-eyed among you may have noticed a fundamental problem with using templates like this. What if `T` is not comparable? For example, if we run this code:
```cpp
// Figure 2.1
struct Apple { string type; float shine; float radius; }; 
	
int main() {
	Apple first = Apple { /*whatever*/ }; 
	Apple second = Apple { /*whatever*/ };
	
	Apple best = max<Apple>(first, second);
}
```
The compiler is not able to generate `max` for the `Apple` struct, since Apple cannot be compared with >, (or any other comparator for that matter). This is because > is not implemented for the type. 
```
// Figure 2.2
source/main.cpp: In instantiation of ‘T max(T, T) [with T = Apple]’:
source/main.cpp:28:30:   required from here
source/main.cpp:20:15: error: no match for ‘operator>’ (operand types are ‘Apple’ and ‘Apple’)
   20 |     return (x > y)? x : y;
      |            ~~~^~~~
make: *** [makefile:82: bin/main.o] Error 1
```
If you wanted to use the Apple struct here, you would have to implement operator> for Apple.
```cpp
// Figure 2.3
struct Apple {
    string type;
    float shine;
    float radius;
    friend bool operator>(const Apple& l, const Apple& r) {
        return l.shine > r.shine;
    }
};
```
Any time we want to use templates like this, it is good to specify in the documentation what needs to be implemented for the type. 

## Class Templates
You can create templates for classes too, with `template<class>`
```cpp
// Figure 3.1:
template<class T>
class BinaryTree {
public:
    BinaryTree();
	// Node is a struct
    Node<T> * root;
};
```
Note: `template<class T>` and `template<typename T>` are interchangable, except for specific situations. 
In this example, `template<class T>` is not required. `template<typename T>` works just as well. 

## Requiring Inheritance in a Class Template

It is possible to require inheritance or other traits using the `Concepts` system, but I don't fully understand that yet. 
	
## Decltype for making templates more elegent

Infers return type from an expression. For example:
```cpp
template<typename R, typename T, typename U>
R add(T x, U y) {
    return x+y;
}
```
Here, we have to use 3 generics for our function. This is verbose and can be more elegently done by inferring the return type:
```cpp
template<typename T, typename U>
auto add2(T x, U y) -> decltype(x+y){
    return x + y;
}
```
Here, we are specifying the return type of the function to be the return type of x + y. For example, if x is double and y is float, then the return type will be double. (since downcasting a double to float would result in a loss of data) This way of declaring this generic means the end-user does not have to specify the return type R, which would be redundant. 

Don't let the -> syntax confuse you. This is a specific to decltype, and informs `auto` which type it is. If you tried this:
```cpp
decltype(x+y) add(T x, U y)
```
You will get a syntax error, since x+y is not known yet. you have to use the -> syntax. 
	
## Using Templates in Header / CPP Files
Declaring Templates in header files and implementing them in CPP files is not very straightforward. To do this, you have to include the cpp file containing the implementation in the bottom of the header file. Example:
```cpp
//template.h
template<typename T> T max(T x, T y);

#include "template.cpp"
```
```cpp
//template.cpp
template<typename T> T max(T x, T y) {
	return x > y ? x : y;
}
```
Note: template.cpp should not `#include template.h`. Doing this will cause a recursive template error. 
	
## One more thing...
C++ supports _declaring variables inside template types_.
```cpp
// Figure 4.1:
template <typename T, int BufSize>
class buffer {
    T data[BufSize];
}

buffer<int, 100> buf; // 100 as template parameter
```
Here, we can use BufSize to declare our array, since BufSize is set at compile time, since template parameters are constants. 
