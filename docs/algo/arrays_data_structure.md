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

```golang
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

TODO

```golang
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

## O(square_root(n)) search

example use case :

- given 2 cristal balls that will break if dropped from high enough distance, determine the exact spot in wich it will break in the most optimized way

- binary search consideration : the fact that we have 2 balls, i.e meaning we could only have one single return true in the test v === needle. then the rest of the search tree will have to be linear search

- instead _lienear search_ by jumping square_root of N (length): on the 1st hit, a ball is lost, but we start back from that hi index minus square_root(N)

```golang

```
