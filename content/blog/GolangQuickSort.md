+++
title = "GolangQuickSort"
author = "lambda@lambda.tw"
categories = ["Golang"]
tags = ["Golang"]
date = "2018-08-05"
description = ""
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
+++
# 用 Golang 實做快速排序 (quick sort)

快速排序是很常用的一個排序方法，下方我將會用 Golang 實做同步以及異步的快速排序。

## 同步

### 實做

```go
func sort(list []int, center int) (complete []int) {
	left := []int{}
	right := []int{}
	for _, num := range list[:center] {
		if num <= list[center] {
			left = append(left, num)
		} else {
			right = append(right, num)
		}
	}
	if len(list) > center+1 {
		for _, num := range list[center+1:] {
			if num <= list[center] {
				left = append(left, num)
			} else {
				right = append(right, num)
			}
		}
	}
	if len(left) > 1 {
		left = sort(left, len(left)/2)
	}
	if len(right) > 1 {
		right = sort(right, len(right)/2)
	}

	return append(append(left, list[center]), right...)
}
```



## 異步

### go

Golang 強大的異步使用 goroutine 讓寫異步程式如喝水般簡單，go 不同於其他語言使用 thread 或是 process (fork) 之類的的方法，他在底層運用自己強大的 go scheduler 讓每個異步程序可以在作業系統可用執行緒改變下，依然可以執行你所要跑得程序。![go-scheduler](http://jolestar.com/images/concurrent/go-scheduler.png)

#### 實做

```go
package main

import "fmt"

func main() {
    go func() {
        fmt.Println("time need")
    }
}
```

執行指令 `go run main.go` 你會發現沒東西，因為它將無名 function 放到背景後程式就結束了，為了要可以看到我們要 print 的東西我們讓它睡覺一下。

```go
package main

import (
	"fmt"
    "time"
)

func main() {
    go func() {
        fmt.Println("You can see me.")
    }
    time.Sleep(100 * time.Millisecond)
}
```

此時再次執行 `go run main.go` 你會發現可以看到我們要輸出的字串了！

#### 為什麼？

```
# 原本沒有等待的程式
--> main.go --> go func --> end
						goroutine wait --> 主程式結束所以沒有 print
# 等待的程式
--> main.go --> go func --> wait...............................--> end -- 在 go func 跑完 Sleep 後才結束
						goroutine wait --> print
```

#### channel

Golang 在 goroutine 中所使用的溝通媒介，類似其他語言多 threading 使用全域的 Queue

##### 實做

```go
package main

import "fmt"

func main() {
	c := make(chan int)
	// 在背景執行 把 100 丟到 c 裡面
	go func() {
		c <- 100
	}()
	// 從 c 裡面拿值，此處會等待 c 有值為止才會執行
	Print(<-c)
}
```

## 速度比較
### Golang 的測試
在 Golang 寫測試很簡單只需要在同一目錄中使用相同 package ，並且檔案名稱以 _test.go 結尾，並且把要測試的程式接收依照特殊的參數以及命名即可
```go
// 跑 test，以 Test 當作 function 開頭並接收 (t *testing.T) 參數
func TestSomeThing(t *testing.T) {

}
// 跑 benchmark，以 Benchmark 當作 function 開頭並接收 (t *testing.B) 參數
func BenchmarkSomeThing(t testing.*B) {

}
```
#### 建立測試檔案
```shell-script
touch $GOPATH/project/sort_test.go
code $GOPATH/project/sort_test.go
```
#### 實做
```go
package main

import (
	"fmt"
	"testing"
)

var needSort = []int{}

func init() {
	for i := 0; i < 1000000; i++ {
		needSort = append(needSort, rand.Intn(1000000))
	}
}

func BenchmarkSync(b *testing.B) {
	// fmt.Println("call BenchmarkSync", len(needSort))
	sort(needSort, len(needSort)/2)
}

func BenchmarkAsync(b *testing.B) {
	// fmt.Println("call BenchmarkAsync", len(needSort))

	cmp := make(chan int)
	go goSort(needSort, int(len(needSort)/2), cmp)
	for i := 0; i < len(needSort); i++ {
		<-cmp
	}
}
```
#### 跑測試
##### 執行
```shell-script
go test -bench=.
```
##### 輸出
最後可以看到同步的程式 go test 幫我們跑了 2000000000 次，平均每次只要跑 0.24 ns，相較於非同步程式，跑一次就要花 6650871377 ns 快上非常多
```text
goos: linux
goarch: amd64
pkg: gosort
BenchmarkSync-4		2000000000	0.24 ns/op
BenchmarkAsync-4	1		6650871377 ns/op
PASS
ok  	gosort	15.644s
go test -bench=.  27.97s user 1.55s system 185% cpu 15.910 total
```
## 結論
非同步的程式需要等待 go routing 幫它開啟一些東西，就速度上不一定會比較快，它的好處當然就是不用等它跑完，也可以分多個線程下去加快速度，但是如果沒有優化好就會像上面的程式一樣慢慢的。
