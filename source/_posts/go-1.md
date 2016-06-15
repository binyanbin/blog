---
title: golang是不是后端开发神器
date: 2016-5-8
desc: golang
---
周末有人介绍golang很合适后端开发，随即把之前C实现的静态资源下载功能(读取服务器上的静态资源文件内容返回给客户端)改成golang实现。然后对比一下效果。

## 实现对比
同样使用标准库,golang实现代码+注释只需要123行,C实现代码+注释412行，golang代码量只有C的1/3。

## 测试对比
我使用的是jmeter测试，首先测试c实现的服务,分别用100,200,300线程并发，每线程访问10。测试数据如下:
![c](/img/go-1.png)

![c](/img/go-2.png)

![c](/img/go-3.png)
C版实现越测数据越看不下去了，出错率越来越大。接着我测golang实现，我分别用300线程，1000线程和5000线程并发，每线程访问10次。测试数据如下:
![c](/img/go-4.png)

![c](/img/go-5.png)

![c](/img/go-6.png)
除了响应时间延长，其它都比较稳定，出错率全为0.难道这是传说中的后端神器，难道老板再也不用担心网站访问人数太多当机了，难道我也转golang开发。
<!-- more -->

## golang实现

``` go
package main

import (
	"flag"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"os"
	"os/exec"
	"path/filepath"
	"strings"
)

var realPath string //当前程序运行目录

/*
www路由，请求处理 w:response  r:request
*/
func staticResource(w http.ResponseWriter, r *http.Request) {
	fmt.Println("url:" + r.RequestURI)
	if r.Method == "get" || r.Method == "GET" {
		disposeGet(w, r)
	} else if r.Method == "post" || r.Method == "POST" {
		disposePost(w, r)
	} else {
		w.WriteHeader(501)
		w.Write([]byte("<HTML><HEAD><TITLE>Method Not Implemented</TITLE></HEAD><BODY><P>HTTP request method not supported.</BODY></HTML>"))
	}
}

/*
取字串 s:字符串 l:取到多少位
*/
func substr(s string, l int) string {
	if len(s) <= l {
		return s
	}
	ss, sl, rl, rs := "", 0, 0, []rune(s)
	for _, r := range rs {
		rint := int(r)
		if rint < 128 {
			rl = 1
		} else {
			rl = 2
		}
		if sl+rl > l {
			break
		}
		sl += rl
		ss += string(r)
	}
	return ss
}

/*
get获取静态资源 w: response r: request
*/
func disposeGet(w http.ResponseWriter, r *http.Request) {
	request_type := r.URL.Path[strings.LastIndex(r.URL.Path, "."):]
	switch request_type { //文件类型判断
	case ".html":
		w.Header().Set("content-type", "text/html")
	case ".htm":
		w.Header().Set("content-type", "text/html")
	case ".css":
		w.Header().Set("content-type", "text/css")
	case ".js":
		w.Header().Set("content-type", "text/javascript")
	case ".png":
		w.Header().Set("content-type", "image/png")
	case ".gif":
		w.Header().Set("content-type", "image/gif")
	case ".jpg":
		w.Header().Set("content-type", "image/jpeg")
	default:
	}
	fin, err := os.Open(realPath + r.URL.Path)
	defer fin.Close()
	if err != nil { //未找到静态资源文件
		if request_type == ".html" || request_type == ".htm" { //未找到html文件
			w.WriteHeader(404)
			fmt.Println("static resource:", err)
			w.Write([]byte("<HTML><TITLE>Not Found</TITLE><BODY><P>The server could not fulfill.your request be.is unavailable or nonexistent.</BODY></HTML>"))
		}
	}
	fd, _ := ioutil.ReadAll(fin)
	w.Write(fd)
}

/*
post上传静态资源 w: response r: request
*/
func disposePost(w http.ResponseWriter, r *http.Request) {
	//未实现
}

/*
程序入口
*/
func main() {
	//获取程序当前路径
	file, _ := exec.LookPath(os.Args[0])
	path, _ := filepath.Abs(file)
	//命令行参数处理
	arg_num := len(os.Args)
	if arg_num == 2 {
		port := os.Args[1] //第一个参数为端口号
		fmt.Println("http server start port:" + port)
		//取前当前目录
		realPath = substr(path, strings.LastIndex(path, "\\"))
		flag.Parse()
		//请求处理
		http.HandleFunc("/www/", staticResource)
		err := http.ListenAndServe(":"+port, nil)
		if err != nil {
			log.Fatal("服务器出错:", err)
		}
	} else {
		fmt.Println("参数错误，程序退出")
	}
}

```


