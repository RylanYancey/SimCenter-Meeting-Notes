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

Let's use this gcc command to generate assembly code for these functions:
```
gcc -S main.cpp
```

This is the assembly for Example 1: (native array, stack allocation)
```asm
_Z19declare_stack_arrayv:
.LFB1880:
	.cfi_startproc         ; start of function
	endbr64
	pushq	%rbp                 
	.cfi_def_cfa_offset 16     
	.cfi_offset 6, -16         
	movq	%rsp, %rbp           
	.cfi_def_cfa_register 6    
	subq	$32, %rsp            
	movq	%fs:40, %rax         
	movq	%rax, -8(%rbp)       
	xorl	%eax, %eax
	movl	$1, -32(%rbp)     ; move "1" into the array
	movl	$2, -28(%rbp)     ; move "2"
	movl	$3, -24(%rbp)     ; move "3"
	movl	$4, -20(%rbp)     ; move "4"
	movl	$5, -16(%rbp)     ; move "5"
	movl	$6, -12(%rbp)     ; move "6"
	nop
	movq	-8(%rbp), %rax
	xorq	%fs:40, %rax
	je	.L2
	call	__stack_chk_fail@PLT
.L2:
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc             ; end of function
```
Don't get too wrapped up in the details - focus on what's important. Theres a few things to notice here:

1. Each element only takes 1 instruction to move into the array. "movl". (ignore everything else, it's not that important)
2. That's it. Thought I had more.

Now, let's look at the assembly code for Example 2: (malloc'd array, heap allocation)
```asm
_Z18declare_heap_arrayv:
.LFB1881:
	.cfi_startproc            ; begin function
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	subq	$16, %rsp
	movl	$24, %edi
	call	malloc@PLT
	movq	%rax, -8(%rbp)   
	movq	-8(%rbp), %rax
	movl	$1, (%rax)          ; move "1" onto array
	movq	-8(%rbp), %rax
	addq	$4, %rax            
	movl	$2, (%rax)          ; move "2" onto array
	movq	-8(%rbp), %rax
	addq	$8, %rax            
	movl	$3, (%rax)          ; move "3"
	movq	-8(%rbp), %rax
	addq	$12, %rax
	movl	$4, (%rax)          ; move "4"
	movq	-8(%rbp), %rax
	addq	$16, %rax
	movl	$5, (%rax)          ; move "5"
	movq	-8(%rbp), %rax
	addq	$20, %rax
	movl	$6, (%rax)          ; move "6"
	nop
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc          ; end function
```
Once again, just focus on what's important. Here's what you should notice:
1. we make a `call` to malloc to allocate the space. (remember, function calls aren't cheap.)
2. There's alot more code for every element we want to move on to the array. 

This example is so much slower because we have to call malloc rather than just generating assembly code directly. Malloc is not a cheap function to call. Because the data is heap allocated, we have to do more work to figure out where it should be. 

Like we've said, Heap Allocation is so much slower because we have to figure out allocation at `runtime`, not `compile-time`. Allocating on the stack in this example is a `compile-time optomization`. 
