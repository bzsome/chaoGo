[![Build Status](https://travis-ci.org/xxjwxc/gowp.svg?branch=master)](https://travis-ci.org/xxjwxc/gowp)
[![Go Report Card](https://goreportcard.com/badge/github.com/bzsome/chaoGo)](https://goreportcard.com/report/github.com/bzsome/chaoGo)
[![codecov](https://codecov.io/gh/xxjwxc/gowp/branch/master/graph/badge.svg)](https://codecov.io/gh/xxjwxc/gowp)
[![GoDoc](https://godoc.org/github.com/bzsome/chaoGo?status.svg)](https://godoc.org/github.com/bzsome/chaoGo)
[![Mentioned in Awesome Go](https://awesome.re/mentioned-badge.svg)](https://github.com/avelino/awesome-go)


## golang worker pool ,线程池 , 工作池
### [English](README_cn.md)
- 并发限制goroutine池。
- 限制任务执行的并发性，而不是排队的任务数。
- 无论排队多少任务，都不会阻止提交任务。
- 通过队列支持[queue](https://github.com/bzsometest/public/tree/master/myqueue)

### golang 工作池公共库

## 安装

安装最简单方法:

```
$ go get github.com/bzsome/chaoGo
```

### 支持最大任务数, 放到工作池里面 并等待全部完成

```
package main

import (
	"fmt"
	"time"

	"github.com/bzsome/chaoGo/workpool"
)

func main() {
	wp := workpool.New(10)             //设置最大线程数
	for i := 0; i < 20; i++ { //开启20个请求
		ii := i
		wp.Do(func() error {
			for j := 0; j < 10; j++ { //每次打印0-10的值
				fmt.Println(fmt.Sprintf("%v->\t%v", ii, j))
				time.Sleep(1 * time.Second)
			}
			//time.Sleep(1 * time.Second)
			return nil
		})
	}

	wp.Wait()
	fmt.Println("down")
}
```

### 支持错误返回
```
package main

import (
	"fmt"
	"time"

	"github.com/bzsome/chaoGo/workpool"
)

func main() {
	wp := workpool.New(10)             //设置最大线程数
	for i := 0; i < 20; i++ { //开启20个请求
		ii := i
		wp.Do(func() error {
			for j := 0; j < 10; j++ { //每次打印0-10的值
				fmt.Println(fmt.Sprintf("%v->\t%v", ii, j))
				if ii == 1 {
					return errors.Cause(errors.New("my test err")) //有err 立即返回
				}
				time.Sleep(1 * time.Second)
			}

			return nil
		})
	}

	err := wp.Wait()
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println("down")
	}
```

### 支持判断是否完成 (非阻塞)

```
package main

import (
	"fmt"
	"time"

	"github.com/bzsome/chaoGo/workpool"
)

func main() {
	wp := workpool.New(5)              //设置最大线程数
	for i := 0; i < 10; i++ { //开启20个请求
		//	ii := i
		wp.Do(func() error {
			for j := 0; j < 5; j++ { 
				//fmt.Println(fmt.Sprintf("%v->\t%v", ii, j))
				time.Sleep(1 * time.Second)
			}
			return nil
		})

		fmt.Println(wp.IsDone())//判断是否完成
	}
	wp.Wait()
	fmt.Println(wp.IsDone())
	fmt.Println("down")
}
```

### 支持同步等待结果

```
package main

import (
	"fmt"
	"time"

	"github.com/bzsome/chaoGo/workpool"
)

func main() {
	wp := workpool.New(5)              //设置最大线程数
	for i := 0; i < 10; i++ { //开启20个请求
		ii := i
		wp.DoWait(func() error {
			for j := 0; j < 5; j++ {
				fmt.Println(fmt.Sprintf("%v->\t%v", ii, j))
				// if ii == 1 {
				// 	return errors.New("my test err")
				// }
				time.Sleep(1 * time.Second)
			}

			return nil
			//time.Sleep(1 * time.Second)
			//return errors.New("my test err")
		})
	}

	err := wp.Wait()
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println("down")
}
```

### 支持超时退出

```
package main

import (
	"fmt"
	"time"
	"time"
	"github.com/bzsome/chaoGo/workpool"
)

func main() {
	wp := workpool.New(5)              // 设置最大线程数
		wp.SetTimeout(time.Millisecond) // 设置超时时间
	for i := 0; i < 10; i++ { // 开启20个请求
		ii := i
		wp.DoWait(func() error {
			for j := 0; j < 5; j++ {
				fmt.Println(fmt.Sprintf("%v->\t%v", ii, j))
				time.Sleep(1 * time.Second)
			}

			return nil
		})
	}

	err := wp.Wait()
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println("down")
}
```