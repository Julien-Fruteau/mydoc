# recursion

A function is composed of:

    a) return address; the returned address make the recurse kind of 'bi directionnal':
        with foo(5),
        going down from 5 to 1,
        then going up from 1 to 5
        with the return address
        this is highlighted when printing the n value
    b) a returned value
    c) arguments

Recurse steps/tools to keep in mind, because "🙋 friends don't let friends recurse without the toolbox":

    1) pre
    2) recurse
    3) post (print again), optional

Always start implementing recursion with the **base case**, defining condition(s) to stop recursion.

```go
func foo(n int) int {
  // 1) Base case step
 if n == 1 {
    fmt.Println("this is 1")
  return 1
 }

  // 2) Pre step
 println(n)
 
  // 3) Recurse step
 out := n + foo(n-1)

  // 4) Post step
 println(n, " again")
 return out
}

func main(){
  fmt.Println(foo(5))
}

```

Prints:

```txt
5
4
3
2
this is 1
2  again
3  again
4  again
5  again
15
```
