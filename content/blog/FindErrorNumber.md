+++
title = "FindErrorNumber"
author = "lambda@lambda.tw"
categories = ["Golang"]
tags = ["Golang"]
date = "2018-07-12"
description = ""
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
+++
# 找出錯誤的數字，使用 Golang

## 題目

在輸入一連串的數字中(從一開始連續[1, 2, 3, 4, 5, 6])找到錯誤(重複)的數字，並且把錯誤的先列出來再將正確的數字附加到到後面

### 輸入

```
[1,2,2,4]
[1,2,2,4,5,5,7]
```



### 輸出

```
[2,3]
[2,5,3,6]
```



## 實做

### main.go

{{< highlight go >}}
package main

import "fmt"

func findErrorNums(nums []int) (ret []int) {
	dict := make(map[int]int)
	for _, num := range nums {
		_, ok := dict[num]
		if ok {
			ret = append(ret, num)
		} else {
			dict[num] = num
		}
	}
	for i := 1; i <= len(nums); i++ {
		_, ok := dict[i]
		if !ok {
			ret = append(ret, i)
		}
	}
	return

}

func main() {
	input := []int{1, 2, 2, 4}
	output := findErrorNums(input)
	fmt.Println(output)
}
{{< /highlight >}}

## 細解

### Package

宣告這檔案在哪個 `package` 裡面，若是在別的 package 裡面你可以在別的 `package` `import` 後直接呼叫

- $GOPATH/src/my/m.go

  ```go
  package my

  import "fmt"

  func Pm(str string) {
      fmt.Println("Pm: ", str)
  }

  ```

- $GOPATH/src/my/y.go

  ```go
  package my

  import "fmt"

  func Py(str string) {
      fmt.Println("Py: ", str)
  }

  ```

- $GOPATH/src/hello/main.go

  ```go
  package main

  import "my"

  func main() {
      my.Pm("Hello")
      my.Py("Hello")
  }

  ```

```
cd $GOPATH/src/hello/
go run main.go
Pm:  Hello
Py:  Hello
```

### import

引入別人寫的或是自己寫的 `package` ，你可以直接呼叫大寫開頭的 `func`

### func

定義 `function`

#### template

```go
func <(name struct_or_type)> functionName(<arg> <type>) <(auto_return_var type)> {

}
```

#### Lambda

```go
func() {
    // do something
}
```

#### 簡易版本

```go
func DoSomething() {
    fmt.Println("Do something")
}
```

#### 使用參數

```go
func DoSomething(str string) {
    fmt.Println("Do something", str)
}
```

#### 使用回傳

```go
func IsNil(obj interface{}) bool {
    return obj==nil
}
```

#### 使用自動回傳

```go
func NotZero(num int) (ret bool) {
    ret = num > 0 || num < 0
    return
}
```

#### 使用榜定

```go
type Obj struct {
	mock bool
	name string
}
func (obj *Obj) IsMock() bool {
	return obj.mock
}
```

### make

建立 `slice`、`map`、`chan`

#### slice

```go
s := make([]string, 3) //建立長度為 3 的 string slice
fmt.Println("emp:", s)
fmt.Println("len:", len(s))
fmt.Println("cap:", cap(s))
```

#### map

```go
m := make(map[int]string) // 建立用 int 為 key, 儲存 string 的 map
m[0] = "str"
fmt.Println(m)
```

#### chan

```go
c := make(chan int, 2) // 建立長度 2 的 chan
c <- 100 // 放個 100 進去
fmt.Println(c)
obj := <-c // 從 chan 拿出東西
fmt.Println(obj)
```

### map

一個用 key 拿值的東西

```go
m := make(map[string]string) // 建立用 string 為 key, 儲存 string 的 map
m["key"] = "val"
m["k"] = "v"
fmt.Println(m)
fmt.Println(m["k"])
m["k"] = "key"
fmt.Println(m)
val, key_in_map := m["k"]
fmt.Println(val, key_in_map)
val, key_in_map := m["m"]
fmt.Println(val, key_in_map)
```

### for

Golang 的 for 有幾個玩法

#### 無限迴圈

```go
for {

}
```

#### 變數宣告與使用

##### template

```go
for <var init>; <condition>; <after do> {

}
```

##### example

```go
for i:=0; i<10; i++ {
    fmt.Println("I have", i)
}

x := 20
for i:=0 ; x>10; i++ {
    x--
    fmt.Println("x: ", x, "i: ", i)
}
```
#### 迭代

```go
nums := []int{1, 2, 3, 4, 5, 6}
for index, num := range(nums){
  fmt.Println(index, num)
}
```
#### While

```go
sum := 10
for sum < 20 {
    sum++
}
fmt.Println(sum)
```

### slice

可以變更大小的，array

```go
s := []int{1, 2, 3} // 宣告裝著 [1, 2, 3] 的 slice
fmt.Println(s)
s := append(s, 100)
fmt.Println(s)
```

