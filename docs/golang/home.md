#

go 语言的优点

1.相比于 python 自带编译器，可以轻易检测出低级语法错误
2.开发速度快，一般 java / c 等编译型语言的开发速度会很慢，但 golang 的开发速度并不慢
3.并行性能高， go 语言能够很方便的直接运行多线程代码，相比于 python 受限于全局解释器来说方便的多
4.部署简单，所有的依赖都可以简单的编译成一个二进制包
5.风格一致，gofmt 可以一键式将 go 语言的格式标准化

go 语言的缺点

1.库支持少。甚至 Math.round() 只支持证书 不支持浮点数的取整
2.不够灵活。强制在 GOPATH 中编程。强制使用特定的文件组织结构 src bin pkg 等


[golang 中的自旋锁](./golang_spin_lock.md)
[go 1.12的若干变化](./some-changes-in-go1.12.md)


## go 进阶

* GPM 模型
* goroutine 调度
* channel 调度
* 内存布局
* 指针陷阱
* CGO
* 反射
* 内存管理
* GC
* 
