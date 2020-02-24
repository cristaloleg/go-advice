# Go-advice

(Some of advices are implemented in [go-critic](https://github.com/go-critic/go-critic))

## Contents

- [Go Proverbs](#go-proverbs)
- [The Zen of Go](#the-zen-of-go)
- [Code](#code)
- [Concurrency](#concurrency)
- [Performance](#performance)
- [Modules](#modules)
- [Build](#build)
- [Testing](#testing)
- [Tools](#tools)
- [Misc](#misc)

### Go Proverbs

- 메모리를 공유하면서 통신하지 말고, 통신하면서 메모리를 공유해라.
- 동시성(concurrency은 병렬(parallelism)처리가 아니다.
- 채널 오케스트레이션; 뮤텍스 직렬화(serialise).
- 인터페이스가 커질수록 추상화는 약해진다.
- 제로값(zero value)를 유용하게 만들자.
- `interface{}`는 아무 말도 하지 않는다 (say nothing).
- `gofmt`의 스타일은 어느 누구의 취향(favourite)도 아니지만, `gofmt`는 모두가 좋아하는 스타일이다.
- 약간의 복사(a little copy) 약간의 의존성(a little dependency)보다 낫다.
- Syscall은 항상 build tags로 가드(guarded)되어야 한다.
- Cgo는 항상 build tags로 가드(guarded)되어야 한다.
- Cgo는 Go가 아니다.
- `unsafe` 패키지는 어떠한 보증(guarantees)도 없다.
- 명료함(clear)은 현명함(clever)보다 낫다.
- 리플렉션(reflection)은 결코 명료하지 않다(is never clear).
- 에러(errors)는 값(values)이다.
- 단지 에러를 검사(check) 할 것만이 아니라, 우아하게 다뤄어라 (handle gracefully).
- 아키텍쳐는 디자인 하라, 컴포넌트를 이름지어라, 상세사항은 문서화 해라(design the architecture, name the components, document the details).
- 문서는 유저를 위한 것이다.
- 패닉을 사용하지 말 것 (don't panic).

저자: Rob Pike
추가 정보: https://go-proverbs.github.io/

### The Zen of Go

- 각 package는 단일 목적을 이행/달성한다.
- 에러를 분명하게(explicitly) 다루어라.
- 깊게 중첩하지 말고(nesting deeply) 초기에 리턴(return early) 해라
- 호출자(caller)에게 동시성(concurrency)를 맡겨라.
- goroutine을 사용(launch)하기 전, 언제 멈춰어야 할 지 알자.
- 패키지 레벨 상태를 피해라(avoid package level state).
- 간단함은 중요하다(simplicity matters).
- 여러러분들의 API의 행동(behaviour)을 고정하기 위해(to lock) 테스트를 작성해라.
- 느리다고 생각되면, 먼저 벤치마크를 위해서 증명하라
- 중재는 미덕이다(moderation is a virtue).
- 유지보수성을 고려해라(maintainability counts).

저자: Dave Cheney
추가 정보: https://the-zen-of-go.netlify.com/

### Code

#### Always `go fmt` your code.

커뮤니티는 공식적인 Go의 포맷을 사용한다. 새로운 바퀴를 재발명하지 않도록 하자. (do not reinvent the wheel).

코드의 엔트로피를 감소하기 위해서 노력해라. 이는 다른사람들로 하여금 코드를 읽기 쉽게 도와 줄 것이다.

#### Multiple if-else statements can be collapsed into a switch

```go
// NOT BAD
if foo() {
    // ...
} else if bar == baz {
    // ...
} else {
    // ...
}

// BETTER
switch {
case foo():
    // ...
case bar == baz:
    // ...
default:
    // ...
}
```

#### To pass a signal prefer `chan struct{}` instead of `chan bool`.

구조체 내에서 `chan bool`의 정의가 있는 경우, 때때로, 이 값이 어떻게 사용될 지에 대해서 이해하는 것이 쉽지 않을 수 있다. 예를들면:

```go
type Service struct {
    deleteCh chan bool // what does this bool mean?
}
```

하지만, 우리는 이를 `chan struct{}`로 변경함으로써 더욱 더 분명하게 만들 수 있다. 여기서 `chan struct{}`는 명시적으로 값에 대해서 고려하지 않음을 의미한다(이는 항상 `struct{}`이다). 하지만, 아래의 예와 같이 발생할 수 있는 이벤트에 대해서 생각해 볼 수 있다:

```go
type Service struct {
    deleteCh chan struct{} // ok, if event than delete something.
}
```

#### Prefer `30 * time.Second` instead of `time.Duration(30) * time.Second`

타입이 지정되지 않은 const에 대해서 type으로 래핑(wrap)할 필요가 없다. 컴파일러가 이를 스스로 알아 낼 것이다. 또한 const를 우선적으로 명기하는 것을 선호한다. 아래를 참고해라:

```go
// BAD
delay := time.Second * 60 * 24 * 60

// VERY BAD
delay := 60 * time.Second * 60 * 24

// GOOD
delay := 24 * 60 * 60 * time.Second
```

#### Use `time.Duration` instead of `int64` + variable name

```go
// BAD
var delayMillis int64 = 15000

// GOOD
var delay time.Duration = 15 * time.Second
```

#### Group `const` declarations by type and `var` by logic and/or type

```go
// BAD
const (
    foo = 1
    bar = 2
    message = "warn message"
)

// MOSTLY BAD
const foo = 1
const bar = 2
const message = "warn message"

// GOOD
const (
    foo = 1
    bar = 2
)

const message = "warn message"
```

이 패턴은 `var`의 경우에도 적용된다.

- [ ] 모든 블로킹(every blocking) 혹은 입출력 함수호출(IO function call)은 취소가능(cancelable)하거나 혹은 적어도 타임아웃가능(timeout-able) 해야 한다.
- [ ] 정수의 상수 값을 위한 `Stringer`인터페이스를 구현해라
  - https://godoc.org/golang.org/x/tools/cmd/stringer
- [ ] 여러분들의 defer의 에러에 대해서 체크하자

```go
  defer func() {
      err := ocp.Close()
      if err != nil {
          rerr = err
      }
  }()
```

- [ ] 패닉이나 `os.Exit`를 수행하는 `checkErr` 함수를 사용하지 말것.
- [ ] 아주 특정한 상황에만 패닉을 사용하자, 여러분들은 에러를 다루어야 한다(You have to handle errors).
- [ ] enums에 대해서 alias를 사용하지 말것. 왜냐하면 이는 타입안정성을 깨트릴 수 있기 때문이다.
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

- [ ] 만약 여러분들이 반환되는 파라미터(returning params)를 생략하고자 한다면, 이를 명시적으로 수행해라
  - 그러므로 ` _ = f()` 을 `f()`보다 선호한다.
- [ ] 슬라이스 초기화의 약식 형식(short form)은 `a := []T{}`
- [ ] `range` loop을 사용해서 배열(array)이나 슬라이스(slice)를 순회(iterate over)하라
  - `for i := 3; i < 7; i++ {...}` 대신에 `for _, c := range a[3:7] {...}`를 사용할 것
- [ ] 여러라인의 문자열(multiline strings)을 위해 backquote(\`)를 사용할 것
- [ ] 사용하지 않는 파라미터는 `_`로 스킵(skip)하자

```go
  func f(a int, _ string) {}
```

- [ ] 타임스탬프(timestamps)를 비교할 경우, `time.Before` 혹은 `time.After`를 사용할 것. Duration과 그것의 값을 구하기 위해서 `time.Sub`을 사용하지 말 것.
- [ ] `context`는 `ctx`라는 이름으로 항상 첫번째 파라미터로 함수에 전달하자
- [ ] 같은 타입의 소수의 파라미터(few params of the same type)는 짧은 방식으로 정의될 수 있다.

```go
  func f(a int, b int, s string, p string)
```

```go
  func f(a, b int, s, p string)
```

- [ ] 슬라이스의 제로값(zero value)는 `nil` 이다.
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

- [ ] 열거형(enum types)을 `<`, `>`, `<=` 그리고 `>=` 와 비교하지 말 것
  - 명시적인 값을 사용하라. 아래와 같이 하지 말 것:

```go
  value := reflect.ValueOf(object)
  kind := value.Kind()
  if kind >= reflect.Chan && kind <= reflect.Slice {
    // ...
  }
```

- [ ] 충분한 세부사항과 함께 데이터를 출력하위해 `%+v`을 사용할 것
- [ ] 빈 구조체(empty struct: `struct{}`)를 주의 할 것, 관련 이슈: https://github.com/golang/go/issues/23440
  - 추가 정보: https://play.golang.org/p/9C0puRUstrP

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

- [ ] http://github.com/pkg/errors 를 참고해 에러를 wrap할 것
  - 그러므로: `errors.Wrap(err, "additional message to a given error")`
- [ ] Go에서의 `range`에 대해서 주의 할 것:
  - `for i := range a` 와 `for i, v := range &a` 에서 `a`의 카피를 만들지 않는다.
  - 그러나, `for i, v := range a` 에서는 `a`의 카피를 만든다 (즉, 이와 같은 경우, for문 내에서 `v`의 수정은 `a`와 무관)
  - 코드: https://play.golang.org/p/4b181zkB1관
- [ ] 존재하지 않는 key값을 map으로 부터 읽으려고 할 때, panic은 일어나지 않는다.
  - `value := map["no_key"]` 는 제로 값(zero value)가 된다.
  - `value, ok := map["no_key"]` 가 훨씬 더 낫다.
- [ ] file operation에 raw 파라미터를 사용하지 말 것
  - 다음과 같은 8진수 파라미터(octal parameter) 대신에: `os.MkdirAll(root, 0700)`
  - 사전정의된 상수 타입을 사용 할 것: `os.FileMode`
- [ ] `iota`에 대한 타입을 특정지을(specify) 것을 잊지 말 것
  - https://play.golang.org/p/mZZdMaI92cI

```go
  const (
    _ = iota
    testvar         // will be int
  )
```

  vs

```go
  type myType int
  const (
    _ myType = iota
    testvar         // will be myType
  )
```

#### Don’t use `encoding/gob` on structs you don’t own.

어느 순간에 있어서 구조체(structure)는 변경될 수 있으며, 여러분들은 이를 놓칠 수 있다. 결론적으로, 이러한 것은 버그를 찾기 어렵게 한다.

#### Don't depend on the evaluation order, especially in a return statement.

```go
// BAD
return res, json.Unmarshal(b, &res)

// GOOD
err := json.Unmarshal(b, &res)
return res, err
```

#### To prevent structs comparison add an empty field of `func` type

```go
type Point struct {
	_ [0]func()	// unexported, zero-width non-comparable field
	X, Y float64
}
```

#### Prefer `http.HandlerFunc` over `http.Handler`

`http.HandlerFunc` 사용하기 위해서 단지 하나의 함수만 필요로 하지만, `http.Handler`를 위해서는 타입이 필요하다.

#### Move `defer` to the top

이는 코드의 가독성(readability)를 향상시켜 주고 어떠한 것이 함수의 종료점(at the end of a function)에 작동할 지(invoked) 명확하게 해 준다.

#### JavaScript parses integers as floats and your int64 might overflow.

대신에 `json:"id,string"` 를 사용할 것.

```go
type Request struct {
	ID int64 `json:"id,string"`
}
```

### Concurrency

- [ ] 쓰레드 안정적인 방법(a thread-safe way)으로 한번에 무언가를 만드는 최적의 방법은 `sync.Once` 이다.
  - 플래그(flags), 뮤텍스(mutexes), 채널(channels) 혹은 아토믹(atomics)를 사용하지 말 것.
- [ ] 영원이 지속되는 것을 막기 위해서, `select{}`를 사용하고, 채널을 제거하고, 시그널을 기다릴 것. (to block forever use `select{}`, omit channels, waiting for a signal)
- [ ] 채널 내부에서 종료하지 말자(don't close in-channel), 이는 채널의 생성자가 할 일이다(this is a responsibility of it's creator).
  - 닫혀진 채널에 무언가를 작성하는 것은 패닉을 야기한다.
- [ ] `math/rand`내의 `func NewSource(seed int64) Source`는 동시성-안정적이지 않음(not concurrency-safe). 디폴트의 `lockedSource`가 동시성 안정적(concurrency-safe)이다, 다음의 이슈를 참고해라: https://github.com/golang/go/issues/3611
  - 추가정보: https://golang.org/pkg/math/rand/
- [ ] 커스텀 타입의 아토믹 값(atomic value)가 필요 할 경우, [atomic.Value](https://godoc.org/sync/atomic#Value)을 사용할 것

### Performance

- [ ] `defer`를 생략하지 말 것
  - 200ns의 속도향상은 대부분의 경우 무시(negligible)해도 좋다.
- [ ] 항상 http의 body를 `defer r.Body.Close()`로 닫을 것
  - 여러분이 누수된 고루틴(leaked goroutine)이 필요하지 않는 한...
- [ ] allocating 없는 filtering

```go
    b := a[:0]
    for _, x := range a {
    	if f(x) {
		    b = append(b, x)
    	}
    }
```

#### To help compiler to remove bound checks see this pattern `_ = b[7]`

- [ ] `time.Time`는 포인터 필드인 `time.Location`를 가지며, 이는 Go의 가비지컬렉터(GC)에 좋지 못함(bad)
  - 이는 `time.Time`의 큰 수에만 유의미하고(relevant), 이외에는 대신 timestamp를 사용할 것.
- [ ] `regexp.Compile` 대신, `regexp.MustCompile`을 선호한다.
  - 거의 모든 케이스에서 정규표현식(regex)은 변경할 수 없으므로, `func init`에서 초기화 해라.
- [ ] Hot path에서 `fmt.Sprintf`를 과하게 사용하지 말것. 인터페이스에 대한 버퍼 풀(Buffer Pool) 및 동적 디스패치(dynamic dispatches)를 유지보수하는 비용이 많이 든다.
  - 만약 `fmt.Sprintf("%s%s", var1, var2)`과 같은 것을 한다면, 간단한 문자열 연결(string concatenation)을 고려 할 것.
  - 만약 `fmt.Sprintf("%x", var)`과 같은 것을 한다면, `hex.EncodeToString` 혹은 `strconv.FormatInt(var, 16)`사용할 것을 고려할 것
- [ ] `io.Copy(ioutil.Discard, resp.Body)` 와 같이, 이를 사용하지 않는다면, 항상 body를 삭제(discard)할 것
  - Body가 종료를 위해서 읽혀지거나 닫히지 않는 한, HTTP client의 Transport는 connections을 재사용하지 않을 것이다.

```go
    res, _ := client.Do(req)
    io.Copy(ioutil.Discard, res.Body)
    defer res.Body.Close()
```

- [ ] 루프 내에서 defer를 사용하지 말자. 그렇지 않으면 약간의의메모리 누수(small memory leak)가 발생할 수 있다.
  - 왜냐하면 defers는 특별한 이유 없이 스택(stack)을 증가(grow)시키기 때문이다.
- [ ] 누수된 채널(leaked channel)이 필요하지 않는 한, ticker를 중지(stop)할 것을 잊지 말자.

```go
  ticker := time.NewTicker(1 * time.Second)
  defer ticker.Stop()
```

- [ ] 사용자 지정 마샬러(custom marshaler)를 사용하여 마샬링(marshaling)속도를 올려라.
  - 그러나 그것을 사용하기 전, 프로파일 해라! 예: https://play.golang.org/p/SEm9Hvsi0r

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

- [ ] `sync.Map`은 만병통치약(silver bullet)이 아니므로, 강력한 이유가 없다면 사용하지 말 것.
  - 참고: https://github.com/golang/go/blob/master/src/sync/map.go#L12
- [ ] non-pointer값을 `sync.Pool`에 저장(storing)하면, 메모리가 할당된다.
  - 참고: https://github.com/dominikh/go-tools/blob/master/cmd/staticcheck/docs/checks/SA6002

- [ ] 탈출 분석(escape analysis)로부터 포인터를 숨기기 위해서, 여러분들은 "신중하게(carefully!!!)" 이 함수를 사용할 수 있다:
  - 출처: https://go-review.googlesource.com/c/go/+/86976

```go
  // noescape hides a pointer from escape analysis.  noescape is
  // the identity function but escape analysis doesn't think the
  // output depends on the input. noescape is inlined and currently
  // compiles down to zero instructions.
  func noescape(p unsafe.Pointer) unsafe.Pointer {
  	x := uintptr(p)
  	return unsafe.Pointer(x ^ 0)
  }
```

- [ ] 가장 빠른 atomic swap을 위해서 아래를 사용할 수 있다:
  `m := (*map[int]int)(atomic.LoadPointer(&ptr))`
- [ ] 다수의 순차적 읽기 또는 쓰기(sequential reads or writes)를 수행 할 경우 buffered I/O를 사용.
  - syscalls의 수를 감소하기 위해서
- [ ] 맵(map)을 클리어(clear)하는 2가지 방법이 있다:
  - 맵 메모리 재사용(reuse map memory)

```go
	for k := range m {
		delete(m, k)
	}
```

  - 새로운 할당(allocate new)

```go
	m = make(map[int]int)
```

### Modules

- [ ] CI에서 `go.mod` (및 `go.sum`)가 최신(Up to date)인지 테스트하려면:
  https://blog.urth.org/2019/08/13/testing-go-mod-tidiness-in-ci/

### Build

- [ ] 이 명령으로 바이너리(binaries)를 제거(strip)해라: `go build -ldflags="-s -w" ...`
- [ ] 테스트를 다른 빌드로 나누는 쉬운 방법
  - `// +build integration`를 사용하고, 이를 `go test -v --tags integration .`과 함께 실행(run)하라
- [ ] 가장 작은 Go Docker image
  - https://twitter.com/bbrodriges/status/873414658178396160
  - `CGO_ENABLED=0 go build -ldflags="-s -w" app.go && tar C app | docker import - myimage:latest`
- [ ] CI상에서 `go format`을 실행하고, diff을 비교할 것
  - 이로 인해 모든것이 생성되고 커밋되는 것을 확인한다(ensure).
- [ ] 최신의 Go와 함께 Travis-CI를 실행하기 위해서 `travis 1`을 사용할 것
  - 추가 정보: https://github.com/travis-ci/travis-build/blob/master/public/version-aliases/go.json
- [ ] 코드 포멧팅(code formatting)상 실수가 있는지 체크 할 것 `diff -u <(echo -n) <(gofmt -d .)`

### Testing

- [ ] 테스트에서 `package`보다 `package_test`의 패키지 이름을 선호한다.
- [ ] `go test -short`는 테스트의 집합(set of tests)이 재실행(rerun)되는 것을 줄여준다.

```go
  func TestSomething(t *testing.T) {
    if testing.Short() {
      t.Skip("skipping test in short mode.")
    }
  }
```

- [ ] 아키텍쳐에 따라 테스트를 건너 뛸 것

```go
  if runtime.GOARM == "arm" {
    t.Skip("this doesn't work under ARM")
  }
```

- [ ] `testing.AllocsPerRun`를 통해서 할당(allocations)을 추적(track)할 것.
  - https://godoc.org/testing#AllocsPerRun
- [ ] 노이즈를 줄이기 위해, 벤치마크(benchmarks)를 여러번 수행할 것.
  - `go test -test.bench=. -count=20`

### Tools

- [ ] 빠른 대체: `gofmt -w -l -r "panic(err) -> log.Error(err)" .`
- [ ] `go list`는 직접적이고 전이적인(direct and transitive) 모든 의존성(dependencies)를 찾을 수 있다.
  - `go list -f '{{ .Imports }}' package`
  - `go list -f '{{ .Deps }}' package`
- [ ] 빠른 벤치마크 비교를 위해서, `benchstat` 툴(tool) 사용
  - https://godoc.org/golang.org/x/perf/cmd/benchstat
- [ ] [go-critic](https://github.com/go-critic/go-critic) 린터(linter)는 이 문서로부터 몇 가지 조언(several advice)를 수행한다.
- [ ] `go mod why -m <module>`는 `go.mod` 파일에 왜 특정 모듈이 있는지 말해준다.
- [ ] `GOGC=off go build ...`는 여러분들의 빌드(builds)의 속도를 높여준다. [source](https://twitter.com/mvdan_/status/1107579946501853191)
- [ ] 메모리 프로파일러는 매 512KB당 하나의 할당을 기록한다. Profile에서 세부사항을 보기 위해서 `GODEBUG`의 환경 변수를 통해서 이 비율(rate)를 높일 수 있다.
  - by https://twitter.com/bboreham/status/1105036740253937664

### Misc

- [ ] 고루틴을 버려랴(dump goroutines) https://stackoverflow.com/a/27398062/433041

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

- [ ] 컴파일 중 인터페이스 구현을 확인할 것

  ```go
  var _ io.Reader = (*MyFastReader)(nil)
  ```

- [ ] 만약 len의 매개변수(param)이 `nil`일 경우, 이는 "zero"다.
  - https://golang.org/pkg/builtin/#len
- [ ] 익명 구조체(anonymous structs)는 멋지다(cool)

```go
  var hits struct {
    sync.Mutex
    n int
  }
  hits.Lock()
  hits.n++
  hits.Unlock()
```

- [ ] `httputil.DumpRequest`는 상당히 유용하다, 그러므로 여러분들의 별도의 것을 만들지 말 것.
  - https://godoc.org/net/http/httputil#DumpRequest
- [ ] 스택을 얻기 위해서 `runtime.Caller`가 있다 https://golang.org/pkg/runtime/#Caller
- [ ] 임의의 JSON(arbitrary JSON)을 마샬링(to marshal)하기 위해, `map[string]interface{}{}`로 먀샬링(marshal)할 수 있다.
- [ ] `CDPATH`를 구성하면, 모든 director로부터 `cd github.com/golang/go`을 할 수 있다.
  - 이 라인을 여러분들의 `bashrc`(or analogue)에 추가하라: `export CDPATH=$CDPATH:$GOPATH/src`
- [ ] 하나의 슬라이스로부터의 간단한 임의 원소(simple random element)
  - `[]string{"one", "two", "three"}[rand.Intn(3)]`
