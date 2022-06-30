
# Templates in C++

Templates are best thought of as _compile-time generated Typing_. 
Effectively, Templating is using _types as parameters_. 

When using Templates, we do not specify the type, but give it an alias. Here, we have called it "T" (which is standard).\
Consider this snippet:
```cpp
// Figure 1:
template<typename T>
T max(T x, T y) {
     return (x > y)? x : y;
}
```
Here, we want to compare two variables, x and y, and return a variable of the same type as x and y.\
If we wanted to use this function, we would call it this way:
```cpp
// Figure 2:
int main () {
     int a   = max<int>(5, 9);
     float b = max<float>(3.0, 10.0);
}
```
When we `make` this project, the compiler will see that we called max with an int one time, and float as another. We specified the type to use with <int> and <float>. It will create these two functions:
```cpp
// Figure 3:
int max(int x, int y) { ...

float max(float x, float y) { ...
```
You can see here that T was replaced with whatever was inside the < >. Some common examples of Templates types include `vectors` and `smart pointers`. 

## Templates vs Generics


  

  
  
  
  
  
