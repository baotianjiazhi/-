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

4.func main() {
    
    s1 := []int{1,2,3}
    s2 := []int{4,5,6}
    
    s = append(s1,s2)
    fmt.Println(s1)
}
---
上述代码编译能通过吗？
答：不能，因为在连接两个切片的时候，追加的切片需要“拍扁”即需要元素一个一个的传入,s1 = append(s1, s2...)

new和make的区别：
​ 二者都是内存的分配（堆上），但是make只用于slice、map以及channel的初始化（非零值）；而new用于类型的内存分配，并且内存置为零。所以在我们编写程序的时候，就可以根据自己的需要很好的选择了。
​ make返回的还是这三个引用类型本身；而new返回的是指向类型的指针。
```

## 单例模式

英文名称：Singleton Pattern，该模式规定一个类只允许有`一个实例`，而且自行实例化并向整个系统提供这个实例。因此单例模式的要点有：1）只有一个实例；2）必须自行创建；3）必须自行向整个系统提供这个实例。

单例模式主要避免一个全局使用的类频繁地创建与销毁。当你想控制实例的数量，或有时候不允许存在多实例时，单例模式就派上用场了。

## Goroutine或内存泄露的场景

1. Time.After导致内存泄露

```golang
func ProcessMessage(ctx context.Context, in <-chan string) {
	for {
		select {
		case s, ok := <-in:
			if !ok {
				return
			}
			// handle `s`
		case <-time.After(5 * time.Minute):
			// do something
		case <-ctx.Done():
			return
		}
	}
}
```

> 等待持续时间过去，然后在返回的channel上发送当前时间。它等效于NewTimer().C。在计时器出发之前，计时器不会被垃圾收集器回收。

所以假设还没等到五分钟，该函数就已经返回，计时器就不会被GC回收，因此出现了内存泄漏。所以，一般选择使用time.NewTimer:

```golang
func ProcessMessage(ctx context.Context, in <-chan string) {
	idleDuration := 5 * time.Minute
	idleDelay := time.NewTimer(idleDuration)
  // 这句必须的
	defer idleDelay.Stop()
	for {
		idleDelay.Reset(idleDuration)
		select {
		case s, ok := <-in:
			if !ok {
				return
			}
			// handle `s`
		case <-idleDelay.C:
			// do something
		case <-ctx.Done():
			return
		}
	}
}
```

2. 发送到无缓冲channel阻塞导致goroutine泄露

假设存在如下的程序

```go
func process(term string) error {
     // 创建一个在 100 ms 内取消的 context
     ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
     defer cancel()

     // 为 goroutine 创建一个传递结果的 channel
     ch := make(chan string)

     // 启动一个 goroutine 来寻找记录，然后得到结果
     // 并将返回值从 channel 中传回
     go func() {
         ch <- search(term)
     }()

     select {
     case <-ctx.Done():
         return errors.New("search canceled")
     case result := <-ch:
         fmt.Println("Received:", result)
         return nil
    }
 }

// search 模拟成一个查找记录的函数
// 在查找记录时。执行此工作需要 200 ms。
func search(term string) string {
     time.Sleep(200 * time.Millisecond)
     return "some value"
}
```

如果超时没处理完，ctx.Done 会执行，函数返回，新开启的 goroutine 会因为 channel 中的另一端没有就绪的接收 goroutine 而一直阻塞，导致 goroutine 泄露。

解决这种因为发送到 channel 阻塞导致 goroutine 泄露的简单办法是将 channel 改为有缓冲的 channel，并保证容量充足。比如上面例子，将 ch 改为：ch := make(chan string, 1) 即可。

3. 从channel接收阻塞导致goroutine泄露

```go
func (u *User) SendMessage(ctx context.Context) {
	for msg := range u.MessageChannel {
		wsjson.Write(ctx, u.conn, msg)
	}
}
```

for-range 循环直到 MessageChannel 这个 channel 关闭才会结束，因此需要有地方调用 close(u.MessageChannel)。

这种情况的另一种情形是：虽然没有 for-range，但给 channel 发送数据的一方已经不再发送数据了，接收的一方还在等待，这个等待会无限持续下去。唯一能取消它等待的就是 close 这个 channel。