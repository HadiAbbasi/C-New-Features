<div align="right">

[🇺🇸 English](./lambda.md) | [🇮🇷 فارسی](../../fa/cpp11/lambda.md)

</div>

# Lambda Expression
`Lambda expressions` are anonymous and inline functions introduced in C++11 that allow writing small pieces of logic directly at the place of use. They improve code readability by keeping behavior close to where it is applied and remove the need for separate named functions.

> - **Used to assign custom logic to algorithms like sort, for_each, and find_if in C++ STL**.
> - **Used in Callbacks and Event Handling for asynchronous tasks and events.**


## Struct Lambda
![Struct Lambda](./assets/lambda_expression_in_c_.webp)
```cpp
[capture](parameters) -> return_type {
    // body
};
```