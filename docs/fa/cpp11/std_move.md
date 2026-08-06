<div align="right">

[🇺🇸 English](../../en/cpp11/std_move.md) | [🇮🇷 فارسی](./std_move.md)

</div>

---

# Move Semantics، `std::move` و مفاهیم `lvalue` و `rvalue` در ++C

> **نسخه معرفی:** C++11

## مقدمه

یکی از مهم‌ترین قابلیت‌هایی که از C++11 به زبان اضافه شد، **Move Semantics** است. این قابلیت باعث می‌شود بسیاری از عملیات‌هایی که قبلاً با کپی کردن اشیاء انجام می‌شدند، تنها با انتقال مالکیت منابع انجام شوند و در نتیجه برنامه هم سریع‌تر اجرا شود و هم حافظه‌ی کمتری مصرف کند.

برای درک Move Semantics ابتدا باید با دو مفهوم **lvalue** و **rvalue** آشنا شویم.

## lvalue چیست؟

lvalue به شیئی گفته می‌شود که مکان مشخصی در حافظه دارد و بعداً نیز می‌توان دوباره به آن دسترسی داشت.

```cpp
std::string name = "Ali";
```

در اینجا `name` یک **lvalue** است.

## rvalue چیست؟

rvalue یک مقدار موقتی (Temporary Object) است که معمولاً فقط در همان عبارت وجود دارد.

```cpp
std::string("Hello")
5
x + y
```

## انواع پارامترها

### ارسال به صورت مقدار

```cpp
void foo(std::string s);
```

برای lvalue معمولاً Copy و برای rvalue معمولاً Move انجام می‌شود.

### رفرنس به lvalue

```cpp
void foo(std::string& s);
```

فقط lvalue را می‌پذیرد.

### رفرنس ثابت

```cpp
void foo(const std::string& s);
```

هم lvalue و هم rvalue را می‌پذیرد.

### رفرنس به rvalue

```cpp
void foo(std::string&& s);
```

این همان rvalue reference است و پایه‌ی Move Semantics محسوب می‌شود.

## `std::move`

تابع `std::move` چیزی را جابه‌جا نمی‌کند؛ فقط شیء را به یک rvalue تبدیل می‌کند تا در صورت وجود، Move Constructor یا Move Assignment فراخوانی شود.

```cpp
std::string a = "Hello";
std::string b = std::move(a);
```

## Copy و Move

### Copy Constructor

```cpp
std::string b = a;
```

### Move Constructor

```cpp
std::string b = std::move(a);
```

### Copy Assignment Operator

```cpp
b = a;
```

### Move Assignment Operator

```cpp
b = std::move(a);
```

## جمع‌بندی

| مفهوم | توضیح |
|--------|-------|
| `std::string s` | ارسال به صورت مقدار (Pass by Value) |
| `std::string& s` | رفرنس به lvalue |
| `const std::string& s` | رفرنس ثابت |
| `std::string&& s` | رفرنس به rvalue |
| Copy Constructor | ساخت شیء جدید با کپی |
| Move Constructor | ساخت شیء جدید با انتقال منابع |
| Copy Assignment | انتساب با کپی |
| Move Assignment | انتساب با انتقال منابع |
| `std::move` | تبدیل شیء به rvalue |

---

## منابع

- https://en.cppreference.com/w/cpp/utility/move
- https://en.cppreference.com/w/cpp/language/move_constructor
- https://en.cppreference.com/w/cpp/language/value_category
- این مقاله با استفاده از توضیحات و بازنویسی انجام‌شده توسط **ChatGPT (OpenAI)** تهیه و ویرایش شده است.

