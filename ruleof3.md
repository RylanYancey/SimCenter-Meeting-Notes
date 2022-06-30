
# The Rule of 3

If a class requires a user-defined destructor, a user-defined copy constructor, or a user-defined copy assignment operator, it almost certainly requires all three. 

Generally, if a class contains pointers, it needs the Rule of 3. 

## User-defined Destructor

A destructor is denoted using ~, and is used to explicitly de-allocate memory. In this example, we are declaring an array of nums with `malloc` that has to be freed at the end of BinaryTree's lifetime. If we did not free nums, we would have memory which is no longer needed nor addressable, but not released, since the pointer to nums is deallocated but not the data itself. 

The destructor is called at the end of an objects' _lifetime_.

```cpp
// Figure 1
class BinaryTree {
public:
     ~BinaryTree ();
     
     int * nums = malloc (8 * sizeof(int));
};

BinaryTree::~BinaryTree () {
     free(nums);
}

int main() {
    BinaryTree btree();
} // Desetructor called at end of function where it is declared. 
```

## User-defined Copy Constructor

The Copy Constructor is for _implicit_ copies, for example copying data into or out of a function. It is important to note that `other` is the object we are copying into, and `this` is the object we are copying from. 

```cpp
// Figure 2
class BinaryTree {
public:
    BinaryTree (const BinaryTree & other);

    int * nums = malloc (8 * sizeof(int));
};

BinaryTree::BinaryTree (const BinaryTree & other) {
    other.nums = this -> nums;
}

int main() {
     BinaryTree btree();
     
     some_function(btree); // copy constructor called
}
```

## User-defined Copy Assignment Operator

The Copy Assignment Operator is for _explicit_ copies. An explicit copy occurs whenever the `=` sign is used. 

```cpp
// Figure 3
BinaryTree {
public:
    BinaryTree & operator= (const BinaryTree & other);

    int * nums = malloc (8 * sizeof(int));
};

BinaryTree::BinaryTree & operator= (const BinaryTree & other) {
    this -> nums = other.nums;
    return *this;
}

int main() {
     BinaryTree btree();
     BinaryTree btree2 = btree(); // copy assignment operator called.
}
```

# Example of the Rule of 3:

Below is a BinaryTree class. Since BinaryTrees require heap-allocated pointers to store nodes, we will have to delete the pointers at the end of BinaryTree's lifetime. This is a perfect example of the use of the Rule of 3, since the BinaryTree class has a high memory complexity. 

We also want to declare user-defined Copy Constructor / Assignment Operator, to not only comply with the rule of three, but be in control of the memory. 

```cpp
// Figure 4
struct Node {
    Node * left;
    Node * right;
    int key;
    string value;
};

class BinaryTree {
public:

    // Primary Constructor
    BinaryTree ();

    // Destructor
    ~BinaryTree ();

    // User-defined Copy Constructor
    BinaryTree (const BinaryTree & other); 

    // User-defined Copy Assignment Operator
    BinaryTree & operator=(const BinaryTree & other);

private:

    Node * root;

};
```
