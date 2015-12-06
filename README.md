# AOJSGuide
A guide to JavaScript.

## The JavaScript Compiler

JavaScript is a programming language. A compiled programming language. What does that mean? When you write some JavaScript code to run in a web browser or any other JavaScript environment, it first needs to be compiled before it gets executed. Let's say you've written some code and you run it in Google's Chrome web browser, firstly the JavaScript engine that Chrome uses - v8 - compiles the code before its executed. In this compilation stage the JavaScript engine runs through your code and compiles it to machine code, performing all manner of optimisation techniques along the way. It is at this time, during compilation, when scope is defined.

## Scope
Scope refers to variable and function scope. When you declare a variable or function in JavaScript it is scoped based on its location within your script at author time. This is known as lexical scoping - scoping at the compiler stage. Pre ES6, variables were scoped either to the global scope or to a function scope. 

```javascript
// global scope
var foo = "foo";

function bar() {
  // function scope
  var foo = "bar";
}
```

### Variable and function scope

Scope dictates a variables availability. If a variable is defined in the global scope, it is available globally, throughout all your code. If a variable is defined in a function it is available only within that function andnot outside of it.

```javascript
function foo() {
  var bar = "bar";
}

// error 
// bar is not available outside of the function scope
console.log(bar);
```

When you write JavaScript you declare things either as variables or as functions. The compiler determines function scope the same way as it does variable scope, so functions declared within functions are not available outside of their parent. Functions can be written either as declarations or expressions, the difference between the two is in the way the compiler treats them. We'll look at why this is important to understand shortly.

```javascript
// declaration
function bar() {}

// expression
var foo = function() {}

```

### Let and const

ES6 introduced two new ways for variables to be declared <code>const</code> and <code>let</code>. These both have different scoping criteria to var or function. Const and let are block scoped which means they are available only within the block <code>{ }</code> they are declared in.

```javascript
for (let i = 0; i < 5; i++) {
  // i available within
}

// error
// i is scoped to the for loop block and not available externally
console.log(i);
```

Let and const are also handled differently to var and function by the compiler. How they are handled will be explained in the next section.

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

The compiler runs through the code and hoists the variables and functions to the top of the parent scope (in the above example the global scope), so when the code is executed at run time it looks like this:

```javascript

// after compiler
function b() {
  return "foo";
}
var a;
a = b();
console.log(a); // foo
```

Function declarations and expressions are treated differently by the compiler. Notice in the above example how the function declaration is hoisted to the top of the scope in its entirety. A function expression would be treated differently. Here's the same example again but using a function expression.

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

At runtime the above example would attempt to call <code>b</code> as a function before the function body is assigned as its value, resulting in an error. At that time its value is still undefined and therefore cannot be called as a function. 

### Hoisting and let and const

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

After the compilation stage the <code>x</code> variable declared with var is hoisted but the <code>y</code> variable declared with let is not. When executed <code>x</code> has been declared but not yet given its value, <code>y</code> however has not been declared and an attempt to access it will throw an error. The area above the let declaration is known as the <code>temporal dead zone</code>.

### Summary

Variables declared with var or function are hoisted to the top of their parent scope in the compilation stage before the code is executed. Variables are declared but are not assigned their specified value. Value assignment happens at run time, not compile time. Initially the value of all variables is set to undefined. Function declarations are evaluated at compile time. Function expressions are just variables and so are treated as a variable. Their value is likewise assigned at runtime. Variables declared with let and const are not hoisted and any attempt to reference them before their value is assigned will result in an error. This area before they are assigned a value is known as the temporal dead zone.

## Context

Context in JavaScript is defined at runtime, after the compilation stage. Every function, when executing, has a reference to its execution context. JavaScript stores this reference using the keyword <code>this</code>. Context can be defined organically or artificially in JavaScript. Consider the following example:

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

When using <code>addEventListener</code> from the DOM API, the context of <code>this</code> in the event handler is set behind the scenes to the element the event was triggered from. This is a convenient design choice as often in an event handler you will need a reference to the trigger element. To understand how context is organically determined take the following example:

```javascript
function bar() {
  console.log(this.a);  // "bar"
}
var a = "bar";
bar();
```

### The default context rule

Context is defined based on the location the function was called from - known as the call site. In the above example the function <code>bar</code> is called in the global scope and attempts to log a variable attached to <code>this</code>. The variables it attempts to log - <code>a</code> - has been defined in the global scope. Since the function was called in the global scope, <code>this</code> refers to the global object and therefore has a reference to the variable <code>a</code>.

This is known as the default context rule. Be aware that in strict mode the default context rule is ignored if the scope is global and <code>this</code> will be undefined. Trying to access a property on undefined obviously will throw an error. To see how the default rule changes according to the call site, take the following example:

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

Now the bar function is called not in the global scope but in the scope of <code>foo</code> which also has a property named <code>a</code>. When bar is called <code>this</code> is assigned to the call site, which is <code>foo</code>, and <code>a</code> is looked up against this call site and not the global object as it was previously.

When determining context, the default rule is the last to be considered. Take the following example of a constructor function:

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

The object returned from the constructor has a method <code>sayName</code> which returns a reference to <code>this</code>. Even though the call to the function is made in the global scope, its call site (and therefore context) is the object <code>foo</code>. In the above we can see that <code>this</code>, when used in a constructor or any property on a constructor's prototype, refers to the instance of the constructor. It can therefore be used to set values specific to that instance.

The default rule and the constructor are the two organic ways in which context is defined in JavaScript. It can be explicitly set through other means however. The definition of <code>this</code> when explicilty set supersedes the organic ways. Context can be explicitly set in four different ways, as of ES6.

### Call and Apply

On the function constructor's prototype, there are two methods which can explicitly set the context of <code>this</code> on any function - <code>call</code> and <code>apply</code>. Consider the following example:

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

In the above the function foo returns a reference to <code>this.bar</code>. There is a variable <code>bar</code> defined in the global scope. There's also a variable <code>obj</code>; an object with a property also named <code>bar</code>. If we were to call foo without the use of <code>call</code>, as its call site is the global scope <code>this</code> would refer to the global object and so the global variable <code>bar</code> would be returned. 

In the example above however foo is executed using <code>call</code>. <code>call</code> and <code>apply</code> can explicilty define the context of <code>this</code> within the function they're called on. Context is set to the first argument passed into both. In the above example foo is called using <code>call</code> and the context is set to the variable <code>obj</code>. The following example demonstrates how <code>call</code> takes precedence over the default context rule and sets the context of <code>this</code> to the global object and not the call site.

```javascript
var addTwo = {
    num : 2,
    calc : function(val) {
        return this.num + val;
    }
};
var num = 5;
console.log(addTwo.calc(2));  // 2
console.log(addTwo.calc.call(this, 5)); // 10
```

The difference between call and apply is in the subsequent argument(s) they accept. The first argument becomes the context of <code>this</code>, any other arguments are passed into the function as parameters. <code>call</code> takes an indefinite amount of comma separated parameters, whilst <code>apply</code> takes a single array of values as its second parameter. This is demonstrated in the following example:

```javascript
function Total(startingValue) {
    this.startingValue = startingValue;
}

Total.prototype.calc = function() {
    var total = this.startingValue;
    for (let i = 0; i < arguments.length; i++) {
        total += arguments[i];
    }
    return total;  
};

var count = new Total(0);
count.calc(1,2,3);  // 6

var startingValue = 10;
total.call(this, 7, 1, 2);  // 20
total.apply(this, [7, 1, 2]); // 20

```

### Bind

Bind is another method on the function prototype. The difference between <code>bind</code> and call / apply is that the latter two execute the function they're called on, <code>bind</code> however preloads the function with a context and arguments, but doesn't call it. Consider the following example:

```javascript
function callback() {
    console.log(this.name); // Mike
}
setTimeout(callback.bind({ name : "Mike" }), 1000);

```

In the above, the callback function passed into setTimeout has its context set before it is called using <code>bind</code>. When the function is called as the timer finishes <code>this</code> is bound to the first argument supplied to bind. The following example shows how to use <code>bind</code> to supply additional arguments to a function. It works in the same way as <code>call</code> in that each subsequent value supplied (after the context) is passed into the function as an argument.

```javascript
function greet(greeting) {
    return greeting + " " + this.name;
}
var greetMike = greet.bind({ name : "Mike" }, "Howdy");
greetMike();  // Howdy Mike

```

It is important to note that bind cannot be used directly on function declarations. To use bind you must create a new reference to the bound function. To illustrate this, consider the following:

```javascript
// syntax error
function foo() {
  return this.name;
}.bind({ name : "foo" });

function foo() {
  return this.name;
}

// does nothing as the bound function is not assigned to a nwe variable
foo.bind({ name : "foo" });

// correct: bar is a reference to the bound function
var bar = foo.bind({ name : "bar" });
bar();  // bar

```

### Fat Arrow functions

ES6 introduced a new syntax for functions, known as fat arrow functions. These functions handle context in a different way to their traditional counterparts. The below example helps illustrate this by first demonstrating how a traditional function declaration handles context. 

```javascript
var foo = "foo";
var bar = {
  foo : "bar",
  timer : function() {
    setTimeout(function() {
      console.log(this.foo);  // foo
    }, 500);
  }
}
bar.timer();

```

In the above, the timer method on the object bar is called. Within this method the context would be set as per the default rule to the object bar. However, the use of a setTimeout within that method takes the context away from bar. setTimeout is attached to the global object and so the call site of its callback would no longer be the object bar, it would be the global object. When logging <code>this.foo</code> in the timeout's callback, the foo property would be looked up against the global object and so in this example would be "foo." Below is the same example using a fat arrow function.

```javascript
var foo = "foo";
var bar = {
  foo : "bar",
  timer : function() {
    setTimeout(() => {
      console.log(this.foo);  // bar
    }, 500);
  }
}
bar.timer();

```

By using a fat arrow function the context of <code>this</code> is preserved to be that of its parent, which is the timer function on the object bar, meaning the <code>foo</code> property would be looked up against bar, not the global object. You can think of this behaviour in fat arrows to be the equivalent of using bind. The setTimeouts in the above example could be written in either of the following ways to achieve the same thing:

```javascript
// equivalents
setTimeout(() => {
  console.log(this.foo);
}, 500);

setTimeout(function() {
  console.log(this.foo);
}.bind(this), 500);

```

What actually happens behind the scenes is not the same as using bind however. In fact the binding of this does not happen at all in a fat arrow function. There is no <code>this</code> in a fat arrow, so context remains that of its parent.

### Summary
<code>this</code> in JavaScript is a reference to a function's execution context. It is dynamic in nature and is set at runtime, not during compilation. Context is defined organically either as the call site of the function or by using a constructor function, or set explicitly through the use of call, apply, bind and fat arrow functions.

## Closures
In order to understand closures you must first have a good understanding of lexical scoping. Take the following example:

```javascript
function bar() {
  var a = "bar";
  function foo() {
    console.log(a);
  }
  foo();
}
bar();  // a

```

In the above the function foo has access to the variable <code>a</code> from its parent scope. This access is set in the compilation stage, so <code>a</code> is lexically scoped to the function foo. When foo is called, the JavaScript engine will look up <code>a</code> against the scope of foo. It won't find it, so will then look it up against foo's parent scope, the function bar, where it will find it, and then is able to log its value. This is, in very simple terms, a closure. The function foo has closed over the function bar (and anything in bar's parent scope, and so on until it reaches the global scope) and retains access to any variables in its scope.

There is nothing revelationary in the above. However, a slight modification of the code can begin to demonstrate the power of closures. In JavaScript functions are first class values, which means they are treated like all other values and can be passed as arguments into and returned from functions. Consider the following, a modified version of the previous code example:

```javascript
function foo() {
    var a = "foo";
    function bar() {
        return a;
    }
    return bar;
}

var bar = foo();
bar();  // foo

```

In this example the function bar is returned from the function foo. The variable <code>a</code> is lexically scoped within foo and any descendant scopes. It is not available outside of the scope of foo. However, by creating a variable <code>bar</code> in the global scope and assigning it to the return value of foo, which is a function, when we call that function we have access to <code>a</code>, even though the call is made outside of the lexical scope of <code>a</code>.

This is the crux of closures. It is the ability to access values outside of their lexical scope. Lexical scope defined at the compilation stage is accessible at runtime even if the reference is made outside of the scope it was defined in. In the above the function foo has returned and is no longer in use, yet closures still allows access to values defined inside it. To help illustrate a practical use of closures consider the following piece of code;

```javascript
function percentage(percent) {
  return function(value) {
    return value / 100 * percent;
  }
}

var vat = percentage(20);
vat(30);  // 6

```

The function <code>percentage</code> calculates the percentage of a number. The percentage is determined by the value of the argument it accepts. Its return value however is not the calculated answer - how could it be, at this point there is no value to calculate against - it is a function. It is this returned function that accepts the value to calculate a percentage of. 

What has happened is the returned function has closed over the scope of its parent. At compilation time the value of percent is undefined, but this doesn't matter - the returned function still has access to its reference through scope. This is a powerful technique. It's often referred to as partial application or currying and allow functions like percentage to be re-used again and again.

```javascript
var vat = percentage(20);
vat(30);  // 6

var half = percentage(50);
half(100);  // 50

```
