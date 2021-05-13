---
title: "Value vs Reference in Javascript: A complete Guide"
date: 2020-05-16T10:12:03+02:00
layout: post
---
A better understanding of the value and reference concept, can help a developer writing better code and avoiding bugs, due to unexpected side effects.

In this article I'll explain the concept of value and reference in Javascript, but you should be able to understand from here, how it works in your favorite programming language.

## Immutables and Mutables
In programming, a variable can either be an immutable or a mutable.
A mutable can be changed after it's created, while an immutable can't.
Let's see what that concretely means.

### Immutable
Immutable types in JS, are: `String`, `Number`, `Boolean`, `undefined` and `null`.

Immutable, means that any operation on the variable, will result into a new variable ; keeping the original one unchanged.

```javascript
let a = 'john';
let b = a;
console.log(a, b) // 'john', 'john'

b = 'jack';
console.log(a, b) // 'john', 'jack'
```
At first, we assign `a` to `b`. They both now contain, the same value: `john`.

Then, we can see that, after changing `b`, `a` does not change.
That's because when we do `b = a`, `b` is now a new String containing the same value as `a`. We say that `a` is copied to `b`, by value.
Hence, changing one does not change the other, as they're separate variables.

Immutables are copied by value.

### Mutable
Mutable types in JS, are : `Object` and `Array`.

Mutable means that any change on the variable, will affect other variables pointing at it.

```javascript
let a = [0];
let b = a;
console.log(a, b) // [0], [0]

b.push(1);
console.log(a, b) // [0,1], [0,1]
```
At first, we assign `a` to `b`. They both now contain, the same value: `[0]`.

Then, we can see that changing `b`, also changes `a` automatically. That's because `b` does not contain a new value ; instead, it points to the same memory address as `a`. We say that `a` is copied to `b`, by reference.
Hence, changing one, changes the other, as they're the same variable.

Mutables are copied by reference.

### Comparison
In javascript we check if two variables are the same, using: `==` or `===`.

For immutables, they both mean the same.
```javascript
let a = 2;
let b = 2;
console.log(a == b, a === b); // true, true
```
Things get more interresting with mutables:
- `==` checks if the variables values are the same
- `===` checks if the variables values and references are the same. In other words, it checks if both variables point to the same address in memory.
```javascript
let a = { name: 'john' };
let b = { name: 'john' };
console.log(a == b, a === b);  // true, false

let a = { name: 'john' };
let b = a;
console.log(a == b, a === b); // true, true
```
For immutables, `==` will be `true` if the variables hold the same values ;  and `===` will be true if the variables point to the same `reference`.

## Copying a mutable by value
By default, an immutable is copied by reference ; meaning that changing the original or the copy variable, changes the other one.

What if, we want to copy only the immutable by value. That means, being able to modify the original or the copy, wihout changing the other one.

Let's see how to achieve that:
```javascript
let a = { name: 'john' }
let b = JSON.parse(JSON.stringify(a))

console.log(a == b, a === b) // true, false
```
We turn the `a` object (or array) into a string, and then create a new object from that string. Thus, `b` contains a new shiny object (or array), with the same value as `a`.

We now have two separated variables, and we can go ahead and modify each of them without worrying about the other one being affected.

Mutable types in JS are Object and Array.

Another "copy mutable by value method", is to iterate over the object keys or the array indexes, to create a new one from scratch.

```javascript
// Copy object by value with iteration
let a = { name: 'john', age: 18 };
let b = {};

Object.keys(a).forEach((key) => {
  b[key] = a[key];
})
console.log(a == b, a === b); // true, false


// Copy array by value with iteration
let a = [0, 1, 2, 3];
let b = [];

a.forEach((index) => {
  b[index] = a[index];
})
console.log(a == b, a === b); // true, false
```

## Functions
Passing arguments to functions, work the same way as copying variables.

That means an immutable argument, is passed by value ; while a mutable argument is passed by reference.

### Immutable argument

```javascript
function ageize(agent, age) {
  agent.age = age;
  return agent;
}

let agent = { name: 'john' };
let ageizedAgent = ageize(agent, 18);

console.log(agent, ageizedAgent) // { agent: 'john', age: 18 }, { agent: 'john', age: 18 }
```
Changing the `agent` argument, inside the `ageize` function, also changes the original `agent` object outside of the function, because it's been copied by reference to the function.

We say that the `ageize` function creates a `side effect` ; because it modifies data outside of its scope.

Such function is also called a `non pure` or `impure` function, opposed to a `pure function` which is one, that does not create any side effect.

### Mutable argument
```javascript
function agentize(name) {
  name = `Agent ${name}`;
  return name;
}

let name = 'John';
let agentizedName = agentize(name);

console.log(name, agentizedName) // 'John', 'Agent John';
```
Changing the `name` argument inside the `agentize` function, does not affect the original `name` variable outside of the function, because it's been copied by value to the function.


## Wrapping Up!
- If copied by value, a variable change does not affect the other. If copied by reference, a variable change, affects the other.
- Imutable types in JS are : `String`, `Number`, `Boolean`, `undefined` and `null`.
- Mutable types in JS are : `Object` and `Array`
- Immutables are copied by value
- Mutables are copied by reference
- `==` checks the variables values, `===` checks the variable values and references
- It's possible to copy mutables by value, using different techniques.
- Functions argument passing work the same way as variable copying

