<div align="right">

[🇺🇸 English](./std_move.md) | [🇮🇷 فارسی](../../fa/cpp11/std_move.md)

</div>

---

# Move Semantics, std::move, and the Concepts of lvalue and rvalue in C++

**Introduced in: C++11

## Introduction

Move Semantics is one of the most important features introduced in C++11. It allows the ownership of an object's resources to be transferred instead of performing an expensive copy operation whenever possible. As a result, programs can achieve better performance and lower memory usage.
To fully understand Move Semantics, it is essential to first become familiar with concepts such as lvalue, rvalue, and rvalue references.
---

## What Are lvalue and rvalue?

Simply put:

- **`lvalues`:** Expressions that refer to an object with an identifiable location in memory. They have a persistent identity and can generally be accessed again later.
- **`rvalues`:** Temporary values or objects that typically exist only within a single expression and have a short lifetime.

Example:

```cpp
int x = 10;
```

in this example:

- variable `x` is a `lvalue`
- data `10` is a `rvalue`

for example:

```cpp
x = 20; // correct
```

Here, x is an lvalue because it is an object with a well-defined location in memory.

```cpp
10 = x; // wrong
```

On the other hand, 10 is an rvalue because it is just a literal value and does not have an assignable storage location.

another example:

```cpp
int a = 3;
int b = a;
```

in this part:

- variable `a` is a `lvalue`!
- variable `b` is a `lvalue`!
- data `3` is a `rvalue`!

---

## result of function

If a function returns its result by value, the returned expression is typically an `rvalue`:

```cpp
int foo()
{
    return 5;
}
```

in this case:

```cpp
foo();
```

is An `rvalue` because the function returns a temporary value.
However, if a function returns a reference, the result can be an `lvalue`:

```cpp
int x = 0;

int& foo()
{
    return x;
}
```

in this case:

```cpp
foo();
```

is An `lvalue` because it refers to a real object.

---

## A Simple Way to recognize the Difference

for example:

```cpp
int x = 5;
```

If you can take the address of an expression, it is usually an `lvalue`:

```cpp
&x; //is correct
```

but:

```cpp
&(x + 1); // wrong
```

because `x + 1` is a temp value!

> Note: Modern C++ has a more precise value category system consisting of `lvalue`, `prvalue`, and `xvalue`.
> For an introductory understanding of Move Semantics, the simple distinction between `lvalue` and `rvalue` is sufficient. However, keep in mind that `std::move(x)` specifically produces an `xvalue`.

---

## Why Does This Distinction Exist?

compilers need to know:

- Will this object be used again later?
- Or is it just a temporary value that can be safely discarded?

This information is essential for optimization.

---

## what was wrong before C++11?

suppose that:

```cpp
std::string s = "Hello";
```

And suppose we have a function that takes its parameter by value:

```cpp
void print(std::string str) {}
```

if we call:

```cpp
print(s);
```

یک کپی از `s` برای ساختن `str` ایجاد می‌شود.

به صورت مفهومی:

```text
s
 ↓ copy
str
```

now if we send a temp value:

```cpp
print("Hello");
```

باز هم ممکن بود یک شیء موقتی ساخته و سپس کپی شود، در حالی که آن شیء موقتی دیگر قرار نیست بعداً استفاده شود. این باعث اتلاف حافظه و زمان می‌شود.

> در C++11 و بعد، با استفاده از move semantics و همچنین copy elision، بسیاری از این کپی‌ها حذف یا کاهش پیدا می‌کنند.

---

## what is the idea of Move Semantics?

the idea is easy yo know:

> اگر شیء موقتی است یا دیگر نیازی به آن نداریم، به‌جای کپی کردن منابعش، منابع آن را منتقل کن.

مثال مفهومی برای کلاسی که حافظه پویا دارد:

```text
Copy:

A ------copy------> B

A -> data
B -> copy of data
```

this means we need 2 different memories!
but Move:

```text
Move:

A ----move----> B

B -> data
A -> resource released / empty / safe state
```

در move، معمولاً فقط اشاره‌گر یا مالکیت منبع منتقل می‌شود و داده اصلی کپی نمی‌شود.

> نکته: بعد از move، شیء مبدأ لزوماً `nullptr` یا خالی نیست. استاندارد برای بسیاری از کلاس‌های کتابخانه‌ای می‌گوید شیء مبدأ باید «معتبر ولی نامشخص» باشد.

---

## Rvalue Reference

there is a new reference type in C++11

```cpp
&&
```

we call this a **rvalue reference**

example:

```cpp
void foo(std::string&& s) {}
```

این تابع می‌تواند rvalueها را بپذیرد.

for example:

```cpp
foo(std::string("Hello")); // correct
```

or:

```cpp
foo("Hello"); // correct
```

در حالت دوم، یک `std::string` موقتی از `"Hello"` ساخته می‌شود و به `foo` پاس داده می‌شود.

but:

```cpp
std::string s = "Hello";
foo(s); // wrong
```

چون `s` یک `lvalue` است و نمی‌توان آن را مستقیم به `std::string&&` متصل کرد.

برای این کار می‌توان نوشت:

```cpp
foo(std::move(s)); // correct
```

---

## نکته مهم درباره پارامتر `std::string&&`

داخل بدنه تابع، خود پارامتر `s` یک `lvalue` است، چون نام دارد و آدرس آن قابل اشاره است.

example:

```cpp
void foo(std::string&& s)
{
    //متغیر s در اینجا یک lvalue است،
    // با وجود اینکه نوع آن std::string&& است.

    std::string local = std::move(s); // درخواست move از s
}
```

یعنی اگر بخواهید از یک rvalue reference داخل تابع move کنید، معمولاً باید دوباره از `std::move` استفاده کنید.

---

## what is the `std::move`?

suppose that:

```cpp
std::string a = "Hello";
```

if we have:

```cpp
std::move(a)
```

این دستور به‌تنهایی چیزی را جابه‌جا نمی‌کند.

`std::move` در واقع یک **cast** است که می‌گوید:

> با این عبارت مثل یک rvalue رفتار کن.

به بیان دقیق‌تر، `std::move` یک rvalue reference از شیء برمی‌گرداند:

```cpp
std::string b = std::move(a);
```

در این حالت، اگر move constructor با ورودی `std::string` داشته باشید، ممکن است عملیات move انجام شود.

بعد از این عملیات:

```text
b -> resource moved from a
a -> valid but unspecified
```

یعنی `a` هنوز یک شیء معتبر است، اما مقدار دقیق آن مشخص نیست و نباید روی محتوای قبلی آن حساب کرد.

> نکته مهم: `std::move` فقط «امکان» move را فراهم می‌کند. اگر move constructor یا move assignment مناسبی وجود نداشته باشد، ممکن است دوباره copy انجام شود.

---

## Move Constructor

فرض کنید کلاسی داریم که منبع پویا مدیریت می‌کند:

```cpp
class Buffer
{
    int* data_ = nullptr;
    std::size_t size_ = 0;

public:
    ~Buffer()
    {
        delete[] data_;
    }

    // Move Constructor
    Buffer(Buffer&& other) noexcept
        : data_(other.data_), size_(other.size_)
    {
        other.data_ = nullptr;
        other.size_ = 0;
    }

    // Move Assignment Operator
    Buffer& operator=(Buffer&& other) noexcept
    {
        if (this != &other)
        {
            delete[] data_;

            data_ = other.data_;
            size_ = other.size_;

            other.data_ = nullptr;
            other.size_ = 0;
        }

        return *this;
    }
};
```

when we have:

```cpp
Buffer a;
Buffer b = a; // Copy Constructor
```

اگر copy constructor تعریف شده باشد، عملیات copy انجام می‌شود.

but:

```cpp
Buffer b = std::move(a); // Move Constructor
```

در این حالت، اگر move constructor وجود داشته باشد، منابع `a` به `b` منتقل می‌شوند.

---

##  Move Assignment operations

همین ایده برای عملگر انتساب نیز وجود دارد.

Copy Assignment:

```cpp
a = b;
```

Move Assignment:

```cpp
a = std::move(b);
```

در Move Assignment، منابع شیء مقصد آزاد یا جایگزین می‌شوند و منابع شیء منبع به مقصد منتقل می‌شوند.

---

## why the Move is faster?

فرض کنید کلاسی داریم که یک بافر بزرگ را مدیریت می‌کند:

```cpp
class Buffer
{
    int* data_;
};
```

اگر اندازه بافر مثلاً 10,000,000 عدد باشد، copy یعنی:

1. allocating new memory
2. کپی کردن 10 میلیون عدد

اما move می‌تواند فقط چند اشاره‌گر یا مقدار داخلی را جابه‌جا کند:

```cpp
data_ = other.data_;
other.data_ = nullptr;
```

در چنین حالتی، پیچیدگی عملیات از حدود `O(n)` به `O(1)` کاهش پیدا می‌کند.

> نکته: برای اشیاء کوچک یا trivially copyable، ممکن است copy به اندازه کافی سریع باشد و move سود چشمگیری نداشته باشد.

---

## a real example

```cpp
std::vector<int> v1(1000000);
std::vector<int> v2 = std::move(v1);
```

without move:

- allocating new memory
- کپی کردن 1,000,000 عدد

with move:

- move the internal pointer
- move the size
- move the capacity

در پیاده‌سازی‌های معمول، این عملیات بسیار سریع است.

after move:

- آرایه `v2` داده‌های قبلی `v1` را دارد.
- آرایه `v1` در وضعیت معتبر ولی نامشخص قرار دارد.

> نکته مهم: Move Semantics فقط زمانی امن است که شیء مبدأ دیگر نیازی نباشد. به همین دلیل C++ به‌طور خودکار فقط برای rvalueها یا عباراتی که با `std::move` به rvalue تبدیل شده‌اند، عملیات move را انتخاب می‌کند.

---

## انواع پارامترها

| نوع پارامتر | مثال | توضیح |
|---|---|---|
| ارسال مقدار | `void foo(std::string s);` | پارامتر با copy یا move مقداردهی می‌شود. برای lvalue معمولاً copy و برای rvalue معمولاً move یا copy elision رخ می‌دهد. |
| مرجع به lvalue غیرثابت | `void foo(std::string& s);` | فقط lvalueهای غیرثابت را می‌پذیرد. |
| مرجع ثابت | `void foo(const std::string& s);` | هم lvalue و هم rvalue را می‌پذیرد، اما نمی‌توان از طریق آن شیء را تغییر داد. |
| مرجع به rvalue | `void foo(std::string&& s);` | مقادیر rvalueها را می‌پذیرد و پایه‌ای برای Move Semantics است. |

---

## difference between Copy و Move

### Copy Constructor

```cpp
std::string b = a;
```

در اینجا یک شیء جدید با کپی از `a` ساخته می‌شود.

### Move Constructor

```cpp
std::string b = std::move(a);
```

در اینجا یک شیء جدید با انتقال منابع `a` ساخته می‌شود، البته اگر move constructor مناسب وجود داشته باشد.

### Copy Assignment Operator

```cpp
b = a;
```

the value of `a` is copied in `b`

### Move Assignment Operator

```cpp
b = std::move(a);
```

منابع `a` به `b` منتقل می‌شود و معمولاً `a` در وضعیت معتبر ولی نامشخص قرار می‌گیرد.

---

## اشتباهات رایج

### 1. فکر کنیم `std::move` حتماً جابه‌جا می‌کند

دستور `std::move` به‌تنهایی هیچ داده‌ای را جابه‌جا نمی‌کند. این دستور فقط عبارت را به شکل rvalue درمی‌آورد.

```cpp
std::move(a); // is a type of castings
```

عملیات move زمانی اتفاق می‌افتد که move constructor یا move assignment مناسب فراخوانی شود.

---

### 2. فکر کنیم شیء move شده حتماً خالی یا `nullptr` است

after:
```cpp
std::string b = std::move(a);
```

نباید فرض کنیم `a` حتماً خالی است. برای بسیاری از کلاس‌های استاندارد، وضعیت `a` معتبر ولی نامشخص است.

می‌توان معمولاً آن را destroy کرد یا مقدار جدیدی به آن assign کرد، اما نباید روی محتوای قبلی آن حساب کرد.

---

### 3. reuse of an object after move

بعد از move، نباید از شیء مبدأ مثل قبل استفاده کنید.

```cpp
std::string a = "Hello";
std::string b = std::move(a);

```

استفاده از a بعد از move ممکن است از نظر منطقی اشتباه باشد،
مگر اینکه فقط عملیات‌های امن مثل assign جدید انجام شود.

---

### 4. using `std::move` on a `const` object

```cpp
const std::string a = "Hello";
std::string b = std::move(a);
```

در این حالت معمولاً move انجام نمی‌شود، زیرا move constructor معمولاً یک `std::string&&` غیرثابت می‌گیرد. در نتیجه ممکن است copy انجام شود.
به بیان بهتر چون در move constructor و move assignment قرار است مقدار داده اول پس از انتقال نا معتبر شود، معمولا ورودی این دو مورد را با شی rvalue reference && غیر const تعریف می کنند و ارسال داده const باعث می شود احتمالا این دو مورد move صدا زده نشوند و copy constructor یا copy assignment صدا زده شود!

---

### 5. using `return std::move(local)`

usually it's not recommended to do that:

```cpp
std::string make()
{
    std::string s = "Hello";
    return s; // Good
}
```

but the below line is not recommended:

```cpp
std::string make()
{
    std::string s = "Hello";
    return std::move(s); // not recommended
}
```

دلیل آن این است که `return std::move(s)` ممکن است جلوی بهینه‌سازی‌هایی مثل NRVO را بگیرد.

---

### 6. فراموش کردن `noexcept`

اگر کلاس شما move constructor یا move assignment دارد، بهتر است آن‌ها را `noexcept` اعلام کنید:

```cpp
Buffer(Buffer&& other) noexcept;
Buffer& operator=(Buffer&& other) noexcept;
```

بسیاری از کانتینرها، مثل `std::vector`، فقط در صورتی از move در reallocation استفاده می‌کنند که move constructor `noexcept` باشد یا شرایط خاصی برقرار باشد. در غیر این صورت ممکن است برای حفظ امنیت استثنا، copy انجام شود.

---

## جمع‌بندی

| مفهوم | توضیح |
|---|---|
| `lvalue` | عبارت دارای هویت مشخص که معمولاً می‌توان به آن ارجاع داد و آدرسش را گرفت. |
| `rvalue` | مقدار یا شیء موقتی که معمولاً عمر کوتاهی دارد و می‌توان از آن عملیات Move انجام داد. |
| `prvalue` | مقدار موقتی بدون هویت؛ مانند `5` یا نتیجهٔ تابعی که مقدار را با Value برمی‌گرداند. |
| `xvalue` | مقدار در حال انقضا؛ مانند نتیجهٔ `std::move(x)`. |
| `&&` | عملگر **rvalue reference** که پایهٔ Move Semantics محسوب می‌شود. |
| `std::move` | تابعی که عبارت را به **rvalue reference** تبدیل می‌کند و به‌تنهایی عملیاتی برای Move انجام نمی‌دهد. |
| Move Constructor | سازنده‌ای که شیء جدید را با انتقال منابع از یک شیء موجود ایجاد می‌کند. |
| Move Assignment | عملگر انتسابی که منابع شیء مقصد را با منابع شیء مبدأ جایگزین می‌کند، بدون انجام Copy پرهزینه. |
| Copy Constructor | سازنده‌ای که شیء جدید را با کپی کردن شیء موجود ایجاد می‌کند. |
| Copy Assignment | عملگر انتسابی که محتوای شیء مقصد را با کپی از شیء مبدأ جایگزین می‌کند. |
| Moved-from object | شیئی که عملیات Move روی آن انجام شده و معمولاً معتبر است، اما مقدار آن مشخص نیست. |

---

## References

- https://en.cppreference.com/w/cpp/utility/move
- https://en.cppreference.com/w/cpp/language/move_constructor
- https://en.cppreference.com/w/cpp/language/value_category

توجه: این مقاله با استفاده از توضیحات و بازنویسی انجام‌شده توسط **ChatGPT (OpenAI)** و **Qwen (Alibaba)** تهیه و ویرایش شده است.

## 🤝 Contributers

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>