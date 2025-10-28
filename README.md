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

## How Strings Work in C++ (and how to use them) (31)


```cpp
char* message = "Hello"; // can be modified (not exactly, see below)
const char* fixedmessage = "World"; // constant one (cannot be modified)
```

C-style strings created with char pointers like above are should be considered as constants. In fact, to avoid confusion (`const` ones gives error at compile time when any modification happens, but non-const ones not), all c-style strings should be defined as `const char*`. [see this answer at stackoverflow](https://stackoverflow.com/a/19071536)

To create modifiable c-style strings use
```cpp
char message[] = "Hello"; // creates 6 sized char array with "Hello\0"
char another[6] = "World"; // 5th index for '\0'
```

`'a'` denotes a char and `"a"` denotes a `const char*`. \
`'\0'` (which is equal to integer 0) is the null termination character. It denotes strings end. If there is no `'\0'` at the end of a `const char*`, then functions such as `std::cout` get all bytes until they reach a `'\0'`. So, a memory violation happens. If there is no segfault, then garbage values will also be written on the screen.

C++ have `std::string` which is mostly the optimal way for handling strings. It has functions like `size()`. \
To concatenate strings, do not use
```cpp
std::string name = "Hello" + "World"; // this gives error
```
instead, use this
```cpp
std::string name = "Hello";
name += "World";
```
or this
```cpp
std::string name = std::string("Hello") + "World";
```

To check if the string contains any substring:
```cpp
std::string name = "Hello World!";
bool contains = name.find("lo") != std::string::npos;
```

To avoid memory and time overhead when copying strings, pass const references of strings to functions if possible (i.e. not changing the value of the string)
```cpp
void PrintString(const std::string& string) {
  std::cout << string;
}
```


## String Literals in C++ (32)

`"Hello"` string literal is a `const char[6]` with a `'\0'` at the end (`"Hello\0"`). \
`String literals` are always stored on the `const segment`.

```cpp
// normal string literal
const char* name = "Hello"; // or u8"Hello"

// wide character string literal
const wchar_t* name = L"Hello";

// with C++ 11
const char16_t* name2 = u"Hello"; // UTF-16
const char32_t* name3 = U"Hello"; // UTF-32
```

`R` is for raw string literals. It is useful when dealing with paragraphs etc.
```cpp
const char* example = R"Line1
Line2
Line3
Line4";
```

Other way for processing multiline string literals:
```cpp
const char* example = "Line1\n"
"Line2\n"
"Line3\n";
```

## CONST in C++ (33)

`const` is just a promise that something will remain constant.

```cpp
const int MAX_HEALTH = 100;
MAX_HEALTH = 5; // this will give a compiler error
```

```cpp
const int* a = new int; // a is a pointer to a constant integer

*a = 3; // this will give a compiler error

int number = 15;
a = &number; // this will work nicely
```

```cpp
int const* a = new int; // a is a pointer to a constant integer

*a = 3; // this will give a compiler error

int number = 15;
a = &number; // this will work nicely
```

```cpp
int* const a = new int; // a is a constant pointer to an integer

*a = 3; // this will work nicely

int number = 15;
a = &number; // this will give a compiler error
```

```cpp
int const* const a = new int; // a is a constant pointer to a constant integer

*a = 3; // this will give a compiler error

int number = 15;
a = &number; // this will give a compiler error
```

Adding `const` at the end of the method signature makes member variables non-modifiable in that method.
```cpp
class Entity {
private:
	int* m_X;
public:
	const int* GetX() const {
		*m_X = 5; // this will work nicely
		m_X = &m_X; // this will give a compiler error
		return m_X;
	}
};
```
It is useful to add `const` to the end of method signature to make it usable in situations like that:
```cpp
class Entity {
private:
	int* m_X;
public:
	const int* GetX() const { // return type is also const because we don't want the caller to change the value
		return m_X;
	}
};

void PrintEntity(const Entity& e) {
	std::cout << e.GetX();
}
```
But, if the method is defined like that:
```cpp
	const int* GetX() { // or int* GetX()
		return m_X;
	}
```
Then, the `e.GetX()` call in the PrintEntity function gives a compiler error. Because, there is no `GetX()` function with `const` keyword at the end of the signature.

There is a way to change member variables values in the const method by making them mutable:
```cpp
class Entity {
private:
	int* m_X;
	mutable bool m_X_Given;
public:
	Entity() {
		m_X = new int(5);
		m_X_Given = false;
	}
	int* getX() const {
		m_X_Given = true;
		return m_X;
	}
};
```


## The Mutable Keyword in C++ (34)

Mutable has two different uses. First use is with `const`s. Check video 33 (CONST in C++). \
The second use is with lambdas. When getting local variables with value (`[=]`), mutable makes them changeable.
```cpp
	int x = 0;
	auto f = [=]() mutable {
		x++;
		std::cout << x;
		};
	f();
```
It is copying the changed variable behind the scenes, so that the actual `x` does not get changed (still `0`):
```cpp
	int x = 0;
	auto f = [=]() {
		int y = x;
		y++;
		std::cout << y;
		};
	f();
```


## Member Initializer Lists in C++ (Constructor Initializer List) (35)

Member initializer lists are for initializing members not within a constructor but just before it. Use them if possible. The order of the member variables must be the same (if it does not, some compilers give an error, some not). Member variables initialized according to their definition order. So, if member initializer list does not comply with it, then object initialized twice (one with default constructor and one with in the member initializer list or in the constructor body) which is an overhead. (for primitives not much problem)
```cpp
class Entity3 {
private:
	std::string m_Name;
	int m_X, m_Y, m_Z;

public:
	Entity3() : m_Name("Unkown"), m_X(0), m_Y(0), m_Z(0) {}

	Entity3(const std::string& name) : m_Name(name), m_X(0), m_Y(0), m_Z(0) {}
};
```
An example for initializing twice:
```cpp
class Entity3 {
public:
	Entity3() {
		std::cout << "Default constructor of Entity3\n";
	}

	Entity3(const std::string& name) {
		std::cout << "Constructor of Entity3 with name parameter\n";
	}
};

class Entity4 {
private:
	Entity3 e;
public:
	Entity4() : e("RandomName") {}
};
```
The output is:
```
Constructor of Entity3 with name parameter
```

```cpp
class Entity4 {
private:
	Entity3 e;
public:
	Entity4() {
		e = Entity3("RandomName");
	}
};
```
The output is:
```
Default constructor of Entity3
Constructor of Entity3 with name parameter
```


## Ternary Operators in C++ (Conditional Assignment) (36)

Ternary operator is basically a syntactic sugar for if else statements.

```cpp
int level = 1;
int speed;

if (level > 5) {
	speed = 10;
} else {
	speed = 5;
}

speed = level > 5 ? 10 : 5; // does the same thing as the if else statement above
```


## How to CREATE/INSTANTIATE OBJECTS in C++ (37)

There are two ways of creating objects: on stack or on heap. \
Object does not have same meaning with Java objects in C++. [see stackoverflow answer refers to cpp standard](https://stackoverflow.com/a/43947032) \
Use stack allocated variables if possible. It is the fast and manageable way(lifetime etc.) of creating objects. \
If the object is too big or it is intended to control the objects lifetime manually, then using heap allocated variables might make sense. \
`new` keyword is used for creating objects on heap.


## The NEW Keyword in C++ (38)

Do not use `malloc` or `free`. Just use `new` and `delete`. Almost every time, `new` and `delete` are enough and makes more sense to use. `new` allocates memory and returns a pointer pointing the start of that memory, and calls constructor. `delete` calls destructor.
```cpp
Entity* e = new Entity; // calls default constructor so equivalent with Entity* e = new Entity();
Entity* e2 = new Entity("Test Name");
Entity* entity_Array = new Entity[50];

delete e;
delete e2;
delete[] entity_Array;
```

## Implicit Conversion and the Explicit Keyword in C++ (39)

C++ converts types implicitly if exactly one implicit conversion is enough. \
Explicit constructor forbids it and forces explicitly casting to the type.
```cpp
class Player {
private:
	std::string m_Name;
	int m_Age;
public:
	Player(const std::string& name) : m_Name(name), m_Age(-1) {}

	Player(int age) : m_Name("Unknown"), m_Age(age) {}
};

void PrintPlayer(const Player& player) {
	// Printing stuff
}

int main() {

	PrintPlayer(22); // gives compiler error if we put explicit on constructor with int parameter
	PrintPlayer(std::string("Cherno")); // gives compiler error if we put explicit on constructor with string parameter
	PrintPlayer(Player("Cherno")); // works anyway because the constructor used directly
	Player c = (Player)22; // works anyway because casting done explicitly

	Player a = std::string("Cherno"); // gives compiler error if we put explicit on constructor with int parameter
	Player b = 22; // gives compiler error if we put explicit on constructor with int parameter
}
```

## OPERATORS and OPERATOR OVERLOADING in C++ (40)

```cpp
struct Vector2 {
	float x, y;

	Vector2(float x, float y) : x(x), y(y) {}
	
	Vector2 Add(const Vector2& other) const {
		return Vector2(x + other.x, y + other.y);
	}

	Vector2 operator+(const Vector2& other) const {
		return Add(other);
	}

	Vector2 AddWithOperator(const Vector2& other) const { // example
		return *this + other;
	}

	Vector2 AddWithOperatorWeirdVersion(const Vector2& other) const { // example
		return operator+(other);
	}

	Vector2 Multiply(const Vector2& other) const {
		return Vector2(x * other.x, y * other.y);
	}

	Vector2 operator*(const Vector2& other) const {
		return Multiply(other);
	}

	bool IsEqual(const Vector2& other) const {
		return x == other.x && y == other.y;
	}

	bool operator==(const Vector2& other) const {
		return IsEqual(other);
	}

	bool operator!=(const Vector2& other) const {
		return !IsEqual(other);
	}

};

std::ostream& operator<<(std::ostream& stream, const Vector2& vector) {
	stream << vector.x << ", " << vector.y;
	return stream;
}

int main() {

	Vector2 position(4.0f, 4.0f);
	Vector2 speed(0.5f, 1.5f);
	Vector2 powerup(1.1f, 1.1f);

	Vector2 result1 = position.Add(speed.Multiply(powerup));
	Vector2 result2 = position + speed * powerup;

	if (result1 == result2) {
		std::cout << result1 << '\n';
	}
}
```


## The "this" keyword in C++ (41)

`this` keyword is a const pointer to the current object.


## Object Lifetime in C++ (Stack/Scope Lifetimes) (42)

Smart pointer is simply a pointer wrapper to delete heap-allocated objects when they goes out of scope. (like the stack-allocated ones)
```cpp
class ScopedPtr {
private:
	Entity* m_Ptr;
public:
	ScopedPtr(Entity* ptr) : m_Ptr(ptr) {}

	~ScopedPtr() {
		delete m_Ptr;
	}
};

int main() {
	{
		ScopedPtr e = new Entity(); // an Entity object created on heap and its pointer implicitly casted to ScopedPtr
	}
	// e destroyed because it lies on stack, and because its destructor involves deleting the internal object
	// the Entity object we created also deleted from heap
}
```
