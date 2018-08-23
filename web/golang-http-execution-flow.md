## 1. http包建立web服务器

```go
package main
import (
    "fmt"
    "log"
    "net/http"
    "strings"
)
func sayhelloName(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()
    fmt.Println(r.Form)
    fmt.Println("path", r.URL.Path)
    fmt.Println("scheme", r.URL.Scheme)
    fmt.Println(r.Form["url_long"])
    for k, v := range r.Form {
        fmt.Println("key:", k)
        fmt.Println("val:", strings.Join((v), ""))
    }
    fmt.Println(w, "hello world")
}
func main() {
    http.HandleFunc("/", sayhelloName)
    err := http.ListenAndServe(":9090", nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }
}
```

## 2. http包的运行机制

相关源码位于：/src/net/http/server.go

**服务端的几个概念**

- Request：用户请求的信息，用来解析用户的请求信息，包括post，get，Cookie，url等信息。
- Response:服务器需要反馈给客户端的信息。
- Conn：用户的每次请求链接。
- Handle:处理请求和生成返回信息的处理逻辑。

**Go实现web服务的流程**

1. 创建Listen Socket，监听指定的端口，等待客户端请求到来。
2. Listen Socket接受客户端的请求，得到Client Socket，接下来通过Client Socket与客户端通信。
3. 处理客户端请求，首先从Client Socket读取HTTP请求的协议头，如果是POST方法，还可能要读取客户端提交的数据，然后交给相应的handler处理请求，handler处理完，将数据通过Client Socket返回给客户端。

### 2.1. http包执行流程图

![image2017-3-5 22-46-35](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578729/article/golang/http/http-flow.png)

### 2.2. 注册路由[HandleFunc]

http.HandlerFunc类型默认实现了ServeHTTP的接口。

```go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler that calls f.
type HandlerFunc func(ResponseWriter, *Request)
// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

```go
// HandleFunc registers the handler function for the given pattern
// in the DefaultServeMux.
// The documentation for ServeMux explains how patterns are matched.
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    DefaultServeMux.HandleFunc(pattern, handler)
}
...
// HandleFunc registers the handler function for the given pattern.
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    mux.Handle(pattern, HandlerFunc(handler))
}
```



**Handle**

```go
// Handle registers the handler for the given pattern.
// If a handler already exists for pattern, Handle panics.
func (mux *ServeMux) Handle(pattern string, handler Handler) {
    mux.mu.Lock()
    defer mux.mu.Unlock()
 
    if pattern == "" {
        panic("http: invalid pattern " + pattern)
    }
    if handler == nil {
        panic("http: nil handler")
    }
    if mux.m[pattern].explicit {
        panic("http: multiple registrations for " + pattern)
    }
 
    mux.m[pattern] = muxEntry{explicit: true, h: handler, pattern: pattern}
 
    if pattern[0] != '/' {
        mux.hosts = true
    }
 
    // Helpful behavior:
    // If pattern is /tree/, insert an implicit permanent redirect for /tree.
    // It can be overridden by an explicit registration.
    n := len(pattern)
    if n > 0 && pattern[n-1] == '/' && !mux.m[pattern[0:n-1]].explicit {
        // If pattern contains a host name, strip it and use remaining
        // path for redirect.
        path := pattern
        if pattern[0] != '/' {
            // In pattern, at least the last character is a '/', so
            // strings.Index can't be -1.
            path = pattern[strings.Index(pattern, "/"):]
        }
        url := &url.URL{Path: path}
        mux.m[pattern[0:n-1]] = muxEntry{h: RedirectHandler(url.String(), StatusMovedPermanently), pattern: pattern}
    }
}
```

### 2.3. 如何监听端口

通过ListenAndServe来监听，底层实现：初始化一个server对象，调用net.Listen("tcp",addr)，也就是底层用TCP协议搭建了一个服务，监听设置的端口。然后调用srv.Serve(net.Listener)函数，这个函数处理接收客户端的请求信息。这个函数里起了一个for循环，通过Listener接收请求，创建conn，开一个goroutine，把请求的数据当作参数给conn去服务：go c.serve()，即每次请求都是在新的goroutine中去服务，利于高并发。

**src/net/http/server.go**

```go
// ListenAndServe always returns a non-nil error.
func ListenAndServe(addr string, handler Handler) error {
    server := &Server{Addr: addr, Handler: handler}
    return server.ListenAndServe()
}
...
// ListenAndServe listens on the TCP network address srv.Addr and then
// calls Serve to handle requests on incoming connections.
// Accepted connections are configured to enable TCP keep-alives.
// If srv.Addr is blank, ":http" is used.
// ListenAndServe always returns a non-nil error.
func (srv *Server) ListenAndServe() error {
    addr := srv.Addr
    if addr == "" {
        addr = ":http"
    }
    ln, err := net.Listen("tcp", addr)
    if err != nil {
        return err
    }
    return srv.Serve(tcpKeepAliveListener{ln.(*net.TCPListener)})
}
```

### 2.4. 如何接收客户端的请求

srv.Serve

```go
// Serve accepts incoming connections on the Listener l, creating a
// new service goroutine for each. The service goroutines read requests and
// then call srv.Handler to reply to them.
// Serve always returns a non-nil error.
func (srv *Server) Serve(l net.Listener) error {
    defer l.Close()
    if fn := testHookServerServe; fn != nil {
        fn(srv, l)
    }
    var tempDelay time.Duration // how long to sleep on accept failure
    if err := srv.setupHTTP2(); err != nil {
        return err
    }
    for {
        rw, e := l.Accept()
        if e != nil {
            if ne, ok := e.(net.Error); ok && ne.Temporary() {
                if tempDelay == 0 {
                    tempDelay = 5 * time.Millisecond
                } else {
                    tempDelay *= 2
                }
                if max := 1 * time.Second; tempDelay > max {
                    tempDelay = max
                }
                srv.logf("http: Accept error: %v; retrying in %v", e, tempDelay)
                time.Sleep(tempDelay)
                continue
            }
            return e
        }
        tempDelay = 0
        c := srv.newConn(rw)
        c.setState(c.rwc, StateNew) // before Serve can return
        go c.serve()
    }
}
```

关键代码：

```go
c := srv.newConn(rw)
c.setState(c.rwc, StateNew) // before Serve can return
go c.serve()
```

**newConn**

```go
// Create new connection from rwc.
func (srv *Server) newConn(rwc net.Conn) *conn {
    c := &conn{
        server: srv,
        rwc:    rwc,
    }
    if debugServerConnections {
        c.rwc = newLoggingConn("server", c.rwc)
    }
    return c
}
```

### 2.5. 如何分配handler

conn先解析request：c.readRequest()，获取相应的handler:handler:=c.server.Handler，即ListenAndServe的第二个参数，因为值为nil，所以默认handler=DefaultServeMux。该变量是一个路由器，用来匹配url跳转到其相应的handle函数。其中http.HandleFunc("/",sayhelloName)即注册了请求“/”的路由规则，当uri为“/”时，路由跳转到函数sayhelloName。DefaultServeMux会调用ServeHTTP方法，这个方法内部调用sayhelloName本身，最后写入response的信息反馈给客户端。

#### 2.5.1. c.serve()

```go
// Serve a new connection.
func (c *conn) serve() {
    ...
    for {
        w, err := c.readRequest()
        ...
        serverHandler{c.server}.ServeHTTP(w, w.req)
        ..
    }
}
```

#### 2.5.2. c.readRequest()

```go
// Read next request from connection.
func (c *conn) readRequest() (w *response, err error) {
    if c.hijacked() {
        return nil, ErrHijacked
    }
 
    if d := c.server.ReadTimeout; d != 0 {
        c.rwc.SetReadDeadline(time.Now().Add(d))
    }
    if d := c.server.WriteTimeout; d != 0 {
        defer func() {
            c.rwc.SetWriteDeadline(time.Now().Add(d))
        }()
    }
 
    c.r.setReadLimit(c.server.initialReadLimitSize())
    c.mu.Lock() // while using bufr
    if c.lastMethod == "POST" {
        // RFC 2616 section 4.1 tolerance for old buggy clients.
        peek, _ := c.bufr.Peek(4) // ReadRequest will get err below
        c.bufr.Discard(numLeadingCRorLF(peek))
    }
    req, err := readRequest(c.bufr, keepHostHeader)
    c.mu.Unlock()
    if err != nil {
        if c.r.hitReadLimit() {
            return nil, errTooLarge
        }
        return nil, err
    }
    c.lastMethod = req.Method
    c.r.setInfiniteReadLimit()
 
    hosts, haveHost := req.Header["Host"]
    if req.ProtoAtLeast(1, 1) && (!haveHost || len(hosts) == 0) {
        return nil, badRequestError("missing required Host header")
    }
    if len(hosts) > 1 {
        return nil, badRequestError("too many Host headers")
    }
    if len(hosts) == 1 && !validHostHeader(hosts[0]) {
        return nil, badRequestError("malformed Host header")
    }
    for k, vv := range req.Header {
        if !validHeaderName(k) {
            return nil, badRequestError("invalid header name")
        }
        for _, v := range vv {
            if !validHeaderValue(v) {
                return nil, badRequestError("invalid header value")
            }
        }
    }
    delete(req.Header, "Host")
 
    req.RemoteAddr = c.remoteAddr
    req.TLS = c.tlsState
    if body, ok := req.Body.(*body); ok {
        body.doEarlyClose = true
    }
 
    w = &response{
        conn:          c,
        req:           req,
        reqBody:       req.Body,
        handlerHeader: make(Header),
        contentLength: -1,
    }
    w.cw.res = w
    w.w = newBufioWriterSize(&w.cw, bufferBeforeChunkingSize)
    return w, nil
}
```

#### 2.5.3. ServeHTTP(w, w.req)

```go
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
    handler := sh.srv.Handler
    if handler == nil {
        handler = DefaultServeMux
    }
    if req.RequestURI == "*" && req.Method == "OPTIONS" {
        handler = globalOptionsHandler{}
    }
    handler.ServeHTTP(rw, req)
}
```

#### 2.5.4. DefaultServeMux

```go
type ServeMux struct {
    mu    sync.RWMutex
    m     map[string]muxEntry
    hosts bool // whether any patterns contain hostnames
}
type muxEntry struct {
    explicit bool
    h        Handler
    pattern  string
}
// NewServeMux allocates and returns a new ServeMux.
func NewServeMux() *ServeMux { return &ServeMux{m: make(map[string]muxEntry)} }
// DefaultServeMux is the default ServeMux used by Serve.
var DefaultServeMux = NewServeMux()
```

**handler接口的定义**

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

#### 2.5.5. ServeMux.ServeHTTP

```go
// ServeHTTP dispatches the request to the handler whose
// pattern most closely matches the request URL.
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
    if r.RequestURI == "*" {
        if r.ProtoAtLeast(1, 1) {
            w.Header().Set("Connection", "close")
        }
        w.WriteHeader(StatusBadRequest)
        return
    }
    h, _ := mux.Handler(r)
    h.ServeHTTP(w, r)
}
```

**mux.Handler(r)**

```go
// Handler returns the handler to use for the given request,
// consulting r.Method, r.Host, and r.URL.Path. It always returns
// a non-nil handler. If the path is not in its canonical form, the
// handler will be an internally-generated handler that redirects
// to the canonical path.
//
// Handler also returns the registered pattern that matches the
// request or, in the case of internally-generated redirects,
// the pattern that will match after following the redirect.
//
// If there is no registered handler that applies to the request,
// Handler returns a ``page not found'' handler and an empty pattern.
func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string) {
    if r.Method != "CONNECT" {
        if p := cleanPath(r.URL.Path); p != r.URL.Path {
            _, pattern = mux.handler(r.Host, p)
            url := *r.URL
            url.Path = p
            return RedirectHandler(url.String(), StatusMovedPermanently), pattern
        }
    }
 
    return mux.handler(r.Host, r.URL.Path)
}
 
// handler is the main implementation of Handler.
// The path is known to be in canonical form, except for CONNECT methods.
func (mux *ServeMux) handler(host, path string) (h Handler, pattern string) {
    mux.mu.RLock()
    defer mux.mu.RUnlock()
 
    // Host-specific pattern takes precedence over generic ones
    if mux.hosts {
        h, pattern = mux.match(host + path)
    }
    if h == nil {
        h, pattern = mux.match(path)
    }
    if h == nil {
        h, pattern = NotFoundHandler(), ""
    }
    return
}
```

### 2.6. http连接处理流程图

![image2017-3-5 23-50-6](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578730/article/golang/http/http-connect-flow.png)

## 3. http的执行流程总结

1、首先调用Http.HandleFunc，按如下顺序执行：

1. 调用了DefaultServerMux的HandleFunc。
2. 调用了DefaultServerMux的Handle。
3. 往DefaultServerMux的map[string] muxEntry中增加对应的handler和路由规则。

2、调用http.ListenAndServe(":9090",nil)，按如下顺序执行：

1. 实例化Server。
2. 调用Server的ListenAndServe()。
3. 调用net.Listen("tcp",addr)监听端口。
4. 启动一个for循环，在循环体中Accept请求。
5. 对每个请求实例化一个Conn，并且开启一个goroutine为这个请求进行服务go c.serve()。
6. 读取每个请求的内容w,err:=c.readRequest()。
7. 判断handler是否为空，如果没有设置handler，handler默认设置为DefaultServeMux。
8. 调用handler的ServeHttp。
9. 根据request选择handler，并且进入到这个handler的ServeHTTP,
   mux.handler(r).ServeHTTP(w,r)
10. 选择handler

- 判断是否有路由能满足这个request（循环遍历ServeMux的muxEntry）。
- 如果有路由满足，调用这个路由handler的ServeHttp。
- 如果没有路由满足，调用NotFoundHandler的ServeHttp。

## 4. 自定义路由

Go支持外部实现路由器，ListenAndServe的第二个参数就是配置外部路由器，它是一个Handler接口。即外部路由器实现Hanlder接口。

Handler接口：

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

自定义路由

```go
package main
import (
    "fmt"
    "net/http"
)
type MyMux struct{
}
func (p *MyMux) ServeHTTP(w http.ResponseWriter,r *http.Request){
    if r.URL.Path=="/"{
        sayhelloName(w,r)
        return
    }
    http.NotFound(w,r)
    return
}
func sayhelloName(w http.ResponseWriter,r *http.Request){
    fmt.Fprintln(w,"Hello myroute")
}
func main() {
    mux:=&MyMux{}
    http.ListenAndServe(":9090",mux)
     
}
```

文章参考：

《Go web编程》
