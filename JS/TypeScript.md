# Basics

- Type system helps catch bugs during development.
- TS does not do any performance optimization.
- After compilation, it is just good old JS. So TS will not execute.

## Installation

```dos
npm i -g typescript ts-node
```

- To check if the TS compiler has been installed correctly:

```dos
tsc --help
```

## First App

- Create a `package.json` by `npm init -y`.
- You can install packages by `npm i whatever` and import it in a `.ts` file using `import whatever from 'whatever';` syntax.
- Create `index.ts` file with a single `console.log("hi");`.
- Run `tsc index.ts` to compile and create `index.js` and then run `node index.js` to actually run it.
- Alternatively, we can use `ts-node index.ts`.

## Configuring TS Compiler

- Run `tsc --init` to create a `tsconfig.json` file in the root of the project.
- Now create two folders `src` and `build` in the root of the project.
- Uncomment `outDir` and `rootDir` in the tsconfig.json and give them the address:

```json
{
  //...
  "outDir": "./build" /* Redirect output structure to the directory. */,
  "rootDir": "./src"
  //...
}
```

- Now if we run `tsc` in the terminal in the root of the project, it compiles `index.ts` from `src` folder and spits out `index.js` to the `build` folder.
- If we use `-w` flag like `tsc -w`, it will watch for any changes and recompile every time that we have a change in `index.ts`.
- It should be noted that by just generating `tsconfig.file`, TS will enter to `strict mode` (by default there is a line in the file: `"strict": true,`). In this mode, TS detects that optional fields can be `undefined | number` for example.
- `!` is a way to tell the `Typescript` compiler "this expression cannot be null or undefined here, so don't complain about the possibility of it being null or undefined."

### Using nodemon and concurrently to run the compiled files

- Run:

```dos
npm init -y
npm i concurrently nodemon --save-dev
```

- Add the following in the `package.json` file:

```json
{
  //...
  "scripts": {
    "start:build": "tsc -w",
    "start:run": "nodemon build/index.js",
    "start": "concurrently npm:start:*"
  }
  //...
}
```

- Now start the project with `npm start`.

## Types

- Every value has a type in typescript. Of course a value can have multiple types (of type of its class and its parent's class and the interface that it implements).
- Type is actually the name of the class that has some fields and methods:

  - Primitive types
    - `any`
    - `number`
    - `string`
    - `boolean`
    - `null`
    - `undefined`
    - `void`
    - `never`, a function return type can be `never` when it always throw and error and never returns anything or completes.
    - `symbol`
    - `number | boolean`
    - example of annotating a function (this is used more often):
    ```ts
    const add1 = (a: number, b: number): number => a + b;
    ```
    - A fixed string can also be used as a type: `let name:'ben';`, in this case the only value that can be assigned to variable `name` is string of `ben`.
  - Reference types (Object types)

    - `Array<number>` (equivalently: `number[]`) or `Array<Object>` (equivalently: `Object[]`) or `Array<number | string>` (equivalently: `(string | number)[]`)
    - `{id: number; name: string}` or `{ [key: string]: number[]}`. The latter means that the keys are string and values of this object literal should be array of numbers.
    - `Interface`,
      - Interfaces are used to define the members of objects (fields and methods).
      - Unlike `C#`, we can have nested levels in defining interfaces and classes here.
      - Unlike `C#`, an interface can have fields (in C# it can be props or methods).
      - There are also some built-in interfaces: `Object`, `Date`, `Function`, and etc.

    ```ts
    interface Vehicle {
      name: string;
      year: Date;
      isBroken: boolean;
      maker: {
        name: string;
      };
      drive(): void;
    }

    interface Drivable {
      drive(): void;
    }
    ```

    - `(a: number, b: number) => number` or `() => void`, examples of function annotation:

    ```ts
    const add2: (a: number, b: number) => number = (a: number, b: number) =>
      a + b;

    const callback: () => void = () => console.log("hi");
    ```

    - `CustomClasses`

- In object and interface types, if the variable of that type has additional fields or methods, it is ok.
- `?` is to mark the parameter as optional.

```ts
import axios from "axios";
const url = "https://jsonplaceholder.typicode.com/todos/2";

interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

axios.get(url).then((res) => {
  const todo: Todo = res.data;

  const id = todo.id;
  const title = todo.title;
  const completed = todo.completed;

  logTodo(id, title, completed);
});

const logTodo = (id: number, title: string, completed: boolean) => {
  console.log(`
    Todo with the id of ${id} is called ${title} and ${
    completed ? "is complete" : "is not completed"
  }.
  `);
};
```

## Type annotation vs type inference

- Just like C#, you don't have to define the types of things by `type annotations` all the time, sometimes the compiler can infer the type by `type inference` (like the time we declare and initialize at the same line):

```ts
const now: Date = new Date();
let n: number = 5;
```

```ts
const now = new Date();
let n = 5;
```

- We use `type annotations` ONLY when:
  - a function returns `any` and we need to clarify it.
  - we declare a variable in one line and don't initialize it, or in case of arrays we initialize it to an empty array.
  - when we want a variable to have a type that can't be inferred, like when it can have two types (we use `|` character).
  - when defining functions (annotate the args and return type both).

## Type alias

- You can define a `type alias` in advance:

```ts
type MyFunc = (n: number) => void;
```

## Difference between type alias and interfaces

1. Interface always introduces a named object type, but type is used for any kind of type.
2. Interfaces can be extended or implemented (In new versions of TS, types can be too).
3. An interface can have multiple merged declarations. Meaning, you define them twice, the merge mechanically joins the members of both declarations into a single interface with the same name.

## Type guards

```ts
if (variable instanceof Array) {
  //The RHS of instanceof must be a constructor function.
}

if (typeof variable === "string") {
  //The RHS of typeof must be a primitive type: number, string, boolean, symbol.
}
```

## Type assertions

- You can use type assertion with any kind of types.

```ts
const person = {
  id: 1,
  name: "Ben",
};

interface Person {
  id: number;
  name: string;
}

const person2 = { ...person } as Person;
```

## Destructuring and annotations

- We can't use destructuring and annotations at the same time, so we should:

```ts
const showWeather = ({
  date,
  weather,
}: {
  date: Date;
  weather: string;
}): void => {
  console.log(date);
  console.log(weather);
};
```

## Tuples

- In TS we have a data structure called tuple (not very useful, because we can't understand what is the value representing. Better to use objects. Maybe in CSV, we use tuples.):

```ts
const drink: [string, boolean, number] = ["brown", true, 40];
```

## Enums

- Enums allow us to define a set of named constants.
- They also define a new type.
- We use enums when we know _ALL_ the different values when we are writing our code.

```ts
enum Direction {
  Up, //will be 0
  Down, //will be 1 and so on
  Left,
  Right,
}
```

```ts
enum Direction {
  Up = 1,
  Down, //will be 2 and so on
  Left,
  Right,
}
```

```ts
enum Direction { // String enums
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}
```

## Classes

### Basics and Inheritance

```ts
class Vehicle {
  private startSound: string = "pt pt..";
  private static count: number = 0;

  constructor(public color: string, private owner: string) {
    Vehicle.count++; //static fields are accessible via class name.
  }

  get startingSound(): string {
    //The `get` keyword makes this method to be used like a property. This is called `accessor` in TS.
    return this.startSound;
  }

  public drive(): void {
    this.start();
    this.changeGear();
    console.log("ghungh ghungh...");
  }

  public honk(): void {
    console.log("beep");
  }

  private start(): void {
    console.log(this.startSound);
  }

  protected changeGear(): void {
    console.log("manual gear changing...");
  }

  static printCount(): number {
    return Vehicle.count;
  }
}

export class Car extends Vehicle {
  //export for Class makes it public. In TS, export default is not used often.
  constructor(public wheels: number, color: string, owner: string) {
    // The inherited fields should not have any access modifiers, because they belong to parent and inherits the behavior from the parent class.
    super(color, owner);
  }

  public honk(): void {
    //We can override parent's implementation of a method.
    //When we override, we can't change the modifier. It should be the same as the parent's.
    console.log("beeeeeeep...");
  }

  protected changeGear(): void {
    console.log("automatic gear changing...");
  }
}

const car0 = new Car(4, "blue", "Ben");
const car = new Car(4, "red", "Ben");
car.color = "blue";
console.log(car.startingSound); //pt pt..
console.log(car.color); //blue
console.log(Car.printCount()); //2 //static methods are accessible through class name.
// console.log(car.owner); //Property 'owner' is private and only accessible within class 'Vehicle'.
// console.log(car.startSound); //Property 'startSound' is private and only accessible within class 'Vehicle'.
// car.start(); //Property 'start' is private and only accessible within class 'Vehicle'.
car.drive();
// pt pt..
// automatic gear changing...
// ghungh ghungh...
car.honk();
// beeeeeeep...
```

### Access modifiers

- public: this member can be accessed any where. This is the default modifier, if we don't write anything.
- private: this member can be accessed only by other members of this class
- protected: this member can be accessed only by other members of this class, or child classes

### Fields initialization

- Either we initialize fields inline when defining them (usually when we want to hard-code them) or we use `constructor` method and initialize the fields in it:

```ts
class Car {
  color: string = "red";
  manufacturer: {
    name: string;
  } = { name: "Honda" };
}

const car = new Car();
console.log(car.color); //red
console.log(car.manufacturer.name); //Honda
car.manufacturer.name = "Ford";
```

OR

```ts
class Car {
  color: string;
  manufacturer: {
    name: string;
  };

  constructor(color: string, manufacturer: { name: string }) {
    this.color = color;
    this.manufacturer = manufacturer; //We cannot write: this.manufacturer.name = manufacturer.name; because `this.manufacturer` is undefined at this moment.
  }
}

const car = new Car("red", { name: "Honda" });
console.log(car.color); //red
console.log(car.manufacturer.name); //Honda
car.manufacturer.name = "Ford";
```

OR equivalently to the above, a `shorthand` version:

```ts
class Car {
  constructor(public color: string, public manufacturer: { name: string }) {}
}

const car = new Car("red", { name: "Honda" });
console.log(car.color); //red
console.log(car.manufacturer.name); //Honda
car.manufacturer.name = "Ford";
```

### Abstract Classes

- abstract methods, don't have any implementation (no {}).
- If a class has an abstract method, it should mark as abstract as well. Of course an abstract class can have non-abstract methods too for common behavior/code-reuse.
- abstract classes cannot be instantiated.
- All the derived classes from an abstract class `must` implement those abstract methods.

## Generics

- Generics are like function arguments, but for types in class/function/interface (so they are not limited to classes) definitions.
- Generics allows us to define the type of `a field of a class`/`arguments of a function`/`return value of a function` at a future point in time.
- They are heavily used when writing reusable code.

```ts
class GenericArray<T> {
  constructor(public collection: T[]) {}

  get(index: number): T {
    return this.collection[index];
  }
}

const arr = new GenericArray<number>([1, 2, 3, 4]);
console.log(arr.get(0)); //1
```

- When using generic `with functions`, you can't use arrow functions:

```ts
function printAnything<T>(arr: T[]): void {
  for (let i = 0; i < arr.length; i++) {
    console.log(arr[i]);
  }
}

printAnything<number>([1, 2, 3]);
```

- Another example (alternative approach for composition in CSV project):

```ts
import fs from "fs";

export abstract class CsvFileReader<T> {
  data: T[] = [];

  constructor(public filename: string) {}

  abstract mapRow(row: string[]): T;

  read(): void {
    this.data = fs
      .readFileSync(this.filename, {
        encoding: "utf-8",
      })
      .split("\n")
      .map((row: string): string[] => {
        return row.split(",");
      })
      .map(this.mapRow);
  }
}
```

```ts
import { CsvFileReader } from "./CsvFileReader";
import { dateStringToDate } from "./utils";
import { MatchResult } from "./MatchResult";

type MatchData = [Date, string, string, number, number, MatchResult, string];

export class MatchReader extends CsvFileReader<MatchData> {
  mapRow(row: string[]): MatchData {
    return [
      dateStringToDate(row[0]),
      row[1],
      row[2],
      parseInt(row[3]),
      parseInt(row[4]),
      row[5] as MatchResult,
      row[6],
    ];
  }
}
```

### Generic Constraints

```ts
class A {
  print() {
    console.log("A");
  }
}

class B {
  print() {
    console.log("B");
  }
}

interface Printable {
  print(): void;
}

function printAnything<T extends Printable>(arr: T[]): void {
  for (let i = 0; i < arr.length; i++) {
    arr[i].print();
  }
}

printAnything([new A(), new A(), new B()]); //Type inference
//A
//A
//B

//or
printAnything<A | B>([new A(), new A(), new B()]);
//A
//A
//B
```

- A very interesting example of constraints (this way, we tell TS that `get` method only can be called with the keys of T and it also knows the return type of each field of the interface.):

```ts
class Attributes<T> {
  constructor(private data: T) {}

  get<K extends keyof T>(key: K): T[K] {
    return this.data[key];
  }
}

interface UserProps {
  id?: number;
  name?: string;
}

const attr = new Attributes<UserProps>({ name: "Ben" });
```

![](/md/ts.jpg)

- Another interesting example:

```ts
interface ModelAttributes<T> {
  set(value: T): void;
  getAll(): T;
  get<K extends keyof T>(key: K): T[K];
}

interface Sync<T> {
  fetch(id: number): Promise<T>;
  save(data: T): Promise<T>;
}

interface Events {
  on(eventName: string, callback: () => void): void;
  trigger(eventName: string): void;
}

interface HasId {
  id?: number;
}

export class Model<T extends HasId> {
  constructor(
    private attributes: ModelAttributes<T>,
    private events: Events,
    private sync: Sync<T>
  ) {}

  on = this.events.on; //equivalent to: get on() { return this.events.on; }
  trigger = this.events.trigger;
  get = this.attributes.get;
  set(update: T): void {
    this.attributes.set(update);
    this.events.trigger("change");
  }

  fetch(): void {
    const id = this.get("id");

    if (typeof id !== "number") {
      throw new Error("Cannot fetch without an id");
    }

    this.sync.fetch(id).then((response: AxiosResponse): void => {
      this.set(response.data);
    });
  }

  save(): void {
    this.sync
      .save(this.attributes.getAll())
      .then((response: AxiosResponse): void => {
        this.trigger("save");
      })
      .catch(() => {
        this.trigger("error");
      });
  }
}

export abstract class View<T extends Model<K>, K> {
  // <------ INTERESTING
  constructor(public parent: Element, public model: T) {
    this.model.on("change", () => this.render());
  }

  abstract template(): string;

  render(): void {
    //Some reference to this.template();
    //Finding the right spot, binding events and rendering...
  }
}
```

## Decorators

- Are functions that can be used to modify/change/mess around with different properties/methods in a class.
- Are used inside/on classes only.
- Understanding the order in which decorators are ran are the key to understand them.
- Are experimental and you have to uncomment `"experimentalDecorators": true` and `"emitDecoratorMetadata": true` in `tsconfig.json` in order to use them. Remember by running `tsc --init` you can create a `tsconfig.json` file.

### Decorators on a property/method/accessor (the prop in C#, the one with get)

- First argument is the `prototype` of the object defining the class.
- Second argument is the key of the property/method/accessor.
- Third argument is the property descriptor which is of type `PropertyDescriptor`. This argument is an object that has some configuration options around the property (only for methods):

  - `writable` a boolean flag which describes whether or not this property can be changed.
  - `enumerable` a boolean flag which describes whether or not this property can get looped over by a `for...in`.
  - `value` current value.
  - `configurable` a boolean flag which describes whether or not this property can be changed or be deleted.

- Decorators are only executed once when the code for the class is ran (not when an instance is created).
- Decorators can be used on static properties, methods, and accessors as well.

```ts
@classDecorator //fifth -> [Function: Boat]
class Boat {
  @testDecorator //first (key) -> color
  color: string = "red";

  @testDecorator //second (key) -> formattedColor
  get formattedColor(): string {
    return `This boats color is ${this.color}`;
  }

  //Because we call the decorator immediately (to pass some args to it),
  //we have to define it as a decorator factory which is a normal function that returns our decorator.
  @logError("Oops, boat was sunk!") //seventh -> Oops, boat was sunk!
  pilot(
    @parameterDecorator speed: string, //third (key, index) -> pilot 1
    @parameterDecorator generateWake: boolean //fourth (key, index) -> pilot 0
  ): void {
    console.log(speed);
    if (speed === "fast") {
      console.log("swish...."); //sixth -> swish...
      throw new Error();
    } else {
      console.log("pssss...");
    }
  }
}

function classDecorator(constructor: typeof Boat) {
  //Boat is a reference to constructor function of that class.we could annotate it as : Function too.
  console.log(constructor); //This is the constructor function of the class.
}

function parameterDecorator(target: any, key: string, index: number) {
  //target is Boat { formattedColor: [Getter], pilot: [Function] }
  //Note that property definitions are not present in the prototype, because they are moved to constructor function when compiling to JS.
  console.log(key, index);
}

function testDecorator(target: any, key: string) {
  //target is Boat { formattedColor: [Getter], pilot: [Function] }
  console.log(key);
}

function logError(errorMessage: string) {
  return function (target: any, key: string, desc: PropertyDescriptor): void {
    //target is Boat { formattedColor: [Getter], pilot: [Function] }
    const method = desc.value;

    desc.value = function (...args: any[]) {
      // We override this method by this assignment.
      try {
        method(...args);
      } catch (e) {
        console.log(errorMessage);
      }
    };
  };
}

const boat = new Boat();
boat.pilot("fast", false);
```

## Metadata

```dos
npm i reflect-metadata
```

- When we import this package by `import "reflect-metadata";`, it automatically adds a single variable to global scope: `Reflect`.
- `Reflect.DefineMetadata` defines a unique metadata entry on the target object or one of its properties.
- Metadata is like secret info that doesn't show up anywhere except through the use of this `reflect-metadata` package.
- The `first` argument is like the `key` of the metadata.
- The `second` argument is like the `value` of the metadata.
- The `third` argument is `the object` that this metadata is going to attach to invisibly.
- The `optional fourth` argument is the `property of the object` that this metadata is going to attach to invisibly.
- The metadata of an object describes some special stuff about the object.

```ts
import "reflect-metadata";

const plane = {
  color: "red",
};

Reflect.defineMetadata("note", "hi there", plane);
Reflect.defineMetadata("height", 10, plane);
Reflect.defineMetadata("note", "bye there", plane, "color");

const note = Reflect.getMetadata("note", plane);
const height = Reflect.getMetadata("height", plane);
const colorNote = Reflect.getMetadata("note", plane, "color");

console.log(note, height, colorNote); //hi there 10 bye there
```

![](/md/metadata.jpg)

### Practical example

```ts
import "reflect-metadata";

@controller
class Plane {
  color: string = "red";

  @get("/login")
  fly(): void {
    console.log("vrrrrrrr");
  }
}

function get(path: string) {
  return function (prototype: Plane, methodName: string) {
    Reflect.defineMetadata("path", path, prototype, methodName);
  };
}

function controller(constructor: typeof Plane) {
  for (let methodName in constructor.prototype) {
    const path = Reflect.getMetadata("path", constructor.prototype, methodName);
    const middleware = Reflect.getMetadata(
      "middleware",
      constructor.prototype,
      methodName
    );

    router.get(path, middleware, constructor.prototype[methodName]);
  }
}
```

# Design Patterns

- We will create a class for every entity in a separate file. In the beginning of this file we define the interfaces for working with this class which will be instructions to every other classes on how they can be an argument to methods of this class. In other classes, we can omit `implements Mappable`. But it is good practice for pinpointing errors.
- To use a JS package in TS (even the standard node libraries such as `fs`), we need a `type definition file` which is like an adapter. Sometimes these files installed when you install the package. If not, try to install `npm i @types/{package name}`. For node standard libraries, use `npm i @types/node`. After installing, by `F12` or `ctrl + click` you can explore the type definition file.
- With `composition` relationship with `private` access modifier for ClassB, we limit the users of ClassA to the methods that we want to expose.

## Maps Project

- `parcel-bundler` is a tool to help us run TS in the browser:

```dos
npm i -g parcel-bundler
```

- Create an `index.html`:

```html
<html>
</head>

<body>
    <script src="./src/index.ts"></script>
</body>

</html>
```

- Create a folder `src` and `index.ts` in it:

```ts
console.log("Hi there!");
```

- Run:

```dos
parcel index.html

Server running at http://localhost:1234
√  Built in 7.73s.
```

- Parcel compiles the TS file and feed the JS file to index.html.
- `index.ts` will be the entry point to our app and we will import other classes in it.
- For using google maps, create a project in `console.developers.google.com`, enable `Maps JavaScript Api` for it, copy the `API key`, and add the script tag inside `index.html`.
- Because we will add google object in `index.html`, it is a global module and we don't have to import it in each file. But to tell TS that `google` is a global variable, we install `npm i @types/googlemaps`
- `index.html`:

```html
<html>
  <body>
    <div id="map" style="height: 100%;"></div>
    <script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDowSjEG4Im1alZ9v8nBkLcyBPsIjJ3x9E"></script>
    <script src="./src/index.ts"></script>
  </body>
</html>
```

- `User.ts`:

```ts
import faker from "faker";
import { Mappable } from "./CustomMap";

export class User implements Mappable {
  name: string;
  location: {
    lat: number;
    lng: number;
  };

  constructor() {
    this.name = faker.name.firstName();
    this.location = {
      lat: parseFloat(faker.address.latitude()), //faker returns a string for lat and lng
      lng: parseFloat(faker.address.longitude()),
    };
  }

  markerContent(): string {
    return `User Name: ${this.name}`;
  }
}
```

- `Company.ts`:

```ts
import faker from "faker";
import { Mappable } from "./CustomMap";

export class Company implements Mappable {
  companyName: string;
  catchPhrase: string;
  location: {
    lat: number;
    lng: number;
  };

  constructor() {
    this.companyName = faker.company.companyName();
    this.catchPhrase = faker.company.catchPhrase();
    this.location = {
      lat: parseFloat(faker.address.latitude()),
      lng: parseFloat(faker.address.longitude()),
    };
  }

  markerContent(): string {
    return `
      <div>
        <h1>Company Name: ${this.companyName}</h1>
        <h3>Catchphrase: ${this.catchPhrase}</h3>
      </div>
    `;
  }
}
```

- `CustomMap.ts`:

```ts
export interface Mappable {
  location: {
    lat: number;
    lng: number;
  };

  markerContent(): string;
}

export class CustomMap {
  private googleMap: google.maps.Map; //google.maps is an object (namespace) which Maps class has been defined in.

  constructor(divId: string) {
    this.googleMap = new google.maps.Map(document.getElementById(divId), {
      zoom: 1,
      center: {
        lat: 0,
        lng: 0,
      },
    });
  }

  addMarker(mappable: Mappable): void {
    const marker = new google.maps.Marker({
      map: this.googleMap,
      position: {
        lat: mappable.location.lat,
        lng: mappable.location.lng,
      },
    });

    marker.addListener("click", () => {
      const infoWindow = new google.maps.InfoWindow({
        content: mappable.markerContent(),
      });

      infoWindow.open(this.googleMap, marker);
    });
  }
}
```

- `index.ts`:

```ts
import { User } from "./User";
import { Company } from "./Company";
import { CustomMap } from "./CustomMap";

const user = new User();
const company = new Company();
const customMap = new CustomMap("map");

customMap.addMarker(user);
customMap.addMarker(company);
```

## Sort Project

- Comparison between inheritance and composition:
  - In this project because class NumbersCollection and class CharactersCollection are similar in nature (compare to the previous project in which User and Company is very different), it is better to use inheritance although it will make Sorter class tightly coupled to them.
  - As you will see, with composition, we have to create a sort method and call sort() method of the Sorter class for both owner classes (or at least passthrough a reference to it). Alternatively, we could make `sorter` field public and use it like `numbersCollection.sorter.sort()` which would be awkward if we had other methods inside `NumbersCollection` and `CharactersCollection` classes; because some methods should be called directly and some methods should be called through `sorter` field.
- `Sorter.ts`:

```ts
export abstract class Sorter {
  abstract length: number;
  abstract compare(leftIndex: number, rightIndex: number): boolean;
  abstract swap(leftIndex: number, rightIndex: number): void;

  sort(): void {
    const { length } = this;

    for (let i = 0; i < length; i++) {
      for (let j = 0; j < length - i - 1; j++) {
        //Bubble sort algorithm: in each iteration, the largest will be on the far right.
        if (this.compare(j, j + 1)) {
          this.swap(j, j + 1);
        }
      }
    }
  }
}
```

- Alternatively, we could use interface approach:

```ts
export interface Sortable {
  length: number;
  compare(leftIndex: number, rightIndex: number): boolean;
  swap(leftIndex: number, rightIndex: number): void;
}

export class Sorter {
  constructor(public collection: Sortable) {}

  sort(): void {
    // sort = (): void => {  <--- OTHER WAY to solve binding issue (using arrow functions)
    const { length } = this.collection;

    for (let i = 0; i < length; i++) {
      for (let j = 0; j < length - i - 1; j++) {
        if (this.collection.compare(j, j + 1)) {
          this.collection.swap(j, j + 1);
        }
      }
    }
  }
}
```

- `NumbersCollection.ts`:

```ts
import { Sorter } from "./Sorter";

export class NumbersCollection extends Sorter {
  constructor(public data: number[]) {
    super();
  }

  get length(): number {
    return this.data.length;
  }

  compare(leftIndex: number, rightIndex: number): boolean {
    return this.data[leftIndex] > this.data[rightIndex];
  }

  swap(leftIndex: number, rightIndex: number): void {
    const leftHand = this.data[leftIndex];
    this.data[leftIndex] = this.data[rightIndex];
    this.data[rightIndex] = leftHand;
  }
}
```

- Alternatively, we could use interface approach:

```ts
import { Sorter } from "./Sorter";
import { Sortable } from "./Sorter";

export class NumbersCollection implements Sortable {
  private sorter: Sorter;

  constructor(public data: number[]) {
    this.sorter = new Sorter(this); //This can be a sign that inheritance is better in this case.
  }

  get length(): number {
    return this.data.length;
  }

  compare(leftIndex: number, rightIndex: number): boolean {
    return this.data[leftIndex] > this.data[rightIndex];
  }

  swap(leftIndex: number, rightIndex: number): void {
    const leftHand = this.data[leftIndex];
    this.data[leftIndex] = this.data[rightIndex];
    this.data[rightIndex] = leftHand;
  }

  get sort() {
    //Direct passthrough. With this trick, we don't have to duplicate the method signature (`sort(): void { this.sorter.sort(); }`).
    return this.sorter.sort.bind(this.sorter); //Instead of binding this.sorter,
    //we could define methods in classes as arrow functions all the time. So on `Sorter` class:
    //Instead of `sort(): void {`, we use
    //sort = (): void => {
    //Note that no const is needed at the beginning.
  }
}
```

- `CharactersCollection.ts`:

```ts
import { Sorter } from "./Sorter";

export class CharactersCollection extends Sorter {
  constructor(public data: string) {
    super();
  }

  get length(): number {
    return this.data.length;
  }

  compare(leftIndex: number, rightIndex: number): boolean {
    return (
      this.data[leftIndex].toLowerCase() > this.data[rightIndex].toLowerCase()
    );
  }

  swap(leftIndex: number, rightIndex: number): void {
    const characters = this.data.split("");

    const leftHand = characters[leftIndex];
    characters[leftIndex] = characters[rightIndex];
    characters[rightIndex] = leftHand;

    this.data = characters.join("");
  }
}
```

- Alternatively, we could use interface approach:

```ts
import { Sorter } from "./Sorter";
import { Sortable } from "./Sorter";

export class CharactersCollection implements Sortable {
  private sorter: Sorter;

  constructor(public data: string) {
    this.sorter = new Sorter(this);
  }

  get length(): number {
    return this.data.length;
  }

  compare(leftIndex: number, rightIndex: number): boolean {
    return (
      this.data[leftIndex].toLowerCase() > this.data[rightIndex].toLowerCase()
    );
  }

  swap(leftIndex: number, rightIndex: number): void {
    const characters = this.data.split("");

    const leftHand = characters[leftIndex];
    characters[leftIndex] = characters[rightIndex];
    characters[rightIndex] = leftHand;

    this.data = characters.join("");
  }

  get sort() {
    //Direct passthrough.
    return this.sorter.sort.bind(this.sorter);
  }
}
```

- `index.ts`:

```ts
import { NumbersCollection } from "./NumbersCollection";
import { CharactersCollection } from "./CharactersCollection";

const numbersCollection = new NumbersCollection([50, 3, -5, 0]);
numbersCollection.sort();
console.log(numbersCollection.data);

const charactersCollection = new CharactersCollection("Xaayb");
charactersCollection.sort();
console.log(charactersCollection.data);
```

## CSV Project

- Imagine you have a CSV like this:

```csv
10/08/2018,Man United,Leicester,2,1,H,A Marriner
11/08/2018,Bournemouth,Cardiff,2,0,H,K Friend
11/08/2018,Fulham,Crystal Palace,0,2,A,M Dean
11/08/2018,Huddersfield,Chelsea,0,3,A,C Kavanagh
11/08/2018,Newcastle,Tottenham,1,2,A,M Atkinson
11/08/2018,Watford,Brighton,2,0,H,J Moss
...
```

- `CsvFileReader.ts`: To read from file and turn it into array of arrays (string).

```ts
import fs from "fs";
import { DataReader } from "./MatchReader";

export class CsvFileReader implements DataReader {
  data: string[][] = [];

  constructor(public filename: string) {}

  read(): void {
    this.data = fs
      .readFileSync(this.filename, {
        encoding: "utf-8", //If omitted, returns a buffer.
      })
      .split("\n")
      .map((row: string): string[] => {
        return row.split(",");
      });
  }
}
```

- `MatchResult.ts`: To avoid magic strings.

```ts
export enum MatchResult {
  HomeWin = "H",
  AwayWin = "A",
  Draw = "D",
}
```

- `utils.ts`:

```ts
export const dateStringToDate = (dateString: string): Date => {
  const dateParts = dateString.split("/").map((value: string): number => {
    return parseInt(value);
  });

  return new Date(dateParts[2], dateParts[1] - 1, dateParts[0]);
};
```

- `MatchData.ts`: The type of each row.

```ts
import { MatchResult } from "./MatchResult";

export type MatchData = [
  //Tuple
  Date,
  string,
  string,
  number,
  number,
  MatchResult,
  string
];
```

- `MatchReader.ts`: To turn it into array of arrays (with correct type).

```ts
import { dateStringToDate } from "./utils";
import { MatchResult } from "./MatchResult";
import { MatchData } from "./MatchData";
import { CsvFileReader } from "./CsvFileReader";

export interface DataReader {
  read(): void;
  data: string[][];
}

export class MatchReader {
  matches: MatchData[] = [];

  constructor(public reader: DataReader) {}

  load(): void {
    this.reader.read();
    this.matches = this.reader.data.map(
      (row: string[]): MatchData => {
        return [
          dateStringToDate(row[0]),
          row[1],
          row[2],
          parseInt(row[3]),
          parseInt(row[4]),
          row[5] as MatchResult,
          row[6],
        ];
      }
    );
  }

  static fromCsv(filename: string): MatchReader {
    //We could take a filename in the ctor and initialize the reader object with it but that was not dependency injection.
    //We could instantiate a CsvFileReader in index.ts and pass it to MatchReader to use dependency injection.
    //But this way, we used a static method to inject dependency.
    return new MatchReader(new CsvFileReader(filename));
  }
}
```

- `WinsAnalysis.ts`:

```ts
import { Analyzer } from "../Summary";
import { MatchData } from "../MatchData";
import { MatchResult } from "../MatchResult";

export class WinsAnalysis implements Analyzer {
  constructor(public team: string) {}

  run(matches: MatchData[]): string {
    let wins = 0;

    for (let match of matches) {
      if (match[1] === this.team && match[5] === MatchResult.HomeWin) {
        wins++;
      } else if (match[2] === this.team && match[5] === MatchResult.AwayWin) {
        wins++;
      }
    }

    return `Team ${this.team} won ${wins} games`;
  }
}
```

- `ConsoleReport.ts`:

```ts
import { OutputTarget } from "../Summary";

export class ConsoleReport implements OutputTarget {
  print(report: string): void {
    console.log(report);
  }
}
```

- `HtmlReport.ts`:

```ts
import fs from "fs";
import { OutputTarget } from "../Summary";

export class HtmlReport implements OutputTarget {
  print(report: string): void {
    const html = `
      <div>
        <h1>Analysis Output</h1>
        <div>${report}</div>
      </div>
    `;

    fs.writeFileSync("report.html", html);
  }
}
```

- `Summary.ts`:

```ts
import { MatchData } from "./MatchData";
import { WinsAnalysis } from "./analyzers/WinsAnalysis";
import { HtmlReport } from "./reportTargets/HtmlReport";

export interface Analyzer {
  run(matches: MatchData[]): string;
}

export interface OutputTarget {
  print(report: string): void;
}

export class Summary {
  constructor(public analyzer: Analyzer, public outputTarget: OutputTarget) {}

  buildAndPrintReport(matches: MatchData[]): void {
    const output = this.analyzer.run(matches);
    this.outputTarget.print(output);
  }

  static winsAnalysisWithHtmlReport(team: string): Summary {
    return new Summary(new WinsAnalysis(team), new HtmlReport());
  }
}
```

- `index.ts`:

```ts
import { MatchReader } from "./MatchReader";
import { Summary } from "./Summary";

const matchReader = MatchReader.fromCsv("football.csv");
const summary = Summary.winsAnalysisWithHtmlReport("Man United");

matchReader.load();
summary.buildAndPrintReport(matchReader.matches);
```

# Express and Typescript

## Project structure

```dos
npm init -y
tsc --init
npm i --save-dev concurrently nodemon
```

- Now create two folders `src` and `build` in the root of the project.
- Uncomment `outDir` and `rootDir` in the `tsconfig.json` and give them the address:

```json
{
  //...
  "outDir": "./build" /* Redirect output structure to the directory. */,
  "rootDir": "./src"
  //...
}
```

- Add the following in the `package.json` file:

```json
{
  //...
  "scripts": {
    "start:build": "tsc -w",
    "start:run": "nodemon build/index.js",
    "start": "concurrently npm:start:*"
  }
  //...
}
```

- Create `index.ts` file in the `src` folder with a single `console.log("hi");`.
- Now start the project with `npm start`.
- Run:

```dos
npm i express @types/express cookie-session @types/cookie-session
```

## Mild change

- Using TS just to add some basic type annotations where possible.
- `index.ts`:

```ts
import express from "express";
import { router } from "./routes/loginRoutes";
import cookieSession from "cookie-session"; //This module stores the session data on the client within a cookie.

const app = express();

app.use(express.urlencoded({ extended: true }));
//It parses the body of an incoming requests (which is in the urlencoded form) and puts the result (JS object form) in the req.body. Note that it is in the body but is in the form of a url: email=ben&password=1234
//express.json parses the incoming requests (which is in the JSON form) and puts the result (JS object form) in the req.body.
app.use(cookieSession({ keys: ["abc"] })); //keys[0] is used to sign & verify cookie values.
//This middleware parses the cookie in the request header and if it is verified, it populates req.session.
app.use(router);

app.listen(3000, () => {
  console.log("Listening on port 3000");
});
```

- `loginRoutes.ts`:

```ts
import { Router, Request, Response, NextFunction } from "express";

interface RequestWithBody extends Request {
  //This is because the type definition file comes with express has defined body poorly: body: any;
  body: { [key: string]: string | undefined };
}

//Middlewares
function requireAuth(req: Request, res: Response, next: NextFunction): void {
  //Request, Response, and NextFunction are interfaces.
  if (req.session && req.session.loggedIn) {
    next();
    return;
  }

  res.status(403);
  res.send("Not permitted");
}

//Routes
const router = Router();

router.get("/login", (req: Request, res: Response) => {
  res.send(`
    <form method="POST">
      <div>
        <label>Email</label>
        <input name="email" />
      </div>
      <div>
        <label>Password</label>
        <input name="password" type="password" />
      </div>
      <button>Submit</button>
    </form>
  `);
});

router.post("/login", (req: RequestWithBody, res: Response) => {
  const { email, password } = req.body;

  if (email && password && email === "test@test.com" && password === "1234") {
    req.session = { loggedIn: true };
    //Represents the session for the given request.
    //When we set a property on a session, it will be signed and set in the response header: Set-Cookie: express:sess=eyJsb2dnZWRJbiI6dHJ1ZX0=; path=/; httponly.
    res.redirect("/"); //This is where actually the response is sent to the client.
  } else {
    res.send("Invalid email or password");
  }
});

router.get("/", (req: Request, res: Response) => {
  if (req.session && req.session.loggedIn) {
    res.send(`
      <div>
        <div>You are logged in</div>
        <a href="/logout">Logout</a>
      </div>
    `);
  } else {
    res.send(`
      <div>
        <div>You are not logged in</div>
        <a href="/login">Login</a>
      </div>
    `);
  }
});

router.get("/logout", (req: Request, res: Response) => {
  req.session = undefined; //Resetting session.
  res.redirect("/");
});

router.get("/protected", requireAuth, (req: Request, res: Response) => {
  res.send("Welcome to protected route, logged in user");
});

export { router };
```

## Harsh change

- Using TS to turn express code into classes + use some advanced features of TS (classes and interfaces).
- In addition to project setup, run `npm i reflect-metadata` and uncomment `"experimentalDecorators": true` and `"emitDecoratorMetadata": true` in `tsconfig.json`.
- In an express app, there is `one` instance of `Router` (one router object) that has five different methods of `get`, `post`, `put`, `del`, and `patch`.
- It is called like: `router.get(path: PathParams, ...handlers: RequestHandler[])`. handlers are essentially middlewares. We want that handlers to be called in this order: validator, other middlewares, and at last route handler.
- The problem is that we want to register different midllerwares by decorating route handler. How to tell the `router` to register `second middleware` (by decorating with second decorator) to the `route handler` that comes out of applying the `first decorator`?
  - When each decorator is called, we will store the configuration info they have on the method property (which is the key of each decorator) of the prototype object (which is the target of each decorator) by using metadata.
  - After running all decorators, class decorator (`@controller`) runs last and it is where we read metadata of each method and create a complete route definitions to router.
- In the root folder:

`AppRouter.ts`:

```ts
import express from "express";

export class AppRouter {
  private static instance: express.Router;

  static getInstance(): express.Router {
    //This is how we setup a singleton in TS:
    if (!AppRouter.instance) {
      AppRouter.instance = express.Router();
    }

    return AppRouter.instance;
  }
}
```

`index.ts`:

```ts
import express from "express";
import cookieSession from "cookie-session";
import { AppRouter } from "./AppRouter";
import "./controllers/LoginController";
import "./controllers/RootController";

const app = express();

app.use(express.urlencoded({ extended: true }));
app.use(cookieSession({ keys: ["abc"] }));
app.use(AppRouter.getInstance());

app.listen(3000, () => {
  console.log("Listening on port 3000");
});
```

- In the `src/controllers/decorators` folder:

`MetadataKeys.ts`:

```ts
export enum MetadataKeys {
  httpMethod = "httpMethod",
  path = "path",
  middleware = "middleware",
  validator = "validator",
}
```

`HttpMethods.ts`:

```ts
export enum HttpMethods {
  get = "get",
  post = "post",
  patch = "patch",
  del = "delete",
  put = "put",
}
```

`bodyValidator.ts`:

```ts
import "reflect-metadata";
import { MetadataKeys } from "./MetadataKeys";

export function bodyValidator(...keys: string[]) {
  return function (
    prototype: any,
    methodName: string,
    desc: PropertyDescriptor
  ) {
    Reflect.defineMetadata(MetadataKeys.validator, keys, prototype, methodName);
    //It is like defining an invisible property: `validator: ["email", "password"]`
    //in the prototype object of the constructor function that this decorator is going to be applied on its method(s).
  };
}
```

`routes.ts`:

```ts
import "reflect-metadata";
import { RequestHandler } from "express";
import { HttpMethods } from "./HttpMethods";
import { MetadataKeys } from "./MetadataKeys";

interface RouteHandlerDescriptor extends PropertyDescriptor {
  value?: RequestHandler; //RequestHandler is a function that receives a request and a response.
}

function routeBinder(httpMethod: string) {
  //To avoid writing same function over and over again we used a factory function.
  return function (path: string) {
    return function (
      prototype: any,
      methodName: string,
      desc: RouteHandlerDescriptor //This is to make sure that `desc.value` which should be a route handler is actually a route handler and not a random function.
      //So with this, we can't apply this decorator to non-routeHandler functions. So with PropertyDescriptor, we can actually limit how we actually use a decorator.
      //We should define an interface that extends PropertyDescriptor and then customize that `value` property.
    ) {
      Reflect.defineMetadata(MetadataKeys.path, path, prototype, methodName);
      Reflect.defineMetadata(
        MetadataKeys.httpMethod,
        httpMethod,
        prototype,
        methodName
      );
    };
  };
}

export const get = routeBinder(HttpMethods.get);
export const put = routeBinder(HttpMethods.put);
export const post = routeBinder(HttpMethods.post);
export const del = routeBinder(HttpMethods.del);
export const patch = routeBinder(HttpMethods.patch);
```

`controller.ts`:

```ts
import "reflect-metadata";
import { Request, Response, RequestHandler, NextFunction } from "express";
import { AppRouter } from "../../AppRouter";
import { HttpMethods } from "./HttpMethods";
import { MetadataKeys } from "./MetadataKeys";

function bodyValidators(keys: string): RequestHandler {
  return function (req: Request, res: Response, next: NextFunction) {
    if (!req.body) {
      res.status(422).send("Invalid request");
      return;
    }

    for (let key of keys) {
      if (!req.body[key]) {
        res.status(422).send(`Missing property ${key}`);
        return;
      }
    }

    next();
  };
}

export function controller(routePrefix: string) {
  return function (constructor: Function) {
    const router = AppRouter.getInstance();

    for (let methodName in constructor.prototype) {
      const routeHandler = constructor.prototype[methodName];
      const path = Reflect.getMetadata(
        MetadataKeys.path,
        constructor.prototype,
        methodName
      );
      const httpMethod: HttpMethods = Reflect.getMetadata(
        MetadataKeys.httpMethod,
        constructor.prototype,
        methodName
      );
      const middlewares =
        Reflect.getMetadata(
          MetadataKeys.middleware,
          constructor.prototype,
          methodName
        ) || [];
      const requiredBodyProps =
        Reflect.getMetadata(
          MetadataKeys.validator,
          constructor.prototype,
          methodName
        ) || [];

      const validator = bodyValidators(requiredBodyProps);

      if (path) {
        router[httpMethod](
          `${routePrefix}${path}`,
          validator,
          ...middlewares,
          routeHandler
        );
      }
    }
  };
}
```

`use.ts`:

```ts
import "reflect-metadata";
import { RequestHandler } from "express";
import { MetadataKeys } from "./MetadataKeys";

export function use(middleware: RequestHandler) {
  return function (
    prototype: any,
    methodName: string,
    desc: PropertyDescriptor
  ) {
    const registeredMiddlewares =
      Reflect.getMetadata(MetadataKeys.middleware, prototype, methodName) || [];

    Reflect.defineMetadata(
      MetadataKeys.middleware,
      [...registeredMiddlewares, middleware],
      prototype,
      methodName
    );
  };
}
```

`index.ts`:

```ts
export * from "./controller";
export * from "./routes";
export * from "./use";
export * from "./bodyValidator";
```

- In the `src/controllers` folder:

`RootController.ts`:

```ts
import { Request, Response, NextFunction } from "express";
import { get, controller, use } from "./decorators";

function requireAuth(req: Request, res: Response, next: NextFunction): void {
  if (req.session && req.session.loggedIn) {
    next();
    return;
  }

  res.status(403);
  res.send("Not permitted");
}

@controller("")
class RootController {
  @get("/")
  getRoot(req: Request, res: Response) {
    if (req.session && req.session.loggedIn) {
      res.send(`
        <div>
          <div>You are logged in</div>
          <a href="/auth/logout">Logout</a>
        </div>
      `);
    } else {
      res.send(`
        <div>
          <div>You are not logged in</div>
          <a href="/auth/login">Login</a>
        </div>
      `);
    }
  }

  @get("/protected")
  @use(requireAuth)
  getProtected(req: Request, res: Response) {
    res.send("Welcome to protected route, logged in user");
  }
}
```

`LoginController.ts`:

```ts
import { Request, Response, NextFunction } from "express";
import { get, controller, bodyValidator, post } from "./decorators";

@controller("/auth")
class LoginController {
  @get("/login")
  getLogin(req: Request, res: Response): void {
    res.send(`
      <form method="POST">
        <div>
          <label>Email</label>
          <input name="email" />
        </div>
        <div>
          <label>Password</label>
          <input name="password" type="password" />
        </div>
        <button>Submit</button>
      </form>
    `);
  }

  @post("/login")
  @bodyValidator("email", "password")
  postLogin(req: Request, res: Response) {
    const { email, password } = req.body;

    if (email === "hi@hi.com" && password === "password") {
      req.session = { loggedIn: true };
      res.redirect("/");
    } else {
      res.send("Invalid email or password");
    }
  }

  @get("/logout")
  getLogout(req: Request, res: Response) {
    req.session = undefined;
    res.redirect("/");
  }
}
```
