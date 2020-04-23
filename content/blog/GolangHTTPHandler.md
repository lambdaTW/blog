+++
title = "GolangHTTPHandler"
author = "lambda@lambda.tw"
categories = ["Golang"]
tags = ["Golang", "HTTP"]
date = "2018-07-21"
description = ""
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
+++
# 簡易復刻出的 Golang HTTP HandleFunc

身為一個 web 狗，用新語言寫個 router 也是應該的，Golang 本身在寫 HTTP 服務就有極大的優勢，官方自帶的 library 就很好用了，以至於到目前為止的統計大部分的人還是直接使用原生的 library 而非使用框架，但是 router 這部份就統計看來已經有了大方向， [Mux](https://github.com/gorilla/mux) 是目前大多數人使用的 router 框架，這邊我們玩一下 Golang 原生的 handler 讓它可以和原生的 HandleFunc 有一樣的感覺

## 第一步：寫一個簡單的 HTTP server

相信大家都不會。。。當然就是要 google

關鍵字 ： golang http

第一篇就會看到官方的 library 的[連結](https://golang.org/pkg/net/http/)囉！

### 開一個新專案

```shell
cd $GOPATH/src/
mkdir router
touch main.go
```

### 依照官方網站的提示寫出一個簡單的 HTTP 服務

```go
package main

import (
	"fmt"
	"net/http"
	"html"
	"log"
)

func main() {
	http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
	})

	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### 瀏覽看看

```shell
go run main.go
firefox 127.0.0.1:8080/bar
```

## 所以

我們可以知道 Golang 本身其實就可以做簡單的 router 讓對應的 URL 可以去執行你的 function，但是如果你想自己搞呢？

## 資料

Router 最重要的資訊其實就是 Domain 後面的 URI 或稱作 Path，所以我就用這兩個關鍵字直接打在 接收 `*http.Request` 的 function 裡面，發現 Path 沒有反應，但是 URI 讓 VScode 給了提示，那另一個 Path 就如同官方的教學，可以用 `r.URL.Path` 取得。

```go
func main() {
	http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
        r.RequestURI
	})
```

## Handler

官方的程式碼中有一段我沒有貼上來，原因就是貼上去會壞掉，那我們要怎麼自己來寫這所謂的 handler 呢？

```go
fooHandler := "" // 至少要宣告吧
http.Handle("/foo", fooHandler)
```

看看錯誤碼吧

```
cannot use foohandler (type string) as type http.Handler in argument to http.Handle:
	string does not implement http.Handler (missing ServeHTTP method)

```

看起來是少了 ServeHTTP 這個方法，所以我們需要讓 fooHandler 有這個 Method 才能跑，但是我們也不知道他要有啥才好，所以回到官方看看找到 [ServeHTTP](https://golang.org/pkg/net/http/#HandlerFunc.ServeHTTP)，我們就照著做一個空的 Method 試試看能不能跑。

```go
type myhandler struct {
}

func (handle *myhandler) ServeHTTP(w http.ResponseWriter, r *http.Request){

}
```

跑一下

```shell
go run main.go
firefox 127.0.0.1:8080/foo
```

可以看到是空的，但是不會壞掉，那就試著在這個 function 裡面加點東西吧

```go
fmt.Fprint(w, "r: "+r.Method+r.URL.Path)
```

重跑一次看看

```shell
go run main.go
firefox 127.0.0.1:8080/foo
```

可以看到 firefox 裡面有文字了

```text
r: GET/foo
```

## 來個對應表吧

### 對應表

為了要簡易的復刻 HandleFunc 我們就用個簡單的 map 來儲存對應的 URL Path 到對應的 function 

```go
type myhandler struct {
	route map[string]func(http.ResponseWriter, *http.Request)
}
```

以上我們定義了一個 struct 來當作我們的 handler 讓它有一個 route 的屬性，之後我們就可以讓使用者透過它來讓進來的 Request 跑去我們要的 function 裡面

### 註冊

有了表，我們要讓別人可以填表，所以我們在 `myhandler` 下面實做一個註冊用的 function

```go
func (handle *myhandler) Register(uri string, f func(http.ResponseWriter, *http.Request)) {
    // 如果還沒初始化，幫它初始化
	if handle.route == nil {
		handle.route = make(map[string]func(http.ResponseWriter, *http.Request))

	}
	handle.route[uri] = f
}
```

### ServeHTTP

最後就是要處理進來的 Request 啦！我們只要確定近來的 URL Path 有在我們的表內，我們就可以去呼叫存在 route 內所對應的 function

```go
func (handle *myhandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	fmt.Println("r: ", r.Method, r.RequestURI) // 在 terminal 可以看到 log
	if _, ok := handle.route[r.RequestURI]; ok {
		handle.route[r.RequestURI](w, r)
	}
}
```

### main

最後就來玩玩我們寫好的 handler 吧！

```go
func main() {
	handler := new(myhandler)
	handler.Register("/ping", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprint(w, "pong")
	})
	s := &http.Server{
		Addr:    ":8080",
		Handler: handler,
	}
	log.Fatal(s.ListenAndServe())
}
```

### Try it!

```shell
go run main.go
firefox 127.0.0.1:8080/ping
```

## 結論

程式就是這樣有了資料就可以很多事情，但是一切一定都不是我們想像的那麼簡單，看看 Mux 上的功能，想想看我們要怎麼做才能完成這麼多功能呢？

如果要實做出 Middle ware 讓別人可以加，你覺得怎麼改比較好呢？

## 最後得程式碼

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

type myhandler struct {
	route map[string]func(http.ResponseWriter, *http.Request)
}

func (handle *myhandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	fmt.Println("r: ", r.Method, r.RequestURI)
	if _, ok := handle.route[r.RequestURI]; ok {
		handle.route[r.RequestURI](w, r)
	}
}

func (handle *myhandler) Register(uri string, f func(http.ResponseWriter, *http.Request)) {
	if handle.route == nil {
		handle.route = make(map[string]func(http.ResponseWriter, *http.Request))

	}
	handle.route[uri] = f
}

func main() {
	handler := new(myhandler)
	handler.Register("/ping", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprint(w, "pong")
	})
	s := &http.Server{
		Addr:    ":8080",
		Handler: handler,
	}
	log.Fatal(s.ListenAndServe())
}
```

