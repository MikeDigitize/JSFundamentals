# AOJSGuide
A guide to JavaScript.

## The JavaScript Compiler

JavaScript is a compiled programming language which means any JavaScript code ran in a JavaScript environment, such as a browser or Node, first gets compiled by the engine before it's executed. It is at this time, during compilation, when scope is defined.

## Lexical scope
Scope can be thought of as the thing that dictates the availability of variables and functions in a JavaScript environment. It's a concept that relies on a number of mechanisms, the behaviour of whom collectively define the fundamental way in which JavaScript works. We'll cover these mechanism over the next several chapters. 

When a variable or function is declared in JavaScript it is scoped to its location within the code at author time. This is known as lexical scoping - lexical refers to lexicon i.e. the source (code). Pre ES6, variables were scoped either to the global scope or to a function scope. 

```javascript
// global scope
var foo = "foo";

function bar() {
  // function scope
  var foo = "bar";
}
```

### Variable and function scope

If a variable is defined in the global scope, it is available globally, throughout any code executed in the global environment. If a variable is defined in a function it is scoped locally to that function and therefore is available only within that function, not outside of it. This is known as <code>nested</code> scope. 

Scope not only defines a variable's availability, it can go some way to determine its lifetime too. If a variable is defined globally, its lifetime will likely be longer than, for example, a variable that's defined within a function that's called only once and doesn't leave its parent scope.

```javascript
function foo() {
  var bar = "bar";
}

console.log(bar); // error - bar is unavailable outside of the scope of `foo`

```

Your data and data operators are described either as variables or functions. The compiler determines function availability in the same way it does with variables, so functions declared within functions are not available outside of their parent. Functions can be written either as declarations or expressions, the difference between the two is in the way the compiler handles them (explained later).

```javascript
// declaration
function bar() {}

// expression
var foo = function() {}

```

### Block scope

ES6 introduced two new ways for variables to be declared - <code>const</code> and <code>let</code>. These both have different scoping criteria to var and function, and are also handled differently by the compiler. Const and let are lexically block scoped which means they are available only within the block <code>{ }</code> they are declared in.

```javascript
for (let i = 0; i < 5; i++) {
  // i available within
}

console.log(i); // error - i is scoped to the for loop block and not available externally

// block scoping could be achieved pre ES6 through IIFEs (see chapter on IIFEs)
var x = 10;
 
if (true) {
  (function() {
    var x = 20;
    // locally scoped to if block
    console.log(x); // 20
  })();
}
 
console.log(x); // 10

```

### Summary
Scope dictates the availablity of variables within your code. Variables can be scoped globally, within a function or within a block. Variable scope is determined in the compilation stage, before the code is executed. Scope determined at compile time is known as lexical scoping as it's determined based on the location of functions and variables within the source at author time.


## Hoisting
The compiler hoists variables and functions to the top of the scope they are declared in. Take the following code:

```javascript
// pre compiler
var a = b();
console.log(a);
function b() {
  return "foo";
}

```

The compiler runs through the code and hoists the variables and functions to the top of their parent scope (in the above example the global scope), so when the code is executed at run time it looks like this:

```javascript

// post compiler
function b() {
  return "foo";
}
var a;
a = b();
console.log(a); // foo

```

Declarations and expressions are treated differently by the compiler, as mentioned earlier. Notice in the above example how the function declaration is hoisted to the top of the scope in its entirety. A function expression would be treated differently. Here's the same example again but using a function expression.

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

a = b();  // error - b is not a function and is at this time still undefined
b = function() {
  return "foo";
}
console.log(a);

```

At runtime the above example would attempt to call <code>b</code> as a function before the function body is assigned as its value, resulting in an error. At that time its value is still undefined and therefore cannot be called as a function. 

### Hoisting let and const

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
  console.log(y); // error - y is not yet defined
  x = 1;
  let y = 2;
}
```

After the compilation stage the <code>x</code> variable declared with var is hoisted but the <code>y</code> variable declared with let is not. When executed <code>x</code> has been declared but not yet given its value, <code>y</code> however has not been declared and an attempt to access it will throw an error. The area above the let declaration is known as the overly dramatic sounding <code>temporal dead zone</code>.

### Summary

Functions and variables declared with <code>var</code> are hoisted to the top of their parent scope by the compiler before execution. The compiler hoists the identifier but not the value assignment, so initially the value of all variables by default will be <code>undefined</code>. Function declarations are hoisted in their entirety. Function expressions are just variables and so are treated like variables in they are declared but not assigned a value. Value assignment happens at runtime. Variables declared with let and const are not hoisted and any attempt to reference them before their value is assigned will result in a reference error. This area before they are assigned a value is known as the temporal dead zone.


## The Execution Context

The execution context can be thought of as an object, dynamically generated whenever a function is called, that holds references to everything in that function's lexical scope, in what is known as the function's <code>variable environment</code> or, prior to ES5, its <code>activation object</code>. The <code>variable environment</code> contains references to everything in that function's immediate scope, including its arguments, and has access to its non-immediate lexical scope through a reference to it's parent's <code>variable environment</code>.

This means any function, when called, has access to its parent's scope, and its parent's parent scope, all the way up to the global scope. When looking up variable references the JavaScript interpreter checks against a function's immediate scope. If the variable isn't found it checks against its parent, and then its parent's parent until finally it reaches the global scope where, if it's not found, throws a reference error.

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

The variable <code>foobar</code> is defined in the scope of <code>bar</code> and so is lexically scoped to <code>bar</code> and any of its descendant functions. When <code>bar</code> is called, a new execution context is created which contains its <code>variable environment</code> and provides access to anything in its lexical scope. The same thing happens for the inner function <code>foo</code>. When <code>foo</code> is called as the return value of <code>bar</code>, the interpreter looks up the value of <code>foobar</code> against its <code>variable environment</code>. In this case <code>foobar</code> is not in its immediate scope so the interpreter goes up a level and looks for it in its parent's scope, where it finds it and is able to return the value.

### Summary

At runtime, when a function is called, the JavaScript engine creates an excution context for that function. This can be thought of as an object that holds, amongs other things, references to everything in its lexical scope. This part of the execution context is known as the <code>variable environment</code>. The <code>variable environment</code> contains everything in the function's immediate scope and also contains a link to its parent's <code>variable environment</code>. When asked to lookup a variable referenced within the function, the interpreter first checks against the function's immediate scope, if it doesn't find it there it continues to search against each parent scope until it reaches the global scope. Only after it cannot find it in the global scope will the interpreter throw a reference error.


## Context

Context in JavaScript is defined at runtime as part of a function's dynamically generated execution context. JavaScript provides a reference to a function's context through the keyword <code>this</code>. Context can be defined organically or artificially in JavaScript. 

### The default context rule

Consider the following example:

```javascript
function bar() {
  console.log(this.a);  
}
var a = "bar";
bar();  // bar

```

Context is defined based on the location the function was called from - known as the call site. In the above example the function <code>bar</code> is called in the global scope and attempts to log a variable attached to <code>this</code>. The variable it attempts to log - <code>a</code> - has been defined in the global scope. Since the function was called in the global scope, <code>this</code> refers to the global object and therefore logs <code>a</code> from the global scope.

This is known as the default context rule. Be aware that in strict mode the default context rule is ignored if the scope is global; if that's the case <code>this</code> will be <code>undefined</code>. To see how the default rule changes according to the call site, take the following example:

```javascript
function bar() {
  console.log(this.a);
}
var a = "bar";
var foo = {
  a : "foo",
  b : bar
};
foo.b();  // foo

```

Now the bar function is called not in the global scope but in the scope of <code>foo</code> which also has a property named <code>a</code>. When bar is called <code>this</code> is assigned to the call site, which is <code>foo</code>, and therefore <code>a</code> is looked up against <code>foo</code> and not the global object as it was previously.

When determining context, the default rule is the last to be considered. Take the following example of a constructor function (constructor functions are covered in detail later):

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

The object returned from the constructor has a method <code>sayName</code> which contains a reference to <code>this</code>. Through the use of the constructor <code>this</code> is bound to the instance, in this case the variable <code>foo</code>. In the above we can see that <code>this</code>, when used in a constructor or any property on a constructor's prototype, will be bound to that instance of the constructor. It can therefore be used to set values specific to that instance. Prototypal inheritance will be covered in a later chapter.

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

In the above the function foo returns a reference to <code>this.bar</code>. There is a variable <code>bar</code> defined in the global scope. There's also a variable <code>obj</code> - an object with a property also named <code>bar</code>. If we were to call foo without the use of <code>call</code>, as its call site is the global scope <code>this</code> would refer to the global object and so the global variable <code>bar</code> would be returned. 

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
    console.log(this.name);
}
setTimeout(callback.bind({ name : "Mike" }), 1000); // Mike

```

In the above, the callback function passed into setTimeout has its context set before it is called using <code>bind</code>. When the function is called as the timer finishes <code>this</code> is bound to the first argument supplied to bind. The following example shows how to use <code>bind</code> to supply additional arguments to a function. It works in the same way as <code>call</code> in that each subsequent value supplied (after the context) is passed into the function as an argument.

```javascript
function greet(greeting) {
    return `${greeting} ${this.name}`;
}
var greetMike = greet.bind({ name : "Mike" }, "Howdy");
greetMike();  // Howdy Mike

```

It is important to note that bind (and call and apply) cannot be used on function literals that aren't supplied as arguments, they must be used only on a reference to the function. Call and apply, as they both execute the function they're called on, are applied directly to the function reference, whilst with bind the bound function must be assigned to a new reference so it can be called. When this reference to the bound function is called the new binding rules will take effect. To illustrate this, consider the following:

```javascript
// syntax error
function foo() {
  return this.name;
}.bind({ name : "foo" });

// this is ok as the function literal is an argument
setTimeout(function() {
  console.log(this.name);
}.bind({ name : "foo" }), 1000);

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

// immediately calling the bound function would work too
foo.bind({ name : "foo" })();

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

By using a fat arrow function the context of <code>this</code> is preserved to be that of its parent, which is the timer function on the object bar, meaning the <code>foo</code> property would be looked up against bar, not the global object. You can think of the behaviour of fat arrow functions to be the equivalent of using bind. The setTimeouts in the above example could be written in either of the following ways to achieve the same thing:

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
A function's context is generated at runtime when the function is called. It's referenced in its execution context and available through the keyword <code>this</code>. Context is set organically - if it's used in a constructor function it's set to the instance of the constructor (takes precedence), or it's set to the call site of the function. This organic behaviour can be overriden and set explicitly through the use of call, apply, bind and fat arrow functions.


## Closures
Understanding closures is something that can only really be done after understanding at least a little of how lexical scoping and execution contexts in JavaScript work. Take the following example:

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

In the above the function <code>foo</code> has access to the variable <code>a</code> from its parent scope. This access is set in the compilation stage, so <code>a</code> is lexically scoped to the function <code>foo</code>. When <code>foo</code> is called, the JavaScript engine looks up <code>a</code> in its immediate scope through its variable environment. It won't find it, so will then look it up against its parent scope, the function <code>bar</code>, where it will find it, and can log its value. This is, in very simple terms, a closure. The function <code>foo</code> has closed over the scope of the function <code>bar</code> (and anything in bar's parent scope, and so on until it reaches the global scope) and retains access to any variables in its scope.

The code illustrates closures in their most literal form but it doesn't demonstrate their usefulness, not without a small modification. In JavaScript functions are first class values, which means they are treated like all other values and can be passed as arguments into and returned from functions. Consider the following, a modified version of the previous code example:

```javascript
function bar() {
    var a = "bar";
    function foo() {
        return a;
    }
    return foo;
}

var foo = bar();
foo();  // bar

```

In this example the function foo is returned from the function bar. The variable <code>a</code> is lexically scoped to foo and any descendant scopes. It's not available outside of the scope of bar. However, by creating a variable <code>foo</code> in the global scope and assigning it to the return value of bar, which is a function, when we call that function we get access to <code>a</code>, even though the call is made outside of the lexical scope of <code>a</code>.

This is the crux of closures. It is the ability to access values outside of their lexical scope. Lexical scope is accessible at runtime even if the reference lookup is made outside of the scope the reference was defined in. This ability is provided by the variable environment created as part of the function's execution context which is created when <code>bar</code> is called. In the above the function <code>bar</code> has returned and is no longer in use, yet through a closure we can still get access to values defined in its lexical environment. To help illustrate a more practical use of closures consider the following:

```javascript
function percentage(percent) {
  return function(value) {
    return value / 100 * percent;
  }
}

var vat = percentage(20);
vat(30);  // 6

```

The function <code>percentage</code> calculates the percentage of a number. The percentage is determined by the value of the argument it accepts. Its return value however is not the calculated answer - how could it be? At this point there is no value to calculate against - the return value is a function. It is this returned function that accepts the value to calculate the percentage of. 

What's happened is the returned function has closed over the scope of its parent. At compilation time the value of <code>percent</code> is undefined, but this doesn't matter - the returned function still has access to its reference through scope. When the percentage function is called the value it gets called with defines <code>percent</code>, essentially pre-loading the returned function with one of the two values needed to make the calculation. Now when the returned function is called, it calculates the pre-loaded percent of the value it's passed. This technique is often referred to as partial application or currying and allows functions like percentage to be re-used over and over producing new functions pre-loaded with different values.

```javascript
var vat = percentage(20);
vat(30);  // 6

var half = percentage(50);
half(100);  // 50

```

### IIFE

It is possible to create closure like effects at runtime through the use of IIFEs (Immediately Invoked Function Expressions). An IIFE is a function wrapped in an expression and immediatley called<code>(function(){})()</code>. Whereas with closures, variables are able to be retrieved through reference, IIFEs run immediately so close around the value at the time of execution, essentially preserving a snapshot of the value at that point within the script. The closure similarity is more in the name, in that they close around values in scope at time of execution. They also, by virtue of them being functions, create their own scope which is inaccessible to anything outside of the IIFE, unless it returns. To demonstrate this, consider the following example:

```javascript
var a = 1;
var b = (function(c){
    return a + c;
})(a);
++a;
console.log(b); // 2

```

In the above IIFE, the variable <code>a</code> is passed in as an argument and also referenced directly within the function body. As the expression is immediately evaluated its value at time of execution is all that matters. This helps demonstrate how IIFEs work but is not all that useful as a pattern. This is because the above code is synchronous. Changes to <code>a</code> do not happen until the expression has been called. To help demonstrate how useful IIFEs can be, consider the following asynchronous example:

```html
<button class="btn">Click me!</button>
<button class="btn">Click me!</button>
<button class="btn">Click me!</button>

```

```javascript
var btns = document.querySelector(".btn");
for(var i = 0; i < btns.length; i++) {
  btns[i].addEventListener("click", () => { 
    console.log(`I am button ${i}`);  // es6 template strings for string concat
  }, false);
}

```

If the above, each button on click would be expected to log out its index <code>i</code>. What actually happens is that all will log the value of <code>i</code> at the time of a click, which in the above after the loop finishes would always be <code>2</code>. The reason for this is each callback function retains only a reference to the same variable <code>i</code>, the value of it at assignment time is not remembered. This example can be fixed through the use of an IIFE:

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

Closures can be thought of as a tool to allow runtime access to variables outside of their lexical scope. Their use is best demonstrated when functions return functions, and the returned function has within it a reference to a value that is not in its immediate scope. That reference has essentially been trapped in that function, and no matter where you call this returned function from it will have access to its value. 

IIFEs work in a similar way, only they close over the current value of the variable they reference at runtime. The immediate invocation forces the function scope to close over the values of references within its lexical scope, creating what can be thought of as a snapshot of their value at that time. 


## The Call Stack

The call stack is generated by the interpreter as soon as code execution begins. It first creates the global execution environment and pushes it onto the stack. This contains all the references to values and functions in the global scope. It begins to step through the code one line at a time. JavaScript is single threaded, so it only executes one code statement at a time. The current execution context is updated as values change. 

When the interpreter encounters a function call it creates a new execution context for this function and pushes it to the top of the stack. It runs through the code in this function and updates the current execution context. If it encounters another function call it creates another execution context, pushes it to the top of the stack and proceeds onwards. Once a function has returned, if its variables are not still referenced elsewhere (i.e. a closure), its execution context is popped from the stack. The context beneath then becomes the new current execution context and the interpreter proceeds onwards. 

Consider the following piece of code and a simulation of how the call stack would behave when the interpreter begins execution:

```javascript
// javascript
var foo = "foo";

function bar() {
  var bar = "bar";
  return `${foo}${bar}`;
}

bar();  // foobar
console.log("done");

// the call stack
var callStack = [];
// create and push in global context
var globalExecutionStack = {
  scope : null,
  variableEnvironment : {
    foo : null,
    bar : <function>
  }
}
callStack.unshift(globalExecutionStack);

// assign foo a value and store in current execution context
callStack[0].variableEnvironment.foo = "foo";
// `bar` function call
callStack[0].variableEnvironment.bar();
// create and push new execution context for `bar`
var barExecutionContext = {
  scope : { global },
  variableEnvironment : {
    bar : null
  }
}
// add to the top of the call stack
callStack.unshift(barExecutionContext);

// assign bar a value and store in current execution context
callStack[1].variableEnvironment.bar = "bar";
// `bar` returns a reference to `foo` which is outside it's lexical scope
// check for it in the immediate scope of `bar`
callStack[1].variableEnvironment.foo; // undefined;
// check for it in the parent scope of `bar`
callStack[0].variableEnvironment.foo; // foo
// return concat of the inner `bar` and `foo` - "foobar"
// once `bar` has returned remove its execution context from the stack
callStack.splice(1,1);

// previous context resumes control
// console log done
// remove global execution context from stack
callStack.splice(0,1);

// execution ends

```

Whenever a function returns but contains a closure, its execution context cannot be removed from memory if there remains access to this closure. If this is the case it's stored instead on what is known as the "heap" - a dynamically allocated piece of memory - until the function's inner variables have been resolved for the final time at which point it can be removed. 

### Summary

During execution, whenever a function is called, an execution context for the function is created. This is essentially an object which contains its global variable environment, a reference to its non-immediate lexical scope and its <code>this</code> context. When execution begins, an execution context is created for the global scope and pushed to the top of the call stack. This becomes the current / active context. Values held in the context change as the interpreter proceeds through the code. When a function call is made, that function's execution context is pushed to the top of the stack and becomes the active context. It continues to be so until the function returns, at which point the previous context resumes control. If the function returns but contains a reference to a value in its lexical scope i.e. a closure, the context is added to the JavaScript heap, i.e. a location in memory, and is preserved until that closure is no longer needed.


## Prototypal inheritance

JavaScript is an object oriented language. Everything is an object or behaves like one. Functions are objects. They can have properties and methods like any other object. Despite that, functions and objects should be thought of as two separate constructs; two constructs which essentially form the building blocks of JavaScript. Whereas with other object constructs such as RegExp or Date, Function is <code>typeof</code> "function" and not "object."

```javascript
var foo = new Date();
typeof foo; // object
var bar = new RegExp();
typeof bar; // object
var foobar = new Function();
typeof foobar;  // function

```

JavaScript's inheritance model is not class based like other OOP languages such as Java or C#, it is prototype based. A prototype based system has two fundamental components - a <code>constructor</code>, which is a function, and a property on the function - <code>prototype</code>, which is an object. Constructors create objects. Every object you use within JavaScript has been created by a constructor. Every object created has a <code>constructor</code> property. The first step towards understanding prototypal inheritance is to recognise how the return value of a function differs when called with and without the <code>new</code> keyword.

```javascript
function A(){}

var foo = A();
foo; // undefined

var bar = new A();
bar; // {}

```

### The Constructor property

A function always returns. Without an explicit return statement a function returns <code>undefined</code> which is why in the above <code>foo</code> logs as <code>undefined</code>. When called with <code>new</code> however a new object - the new instance - is returned. This object has a <code>constructor</code> property. 

```javascript
bar.constructor === A;  // true
bar instanceof A; // true

```

Whilst the <code>constructor</code> property implies by name that it is a reference to the function it was created from, this is not always the case. We'll see why shortly. Any function in JavaScript can be used as a constructor function, but it will only be of any use in terms of inheritance if some additional setup has been carried out to the function beforehand. Before delving any further into the mechanics behind the constructor, let's look at an important detail of function definition. 

### The Prototype property

When a function is declared, the function object that's created as a result implicilty inherits a <code>prototype</code> property - an empty object. It is this property that instances of a function inherit from. The below example demonstrates how:

```javascript
function A(){}
var foo = new A();
foo.bar;  // undefined
A.prototype.bar = "foobar";
foo.bar; // foobar

```

In the above a <code>bar</code> property is set on the prototype property of <code>A</code>. Any instance of <code>A</code> now has access to that property, demonstrating an implicit link between constructor and instance, behind the scenes. This implicit link on an instance is, confusingly, also known as the <code>[[Prototype]]</code> and will be detailed shortly. 

The above outlines how the <code>new</code> keyword alters a function's return value. But what exactly happens when <code>new</code> is used? Well firstly and most obviously a new object is created - the new instance of the constructor, and this instance gets linked to the prototype of its constructor via its <code>[[Prototype]]</code>. Properties defined on the constructor's prototype do not appear on the instance, but can be referenced as if they were through the prototype chain, facilitated by <code>[[Prototype]]</code>.

```javascript
function A(){}
var foo = new A();  // foo is an instance of A
A.prototype === foo.constructor.prototype;  // true
A.bar = "bar";  //  static property on the constructor
A.prototype.foobar = "foobar";  // accessible now on any instance of A
foo.foobar; // foobar

```

Secondly, the context of <code>this</code> in the constructor points to the new instance, and thirdly that context is implicitly set as the constructor's return value. If that's not clear, the following will likely explain it better:

```javascript
function Greet(greeting) {
  // `this` is set to any instance of Greet that's being instantiated
  this.greeting = greeting;
  // implicitly returns the instance despite no return statement
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

### Extending prototypes

Constructors created to provide a simple piece of functionality can be extended to create more complex pieces of functionality. Consider the following example of a constructor that gets and sets a DOM element:

```html
<p>This</p>
<p>is</p>
<p>Text</p>

```

```javascript
// constructor
function GetEl(){}

GetEl.prototype.setDOM = function(selector) {
  this.DOM = Array.from(document.querySelectorAll(selector));
};

GetEl.prototype.getDOM = function() {
  return this.DOM;
};

var p = new GetEl();
p.setDOM("p");
p.getDOM(); // array of p tags

```

This is common behaviour so it makes sense for it to be wrapped in a re-usable function. It's important to recognise the reduction in memory consumption that prototypal inheritance offers. Any new instances of the above will be able to retrieve DOM elements but the function that sets the DOM element is only defined once, on the constructor's prototype. Every instance does not carry its own version of that function, they all just point to the one on the prototype, therefore reducing their memory footprint. Something else that is common behaviour, besides retrieving DOM elements, is adding and removing event listeners to DOM elements. So wouldn't it be useful if we could extend the above to accomodate event handling? Consider the following: 

```javascript
function AddEvents(){
  this.events = {};
}

AddEvents.prototype = new GetEl();

// add event listeners
AddEvents.prototype.add = function(evt, fn, capture) {
  capture = capture ? capture : false; 
  this.events[evt] = [];
  this.DOM.forEach(el => {
    let listener = { 
      el : el,
      evt : evt, 
      fn : fn, 
      capture : capture 
    };
    el.addEventListener(evt, fn, capture);
    this.events[evt].push(listener);
  }); 
};

// remove event listeners
AddEvents.prototype.remove = function(evt) {
  this.events[evt].forEach(listener => {
    listener.el.removeEventListener(listener.evt, listener.fn, listener.capture);
  });
  this.events[evt] = [];
};

var pEvts = new AddEvents();
pEvts.setDOM("p");  // inherited from GetEl
pEvts.add("click", () => console.log("click")); // adds click event to all p tags
pEvts.remove("click");  // removes click listener from all p tags

```

The above extends the <code>GetEl</code> function by adding in two new methods (to add and remove event listeners) and one new property which keeps a record of any event listeners added so they can be removed if needs be. The important part of this code as far as prototypal inheritance is concerned is this line:

```javascript
AddEvents.prototype = new GetEl();

```

This line sets the prototype of <code>AddEvents</code> to a new instance of <code>GetEl</code>, which means the <code>getDOM</code> and <code>setDOM</code> methods and <code>DOM</code> property of <code>GetEl</code> are now available, via its prototype, to any new instance of <code>AddEvents</code>. This, in the above example, is vital as <code>AddEvents</code> is written specifically to use the internals of <code>GetEl</code>.

By setting the prototype of <code>AddEvents</code> to point at the prototype of <code>GetEl</code> we have created a link between the two constructors. We can continue to add to the prototype of <code>AddEvents</code> without affecting <code>GetEl</code> as prototypal inheritance flows downwards, not upwards. This means however that when we add to the prototype of <code>GetEl</code>, any instances of <code>AddEvents</code> will automatically inherit any new properties. Consider the following:

```javascript
GetEl.prototype.removeDOM = function() {
  let reset;
  this.DOM = reset;
};

pEvts.removeDOM();  // available on an instance of AddEvents through prototypal inheritance

```

Prototypal inheritance is a clearly useful pattern but there are a few limitations with it. Firstly, it doesn't allow for private properties - everything must be attached to the instance at instantiation or through the prototype, making all its properties publicly available. Secondly, when extending prototypes there is no mechanism to inherit only the methods or properties that are needed - extending means inheriting everything, which is not always ideal. Thirdly, there is its over reliance on <code>this</code> and the potential for that reference to be overwritten by one of the means discussed earlier in the context chapter, making code within potentially brittle. 

### [[Prototype]]

Things can become a little confusing regarding JavaScript's inheritance model when you discover that it is not the explicit property <code>prototype</code> property found on functions is not the only property using that name in JavaScript's prototypal inheritance model. Objects that are instances of constructors inherit from the constructor's <code>prototype</code> property and all objects also have an internal property that points to their constructor's <code>prototype</code>, a property that is also called, confusingly, prototype, but written in the ECMAScript spec as [[Prototype]]. This property is aliased in script as <code>\__proto__</code> and referred to as the "dunder" prototype. The way to differentiate between the two is to think of a function's <code>prototype</code> property as the one used to create new objects, whilst the dunder proto is a property on objects used to lookup properties in the prototype chain.

```javascript
var Foo = function(){}
var foo = new Foo();
foo.constructor === Foo;  // true
foo.__proto__ === foo.constructor.prototype;  // true

```

The <code>\__proto__</code> property has been standardised as of ES6 which now gives us another means to create objects, only this time not from constructors but from other objects.

```javascript
var foo = { name : "foo" };
var bar = {};
bar.__proto__ = foo;
bar.name; //  foo

```

To demonstrate how <code>\__proto__</code> is used to lookup inherited properties consider the following:

```javascript
// the prototype chain using objects
var foo = { bar : "foo" };
var bar = {};
bar.__proto__ = foo;
bar.foo = "bar";
var foobar = {};
foobar.__proto__ = bar;
foobar.foo; // bar
foobar.bar;  // foo

foobar.__proto__ === bar; // true
foobar.__proto__.foo; // bar
foobar.__proto__.__proto__ === foo; // true
foobar.__proto__.__proto__.bar;  // foo

// the prototype chain using constructors
function Bar(){}
Bar.prototype.foo = "foo";
function Foo(){}
Foo.prototype = new Bar();
Foo.prototype.bar = "bar";
function FooBar() {};
FooBar.prototype = new Foo();
var bar = new FooBar();
bar.bar;  //bar
bar.foo;  // foo

bar.__proto__ === Foo;  // true
bar.__proto__.bar; // bar
bar.__proto__.__proto__ === Bar;  // true
bar.__proto__.__proto__.foo; // foo

```

JavaScript resolves object properties in the same way scope resolves references within functions and the global scope, only instead of using the variable environment / activation object, it uses the <code>[[Prototype]]</code> chain through the dunder proto property. The above examples essentially achieve the same thing. In both, when looking up properties, the JavaScript engine first looks to see if the property resides on the instance. If the engine doesn't find it there, it begins to traverse up the <code>[[Prototype]]</code> chain checking against each <code>\__proto__</code> object until it reaches Object's dunder prototype, which is where the chain ends and resolves as <code>null</code>. If the engine can't find the property on Object - the dunder proto equivalent of scope's global environment - <code>undefined</code> is returned. Changes to the source object will be reflected in any instances via the prototype chain, essentially producing a similar inheritance pattern to that achieved by a function's <code>prototype</code> property.

### Constructor property gotchas

When creating an instance from a constructor it would be reasonable to assume that the instance object's constructor property should point to the constructor it came from. This is the case for instances created from a constructor whose prototype has not been extended, but not so from a constructor whose prototype has been extended. Consider the following:

```javascript
function Foo(){}
var foo = new Foo();
foo.constructor;  // Foo
foo instanceof Foo; // true

function Bar(){}
// extended prototype
Bar.prototype = new Foo();
var bar = new Bar();
// initially points to its extended constructor
bar.constructor;  // Foo
bar instanceof Bar; // true
// explicitly define its constructor
bar.constructor = Bar;
bar.constructor;  // Bar

function FooBar(){}
FooBar.prototype = new Bar();
var foobar = new FooBar();
foobar.constructor === Foo; // true
// explicitly define its constructor
foobar.constructor = FooBar;
foobar.constructor; // FooBar

```

Whilst the <code>instanceof</code> operator returns an accurate result, when creating objects with the <code>new</code> keyword, the <code>constructor</code> property does not and needs to be explicitly set. The <code>instanceof</code> operator will return true on a constructor if an instance is part of its inheritance chain, not just if it's an immediate instance. Consider from the above example how <code>instanceof</code> returns for each of the instances:

```javascript
foobar instanceof FooBar; // true
foobar.instanceof Bar;  // true
foobar.instanceof Foo;  // true

bar instanceof FooBar; // false
bar.instanceof Bar;  // true
bar.instanceof Foo;  // true

foo instanceof FooBar; // false
foo.instanceof Bar;  // false
foo.instanceof Foo;  // true

```

### Static methods

As functions are objects it's possible to store properties on the constructor itself. These are known as <code>static</code> properties. Any instances of the constructor do not inherit these properties as they're not on its <code>prototype</code>.

```javascript
var foo = { a : "a", b : "b", c : "c" };
// `keys` is a static property on the Object function
Object.keys(foo).forEach(key => console.log(key));  // a, b, c

```

### Object.Create

The use of constructors and the <code>new</code> keyword is often controversial in JavaScript. Some developers prefer the readability and easy to grasp semantics, whilst others say they wrongly imply a similarity to Java's class based constructor pattern, see below, encouraging developers to write JavaScript through a paradigm not suited to the language's strengths.

```java
// java constructor
public Foo(int startBar) {
    bar = startBar;
}

// instance
Foo myFoo = new Foo(30);

```

ES5 saw the introduction of <code>Object.create</code>, designed as a counter to <code>new constructor</code>. It takes as an argument an object to copy and returns a new object with properties that point to those of its creator (properties specific to the object, not ones gained through its prototype chain). Consider the following:

```javascript
var foo = { foo : "foo", bar : "bar" };
var bar = Object.create(foo);

bar;  // Object
foo === bar;  // false
bar.__proto__ === foo;  // true

foo.foo;  // foo - held on object `foo`
bar.foo;  // foo - accessed via __proto__ link to `foo`

bar.foo = "FOO";
foo.foo = "bar";
bar.foo;  // FOO - property on instance takes precedence over prototype chain

foo.foobar = "foobar";
bar.foobar; // foobar - accessed via __proto__ link to `foo`
bar.foobar = "barfoo";
foo.foobar; // foobar - inheritance flows down the chain, not up it

```

In the above <code>bar</code> is a copy of <code>foo</code>. Properties that are inherited from <code>foo</code> are available via lookup through its <code>\__proto__</code> property. Any changes to <code>foo</code> will be reflected in <code>bar</code>. The JavaScript interpreter when looking for properties, like always, first checks against the instance, before checking against the object's prototype chain.

Prototyping is of course possible with <code>Object.create</code>. Consider the following:

```javascript
function Foo(){}
Foo.prototype.bar = "bar";

var foo = Object.create(Foo.prototype);
foo.bar;  // bar
foo.__proto__;  // Foo

```

<code>Object.create</code> also takes an object as a second parameter that defines properties for that instance, including getters and setters. Consider the following:

```javascript
// constructor / new inheritance pattern
function Foo(bar) {
  this.bar = bar;
}
var foo = new Foo("foobar");
foo.bar;  // foobar

// the above using object create
var foo =  Object.create(Foo.prototype, { 
  bar : { writable: true, configurable: true, value: "foobar" }
});
foo.bar;  // foobar

```

### Summary

Prototypal inheritance is the mechanism JavaScript uses to allow objects to inherit from other objects or functions. Its facilitated either through the use of a function - the constructor - and the <code>new</code> keyword or an object and it's [[Prototype]] property, known as the dunder proto property and written as <code>\__proto__</code>. Whenever a new instance of a constructor is created it inherits any properties that are on the constructor's <code>prototype</code> property. The link between the instance and the constructor remains even after the new instance has been created. Subsequent additions or changes to any property on the <code>prototype</code> will be reflected on every instance. 

Any properties added to the constructor directly are known as static properties and are available only via the constructor and not on any instances of it. The dunder prototype is a newly standardised property (previously implementations differed from engine to engine) which points to the prototype of an object's constructor, traversal of which is the underlying way in which JavaScript looks up properties against an object's prototype chain. ES5 saw the introduction of <code>Object.create</code>, a static method on the <code>Object</code> function for use as an alternative (and more inline with JavaScript's prototypal nature) to the constructor / new pattern. 


## JavaScript Design Patterns

Prototypal inheritance is not the only way to create objects as we've seen previously using the, as of ES6, standardised <code>[[Prototype]]</code> property. But with a language as flexible as JavaScript there are a multitude of alternatives available if either of these do not fit the use case.

### Factories

A factory is any function that returns an object. Constructor functions are factories. When a constructor function is invoked with the <code>new</code> keyword, it returns an object. This forces functions to behave differently than usual, for example, as we saw earlier, they return things (objects) that aren't explicitly returned. But what if we used standard functions that returned objects instead of constructors and omitted the <code>new</code> keyword?

```javascript
function Foo(name) {
  return {
    getName : function() {
      return name || "Unknown";
    }
  }
}

var foo = Foo("foo"); // { name : "foo" }
var bar = Foo("bar"); // { name : "bar" }
foo.getName();  // foo
bar.getName();  // bar
foo === bar;  // false

```

In the above the factory function <code>Foo</code> returns an object that, through closure, retains a reference to the <code>name</code> parameter passed into <code>Foo</code> when it's called. The ability to return only what we want means that when creating factories, unlike in prototypal inheritance, we can make use of private properties. 

Factory functions are great for creating objects that don't need inheritance. They can't be extended. They return instances that differ from those created with the <code>new</code> keyword for a number of ways, but most importantly - as <code>Foo</code> returns an object literal, any instances of <code>Foo</code> will inherit from <code>Object</code> and not <code>Foo</code>. So adding anything to the prototype of <code>Foo</code> will have no effect on any instances of it because there is no prototypal link between it and the object it returns.

### Composition

Factory functions have an obvious limitation in that they don't support inheritance. But inheritance is not always the best solution for sharing methods. When an instance inherits from a constructor's blueprint it inherits everything on its prototype. But what if we only actually need one or two of its properties? With this in mind an often preferable alternative to pure inheritance is composition - creating objects composed of other objects or functions. Think of the difference between the two as - inheritance is when objects are defined based on what they are whereas composition is objects defined based on what they do. 

The two techniques, like most patterns in JavaScript, can be mixed together. Consider the following:

```javascript
function Total(costs) {
  return costs.reduce((total, cost) => total += cost, 0);
}

function Percent(percent, of) {
  return Number((of / 100 * percent).toFixed(2));
}

function Bill() {}

Bill.prototype.getTotal = function(costs) {
  this.total = Total(costs);
};

Bill.prototype.getTip = function() {
  this.tip = Percent(20, this.total);
};

var tableOfFour = new Bill();
tableOfFour.getTotal([20,45,12,54,34,23,33,38,]); // 259
tableOfFour.getTip(); // 51.8

var tableForTwo = new Bill();
tableForTwo.getTotal([8,24,12,26]); // 70
tableForTwo.getTip(); // 14

```

In the above <code>Bill</code> is a constructor that creates instances capable of returning the total of a bill and the amount to tip. Its prototype is composed using two global functions <code>Percent</code> and <code>Total</code>. Consider the following code that calculates income tax which requires similar functionality to that of <code>Bill</code>, but not the same. Inheritance wouldn't work in this scenario, but fortunately composition allows us to take only the bits we need an adapt them accordingly.

```javascript
function IncomeTax(){}

// create a record for each job type and the total earned from each
IncomeTax.prototype.jobs = function(jobs) {
  this.jobRecord = jobs.reduce((record, job) => {
    record[job.title] = {};
    record[job.title].total = Total(job.amounts);
    return record;
  }, {});
};

// calculate the total from all jobs
IncomeTax.prototype.total = function() {
  this.totalIncome = Object.keys(this.jobRecord)
                      .map(job => this.jobRecord[job].total)
                      .reduce((total, job) => total += job, 0);
};

IncomeTax.prototype.taxToPay = function() {
  this.tax = Percent(22, this.totalIncome); // flat rate of 22% tax (if only!)
};

var work = [{ title : "freelance", amounts : [100, 340, 700] },{ title : "contracts", amounts : [500, 770, 350] }];
var myTax = new IncomeTax();
myTax.jobs(work);
myTax.total();  // 2760
myTax.taxToPay(); // 607.2

```

### The Revealing Module Pattern

There are a multitude of patterns that combine one or more features from closures, prototypal inheritance, IIFEs, factories and composition to create objects in JavaScript. The revealing module pattern combines closure, IIFEs and factories. Take the following example:

```javascript
var Greeting = (function() {
  var greet = "Hello";
  function sayHi(person) {
    return `${greet} ${person}`;
  }
  return { sayHi };
})();

Greet.sayHi("Mike");  // Hello Mike

```

The module pattern provides encapuslation and control over what is exposed publically. It is by nature a singleton as there'll only ever be one instance of it and so we always operate directly on that one instance. We also don't have to assign a variable to its return value as it's an expression so we get that for free via the IIFE. Not surprisingly the revealing module pattern is a popular pattern.

### The Revealing Prototype Pattern

To emphasise the malleability of JavaScript, it's possible to re-engineer the prototype pattern into one that supports private properties. By default the <code>prototype</code> property of a function is an object, but we can sidestep this requirement by instead assigning it to an IIFE that returns an object.

```javascript
function Greeting(){}
Greeting.prototype = (function() {
  var greet = "Hello";
  function sayHi(person) {
    return `${greet} ${person}`;
  }
  return { sayHi };
})();

var foo = new Greeting();
foo.sayHi("Mike");  // Hello Mike

```

Greeting can still be extended through its prototype, however with this pattern it is preferable to include all necessary properties when the constructor is first defined, so each method can benefit from the encapsulation it provides. This extra ability makes the revealing prototype a very useful pattern.

### Summary

There are a plethora of possible ways to create objects in JavaScript, each with certain characteristics or idiosyncracies that give them specific advantages or disadvantages in relation to other patterns.


## Data Types

As of ES6 there are seven different data types in JavaScript - 

* String
* Number
* Boolean
* Undefined
* Symbol
* Object

### Primitives and Objects

All types, except objects, are known as primitives. All primitive values are immutable (values that are incapable of being changed). When creating a primitive as a type literal - i.e. without the use of its constructor - the primitive simply represents a value of that type in memory, therefore leaving a very small memory footprint. Consider the following:

```javascript
// primitive string
var foo = "foo";
typeof foo === "string";  // true
// primitive number
var bar = 1;
typeof bar === "number";  // true
// primitive boolean
var foobar = true;
typeof foobar === "boolean";  // true

//primitive values
foo;  // foo
bar;  // 1
foobar; // true

// object string
var foo = new String("foo");
typeof foo === "object";  // true
// object number
var bar = new Number(1);
typeof bar === "object";  // true
// object boolean
var foobar = new Boolean(true);
typeof foobar === "object";  // true

// object value
foo;  // String { 0: "s", 1: "t", 2: "r", length: 3, [[PrimitiveValue]]: "str" }
bar;  // Number { [[PrimitiveValue]]: 1 }
foobar; // Boolean { [[PrimitiveValue]]: true }

```

The above demonstrates the difference in resulting value when defining a variable as a literal or via a constructor. As a constructor returns an object, the memory footprint of is obviously larger than that of a primitive. But what if, for example, you had a primitive string that needed access to its length property (on the String prototype)? It wouldn't be unreasonable to assume the string would've needed to be created via its constructor to get access to this property, as a primitive is only the value and doesn't inherit from the constructor. Fortunately this is not the case.

When attempting to access a prototype property on a literal, JavaScript coerces the primitive into an object for the operation, giving temporary access to static and prototypal properties. As soon as the operation is completed these properties are garbage collected and the variable is restored to a primitive value. 

```javascript
// primitive
var foo = "foo";
// temporary access to an object property 
foo.length; // 3
// restored to primitive
typeof foo; // string
// set property
foo.bar = "bar";
// primitives are immutable
foo.bar;  // undefined

```

### Null and Undefined

Null and Undefined are both types in JavaScript. The difference between the two is simply a case of implict or explicit declaration. Both are meant to represent a yet to be determined value. However, a null value has to be explicitly declared, it does not occur organically, unlike a value of undefined which is automatically assigned to a variable that is declared but not defined.

```javascript
// foo is declared but its value is yet to be determined
var foo;
foo;  // undefined

// explictly declared that its value is yet to be determined
foo = null;
foo;  // null

```

A common source of confusion between the two comes from the result of the <code>typeof</code> operator when used against them. Pre ES6 Undefined is recognised as a type but Null, due to a bug in the ECMAScript implementation, is interpreted as an object.

```javascript
var foo;
typeof foo === "undefined"; // true

var bar = null;
typeof bar === "object"; // true

```

Pre ES6, null was the only way to declare properties of objects that were yet to be defined.

```javascript
// Pre ES6, a yet to be determined value within an object
var foo = { bar : null };
foo.bar;  // null

// ES6 via object destructuring
var bar;
var foo = { bar };
foo.bar;  // undefined

```

### Variable binding and unbinding

JavaScript is a weakly typed, dynamic language which means that firstly variables do not have to be declared with a specific type and secondly that variables can be rebound from one value (regardless of type) to another.

```javascript
var obj = {};
typeof obj === "object"; // true
obj = function(){};
typeof obj === "function"; // true

```

In JavaScript functions and variables are given names to associate them with the objects or values they represent. The names given are often referred to as identifiers or references. It is important to remember whenever dealing with idenitifiers that they point only at values and not at each other. Consider the following example:

```javascript
var foo = { name : "bar" };
var bar = foo;

bar.name; // bar
foo === bar;  // true

```

It wouldn't be unreasonable after seeing the above to think that <code>bar</code> is a reference to <code>foo</code> as, firstly we assigned <code>bar</code> to <code>foo</code>, secondly it has access to a property on <code>foo</code> and a strict equality check between the two returns true. This isn't the case though. By assigning <code>bar</code> to <code>foo</code> we've just pointed another identifier to wherever the object value of <code>foo</code> is held in memory. Therefore any comparisons between the two are comparing the same value in memory. Consider the following:

```javascript
// re-assign `foo` to a new object
foo = { name : "foo" };

// `bar` still points to the old object
bar.name; // bar
// therefore their values are no longer the same
foo === bar;  // false

```

Assigning the <code>foo</code> identifier to another object in memory does not affect <code>bar</code> which still points to the original value of <code>foo</code>. This may all seem fairly obvious but it's a concept often not fully appreciated and is worth highlighting.

### Mutation

Mutation is similar to but not the same as rebinding. Rebinding affects the identifiers whilst mutation affects the values. Consider the following:

```javascript
var foo = { name : "bar" };
// mutation
foo.name = "foo";
// rebinding
foo = { name : "foo" };

// rebinding
foo = [1, 2, 3];
// mutation
foo.push(4);
// mutation
foo[0] = 0;

```


### Summary

Variables defined as type literals, besides object, are known as primitives. They are represented in memory as their primitive values. Variables defined from a type constructor are objects and are stored in memory as objects. They consequently contain all properties from their constructor's prototype and therefore have a larger memory footprint than primitives. Since primitives can still temporarily access properties on their type constructor's prototype they are nearly always the preferable choice when defining variables over their constructor counterparts. When pointing a variable at another, the identifier points to its value, not to the reference itself.
