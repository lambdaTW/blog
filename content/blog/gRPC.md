+++
title = "gRPC"
author = "lambda@lambda.tw"
categories = ["golang", "gRPC"]
tags = ["golang", "gRPC"]
date = "2018-11-18"
description = ""
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
+++
# 實做一個可以寄信的 gRPC
## Proto
### 寫一個寄發信件服務所需要的資料格式
Proto 是一個文件用來儲存 gRPC server 與 client 交換資料時鎖需要的資料格式，建議可以看看它與 JSON 的[對照表](https://developers.google.com/protocol-buffers/docs/proto3#json)來迅速了解需要怎樣寫
```proto3
syntax = "proto3"; // use proto version 3

package pb; // package name

/*
Add the Send function for use
*/
service Mail{
    rpc Send (MailRequest) returns (MailStatus) {}
}

/*
Declare what data you need to let server know
and server will use it to send a mail
*/
message MailRequest{
    string from = 1;
    repeated string to = 2;
    repeated string cc = 3;
    string subject = 4;
    string body = 5;
    string type = 6;
}

/*
Means what the mail status
be send or not
*/
message MailStatus{
    int32 status = 1;
    string code = 2;
}
```
### 產生 golang 的程式
```shell-script
go get -u github.com/golang/protobuf/protoc-gen-go
protoc --go_out=plugins=grpc:. *.proto
```
### 觀察產生出來的檔案
可以看到 MailRequest 直接幫你轉換成 golang 的 struct，還多了一些奇怪的東西，但是我們只要知道以後不管是 client 還是 server 都可以用這一些定義好的 `protocol` 來 import 來使用
```go
type MailRequest struct {
	From                 string   `protobuf:"bytes,1,opt,name=from,proto3" json:"from,omitempty"`
	To                   []string `protobuf:"bytes,2,rep,name=to,proto3" json:"to,omitempty"`
	Cc                   []string `protobuf:"bytes,3,rep,name=cc,proto3" json:"cc,omitempty"`
	Subject              string   `protobuf:"bytes,4,opt,name=subject,proto3" json:"subject,omitempty"`
	Body                 string   `protobuf:"bytes,5,opt,name=body,proto3" json:"body,omitempty"`
	Type                 string   `protobuf:"bytes,6,opt,name=type,proto3" json:"type,omitempty"`
	XXX_NoUnkeyedLiteral struct{} `json:"-"`
	XXX_unrecognized     []byte   `json:"-"`
	XXX_sizecache        int32    `json:"-"`
}
```
## Server
這邊我們就可以`實做`一個可以寄信的服務 (此處使用 gomail 套件))
```go
package main

import (
	"log"
	"net"
	"os"
	"time"

	"golang.org/x/net/context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/reflection"

	"myMail/pb"

	gomail "gopkg.in/gomail.v2"
)

type server struct{}

var ch = make(chan *gomail.Message)

/*
Send is a simple function for send email
*/
func (s *server) Send(ctx context.Context, mail *pb.MailRequest) (*pb.MailStatus, error) {
	m := gomail.NewMessage()
	m.SetHeader("From", mail.From)
	m.SetHeader("To", mail.To...)
	m.SetHeader("Subject", mail.Subject)
	m.SetBody(mail.Type, mail.Body)
	ch <- m
	return &pb.MailStatus{Status: int32(0), Code: ""}, nil
}

func main() {
    // 監聽 50051 port
	lis, err := net.Listen("tcp", ":50051")
	if err != nil {
		log.Fatalf("無法監聽該埠口：%v", err)
	}
	s := grpc.NewServer()
	pb.RegisterMailServer(s, &server{})
	reflection.Register(s)
	go func() {
		d := gomail.NewDialer("smtp.gmail.com", 587, os.Getenv("GMAIL_ACC"), os.Getenv("GMAIL_PASS"))

		var s gomail.SendCloser
		var err error
		open := false
		for {
			select {
			case m, ok := <-ch:
				if !ok {
					return
				}
				if !open {
					if s, err = d.Dial(); err != nil {
						panic(err)
					}
					open = true
				}
				if err := gomail.Send(s, m); err != nil {
					log.Print(err)
				}
			// Close the connection to the SMTP server if no email was sent in
			// the last 30 seconds.
			case <-time.After(30 * time.Second):
				if open {
					if err := s.Close(); err != nil {
						panic(err)
					}
					open = false
				}
			}
		}
	}()
	if err := s.Serve(lis); err != nil {
		log.Fatalf("無法提供服務：%v", err)
		close(ch)
	}
}
```
## Client
我們希望 client 模擬一般會用到的 http 服務，但是我們不寫邏輯在裡面，就瀏覽就呼叫 server 寄信了
```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"
	"net/http"

	"myMail/pb"

	"google.golang.org/grpc"
)

func main() {
	// 連線到遠端 gRPC 伺服器。
	conn, err := grpc.Dial("server:50051", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("連線失敗：%v", err)
	}
	defer conn.Close()

	// 建立新的 Mail 客戶端，所以等一下就能夠使用 Mail 的所有方法。
	c := pb.NewMailClient(conn)

	// 傳送新請求到遠端 gRPC 伺服器 Mail 中，並呼叫 Send 函式
	mr := pb.MailRequest{
		From:    os.Getenv("MAIL_FROM"),
		To:      []string{"abc@example.com"},
		Cc:      []string{},
		Subject: "How to use gRPC",
		Body:    "Just done",
		Type:    "text/html",
	}
	http.HandleFunc("/send", func(w http.ResponseWriter, r *http.Request) {
		ret, err := c.Send(context.Background(), &mr)
		if err != nil {
			log.Fatalf("無法執行 Send 函式：%v", err)
		} else {
			fmt.Fprintf(w, "Send %s", ret.Code)
		}
	})

	log.Fatal(http.ListenAndServe(":8080", nil))
}
```
## 中場休息
看看我們目錄們現在長怎樣
```shell-script
tree myMail
.
├── client
│   └── main.go
├── pb
│   ├── mail.pb.go
│   └── mail.proto
└── server
    └── main.go
```
我們可能有幾種管理方式

- 可以看出來我們現在其實可以拆開成為三個 git repo
一個是 client 一個是 server 一個是 pb 由團隊們共同協同修改 pb 由個人或團隊維護單一個或多個 client(可能是某商業應用)，再由個人或一個團隊維護 server (實做單純的 mail service)，此時 pb 的修改將會關忽到所有人、client 的修改不會動到 mail serice，此時 mail service 團隊如果想要修改訊息格式必須要交給 pb 團隊去實現或是交付 PR 給 pb 團隊
- 但是如果把 mail service 的 pb 單獨綁到 server 這個專案，使得最後只有 client(1~*) & server(1) 個 repo 將會發生以下事情，client 只需要安裝 pb 但是卻要把整個 service 下載下來，就算 golang 不會幫你 compiled 沒用到的東西，但是在 CI/CD 時還是會去下載那些用不到的玩意兒，在第一次使用以及後來 service 有更新時都會影響整個部屬時間

故在此我覺得第一個方案比較妥當，也比較符合 micro service 的感覺

## Dockerize
其實就是包成 Docker
### Dockerfile
這裡很懶惰的接受參數後 build `client` or `server`
```Dockerfile
cat Dockerfile
FROM golang AS build-env
ARG BUILD_PATH
ADD . /go/src/myMail
RUN cd /go/src/myMail/$BUILD_PATH && go get && GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o app

FROM alpine
ARG BUILD_PATH
ARG EXPOSE_PROT
WORKDIR /app
COPY --from=build-env /go/src/myMail/$BUILD_PATH/app /app/
RUN apk update && apk add ca-certificates && rm -rf /var/cache/apk/*
EXPOSE ${EXPOSE_PROT}
# 50051, 8080
```
### Build
```shell-script
docker build \
--build-arg BUILD_PATH=server \
--build-arg EXPOSE_PROT=50051 \
-t mailserver .
```
```shell-script
docker build \
--build-arg BUILD_PATH=client \
--build-arg EXPOSE_PROT=8080 \
-t mailclient .
```
## K8s
拆拆拆，拆成 micro service 以後部屬變成麻煩，資料傳遞的網路也變成麻煩，K8s 可能可以幫我們少點這種麻煩，但是還不夠，這裡只的範例只其實架構還是爛爛的，少了很多微服務必要的元件，如： message queue, circuit breaker, API route...
```kubernetes
kind: Service
apiVersion: v1
metadata:
  name:  server
spec:
  selector:
    app:  server
  ports:
  - port:  50051
    protocol: TCP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: server-depolyment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: server
  template:
    metadata:
      labels:
        app: server
    spec:
      containers:
      - name: server
        image: mailserver
        ports:
        - containerPort: 50051
        env:
	- name: GMAIL_ACC
          valueFrom:
            secretKeyRef:
              name: gmail-acc
              key: env
        - name: GMAIL_PASS
          valueFrom:
            secretKeyRef:
              name: gmail-pass
              key: env
```
```kubernetes
kind: Service
apiVersion: v1
metadata:
  name:  client
spec:
  selector:
    app:  client
  type:  NodePort
  ports:
  - port:  8080
    nodePort: 30290
    protocol: TCP
    targetPort:  8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client-depolyment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: client
  template:
    metadata:
      labels:
        app: client
    spec:
      containers:
      - name: client
        image: mailclient
        ports:
        - containerPort: 8080
        env:
	- name: MAIL_FROM
          valueFrom:
            secretKeyRef:
              name: gmail-acc
              key: env
```
### Try it
```shell-script
kubectl -f server.yaml
kubectl -f client.yaml
```
