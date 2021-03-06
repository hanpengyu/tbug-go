### 16、go语句及其执行规则

Go语言有着独特的并发编程模型，以及用户级线程 goroutine，还拥有强大的用于调度 goroutine、对接系统级线程的调度器。  
这个调度器是 Go 语言运行时系统的重要组成部分，它主要负责统筹调配 Go 并发编程模型中的三个主要元素，即：G（goroutine 的缩写）、P（processor 的缩写）和 M（machine 的缩写）。

#### 这个命令源码文件被执行后会打印出什么内容？

```go
package main

import "fmt"

func main() {
	for i := 0; i < 10; i++ {
		go func() {
			fmt.Println(i)
		}()
	}
}


// 不会输出任何的内容。
// 主goroutine执行完毕之后，是不会等待go func 函数的执行，所以go func来不及执行（也就是说go程序会立即结束运行）

```

#### 下面这个会打印出什么内容？
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	for i := 0; i < 10; i++ {
		go func() {
			fmt.Println(i)
		}()
	}

	time.Sleep(time.Second * 1)
}

/* 会输出十次，但是并不是顺序执行的，有重复的数据 */
// 出现重复是因为fmt.Println()执行的时候对i是引用


```

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	for i := 0; i < 10; i++ {
		go func(i int) {
			fmt.Println(i)
		}(i)
	}

	time.Sleep(time.Second * 1)
}

/* 输出10次，不过是无序的 */
// Go 语言并不会去保证这些 goroutine 会以怎样的顺序运行。
// 所以哪个 goroutine 先执行完、哪个 goroutine 后执行完往往是不可预知的。

/*
P (* * * 1)
P (* * * * 0)
P (* * 2)
输出 2 1 0
*/

```

#### 10次全部输出
```go
package main

import "fmt"

func main() {
	num := 10
	sign := make(chan struct{}, num)
	for i := 0; i < num; i++ {
		go func() {
			fmt.Println(i)
			sign <- struct{}{}
		}()
	}

	for j := 0; j < num; j++ {
		<-sign
	}
}
```
#### 打印出有序的怎么写
```go
package main

import (
	"fmt"
	"sync/atomic"
	"time"
)

func main() {
	var count uint32
	trigger := func(i uint32, fn func()) {
		for {
			if n := atomic.LoadUint32(&count); n == i {
				fn()
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
}

// go func 那个循环每次都会拿到唯一的一个值，即使是2的先到trigger也不会先执行，因为还有count在控制，所以基本上就是很多trigger在等待
// trigger(3, fn)
// trigger(1, fn)
// trigger(5, fn)
//每个trigger又在自旋所以能保证打印的顺序，但是这样会不会影响并发的性能，这个我还得想想。






```