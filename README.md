# Go-advices

### Code
- [ ] go fmt your code, make everyone happier
- [ ] multiple if statements can be collapsed into switch
- [ ] use `chan struct{}` to pass signal
  - `chan bool` makes it less clear, btw `struct{}` is more optimal
- [ ] prefer `30 * time.Seconds` instead of `time.Duration(30) * time.Seconds`
- [ ] always wrap for-select idiom to a function
- [ ] group `const` declarations by type and `var` by logic and/or type
- [ ] every blocking or IO function call should be cancelable or at least timeoutable
- [ ] implement `Stringer` interface for integers const values
- [ ] check your defer's error
  ```go
  defer func() {
      err := ocp.Close()
      if err != nil {
          rerr = err
      }
  }()
  ```
- [ ] don't use `checkErr` function which panics or does `os.Exit`
- [ ] don't use alias for enums 'cause this breaks type safety
  - https://play.golang.org/p/MGbeDwtXN3

### CI
- [ ] run `go format` on CI and compare diff
  - this will ensure that everything was generated and commited
- [ ] to run Travis-CI with the latest Go use `travis 1`
  - see more: https://github.com/travis-ci/travis-build/blob/master/public/version-aliases/go.json
- [ ] check if there are mistakes in code formatting `diff -u <(echo -n) <(gofmt -d .)`

### Concurrency
- [ ] best candidate to make something once in a thread-safe way is `sync.Once`
  - don't user flags, mutexes, channels or atomics
- [ ] to block forever use `select{}`, omit channels, waiting for a signal

### Performance
- [ ] do not omit `defer`
  - 200ns speedup is negligible in most cases
- [ ] always close http body aka `defer r.Body.Close()`
  - unless you need leaked goroutine
- [ ] filtering without allocating
  ```go
    b := a[:0]
    for _, x := range a {
    	if f(x) {
		    b = append(b, x)
    	}
    }
  ```
- [ ] `time.Time` has pointer field `time.Location` and this is bad go GC
  - it's relevant only for big number of `time.Time`, use timestamp instead
- [ ] prefer `regexp.MustCompile` instead of `regexp.Compile`
  - in most cases your regex is immutable, so init it in `func init`
- [ ] do not overuse `fmt.Sprintf` in your hot path. It is costly due to maintaining the buffer pool and dynamic dispatches for interfaces.
  - if you are doing `fmt.Sprintf("%s%s", var1, var2)`, consider simple string concatenation.
  - if you are doing `fmt.Sprintf("%x", var)`, consider using `hex.EncodeToString` or `strconv.FormatInt(var, 16)`
- [ ] always discard body e.g. `io.Copy(ioutil.Discard, resp.Body)` if you don't use it
  - HTTP client's Transport will not reuse connections unless the body is read to completion and closed
  ```go
    res, _ := client.Do(req)
    io.Copy(ioutil.Discard, res.Body)
    defer res.Body.Close()
  ```
- [ ] don't user defer in a loop or you'll get a small memory leak
  - 'cause defers will grow your stack without the reason

### Build
- [ ] strip your binaries with this command `go build -ldflags="-s -w" ...`
- [ ] easy way to split test into different builds
  - use `// +build integration` and run them with `go test -v --tags integration .`

### Testing
- [ ] `go test -short` allows to reduce set of tests to be runned
  ```go
  func TestSomething(t *testing.T) {
    if testing.Short() {
      t.Skip("skipping test in short mode.")
    }
  }
  ```
- [ ] skip test deppending on architecture
  ```go
  if runtime.GOARM == "arm" {
    t.Skip("this doesn't work under ARM")
  }
  ```
- [ ] prefer `package_test` name for tests, rather than `package`
- [ ] for fast benchmark comparison we've a `benchcmp` tool
  - https://godoc.org/golang.org/x/tools/cmd/benchcmp
### Tools
- [ ] quick replace `gofmt -w -l -r "panic(err) -> log.Error(err)" .`

### Misc
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
- [ ] check interface implementation during compilation
  ```go
    var _ io.Reader = (*MyFastReader)(nil)
  ```
- [ ] if a param of len is nil then it's zero
  - https://golang.org/pkg/builtin/#len
- [ ] anonymous structs are cool
  ```go
  var hits struct {
    sync.Mutex
    n int
  }
  hits.Lock()
  hits.n++
  hits.Unlock()
  ```
- [ ] `httputil.DumpRequest` is very useful thing, don't create your own
  - https://godoc.org/net/http/httputil#DumpRequest
