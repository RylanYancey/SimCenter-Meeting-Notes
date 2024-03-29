# Fun with Assembly Code and Optomization

In the HPC Trivia game we played last week, this example was shown:
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
Everybody said that the `malloc` was faster, even though it is slower since it gets heap-allocated. 

Let's take a look at this example:
```cpp
// EXAMPLE 1:
void declare_stack_array() {
    int num[6] = {1, 2, 3, 4, 5, 6};
}

// EXAMPLE 2:
void declare_heap_array() {
    int* num = (int*) malloc(sizeof(int) * 6);
    num[0] = 1;
    num[1] = 2;
    num[2] = 3;
    num[3] = 4;
    num[4] = 5;
    num[5] = 6;
}
```
Example 1 is still faster. It's faster for two reasons:
1. num is stack allocated, which is a more efficient data structure than the heap.
2. num is resolved at compile-time, which translates to less work during run-time. 

The following Assembly code was generated by [godbolt](https://godbolt.org/).

This is the assembly for Example 1: (native array, stack allocation)
```asm
declare_stack_array():
     	push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-32], 1  ; <- move 1 into array
        mov     DWORD PTR [rbp-28], 2  ; <- move 2 into array
        mov     DWORD PTR [rbp-24], 3  ; and so on
        mov     DWORD PTR [rbp-20], 4
        mov     DWORD PTR [rbp-16], 5
        mov     DWORD PTR [rbp-12], 6
        nop
        pop     rbp
        ret
```
Don't get too wrapped up in the details - focus on what's important. Each element we want to allocate, 1 - 6, is created with only a single instruction. 

A DWORD is "Double-word", or two words, or 16x2 bytes, or 32 bytes = int.

Now, let's look at the assembly code for Example 2: (malloc'd array, heap allocation)
```asm
declare_heap_array():
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     edi, 24
        call    malloc
        mov     QWORD PTR [rbp-8], rax
        mov     rax, QWORD PTR [rbp-8]
        mov     DWORD PTR [rax], 1
        mov     rax, QWORD PTR [rbp-8]
        add     rax, 4
        mov     DWORD PTR [rax], 2
        mov     rax, QWORD PTR [rbp-8]
        add     rax, 8
        mov     DWORD PTR [rax], 3
        mov     rax, QWORD PTR [rbp-8]
        add     rax, 12
        mov     DWORD PTR [rax], 4
        mov     rax, QWORD PTR [rbp-8]
        add     rax, 16
        mov     DWORD PTR [rax], 5
        mov     rax, QWORD PTR [rbp-8]
        add     rax, 20
        mov     DWORD PTR [rax], 6
        nop
        leave
        ret
```
Once again, just focus on what's important. Here's what you should notice:
1. we make a `call` to malloc to allocate the space. (remember, function calls aren't cheap.)
2. There's alot more code for every element we want to move on to the array. 

We now create a DWORD for each element, just like before, but we also have to create a QWORD PTR, or a quad-word. A heap pointer is 64 bits - a "long int" - and since we are allocating on the heap, we have to generate a heap pointer for each element. 

This means we have to do more work for each element, resulting in a slower runtime. 

# Constexpr in Assembly

Let's look at this example:
```cpp
// EXAMPLE 1:
int multiply_function(int a, int b) {
    return a * b;
}

// EXAMPLE 2:
int constexpr multiply_constexpr(int a, int b) {
    return a * b;
}
```
Constexpr vs a regular function. Constexpr is faster. Let's look at the regulalr function in assembly:
```cpp
// EXAMPLE 1:
int multiply_function(int a, int b) {
    return a * b;
}

int main() {
    int n = multiply_function(5, 5);
}
```
When we compile this example to assembly, our `main` looks like this:
```asm
multiply(int, int):
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], edi
        mov     DWORD PTR [rbp-8], esi
        mov     eax, DWORD PTR [rbp-4]
        imul    eax, DWORD PTR [rbp-8]
        pop     rbp
        ret
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     esi, 5   ; <- move 5 into the register to work with
        mov     edi, 5   ; <- move other 5 into the register to work with
        call    multiply_function(int, int) ; <- call function
        mov     DWORD PTR [rbp-4], eax      ; <- assign result of function
        mov     eax, 0
        leave
        ret
```
In this example, we have to call multiply_function. This involves having to do a `move` into the register to work with it in the function. Then, we can move the result into n. 

This is the result of compiling this using constexpr instead:
```asm
main:
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], 25  ; <- compiler evaluated the expression as 25, and assigned (1 instruction)
        mov     eax, 0
        pop     rbp
        ret
```
You can already see it's quite alot smaller. Rather than calling a function, the compiler evaluated `5 * 5 = 25`, and assigned the variable. It's another example of a `compile-time optomization`. 

Inlining is a simliar concept, except we still have to execute the function. But, we *can* remove the need to move the data into the function, since we're just copying the function contents into it's calling function. 

Lets look at the assembly for this function as inline:

GCC did not want to cooperate so I can't show anything. Keep in mind `inline` is just a suggestion, and your code may not actually be inlined. 
