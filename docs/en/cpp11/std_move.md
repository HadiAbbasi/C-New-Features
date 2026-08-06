<div align="right">

[🇺🇸 English](./std_move.md) | [🇮🇷 فارسی](../../fa/cpp11/std_move.md)

</div>

---

The Topic of Move Semantics, `std::move`, and the Concepts of `lvalue` and `rvalue` in C++
Introduced in: C++11

Introduction
The concept of Move Semantics is one of the most important features of C++11. To better understand this topic, knowing the lvalue and rvalue categories is essential, and it makes understanding move semantics and rvalue references much easier.

What Do lvalue and rvalue Values Mean?
In simple terms:

lvalue values: something that has a specific location in memory and can be referred to again later.

rvalue values: a temporary value that usually exists only in the same expression.

Example:

```cpp
int x = 10;
```

Here:

The data `x` is an lvalue.

The data `10` is an rvalue.

Because:

```cpp
x = 20;
```

means `x` has a location in memory.

But:

```cpp
10 = x;
```

is invalid.

Because `10` is only a value and has no place to be stored.

Another Example

```cpp
int a = 3;
int b = a;
```

Here:

```text
a   -> lvalue
b   -> lvalue
3   -> rvalue
```

The Result of a Function
If the function is like this:

```cpp
int foo()
{
    return 5;
}
```

then

```cpp
foo()
```

is an rvalue.

Because the result of the function is a temporary value.

But:

```cpp
int x;
int& foo()
{
    return x;
}
```

Now

```cpp
foo()
```

is an lvalue.

Because it returns a reference to a real object.

How Do We Distinguish?
Suppose you have this code:

```cpp
int x = 5;
```

If we can take its address:

```cpp
&x;
```

then it is an lvalue. But

```cpp
&(x + 1);
```

is invalid. Because

```cpp
x + 1
```

is an rvalue.

Why Does This Classification Exist at All?
Because the compiler wants to know whether this object:

will be used later?

or is just a temporary value?

This information is very important for optimization.

What Problem Existed Before C++11?
Suppose:

```cpp
std::string s = "Hello";
```

And we have a function:

```cpp
void print(std::string str){}
```

If we write:

```cpp
print(s);
```

what happens is:

```text
s
 ↓
copy
 ↓
str
```

A full copy is made.

Now:

```cpp
print("Hello");
```

Again

```text
temporary string
        ↓
      copy
        ↓
      str
```

That means even the temporary object is copied. While that object is not supposed to be used anymore. This wastes memory and time.

What Does Move Semantics Say?
The idea is very simple:

If the object is temporary, we no longer need to copy from it. We can steal its resources.

Suppose:

```text
string A
+----------------------+
| Hello World          |
+----------------------+
```

Copy means:

```text
A ------copy------> B
A -> Hello World
B -> Hello World
```

We need two memories.

But Move:

```text
A ----move----> B
A -> nullptr
B -> Hello World
```

Only the pointer is transferred. No data is copied.

How Did This Become Possible?
C++11 introduced something called rvalue reference. Its sign is:

```cpp
&&
```

Example:

```cpp
void foo(std::string&& s){}
```

This function accepts only rvalues. For example:

```cpp
foo(std::string("Hello"));
```

or

```cpp
foo("Hello");
```

is valid. But

```cpp
std::string s = "Hello";
foo(s);
```

is invalid. Because `s` is an lvalue.

What Is the std::move Command?
Suppose:

```cpp
std::string s = "Hello";
```

If we write:

```cpp
std::move(s)
```

it actually does not say to move something. Rather, it only tells the compiler:

"From this moment, treat `s` like an rvalue."

Then if the class has a Move Constructor:

```cpp
std::string b = std::move(s);
```

what happens is:

```text
s
 ↓
move
 ↓
b
```

After that:

```text
b = Hello
s = valid but unspecified
```

That means `s` is still a valid object, but its value is unspecified and you should not rely on its previous contents.

Move Constructor
Suppose we have the following class:

```cpp
class String
{
public:
    String(const String& other)
    {
        // copy
    }
    String(String&& other)
    {
        // move
    }
};
```

When we write:

```cpp
String a;
String b = a;
```

the

```text
copy constructor
```

is used. But:

```cpp
String b = std::move(a);
```

the

```text
move constructor
```

is used.

Move Assignment
The same idea exists for the assignment operator as well:

```cpp
a = b;
```

↓

```text
Copy Assignment
```

But

```cpp
a = std::move(b);
```

↓

```text
Move Assignment
```

Why Is Move Faster?
Suppose we have the following class:

```cpp
class Buffer
{
    int* data;
};
```

If the buffer size is

```text
10,000,000
```

numbers, copy means:

```text
Allocate
Copy 10 million ints
```

But Move only:

```cpp
data = other.data;
other.data = nullptr;
```

It is three simple lines. Time complexity is reduced almost from O(n) to O(1).

A Real Example

```cpp
std::vector<int> v1(1000000);
std::vector<int> v2 = std::move(v1);
```

Without Move:

```text
Allocate
Copy 1,000,000 ints
```

With Move:

```text
pointer
size
capacity
```

Only these three values are moved.

The important point is that Move Semantics is only safe when the source object is no longer needed. For this reason, C++ automatically uses move operations only for rvalues (or objects that have been converted to rvalues with `std::move`); because it is assumed that these objects are no longer supposed to be used as before.

One of the most important features added to the language since C++11 is Move Semantics. This feature causes many operations that previously were performed by copying objects to be done only by transferring ownership of resources, and as a result the program runs faster and consumes less memory.

To understand Move Semantics, first we must become familiar with the two concepts lvalue and rvalue.

What Are lvalue Values?
An lvalue object is called an object that has a specific location in memory and can be accessed again later.

```cpp
std::string name = "Ali";
```

Here `name` is an lvalue.

What Is an rvalue Object?
An rvalue object is a temporary object that usually exists only in the same expression.

```cpp
std::string("Hello")
5
x + y
```

Parameter Types
Pass by Value

```cpp
void foo(std::string s);
```

For lvalues, Copy is usually performed, and for rvalues, Move is usually performed.

Reference to lvalue

```cpp
void foo(std::string& s);
```

It accepts only lvalue.

Const Reference

```cpp
void foo(const std::string& s);
```

It accepts both lvalue and rvalue.

Reference to rvalue

```cpp
void foo(std::string&& s);
```

This is the same rvalue reference and is the foundation of Move Semantics.

The `std::move` Command
The `std::move` function does not move anything; it only converts the object to an rvalue so that, if available, Move Constructor or Move Assignment is called.

```cpp
std::string a = "Hello";
std::string b = std::move(a);
```

Difference Between Copy and Move
Copy Constructor

```cpp
std::string b = a;
```

Move Constructor

```cpp
std::string b = std::move(a);
```

Copy Assignment Operator

```cpp
b = a;
```

Move Assignment Operator

```cpp
b = std::move(a);
```

Conclusion

| Concept | Description |
| ---|---|
| lvalue | An object with a specific location in memory that can be accessed multiple times. |
| rvalue | A temporary value that is usually alive only until the end of the expression. |
| && | Reference to rvalue (rvalue reference). |
| std::move | Converts the object to an rvalue; it does not move the object; it only presents it as an rvalue so that, if possible, a move operation is performed. |
| std::string s | Pass by value (Pass by Value) |
| std::string& s | Reference to lvalue |
| const std::string& s | Const reference |
| std::string&& s | Reference to rvalue |
| Copy Assignment | Assignment by copy, copying the value of one object into an existing object. |
| Move Assignment | Assignment by transferring resources, copying the value of one object into an existing object. |
| Copy Constructor | Creating a new object by copy. |
| Move Constructor | Creating a new object by transferring resources; it transfers the source object's resources to the new object and avoids expensive copying. |

---

## Resources

- https://en.cppreference.com/w/cpp/utility/move
- https://en.cppreference.com/w/cpp/language/move_constructor
- https://en.cppreference.com/w/cpp/language/value_category
- This article has been prepared and edited using explanations and rewriting done by ChatGPT (OpenAI) and Qwen(Alibaba).

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>