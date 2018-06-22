# Go-advices

## Contents

- [Code](#code)
- [Concurrency](#concurrency)
- [Performance](#performance)
- [Build](#build)
- [Testing](#testing)
- [Tools](#tools)
- [Misc](#misc)
    
### Code
- [ ] go fmt your code, make everyone happier
- [ ] multiple if statements can be collapsed into switch
- [ ] use `chan struct{}` to pass signal, `chan bool` makes it less clear
- [ ] prefer `30 * time.Second` instead of `time.Duration(30) * time.Second`
- [ ] always wrap for-select idiom to a function
- [ ] group `const` declarations by type and `var` by logic and/or type
- [ ] every blocking or IO function call should be cancelable or at least timeoutable
- [ ] implement `Stringer` interface for integers const values
  - https://godoc.org/golang.org/x/tools/cmd/stringer
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
- [ ] use panic only in very specific situations, you have to handle error
- [ ] don't use alias for enums 'cause this breaks type safety
  - https://play.golang.org/p/MGbeDwtXN3
  - 
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
- [ ] if you're going to omit returning params, do it explicitly
  - so prefer this ` _ = f()` to this `f()`
- [ ] we've a short form for slice initialization `a := []T{}`
- [ ] iterate over array or slice using range loop
  -  instead of `for i := 3; i < 7; i++ {...}` prefer `for _, c := range a[3:7] {...}`
- [ ] use backquote(\`) for multiline strings
- [ ] skip unused param with _
  ```go
  func f(a int, _ string() {}
  ```
- [ ] If you are comparing timestamps, use `time.Before` or `time.After`. Don't use `time.Sub` to get a duration and then check its value.
- [ ] always pass context as a first param to a func with a `ctx` name
- [ ] few params of the same type can be defined in a short way
  ```go
  func f(a int, b int, s string, p string)
  ```
  ```go
  func f(a, b int, s, p string)
  ```
- [ ] the zero value of a slice is nil
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
- [ ] do not compare enum types with `<`, `>`, `<=` and `>=`
  - use explicit values, don't do this:
  ```go
  value := reflect.ValueOf(object)
  kind := value.Kind()
  if kind >= reflect.Chan && kind <= reflect.Slice {
    // ...
  }
  ```
- [ ] use `%+v` to print data with sufficient details
- [ ] be careful with empty struct `struct{}`, see issue: https://github.com/golang/go/issues/23440
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
- [ ] wrap errors with http://github.com/pkg/errors
  - so: `errors.Wrap(err, "additional message to a given error")`
- [ ] be careful with `range` in Go:
  - `for i := range a` and `for i, v := range &a` doesn't make a copy of `a`
  - but `for i, v := range a` does
  - more: https://play.golang.org/p/4b181zkB1O
- [ ] reading nonexistent key from map will not panic
  - `value := map["no_key"]` will be zero value
  - `value, ok := map["no_key"]` is much better
- [ ] do not use raw params for file operation
  - instead of an octal parameter like `os.MkdirAll(root, 0700)`
  - use predefined constants of this type `os.FileMode`
- [ ] don't forget to specify a type for `iota`
  - https://play.golang.org/p/mZZdMaI92cI
  ```
  const (
    _ = iota
    testvar         // will be int
  )
  ```
   vs
   ```
  type myType int
  const (
    _ myType = iota
    testvar         // will be myType
  )```
- [ ] use `_ = b[7]` for early bounds check to guarantee safety of writes below
  - https://stackoverflow.com/questions/38548911/is-it-necessary-to-early-bounds-check-to-guarantee-safety-of-writes-in-golang
  - https://github.com/golang/go/blob/master/src/encoding/binary/binary.go#L82

### Concurrency
- [ ] best candidate to make something once in a thread-safe way is `sync.Once`
  - don't use flags, mutexes, channels or atomics
- [ ] to block forever use `select{}`, omit channels, waiting for a signal
- [ ] don't close in-channel, this is a responsibility of it's creator
  - writing to a closed channel will cause a panic
- [ ] `func NewSource(seed int64) Source` in `math/rand` is not concurrency-safe. The default `lockedSource` is concurrency-safe, see issue: https://github.com/golang/go/issues/3611
  - more: https://golang.org/pkg/math/rand/

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
- [ ] `time.Time` has pointer field `time.Location` and this is bad for go GC
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
- [ ] don't use defer in a loop or you'll get a small memory leak
  - 'cause defers will grow your stack without the reason
- [ ] don't forget to stop ticker, unless you need a leaked channel
  ```go
  ticker := time.NewTicker(1 * time.Second)
  defer ticker.Stop()
  ```
- [ ] use custom marshaler to speed up marshaling
  - but before using it - profile! ex: https://play.golang.org/p/SEm9Hvsi0r
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
- [ ] `sync.Map` isn't a silver bullet, do not use it without a strong reasons
  - more: https://github.com/golang/go/blob/master/src/sync/map.go#L12
- [ ] storing non-pointer values in `sync.Pool` allocates memory
  - more: https://github.com/dominikh/go-tools/blob/master/cmd/staticcheck/docs/checks/SA6002
- [ ] regular expressions are mutexed
  - to avoid performance degradation in concurrent programs make a copy:
  ```go
  re, err := regexp.Compile(pattern)
  re2 := re.Copy()
  ```
- [ ] to hide a pointer from escape analysis you might carefully(!!!) use this func:
  - source: https://go-review.googlesource.com/c/go/+/86976
  ```
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
- [ ] for fastest atomic swap you might use this
  `m := (*map[int]int)(atomic.LoadPointer(&ptr))`
- [ ] use buffered I/O if you do many sequential reads or writes
  - to reduce number of syscalls

### Build
- [ ] strip your binaries with this command `go build -ldflags="-s -w" ...`
- [ ] easy way to split test into different builds
  - use `// +build integration` and run them with `go test -v --tags integration .`
- [ ] tiniest Go docker image
  - https://twitter.com/bbrodriges/status/873414658178396160
  - `CGO_ENABLED=0 go build -ldflags="-s -w" app.go && tar C app | docker import - myimage:latest`
- [ ] run `go format` on CI and compare diff
  - this will ensure that everything was generated and committed
- [ ] to run Travis-CI with the latest Go use `travis 1`
  - see more: https://github.com/travis-ci/travis-build/blob/master/public/version-aliases/go.json
- [ ] check if there are mistakes in code formatting `diff -u <(echo -n) <(gofmt -d .)`

### Testing
- [ ] prefer `package_test` name for tests, rather than `package`
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
- [ ] track your allocations with `testing.AllocsPerRun`
  - https://godoc.org/testing#AllocsPerRun
- [ ] run your benchmarks multiple times, to get rid of noise
  - `go test -test.bench=. -count=20`

### Tools
- [ ] quick replace `gofmt -w -l -r "panic(err) -> log.Error(err)" .`
- [ ] `go list` allows to find all direct and transitive dependencies
  - `go list -f '{{ .Imports }}' package`
  - `go list -f '{{ .Deps }}' package`
- [ ] for fast benchmark comparison we've a `benchstat` tool
  - https://godoc.org/golang.org/x/perf/cmd/benchstat

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
- [ ] to get call stack we've `runtime.Caller` https://golang.org/pkg/runtime/#Caller
- [ ] to marshal arbitrary JSON you can marshal to `map[string]interface{}{}`
- [ ] configure your `CDPATH` so you can do `cd github.com/golang/go` from any directore
  - add this line to your `bashrc`(or analogue) `export CDPATH=$CDPATH:$GOPATH/src`
