<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

(有些建议在 [go-critic](https://github.com/go-critic/go-critic) 中实现)

- [Go-advices 中文版本](#go-advices-%E4%B8%AD%E6%96%87%E7%89%88%E6%9C%AC)
    - [代码](#%E4%BB%A3%E7%A0%81)
    - [并发](#%E5%B9%B6%E5%8F%91)
    - [性能](#%E6%80%A7%E8%83%BD)
    - [构建](#%E6%9E%84%E5%BB%BA)
    - [测试](#%E6%B5%8B%E8%AF%95)
    - [工具](#%E5%B7%A5%E5%85%B7)
    - [Misc](#misc)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Go-advices 中文版本 #

### 代码 ###

- [ ] 使用 `go fmt` / `gofmt` 格式化你的代码, 让大家都开心
- [ ] 多个 if 语句可以折叠成 switch
- [ ] 用 `chan struct{}` 来传递信号, `chan bool` 表达的不够清楚
- [ ] `30 * time.Second` 比 `time.Duration(30) * time.Second` 更好
- [ ] 总是把 for-select 换成一个函数
- [ ] 分组定义 `const` 类型声明和 `var` 逻辑类型声明
- [ ] 每个阻塞或者 IO 函数操作应该是可取消的或者至少是可超时的
- [ ] 为整型常量值实现 `Stringer` 接口
    
    - https://godoc.org/golang.org/x/tools/cmd/stringer）

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
- [ ] 仅仅在很特殊情况下才使用 panic, 你必须要去处理 error
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

- [ ] 如果你想省略返回参数，你最好表示出来

    - ` _ = f()` 比 `f()` 更好

- [ ] 我们用 `a := []T{}` 来简短初始化 slice
- [ ] 用 range 循环来进行数组或 slice 的迭代

    -  `for _, c := range a[3:7] {...}` 比 `for i := 3; i < 7; i++ {...}` 更好

- [ ] 多行字符串用反引号(\`)
- [ ] 用 `_` 来跳过不用的参数

```go
  func f(a int, _ string) {}
```

- [ ] 如果你是比较时间戳，请使用 `time.Before` 或 `time.After` ，不要使用 `time.Sub` 来获得 duration (持续时间)，然后检查它的值。 #17
- [ ] 总是将上下文作为第一个参数传递给具有 `ctx` 名称的 func
- [ ] few params of the same type can be defined in a short way
- [ ] 几个相同类型的参数定义可以用简短的方式来进行

```go
  func f(a int, b int, s string, p string)
```

```go
  func f(a, b int, s, p string)
```

- [ ] 一个 slice 的零值是 nil
  
    - https://play.golang.org/p/pNT0d_Bunq

```go
  var s []int
  fmt.Println(s, len(s), cap(s))
  if s == nil {
    fmt.Println("nil!")
  }
  // Output:
  // [] 0 0
  // nil!
```

  - https://play.golang.org/p/meTInNyxtk

```go
  var a []string
  b := []string{}

  fmt.Println(reflect.DeepEqual(a, []string{}))
  fmt.Println(reflect.DeepEqual(b, []string{}))
  // Output:
  // false
  // true
```

- [ ] 不要将枚举类型与 `<`, `>`, `<=` 和 `>=` 进行比较
  
    - 使用确定的值，不要像下面这样做:
    
```go
  value := reflect.ValueOf(object)
  kind := value.Kind()
  if kind >= reflect.Chan && kind <= reflect.Slice {
    // ...
  }
```
- [ ] 用 `%+v` 来打印数据的比较全的信息
- [ ] 注意空结构 `struct{}`, 看 issue: https://github.com/golang/go/issues/23440
    
    - more: https://play.golang.org/p/9C0puRUstrP
    
```go
  func f1() {
    var a, b struct{}
    print(&a, "\n", &b, "\n") // Prints same address
    fmt.Println(&a == &b)     // Comparison returns false
  }
      
  func f2() {
    var a, b struct{}
    fmt.Printf("%p\n%p\n", &a, &b) // Again, same address
    fmt.Println(&a == &b)          // ...but the comparison returns true
  }
```

- [ ] 包装错误： http://github.com/pkg/errors

    - 例如: `errors.Wrap(err, "additional message to a given error")`

- [ ] 在 Go 里面要小心使用 `range`:
    
    - `for i := range a` and `for i, v := range &a` ，都不是 `a` 的副本
    - 但是 `for i, v := range a` 里面的就是 `a` 的副本
    - 更多: https://play.golang.org/p/4b181zkB1O

- [ ] 从 map 读取一个不存在的 key 将不会 panic
  
    - `value := map["no_key"]` 将得到一个 0 值
    - `value, ok := map["no_key"]` 更好

- [ ] 不要使用原始参数进行文件操作
  
    - 而不是一个八进制参数 `os.MkdirAll(root, 0700)`
    - 使用此类型的预定义常量 `os.FileMode`


- [ ] 不要忘记为 `iota` 指定一种类型
    - https://play.golang.org/p/mZZdMaI92cI
    
    ```go
    const (
      _ = iota
      testvar         // testvar 将是 int 类型
    )
    ```
    
    vs
    
    ```go
    type myType int
    const (
      _ myType = iota
      testvar         // testvar 将是 myType 类型
    )
    ```

- [ ] 用 `_ = b[7]` 为了早期检查以确保下面的写入安全
  - https://stackoverflow.com/questions/38548911/is-it-necessary-to-early-bounds-check-to-guarantee-safety-of-writes-in-golang
  - https://github.com/golang/go/blob/master/src/encoding/binary/binary.go#L82

- [ ] 不要在你不拥有的结构上使用 `encoding / gob`

  - 它不受新添加或重新排序字段的保护

- [ ] 在 Go 里面要小心使用 `range`:
    
    - `for i := range a` and `for i, v := range &a` ，都不是 `a` 的副本
    - 但是 `for i, v := range a` 里面的就是 `a` 的副本
    - 更多: https://play.golang.org/p/4b181zkB1O

### 并发 ###

- [ ] 以线程安全的方式创建一些东西的最好选择是 `sync.Once`

    - 不要用 flags, mutexes, channels or atomics

- [ ] 永远不要使用 `select{}`, 省略通道， 等待信号
- [ ] 不要在 channel 里关闭，这是它的创作者的责任。
   
    - 往一个关闭的 channel 会引起 panic 

- [ ] `math/rand` 中的 `func NewSource(seed int64) Source` 不是并发安全的，默认的 `lockedSource` 是并发安全的, see issue: https://github.com/golang/go/issues/3611
  
    - 更多: https://golang.org/pkg/math/rand/

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

- [ ] 用自定义的 marshaler 去加速 marshaler 过程
  
    - 但是在使用它之前要进行定制！例如：https://play.golang.org/p/SEm9Hvsi0r
  
```go
  func (entry Entry) MarshalJSON() ([]byte, error) {
    buffer := bytes.NewBufferString("{")
    first := true
    for key, value := range entry {
    	jsonValue, err := json.Marshal(value)
    	if err != nil {
    		return nil, err
    	}
    	if !first {
    		buffer.WriteString(",")
    	}
    	first = false
    	buffer.WriteString(key + ":" + string(jsonValue))
    }
    buffer.WriteString("}")
    return buffer.Bytes(), nil
  }
```

- [ ] `sync.Map` 不是万能的，没有很强的理由就不要使用它。
  
    - 了解更多: https://github.com/golang/go/blob/master/src/sync/map.go#L12
  
- [ ] 在 `sync.Pool` 中分配内存存储非指针数据
    
    - 了解更多: https://github.com/dominikh/go-tools/blob/master/cmd/staticcheck/docs/checks/SA6002

- [ ] 正则表达式是互斥的
  
    - 为了避免在并行程序性能下降，使用复制:
  
```go
  re, err := regexp.Compile(pattern)
  re2 := re.Copy()
```

- [ ] 为了隐藏逃生分析的指针，你可以小心使用这个函数：:
    
    - 来源: https://go-review.googlesource.com/c/go/+/86976
    
```go
  // noescape hides a pointer from escape analysis.  noescape is
  // the identity function but escape analysis doesn't think the
  // output depends on the input. noescape is inlined and currently
  // compiles down to zero instructions.
  //go:nosplit
  func noescape(p unsafe.Pointer) unsafe.Pointer {
    x := uintptr(p)
    return unsafe.Pointer(x ^ 0)
  }
```

- [ ] 对于最快的原子交换，你可以使用这个 `m := (*map[int]int)(atomic.LoadPointer(&ptr))`
- [ ] 如果执行许多顺序读取或写入操作，请使用缓冲 I/O

   - 减少系统调用次数

### 构建 ###

- [ ] 用这个命令 `go build -ldflags="-s -w" ...` 去掉你的二进制文件
- [ ] 拆分构建不同版本的简单方法

    - 用 `// +build integration` 并且运行他们 `go test -v --tags integration .`

- [ ] 最小的 Go Docker 镜像
  
    - https://twitter.com/bbrodriges/status/873414658178396160
    - `CGO_ENABLED=0 go build -ldflags="-s -w" app.go && tar C app | docker import - myimage:latest`

- [ ] run go format on CI and compare diff
  
    - 这将确保一切都是生成的和承诺的

- [ ] 用最新的 Go 运行 Travis-CI，用 `travis 1`
  
    - 了解更多：https://github.com/travis-ci/travis-build/blob/master/public/version-aliases/go.json

- [ ] 检查代码格式是否有错误 `diff -u <(echo -n) <(gofmt -d .)`

### 测试 ###

- [ ] 更喜欢 `package_test` 命名，而不是 `package` 。
- [ ] `go test -short` 允许减少要运行的一组测试

```go
  func TestSomething(t *testing.T) {
    if testing.Short() {
      t.Skip("skipping test in short mode.")
    }
  }
```

- [ ] 根据系统架构跳过测试

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

- [ ] 多次运行你的基准测试可以避免噪音。
    
    - `go test -test.bench=. -count=20`

### 工具 ###

- [ ] 快速替换 `gofmt -w -l -r "panic(err) -> log.Error(err)" .`
- [ ] `go list` 允许找到所有直接和传递的依赖关系

    - `go list -f '{{ .Imports }}' package`
    - `go list -f '{{ .Deps }}' package`

- [ ] 对于快速基准比较，我们有一个 `benchstat` 工具。

    - https://godoc.org/golang.org/x/perf/cmd/benchstat

- [ ] [go-critic](https://github.com/go-critic/go-critic) linter 从这个文件中强制执行几条建议

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
      log.Printf("=== received SIGQUIT ===\n*** goroutine dump...\n%s\n*** end\n"  , buf[:stacklen])
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

- [ ] 要 marshal 任意的 JSON， 你可以 marshal 为 `map[string]interface{}{}`

- [ ] 配置你的 `CDPATH` 以便你能在任何目录执行 `cd github.com/golang/go`
  
    - 添加这一行代码到 `bashrc`(或者其他类似的) `export CDPATH=$CDPATH:$GOPATH/src`
