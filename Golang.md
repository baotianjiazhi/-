## Golang

### 非安全指针

第一是 `unsafe.Pointer` 可以让你的变量在不同的指针类型转来转去，也就是表示为任意可寻址的指针类型.第二是 `uintptr` 常用于与 `unsafe.Pointer` 打配合，用于做指针运算。



## 接口

> 在 Go 语言的语义上，只要某个类型实现了所定义的一组方法集，则就认为其就是同一种类型，是一个东西。大家常常称其为鸭子类型（Duck typing），因为其与鸭子类型类型的定义相对吻合。

在`Golang`中`interface`包括两种数据结构：`iface`和`eface`

* `runtime.eface`结构体：不包含任何方法的空接口，也称为empty interface
* `runtime.iface`结构体：表示包含方法的接口

```golang
type efaces struct {
    _type *_type
    data unsafe.Pointer
}

type iface struct {
    tab *itab
    data unsafe.Pointer
}
```



## 面试题

```go
1.func main() {
    var whatever [6]struct{}
    for i := range whatever {
        defer func() {
            fmt.Println(i)
        }()
    }
}

---
上述代码打印结果如何？
正确答案：5 5 5 5 5 5
解释为何全是5，而不是 0，1，2，3，4，5这样的输出？
其根本原因是`闭包`所导致的，有两点原因：
* 在`for`循环结束后，局部变量`i`的值已经是5了，并且defer的闭包直接引用变量i
* 结合`defer`关键字的特性，可得知会在`main`方法执行结束后再执行

2. 结构比较的几个注意点
* 只有`相同类型`的结构体才可以比较，结构体是否相同不但与`属性类型个数`有关，还与`属性顺序`相关.
* 结构体是`相同`的，但是结构体属性中有`不可以比较的类型`，如map,slice，则结构体不能用==比较.

3. `nil`的作用
nil可以作用于`interface`、`function`、`pointer`、`map`、`slice`和`channel`的`空值`，不能使用于`string`等常规数据类型

4.
```

