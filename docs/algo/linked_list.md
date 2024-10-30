---
title: linked list
tags:
  - algo
  - linked-list
---

## API interface

```js
interface LinkedList<T> {
  get length(): number;
  insertAt(item T, index: number): void;
  remove(item T): T | undefined;
  removeAt(index: number): T | undefined;
  append(item: T): void;
  prepend(iten: T): void;
  get(index: number): T | undefined;
}
```
