# Context标准库

```go
type Context interface {
    // Deadline返回的时间是代表该上下文所做的工作应该被取消的时间。
    // 如果没有设置截止日期，则返回ok==false。连续调用Deadline会返回相同的结果。
    Deadline() (deadline time.Time, ok bool)
    
    // Done返回一个channel通道，该通道代表完成工作时关闭取消上下文。
    // 需要在select-case语句中使用，case <- context.Done()。
    // 如果上下文未关闭，Done返回nil。
    // 当context关闭后，Done返回一个被关闭的通道，关闭仍然是可读的，goroutine可以接收到关闭请求。
    // 连续调用Done将返回相同的值。Done通道的关闭可能会异步发生，当cancel函数返回。
    Done() <-chan struct{}
    
    // 该方法描述context关闭的原因
    // 如果Done未关闭，Err返回nil
    // 如果Done被关闭，Err返回一个非nil错误
    Err() error
    
    // 该方法根据key值查询map中value
    // Value returns the value associated with this context for key, 
    // or nil if no value is associated with key. 
    // Successive calls to Value with the same key returns the same result.
    Value(key any) any
}
```

根context

```go
func Background() Context
func TODO() Context
```

派生context函数

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
func WithValue(parent Context, key, val any) Context
```

## 到期取消

```go
func main()  {
    ctx,cancel := context.WithTimeout(context.Background(),10 * time.Second)
    defer cancel()
    go Monitor(ctx)

    time.Sleep(20 * time.Second)
}

func Monitor(ctx context.Context)  {
    select {
    case <- ctx.Done():
        fmt.Println(ctx.Err())
    case <-time.After(20*time.Second):
        fmt.Println("stop monitor")
    }
}
```

## 携带数据

```go
type key string

func main()  {
    ctx := context.WithValue(context.Background(),key("lineshen"),"tencent")
    Get(ctx,key("lineshen"))
    Get(ctx,key("line"))
}

func Get(ctx context.Context,k key)  {
    if v, ok := ctx.Value(k).(string); ok {
        fmt.Println(v)
    }
}
```

