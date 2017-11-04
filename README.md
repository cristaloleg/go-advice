# go-advices

- [ ] go fmt your code
  - make everyone happier
- [ ] do not omit `defer`
  - 200ns speedup is neglecatble in a most cases
- [ ] dump goroutines
  - https://stackoverflow.com/a/27398062/433041
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
- [ ] multiple if statements can be collapsed into switch
