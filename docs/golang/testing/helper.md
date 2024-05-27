---
title: helper
tags:
  - go
  - golang
  - testing
  - helper
  - subtest
---

## helper

- a helper function in a test file helps reuse code by marking it as a helper so that it is not considered by `go test`

```go
package main

import "testing"

func assertOk (t testing.TB, got, want int){
  t.Helper()

  if got != want {
      t.Fatalf("want %v, got %v", want, got)
  }
}

func TestFoo(t *testing.T) {
  got:= Foo(10)
  want:= 42

  assertOk(t, got, want)
}
```

## subtest

- run a subtest of a function for clarity

```go
package main

import "testing"

func TestFoo(t *testing.T) {
  assertOk:= func(t testing.TB, got, want int){
    t.Helper()

    if got != want {
        t.Fatalf("want %v, got %v", want, got)
    }
  }

  t.Run("foo test case bar", func(t *testing.T) {
    got:= Foo(10)
    want:= 42

    assertOk(t, got, want)
  }

  t.Run("foo test case next", func(t *testing.T) {
    got:= Foo(100)
    want:= 42

    assertOk(t, got, want)
  }
}
```
