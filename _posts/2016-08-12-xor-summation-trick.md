---
layout: "post"
title: "XOR over a range in O(1)"
date: "2016-08-12 22:14"
tags:
- algorithms
---

To calculate XOR of integers (i...j), basic approach is to iterate over the range. Intuition says that there must be some pattern involved as numbers are consecutive and turns out there is.

Lets write the binary table

| Numbers        | XOR            | Comment |
| -------------: | -------------: | --- |
| 0 | 0 |
| 1 | 1 |
| 10  | 11 |
| 11  | 00 | Returns to 0
| 100 | 100 | `0 ⊕ n = n` Here, n ends with 0, so next value will have only last digit changing
| 101 | 001 | Only last digit changes to 1, XOR value will be 1
| 110 | 111 | `b ⊕ 1 = ~b` Thus, last digit gets inverted to 1. This is like adding 1. Thus, value is (n+1)
| 111 | 000 | `n ⊕ n = 0`. Thus value returns to 0.
| 1000  | 1000 | n
| 1001  | 0001 | 1
| 1010  | 1011 | n+1
| 1011  | 0000 | 0
| 1100  | 1100 | n
| 1101  | 0001 | 1
| 1110  | 1111 | n+1
| 1111  | 0000 | 0
| 10000 | 10000 | n

and so on.

This can be verified with a script:

```swift
func xorOptimised(n: Int) -> Int {
    switch n % 4 {
    case 0:
        return n
    case 1:
        return 1
    case 2:
        return n+1
    case 3:
        return 0
    default:
        return -1 // Never happens, but oh Swift
    }
}

func xorStandard(n: Int) -> Int {
    return (1...n).reduce(0, combine: ^)
}

for i in 1...10000 {
    assert(xorOptimised(i) == xorStandard(i))
}

print("Success")
```

And it works :)

Based on this, we can build a method for computing XOR for a range:

```swift
func xorRange(start: Int, _ end: Int) -> Int {
    return xorOptimised(end) ^ xorOptimised(start-1)
}
```
And we have a method to compute XOR of a range in O(1)

This was more of an empirical observation. It would be interesting to derive it mathematically.

> Perhaps there is already a derivation out there with a fancy name
