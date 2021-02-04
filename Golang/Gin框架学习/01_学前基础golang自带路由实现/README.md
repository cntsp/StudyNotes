# http.ServeMux 解析

golang自带的 `http.SeverMux路由实现`简单，本质是一个map[string]Handler，是请求路径与该路径对应的处理函数的映射关系。实现简单功能也比较单一：

1. 不支持正则路由，这个是比较致命的
2. 只支持路径匹配，不支持按照Method, header, host等信息匹配，所以也就没有办法实现RESTful架构



## Web Server 概述

使用go语言搭建一个web服务器是很简单的，几行代码就可以搭建一个稳定的高并发的web server。

```go
import (
	"net/http"
)
// hello world, the web server
func HelloServer(w http.ResponseWriter, req *http.Request) {
    io.WriterString(w, "hello, world!\n")
}

func main() {
    http.HandleFunc("/hello/", HelloServer)
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }
}
```

一个 go web 服务器正常运行起来大概需要以下几个步骤：

1. 创建 listen socket， 循环监听 listen socket
2. accept 接受新的链接请求， 并创建网络连接conn，然后开启一个 goroutine 负责处理该链接。
3. 从该链接读取请求参数构造出 http.Request 对象， 然后根据请求路径在路由表中查找，找到对应的上层应用的处理函数，把请求交给应用处理函数
4. 应用处理函数根据请求的参数等信息做处理，返回不同的信息给用户
5. 应用层处理完该链接请求后关闭该链接（正常流程，如果是 keepalive 长链接则不关闭该链接）

这里路由表是比较重要的，我们具体分析下 http.Server 是如何做路由的

路由表实际上是一个map

key 是路径 ==> "/hello"

value 是该路径所对应的处理函数  ==》 HelloServer



## 路由表结构

go语言默认的路由表是 ServeMux， 结构如下：

```go
type ServeMux struct {
    	mu	sync.RWMutex
    	m   map[string]muxEntry   // 存放具体的路由信息
}

type muxEntry struct {
    explicit 	bool
    h 			Handler
    pattern		string
}

// muxEntry.Handler 是一个接口
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}

// 这边可能会有点疑惑
// http.HandleFunc("/hello/", HelloServer)
// helloServer是一个function啊，并没有实现ServerHTTP接口啊
// 这是因为虽然我们传入的是一个function， 但是HandleFunc 会把function转为实现了ServeHTTP 接口的一个新类型 HandlerFunc

/*
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	mux.Handle(pattern, HandlerFunc(handler))
}

type HandlerFunc func(ResponseWriter, *Request)
// ServeHTTP calls f(w, r)
func (f HandlerFunc) ServerHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
*/
```



## 路由注册过程

注册过程其实就是往map中插入数据，值得注意的一个地方就是如果注册路由路径是 `/hello/` 并没有 `/hello`的路由信息，那么会在路由表中自动增加一条 `/hello`的路由，`/hello`的处理函数是重定向到`/hello/`。但是如果注册的时候是`/hello`是不会自动添加`/hello/`的路由的

```go
// Handle registers the handler for the given pattern.
// If a handler already exists for pattern, Handle panics.
func (mux *ServeMux) Handle(pattern string, handler Handler) {
    mux.mu.Lock()
    defer mux.mu.Unlock()
    mux.m[pattern] = muxEntry{explicit: true, h: handler, pattern: pattern}
    
    n := len(pattern)
    if n > 0 && pattern[n-1] == '/' && !mux.m[pattern[0:n-1]].explicit {
        path := pattern
        fmt.Printf("redirect for :%s to :%s", pattern, path)
        mux.m[pattern[0:n-1]] = muxEntry{h: RedirectHandler(path, StatusMovedPermanently), pattern: pattern}
    }
}
```

## 路由查找过程

路由查找过程就是遍历路由表，找到最长匹配请求路径的路由信息并返回，如果找不到返回  NotFoundHandler

```go
func (mux *ServeMux) handler(host, path string) (h Handler, pattern string) {
    mux.mu.RLock()
    defer mux.mu.RUnlock()
    if h == nil {
        h, pattern = mux.match(path)
    }
    if h == nil {
        h, pattern = NotFoundHandler(), ""
    }
    return
}

func (mux *ServeMux) match(path string) (h Handler, pattern string) {
    var n = 0
    for k, v := range mux.m {
        if !pathMatch(k, path) {
            continue
        }
        // 找出匹配度最长的
        if h == nil || len(k) > n {
            n = len(k)
            n = v.h
            pattern = v.pattern
        }
    }
    return
}

// 如果路由表中的路径是不以下划线结尾的 /hello
// 那么只有请求路径为 /hello 完全匹配时才符合
// 如果路由表中的注册路径是以下划线结尾的 /hello/
// 那么请求路径只要满足 /hello/* 就符合该路由
func pathMatch(pattern, path string) bool {
    n := len(pattern)
    if pattern[n-1] != '/' {
        return pattern == path
    }
    return len(path) >= n && path[0:n] == pattern
}
```





