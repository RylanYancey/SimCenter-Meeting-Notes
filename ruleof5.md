# The Rule of 5

"Because the presence of a user-defined, copy-constructor, or copy-assignment operator prevents implicit definition of the move constructor and the move assignment operator, any class for which move semantics are desirable, all 5 special memeber functions have to be declared.

Just like the Rule of 3, if a class contains pointers and you want to use std::move() on it, you need to Rule of 5. 

## Copy vs Reference vs Move

Copying - use of the Copy Assignment Operator or Copy Constructor copy an object, creating a second object identical to the first.

```cpp
// Figure 1
int main() {
     BinaryTree btree ();
     
     some_function (btree); // copy constructor called.
}
```

Address - The use of & to pass an object as a reference. This way, Btree exists in both `main` and `some_function`, but only as a reference in `some_function`.

```cpp
// Figure 2
int main() {
     BinaryTree btree ();
     
     some_function (&btree); // btree is passed by reference, meaning there is no copy.
}
```

Move - moving an object into a new container.

```cpp
// Figure 3
int main() {
     BinaryTree btree ();
     
     some_function (std::move(btree)); // btree is moved into fn, no longer exists in this scope. 
     
     btree.insert (5, "value"); // Segmentation Fault. btree does not exist in this scope. 
}
```

# Move Constructor & Move Assignment Operator

In a nutshell, using move allows you to swap the resources instead of copying them around. Copying is much slower than moving, so in certain cases moving can be superior to copying. 

```cpp
// Figure 4
// No copies occur here
template <class T>
swap(T& a, T& b) {
    T tmp(std::move(a));
    a = std::move(b);   
    b = std::move(tmp);
}
```

In the Rule of 5, we are trying to create Move Semantics for an object that will allow us to use std::move(object);

## Move Constructor

Within a move constructor, you can explicitly move all members to the new object using std::move.

```cpp
int main() {
     BinaryTree btree();
     some_function(std::move(btree)); // Move Constructor called. 
}
```
```cpp
// Figure 5
BinaryTree {
public:
     BinaryTree (BinaryTree && other);
     
     int * nums = malloc (8 * sizeof(int));
}

// 'this' is the object we are moving into. 'other' is the object we are moving from. 
BinaryTree::BinaryTree (BinaryTree && other) {
     this -> nums = std::move(other.nums);
}
```

## Move Assignment Operator

The Move Assignment Operator is the same as the Move Constructor, but for the operator. 

```cpp
int main() {
     BinaryTree btree();
     BinaryTree btree2 = std::move(btree); // Move Assignment Operator called.
}
```

```cpp
// Figure 6
BinaryTree {
public:
     BinaryTree & operator= (BinaryTree&& other);
     
     int * nums = malloc (8 * sizeof(int));
}

// 'This' is the object we are moving into. 'other' is the object we are moving from. 
BinaryTree::BinaryTree & operator= (BinaryTree && other) {
     this -> nums = std::move(other.nums);
     return *this;
}
```
