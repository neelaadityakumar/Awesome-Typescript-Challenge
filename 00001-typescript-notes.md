### Differences Between Type Aliases and Interfaces

Type aliases and interfaces are very similar, and in many cases, you can choose between them freely.
Almost all features of an `interface` are available in `type`.
The key distinction is that a **type cannot be re-opened to add new properties**, whereas an **interface is always extendable**.

---

#### **1. Extending Types and Interfaces**

- **Using `interface` (Extending with `extends`)**

  ```ts
  interface Animal {
    name: string;
  }

  interface Bear extends Animal {
    honey: boolean;
  }

  const bear = getBear();
  bear.name;
  bear.honey;
  ```

- **Adding new fields to an existing interface**

````ts

interface Window {
  title: string;
}
interface Window {
  ts: TypeScriptAPI;
}
const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});```
````

#### **2. Extending a type via intersections**

```ts
type Animal = {
  name: string;
};
type Bear = Animal & {
  honey: boolean;
};
const bear = getBear();
bear.name;
bear.honey;
```

- **A type cannot be changed after being created**

```ts
type Window = {
  title: string;
};
type Window = {
  ts: TypeScriptAPI;
}; // Error: Duplicate identifier 'Window'.
```

#### **3. The `in` operator narrowing**

JavaScript has an operator for determining if an object or its prototype chain has a property with a name: the `in` operator. TypeScript takes this into account as a way to narrow down potential types.

For example, with the code: `"value" in x`. where `"value"` is a string literal and `x` is a union type. The “true” branch narrows `x`’s types which have either an optional or required property `value`, and the “false” branch narrows to types which have an optional or missing property `value`.

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }

  return animal.fly();
}
```

To reiterate, optional properties will exist in both sides for narrowing. For example, a human could both swim and fly (with the right equipment) and thus should show up in both sides of the `in` check:

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };

function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) {
    animal;
      (parameter) animal: Fish | Human  } else {
    animal;
      (parameter) animal: Bird | Human  }
}

```

#### **4. Constraints**

We’ve written some generic functions that can work on *any* kind of value. Sometimes we want to relate two values, but can only operate on a certain subset of values. In this case, we can use a *constraint* to limit the kinds of types that a type parameter can accept.

Let’s write a function that returns the longer of two values. To do this, we need a `length` property that’s a number. We *constrain* the type parameter to that type by writing an `extends` clause:

```ts
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}

// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");
// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.Try

```

There are a few interesting things to note in this example. We allowed TypeScript to *infer* the return type of `longest`. Return type inference also works on generic functions.

Because we constrained `Type` to `{ length: number }`, we were allowed to access the `.length` property of the `a` and `b` parameters. Without the type constraint, we wouldn’t be able to access those properties because the values might have been some other type without a length property.

The types of `longerArray` and `longerString` were inferred based on the arguments. Remember, generics are all about relating two or more values with the same type!

Finally, just as we’d like, the call to `longest(10, 100)` is rejected because the `number` type doesn’t have a `.length` property.

### Working with Constrained Values

Here’s a common error when working with generic constraints:

```ts
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum };
Type '{ length: number; }' is not assignable to type 'Type'.
  '{ length: number; }' is assignable to the constraint of type 'Type', but 'Type' could be instantiated with a different subtype of constraint '{ length: number; }'.Type '{ length: number; }' is not assignable to type 'Type'.
  '{ length: number; }' is assignable to the constraint of type 'Type', but 'Type' could be instantiated with a different subtype of constraint '{ length: number; }'.  }
}
Try

```

It might look like this function is OK - `Type` is constrained to `{ length: number }`, and the function either returns `Type` or a value matching that constraint. The problem is that the function promises to return the *same* kind of object as was passed in, not just *some* object matching the constraint. If this code were legal, you could write code that definitely wouldn’t work:

```ts
// 'arr' gets value { length: 6 }
const arr = minimumLength([1, 2, 3], 6);
// and crashes here because arrays have
// a 'slice' method, but not the returned object!
console.log(arr.slice(0));
```

###

## Function Overloads

Some JavaScript functions can be called in a variety of argument counts and types. For example, you might write a function to produce a `Date` that takes either a timestamp (one argument) or a month/day/year specification (three arguments).

In TypeScript, we can specify a function that can be called in different ways by writing *overload signatures*. To do this, write some number of function signatures (usually two or more), followed by the body of the function:

```ts
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
No overload expects 2 arguments, but overloads do exist that expect either 1 or 3 arguments.

```

### `unknown`

The `unknown` type represents *any* value. This is similar to the `any` type, but is safer because it’s not legal to do anything with an `unknown` value:

```ts
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b();
'a' is of type 'unknown'.'a' is of type 'unknown'.}
Try

```

This is useful when describing function types because you can describe functions that accept any value without having `any` values in your function body.

Conversely, you can describe a function that returns a value of unknown type:

```ts
function safeParse(s: string): unknown {
  return JSON.parse(s);
}

// Need to be careful with 'obj'!
const obj = safeParse(someRandomString);
```

### `never`

Some functions *never* return a value:

```ts
function fail(msg: string): never {
  throw new Error(msg);
}
```

The `never` type represents values which are *never* observed. In a return type, this means that the function throws an exception or terminates execution of the program.

`never` also appears when TypeScript determines there’s nothing left in a union.

```ts
function fn(x: string | number) {
  if (typeof x === "string") {
    // do something
  } else if (typeof x === "number") {
    // do something else
  } else {
    x; // has type 'never'!
  }
}
```

###

### `Function`

The global type `Function` describes properties like `bind`, `call`, `apply`, and others present on all function values in JavaScript. It also has the special property that values of type `Function` can always be called; these calls return `any`:

```ts
function doSomething(f: Function) {
  return f(1, 2, 3);
}
```

This is an *untyped function call* and is generally best avoided because of the unsafe `any` return type.

If you need to accept an arbitrary function but don’t intend to call it, the type `() => void` is generally safer.

TypeScript does not assume that arrays are immutable. This can lead to some surprising behavior:

```ts
// Inferred type is number[] -- "an array with zero or more numbers",
// not specifically two numbers
const args = [8, 5];
const angle = Math.atan2(...args);
A spread argument must either have a tuple type or be passed to a rest parameter.A spread argument must either have a tuple type or be passed to a rest parameter.Try

```

The best fix for this situation depends a bit on your code, but in general a `const` context is the most straightforward solution:

```ts
// Inferred as 2-length tuple
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```

### Return type `void`

The `void` return type for functions can produce some unusual, but expected behavior.

Contextual typing with a return type of `void` does **not** force functions to **not** return something. Another way to say this is a contextual function type with a `void` return type (`type voidFunc = () => void`), when implemented, can return *any* other value, but it will be ignored.\

Note that there is currently no way to place type annotations within destructuring patterns. This is because the following syntax already means something different in JavaScript.

```ts
function draw({ shape: Shape, xPos: number = 100 /*...*/ }) {
  render(shape);
Cannot find name 'shape'. Did you mean 'Shape'?Cannot find name 'shape'. Did you mean 'Shape'?  render(xPos);
Cannot find name 'xPos'.Cannot find name 'xPos'.}
Try

```

In an object destructuring pattern, `shape: Shape` means “grab the property `shape` and redefine it locally as a variable named `Shape`.” Likewise `xPos: number` creates a variable named `number` whose value is based on the parameter’s `xPos`.

It’s important to manage expectations of what `readonly` implies. It’s useful to signal intent during development time for TypeScript on how an object should be used. TypeScript doesn’t factor in whether properties on two types are `readonly` when checking whether those types are compatible, so `readonly` properties can also change via aliasing.

```ts
interface Person {
  name: string;
  age: number;
}

interface ReadonlyPerson {
  readonly name: string;
  readonly age: number;
}

let writablePerson: Person = {
  name: "Person McPersonface",
  age: 42,
};

// works
let readonlyPerson: ReadonlyPerson = writablePerson;

console.log(readonlyPerson.age); // prints '42'
writablePerson.age++;
console.log(readonlyPerson.age); // prints '43'
```

we can assign regular `Array`s to `ReadonlyArray`s.

```ts
const roArray: ReadonlyArray<string> = ["red", "green", "blue"];
```

Just as TypeScript provides a shorthand syntax for `Array<Type>` with `Type[]`, it also provides a shorthand syntax for `ReadonlyArray<Type>` with `readonly Type[]`.

```ts
function doStuff(values: readonly string[]) {
  // We can read from 'values'...
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);

  // ...but we can't mutate 'values'.
  values.push("hello!");
Property 'push' does not exist on type 'readonly string[]'.Property 'push' does not exist on type 'readonly string[]'.}
Try

```

One last thing to note is that unlike the `readonly` property modifier, assignability isn’t bidirectional between regular `Array`s and `ReadonlyArray`s.

```ts
let x: readonly string[] = [];
let y: string[] = [];

x = y;
y = x;
The type 'readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'.

```

### Tuple Types

If we try to index past the number of elements, we’ll get an error.

```ts
function doSomething(pair: [string, number]) {
  // ...

  const c = pair[2];
Tuple type '[string, number]' of length '2' has no element at index '2'.Tuple type '[string, number]' of length '2' has no element at index '2'.}

```

```ts
simple tuple types like these are equivalent to types which are versions of Arrays that declare properties for specific indexes, and that declare length with a numeric literal type.

interface StringNumberPair {
  // specialized properties
  length: 2;
  0: string;
  1: number;
 
  // Other 'Array<string | number>' members...
  slice(start?: number, end?: number): Array<string | number>;
}
Try
Another thing you may be interested in is that tuples can have optional properties by writing out a question mark (? after an element’s type). Optional tuple elements can only come at the end, and also affect the type of length.

type Either2dOr3d = [number, number, number?];
 
function setCoordinate(coord: Either2dOr3d) {
  const [x, y, z] = coord;

const z: number | undefined
 
  console.log(`Provided coordinates had ${coord.length} dimensions`);

(property) length: 2 | 3
}
Try
Tuples can also have rest elements, which have to be an array/tuple type.

type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
```

Tuples tend to be created and left un-modified in most code, so annotating types as `readonly` tuples when possible is a good default. This is also important given that array literals with `const` assertions will be inferred with `readonly` tuple types.

```ts
let point = [3, 4] as const;

function distanceFromOrigin([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2);
}

distanceFromOrigin(point);
Argument of type 'readonly [3, 4]' is not assignable to parameter of type '[number, number]'.
  The type 'readonly [3, 4]' is 'readonly' and cannot be assigned to the mutable type '[number, number]'.Argument of type 'readonly [3, 4]' is not assignable to parameter of type '[number, number]'.
  The type 'readonly [3, 4]' is 'readonly' and cannot be assigned to the mutable type '[number, number]'.Try

```

Here, `distanceFromOrigin` never modifies its elements, but expects a mutable tuple. Since `point`’s type was inferred as `readonly [3, 4]`, it won’t be compatible with `[number, number]` since that type can’t guarantee `point`’s elements won’t be mutated.

Just as with interface, putting the type parameter on the class itself lets us make sure all of the properties of the class are working with the same type.

As we cover in [our section on classes](https://www.typescriptlang.org/docs/handbook/2/classes.html), a class has two sides to its type: the static side and the instance side. Generic classes are only generic over their instance side rather than their static side, so when working with classes, static members can not use the class’s type parameter.

##

## Generic Constraints

If you remember from an earlier example, you may sometimes want to write a generic function that works on a set of types where you have *some* knowledge about what capabilities that set of types will have. In our `loggingIdentity` example, we wanted to be able to access the `.length` property of `arg`, but the compiler could not prove that every type had a `.length` property, so it warns us that we can’t make this assumption.

```ts
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
Property 'length' does not exist on type 'Type'.Property 'length' does not exist on type 'Type'.  return arg;
}
Try

```

Instead of working with any and all types, we’d like to constrain this function to work with any and all types that *also*  have the `.length` property. As long as the type has this member, we’ll allow it, but it’s required to have at least this member. To do so, we must list our requirement as a constraint on what `Type` can be.

To do so, we’ll create an interface that describes our constraint. Here, we’ll create an interface that has a single `.length` property and then we’ll use this interface and the `extends` keyword to denote our constraint:

```ts
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length); // Now we know it has a .length property, so no more error
  return arg;
}
```

## Using Type Parameters in Generic Constraints

You can declare a type parameter that is constrained by another type parameter. For example, here we’d like to get a property from an object given its name. We’d like to ensure that we’re not accidentally grabbing a property that does not exist on the `obj`, so we’ll place a constraint between the two types:

```ts
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a");
getProperty(x, "m");
Argument of type '"m"' is not assignable to parameter of type '"a" | "b" | "c" | "d"'.

```

## Using Class Types in Generics

When creating factories in TypeScript using generics, it is necessary to refer to class types by their constructor functions. For example,

```ts
function create<Type>(c: { new (): Type }): Type {
  return new c();
}
```

A more advanced example uses the prototype property to infer and constrain relationships between the constructor function and the instance side of class types.

```ts
class BeeKeeper {
  hasMask: boolean = true;
}

class ZooKeeper {
  nametag: string = "Mikle";
}

class Animal {
  numLegs: number = 4;
}

class Bee extends Animal {
  numLegs = 6;
  keeper: BeeKeeper = new BeeKeeper();
}

class Lion extends Animal {
  keeper: ZooKeeper = new ZooKeeper();
}

function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}

createInstance(Lion).keeper.nametag;
createInstance(Bee).keeper.hasMask;
```

This pattern is used to power the [mixins](https://www.typescriptlang.org/docs/handbook/mixins.html) design pattern.

##

## Generic Parameter Defaults

By declaring a default for a generic type parameter, you make it optional to specify the corresponding type argument. For example, a function which creates a new `HTMLElement`. Calling the function with no arguments generates a `HTMLDivElement`; calling the function with an element as the first argument generates an element of the argument’s type. You can optionally pass a list of children as well. Previously you would have to define the function as:

```ts
declare function create(): Container<HTMLDivElement, HTMLDivElement[]>;
declare function create<T extends HTMLElement>(element: T): Container<T, T[]>;
declare function create<T extends HTMLElement, U extends HTMLElement>(
  element: T,
  children: U[]
): Container<T, U[]>;
ts;
```

With generic parameter defaults we can reduce it to:

```ts
declare function create<
  T extends HTMLElement = HTMLDivElement,
  U extends HTMLElement[] = T[]
>(element?: T, children?: U): Container<T, U>;

const div = create();
const div: Container<HTMLDivElement, HTMLDivElement[]>;
const p = create(new HTMLParagraphElement());
const p: Container<HTMLParagraphElement, HTMLParagraphElement[]>;
```

## The `keyof` type operator

The `keyof` operator takes an object type and produces a string or numeric literal union of its keys. The following type `P` is the same type as `type P = "x" | "y"`:

```ts
type Point = { x: number; y: number };
type P = keyof Point;
type P = keyof Point;
```

If the type has a `string` or `number` index signature, `keyof` will return those types instead:

```ts
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;
type A = number;
type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
type M = string | number;
```

```ts
TypeScript intentionally limits the sorts of expressions you can use typeof on.

Specifically, it’s only legal to use typeof on identifiers (i.e. variable names) or their properties. This helps avoid the confusing trap of writing code you think is executing, but isn’t:

// Meant to use = ReturnType<typeof msgbox>
let shouldContinue: typeof msgbox("Are you sure you want to continue?");
',' expected.
```

# Indexed Access Types

We can use an *indexed access type* to look up a specific property on another type:

```ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"];
type Age = number;
```

The indexing type is itself a type, so we can use unions, `keyof`, or other types entirely:

```ts
type I1 = Person["age" | "name"];
type I1 = string | number;
type I2 = Person[keyof Person];
type I2 = string | number | boolean;
type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName];
type I3 = string | boolean;
```

```ts
Another example of indexing with an arbitrary type is using number to get the type of an array’s elements. We can combine this with typeof to conveniently capture the element type of an array literal:

const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];
 
type Person = typeof MyArray[number];

type Person = {
    name: string;
    age: number;
}
type Age = typeof MyArray[number]["age"];

type Age = number
// Or
type Age2 = Person["age"];

type Age2 = number
```

You can only use types when indexing, meaning you can’t use a `const` to make a variable reference:

```ts
const key = "age";
type Age = Person[key];
Type 'key' cannot be used as an index type.
'key' refers to a value, but is being used as a type here. Did you mean 'typeof key'?Type 'key' cannot be used as an index type.
'key' refers to a value, but is being used as a type here. Did you mean 'typeof key'?Try

```
