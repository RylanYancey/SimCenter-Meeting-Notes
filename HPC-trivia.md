# HIGH PERFORMANCE COMPUTING TRIVIA!!!!!

You will be shown two code examples.

You have to say which one is faster...

...and ***WHY!!!!!***

# EXAMPLE 1: MALLOC vs NATIVE ARRAY

```cpp
// EXAMPLE 1:
int main() {
    int array[5];
    array[3] = 1;
}

// EXAMPLE 2:
int main() {
    int* array = (int*) malloc(sizeof(int) * 5);
    array[3] = 1;
}
```
<details>
  <summary>ANSWER:</summary>

The difference here is stack vs heap allocation.

In Example 1, `array` is allocated on the stack.

In Example 2, `array` is allocated using malloc, which allocates on the heap. It must be allocated on the heap because its' size is resolved at runtime.

In this example, where we only want a small array, it is *much* better to allocate on the stack.  However, for a larger data structure, it is far more optimal to allocate on the heap, since the stack is small relative to the heap.
</details>

# EXAMPLE 2: VECTOR OPTOMIZATION
```cpp
// EXAMPLE 1:
int main() {
    std::vector<int> data();
    data.reserve(10000);

    for (int i = 0; i < 10000; i++)
        data.push_back(i);
}

// EXAMPLE 2:
int main() {
    std::vector<int> data();

    for (int i = 0; i < 10000; i++)
        data.push_back(i);
}
```
<details>
  <summary>ANSWER:</summary>

This example is about reducing the number of allocations that have to occur. 

Vectors allocate data in blocks. Take this example:

```cpp
int main() {
    // Note: repeat(n) not actually in C++... just for demonstration
  
    std::vector<int> data();
    data.push_back(5); // "data" allocates enough space for 4 ints on first push_back
    repeat(5) { data.push_back(10) } // "data" allocates another 8 slots on the 5th push_back
    repeat(16) { data.push_back(20) } // "data" allocates another 16 slots on the 13th push_back
}
```

The sequence looks like this: 0, 4, 8, 16, 32, 64, 128... etc.

When we use the `reserve` keyword, we skip all these allocations, essentially compressing them into a single allocation.

In practice, doing this is *orders of magnitude* faster. Take a look at this speed heirarchy, from fastest to slowest:

1. cpu cycle
2. read value from cache
3. access ram
4. function call
5. virtual function call
6. system call (ALLOCATION!!)
7. access disk

source: https://fekir.info/post/cpp-perf-guidelines/##avoid-optimization-barriers

As you can see, system calls are very very slow. 
  </details>
  
## EXAMPLE 3: VECTORS vs SETS
```cpp
// EXAMPLE 1:
int main() {
    // Create Vector
    std::vector<int> data { 1, 1, 1, 5, 5, 8, 8 };

    // Sort Elements
    sort(data.begin(), data.end());

    // Remove duplicates
    data.erase(unique(data.begin(), data.end()), data.end());
}

// EXAMPLE 2:
int main() {
    // Create set
    std::set<int> data { 1, 1, 1, 5, 5, 8, 8 };
}
```

<details>
  <summary> ANSWER: </summary>
This doesn't make sense, in theory, set should handle this better. But, it doesnt.

Modern Computer Architectures require that data structures be intensely optomized to work quickly. In most languages, Vectors and Hashmaps are extraordinarily well optomized, while other data structures don't get the same attention. 

In general, the only data structures that should be used are Vectors, Hashmaps, and native Arrays.

That being said, iterating over sets vs vectors is about equivalent. Additionally in some situations, other data structures ARE optimal.

Source: https://fekir.info/post/cpp-perf-guidelines/##avoid-optimization-barriers
 </details>

# EXAMPLE 4: FUNCTION vs CONSTEXPR
```cpp
// EXAMPLE 1:
int main() {
   int n = multiply(5, 5); 
}

int multiply(int a, int b) {
    return a * b;
}

// EXAMPLE 2:
int main() {
    int n = multiply(5, 5)
}

constexpr multiply(int a, int b) { 
    return a * b;
} 
```
<details>
  <summary>ANSWER:</summary>
  
  Functions are NOT free to create. They are not free because the process has to figure out where in the instruction cache the function lives. Then, parameters have to be resolved.

When we use constexpr, we are creating an expression that can be evaluated at compile time. (parameters to constexpr must be constants)

This constexpr will look like this once it is evaluated:  
```cpp
int main() {
  int n = 5 * 5;
}
```
Then, the compiler will take 5 * 5, and since 5 is a constant, evaluates `n = 10` at compile time, removing the need for runtime resolution. For this, and even more complex constexpr's, this is a massive speed-up.
 
</details>

# EXAMPLE 5: INLINING

```cpp
// EXAMPLE 1:
int main() {
   int a = 5, b = 6, c = 7;

   int n = multiply(a, b, c);
}

int inline multiply(int a, int b, int c) {
    return a * b * c;
}

// EXAMPLE 2:
int main() {
    int a = 5, b = 6, c = 7;
  
    int n = multiply(a, b, c);
}

int multiply(int a, int b, int c) {
    return a * b * c;
}
```
<details>
  <summary>ANSWER:</summary>
Creating a function as `inline` suggests to the compiler that a function should be `inlined`.  The compiler will evaluate the function and decide whether or not the function should be inlined. 

If the compiler inlines a function, it will replace any instance of it with the function itself. 
  
For example, after evaluation, the compiler may generate `main()` to look like this:
  
```cpp
int main() {
     int a = 5, b = 6, c = 7;
  
     int n = a * b * c;
}
```
  
This can be a considerable speedup, but it should only be used for small functions. Inlining alot can bloat the instruction cache, which can have major performance repercussions.
  
This is similar to constexpr, but the inputs to an inline function are not constants and are not evaluated at compile-time. 
  </details>

# EXAMPLE 6: COLUMN MAJOR vs ROW MAJOR

```cpp
// EXAMPLE 1:
int main() {
    // Create a 3x3 matrix
    int array[9] = {0, 1, 2, 3, 4, 5, 6, 7, 8};

    // width of matrix
    int width = 3;

    // Row Major iteration
    for (int x = 0; x < 3; x++) {
        for (int y = 0; y < 3; y++) {
            array[x + width * y] += 2;
        }
    }
}

// EXAMPLE 2:
int main1() {
    // Create a 3x3 matrix
    int array[9] = {0, 1, 2, 3, 4, 5, 6, 7, 8};

    // Width of matrix
    int width = 3;

    // Column Major iteration
    for (int y = 0; y < 3; y++) {
        for (int x = 0; x < 3; x++) {
            array[x + width * y] += 2;
        }
    }
}
```
<details>
  <summary>ANSWER:</summary>
Row Major is much faster than Column Major because it maximizes `cache hits`.

When we iterate by Row Major, memory is contiguous, meaning it is all right next to each other in memory. When you retrieve a variable from memory, it is moved into a register, but the data around that variable is moved into cache, which is much much faster than main memory. This means that the next iteration of the array is fast, since it has already been moved into cache.
  
When iterating by Column Major, we have to move across large strides in memory (especially for large data structures), which means we do not have the benefit of caching the neighbours of the element. 
  
Iteration order:
```cpp
// row major index order:
[ 0, 1, 2, 3, 4, 5, 6, 7, 8]

// column major index order:
[ 0, 3, 6, 1, 4, 7, 2, 5, 8]
```
  
Column Major vs Row Major is very important for Matrix Multiplication optomization.
  
  </details>
