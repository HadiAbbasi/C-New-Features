<div align="right">

[🇺🇸 English](../../en/cpp11/std_move.md) | [🇮🇷 فارسی](./std_move.md)

</div>

---

# مبحث Move Semantics، `std::move` و مفاهیم `lvalue` و `rvalue` در ++C

> **نسخه معرفی:** C++11

## مقدمه

مبحث انتقال مقدار توسط std::move یکی از مهم‌ترین مفاهیم C++ مدرن (از C++11 به بعد) است. برای درک بهتر این مبحث، دانستن انواع داده های **lvalue** و **rvalue** بسیار ضروری است که فهم **move semantics** و **rvalue references** را بسیار ساده تر می کند.

---

# مقادیر lvalue و rvalue یعنی چه؟

به زبان ساده:

* **مقادیر lvalue**: چیزی که **مکان مشخصی در حافظه دارد** و می‌توان بعداً دوباره به آن مراجعه کرد.
* **مقادیر rvalue**: یک **مقدار موقتی (temporary)** که معمولاً فقط در همان عبارت (expression) وجود دارد.

مثال:

```cpp
int x = 10;
```

اینجا:

* داده `x` یک **lvalue** است.
* داده `10` یک **rvalue** است.

چون:

```cpp
x = 20;
```

یعنی `x` یک محل در حافظه دارد.

ولی:

```cpp
10 = x;
```

غلط است.

چون `10` فقط یک مقدار است و جایی برای ذخیره شدن ندارد.

---

## مثال دیگر

```cpp
int a = 3;
int b = a;
```

اینجا:

```
a   -> lvalue
b   -> lvalue
3   -> rvalue
```

---

## نتیجه‌ی یک تابع

اگر تابع اینگونه باشد:

```cpp
int foo()
{
    return 5;
}
```

آنگاه

```cpp
foo()
```

یک **rvalue** است.

زیرا نتیجه‌ی تابع یک مقدار موقتی است.

---

اما:

```cpp
int x;

int& foo()
{
    return x;
}
```

حالا

```cpp
foo()
```

یک **lvalue** است.

چون رفرنس به یک شیء واقعی برمی‌گرداند.

---

# چطور تشخیص بدهیم؟

فرض کنید این کد را دارید:

```cpp
int x = 5;
```

اگر بتوانیم از آن آدرس بگیریم:

```cpp
&x;
```

پس lvalue است. ولی

```cpp
&(x + 1);
```

غلط است. چون

```cpp
x + 1
```

یک rvalue است.

---

# چرا اصلاً این تقسیم‌بندی وجود دارد؟

چون کامپایلر می‌خواهد بداند این شیء:

* قرار است بعداً هم استفاده شود؟
* یا فقط یک مقدار موقتی است؟

این اطلاعات برای بهینه‌سازی بسیار مهم است.

---

# قبل از C++11 چه مشکلی وجود داشت؟

فرض کنید:

```cpp
std::string s = "Hello";
```

و تابعی داریم:

```cpp
void print(std::string str){}
```

اگر بنویسیم:

```cpp
print(s);
```

اتفاق می‌افتد:

```
s
 ↓
copy
 ↓
str
```

یک کپی کامل ساخته می‌شود.

---

حالا:

```cpp
print("Hello");
```

باز هم

```
temporary string
        ↓
      copy
        ↓
      str
```

یعنی حتی شیء موقتی هم کپی می‌شود. در حالی که آن شیء دیگر قرار نیست استفاده شود. این اتلاف حافظه و زمان است.

---

# عملیات Move Semantics چه می‌گوید؟

ایده بسیار ساده است:

اگر شیء موقتی است،  دیگر لازم نیست از آن **کپی** بگیریم. می‌توانیم منابعش را **بدزدیم (steal)**.

---

فرض کنید:

```
string A

+----------------------+
| Hello World          |
+----------------------+
```

کپی یعنی:

```
A ------copy------> B

A -> Hello World
B -> Hello World
```

دو حافظه لازم داریم.

---

ولی Move:

```
A ----move----> B

A -> nullptr
B -> Hello World
```

فقط اشاره‌گر منتقل می‌شود. هیچ داده‌ای کپی نمی‌شود.

---

# این کار چگونه ممکن شد؟

نسخه C++11 چیزی به نام **rvalue reference** معرفی کرد. علامتش:

```cpp
&&
```

مثال:

```cpp
void foo(std::string&& s){}
```

این تابع فقط rvalue قبول می‌کند. مثلاً:

```cpp
foo(std::string("Hello"));
```

یا

```cpp
foo("Hello");
```

درست است. اما

```cpp
std::string s = "Hello";

foo(s);
```

غلط است. چون `s` یک lvalue است.

---

# دستور std::move چیست؟

فرض کنید:

```cpp
std::string s = "Hello";
```

اگر بنویسیم:

```cpp
std::move(s)
```

در واقع نمی‌گوید چیزی را جابه‌جا کن. بلکه فقط به کامپایلر می‌گوید:

> "از این لحظه با `s` مثل یک rvalue رفتار کن."

بعد اگر کلاس Move Constructor داشته باشد:

```cpp
std::string b = std::move(s);
```

اتفاق می‌افتد:

```
s
 ↓
move
 ↓
b
```

بعد از آن:

```
b = Hello

s = valid but unspecified
```

یعنی `s` هنوز یک شیء معتبر است، اما مقدارش مشخص نیست و نباید روی محتوای قبلی آن حساب کنی.

---

# Move Constructor

فرض کنید کلاس زیر را داریم:

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

وقتی بنویسیم:

```cpp
String a;

String b = a;
```

از

```
copy constructor
```

استفاده می‌شود. اما:

```cpp
String b = std::move(a);
```

از

```
move constructor
```

استفاده می‌شود.

---

# Move Assignment

همین ایده برای عملگر انتساب هم وجود دارد:

```cpp
a = b;
```

↓

Copy Assignment

ولی

```cpp
a = std::move(b);
```

↓

Move Assignment

---

# چرا Move سریع‌تر است؟

فرض کنید کلاس زیر را داریم:

```cpp
class Buffer
{
    int* data;
};
```

اگر اندازه بافر

```
10,000,000
```

عدد باشد، کپی یعنی:

```
Allocate

Copy 10 million ints
```

اما Move فقط:

```cpp
data = other.data;
other.data = nullptr;
```

سه خط ساده است. پیچیدگی زمانی تقریباً از **O(n)** به **O(1)** کاهش پیدا می‌کند.

---

# یک مثال واقعی

```cpp
std::vector<int> v1(1000000);

std::vector<int> v2 = std::move(v1);
```

بدون Move:

```
Allocate

Copy 1,000,000 ints
```

با Move:

```
pointer

size

capacity
```

فقط همین سه مقدار جابه‌جا می‌شوند.

---

نکته‌ی مهم این است که **Move Semantics فقط زمانی امن است که شیء مبدأ دیگر لازم نباشد**. به همین دلیل C++ به‌طور خودکار فقط برای **rvalue**ها (یا اشیایی که با `std::move` به rvalue تبدیل شده‌اند) از عملیات move استفاده می‌کند؛ زیرا فرض می‌شود این اشیا دیگر قرار نیست مانند قبل مورد استفاده قرار گیرند.

یکی از مهم‌ترین قابلیت‌هایی که از C++11 به زبان اضافه شد، **Move Semantics** است. این قابلیت باعث می‌شود بسیاری از عملیات‌هایی که قبلاً با کپی کردن اشیاء انجام می‌شدند، تنها با انتقال مالکیت منابع انجام شوند و در نتیجه برنامه هم سریع‌تر اجرا شود و هم حافظه‌ی کمتری مصرف کند.

برای درک Move Semantics ابتدا باید با دو مفهوم **lvalue** و **rvalue** آشنا شویم.

## مقادیر lvalue چیست؟

شی lvalue به شیئی گفته می‌شود که مکان مشخصی در حافظه دارد و بعداً نیز می‌توان دوباره به آن دسترسی داشت.

```cpp
std::string name = "Ali";
```

در اینجا `name` یک **lvalue** است.

## شی rvalue چیست؟

شی rvalue یک مقدار موقتی (Temporary Object) است که معمولاً فقط در همان عبارت وجود دارد.

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

## دستور `std::move`

تابع `std::move` چیزی را جابه‌جا نمی‌کند؛ فقط شیء را به یک rvalue تبدیل می‌کند تا در صورت وجود، Move Constructor یا Move Assignment فراخوانی شود.

```cpp
std::string a = "Hello";
std::string b = std::move(a);
```

## فرق Copy و Move

### سازنده کپی یا Copy Constructor

```cpp
std::string b = a;
```

### کانسترکتور انتقال یا Move Constructor

```cpp
std::string b = std::move(a);
```

### عملگر انتساب کپی یا Copy Assignment Operator

```cpp
b = a;
```

### عملگر انتساب از طریق انتقال یا Move Assignment Operator

```cpp
b = std::move(a);
```

## جمع‌بندی

| مفهوم            | توضیح                                                                                                     |
| ---------------- | --------------------------------------------------------------------------------------------------------- |
| lvalue           | شیئی با مکان مشخص در حافظه که می‌توان چندباره به آن دسترسی داشت.                                          |
| rvalue           | مقدار موقتی که معمولاً فقط تا پایان عبارت زنده است.                                                       |
| `&&`             | رفرنس به rvalue (rvalue reference).                                                                       |
| `std::move`      | تبدیل شیء به rvalue، شیء را جابه‌جا نمی‌کند؛ فقط آن را به صورت یک rvalue معرفی می‌کند تا در صورت امکان، عملیات move انجام شود. |
| `std::string s` | ارسال به صورت مقدار (Pass by Value) |
| `std::string& s` | رفرنس به lvalue |
| `const std::string& s` | رفرنس ثابت |
| `std::string&& s` | رفرنس به rvalue |
| Copy Assignment | انتساب با کپی، منابع را در انتساب منتقل می‌کند و معمولاً سریع‌تر از کپی است. |
| Move Assignment  | انتساب با انتقال منابع، منابع را در انتساب منتقل می‌کند و معمولاً سریع‌تر از کپی است.                                             |
| Copy Constructor | ساخت شیء جدید با کپی |
| Move Constructor | ساخت شیء جدید با انتقال منابع، منابع شیء مبدأ را به شیء جدید منتقل می‌کند و از کپی پرهزینه جلوگیری می‌کند.                               |

---

## منابع

- https://en.cppreference.com/w/cpp/utility/move
- https://en.cppreference.com/w/cpp/language/move_constructor
- https://en.cppreference.com/w/cpp/language/value_category
- این مقاله با استفاده از توضیحات و بازنویسی انجام‌شده توسط **ChatGPT (OpenAI)** تهیه و ویرایش شده است.

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>