# ES6 concepts

**Array Function Expression**

> An arrow function expression has a shorter syntax than a `function expression` and does not bind its own `this`, `arguments`, `super`, or `new.target`. These function expressions are best suited for `non-method` functions, and they `cannot` be used as `constructors`.
>
> 1. `arguments`: The arguments object is a local variable 	available within all functions. You can refer to a function's arguments within the function by using the arguments object. This object contains an entry for each argument passed to the function, the first entry's index starting at 0.
> 2. `new.target`: The `new.target` property lets you detect whether a function or constructor was called using the new operator. In constructors and functions instantiated with the new operator, `new.target` returns a reference to the constructor or function. In normal function calls, new.target is undefined.

How no binding of `this` is beneficial? Consider the below snippet:
```js
function Person() {
	// The Person() constructor defines `this` as an instance of itself.
	this.age = 0;

	setInterval(function growUp() {
		// In non-strict mode, the growUp() function defines `this` 
		// as the global object, which is different from the `this`
	    	// defined by the Person() constructor.
		this.age++;
	}, 1000);
}
var p = new Person();
```
In ECMAScript 3/5, the this issue was fixable by assigning the value in this to a variable that could be closed over.

```js
function Person() {
	var that = this;
  	that.age = 0;

  	setInterval(function growUp() {
    		// The callback refers to the `that` variable of which
    		// the value is the expected object.
    		that.age++;
  	}, 1000);
}
```

Alternatively, a bound function could be created so that the proper this value would be passed to the bound target function (the `growUp()` function in the example above).

An arrow function does not create its own `this` context, so `this` has its original meaning from the `enclosing context`. Thus, the following code works as expected:

```js
function Person(){
    this.age = 0;
    setInterval(() => {
        this.age++;
	// |this| properly refers to the person object
    }, 1000);
}
var p = new Person();

```
Some properties of `arrow functions`:

1. Does not have `prototype` property.
2. Does not bind to `this`.
3. Use of `new` operator with arrow function will raise error.
4. Arrow functions can not be used as `generators`.
5. An arrow function cannot containa line break between its parameters and its arrow.

---

**Classes in JS**
> JavaScript classes introduced in ECMAScript 2015 are syntactical sugar over JavaScript's existing `prototype-based` inheritance. The class syntax is `not` introducing a new object-oriented inheritance model to JavaScript. JavaScript classes provide a much simpler and clearer syntax to create objects and deal with inheritance.

**Defining Classes**
>Classes are in fact "special `functions`", and just as you can define `function expressions` and `function declarations`, the class syntax has two components: `class expressions` and `class declarations`.

* _Class Declaration_: One way to define a class is using a `class declaration`. To declare a class, you use the `class` keyword with the `name` of the class ("Rectangle" here).
        
```js
class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}
```

_Hoisting_:
An important difference between `function declarations` and `class declarations` is that `function declarations` are `hoisted` and `class declarations` are `not`. You first need to declare your class and then access it, otherwise code like the following will throw a `ReferenceError`:

```js
var p = new Rectangle(); // ReferenceError

class Rectangle {}
```

* _Class Expression_: A class expression is another way to define a class. Class expressions can be named or unnamed. The name given to a named class expression is local to the class's body.

```js
// unnamed
var Rectangle = class {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
};

// named
var Rectangle = class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
};
```

**Note**: Class expressions also suffer from the same hoisting issues mentioned for Class declarations.


**Constructor**
>The constructor method is a special method for creating and initializing an object created with a class. There can only be `one special method` with the name "constructor" in a class. A `SyntaxError` will be thrown if the class contains `more` than `one` occurrence of a `constructor` method.
A constructor can use the `super` keyword to call the constructor of a parent class.

**Prototype Methods**
```js
class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
  
  get area() {
    return this.calcArea();
  }

  calcArea() {
    return this.height * this.width;
  }
}

const square = new Rectangle(10, 10);

console.log(square.area);
```

**Static Methods**
>The `static` keyword defines a static method for a class. Static methods are called `without` instantiating their class and `cannot` be called through a `class instance`. Static methods are often used to `create` `utility functions` for an application.

```js
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  static distance(a, b) {
    const dx = a.x - b.x;
    const dy = a.y - b.y;

    return Math.sqrt(dx*dx + dy*dy);
  }
}

const p1 = new Point(5, 5);
const p2 = new Point(10, 10);

console.log(Point.distance(p1, p2));
```

---

**Template Literals**
>Template literals are string literals allowing `embedded expressions`. You can use multi-line strings and `string interpolation` features with them. They were called "template strings" in prior editions of the ES2015 specification.

**Syntax**
```js
`string text`

`string text line 1
 string text line 2`

`string text ${expression} string text`

tag `string text ${expression} string text`
```

>Template literals are enclosed by the back-tick (` `)  character instead of `double or single` quotes. Template literals can contain `place holders`. These are indicated by the `Dollar sign` and `curly braces` `(${expression}).` The expressions in the place holders and the text between them get passed to a function. The `default function` just `concatenates` the parts into a `single string`. If there is an expression preceding the template literal (tag here),  the template string is called "tagged template literal". In that case, the tag expression (usually a function) gets called with the processed template literal, which you can then manipulate before outputting. To `escape` a `back-tick` in a template literal, put a backslash \ before the back-tick.


---

**let**
>The **let** statement declares a `block scope` local variable, optionally initializing it to a value.

**Syntax**

```js
let var1 [= value1] [, var2 [= value2]] [, ..., varN [= valueN]];
```
**Parameters**

* var1, var2, …, varN
	Variable name. It can be any legal identifier.
	
* value1, value2, …, valueN
	Initial value of the variable. It can be any legal expression.

**Description**
>`let` allows you to declare variables that are limited in scope to the block, statement, or expression on which it is used. This is unlike the `var` keyword, which defines a `variable globally`, or `locally to an entire function` regardless of `block scope`.

**Scoping rules**
>Variables declared by `let` have as `their scope` the `block` in which they are defined, as well as in any contained sub-blocks . In this way, `let` works very much like `var`. The main difference is that the `scope` of a `var` variable is the `entire enclosing function`:

```js
function varTest() {
  var x = 1;
  if (true) {
    var x = 2;  // same variable!
    console.log(x);  // 2
  }
  console.log(x);  // 2
}

function letTest() {
  let x = 1;
  if (true) {
    let x = 2;  // different variable
    console.log(x);  // 2
  }
  console.log(x);  // 1
}
```

_Cleaner code in inner functions_
>`let` sometimes makes the code cleaner when `inner functions` are used.

```js
var list = document.getElementById('list');

for (let i = 1; i <= 5; i++) {
  let item = document.createElement('li');
  item.appendChild(document.createTextNode('Item ' + i));

  item.onclick = function(ev) {
    console.log('Item ' + i + ' is clicked.');
  };
  list.appendChild(item);
}

// to achieve the same effect with 'var'
// you've to create a different context
// using a closure to preserve the value
for (var i = 1; i <= 5; i++) {
  var item = document.createElement('li');
  item.appendChild(document.createTextNode('Item ' + i));

    (function(i){
        item.onclick = function(ev) {
            console.log('Item ' + i + ' is clicked.');
        };
    })(i);
  list.appendChild(item);
}
```
The example above works as intended because the `five` instances of the (anonymous) `inner function` refer to `five different instances` of the variable `i`. Note that it does not work as intended if you replace `let` with `var`, since all of the `inner functions` would then return the `same final value` of `i`: 6. Also, we can keep the scope around the loop cleaner by moving the code that creates the new elements into the scope of each loop.

---

**Constant**
>Constants are block-scoped, much like variables defined using the `let` statement. The value of a constant cannot change through re-assignment, and it can't be `redeclared`.

**Syntax**
```js
const name1 = value1 [, name2 = value2 [, ... [, nameN = valueN]]];
```

This declaration creates a constant that can be either `global` or `local` to the function in which it is declared. An initializer for a constant is required; that is, you must specify its value in the same statement in which it's declared (which makes sense, given that it can't be changed later).

The const declaration creates a `read-only` reference to a value. It does not mean the value it holds is `immutable`, just that the variable identifier cannot be `reassigned`. For instance, in case the content is an object, this means the `object` itself can still be `altered`.

**Example**
```js
// NOTE: Constants can be declared with uppercase or lowercase, but a common
// convention is to use all-uppercase letters.

// define MY_FAV as a constant and give it the value 7
const MY_FAV = 7;

// this will throw an error
MY_FAV = 20;

// will print 7
console.log('my favorite number is: ' + MY_FAV);

// trying to redeclare a constant throws an error
const MY_FAV = 20;

// the name MY_FAV is reserved for constant above, so this will also fail
var MY_FAV = 20;

// this throws an error also
let MY_FAV = 20;

// it's important to note the nature of block scoping
if (MY_FAV === 7) { 
    // this is fine and creates a block scoped MY_FAV variable 
    // (works equally well with let to declare a block scoped non const variable)
    const MY_FAV = 20;

    // MY_FAV is now 20
    console.log('my favorite number is ' + MY_FAV);

    // this gets hoisted into the global context and throws an error
    var MY_FAV = 20;
}

// MY_FAV is still 7
console.log('my favorite number is ' + MY_FAV);

// throws an error, missing initializer in const declaration
const FOO; 

// const also works on objects
const MY_OBJECT = {'key': 'value'};

// Attempting to overwrite the object throws an error
MY_OBJECT = {'OTHER_KEY': 'value'};

// However, object keys are not protected,
// so the following statement is executed without problem
MY_OBJECT.key = 'otherValue'; // Use Object.freeze() to make object immutable

// The same applies to arrays
const MY_ARRAY = [];
// It's possible to push items into the array
MY_ARRAY.push('A'); // ["A"]
// However, assigning a new array to the variable throws an error
MY_ARRAY = ['B']
```

