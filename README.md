# AOJSGuide
An guide to JavaScript

## What is JavaScript?

A programming language. A compiled programming language. What does that mean? When you write some JavaScript code to run in a web browser or any other JavaScript environment, it first needs to be compiled before it gets executed. Let's say you've written some code and you run it in Google's Chrome web browser, firstly the JavaScript engine that Chrome uses - v8 - compiles the code before running it. In this compilation stage a lot happens to your code, including lots of micro optimisations, but the key thing to be aware of for JavaScript developers is that this is the time at which scope is defined.

## Scope
Scope refers to variable scope. When you declare a variable in JavaScript it is scoped based on its location within your script. Pre ES6 variables were scoped either to the global scope or to a function scope. 

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

When you write JavaScript you declare things either as variables or as functions. The compiler determines function scope the same way as it does variable scope, so functions declared within functions are not available externally.

Functions can be written either as declarations or expressions;

```javascript
// declaration
function bar() {
}

// expression
var foo = function() {
}

```

The compiler treats these two in slightly different ways, the important thing to be aware of is the compiler converts declarations into expressions, meaning that everything you declare in JavaScript, be it variable or function, ultimately is converted to a variable.

ES6 introduced two new ways for variables to be declared; <code>const</code> and <code>let</code>. Both of these scope variables differently to a var or function. Const and let are block scoped which means they are available only within the block <code>{ }</code> they are declared in.

```javascript
for (let i = 0; i < 5l i++) {
  // i available within
}

// error
// i is scoped to the for loop block and not available externally
console.log(i);
```
If the above was declared with var it would be available outside of the block.

### Summary
Scope dictates the availablity of variables within your code. Variables can be scoped globally, within a function or within a block. Variable scope is determined in the compile stage, before the code is executed.


