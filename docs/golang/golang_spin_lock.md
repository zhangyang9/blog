# golang 自旋锁

## linux 内核中的自旋锁
自旋锁最多可被一个线程持有，如果其他线程想要持有一个被占用的自旋锁，就必须一直循环-旋转（不断的判断该锁是否释放）-等待自旋锁被持有锁的线程释放。    
自旋锁可以保证锁在多线程并发时只被一个线程持有    
`注意：`
* 一个被争用的自旋锁会导致请求它的线程在等待的过程一直自旋，很耗 cpu ，故自旋锁不能被长时间持有，必须用在`短期轻量级加锁`
* linux 内核自旋锁不可递归，自旋等待自己持有的锁释放会造成死锁
* 互斥锁和自旋锁的区别是互斥锁机制会让等待锁释放的线程进入睡眠状态而不是一直循环判断，请求睡眠会让线程进入两次上下文切换，所以自旋持有锁的时间要小于两次上下文切换的时间其性能才会优于互斥锁
* 自旋锁无法保证 **公平性**(先等待的线程先获得缩) 也无法保证 **可重入性**（已经获得锁的线程再次获得该锁）
* TichetLock:每当有一个线程想要获取锁时给该线程分配一个id：（排队号A），锁本身会有一个id(服务号B),每当一个线程释放该锁时 B 会自增，只有A == B 时，A代表的线程才会获得该锁，实现公平性，类似于银行排号



## golang 中的自旋锁实现

```golang
type spinLock uint32
func (sl *spinLock) Lock() {
    for !atomic.CompareAndSwapUint32((*uint32)(sl), 0, 1) {
        runtime.Gosched()
    }
}
func (sl *spinLock) Unlock() {
    atomic.StoreUint32((*uint32)(sl), 0)
}
func NewSpinLock() sync.Locker {
    var lock spinLock
    return &lock
}

```
应用实例: 使用自旋锁让并发的多个线程按照顺序执行
```golang
package main

import (
	"fmt"
	"sync/atomic"
	"time"
)

func main() {
	var count uint32
	trigger := func(i uint32, fn func()) {
		// 自旋锁 CAS 算法
		for {
			if n := atomic.LoadUint32(&count); n == i {
				fn()
				// 执行完函数后 原子 +1
				atomic.AddUint32(&count, 1)
				break
			}
			time.Sleep(time.Nanosecond)
		}
	}
	for i := uint32(0); i < 10; i++ {
		go func(i uint32) {
			fn := func() {
				fmt.Println(i)
			}
			trigger(i, fn)
		}(i)
	}
	trigger(10, func() {})
	//按照自然顺序打印
}

```












