# AOJSGuide
An guide to JavaScript

## What is JavaScript?

A programming language. A compiled programming language. What does that mean? When you write some JavaScript code to run in a web browser or any other JavaScript environment, it first needs to be compiled before it gets executed. Let's say you've written some code and you run it in Google's Chrome web browser, firstly the JavaScript engine that Chrome uses - v8 - compiles the code before running it. In this compilation stage a lot happens to your code, including lots of micro optimisations, but the key thing to be aware of for JavaScript developers is that this is the time at which scope is defined.

## Scope
Scope refers to variable scope. When you declare a variable in JavaScript it is scoped based on its location within your script at author time. This is known as lexical scoping - scoping at the compiler stage. Pre ES6 variables were scoped either to the global scope or to a function scope. 

```javascript
// global scope
var foo = "foo";

function bar() {
  // function scope
  var foo = "bar";
}
```

Scope dictates a variables availability. If a variable is defined in the global scope, it is available globally, throughout all your code. If a variable is defined in a function it is available only within that function.

```javascript
function foo() {
  var bar = "bar";
}

// error 
// bar is not available outside of the function scope
console.log(bar);
```

When you write JavaScript you declare things either as variables or as functions. The compiler determines function scope the same way as it does variable scope, so functions declared within functions are not available externally. Functions can be written either as declarations or expressions, the difference between the two is in the way the compiler handles them, which we'll get to shortly.

```javascript
// declaration
function bar() {
}

// expression
var foo = function() {
}

```

ES6 introduced two new ways for variables to be declared; <code>const</code> and <code>let</code>. These both have different scoping criteria to var or function. Const and let are block scoped which means they are available only within the block <code>{ }</code> they are declared in.

```javascript
for (let i = 0; i < 5l i++) {
  // i available within
}

// error
// i is scoped to the for loop block and not available externally
console.log(i);
```

If the above was declared with var it would be available outside of the block. Let and const are also handled differently to var and function by the compiler, which we'll cover shortly.

### Summary
Scope dictates the availablity of variables within your code. Variables can be scoped globally, within a function or within a block. Variable scope is determined in the compile stage, before the code is executed. Scope determined at compile time is known as lexical scoping.

## Hoisting
The compiler hoists variables and functions in your code to the top of the scope they are declared in. Take the following code:

```javascript
// pre compiler
var a = b();
console.log(a);
function b() {
  return "foo";
}

```

The compiler runs through the code and hoists the variables and functions to the top of the parent scope (in this case, global), so when the code is executed at run time it will look like this:

```javascript

// after compiler
function b() {
  return "foo";
}
var a;
a = b();
console.log(a); // foo
```

Earlier it was mentioned that function declarations and expressions are treated differently by the compiler. Notice in the above example how the function declaration is hoisted to the top of the scope in its entirety. A function expression would be treated differently. Here's the same example again but using a function expression.

```javascript
// pre compiler
var a = b();
console.log(a);
var b = function() {
  return "foo";
}

// post compiler
var a;
var b;

// error
// b is not a function and is at this time undefined
a = b();
b = function() {
  return "foo";
}

console.log(a);
```

At runtime the above example would attempt to call <code>b</code> as a function before the function is assigned as its value, resulting in an error. 

Let and const are not hoisted like var and function. Take the following example:

```javascript
// pre compiler
function foo() {
  console.log(x);
  console.log(y);
  var x = 1;
  let y = 2;
}

// post compiler
function foo() {
  var x;
  console.log(x); // undefined
  
  // error y is not definied
  console.log(y);
  x = 1;
  let y = 2;
}
```

After the compilation stage the <code>x</code> variable declared with var is hoisted but the <code>y</code> variable declared with let is not. When executed x has been declared but not yet given its value, y however has not been declared and will throw an error. The area above the let declaration is known as the <code>temporal dead zone</code>.

### Summary

Variables declared with var or function are hoisted to the top of their parent scope. Variables are declared but are not assigned a value. This happens at run time, not compile time. Function declarations however are assigned their value which is the function they represent. Function expressions are not assigned a value and are treated like a variable. Their value is likewise assigned at runtime. Variables declared with let and const are not hoisted and any attempt to reference them before their value is assigned will result in an error. This area before they are assigned a value is known as the temporal dead zone.

## Context






