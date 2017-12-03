# Go-advices
译者: 自己可能水平还不到家, 欢迎大家批评指正.  
碰到这些: !() 是我的注释  
[tannineo](https://github.com/tannineo)  

### Code 代码
- [ ] `go fmt`你的代码, 大家都开心
- [ ] 多重`if`语句可以折叠成`switch`
- [ ] 使用空结构`chan struct{}`传递信号
  - `chan bool`稍显不清晰, `struct{}`更合适
- [ ] 用`30 * time.Second`, 而不是`time.Duration(30) * time.Second`
- [ ] 将for-select包装成函数
- [ ] 按照类型组织`const`声明, 按照 逻辑 和/或 类型 组织`var`声明
- [ ] 所有阻塞或者IO的函数调用都应该能被取消或者有超时(timeout)检测
- [ ] 为整型常量实现`Stringer`接口
- [ ] 检查defer中的错误
  ```go
  defer func() {
      err := ocp.Close()
      if err != nil {
          rerr = err
      }
  }()
  ```
- [ ] 不要使用`checkErr`函数, 会产生panic或者`os.Exit`
- [ ] enum不要使用类型别名, 这会破坏类型安全(type safety)
  - https://play.golang.org/p/MGbeDwtXN3
  -
  ```go
  package main
  type Status = int
  type Format = int // 去掉 `=` 就能保证类型安全

  const A Status = 1
  const B Format = 1

  func main() {
    println(A == B) // true 不是我们想要的
    // 我们希望能在编译时就找出AB两者不是一个类型的错误
  }
  ```
- [ ] 如果你要省略返回值, 做的明确些
  - 所以` _ = f()`好过`f()`
- [ ] 我们可以简化切片的初始化`a := []T{}`
- [ ] 数组或切片的遍历用`range`
  - 与其这么写`for i := 3; i < 7; i++ {...}`, 不如这么写`for _, c := range a[3:7] {...}`
- [ ] 多行`string`用反引号(\`)

### CI 可持续集成
- [ ] 在CI上运行`go format`, 然后比较不同
  - 这保证了生成和提交所有代码
- [ ] 使用`travis 1`在Travis CI上运行最新的Go
  - 看更多: https://github.com/travis-ci/travis-build/blob/master/public/version-aliases/go.json
- [ ] 查看在代码格式化时是否出错`diff -u <(echo -n) <(gofmt -d .)`

### Concurrency 并发
- [ ] 保证线程安全的情况下只运行一次, 用`sync.Once`
  - 不要使用flag, mutex, channel或atomic
- [ ] 永久阻塞用`select{}`, 省去channel等待signal

### Performance 性能
- [ ] 不要省去`defer`
  - 在大多数例子中快个200ns微不足道
- [ ] 保证关闭HTTP body, 也就是`defer r.Body.Close()`
  - 除非你要goroutine多的泄露(leak)
- [ ] 筛选(filter)时不分配内存(allocate)
  ```go
    b := a[:0]
    for _, x := range a {
        if f(x) {
            b = append(b, x)  // slice机制有关
        }
    }
  ```
- [ ] `time.Time`里面有个指针`time.Location`, 这对Go的垃圾回收(GC)不友好
  - 若是只用到了`time.Time`里面的大数, 建议用timestamp取代
- [ ] `regexp.MustCompile`比`regexp.Compile`更好
  - 在大多数例子中正则是需要一直存在的, 所以在`func init`中初始化
- [ ] 在经常运行的代码上不要过分使用`fmt.Sprintf`. 这样开销很大, 因为需要维护缓存池(buffer pool)并动态分派接口
  - 与其`fmt.Sprintf("%s%s", var1, var2)`, 不如考虑使用简单的字符串连接`+`.
    !([参考](https://sheepbao.github.io/post/golang_string_connect_performance/))
  - 与其`fmt.Sprintf("%x", var)`, 不如`hex.EncodeToString`或`strconv.FormatInt(var, 16)`
- [ ] 像这样弃用body之类的东西: `io.Copy(ioutil.Discard, resp.Body)` 如果不这么做...
  - HTTP客户端传输不会重用这些连接, 直到body被读取完和关闭
  ```go
  res, _ := client.Do(req)
  io.Copy(ioutil.Discard, res.Body)
  defer res.Body.Close()
  ```
- [ ] 不要在循环中使用defer, 不然会造成内存泄漏
  - 因为defer会不明地增加栈
- [ ] 别忘了停掉ticker, 除非你想channel溢出
  ```go
  ticker := time.NewTicker(1 * time.Second)
  defer ticker.Stop()
  ```
- [ ] 使用定制的marshaler加快解析JSON的速度
  - 但在使用之前 - 别忘了扫整个struct(profile)! ex: https://play.golang.org/p/SEm9Hvsi0r
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

### Build 构建
- [ ] 为二进制文件瘦身`go build -ldflags="-s -w" ...` !(去除调试信息和符号表)
- [ ] 对不同构建分割测试的简单方法
  - 加上注释`// +build integration`, 运行测试使用命令:`go test -v --tags integration .`
  - [ ] 最小Go docker镜像
    - https://twitter.com/bbrodriges/status/873414658178396160
    - `CGO_ENABLED=0 go build -ldflags="-s -w" app.go && tar C app | docker import - myimage:latest`

### Testing 测试
- [ ] `go test -short` 可以减少一些测试 !(控制测试粒度)
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
- [ ] 类似`package_test`为测试命名, 比`package`更好
- [ ] 为快速比较benchmark, 我们有工具`benchcmp`
  - https://godoc.org/golang.org/x/tools/cmd/benchcmp
- [ ] 追踪内存分配状态: `testing.AllocsPerRun`
  - https://godoc.org/testing#AllocsPerRun

### Tools 工具
- [ ] 快速替换 `gofmt -w -l -r "panic(err) -> log.Error(err)" .`
- [ ] `go list` 可以列出所有直接和间接依赖
  - `go list -f '{{ .Imports }}' package`
  - `go list -f '{{ .Deps }}' package`

### Misc 杂项
- [ ] 转存(dump)goroutine https://stackoverflow.com/a/27398062/433041
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
- [ ] 编译时检查接口的实现
  ```go
  var _ io.Reader = (*MyFastReader)(nil) // 声明一个要测试的接口给_
  ```
- [ ] `len()`的参数是nil, 则值为0
  - https://golang.org/pkg/builtin/#len
- [ ] 一些匿名结构很酷
  ```go
  var hits struct {
      sync.Mutex
      n int
  }
  hits.Lock()
  hits.n++
  hits.Unlock()
  ```
- [ ] `httputil.DumpRequest`很有用, 不用重复造轮子
  - https://godoc.org/net/http/httputil#DumpRequest
- [ ] 获取堆栈调用: `runtime.Caller` https://golang.org/pkg/runtime/#Caller
- [ ] 解析任意JSON, 放进进`map[string]interface{}{}`
