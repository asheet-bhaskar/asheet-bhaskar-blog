---
title: "Context About Context Package in Golang"
date: 2021-05-29T09:37:32+05:30
draft: false
summary: This post is about the context package of golang and how to get started with it. We'll also disucss the various use cases of it. After reading this blog user should have comprehensive understanding of context package of golang. 
featuredImage: /images/context.jpg
featuredImage: /images/golang-context.png
theme: "full"
tags: ["golang", "context"]
categories: ["golang"]
summaryStyle:
    hiddenImage: false
    hiddenDescription: false
    hiddenTitle: true
    tags:
      theme: "image"
      color: "white"
      background: "black"
      transparency: 0.9
---

This post is about the context package of golang and how to get started with it. We'll also disucss the various use cases of it. After reading this blog user should have comprehensive understanding of context package of golang. 

### Pre-requisites

- **goroutines**: goroutines are the light weight threads which can be used to execute functions concurrently with other functions. You can read more about goroutines at [link](https://tour.golang.org/concurrency/1)
- **channels**: channels can be thought of as a pipe that connect multiple concurrent goroutines. Read more about channels at [link](https://tour.golang.org/concurrency/2)
- **defer**: A defer statement postpones the execution of a function until the surrounding function returns. Read more at [link](https://tour.golang.org/flowcontrol/12)

### Why should we use [context](https://golang.org/pkg/context/)

Let's take the example of following timeline for any action.
- action starts at time instant t0.
- action completes at time instant t5.
- this action initiates three different goroutines namely, goroutine 1, goroutine 2 and goroutine 3 etc.
- goroutine 1 starts and completes at t1 and completes at t'1.
- goroutine 2 starts and completes at t2 and completes at t'2.
- goroutine 3 starts and completes at t3 and completes at t'3.

![Context-golang](/images/golang-context.png)

Golang enables us to initiate goroutines to perform different parts of an action concurrently. If due to some unavoidable reason if above action has to be terminated at time instant t4, It's importnat to terminate goroutines 2 and 3 as well in order to avoid the sideeffects that'll be caused by goroutines 2 and 3. *Context* provides the ability to time-out and terminate goroutines to handle such cases.


### Using context | Examples


#### Example 1:  [run](https://play.golang.org/p/oTZHJZNVgzn)

```go

  func operationOne(ctx context.Context) {
    n := 1
    for {
      select {
      case <-ctx.Done():
        fmt.Println("context canceled for op-1")
        return // returning not to leak the goroutine
      default:
        fmt.Printf("OperationOne: %d\n", n)
        time.Sleep(500 * time.Millisecond)
        n++
      }
    }
  }

  func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    operationOne(ctx)
  }

```

##### Observation
- function `operationOne` is not being called in a separate goroutine, it's blocking call and main funtion won't exits util operationOne returns.
- function `operationOne` is being passed context as argument.
- context is not being cancelled or being mark as Done.
- operationOne will never return as context is not being cancelled at all.


#### Example 2: [run](https://play.golang.org/p/XNIQxgrNe6T)

```go

  func operationOne(ctx context.Context) {
    n := 1
    for {
      select {
      case <-ctx.Done():
        fmt.Println("context canceled for op-1")
        return // returning not to leak the goroutine
      default:
        fmt.Printf("OperationOne: %d\n", n)
        time.Sleep(500 * time.Millisecond)
        n++
      }
    }
  }

  func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    go operationOne(ctx)
    time.Sleep(5 * time.Second)
  }

```

##### Observation
- function `operationOne` is  being called in a separate goroutine.
- function `operationOne` is being passed context as argument.
- main function will return after 5 seconds from the start of execution.
- context will cancelled at return of main function.


#### Example 3: [run](https://play.golang.org/p/rHnsEO9UvGQ) 

```go

  func operationOne(ctx context.Context) {
    n := 1
    for {
      select {
      case <-ctx.Done():
        fmt.Println("context canceled for op-1")
        return // returning not to leak the goroutine
      default:
        fmt.Printf("OperationOne: %d\n", n)
        time.Sleep(500 * time.Millisecond)
        n++
      }
    }
  }

  func operationTwo(ctx context.Context) {
    n := 1
    for {
      select {
      case <-ctx.Done():
        fmt.Println("context canceled for op-2")
        return // returning not to leak the goroutine
      default:
        fmt.Printf("OperationTwo: %d\n", n)
        time.Sleep(250 * time.Millisecond)
        n++
      }
    }
  }

  func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    d := time.Now().Add(5000 * time.Millisecond)
    ctx, cancel = context.WithDeadline(context.Background(), d)
    defer cancel()
    go operationOne(ctx)

    d = time.Now().Add(10000 * time.Millisecond)
    ctx, cancel = context.WithDeadline(context.Background(), d)
    defer cancel()
    go operationTwo(ctx)

    time.Sleep(20 * time.Second)
  }

```

##### Observation
- function *operationOne* and `operationTwo` are  being called in a separate goroutines.
- drived context with differenty deadlines are being passed to *operationOne* and `operationTwo` functions.
- main function will return after 20 seconds from the start of execution.
- function `operationOne` will return after 5 seconds as deadline of context passed to it will be reached.
- function `operationTwo` will return after 10 seconds as deadline of context passed to it will be reached.
- main function will return only after 20 seconds even though *operationOne* and `operationTwo` will be completed in first 10 seconds.


#### Example 4: [run](https://play.golang.org/p/jV30esabmo6) 

```go

  func operationOne(ctx context.Context) {
    n := 1
    for {
      select {
      case <-ctx.Done():
        fmt.Println("context canceled for op-1")
        return // returning not to leak the goroutine
      default:
        fmt.Printf("OperationOne: %d\n", n)
        time.Sleep(500 * time.Millisecond)
        n++
      }
    }
  }

  func operationTwo(ctx context.Context) {
    n := 1
    for {
      select {
      case <-ctx.Done():
        fmt.Println("context canceled for op-2")
        return // returning not to leak the goroutine
      default:
        fmt.Printf("OperationTwo: %d\n", n)
        time.Sleep(250 * time.Millisecond)
        n++
      }
    }
  }

  func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    d := time.Now().Add(5000 * time.Millisecond)
    ctx, _ = context.WithDeadline(context.Background(), d)
    go operationOne(ctx)

    d = time.Now().Add(10000 * time.Millisecond)
    ctx, _ = context.WithDeadline(context.Background(), d)
    go operationTwo(ctx)

    time.Sleep(3 * time.Second)
  }

```
##### Observation

- function *operationOne* and `operationTwo` are  being called in a separate goroutines.
- drived context with differenty deadlines are being passed to *operationOne* and `operationTwo` functions.
- function `operationOne` will return after 5 seconds as deadline of context passed to it will be reached.
- function `operationTwo` will return after 10 seconds as deadline of context passed to it will be reached.
- main function will return after first 3 seconds.

##### what will happen to other two goroutines?
- This is a case of context leak. even after return of main function other two drived contexts passed to operationOne* and `operationTwo` will not be discarded. To avoid context leak it's a good practice to use `defer cancel()` even for drived contexts


#### Example 5: [run](https://play.golang.org/p/d6ctLQiukYk) 

```go

  type Key string

  func operationOne(ctx context.Context) {
    n := 1
    for {
      select {
      case <-ctx.Done():
        fmt.Printf("context canceled for %s\n", ctx.Value(Key("op_id")))
        return // returning not to leak the goroutine
      default:
        fmt.Printf("OperationOne: %d : opeartion_id = %s\n", n, ctx.Value(Key("op_id")))
        time.Sleep(500 * time.Millisecond)
        n++
      }
    }
  }

  func operationTwo(ctx context.Context) {
    n := 1
    for {
      select {
      case <-ctx.Done():
        fmt.Printf("context canceled for %s\n", ctx.Value(Key("op_id")))
        return // returning not to leak the goroutine
      default:
        fmt.Printf("OperationTwo: %d : opeartion_id = %s\n", n, ctx.Value(Key("op_id")))
        time.Sleep(250 * time.Millisecond)
        n++
      }
    }
  }

  func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    d := time.Now().Add(5000 * time.Millisecond)
    ctx, cancel = context.WithDeadline(context.Background(), d)
    defer cancel()
    ctx = context.WithValue(ctx, Key("op_id"), "ONE")
    go operationOne(ctx)

    d = time.Now().Add(10000 * time.Millisecond)
    ctx, cancel = context.WithDeadline(context.Background(), d)
    defer cancel()
    ctx = context.WithValue(ctx, Key("op_id"), "TWO")
    go operationTwo(ctx)

    time.Sleep(20 * time.Second)
  }

```


##### Observation
- function *operationOne* and `operationTwo` are  being called in a separate goroutines.
- drived context with differenty deadlines are being passed to *operationOne* and `operationTwo` functions.
- a variable `op_id` and it's value is also being passed with drived context in both the cases.
- main function will return after 20 seconds from the start of execution.
- function `operationOne` will return after 5 seconds as deadline of context passed to it will be reached.
- function `operationTwo` will return after 10 seconds as deadline of context passed to it will be reached.
- main function will return only after 20 seconds even though *operationOne* and `operationTwo` will be completed in first 10 seconds.


#### Example 6: [run](https://play.golang.org/p/47a0qVAflzm) 

```go

  type Key string

  func operationOneChild(ctx context.Context) {
    for {
      select {
      case <-ctx.Done():
        fmt.Printf("context canceled for %s\n", ctx.Value(Key("op_id")))
        return // returning not to leak the goroutine
      default:
        fmt.Println("Child of operation one")
        time.Sleep(100 * time.Millisecond)
      }
    }

  }

  func operationOne(ctx context.Context) {
    n := 1
    // go operationOneChild(context.WithValue(ctx, Key("op_id"), "CHILD OF ONE"))
    go operationOneChild(nil)
    for {
      select {
      case <-ctx.Done():
        fmt.Printf("context canceled for %s\n", ctx.Value(Key("op_id")))
        return // returning not to leak the goroutine
      default:
        fmt.Printf("OperationOne: %d : opeartion_id = %s\n", n, ctx.Value(Key("op_id")))
        time.Sleep(500 * time.Millisecond)
        n++
      }
    }
  }

  func operationTwo(ctx context.Context) {
    n := 1
    for {
      select {
      case <-ctx.Done():
        fmt.Printf("context canceled for %s\n", ctx.Value(Key("op_id")))
        return // returning not to leak the goroutine
      default:
        fmt.Printf("OperationTwo: %d : opeartion_id = %s\n", n, ctx.Value(Key("op_id")))
        time.Sleep(250 * time.Millisecond)
        n++
      }
    }
  }

  func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    d := time.Now().Add(5000 * time.Millisecond)
    ctx, cancel = context.WithDeadline(context.Background(), d)
    defer cancel()
    ctx = context.WithValue(ctx, Key("op_id"), "ONE")
    go operationOne(ctx)

    d = time.Now().Add(10000 * time.Millisecond)
    ctx, cancel = context.WithDeadline(context.Background(), d)
    defer cancel()
    ctx = context.WithValue(ctx, Key("op_id"), "TWO")
    go operationTwo(ctx)

    time.Sleep(20 * time.Second)
  }

```

##### Observation
- function *operationOne* and `operationTwo` are  being called in a separate goroutines.
- drived context with differenty deadlines are being passed to *operationOne* and `operationTwo` functions.
- function `operationOne` will return after 5 seconds as deadline of context passed to it will be reached.
- function `operationTwo` will return after 10 seconds as deadline of context passed to it will be reached.
- main function will return only after 20 seconds even though *operationOne* and `operationTwo` will be completed in first 10 seconds.
- nil drived context is being passed to function `operationOneChild` which will panic with following result.

```

  panic: runtime error: invalid memory address or nil pointer dereference
  [signal SIGSEGV: segmentation violation code=0xffffffff addr=0x0 pc=0xf3966]

  goroutine 9 [running]:
  main.operationOneChild(0x0, 0x0)
    /tmp/sandbox968352470/prog.go:14 +0xa6
  created by main.operationOne
    /tmp/sandbox968352470/prog.go:28 +0x40

```


#### Example 7: [run](https://play.golang.org/p/gsWBnLDl7Bm)

```go

  type Key string

  func operationOneChild(ctx context.Context) {
    for {
      select {
      case <-ctx.Done():
        fmt.Printf("context canceled for %s\n", ctx.Value(Key("op_id")))
        return // returning not to leak the goroutine
      default:
        fmt.Println("Child of operation one")
        time.Sleep(100 * time.Millisecond)
      }
    }

  }

  func operationOne(ctx context.Context) {
    n := 1
    go operationOneChild(context.WithValue(ctx, Key("op_id"), "CHILD OF ONE"))
    // go operationOneChild(nil)
    for {
      select {
      case <-ctx.Done():
        fmt.Printf("context canceled for %s\n", ctx.Value(Key("op_id")))
        return // returning not to leak the goroutine
      default:
        fmt.Printf("OperationOne: %d : opeartion_id = %s\n", n, ctx.Value(Key("op_id")))
        time.Sleep(500 * time.Millisecond)
        n++
      }
    }
  }

  func operationTwo(ctx context.Context) {
    n := 1
    for {
      select {
      case <-ctx.Done():
        fmt.Printf("context canceled for %s\n", ctx.Value(Key("op_id")))
        return // returning not to leak the goroutine
      default:
        fmt.Printf("OperationTwo: %d : opeartion_id = %s\n", n, ctx.Value(Key("op_id")))
        time.Sleep(250 * time.Millisecond)
        n++
      }
    }
  }

  func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    d := time.Now().Add(5000 * time.Millisecond)
    ctx, cancel = context.WithDeadline(context.Background(), d)
    defer cancel()
    ctx = context.WithValue(ctx, Key("op_id"), "ONE")
    go operationOne(ctx)

    d = time.Now().Add(10000 * time.Millisecond)
    ctx, cancel = context.WithDeadline(context.Background(), d)
    defer cancel()
    ctx = context.WithValue(ctx, Key("op_id"), "TWO")
    go operationTwo(ctx)

    time.Sleep(20 * time.Second)
  }

```

##### Observation
- function *operationOne* and `operationTwo` are  being called in a separate goroutines.
- drived context with differenty deadlines are being passed to *operationOne* and `operationTwo` functions.
- function `operationOne` will return after 5 seconds as deadline of context passed to it will be reached.
- function `operationTwo` will return after 10 seconds as deadline of context passed to it will be reached.
- main function will return only after 20 seconds even though *operationOne* and `operationTwo` will be completed in first 10 seconds.
- context which is being passed to `operationOneChild` is non nil, non drived context. This child go routine will also be terminated on cancellation of parent context.

### Things to keep in mind while using the context

* Incoming requests to a server should create a Context.
* Outgoing calls to servers should accept a Context.
* The chain of function calls between them must propagate the Context.
* Replace the current Context with the derived Context.
* Do not store Contexts inside a struct type; instead, pass a Context explicitly to each function that needs it.
* Do not pass a nil Context, even if a function permits it. Pass context.TODO if you are unsure about which Context to use. 
* The same Context may be passed to functions running in different goroutines; Contexts are safe for simultaneous use by multiple goroutines. 


You can use use run link for each example to run these in golang playground. After reading this blog user should have comprehensive understanding of context package of golang.
If you have any suggestions about plugins to use, please let me know in comments.


