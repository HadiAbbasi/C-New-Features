# auto

## English

### What is it?
`auto` lets the compiler deduce the type of a variable from its initializer. Introduced in C++11, it reduces verbosity while preserving static typing.

### Syntax
```cpp
auto variable = expression;
```

### Example
```cpp
auto i = 42;          // int
auto d = 3.14;        // double
auto s = std::string("Hello");

std::vector<int> v{1,2,3};
for (auto& x : v)
    x *= 2;
```

### Why was it added?
- Reduce repetitive type names
- Improve maintainability
- Simplify generic programming

### Performance
`auto` has **zero runtime overhead**. Type deduction happens entirely at compile time.

### Best Practices
- Prefer `auto` when the type is obvious from the initializer.
- Avoid `auto` if it hides important semantic information.

---

## فارسی

### auto چیست؟
کلمهٔ کلیدی `auto` به کامپایلر اجازه می‌دهد نوع متغیر را از روی مقدار اولیه به صورت خودکار استنتاج کند. این قابلیت در C++11 معرفی شد تا کدها خواناتر و نگهداری آن‌ها ساده‌تر شود.

### سینتکس
```cpp
auto variable = expression;
```

### مثال
```cpp
auto count = 10;
auto price = 19.95;

std::vector<int> values{1,2,3};
for (auto& value : values)
    value++;
```

### چرا اضافه شد؟
- حذف تکرار نام نوع‌ها
- افزایش خوانایی کد
- ساده‌تر شدن Generic Programming

### نکات مهم
- `auto` همیشه به مقدار اولیه نیاز دارد.
- نوع واقعی در زمان کامپایل تعیین می‌شود.
- `const` و `&` را در صورت نیاز باید صریح نوشت.

### اشتباهات رایج
```cpp
auto x;      // Error
```

```cpp
const auto& ref = object;
```

## References
- https://en.cppreference.com/w/cpp/language/auto
- ISO C++11
