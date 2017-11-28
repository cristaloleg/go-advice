# Go-adivces 中文版本 #

### 代码 ###

- [ ] 使用 `go fmt` / `gofmt` 格式化你的代码, 让每个人都更开心
- [ ] 多个 if 语句可以折叠成 switch
- [ ] 用 `chan struct{}` 来传递信号

    - `chan bool` 表示的不够清楚, 但是 `struct{}` 会更好

- [ ] `30 * time.Second` 比 `time.Duration(30) * time.Second` 更好
- [ ] 总是把 for-select 换成一个函数
- [ ] 分组定义 `const` 类型声明和 `var` 逻辑类型声明
- [ ] 每个阻塞或者 IO 函数操作应该是可取消的或者至少是可超时的
- [ ] 为整型常量值实现 `Stringer` 接口
- [ ] 用 defer 来检查你的错误
  
```go
defer func() {
    err := ocp.Close()
    if err != nil {
        rerr = err
    }
}()
```

- [ ] 任何 panic 都不要使用 `checkErr` 函数或者用 `os.Exit`
- [ ] 不要给枚举使用别名，因为这打破了类型安全

    - https://play.golang.org/p/MGbeDwtXN3
```go
package main
type Status = int
type Format = int // remove `=` to have type safety

const A Status = 1
const B Format = 1

func main() {
println(A == B)
}
```

- [ ] 如果你想省略返回参数，你最好明确的做

    - ` _ = f()` 比 `f()` 更好

- [ ] 我们 slice 初始化的更短形式是 `a := []T{}`
- [ ] 用 range 循环来进行数组或 slice 的迭代

    -  `for _, c := range a[3:7] {...}` 比 `for i := 3; i < 7; i++ {...}` 更好

- [ ] 多行字符串用反引号(\`)

### 持续集成 ###

- [ ] run go format on CI and compare diff
  
    - 这将确保一切都是生成的和承诺的

- [ ] 用最新的 Go 运行 Travis-CI，用 `travis 1`
  
    - 了解更多：https://github.com/travis-ci/travis-build/blob/master/public/version-aliases/go.json

- [ ] 检查代码格式是否有错误 `diff -u <(echo -n) <(gofmt -d .)`

### 并发 ###

- [ ] 以线程安全的方式创建一些东西的最好选择是 `sync.Once`

    - 不要用 flags, mutexes, channels or atomics

- [ ] 永远不要使用 `select{}`, 省略通道， 等待信号

### 性能 ###

- [ ] 不要省略 `defer`

    - 在大多数情况下 200ns 加速可以忽略不计

- [ ] 总是关闭 http body `defer r.Body.Close()`
  
    - 除非你需要泄露 goroutine

- [ ] 过滤不分配
  
```go
  b := a[:0]
  for _, x := range a {
  	if f(x) {
	    b = append(b, x)
  	}
  }
```

- [ ] `time.Time` 有指针字段 `time.Location` 并且这对go GC不好

    - 只在大量的`time.Time`才有意义，用 timestamp 代替

- [ ] `regexp.MustCompile` 比 `regexp.Compile` 更好

    - 在大多数情况下，你的正则表达式是不可变的，所以你最好在 `func init` 中初始化它

- [ ] 请勿在你的热路径中过度使用 `fmt.Sprintf`. 由于维护接口的缓冲池和动态调度，它是很昂贵的。

    - 如果你正在使用 `fmt.Sprintf("%s%s", var1, var2)`, 考虑使用简单的字符串连接。
    - 如果你正在使用 `fmt.Sprintf("%x", var)`, 考虑使用 `hex.EncodeToString` or `strconv.FormatInt(var, 16)`

- [ ] 如果你不需要用它，可以考虑丢弃它，例如`io.Copy(ioutil.Discard, resp.Body)`

    - HTTP 客户端的传输不会重用连接，直到body被读完和关闭。
```go
  res, _ := client.Do(req)
  io.Copy(ioutil.Discard, res.Body)
  defer res.Body.Close()
```

- [ ] 不要在循环中使用 defer，否则会导致内存泄露

    - 'cause defers will grow your stack without the reason

- [ ] 不要忘记停止 ticker, 除非你需要泄露 channel
  
```go
ticker := time.NewTicker(1 * time.Second)
defer ticker.Stop()
```

### 构建 ###

- [ ] 用这个命令 `go build -ldflags="-s -w" ...` 去掉你的二进制文件
- [ ] 拆分构建不同版本的简单方法

  - 用 `// +build integration` 并且运行他们 `go test -v --tags integration .`

### 测试 ###

- [ ] `go test -short` 允许减少要运行的一组测试

```go
func TestSomething(t *testing.T) {
  if testing.Short() {
    t.Skip("skipping test in short mode.")
  }
}
```

- [ ] 根据架构跳过测试

```go
if runtime.GOARM == "arm" {
  t.Skip("this doesn't work under ARM")
}
```

- [ ] 测试名称 `package_test` 比 `package` 要好
- [ ] 对于快速基准比较，我们有一个 `benchcmp` 工具

    - https://godoc.org/golang.org/x/tools/cmd/benchcmp

- [ ] 用 `testing.AllocsPerRun` 跟踪你的分配

    - https://godoc.org/testing#AllocsPerRun

### 工具 ###

- [ ] 快速替换 `gofmt -w -l -r "panic(err) -> log.Error(err)" .`
- [ ] `go list` 允许找到所有直接和传递的依赖关系

    - `go list -f '{{ .Imports }}' package`
    - `go list -f '{{ .Deps }}' package`

### Misc ###

- [ ] dump goroutines https://stackoverflow.com/a/27398062/433041

```go
go func() {
  sigs := make(chan os.Signal, 1)
  signal.Notify(sigs, syscall.SIGQUIT)
  buf := make([]byte, 1<<20)
  for {
    <-sigs
    stacklen := runtime.Stack(buf, true)
    log.Printf("=== received SIGQUIT ===\n*** goroutine dump...\n%s\n*** end\n", buf[:stacklen])
  }
}()
```

- [ ] 在编译期检查接口的实现

```go
var _ io.Reader = (*MyFastReader)(nil)
```

- [ ] len(nil) = 0

    - https://golang.org/pkg/builtin/#len

- [ ] 匿名结构很酷

```go
var hits struct {
  sync.Mutex
  n int
}
hits.Lock()
hits.n++
hits.Unlock()
```

- [ ] `httputil.DumpRequest` 是非常有用的东西，不要自己创建

    - https://godoc.org/net/http/httputil#DumpRequest

- [ ] 获得调用堆栈，我们可以使用 `runtime.Caller`

    - https://golang.org/pkg/runtime/#Caller
