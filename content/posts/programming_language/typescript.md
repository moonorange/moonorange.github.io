---
title: 'Typescript Basic'
date: '2025-11-22'
categories: ["Programming Language", "TypeScript"]
tags: ["TypeScript", "English Article"]
---

## Introduction

Random notes about TypeScript

## What is TypeScript?

TypeScript is a typed superset of JavaScript meaning TypeScript is JavaScript’s runtime with a compile-time type checker.

It preserves the runtime behavior of JavaScript meaning you can move code from JavaScript to TypeScript even if that the code has type errors.

## Declarations

`var`: declares a variable

`let`: block-scoped local variable

`const`: block-scoped, read-only named constant

### Examples

**Scope Difference:**

```typescript
function varExample() {
  if (true) {
    var x = 10;
  }
  console.log(x); // 10: var is function-scoped
}

function letExample() {
  if (true) {
    let y = 10;
  }
  console.log(y); // Error: y is not defined, let is block-scoped
}
```

**Re-declaration:**

```typescript
var a = 1;
var a = 2; // OK: var allows re-declaration

let b = 1;
let b = 2; // Error: Cannot redeclare 'b'

const c = 1;
const c = 2; // Error: Cannot redeclare 'c'
```

**Re-assignment:**

```typescript
var x = 1;
x = 2; // OK

let y = 1;
y = 2; // OK

const z = 1;
z = 2; // Error: Cannot assign to 'z' because it is a constant
```

**Hoisting Behavior:**

```typescript
console.log(a); // undefined: var is hoisted
var a = 5;

// let/const are also hoisted, but they're in a "Temporal Dead Zone" (TDZ)
// Accessing them before declaration throws a ReferenceError
console.log(b); // Error: Cannot access 'b' before initialization
let b = 5;

console.log(c); // Error: Cannot access 'c' before initialization
const c = 5;
```

**Loop Example:**

```typescript
// var: all callbacks share the same variable
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 3, 3, 3
}

// let: each iteration has its own binding
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 100); // 0, 1, 2
}
```

## Ref

https://www.typescriptlang.org/docs/handbook/typescript-from-scratch.html
