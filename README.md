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
