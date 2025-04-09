---
id: arrays_data_structure
aliases: []
tags:
  - arrays
  - data-structures
  - algo
  - frontend-masters
area: ""
project: ""
title: arrays data structure
---

## Linear Search

```go
func linearSearch(haystack []int, needle int) bool {
  for _, v := range haystack {
    if v == needle {
      return true
    }
  }
 return false
}
```

## Binary Search List

⚠️ hypothesis: the list must be sorted, ascending

```go
func binarySearch(haystack []int, needle int) bool {
  lo := 0
  hi := len(haystack)
  for lo < hi {
    m := lo + (hi - lo) / 2
    v := haystack[m]
    if v == needle {
      return true
    } else if needle > v {
      lo = m + 1
    } else {
      hi = m
    }
  }
  return false
}
```

## O($\sqrt{N}$) search

Example use case :

- given 2 crystal balls that will break if dropped from high enough distance, determine the exact spot in which it will break in the most optimized way

- binary search consideration : the fact that we have 2 balls, i.e meaning we could only have one single return true in the test v === needle. then the rest of the search tree will have to be linear search

- instead _linear search_ by jumping square_root of N (length): on the 1st hit, a ball is lost, but we start back from that hi index minus square_root(N)

```go
func two_cristal_balls(breaks []bool) (v int) {
  jumpAmout := int(math.Sqrt(float64(len(breaks))))
  i := 0

  // search jumping by N^1/2
  for ; i < len(breaks); i += jumpAmout {
    if breaks[i] {
    break
    }
  }

  // then search linearly up to N^1/2 at most
  i -= jumpAmout
  for j := 0; j < jumpAmout && i < len(breaks); j++ {
      if breaks[i] {
      return i
    }
    i++
  }
  return -1
}
```
