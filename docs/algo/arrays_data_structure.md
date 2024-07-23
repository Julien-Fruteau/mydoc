---
title: arrays data structure
tags:
  - arrays
  - data-structures
  - algo
  - frontend-masters
---

## Linear Search
u8 qq

### Golang implementation

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

### Pseudo code

TODO

### Golang implementation

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
