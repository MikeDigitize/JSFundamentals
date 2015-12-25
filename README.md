# AOJSGuide
A guide to JavaScript.

## The JavaScript Compiler

JavaScript is a compiled programming language which means any JavaScript code ran in a JavaScript environment, such as a browser or Node, first gets compiled by the engine before it's executed. It is at this time, during compilation, when scope is defined.

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

In JavaScript you describe your data and your data handlers either as variables or functions. The compiler determines function availability in the same way it does with variables, so functions declared within functions are not available outside of their parent. Functions can be written either as declarations or expressions, the difference between the two, other than the way they're declared (see below), is in the way the compiler treats them.

```javascript
// declaration
function bar() {}

// expression
var foo = function() {}

```

### Let and const

ES6 introduced two new ways for variables to be declared - <code>const</code> and <code>let</code>. These both have different scoping criteria to var and function, and are also handled differently by the compiler. Const and let are lexically block scoped which means they are available only within the block <code>{ }</code> they are declared in.

```javascript
for (let i = 0; i < 5; i++) {
  // i available within
}

// error
// i is scoped to the for loop block and not available externally
console.log(i);
```

### Summary
Scope dictates the availablity of variables within your code. Variables can be scoped globally, within a function or within a block. Variable scope is determined in the compilation stage, before the code is executed. Scope determined at compile time is known as lexical scoping.

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

// post compiler
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

This property gives the function a reference to its parent scope, and its parent's parent scope, all the way up to the global scope. When looking up variable references the JavaScript engine will check against a function's immediate scope. If not found it looks to its parent, and then its parent's parent until it reaches the global scope where, if it's not found, the engine will throw a reference error.

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

The variable <code>foobar</code> is defined in the scope of <code>bar</code> and so is lexically scoped to <code>bar</code> and any of its descendant functions. When <code>bar</code> is called, a new object is created - its <code>environment</code>, which holds anything in its lexical scope. The same thing happens for the inner function <code>foo</code>. When <code>foo</code> is called as the return value of <code>bar</code>, it looks up the value of <code>foobar</code> against its <code>variable environment</code>. It's not in its immediate scope so it looks for it in its parent scope where it finds it and is able to return it.

### Summary

At runtime, when a function is called, the JavaScript engine creates an object representing that function's variable <code>environment</code> which holds references to everything in its lexical scope. When asked to lookup a variable referenced within the function, the engine first checks against its immediate scope via its <code>environment</code>. If it doesn't find it there, it continues to search against each parent until it reaches the global scope. A function's <code>environment</code> is created at runtime as the code executes and is known as dynamic scoping.

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

Context is defined based on the location the function was called from - known as the call site. In the above example the function <code>bar</code> is called in the global scope and attempts to log a variable attached to <code>this</code>. The variable it attempts to log - <code>a</code> - has been defined in the global scope. Since the function was called in the global scope, <code>this</code> refers to the global object and therefore logs <code>a</code> from the global scope.

This is known as the default context rule. Be aware that in strict mode the default context rule is ignored if the scope is global and <code>this</code> will be undefined. To see how the default rule changes according to the call site, take the following example:

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

The object returned from the constructor has a method <code>sayName</code> which returns a reference to <code>this</code>. Even though the call to the function is made in the global scope, its call site (and therefore context) is the object <code>foo</code>. In the above we can see that <code>this</code>, when used in a constructor or any property on a constructor's prototype, refers to the instance of the constructor. It can therefore be used to set values specific to that instance. Prototypal inheritance will be covered in a later chapter.

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
Context or <code>this</code> in JavaScript is a reference to a function's execution context. It is dynamic in nature and set at runtime, not during compilation. Context is defined organically either as the call site of the function or by using a constructor function, or set explicitly through the use of call, apply, bind and fat arrow functions.

## Closures
Understanding closures is something that can only really be done when aware of how lexical and dynamic scoping in JavaScript work. Take the following example:

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

In the above the function foo has access to the variable <code>a</code> from its parent scope. This access is set in the compilation stage, so <code>a</code> is lexically scoped to the function foo. When foo is called, the JavaScript engine looks up <code>a</code> in its immediate scope through its variable environment. It won't find it, so will then look it up against foo's parent scope, the function bar, where it will find it, and can log its value. This is, in very simple terms, a closure. The function foo has closed over the scope of the function bar (and anything in bar's parent scope, and so on until it reaches the global scope) and retains access to any variables in its scope.

The code above might help illustrate closures in their most simple form but it doesn't demonstrate their usefulness as a pattern, not without a small modification. In JavaScript functions are first class values, which means they are treated like all other values and can be passed as arguments into and returned from functions. Consider the following, a modified version of the previous code example:

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

This is the crux of closures. It is the ability to access values outside of their lexical scope. Lexical scope defined at the compilation stage is accessible at runtime even if the reference is made outside of the scope it was defined in. This ability is provided by the variable environment created at runtime when <code>bar</code> is called. In the above the function <code>bar</code> has returned and is no longer in use, yet closures still allow access to values defined inside functions that have returned. To help illustrate a more practical use of closures consider the following piece of code;

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

What has happened is the returned function has closed over the scope of its parent. At compilation time the value of <code>percent</code> is undefined, but this doesn't matter - the returned function still has access to its reference through scope. When the percentage function is called the value it gets called with defines <code>percent</code>, essentially pre-loading the returned function with one of the two values needed to make the calculation. Now when the returned function is called, it calculates the pre-loaded percent of the value it's passed. This technique is often referred to as partial application or currying and allows functions like percentage to be re-used over and over producing new functions pre-loaded with different values.

```javascript
var vat = percentage(20);
vat(30);  // 6

var half = percentage(50);
half(100);  // 50

```

### IIFE

It is possible to create closure like effects at runtime through the use of IIFEs (Immediately Invoked Function Expressions). An IIFE is a function wrapped in an expression and immediatley called<code>(function(){})()</code>. Whereas with closures, variables are able to be retrieved through reference, IIFEs run immediately so close around the value at the time of execution, essentially preserving a snapshot of the value at that point within the script. To illustrate that, consider the following example:

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
    console.log(`I am button ${i}`);  // es6 template strings will be covered later
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

Closures can be thought of as a tool to allow runtime access to variables outside of their lexical scope. Their use is best demonstrated when functions return functions, and the returned function has within it a reference to a value that is not in its immediate scope. That reference has essentially been trapped in that function, and no matter where you call this returned function from it will have access to that value. 

IIFEs work in a similar way, only they close over the current value of the variable they reference at runtime. They are expressions and contain a function that is immediately invoked inside them. This immediate invocation forces the function scope to close over the values of variables it references that are within its lexical scope, creating what can be thought of as a snapshot of their value at that time. 

## Prototypal inheritance

JavaScript is an object oriented language. Everything is an object or behaves like one. Functions are objects. They can have properties and methods like any other object. Despite that, functions and objects should be thought of as two separate constructs; two constructs which essentially form the building blocks of JavaScript. JavaScript's inheritance model is not class based like other OOP languages such as Java or C#, it is prototype based. 

A prototype based system has two fundamental components - a <code>constructor</code>, which is a function, and a property on the function - <code>prototype</code>, which is an object. Constructors create objects. Every object you use within JavaScript has been created by a constructor. Every object created has a <code>constructor</code> property. You'd think this is a reference to the constructor function which created the object but alas it isn't quite that simple. The first step towards understanding prototypal inheritance is to recognise how the return value of a function differs when called with and without the <code>new</code> keyword.

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

When a function is declared, the function object that is created as a result gets a <code>prototype</code> property - an empty object - automatically attached to it. It is this property that facilitates prototypal inheritance. The example demonstrates how:

```javascript
function A(){}
var foo = new A();
foo.bar;  // undefined
A.prototype.bar = "foobar";
foo.bar; // foobar

```

In the above a <code>bar</code> property is set on the prototype property of <code>A</code>. Any instance of <code>A</code> now has access to that property, demonstrating an implicit link between constructor and instance, behind the scenes. The above outlines how the <code>new</code> keyword alters a function's return value. But what exactly happens when <code>new</code> is used? Well firstly and most obviously a new object is created - the new instance of the constructor, and this instance has an implicit link to the prototype of its constructor. Properties defined on the constructor's prototype do not appear on the instance, but can be referenced as if they were.

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
<!-- DOM -->
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

This is common behaviour so it makes sense for it to be wrapped in a re-usable function. It's important to recognise the reduction in memory consumption that prototypal inheritance offers. Any new instances of the above will be able to retrieve DOM elements but the function that sets the DOM element is only defined once, on the constructor's prototype. Every instance does not carry its own version of that function, they all just point to the one on the prototype therefore saving memory. Something else that is common behaviour, besides retrieving DOM elements, is adding and removing event listeners to DOM elements. So wouldn't it be useful if we could extend the above to accomodate event handling? Consider the following: 

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

Prototypal inheritance is a very useful pattern but there are a few limitations with it. Firstly, it doesn't allow for private properties - everything must be attached to the instance at instantiation or through the prototype, making all its properties publicly available. Secondly, when extending prototypes there is no mechanism to inherit only the methods or properties that are needed - extending means inheriting everything, which is not always ideal. Thirdly, there is its over reliance on <code>this</code> and the potential for that reference to be overwritten by one of the means discussed earlier in the context chapter, making code within potentially brittle. 

## Static methods

As functions are objects it's possible to store properties on the constructor itself. These are known as <code>static</code> properties. Any instances of the constructor do not inherit these properties as they're not on its <code>prototype</code>.

```javascript
var foo = { a : "a", b : "b", c : "c" };
// `keys` is a static property on the Object function
Object.keys(foo).forEach(key => console.log(key));  // a, b, c

```

### Summary

Prototypal inheritance is the mechanism JavaScript uses to allow objects to inherit from functions. Its facilitated through the use of a function - the constructor - and the <code>new</code> keyword. Whenever a new instance of a constructor is created it inherits any properties that are on the constructor's <code>prototype</code> property. The link between the instance and the constructor remains even after the new instance has been created. Subsequent additions or changes to any property on the <code>prototype</code> will be reflected on every instance. Any properties added to the constructor directly are known as static properties and are available only via the constructor and not on any instances of it.

## JavaScript Design Patterns

Prototypal inheritance is not the only way to create objects. With a language as flexible as JavaScript there are a multitude of alternatives available if prototypal inheritance does not fit the use case.

### Factories

A factory is any function that returns an object. Constructor functions are factories. When a constructor function is invoked with the <code>new</code> keyword, it returns an object. This forces functions to behave differently than they usually would when called, for example, as we saw earlier, they return things (objects) that aren't explicitly returned. But what if we used standard functions that returned objects instead of constructors and omitted the <code>new</code> keyword?

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

Factory functions have an obvious limitation in that they don't support inheritance. But inheritance is not always the best solution for sharing methods. When an instance inherits from a constructor's blueprint it inherits everything on its prototype. But what if we only actually need one or two of its properties? With this in mind an often preferable alternative to pure inheritance is composition - creating objects composed of other objects or functions. Think of the difference between the two as - inheritance is when objects are defined based on what they are whereas composition is objects defined based on what they do. The two can be mixed together though. Consider the following:

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

// create a record for each job type and the total earnt from each
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
foo;  // "foo"
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
foo.length; // 2
// restored to primitive
typeof foo; // "string"
// set property on primitive - temporary access to an object property
foo.bar = "bar";
// restored to primitive
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

### Summary

Variables defined as type literals, besides object, are known as primitives. They are represented in memory as their primitive values. Variables defined from a type constructor are objects and are stored in memory as objects. They consequently contain all properties from their constructor's prototype and therefore have a larger memory footprint than primitives. Since primitives can still temporarily access properties on their type constructor's prototype they are nearly always the preferable choice when defining variables over their constructor counterparts.

### Chaining

As long as a function returns a function in some form it is able to be chained.

```javascript
function andOn(){ 
  return { 
    andOn : () => this 
  } 
}
var on = andOn();
on.andOn().andOn().andOn().andOn(); // etc

```









