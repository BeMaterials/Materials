## Console

```js
console.log("Hello World!");
console.error("This is an error"); // Shows red in the console
console.warn("This is a warning"); // Shows yellow in the console
```

## var, let, const

- Varibale and function names can contain letters, digits, underscores, and dollar signs but not start with a digit.
- A variable declared without a value will have the value `undefined`. The type is also undefined.
- `var` is globally (or functionally if it defined inside of a function) scope (do not use var anymore).
- `let` and `const` are block scoped.
- In HTML, the global scope is the window object. All global variables belong to the window object.
- Hoisting is JavaScript's default behavior of moving all declarations to the top of the current scope (to the top of the current script or the current function). So in JavaScript, a variable or a function can be declared after it has been used (not with let or const. So arrow functions are not hoisted as well.). Note that JavaScript only hoists declarations, not initializations.

  ```js
  carName = "Volvo"; // OK
  alert(carName);
  var carName;

  carName = "Volvo"; // ReferenceError
  alert(carname);
  let carName;
  ```

```js
let volume = 30;
volume = 31;

console.log(volume);
```

- Always use const unless you know, you are going to reassign the variable.
- You have to initialize when using `const`.

```js
const rate = 20;
```

## Primitive data types

Strings, Numbers, Boolean, null, undefined

- In `js`, unlike `c#`, string is value-type. It can be object but not recommended.
- Primitive values are immutable in JS.

```js
const name = "Ben";
const age = 30; // JavaScript numbers are always stored as double precision floating point numbers.
const rating = 4.5;
const y = 123e-5; // 0.00123 -> Extra large or extra small numbers can be written with scientific (exponent) notation.
const isCool = true;
const x = null;
const y = undefined;
let z; //undefined too

// typeof operator always returns string -> the type of operand
console.log(typeof name); //string
console.log(typeof age); //number
console.log(typeof rating); //number
console.log(typeof isCool); //boolean
console.log(typeof x); //object! It is a bug in js.
console.log(typeof y); //undefined -> undefined and null are equal in value but different in type: x == y but x !==y
console.log(typeof z); //undefined
```

## Complex data types

```js
typeof { name: "John", age: 34 }; // Returns "object"
typeof [1, 2, 3, 4]; // Returns "object" (not "array", see note below)
typeof null; // Returns "object"
typeof function myFunc() {}; // Returns "function"
```

- Strings Can be Objects. Normally, JavaScript strings are primitive values, created from literals:

```js
var firstName = "John";
```

- But strings can also be defined as objects with the keyword new:

```js
var firstName = new String("John");
```

- Don't create numbers or strings as objects with `new` keyword. It slows down execution speed and you cannot compare two strings in that way because they are objects (Comparing two JavaScript objects will always return false)!

- In summary, in JavaScript there are 5 different data types that can contain values:
  - string
  - number
  - boolean
  - object
  - function
- There are 6 types of `objects` (these are constructor functions):
  - Object
  - Date
  - Array
  - String
  - Number
  - Boolean
- And 2 data types that cannot contain values:
  - null
  - undefined
- The constructor property returns the constructor function for all JavaScript variables.

```js
function isArray(myArray) {
  return myArray.constructor === Array;
}
```

### Summary

```js
var x1 = {}; // new object
var x2 = ""; // new primitive string
var x3 = 0; // new primitive number
var x4 = false; // new primitive boolean
var x5 = []; // new array object
var x6 = /()/; // new regexp object
var x7 = function () {}; // new function object
```

### Converting

```js
String(123)(
  // returns a string from a number literal 123
  123
).toString();

String(false); // returns "false"
false.toString(); // returns "false"

String(Date()); // returns "Thu Jul 17 2014 15:38:19 GMT+0200 (W. Europe Daylight Time)"
Date().toString(); // returns "Thu Jul 17 2014 15:38:19 GMT+0200 (W. Europe Daylight Time)"

Number("3.14"); // returns 3.14
parseFloat("3.14");

// The unary + operator can be used to convert a variable to a number:
var y = "5"; // y is a string
var x = +y; // x is a number

var y = "John"; // y is a string
var x = +y; // x is a number (NaN)

Number(false); // returns 0

d = new Date();
Number(d); // returns 1404568027739

d = new Date();
d.getTime(); // returns 1404568027739
```

### Automatic Type Conversion

```js
5 + null; // returns 5         because null is converted to 0
"5" + null; // returns "5null"   because null is converted to "null"
"5" + 2; // returns "52"      because 2 is converted to "2"
"5" - 2; // returns 3         because "5" is converted to 5
"5" * "2"; // returns 10        because "5" and "2" are converted to 5 and 2

document.getElementById("demo").innerHTML = myVar;

// if myVar = {name:"Fjohn"}  // toString converts to "[object Object]"
// if myVar = [1,2,3,4]       // toString converts to "1,2,3,4"
// if myVar = new Date()      // toString converts to "Fri Jul 18 2014 09:08:55 GMT+0200"

// if myVar = 123             // toString converts to "123"
// if myVar = true            // toString converts to "true"
// if myVar = false           // toString converts to "false"
```

## Strings

### Concatenation

#### Template String

1. use of placeholders,
2. using " and ' freely,
3. multi-lines

```js
console.log(`My name is ${name} and I am ${age}`);
```

### String properties and methods

- String are immutable in JS and we cannot have `s[0] = 'J';`.

```js
const s = "Hello World ";

console.log(s.length); //12
console.log(s.indexOf("l")); //2
console.log(s.indexOf("l", 5)); //9 -> second parameter as the starting position for the search
console.log(s.indexOf("Zello")); //-1
console.log(s.lastIndexOf("l")); //9
console.log(s.search("l")); //It is like indexOf but can take powerful search values (regular expressions)
console.log(s[0]); //H -> equal to old way of s.charAt(0)
console.log(s.charCodeAt(0)); //72
console.log(s.toLowerCase());
console.log(s.toUpperCase());
console.log(s.substring(0, 5)); //'Hello' -> it is similar to slice method but slice can take negative values as well. If a parameter is negative, the position is counted from the end of the string.
console.log(s.substr(6, 5)); //'World' -> the second parameter specifies the length of the extracted part
console.log(s.substr(6)); //'World ' -> If you omit the second parameter, substr() will slice out the rest of the string.
console.log(s.substr(-6)); //'World ' -> If the first parameter is negative, the position counts from the end of the string.
console.log(s.replace(/hell/i, "Heaven")); //replace() method is case sensitive and replaces only the first match. To replace all matches, use a regular expression with a /g flag (global match).
console.log(s.split(" ")); //returns an array
console.log(s.split("")); //Split in characters
console.log(s.trim());
console.log(s.startsWith("Hell"));
console.log(s.endsWith("ld"));
console.log(s.includes("i"));
```

## Numbers

```js
console.log(0xff, 0b111111111, 0o1000); // Hex, binary and octal numbers

typeof Infinity; // returns "number"
var x = 2 / 0; // x will be Infinity
var y = -2 / 0; // y will be -Infinity

typeof NaN; // returns "number"
var x = 100 / "Apple";
isNaN(x); // returns true because x is Not a Number

var myNumber = 32;
myNumber.toString(10); // returns 32
myNumber.toString(32); // returns 10
myNumber.toString(16); // returns 20
myNumber.toString(8); // returns 40
myNumber.toString(2); // returns 100000
```

### Number methods and properties

```js
var x = 123;

x.toString(); // returns 123 from variable x
x.toExponential(2); // returns 1.23e+2 -> returns a string, with a number rounded and written using exponential notation.
x.toFixed(2); // 123.00 -> returns a string, with the number written with a specified number of decimals
x.toPrecision(4); // 123.0 -> returns a string, with a number written with a specified length

x = Number("10"); //10 -> If the number cannot be converted, NaN (Not a Number) is returned.
Number("10 20 30"); // NaN
Number(new Date("2017-09-30")); // returns 1506729600000
parseInt("10"); // returns 10
parseInt("10.33"); // returns 10
parseInt("10 20 30"); // returns 10 -> Only the first number is returned
parseFloat("10"); // returns 10
parseFloat("10.33"); // returns 10.33

var x = Number.MAX_VALUE;
var x = Number.MIN_VALUE;
var x = Number.POSITIVE_INFINITY;
var x = Number.NEGATIVE_INFINITY;
var x = Number.NaN;
```

for numbers we have these methods as well:

- Number.isNaN(a),
- Number.isFinite(a),
- Number.isInteger(a)
- isFinite(10/0); // returns false

## Arrays

```js
const numbers = new Array(1, 2, 3, 4, 5);
const fruits = ["apples", "oranges", "pears", 1, true];
```

- The two examples above do exactly the same. There is no need to use new Array(). For simplicity, readability and execution speed, use the first one (the array literal method).

- Accessing elements:

```js
console.log(fruits[0]);
```

- Although it is a const but you can manipulate the members. You cannot reassign the whole array: `fruits = [];`.

```js
fruits[6] = "grapes";
```

- We also have another concept, `associative arrays` which is like objects and you can assign a key to each member instead of index. Array methods and properties will produce undefined or incorrect results in this case. So it is basically objets and don't use associative arrays.

### Array methods

```js
fruits.push("mangos"); //Adds member to the end.
fruits[fruits.length] = "Lemon"; // adds a new element (Lemon) to fruits (just like above)
fruits.unshift("strawberries"); //Adds member to the start.
fruits.pop(); //Removes the last one.
fruits.shift(); // Removes the first element.
console.log(Array.isArray(name)); //Checks if sth is an array.
fruits instanceof Array; // returns true. Another way to check if sth is an array.
console.log(fruits.indexOf("oranges"));
// array.indexOf(item, start) -> start can be negative counting form the end -> we also have lastIndexOf

const numbers = [1, 2, 3, 4, 5];
console.log(numbers.every((number) => number > 0)); //true
console.log(numbers.every((number) => number > 3)); //false

console.log(numbers.some((number) => number > 0)); //true
console.log(numbers.some((number) => number > 3)); //true
console.log(numbers.some((number) => number > 10)); //false

console.log(numbers.includes(6)); //false
console.log(numbers.includes(4)); //true

numbers.splice(1, 2); // using splice to remove elements -> from postion 1 remove two elements
console.log(numbers); //[ 1, 4, 5 ]
fruits.splice(2, 0, "Lemon", "Kiwi"); // using splice to add elements -> In postion 2 remove 0 elements and add these elements

numbers.reverse();
console.log(numbers); //[ 5, 4, 1 ]

numbers.sort();
console.log(numbers); //[ 1, 4, 5 ]
// numbers.sort(function(a, b){return a - b}); //descending 5, 4, 1
// numbers.sort(function(a, b){return 0.5 - Math.random()}); // sorting randomly

console.log(numbers.find((number) => number === 1)); //1

console.log(numbers.findIndex((number) => number === 1)); //0 --> -1 if doesn't find

console.log(numbers.slice(0, 2)); //[ 1, 4 ] -> If the second argument is omitted, the slice() method slices out the rest of the array.
console.log(numbers); //[ 1, 4, 5 ] The original array will not be changed.

console.log(numbers.join(", ")); //"1, 4, 5"

console.log(numbers.reduce((acc, number) => (acc += number), 0)); //10
```

- Instead of `concat()` method to join two arrays, we will concat them by spread operator.

### destructuring of arrays

```js
let [, , thirdFruit] = fruits; // will a create a variable with the value of pears
let [firstFruit, ...otherFruits] = fruits; // with rest operator, we will have an array for the rest of fruits
let [fruit1, fruit2, fruit3, id, isRotten = false, isAvailable = true] = fruits; // by using default value, if the array does not have content for that variable (like isAvailable here), the default value will be used
// here, both isRotten and isAvailable is true
```

## Objects

- JavaScript objects are containers for named values called `properties` or `methods`.
- There are different ways to create new objects:
  - Define and create a single object, using an `object literal`.
  - Define an `object constructor`, and then create objects of the constructed type.
- Displaying a JavaScript object will output `[object Object]`. So we have to use `JSON.stringify` or iterate through keys and values.

### Object Literal

```js
const person = {
  firstName: "Ben",
  //In ES6 we can have space in the key value but we have to surround it with "":
  //"first name": 'Ben', in this way we only can use person["first name"] and not the dot notation.
  lastName: "Doe",
  //In ES6 if we want to construct an object from a variable outside and the key and value is identical, we can just bring that variable name:
  //lastName, as long as we have a lastName ='Doe'; outside and before object literal.
  age: 30,
  //In ES6 we can use a string literal as a key and the key will be the value of that string but we have to surround it by []:
  //[a] :30, as long as we have a string outside and before the object literal: a = 'age';
  hobbies: ["music", "movies"],
  address: {
    street: "Conversation st.",
    city: "Sydney",
    state: "NSW",
  },
  fullName() {
    // fullName: function() {
    return `${this.firstName} ${this.lastName}`;
  },
};

console.log(person);
console.log(person.firstName);
console.log(person["firstName"]); // Bracket notation: This way is useful for dynamic accessing.
console.log(person.hobbies[0]);
console.log(person.address.city);
console.log(person.fullName());
```

```js
let fullName = person.fullName; // because there is "this" in fullName method, when you use fullName function without binding an object to it, you will get undefined
console.log(fullName());
fullName = person.fullName.bind(person);
console.log(fullName());
```

```js
console.log(Object.values(person)); // returns an array
console.log(Object.keys(person)); // returns an array
```

```js
if ("address" in person)
  // to check if the property or method is available
  console.log("this property or method is available");
else console.log("this property or method is NOT available");
```

### "this" in js

- In JS, this depends on how a function is called (the left of the function).

```js
const person = {
  walk() {
    console.log(this);
  },
};

const walk = person.walk;
person.walk(); // this will be the person object
walk(); // this will be lost! -> this will be the global object which is window in browser but in strict mode, it is undefined

const walk2 = person.walk.bind(person); // every function is an object in JS (so we call bind method on it)
walk2(); // returns person object.
```

- Look at this example:

```js
const person = {
  walk() {
    setTimeout(function () {
      console.log(this);
    }, 1000);
  },
};

person.walk(); // window object! -> we have to use arrow function for not changing this context.
```

- In summary:
  - In JavaScript, the thing called this, is the object that "owns" the current code.
  - The value of this, when used in a function, is the object that "owns" the function.
  - When a function is called without an owner object, the value of this becomes the global object.
  - In a function definition, this refers to the "owner" of the function.
  - In a `method`, this refers to the `owner object`. So for methods we should always use regular functions.
  - `Alone`, this refers to the `global object`.
  - In a `function`, this refers to the `global object` (Even if the function is defined in a method like the example above).
  - In a `function`, in `strict mode`, this is `undefined`.
  - In an `event handler defined by function`, this refers to the `HTML element` that received the event.
  - In an `event handler defined by arrow function`, this refers to the `global object` (With arrow functions there are no binding of this).
  - Methods like call(), and apply() can refer this to any object.
    ```js
    var person1 = {
      fullName: function () {
        return this.firstName + " " + this.lastName;
      },
    };
    var person2 = {
      firstName: "John",
      lastName: "Doe",
    };
    person1.fullName.call(person2); // Will return "John Doe" -> calling person1.fullName with person2 as argument, this will refer to person2
    ```

### Destructuring of objects

```js
const {
  firstName: fName,
  lastName,
  address: { city },
} = person; //pulling some values from object. the first : is to give a different name to the extracted variable
console.log(fName);
console.log(city);
```

### Adding property

- This is not really recommended.

```js
person.email = "ben@gmail.com";
console.log(person);
```

- The keys of an object is always string. Even if you assign a number as a key:

```js
const person = {
  name: "Ben",
};

person[1] = 1;
console.log(person);
//{ '1': 1, name: 'Ben' }
```

- `Object.defineProperty()` is a new Object method in ES5.

```js
// Create an Object:
var person = {
  firstName: "John",
  lastName: "Doe",
  language: "NO",
};

// Change a Property:
Object.defineProperty(person, "language", {
  value: "EN",
  writable: true,
  enumerable: true, // if false, it hides the language property from enumeration
  configurable: true,
});

// Enumerate Properties
var txt = "";
for (var x in person) {
  txt += person[x] + "<br>";
}
document.getElementById("demo").innerHTML = txt;

// Or
// Change a Property:
Object.defineProperty(person, "language", {
  get: function () {
    return language;
  },
  set: function (value) {
    language = value.toUpperCase();
  },
});

// Change Language
person.language = "en";

// Display Language
document.getElementById("demo").innerHTML = person.language;
```

### Deleting property

```js
var person = { firstName: "John", lastName: "Doe", age: 50, eyeColor: "blue" };
delete person.age; // or delete person["age"];
```

### Accessors (getters and setters)

- A getter is better than a function because it can be used like a property.
- It can secure better data quality and things behind the scene.

```js
var person = {
  firstName: "John",
  lastName: "Doe",
  language: "",
  set lang(lang) {
    this.language = lang.toUpperCase() + " Language";
  },
  get lang() {
    return this.language.toLowerCase();
  },
};

person.lang = "en";

console.log(person.lang); // en language
```

### Array of objects

```js
const todos = [
  {
    id: 1,
    text: "Take out trash",
    isCompleted: true,
  },
  {
    id: 2,
    text: "Meeting with boss",
    isCompleted: true,
  },
  {
    id: 3,
    text: "Dentist appt",
    isCompleted: false,
  },
]; // it is not JSON. in JSON, the keys have "" and also strings have "" not ''

console.log(todos[1].text);
console.log(JSON.stringify(todos)); // this is how we send data to the server
```

## debugger statement

- With the debugger turned on, this code should stop executing before it executes the third line:

```js
var x = 15 * 5;
debugger;
document.getElementbyId("demo").innerHTML = x;
```

- Then, in the browser console, you can see the values of different variables.

## use strict

- The purpose of "use strict" is to indicate that the code should be executed in "strict mode".
- With strict mode, you can not, for example, use undeclared variables.
- As an example, in normal JavaScript, mistyping a variable name creates a new global variable. In strict mode, this will throw an error, making it impossible to accidentally create a global variable.
- In "Strict Mode", undeclared variables are not automatically global (otherwise if you assign something inside a function, it will be accessed outside and will be global).
- `this` keyword: If the object is not specified, functions in strict mode will return undefined and functions in normal mode will return the global object (window)

```js
"use strict";

x = 3.14; // This will cause an error -> Using a variable, without declaring it, is not allowed

function x(p1, p1) {} // This will cause an error -> Duplicating a parameter name is not allowed

var obj = {};
Object.defineProperty(obj, "x", { value: 0, writable: false });
obj.x = 3.14; // This will cause an error -> Writing to a read-only property is not allowed

var obj = {
  get x() {
    return 0;
  },
};
obj.x = 3.14; // This will cause an error -> Writing to a get-only property is not allowed
```

## Loops

- The `break;` statement "jumps out" of a loop.
- The `continue;` statement "jumps over" one iteration in the loop.

### for loop

```js
for (let i = 0; i <= 5; i++) {
  console.log(i);
}
```

### while loop

```js
let i = 0;
while (i <= 5) {
  console.log(i);
  i++;
}

do {
  text += "The number is " + i;
  i++;
} while (i < 10);
```

### for loop: for arrays

- Not recommended.

```js
for (let i = 0; i < todos.length; i++) {
  console.log(todos[i].text);
}
```

### for in: for arrays

- Goes through indexes.

```js
for (let i in todos) {
  console.log(todos[i].text);
}
```

### for of: for arrays

- Read the elements directly. recommended. You can also use for of loop with **strings** to read each character directly.

```js
for (let todo of todos) {
  console.log(todo.isCompleted);
}

var txt = "JavaScript";
var x;

for (x of txt) {
  document.write(x + "<br >");
}
```

### for in: for objects

- Goes through keys.

```js
for (let key in person) {
  console.log(key, person[key]);
}
```

### for of: for objects:

- Does not work! functions are not iterable!

## Higher order arrays methods

```js
const ages = [33, 12, 20, 16, 5, 54, 21, 44, 61];
```

### forEach: pretty much like for of

- Note that the function takes 3 arguments: The item value, The item index, and The array itself.

```js
todos.forEach(function (todo) {
  console.log(todo.id);
});
```

### map: returns an array

- Note that the function takes 3 arguments: The item value, The item index, and The array itself.

```js
const todoText = todos.map(function (todo) {
  return todo.text;
});
console.log(todoText);
```

#### map and arrow function together: professional way!

```js
console.log(todos.map((todo) => todo.text));
```

### filter: filters array members

- array elements that passes a test. 3 arguments again like the above.

```js
const todoCompleted = todos.filter(function (todo) {
  return todo.isCompleted === true;
});
console.log(todoCompleted);
```

#### filter and arrow function together: professional way!

```js
console.log(todos.filter((todo) => todo.isCompleted === true));
```

#### filter and map together

```js
const todoCompletedText = todos
  .filter(function (todo) {
    return todo.isCompleted === true;
  })
  .map(function (todo) {
    return todo.text;
  });
console.log(todoCompletedText);
```

### sort

```js
console.log(
  todos.sort(function (a, b) {
    if (a.id > b.id) {
      return 1;
    } else {
      return -1;
    }
  })
);
```

#### sort and arrow function together: professional way!

```js
console.log(ages.sort((a, b) => a - b));
```

### reduce

- runs a function on each array element to produce (reduce it to) a single value.
- The function will be called with these parameters: total (the initial value / previously returned value), value, index, array.
- We have also `reduceRight()` which works from right-to-left in the array.

```js
console.log(
  ages.reduce(function (total, age) {
    return total + age;
  }, 0)
);
```

#### reduce and arrow function

```js
console.log(ages.reduce((total, age) => total + age, 0));
```

## Sets, WeakSets, Maps, and WeakMaps:

### Sets

Unlike arrays, sets can only contain unique members.

```js
let mySet = new Set();

mySet.add("Hello");
mySet.add(1);
mySet.add(1);
let ob1 = { name: "john" };
let ob2 = { name: "john" };
mySet.add(ob1);
mySet.add(ob2); // 2 objects are not the same

console.log(mySet.size);
console.log(mySet.has(1));
console.log(mySet.delete(1));
console.log(mySet.has(1));
console.log(mySet.size);
mySet.clear(); // empties the set

let newSet = new Set([1, 2, 1, 1, 2, 2, 2, 2, 3, "a"]); // creating set from array
console.log(newSet.size);

newSet.forEach(function (value, key, callingSet) {
  console.log(key, value);
});

let chainSet = new Set().add("Hello").add("Doe");
console.log(chainSet.size);
```

### WeakSet

Is when all the elements are objects.

### Maps

Maps are similar to objects (pair of key and values) but unlike the objects the property does not have to be just strings and can be of any type.

```js
let myMap = new Map();

myMap.set("fname", "John");
myMap.set("age", 30);

console.log(myMap.get("fname"));

let ob1 = {};
let ob2 = {};

myMap.set(ob1, 10);
myMap.set(ob2, 10);

console.log(myMap.get(ob1));
myMap.delete("fname");

console.log(myMap.size);
console.log(myMap.has("fname"));

let newMap = new Map([
  ["fname", "John"],
  ["lname", "Doe"],
]); // passing members directly to the map

for (let key of newMap.keys()) {
  console.log(key);
}
for (let value of newMap.values()) {
  console.log(value);
}
for (let [key, value] of newMap.entries()) {
  console.log(key, value);
}
newMap.forEach(function (value, key, callingMap) {
  console.log(key, value);
});
```

### WeakMap

Is when all the keys and values are objects.

## Date

- There are four ways to create a date object:

```js
var d = new Date(); // current date and time

var d = new Date(2018, 11, 24, 10, 33, 30, 0); // year, month, day, hours, minutes, seconds, milliseconds -> January is 0. December is 11.
// you can omit the parameters up to month. But you cannot provide just year because it will be treated as milliseconds.

var d = new Date("October 13, 2014 11:13:00"); // from date strings
var d = new Date("January 25 2015"); // month can be written in full (January), or abbreviated (Jan)
var d = new Date("JANUARY, 25, 2015"); // Commas are ignored. Names are case insensitive
var d = new Date("Mar 25 2015");
var d = new Date("25 Mar 2015");
var d = new Date("03/25/2015"); // from date strings: mm/dd/yyyy
var d = new Date("2015-03-25"); // from date strings: yyyy-mm-dd
var d = new Date("2015-03"); // from date strings: yyyy-mm
var d = new Date("2015");
var d = new Date("2015-03-25T12:00:00Z"); // UTC time is defined with a capital letter Z. UTC is the same as GMT.
var d = new Date("2015-03-25T12:00:00-06:30"); // If you want to modify the time relative to UTC, remove the Z and add +HH:MM or -HH:MM instead
// You can convert all the date strings to milliseconds
var msec = Date.parse("March 21, 2012");

var d = new Date(100000000000); // 01 January 1970 plus 100 000 000 000 milliseconds is approximately 03 March 1973
```

- When you display a date object in HTML, it is automatically converted to a string, with the toString() method.

```js
var d = new Date();
console.log(d); // Wed Sep 30 2020 11:39:57 GMT+1000 (Australian Eastern Standard Time)
console.log(d.toString()); // Wed Sep 30 2020 11:39:57 GMT+1000 (Australian Eastern Standard Time)
console.log(d.toISOString()); // 2020-09-30T01:39:57.814Z
console.log(d.toDateString()); // Wed Sep 30 2020
console.log(d.toUTCString()); // Wed, 30 Sep 2020 01:39:57 GMT
```

### Date methods

```js
var d = new Date();

console.log(d.getTime()); // 1601433296044 milliseconds since EPOCH
console.log(Date.now()); // 1601433296044 milliseconds since EPOCH
console.log(d.getFullYear()); // 2020 as a number
console.log(d.getMonth()); // 8 means Sep
console.log(d.getDate()); // 30: day in the month

// getHours()	Get the hour (0-23)
// getMinutes()	Get the minute (0-59)
// getSeconds()	Get the second (0-59)
// getMilliseconds()	Get the millisecond (0-999)
// getDay()	Get the weekday as a number (0-6) -> the first day of the week (0) means "Sunday", even if some countries in the world consider the first day of the week to be "Monday".

// getUTCDate()	Same as getDate(), but returns the UTC date
// getUTCDay()	Same as getDay(), but returns the UTC day
// getUTCFullYear()	Same as getFullYear(), but returns the UTC year
// getUTCHours()	Same as getHours(), but returns the UTC hour
// getUTCMilliseconds()	Same as getMilliseconds(), but returns the UTC milliseconds
// getUTCMinutes()	Same as getMinutes(), but returns the UTC minutes
// getUTCMonth()	Same as getMonth(), but returns the UTC month
// getUTCSeconds()	Same as getSeconds(), but returns the UTC seconds

d.setFullYear(2020);
d.setFullYear(2020, 11, 3); // can optionally set month and day
d.setMonth(11);
d.setDate(15);
d.setDate(d.getDate() + 50); // to add days to a date. If adding days shifts the month or year, the changes are handled automatically by the Date object.
d.setHours(22); // 0-23
d.setMinutes(30); // 0-59
d.setSeconds(30); // 0-59
```

- Dates can easily be compared by `>` or `<`, ...

## Math

```js
console.log(Math.PI); //3.141592653589793
Math.E; // returns Euler's number
Math.round(4.7); // returns 5
Math.round(4.4); // returns 4
Math.pow(8, 2); // returns 64
console.log(8 ** 2); // returns 64
Math.sqrt(64); // returns 8
Math.abs(-4.7); // returns 4.7
Math.ceil(4.4); // returns 5
Math.floor(4.7); // returns 4
Math.sin((90 * Math.PI) / 180); // returns 1 (the sine of 90 degrees)
Math.min(0, 150, 30, 20, -8, -200); // returns -200
Math.max(0, 150, 30, 20, -8, -200); // returns 150
Math.random(); // returns a random number between 0 (inclusive), and 1 (exclusive)
Math.floor(Math.random() * 11); // returns a random integer from 0 to 10
Math.floor(Math.random() * 10) + 1; // returns a random integer from 1 to 10

function getRndInteger(min, max) {
  // both included
  return Math.floor(Math.random() * (max - min + 1)) + min;
}
```

## Conditionals

- An assignment always returns the value of the assignment.So be aware in using in conditionals instead of comparison.

### if-else

```js
const xx = "12";
if (xx == 10) {
  console.log("xx is 10");
} else if (xx > 10) {
  console.log("xx is greater than 10");
} else {
  console.log("xx is less than 10");
}

if (xx !== 12) {
  console.log("xx is not exactly 12");
}

yy = 10;
if (xx > 10 && yy === 10) {
  console.log("xx > 10 and yy === 10");
}

if (xx > 10 || yy === 10) {
  console.log("xx > 10 or yy === 10");
}
```

### Ternary operator

```js
const a = 10;
let color = a < 10 ? "red" : "blue";
console.log(color);

color = "green";
```

### switch

- switch statements use strict comparison (===).

```js
switch (color) {
  case "blue":
    console.log("the color is blue");
    break;
  case "red":
    console.log("the color is red");
    break;
  default:
    console.log("color is not red or blue");
}
```

- If you omit the break statement, the next case will be executed even if the evaluation does not match the case.
- The default keyword specifies the code to run if there is no case match.
- If default is not the last case in the switch block, remember to end the default case with a break.
- If no default label is found, the program continues to the statement(s) after the switch.
- If multiple cases matches a case value, the first case is selected.
- Sometimes you will want different switch cases to use the same code:

```js
case 4:
case 5:
    text = "Soon it is Weekend";
    break;
```

## Functions

- Functions are object in JS.
- If a function is called with a missing argument, the value of the missing argument is set to `undefined`. So we can use default parameters.
- If a function changes an object property, it changes the original value.

```js
//Function declaration
function addNums(x = 0, y = 0) {
  console.log(arguments.length); //arguments object (an array) is only accessible inside the function
  return x + y;
}

console.log(addNums(5, 4));
// 2 9
console.log(addNums()); // using default values
// 0 0
console.log(addNums(5)); // using default values for the second argument
// 1 5
console.log(addNums(undefined, 4)); // using default values for the first argument
// 2 4
```

### Function expression

- Functions defined using an expression are not hoisted.
- You cannot self-invoke a function declaration but with function expression you can, if the expression is followed by ().

```js
var addNums = function (x = 0, y = 0) {
  // ...
};

// anonymous self-invoking function
(function () {
  var x = "Hello!!"; // I will invoke myself
})();
```

### Functions are object

```js
function myFunction(a, b) {
  console.log(arguments.length); // the argument object contains an array of the arguments used when the function was called
}

myFunction(1, 2); // 2
myFunction(1); //1
console.log(myFunction.toString()); // function myFunction(a, b) {
```

### Functions are methods

- In JavaScript all functions are object methods.
- If a function is not a method of a JavaScript object, it is a function of the global object
- With `call()`, an object can use a method belonging to another object.

```js
person1.fullName.call(person2);

// with arguments
person1.fullName.call(person2, "Oslo", "Norway");
```

- The `apply()` method, is similar to `call()` method but it takes arguments as an array (instead of taking them separately).

```js
Math.max.apply(null, [1, 2, 3]); // 3
```

### Functions closures

- It makes it possible for a function to have "private" variables.
- The `counter` is protected by the scope of the anonymous function, and can only be changed using the add function.

```js
var add = (function () {
  var counter = 0;
  return function () {
    counter += 1;
    return counter;
  };
})(); // The self-invoking function only runs once and initializes the counter to zero

add();
add();
console.log(add()); // 3
```

### Spread Operator

- Allows an iterable such as an array expression or object or string to be expanded in places where zero or more arguments (for function calls) or elements (for array literals) are expected. The older way in ES5: `console.log(addNums.apply(null, args));`

```js
args = [4, 5];
console.log(addNums(...args));

const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

const arr3 = [...arr1, 0, ...arr2];

const person = { name: "Ben" };
const clone = { ...person, age: 35 };
```

### arrow functions

- The first advantage of arrow functions that they are cleaner but most importantly, they do not bind this.

```js
const addNumsArrow1 = (x = 0, y = 0) => {
  console.log(x + y);
};
addNumsArrow1(1, 1);

const addNumsArrow2 = (x = 0, y = 0) => console.log(x + y); // Better practice for when we have only one statement in the function
addNumsArrow2(2, 2);

const addNumsArrow3 = (x = 0, y = 0) => {
  return x + y;
};
console.log(addNumsArrow3(3, 3));

const addNumsArrow4 = (x = 0, y = 0) => x + y; // If there is just one return, you better get rid of return.
console.log(addNumsArrow4(4, 4));

const addNumsArrow5 = (x) => x + x; // If there is just one parameter, get rid of the (). this is great for forEach (because we just have one parameter). It does not work if you want to have default value.
console.log(addNumsArrow5(5));

const addNumsArrow6 = (x = 0) => x + x;
console.log(addNumsArrow6());
```

```js
todos.forEach((todo) => console.log(todo.id));
```

## Generators and Iterators

```js
function \*g1(){
console.log('Hello');
yield 'Yield 1 Ran..';
console.log('World');
yield 'Yield 2 Ran...';
return 'Returned..';
}

var g = g1();

console.log(g.next().value);
console.log(g.next().value);
console.log(g.next().value);

for(let val of g){
console.log(val);
}
```

### another example of generators

```js
function \*createGenerator(){
yield 1;
console.log("After 1st yield");
yield 2;
}

let myGen = createGenerator();

console.log(myGen.next()); //{value: 1, done: false}
console.log(myGen.next()); //{value: 2, done: false}
console.log(myGen.next()); //{value: undefined, done: true}
```

## Regular Expressions

- A regular expression is a sequence of characters that forms a search pattern.
- The search pattern can be used for text search and text replace operations.
- In JavaScript, regular expressions are often used with the two string methods: `search()` and `replace()`.

```js
var patt = /w3schools/i; // -> /pattern/modifiers -> typeof patt is object

var str = "Visit W3Schools";
var n = str.search(/w3schools/i); //6

var str = "Visit Microsoft!";
var res = str.replace(/microsoft/i, "W3Schools"); //Visit W3Schools!
```

### Modifiers

- i Perform case-insensitive matching.
- g Perform a global match (find all matches rather than stopping after the first match).
- m Perform multiline matching.

### Regular Expression Patterns

```js
//Brackets
var str = "Is this all there is?";
var patt1 = /[h]/g; // Find any of the characters between the brackets -> another example[a-z]
var result = str.match(patt1); //  ["h", "h"]

var str = "123456789";
var patt1 = /[1-4]/g; // Find any of the digits between the brackets
var result = str.match(patt1); // ["1", "2", "3", "4"]

var str = "re, green, red, green, gren, gr, blue, yellow";
var patt1 = /(red|green)/g; // Find any of the alternatives separated with |
var result = str.match(patt1); // ["green", "red", "green"]

//Metacharacters
var str = "Give 100%!";
var patt1 = /\d/g; // Find a digit
var result = str.match(patt1); // ["1", "0", "0"]

var str = "Is this all there is?";
var patt1 = /\s/g; // Find a whitespace character
var result = str.match(patt1); // [" ", " ", " ", " "]

var str = "HELLO, LOOK AT YOU!";
var patt1 = /\bLO/; // Find a match at the beginning of a word like this: \bWORD, or at the end of a word like this: WORD\b
var result = str.search(patt1); // 7

var str = "Visit W3Schools. Hello World!";
var patt1 = /\u0057/g; // Find the Unicode character specified by the hexadecimal number xxxx
var result = str.match(patt1); // ["W", "W"]

//Quantifiers
var str = "Hellooo World! Hello W3Schools!";
var patt1 = /o+/g; // Matches any string that contains at least one o
var result = str.match(patt1); // ["ooo", "o", "o", "oo"]

var str = "Hellooo World! Hello W3Schools!";
var patt1 = /lo*/g; // global search for an "l", followed by zero or more "o" characters
var result = str.match(patt1); // ["l", "looo", "l", "l", "lo", "l"]

var str = "1, 100 or 1000?";
var patt1 = /10?/g; // global search for a "1", followed by zero or one "0" characters
var result = str.match(patt1); // ["1", "10", "10"]
```

### RegExp methods

```js
// test method searches a string for a pattern, and returns true or false, depending on the result.
/e/.test("The best things in life are free!"); // true

// exec method searches a string for a specified pattern, and returns the found text as an object.
/e/.exec("The best things in life are free!"); // [0: "e", index: 2, input: "The best things in life are free!", groups: undefined]
```

## Error

```js
try {
  throw new Error("Hi");
} catch (err) {
  console.log(err.name); // Error. If you throw different type of error like EvalError, RangeError, ReferenceError, SyntaxError, TypeError, URIError you will get different name.
  console.log(err.message); // Hi
} finally {
  // Block of code to be executed regardless of the try / catch result
  console.log("bye");
}
```

## OOP

### Using constructor function

- If a function invocation is preceded with the `new` keyword, it is a constructor invocation. A constructor invocation creates a new object. The new object inherits the properties and methods from its constructor. The value of `this` will be the new object created when the function is invoked.

```js
function Person(first, last, age) {
  this.firstName = first;
  this.lastName = last;
  this.age = age;
  this.fullName = function () {
    return this.firstName + " " + this.lastName;
  };
}

var myFather = new Person("John", "Doe", 50);
```

- You can add a method or property to an object (myFather) but adding properties or methods to an object constructor (Person) must be done inside the constructor function.

#### Prototype inheritance

```js
function Person(first, last, age) {
  this.name = {
    first,
    last,
  };
  this.age = age;
}

Person.prototype.fullName = function () {
  // The methods are all defined on the constructor's prototype. If you define it in the constructor's function, you can't change it easily.
  return this.name.first + " " + this.name.last;
};

console.log(new Person("a", "b", 1).fullName()); // a b

function Teacher(first, last, age, subject) {
  Person.call(this, first, last, age); // This function basically allows you to call a function defined somewhere else, but in the current context
  this.subject = subject;
}

//Teacher.prototype = Person.prototype;
Teacher.prototype = new Person(); // inherit all the methods available on Person.prototype.
Teacher.prototype.constructor = Teacher; // except we should change the constructor

Teacher.prototype.fullName = function () {
  return this.name.first + " " + this.name.last + " (" + this.subject + ")";
};

console.log(new Teacher("aa", "bb", 1, "Math").fullName()); // aa bb (Math)
```

### Using class keyword

- In JS, classes are objects! Because, classes are actually functions and functions are objects in JS.
- There is no actual class in JS. There is a constructor function which initializes the properties (numbers, strings, booleans, arrays, ...). This function has also a `prototype` property (`__proto__` in chrome) in which all the methods reside in. We can change the prototype or add methods to it even after we defined our class and it changes the behavior of all the instances as well! So it is a syntactic sugar over prototype inheritance above.
- Although classes are actually functions in ES6 but they cannot be instantiated before declaration (no hoisting).

```js
class Employee {
  constructor(firstName, lastName, dob) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.dob = new Date(dob);
    this.defaultRate = 20;
  }

  getFirstName() {
    return this.firstName;
  }

  getDefaultRate() {
    return this.defaultRate;
  }

  getFullName() {
    return `${this.firstName} ${this.lastName}`;
  }

  getBirthYear() {
    return this.dob.getFullYear();
  }

  setFirstName(newFirstName) {
    this.firstName = newFirstName;
    this.revised = true; // adding a property
    return this; // This way, we can chain methods together
  }

  increaseRate() {
    this.defaultRate++;
    return this; // This way, we can chain methods together
  }

  static topEmployee() {
    // static method
    return "John";
  }
}
```

### Instantiate object

```js
const employee1 = new Employee("Ben", "Abe", "1-28-1985");
const employee2 = new Employee("Nas", "Tavak", "5-5-1992");

console.log(employee1);
console.log(employee1.getFirstName()); //Ben
console.log(employee1.firstName); //Ben
console.log(employee1.getDefaultRate()); //20
console.log(employee1.defaultRate); //undefined
console.log(employee1.getFullName()); //Ben Abe
console.log(employee1.getBirthYear()); //1985

employee1.setFirstName("John").increaseRate();
console.log(employee1);

console.log(Employee.topEmployee()); // to use static methods, we should use class name not instance name
```

### Inheritance

```js
class Developer extends Employee {
  constructor(firstName, lastName, dob, programmingLanguage) {
    //if the inherited class has the same properties, we can remove this constructor function. Of course in this case, the inherited class has extra methods to be different from the super class.
    super(firstName, lastName, dob);
    this._programmingLanguage = programmingLanguage;
  }

  get programmingLanguage() {
    return this._programmingLanguage;
  }

  set programmingLanguage(newLang) {
    this._programmingLanguage = newLang;
  }

  increaseRate() {
    // if we have duplicated methods with the same name as parent, the child method will run
    super.increaseRate(); // increase 1
    this.defaultRate += 2; // increase 2
    return this;
  }
}
```

#### Instantiate Developer Object

```js
const developer1 = new Developer("John", "Abe", "1-1-1985", "JS");
developer1.increaseRate();
console.log(developer1); // defaultRate is 23
console.log(developer1.getFullName());

const dev1 = new Developer("a", "b", "2000", "JS");
console.log(dev1.programmingLanguage); //JS
dev1.programmingLanguage = "JavaScript";
console.log(dev1.programmingLanguage); //JavaScript
```

## Modularization: stroring each class in each file.js

```js
export default class Developer{} ---> import Developer from './developer'; // for default export, you dont need the {} and also the name can be different (it is the same here)
export function coding{} ---> import {coding} from './developer'; // this is the named export
export let a = 'hi'; ---> import {a as message} from './moduleA.js'; // using alias. of course you can export variables as well.
```

Import statements are hoisted (no matter where they are, they will execute at the top and then rest of the code.
You cannot reassign imported variables because they are read-only but you can change the properties of an imported object.

## Async JS

### Promise Fundamentals

Promise object used for deferred computation.

```js
// Immediately Resolved
const myPromise = Promise.resolve("Foo"); // resolve is what the promise returns if it runs by then. The output of resolve is the input of then.
myPromise.then((res) => console.log(res));

const hisPromise = new Promise((resolve, reject) => {
  // reject is what promise returns if it runs by catch. The output of reject is the input of catch.
  setTimeout(() => resolve(4), 2000);
});
hisPromise.then((res) => console.log(res + 3));

const promise1 = Promise.resolve("Hello World");
const promise2 = 10;
const promise3 = new Promise((resolve, reject) =>
  setTimeout(resolve("Goodbye"), 2000)
);

Promise.all([promise1, promise2, promise3]).then(
  (results) => console.log(results) // [ 'Hello World', 10, 'Goodbye' ]
); //It takes the longest promise to show the values.

Promise.race([promise1, promise2, promise3]).then(
  (result) => console.log(result) // 'Hello World'
); //It returns the value of the first promise that is fulfilled.
```

```js
let posts = [
  { title: "Post One", body: "This is post one" },
  { title: "Post Two", body: "This is post two" },
];

const getPosts = () => setTimeout(() => console.log(posts), 1000);
```

### Callbacks

```js
function createPost(post, callback) {
  // callback function is a parameter
  setTimeout(() => {
    posts.push(post);
    callback(); // executing callback function
  }, 1500);
}

createPost({ title: "Post Three", body: "This is post three" }, getPosts); // feeding the callback
```

### Promises

- You can re-write each callback to return a promise.

```js
function createPostPromise(post) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      posts.push(post);

      const error = false;

      if (!error) {
        resolve();
      } else {
        reject("Error: Something went wrong");
      }
    }, 3000);
  });
}

createPostPromise({ title: "Post Three", body: "This is post three" })
  .then(getPosts)
  .catch((err) => console.log(err));
```

### Async / Await

- For the `catch` part, we have to wrap the async operation in a `try-catch` block.

```js
const exec = async () => {
  await createPostPromise({ title: "Post Three", body: "This is post three" }); // first this line is run then next line will be run
  getPosts();
};
exec();
```

## BOM (browser) & DOM

![](/md/144.jpg)

- We have this window object in the browser. For example `alert("hi")` is actually `window.alert("hi")`.
- This object has `localStorage` object which has some methods such as `getItem` and `setItem`:

  ```js
  // Storing data:
  myObj = { name: "John", age: 31, city: "New York" };
  myJSON = JSON.stringify(myObj);
  localStorage.setItem("testJSON", myJSON);

  // Retrieving data:
  text = localStorage.getItem("testJSON");
  obj = JSON.parse(text);
  document.getElementById("demo").innerHTML = obj.name;
  ```

- The `sessionStorage` object is almost identical to the localStorage object (it has setItem and getItem methods). The difference is that the sessionStorage object stores data for one session. The data is deleted when the browser is closed.
- This object has fetch method.
- This object has the document object as well.

```js
// Single Element Selectors
console.log(document.getElementById("my-form")); // <form id="my-form">...</form>
console.log(document.querySelector(".container")); // it is newer and we can use css selectors in it. It replaces the jQuery.

// Multiple Element Selectors
console.log(document.getElementsByTagName("li"));
console.log(document.getElementsByClassName("item")); // it returns HTMLCollection. You have to convert it to array first to run array methods.
console.log(document.querySelectorAll(".item")); // it returns a NodeList which is similar to arrays and you can run array methods on it.
```

- `document.body` property returns the `<body>` element and `document.documentElement` property returns `<html>` element.

- Both an `HTMLCollection` object and a `NodeList` object is an array-like list (collection) of objects.
- Both have a `length` property defining the number of items in the list (collection).
- Both provide an index (0, 1, 2, 3, 4, ...) to access each item like an array.
- `HTMLCollection` items can be accessed by their `name`, `id`, or `index number`.
- `NodeList` items can `only` be accessed by their `index number`.
- Only the `NodeList` object can contain `attribute nodes` and `text nodes`.
- You can loop through both and refer to its nodes like an array.
- However, you cannot use Array Methods, like valueOf(), push(), pop(), or join() on both.

- For traversing and manipulating the DOM:

```js
const ul = document.querySelector(".items");
ul.remove(); // the element will be removed from the dom
ul.lastElementChild.remove();
ul.firstElementChild.textContent = "Hello";
ul.children[1].innerText = "Brad"; // this is the second child. The textContent and innerText have subtle differences.
ul.lastElementChild.innerHTML = "<h1>Hello</h1>";

// Changing the style
const btn = document.querySelector(".btn");
btn.style.background = "red";

// Change attribute
document.getElementById("myImage").src = "landscape.jpg";
```

- Adding, removing, replacing elements:

```html
<div id="div1">
  <p id="p1">This is a paragraph.</p>
  <p id="p2">This is another paragraph.</p>
</div>

<script>
  var para = document.createElement("p");
  var node = document.createTextNode("This is new.");
  para.appendChild(node);

  var element = document.getElementById("div1");
  element.appendChild(para);
</script>

<div id="div1">
  <p id="p1">This is a paragraph.</p>
  <p id="p2">This is another paragraph.</p>
</div>

<script>
  var para = document.createElement("p");
  var node = document.createTextNode("This is new.");
  para.appendChild(node);

  var element = document.getElementById("div1");
  var child = document.getElementById("p1");
  element.insertBefore(para, child);
</script>

<div>
  <p id="p1">This is a paragraph.</p>
  <p id="p2">This is another paragraph.</p>
</div>

<script>
  var elmnt = document.getElementById("p1");
  elmnt.remove();
</script>

<!-- Remove from parent if remove method above not supported -->
<div id="div1">
  <p id="p1">This is a paragraph.</p>
  <p id="p2">This is another paragraph.</p>
</div>

<script>
  var parent = document.getElementById("div1");
  var child = document.getElementById("p1");
  parent.removeChild(child);
</script>

<div id="div1">
  <p id="p1">This is a paragraph.</p>
  <p id="p2">This is another paragraph.</p>
</div>

<script>
  var para = document.createElement("p");
  var node = document.createTextNode("This is new.");
  para.appendChild(node);

  var parent = document.getElementById("div1");
  var child = document.getElementById("p1");
  parent.replaceChild(para, child);
</script>
```

- Events: An HTML event can be something the browser does, or something a user does.
- The `addEventListener()` method allows you to add many events to the same element, without overwriting existing events.
- In addition to HTML elements, you can add event listeners to window object to listen for "resize" event for example.
- The `removeEventListener()` method removes event handlers that have been attached with the addEventListener() method.

```js
// Mouse Event
btn.addEventListener("click", (e) => {
  // we have other events such as mouseover, mouseout
  e.preventDefault(); // because that button is a submit button

  console.log(e.target.className); // e.target is the element that fired the event
  document.getElementById("my-form").style.background = "#ccc"; // This way you should camelCase for css properties. To use dahses:
  document.getElementById("my-form").style["background"] = "#ccc";
  document.querySelector("body").classList.add("bg-dark"); // adding class to an element
  ul.lastElementChild.innerHTML = "<h1>Changed</h1>";
});

// Keyboard Event
const nameInput = document.querySelector("#name");
nameInput.addEventListener("input", (e) => {
  document.querySelector(".container").append(nameInput.value);
});
```

- The `onload` and `onunload` events are triggered when the user enters or leaves the page.
- The `onchange` event is often used in combination with validation of input fields.
- The `onmouseover` and `onmouseout` events can be used to trigger a function when the user mouses over, or out of, an HTML element.
- The `onmousedown`, `onmouseup`, and `onclick` events are all parts of a mouse-click. First when a mouse-button is clicked, the onmousedown event is triggered, then, when the mouse-button is released, the onmouseup event is triggered, finally, when the mouse-click is completed, the onclick event is triggered.

```js
<body onload="checkCookies()">

<input type="text" id="fname" onchange="upperCase()">

<div onmouseover="mOver(this)" onmouseout="mOut(this)">Mouse Over Me</div>
```

- Forms:

```js
// Put DOM elements into variables
const myForm = document.querySelector("#my-form");
const nameInput = document.querySelector("#name");
const emailInput = document.querySelector("#email");
const msg = document.querySelector(".msg");
const userList = document.querySelector("#users");

// Listen for form submit
myForm.addEventListener("submit", onSubmit);

function onSubmit(e) {
  e.preventDefault();

  if (nameInput.value === "" || emailInput.value === "") {
    // alert('Please enter all fields');
    msg.classList.add("error");
    msg.innerHTML = "Please enter all fields";

    // Remove error after 3 seconds
    setTimeout(() => msg.remove(), 3000);
  } else {
    // Create new list item with user
    const li = document.createElement("li");

    // Add text node with input values
    li.appendChild(
      document.createTextNode(`${nameInput.value}: ${emailInput.value}`)
    );

    // Add HTML
    // li.innerHTML = `<strong>${nameInput.value}</strong>e: ${emailInput.value}`;

    // Append to ul
    userList.appendChild(li);

    // Clear fields
    nameInput.value = "";
    emailInput.value = "";
  }
}
```

### Event bubbling

- When for example you click on an element, JS creates an event object and see if that element has an `onclick` event handler and if it does it will run it. Then, JS takes that event object and gives it to the parent of that element and so on...
- We can stop this bubbling by `e.stopPropagation();`.
- If we want to close a dropdown, based on clicking outside the dropdown, we should check:

```js
document.body.addEventListener("click", (e) => {
  if (!dropdownWrapper.contains(e.target)) {
    dropdown.style.display = "none";
  }
});
```

- With the addEventListener() method you can specify the propagation type by using the "useCapture" parameter.

```js
addEventListener(event, function, useCapture);
```

- The `default` value is `false`, which will use the `bubbling propagation`, when the value is set to `true`, the event uses the `capturing propagation` (from outer ro inner).

### BOM

```js
console.log(window.innerHeight); // 722
console.log(window.innerWidth); // 730

console.log(window.screen.height); // 1536 user's screen resolution in pixel
console.log(window.screen.width); // 864 user's screen resolution in pixel
console.log(screen.availWidth); // 1536: the width of the visitor's screen, in pixels, minus interface features like the Windows Taskbar
console.log(screen.availHeight); // 824 because of taskbar
console.log(screen.colorDepth); //  24. returns the number of bits used to display one color
console.log(screen.pixelDepth); //  24

window.open(); // open a new window
window.close(); // close the current window
window.moveTo(); // move the current window
window.resizeTo(); // resize the current window

console.log(window.location.href); // https://www.w3schools.com/js/js_window_location.asp returns the href (URL) of the current page
console.log(window.location.hostname); // www.w3schools.com returns the domain name of the web host
console.log(window.location.pathname); // /js/js_window_location.asp returns the path and filename of the current page
console.log(window.location.protocol); // https: returns the web protocol used (http: or https:)
console.log(window.location.port); // 443 the number of the internet host port
window.location.assign("https://www.w3schools.com"); // loads a new document

history.back(); // same as clicking back in the browser
history.forward(); // same as clicking forward in the browser
history.go(-2); // go Back 2 Pages

console.log(navigator.appName); // Netscape
console.log(navigator.appCodeName); // Mozilla
console.log(navigator.appVersion); // 5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Safari/537.36
console.log(navigator.platform); // Win32
console.log(navigator.cookieEnabled); // true
console.log(navigator.product); // Gecko
console.log(navigator.userAgent); // Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Safari/537.36
console.log(navigator.language); // en-US
console.log(navigator.onLine); //true
console.log(navigator.javaEnabled()); // false

navigator.geolocation.getCurrentPosition(
  (pos) => {
    console.log(`Lat: ${pos.coords.latitude}, Long: ${pos.coords.longitude}`);
  },
  (err) => {
    console.log(err.code); // 1 is user denied
  }
);

// Only OK
window.alert("sometext");

// Ok and Cancel
if (window.confirm("Press a button!")) {
  txt = "You pressed OK!";
} else {
  txt = "You pressed Cancel!";
}

// Get sth from user
var person = prompt("Please enter your name", "Harry Potter");

if (person == null || person == "") {
  txt = "User cancelled the prompt.";
} else {
  txt = "Hello " + person + "! How are you today?";
}
```

#### Timing

```html
<button onclick="myVar = setTimeout(myFunction, 3000)">Try it</button>

<button onclick="clearTimeout(myVar)">Stop it</button>

<script>
  function myFunction() {
    alert("Hello");
  }
</script>

<p id="demo"></p>

<button onclick="clearInterval(myVar)">Stop time</button>

<script>
  var myVar = setInterval(myTimer, 1000);
  function myTimer() {
    var d = new Date();
    document.getElementById("demo").innerHTML = d.toLocaleTimeString();
  }
</script>
```

## Cookies

- Cookies are data, stored in small text files, on your computer.
- When a web server has sent a web page to a browser, the connection is shut down, and the server forgets everything about the user.
- Cookies were invented to solve the problem "how to remember information about the user":
- When a user visits a web page, his/her name can be stored in a cookie.
- Next time the user visits the page, the cookie "remembers" his/her name.
- Cookies are saved in name-value pairs.
- When a browser requests a web page from a server, cookies belonging to the page are added to the request. This way the server gets the necessary data to "remember" information about users.

### Create a cookie

- If you set a new cookie, older cookies are not overwritten. The new cookie is added to `document.cookie`.
- You can add an expiry date (in UTC time). By default, the cookie is deleted when the browser is closed:
- With a path parameter, you can tell the browser what path the cookie belongs to. By default, the cookie belongs to the current page.

```js
document.cookie =
  "username=John Doe; expires=Thu, 18 Dec 2013 12:00:00 UTC; path=/";
```

### Read a cookie

- `document.cookie` will return all cookies in one string much like: `cookie1=value; cookie2=value; cookie3=value`;

### Change a cookie

- It is just like creating a cookie. You just have to overwrite it.

### Delete a cookie

- Just set the expires parameter to a passed date:

```js
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;";
```

### Example

```html
<!DOCTYPE html>
<html>
  <head>
    <script>
      function setCookie(cname, cvalue, exdays) {
        var d = new Date();
        d.setTime(d.getTime() + exdays * 24 * 60 * 60 * 1000);
        var expires = "expires=" + d.toGMTString();
        document.cookie = cname + "=" + cvalue + ";" + expires + ";path=/";
      }

      function getCookie(cname) {
        var name = cname + "=";
        var decodedCookie = decodeURIComponent(document.cookie);
        var ca = decodedCookie.split(";");
        for (var i = 0; i < ca.length; i++) {
          var c = ca[i];
          while (c.charAt(0) == " ") {
            c = c.substring(1);
          }
          if (c.indexOf(name) == 0) {
            return c.substring(name.length, c.length);
          }
        }
        return "";
      }

      function checkCookie() {
        var user = getCookie("username");
        if (user != "") {
          alert("Welcome again " + user);
        } else {
          user = prompt("Please enter your name:", "");
          if (user != "" && user != null) {
            setCookie("username", user, 30);
          }
        }
      }
    </script>
  </head>

  <body onload="checkCookie()"></body>
</html>
```

## JSON

- In JSON, values must be one of the following data types:

  - a string
  - a number
  - an object (JSON object)
  - an array
  - a boolean
  - null

- JSON values cannot be one of the following data types:

  - a function
  - a date

- If you want to parse date or functions, you should use `reviver` for date (second argument to JSON.parse) or `eval` for function:

```js
var text =
  '{ "name":"John", "birth":"1986-12-14", "city":"New York", "age":"function () {return 30;}"}';
var obj = JSON.parse(text, function (key, value) {
  if (key == "birth") {
    return new Date(value);
  } else {
    return value;
  }
});
obj.age = eval("(" + obj.age + ")");

document.getElementById("demo").innerHTML = obj.name + ", " + obj.birth;
```

- The `JSON.stringify()` function will remove any functions from a JavaScript object, both the key and the value.
- This can be omitted if you convert your functions into strings (`obj.age = obj.age.toString();`) before running the `JSON.stringify()` function.

## Webpack

- Is a module bundler for JS. So we can split JS into multiple files and have webpack bundled them into one file.
- In `webpack.config.js`, we specify the entry to our application and the output location (where generated file to be saved).

## Functional Programming

- We have some functions that will be called in sequence (composition of functions) but they do not mutate the data.
- In JS, functions are first-class citizens just like variables.
- If a function, receives a function or returns it or both, it is a higher order function.
- Because multiple function calls can lead to deeply-nested structure, we can use `import {compose, pipe} from "lodash/fp";`.

```js
const transform = pipe(trim, toLowerCase, wrapInDiv); // from left to right. compose is from right to left.
const result = transform("  hi ");
```

- `Currying` is a technique so that a function that receives n parameter, will receive 1 parameter:

```js
function add(a) {
  return function (b) {
    return a + b;
  };
}

const add = (a) => (b) => a + b;

add(1)(2); // instead of separating arguments with , we use () and =>
```

- `Pure functions` are the ones that with `same args` => `same results`.
  - So no random,
  - no current date/time,
  - no reach to outside of the function, global state, database, file, API calls ...
  - no mutations of parameters
  - no side effects
- Reducers in redux must be pure.
- Pure functions:
  - are easier to test -> because no setup is needed (no reach to outside).
  - can be run concurrently -> no dependency on any state.
  - are cacheable -> same input always same output.
- In functional programming languages, we cannot mutate data but in JS we can mutate objects and arrays so JS is not a functional programming language (it is a multi-paradigm).
- With immutability, React can quickly detect that the state has been changed or not and it don't need to compare each property.
- When cloning object with spread operator, be aware of shallow copying. So if you want a deep copy, you should de-structure the nested objects as well. Because otherwise if you mutate a nested object in the cloned, it will change the original as well.
- Not mutating objects (specially when they are deeply nested) is a pain. We should use `immer` library.

```js
import { produce } from "immer";

const book = { title: "Man and the sea" };

function publish(book) {
  return produce(book, (draftBook) => {
    draftBook.isPublished = true; // we can mutate as much as we want and the original book won't be affected
  });
}

const updated = publish(book); // a new object will be returned
```
