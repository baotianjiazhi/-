## Golang

### 非安全指针

第一是 `unsafe.Pointer` 可以让你的变量在不同的指针类型转来转去，也就是表示为任意可寻址的指针类型.第二是 `uintptr` 常用于与 `unsafe.Pointer` 打配合，用于做指针运算。

