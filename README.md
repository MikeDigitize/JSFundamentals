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

Context in JavaScript is defined at runtime, after the compilation stage. Context refers to the value of the keyword <code>this</code>. Every function, when executing, has a reference to its execution context and this reference can be accessed through <code>this</code>. Context can be defined organically or artificially in JavaScript and can be used to great effect when creating coding patterns and APIs. Consider the following example:

```html
<button id="add" data-value="0">Add 1</button>
```

```javascript
var btn = document.querySelector("#add");
btn.addEventListener("click", function() {
  var count = this.getAttribute("data-value");
  this.setAttribute("data-value", ++Number(count));
}, false);
```

When using <code>addEventListener</code> from the DOM API, the context of <code>this</code> in the event handler is set as the element the event was triggered from. This is an extremely convenient design choice as often in an event handler you will need a reference to the trigger element to perform an update or retrieve a value. To understand how context is organically determined take the following example:

```javascript
function bar() {
  console.log(this.a);  // "bar"
}
var a = "bar";
bar();
```

### The default context rule

Context is defined based on the location the function was called from - known as the call site. In the above example the function <code>bar</code> is called in the global scope and attempts to log a variable attached to <code>this</code>. The variables it attempts to log - <code>a</code> - has been defined in the global scope. Since the function was called in the global scope, <code>this</code> refers to the global object and therefore has a reference to the variable <code>a</code>.

This is known as the default context rule. Be aware however that in strict mode, as this has not been explicitly set, it will throw an error. To see how the default rule changes according to the call site, take the following example:

```javascript
function bar() {
  console.log(this.a);  // "foo"
}
var a = "bar";
var foo = {
  a : "foo",
  b : bar
};
foo.b();
```

Now the bar function is called not in the global scope but in the scope of <code>foo</code> which also has a variable <code>a</code> declared within it. When bar is called <code>this</code> is assigned to the call site, which is <code>foo</code>, and <code>a</code> is looked up against this call site and not the global object as it was previously.

When determining context, the default rule is the last to be considered. Other ways of determining context take precedence over the default rule. Take the following example of a constructor function:

```javascript
function Foo(name) {
    this.name = name;
}
Foo.prototype.sayName = function() {
  return this.name;
};
var foo = new Foo("mike");
foo.sayName();  // mike
```

<code>this</code> is a device that can be used in a constructor function to assign properties specific to an instance of that constructor. In the above we can see that <code>this</code>, when used in a constructor, refers to the instance of the constructor.

The default rule and the constructor are the two organic ways in which context is defined in JavaScript. It can be explicitly set through other means however. The definition of <code>this</code> when explicilty set supersedes the organic ways. Context can be explicitly set in four different ways, as of ES6.

### Call and Apply

On the function constructor's prototype, there are two methods that you can use to explicitly set the context of <code>this</code>: <code>call</code> and <code>apply</code> on any function you declare. Consider the following example:

```javascript
function foo() {
  return this.bar;
}
var bar = "bar";
var obj = { 
  bar : "foo"
};
foo.call(obj);  // foo
```

In the above we have a function foo that returns <code>this.bar</code>. We have a variable <code>bar</code> defined in the global scope. We also have a variable <code>obj</code> which is an object that has a property also called <code>bar</code>. If we were to call foo normally in the global scope <code>this</code> would refer to the global object and so the global variable <code>bar</code> would be returned. In the example above however we call the function using <code>call</code>. <code>call</code> and <code>apply</code> allow you to explicilty define the context of <code>this</code> within the function they're attached to. It is set to the first argument either are passed. In the above example we can see foo is called using <code>call</code> and the context is set to the variable <code>obj</code>.

The difference between call and apply is in the subsequent parameter(s) they accept. The first argument is set to the context of <code>this</code>, any other parameters are the arguments passed into the function that they are called on. <code>call</code> takes an indefinite amount of comma separated parameters, whilst <code>apply</code> takes a single array of values as its second parameter.

```javascript
function add(num1, num2) {
  return num1 + num2;
}
addTwo.call(null, 5, 2); // 7
addTwo.apply(null, [5, 2]);  // 7
```

Notice how in the example above both call and apply can be used in a non-context based scenario. In the above there is no reference to <code>this</code> so the first argument supplied to both is null, as we're not interested in setting context in this case, we just want to call the function and supply it with arguments.
