# AOJSGuide
A guide to JavaScript.

## The JavaScript Compiler

JavaScript is a compiled programming language. What does that mean? It means JavaScript code is compiled by a JavaScript engine first before it is executed. Say you've written some code and you run it in Google's Chrome web browser, firstly the JavaScript engine that Chrome uses - v8 - compiles the code, then executes it. In this compilation stage the JavaScript engine runs through the code and compiles it into machine code, performing all manner of optimisation techniques along the way. It is at this time, during compilation, when lexical scope is defined.

## Lexical scope
Scope can be thought of as the thing that dictates the availability of variables and functions in a JavaScript environment. When a variable or function is declared in JavaScript it is scoped to its location within the code at author time. This is known as lexical scoping - lexical refers to lexicon i.e. the source. Pre ES6, variables were scoped either to the global scope or to a function scope. 

```javascript
// global scope
var foo = "foo";

function bar() {
  // function scope
  var foo = "bar";
}
```

### Variable and function scope

If a variable is defined in the global scope, it is available globally, throughout any code executed in the global environment. If a variable is defined in a function it is scoped to that function and therefore is available only within that function, not outside of it.

```javascript
function foo() {
  var bar = "bar";
}

// error 
// bar is not available outside of the function scope
console.log(bar);
```

In JavaScript things are declared either as variables or functions. The compiler determines function availability in the same way it does with variables, so functions declared within functions are not available outside of their parent. Functions can be written either as declarations or expressions, the difference between the two, other than the way they're declared (see below), is in the way the compiler treats them. The next section covers this.

```javascript
// declaration
function bar() {}

// expression
var foo = function() {}

```

### Let and const

ES6 introduced two new ways for variables to be declared - <code>const</code> and <code>let</code>. These both have different scoping criteria to var and function, and are also handled differently by the compiler (see the next section for an explanation as to how). Const and let are lexically block scoped which means they are available only within the block <code>{ }</code> they are declared in.

```javascript
for (let i = 0; i < 5; i++) {
  // i available within
}

// error
// i is scoped to the for loop block and not available externally
console.log(i);
```

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

After the compilation stage the <code>x</code> variable declared with var is hoisted but the <code>y</code> variable declared with let is not. When executed <code>x</code> has been declared but not yet given its value, <code>y</code> however has not been declared and an attempt to access it will throw an error. The area above the let declaration is known as the overly dramatic sounding <code>temporal dead zone</code>.

### Summary

Variables declared with var or function are hoisted to the top of their parent scope in the compilation stage before the code is executed. Variables are declared but are not assigned their specified value. Value assignment happens at run time, not compile time. Initially the value of all variables is set to undefined. Function declarations are evaluated at compile time. Function expressions are just variables and so are treated as a variable. Their value is likewise assigned at runtime. Variables declared with let and const are not hoisted and any attempt to reference them before their value is assigned will result in an error. This area before they are assigned a value is known as the temporal dead zone.

## Dynamic Scope

Dynamic scope is the runtime counterpart to lexical scope. Dynamic scope in JavaScript is scope created whenever a function is called, and is referred to as its variable <code>environment</code>, or prior to ES5, <code>activation object</code>. The <code>environment</code> creates a reference in memory to all the variables defined in that function scope including its arguments, and has access to its lexical scope through an inaccessible property <code>[[scope]]</code>. 

This property gives the function a reference to its parent scope, and its parent's parent scope, all the way up to the global scope. When looking up variable references the JavaScript engine will check against a function's immediate scope. If not found it looks to its parent, and then its parent's parent until it reaches the global scope where if it's not found the engine will throw a reference error.

Consider the following:

```javascript
function bar() {
  var foobar = "foobar"; 
  function foo() {
    return foobar;
  }
  return foo();
}

bar();  // foobar

```

The variable <code>foobar</code> is defined in the scope of <code>bar</code> and so is lexically scoped to <code>bar</code> and any of its descendant functions. When <code>bar</code> is called, a new object is created - its <code>environment</code>, which holds anything in its immediate and lexical scopes. The same thing happens for the inner function <code>foo</code>. When <code>foo</code> is called as the return value of <code>bar</code>, it looks up the value of <code>foobar</code> against its <code>variable environment</code>. It's not in its immediate scope so it looks for it in its parent scope where it finds it and is able to return it.

### Summary

Lexical scope is defined at compile time. At runtime when a function call is made, the JavaScript engine creates an object representing that function's variable <code>environment</code> which holds references to everything in its lexical scope. When asked to lookup a variable referenced within the function, the engine first checks against its immediate scope via its <code>environment</code>. If it doesn't find it there, it continues to search against each parent scope of the function until it reaches the global scope. The reference to these parent scopes is held in the <code>environment</code> which, as it's created and defined at runtime, is known as dynamic scoping.

## Context

Context, like dynamic scope, in JavaScript is defined at runtime, after the compilation stage. Every function, when executing, has a reference to its execution context. JavaScript stores this reference as the keyword <code>this</code>. Context can be defined organically or artificially in JavaScript. 

### The default context rule

Consider the following example:

```javascript
function bar() {
  console.log(this.a);  // "bar"
}
var a = "bar";
bar();

```

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

It is important to note that bind (and call and apply) cannot be used on function literals, they must be used only on a reference to the function. Call and apply, as they both execute the function they're called on, are applied directly to the function reference, whilst with bind the bound function must be assigned to a new reference. When this reference to the bound function is called the new binding rules will take effect. To illustrate this, consider the following:

```javascript
// syntax error
function foo() {
  return this.name;
}.bind({ name : "foo" });

function foo() {
  return this.name;
}

// calls foo
foo.call({ name : "foo" }); // foo

// does nothing as the bound function is not assigned to a new reference
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
Understanding closures is something that can only really be done after understanding lexical scoping. Take the following example:

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

The code above might help illustrate closures in their most simple form but it doesn't demonstrate their usefulness as a pattern, not without a small modification. In JavaScript functions are first class values, which means they are treated like all other values and can be passed as arguments into and returned from functions. Consider the following, a modified version of the previous code example:

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

This is the crux of closures. It is the ability to access values outside of their lexical scope. Lexical scope defined at the compilation stage is accessible at runtime even if the reference is made outside of the scope it was defined in. In the above the function foo has returned and is no longer in use, yet closures still allow access to values defined inside them. To help illustrate a more practical use of closures consider the following piece of code;

```javascript
function percentage(percent) {
  return function(value) {
    return value / 100 * percent;
  }
}

var vat = percentage(20);
vat(30);  // 6

```

The function <code>percentage</code> calculates the percentage of a number. The percentage is determined by the value of the argument it accepts. Its return value however is not the calculated answer - how could it be, at this point there is no value to calculate against - it is a function. It is this returned function that accepts a value to calculate the percentage of. 

What has happened is the returned function has closed over the scope of its parent. At compilation time the value of <code>percent</code> is undefined, but this doesn't matter - the returned function still has access to its reference through scope. When the percentage function is called the value it gets called with defines <code>percent</code>, essentially pre-loading the returned function with one of the two values needed to make the calculation. Now when the returned function is called, it calculates the pre-loaded percent of the value it's passed. This technique is often referred to as partial application or currying and allows functions like percentage to be re-used over and over producing new functions pre-loaded with different values.

```javascript
var vat = percentage(20);
vat(30);  // 6

var half = percentage(50);
half(100);  // 50

```

### IIFE

It is possible to create closure like effects at runtime through the use of IIFEs (Immediately Invoked Function Expressions). An IIFE is a function wrapped in an expression and immediatley called<code>(function(){})();</code>. Whereas with closures, variables are able to be retrieved through reference, IIFEs close around values (not references to a value), essentially preserving a snapshot of the value at the point of execution. To illustrate that, consider the following example:

```javascript
var a = 1;
var b = (function(c){
    return a + c;
})(a);
++a;
console.log(b); // 2

```

In the above IIFE, the variable <code>a</code> is passed in as an argument and also referenced directly within the function body. As the expression is immediately evaluated its value at time of execution is all that matters. This is great, however not all that useful. This is because the above code is synchronous. Changes to <code>a</code> do not happen until the expression finishes executing. To help demonstrate how useful IIFEs can be, consider the following asynchronous example:

```html
<button class="btn">Click me!</button>
<button class="btn">Click me!</button>
<button class="btn">Click me!</button>
```

```javascript
var btns = document.querySelector(".btn");
for(var i = 0; i < btns.length; i++) {
  btns[i].addEventListener("click", () => { 
    console.log(`I am button ${i}`);
  }, false);
}

```

If the above, each button on click would be expected to log out its index <code>i</code>. What actually happens is that all will log the current value of <code>i</code>, which in the above after the loop finishes would be <code>2</code>. The reason for this is each callback function retains only a reference to the same variable <code>i</code>, the value of it at assignment time is not remembered. This example can be fixed through the use of an IIFE:

```javascript
var btns = document.querySelector(".btn");
for(var i = 0; i < btns.length; i++) {
  (function(index){
    btns[index].addEventListener("click", () => { 
      console.log(`I am button ${index}`);
    }, false);
  })(i);
}

```

The above redefines <code>i</code> to <code>index</code> and traps that value at runtime time in the IIFE, meaning each button will log the current value of <code>index</code> and not the final value of <code>i</code>. Pre ES6, an IIFE was the only way to combat this specific problem. Fortunately block scoped variables don't have the same limitations as var. In ES6, swapping out var for let will fix the problem without the need for an IIFE.

```javascript
const btns = document.querySelector(".btn");
for(let i = 0; i < btns.length; i++) {
  btns[i].addEventListener("click", () => { 
    console.log(`I am button ${i}`);
  }, false);
}

```

As <code>i</code> is block scoped it is only available within the for loop block and so is not available for lookup after the block expires. However, moving the <code>let i = 0;</code> declaration outside the for loop makes it available again and mimics the behaviour of var in the previous examples.

### Summary

Closures can be thought of as a tool to allow runtime access to variables outside of their lexical scope. Their use is best demonstrated when functions return functions, and the returned function has within it a reference to a value that is not in its immediate scope. That reference has essentially been trapped in that function, and no matter where you call this returned function from it will have access to that value. 

IIFEs work in a similar way, only they close over values not references to values. They are evaluated at runtime as expressions and contain a function that is immediately invoked inside them. This immediate invocation forces the function scope to close over the values of variables it references that are within its lexical scope, creating what can be thought of as a snapshot of their value at that time. 

## Prototypal inheritance

JavaScript is an object oriented language. Everything is an object or behaves like one. Functions are objects. They can have properties and methods like any other object. Despite that functions and objects should be thought of as two separate constructs whcih essentially form the building blocks of JavaScript. JavaScript's inheritance model is not class based like other OOP languages such as Java or C#, it is prototype based. 

A prototype based system has two fundamental components - a <code>constructor</code>, which is a function, and a <code>prototype</code> which is an object. Constructors create objects. Every object you use within JavaScript has been created by a constructor. The first step towards understanding prototypal inheritance is to recognise how the return value of a function differs when called with and without the <code>new</code> keyword.

```javascript
function A(){}

var foo = A();
foo; // undefined

var bar = new A();
bar; // {}

```

### The constructor

A function always returns. Without an explicit return statement a function returns <code>undefined</code> which is why in the above <code>foo</code> logs as <code>undefined</code>. When called with <code>new</code> however a new object - the new instance - is returned. This object has a <code>constructor</code> property. 

```javascript
bar.constructor === A;  // true
bar instanceof A; // true

```

Whilst the <code>constructor</code> property implies by name that it is a reference to the function it was created from, this is not always the case. We'll see why shortly. Any function in JavaScript can be used as a constructor, but it will only be of any use in terms of inheritance if some additional setup has been carried out to the function beforehand. Before delving any further into the mechanics behind the constructor, let's look at an important detail of function definition. 

### The Prototype property

When a function is declared, the function object that is created as a result has a <code>prototype</code> property - an empty object - that is automatically created. It is this property that facilitates prototypal inheritance. The example demonstrates how:

```javascript
function A(){}
var foo = new A();
foo.bar;  // undefined
A.prototype.bar = "foobar";
foo.bar; // foobar

```

In the above a <code>bar</code> property is set on the prototype property of <code>A</code>. Any instance of <code>A</code> now has access to that property, which demonstrates that there is an implicit link between constructor and instance behind the scenes.   


The above outlines how the <code>new</code> keyword alters a function's return value. But what exactly happens when <code>new</code> is used? Well firstly and most obviously a new object is created - the new instance of the constructor.

```javascript
function A(){}
var foo = new A();  // foo is an instance of A

```

The context of the constructor is set to the new instance and that context is set as the constructor's return value. The following will likely explain this more clearly:

```javascript
function Greet(greeting) {
  // `this` is set to the instance of Greet
  this.greeting = greeting;
  // implicitly returns the instance
}

var hi = new Greet("hi");
hi.greeting;  // hi

// is the same behaviour as
function Greet(greeting) {
  this.greeting = greeting;
  return this;
}

var hi = {};
hi = Greet.call(hi, "hi");
hi.greeting;  // hi

```

linked to prototype
this context
returns this







