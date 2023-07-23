---
layout: post
title: How to empty an array in JavaScript?
date: 2019-10-22 19:12:04
categories: UMUTech
tags:
- nodejs
---
When I write [pathfinding](https://blog.umu618.com/pathfinding/) codes in JavaScript, I ran into this problem: how to empty the array?

## Methods

- Method 1

```js
array = []
```

- Method 2

```js
array.length = 0
```

- Method 3

```js
array.splice(0, array.length)
```

- Method 4

```js
while (array.length > 0) {
    array.pop()
}
```

## Difference between Method 1 and the others

Method 1 only reassign a:

```
let a = [1, 2, 3]
let b = a

// empty a, but doesn't affect b
a = []

console.log('a =', a)
console.log('b =', b)
```

Method 2, 3, 4, will empty multiple variables referencing the same object:

```
let a = [1, 2, 3]
let b = a

// empty a and b
a.length = 0

console.log('a =', a)
console.log('b =', b)
```

## More

See <https://github.com/UMU618/js-empty-array#example>
