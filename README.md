# C++ Notes


## Visibility in C++ (29)

C++ has three different visibility modifiers: \
public: That member or method visible to everyone. \
protected: That member or method can only visible to class and its subclasses. \
private: That member or method can only visible to class itself.

There is a *friend* keyword which provides access to private or protected members or methods.

Class members are private if not any access modifier stated. \
Struct members are private if not any access modifier stated.

```cpp
class Entity {
  int X, Y;
}
```
is identical to:
```cpp
class Entity {
private:
  int X, Y;
}
```

The vital part of encapsulation (hiding members or methods of the class) is that it makes developers to think private members or methods should only be accesible from the class itself. \
So, for example we have a GUI application and private X, Y positions of some element. That element also have public move function. \
The developer should not try to change the value of X, Y manually and should move the element through move function which is public.


## Arrays in C++ (30)

In debug mode, accessing an element that is outside of the array (for instance 5th element of 5 sized array, in C/C++ arrays are zero-indexed) that memory access violation results with an error.

In pointer arithmetic, offsets are calculated using pointer type's element size. For example, with using integer pointer `+2` as `+8` bytes offset. (2 times 4 bytes which is size of an integer)
```cpp
int example[5];
example[2] = 6;
*(example + 2) = 6;
*(int*)((char*)example + 8) = 6;
```

Arrays can also be created on heap.
```cpp
int* another = new int[5];
delete[] another; // for deleting from the heap
```

To avoid memory fragmentation which reduces performance, (it is bad for cache, also jumping around memory needs time) arrays should be created on stack if possible. (like creating an array and passing its reference to the function)

With C++ 11, `std::array` can be used. It has some advantages like: \
getting array size as `myArray.size()` \
does bounds checking etc. (it has its overhead for sure)

With stack allocated arrays, there is a way of getting the size of the array:
```cpp
int array[5];
int count = sizeof(array) / sizeof(int);
```
But, this should not be trusted because arrays usually passed to functions via pointer and it causes the same problem as with the heap-allocated arrays: `sizeof(ptr)` gives pointers size not the arrays actual size.

```cpp
void print(int array[4]);
```
is identical to
```cpp
void print(int* array);
```

To force size-compatibility check, the array needs to be passed like this:
```cpp
void print(int (*array)[4]);
```
or this
```cpp
void print(int (&array)[4]);
```
