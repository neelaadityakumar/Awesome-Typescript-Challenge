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
Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.

```

There are a few interesting things to note in this example. We allowed TypeScript to *infer* the return type of `longest`. Return type inference also works on generic functions.

Because we constrained `Type` to `{ length: number }`, we were allowed to access the `.length` property of the `a` and `b` parameters. Without the type constraint, we wouldn’t be able to access those properties because the values might have been some other type without a length property.

The types of `longerArray` and `longerString` were inferred based on the arguments. Remember, generics are all about relating two or more values with the same type!

Finally, just as we’d like, the call to `longest(10, 100)` is rejected because the `number` type doesn’t have a `.length` property.

#### **5.Working with Constrained Values**

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


```

It might look like this function is OK - `Type` is constrained to `{ length: number }`, and the function either returns `Type` or a value matching that constraint. The problem is that the function promises to return the *same* kind of object as was passed in, not just *some* object matching the constraint. If this code were legal, you could write code that definitely wouldn’t work:

```ts
// 'arr' gets value { length: 6 }
const arr = minimumLength([1, 2, 3], 6);
// and crashes here because arrays have
// a 'slice' method, but not the returned object!
console.log(arr.slice(0));
```

#### **6. Function Overloads**

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

#### **7.`unknown`**

The `unknown` type represents *any* value. This is similar to the `any` type, but is safer because it’s not legal to do anything with an `unknown` value:

```ts
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b();
'a' is of type 'unknown'.'a' is of type 'unknown'.}


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

#### **8.`never`**

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

#### **9.`Function`**

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
A spread argument must either have a tuple type or be passed to a rest parameter.A spread argument must either have a tuple type or be passed to a rest parameter.

```

The best fix for this situation depends a bit on your code, but in general a `const` context is the most straightforward solution:

```ts
// Inferred as 2-length tuple
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```

#### **10.Return type `void`**

The `void` return type for functions can produce some unusual, but expected behavior.

Contextual typing with a return type of `void` does **not** force functions to **not** return something. Another way to say this is a contextual function type with a `void` return type (`type voidFunc = () => void`), when implemented, can return *any* other value, but it will be ignored.\

Note that there is currently no way to place type annotations within destructuring patterns. This is because the following syntax already means something different in JavaScript.

```ts
function draw({ shape: Shape, xPos: number = 100 /*...*/ }) {
  render(shape);
Cannot find name 'shape'. Did you mean 'Shape'?Cannot find name 'shape'. Did you mean 'Shape'?  render(xPos);
Cannot find name 'xPos'.Cannot find name 'xPos'.}


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


```

One last thing to note is that unlike the `readonly` property modifier, assignability isn’t bidirectional between regular `Array`s and `ReadonlyArray`s.

```ts
let x: readonly string[] = [];
let y: string[] = [];

x = y;
y = x;
The type 'readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'.

```

#### **11.Tuple Types**

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

Another thing you may be interested in is that tuples can have optional properties by writing out a question mark (? after an element’s type). Optional tuple elements can only come at the end, and also affect the type of length.

type Either2dOr3d = [number, number, number?];
 
function setCoordinate(coord: Either2dOr3d) {
  const [x, y, z] = coord;

const z: number | undefined
 
  console.log(`Provided coordinates had ${coord.length} dimensions`);

(property) length: 2 | 3
}

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
  The type 'readonly [3, 4]' is 'readonly' and cannot be assigned to the mutable type '[number, number]'.

```

Here, `distanceFromOrigin` never modifies its elements, but expects a mutable tuple. Since `point`’s type was inferred as `readonly [3, 4]`, it won’t be compatible with `[number, number]` since that type can’t guarantee `point`’s elements won’t be mutated.

Just as with interface, putting the type parameter on the class itself lets us make sure all of the properties of the class are working with the same type.

As we cover in [our section on classes](https://www.typescriptlang.org/docs/handbook/2/classes.html), a class has two sides to its type: the static side and the instance side. Generic classes are only generic over their instance side rather than their static side, so when working with classes, static members can not use the class’s type parameter.

#### **12 Generic Constraints**

If you remember from an earlier example, you may sometimes want to write a generic function that works on a set of types where you have *some* knowledge about what capabilities that set of types will have. In our `loggingIdentity` example, we wanted to be able to access the `.length` property of `arg`, but the compiler could not prove that every type had a `.length` property, so it warns us that we can’t make this assumption.

```ts
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
Property 'length' does not exist on type 'Type'.Property 'length' does not exist on type 'Type'.  return arg;
}


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

#### **13 Using Type Parameters in Generic Constraints**

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

#### **14 Using Class Types in Generics**

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

#### **15 Generic Parameter Defaults**

By declaring a default for a generic type parameter, you make it optional to specify the corresponding type argument. For example, a function which creates a new `HTMLElement`. Calling the function with no arguments generates a `HTMLDivElement`; calling the function with an element as the first argument generates an element of the argument’s type. You can optionally pass a list of children as well. Previously you would have to define the function as:

```ts
declare function create(): Container<HTMLDivElement, HTMLDivElement[]>;
declare function create<T extends HTMLElement>(element: T): Container<T, T[]>;
declare function create<T extends HTMLElement, U extends HTMLElement>(
  element: T,
  children: U[]
): Container<T, U[]>;
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

#### **16 The `keyof` type operator**

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
'key' refers to a value, but is being used as a type here. Did you mean 'typeof key'?

```

However, you can use a type alias for a similar style of refactor:

```ts
type key = "age";
type Age = Person[key];
```

we could also write a type called `Flatten` that flattens array types to their element types, but leaves them alone otherwise:

```ts
type Flatten<T> = T extends any[] ? T[number] : T;

// Extracts out the element type.
type Str = Flatten<string[]>;
type Str = string;
// Leaves the type alone.
type Num = Flatten<number>;
type Num = number;
```

When `Flatten` is given an array type, it uses an indexed access with `number` to fetch out `string[]`’s element type. Otherwise, it just returns the type it was given.

#### **17.Inferring Within Conditional Types**

We just found ourselves using conditional types to apply constraints and then extract out types. This ends up being such a common operation that conditional types make it easier.

Conditional types provide us with a way to infer from types we compare against in the true branch using the `infer` keyword. For example, we could have inferred the element type in `Flatten` instead of fetching it out “manually” with an indexed access type:

```ts
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
```

```ts
When you don’t want to repeat yourself, sometimes a type needs to be based on another type.

Mapped types build on the syntax for index signatures, which are used to declare the types of properties which have not been declared ahead of time:

type OnlyBoolsAndHorses = {
  [key: string]: boolean | Horse;
};
 
const conforms: OnlyBoolsAndHorses = {
  del: true,
  rodney: false,
};

A mapped type is a generic type which uses a union of PropertyKeys (frequently created via a keyof) to iterate through keys to create a type:

type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};

In this example, OptionsFlags will take all the properties from the type Type and change their values to be a boolean.

type Features = {
  darkMode: () => void;
  newUserProfile: () => void;
};
 
type FeatureOptions = OptionsFlags<Features>;

type FeatureOptions = {
    darkMode: boolean;
    newUserProfile: boolean;
}
```

```ts
Mapping Modifiers
There are two additional modifiers which can be applied during mapping: readonly and ? which affect mutability and optionality respectively.

You can remove or add these modifiers by prefixing with - or +. If you don’t add a prefix, then + is assumed.

// Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};
 
type LockedAccount = {
  readonly id: string;
  readonly name: string;
};
 
type UnlockedAccount = CreateMutable<LockedAccount>;

type UnlockedAccount = {
    id: string;
    name: string;
}

// Removes 'optional' attributes from a type's properties
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};
 
type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};
 
type User = Concrete<MaybeUser>;

type User = {
    id: string;
    name: string;
    age: number;
}

```

#### **18 Key Remapping via `as`**

In TypeScript 4.1 and onwards, you can re-map keys in mapped types with an `as` clause in a mapped type:

```ts
type MappedTypeWithNewProperties<Type> = {
  [Properties in keyof Type as NewKeyType]: Type[Properties];
};
```

You can leverage features like [template literal types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html) to create new property names from prior ones:

```ts
type Getters<Type> = {
  [Property in keyof Type as `get${Capitalize<
    string & Property
  >}`]: () => Type[Property];
};

interface Person {
  name: string;
  age: number;
  location: string;
}

type LazyPerson = Getters<Person>;
type LazyPerson = {
  getName: () => string;
  getAge: () => number;
  getLocation: () => string;
};
```

You can filter out keys by producing `never` via a conditional type:

```ts
// Remove the 'kind' property
type RemoveKindField<Type> = {
  [Property in keyof Type as Exclude<Property, "kind">]: Type[Property];
};

interface Circle {
  kind: "circle";
  radius: number;
}

type KindlessCircle = RemoveKindField<Circle>;
type KindlessCircle = {
  radius: number;
};
```

You can map over arbitrary unions, not just unions of `string | number | symbol`, but unions of any type:

```ts
type EventConfig<Events extends { kind: string }> = {
  [E in Events as E["kind"]]: (event: E) => void;
};

type SquareEvent = { kind: "square"; x: number; y: number };
type CircleEvent = { kind: "circle"; radius: number };

type Config = EventConfig<SquareEvent | CircleEvent>;
type Config = {
  square: (event: SquareEvent) => void;
  circle: (event: CircleEvent) => void;
};
```

#### **19.`readonly`**

Fields may be prefixed with the `readonly` modifier. This prevents assignments to the field outside of the constructor.

```ts
class Greeter {
  readonly name: string = "world";

  constructor(otherName?: string) {
    if (otherName !== undefined) {
      this.name = otherName;
    }
  }

  err() {
    this.name = "not ok";
Cannot assign to 'name' because it is a read-only property.Cannot assign to 'name' because it is a read-only property.  }

```

#### **20.Constructors**

> Background Reading:
>
> [Constructor (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor)

Class constructors are very similar to functions. You can add parameters with type annotations, default values, and overloads:

```ts
class Point {
  x: number;
  y: number;

  // Normal signature with defaults
  constructor(x = 0, y = 0) {
    this.x = x;
    this.y = y;
  }
}
```

```ts
class Point {
  x: number = 0;
  y: number = 0;

  // Constructor overloads
  constructor(x: number, y: number);
  constructor(xy: string);
  constructor(x: string | number, y: number = 0) {
    // Code logic here
  }
}
```

There are just a few differences between class constructor signatures and function signatures:

- Constructors can’t have type parameters - these belong on the outer class declaration, which we’ll learn about later
- Constructors can’t have return type annotations - the class instance type is always what’s returned

#### **21.Super Calls**

Just as in JavaScript, if you have a base class, you’ll need to call `super();` in your constructor body before using any `this.` members:

```ts
class Base {
  k = 4;
}

class Derived extends Base {
  constructor() {
    // Prints a wrong value in ES5; throws exception in ES6
    console.log(this.k);
'super' must be called before accessing 'this' in the constructor of a derived class.'super' must be called before accessing 'this' in the constructor of a derived class.    super();
  }
}


```

Forgetting to call `super` is an easy mistake to make in JavaScript, but TypeScript will tell you when it’s necessary.

#### **22 TypeScript has some special inference rules for accessors:**

- If `get` exists but no `set`, the property is automatically `readonly`
- If the type of the setter parameter is not specified, it is inferred from the return type of the getter

Since [TypeScript 4.3](https://devblogs.microsoft.com/typescript/announcing-typescript-4-3/), it is possible to have accessors with different types for getting and setting.

```ts
class Thing {
  _size = 0;

  get size(): number {
    return this._size;
  }

  set size(value: string | number | boolean) {
    let num = Number(value);

    // Don't allow NaN, Infinity, etc

    if (!Number.isFinite(num)) {
      this._size = 0;
      return;
    }

    this._size = num;
  }
}
```

#### **23 Class Heritage**

Like other languages with object-oriented features, classes in JavaScript can inherit from base classes.

#### **24.`implements` Clauses**

You can use an `implements` clause to check that a class satisfies a particular `interface`. An error will be issued if a class fails to correctly implement it:

```ts
interface Pingable {
  ping(): void;
}

class Sonar implements Pingable {
  ping() {
    console.log("ping!");
  }
}

class Ball implements Pingable {
Class 'Ball' incorrectly implements interface 'Pingable'.
  Property 'ping' is missing in type 'Ball' but required in type 'Pingable'.Class 'Ball' incorrectly implements interface 'Pingable'.
  Property 'ping' is missing in type 'Ball' but required in type 'Pingable'.  pong() {
    console.log("pong!");
  }
}


```

Classes may also implement multiple interfaces, e.g. `class C implements A, B {`.

#### **25.Cautions**

It’s important to understand that an `implements` clause is only a check that the class can be treated as the interface type. It doesn’t change the type of the class or its methods *at all*. A common source of error is to assume that an `implements` clause will change the class type - it doesn’t!

```ts
interface Checkable {
  check(name: string): boolean;
}

class NameChecker implements Checkable {
  check(s) {
Parameter 's' implicitly has an 'any' type.Parameter 's' implicitly has an 'any' type.    // Notice no error here
    return s.toLowerCase() === "ok";
                 any  }
}


```

In this example, we perhaps expected that `s`’s type would be influenced by the `name: string` parameter of `check`. It is not - `implements` clauses don’t change how the class body is checked or its type inferred.

Similarly, implementing an interface with an optional property doesn’t create that property:

```ts
interface A {
  x: number;
  y?: number;
}
class C implements A {
  x = 0;
}
const c = new C();
c.y = 10;
Property 'y' does not exist on type 'C'.Property 'y' does not exist on type 'C'.

```

#### **26.`extends` Clauses**

> Background Reading:
>
> [extends keyword (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends)

Classes may `extend` from a base class. A derived class has all the properties and methods of its base class, and can also define additional members.

```ts
class Animal {
  move() {
    console.log("Moving along!");
  }
}

class Dog extends Animal {
  woof(times: number) {
    for (let i = 0; i < times; i++) {
      console.log("woof!");
    }
  }
}

const d = new Dog();
// Base class method
d.move();
// Derived class method
d.woof(3);
```

#### **27.Overriding Methods**

> Background Reading:
>
> [super keyword (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super)

A derived class can also override a base class field or property. You can use the `super.` syntax to access base class methods. Note that because JavaScript classes are a simple lookup object, there is no notion of a “super field”.

TypeScript enforces that a derived class is always a subtype of its base class.

For example, here’s a legal way to override a method:

```ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) {
      super.greet();
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}

const d = new Derived();
d.greet();
d.greet("reader");
```

It’s important that a derived class follow its base class contract. Remember that it’s very common (and always legal!) to refer to a derived class instance through a base class reference:

```ts
protected
protected members are only visible to subclasses of the class they’re declared in.

class Greeter {
  public greet() {
    console.log("Hello, " + this.getName());
  }
  protected getName() {
    return "hi";
  }
}
```

```ts
class SpecialGreeter extends Greeter {
public howdy() {
// OK to access protected member here
console.log("Howdy, " + this.getName());
}
}
const g = new SpecialGreeter();
g.greet(); // OK
g.getName();
Property 'getName' is protected and only accessible within class 'Greeter' and its subclasses.
```

// No modifier, so default is 'public'

```ts
private
private is like protected, but doesn’t allow access to the member even from subclasses:

class Base {
  private x = 0;
}
const b = new Base();
// Can't access from outside the class
console.log(b.x);
Property 'x' is private and only accessible within class 'Base'.
```

#### **28.Cross-instance `private` access**

Different OOP languages disagree about whether different instances of the same class may access each others’ `private` members. While languages like Java, C#, C++, Swift, and PHP allow this, Ruby does not.

TypeScript does allow cross-instance `private` access:

```ts
class A {
  private x = 10;

  public sameAs(other: A) {
    // No error
    return other.x === this.x;
  }
}
```

#### **29.Caveats**

This means that JavaScript runtime constructs like `in` or simple property lookup can still access a `private` or `protected` member:

```ts
class MySafe {
  private secretKey = 12345;
}
```

```ts
// In a JavaScript file...
const s = new MySafe();
// Will print 12345
console.log(s.secretKey);
```

`private` also allows access using bracket notation during type checking. This makes `private`-declared fields potentially easier to access for things like unit tests, with the drawback that these fields are *soft private* and don’t strictly enforce privacy.

```ts
class MySafe {
  private secretKey = 12345;
}

const s = new MySafe();

// Not allowed during type checking
console.log(s.secretKey);
Property 'secretKey' is private and only accessible within class 'MySafe'.Property 'secretKey' is private and only accessible within class 'MySafe'.
// OK
console.log(s["secretKey"]);


```

```ts
Unlike TypeScripts’s private, JavaScript’s private fields (#) remain private after compilation and do not provide the previously mentioned escape hatches like bracket notation access, making them hard private.

class Dog {
  #barkAmount = 0;
  personality = "happy";
 
  constructor() {}
}

"use strict";
class Dog {
    #barkAmount = 0;
    personality = "happy";
    constructor() { }
}
 

When compiling to ES2021 or less, TypeScript will use WeakMaps in place of #.

"use strict";
var _Dog_barkAmount;
class Dog {
    constructor() {
        _Dog_barkAmount.set(this, 0);
        this.personality = "happy";
    }
}
_Dog_barkAmount = new WeakMap();
 

If you need to protect values in your class from malicious actors, you should use mechanisms that offer hard runtime privacy, such as closures, WeakMaps, or private fields. Note that these added privacy checks during runtime could affect performance.

```

#### **30 Static Members**

> Background Reading:
>
> [Static Members (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static)

Classes may have `static` members. These members aren’t associated with a particular instance of the class. They can be accessed through the class constructor object itself:

```ts
class MyClass {
  static x = 0;
  static printX() {
    console.log(MyClass.x);
  }
}
console.log(MyClass.x);
MyClass.printX();
```

Static members can also use the same `public`, `protected`, and `private` visibility modifiers:

```ts
class MyClass {
  private static x = 0;
}
console.log(MyClass.x);
Property 'x' is private and only accessible within class 'MyClass'.

```

Why No Static Classes?
TypeScript (and JavaScript) don’t have a construct called static class the same way as, for example, C# does.

Those constructs only exist because those languages force all data and functions to be inside a class; because that restriction doesn’t exist in TypeScript, there’s no need for them. A class with only a single instance is typically just represented as a normal object in JavaScript/TypeScript.

For example, we don’t need a “static class” syntax in TypeScript because a regular object (or even top-level function) will do the job just as well:

```ts
// Unnecessary "static" class
class MyStaticClass {
  static doSomething() {}
}
// Preferred (alternative 1)
function doSomething() {}
// Preferred (alternative 2)
const MyHelperObject = {
  dosomething() {},
};
```

static Blocks in Classes
Static blocks allow you to write a sequence of statements with their own scope that can access private fields within the containing class. This means that we can write initialization code with all the capabilities of writing statements, no leakage of variables, and full access to our class’s internals.

```ts
class Foo {
  static #count = 0;
  get count() {
    return Foo.#count;
  }
  static {
    try {
      const lastInstances = loadLastInstances();
      Foo.#count += lastInstances.length;
    } catch {}
  }
}
```

Generic Classes
Classes, much like interfaces, can be generic. When a generic class is instantiated with new, its type parameters are inferred the same way as in a function call:

```ts
class Box<Type> {
  contents: Type;
  constructor(value: Type) {
    this.contents = value;
  }
}
const b = new Box("hello!");

const b: Box<string>;
```

Classes can use generic constraints and defaults the same way as interfaces.

Type Parameters in Static Members
This code isn’t legal, and it may not be obvious why:

```ts

class Box<Type> {
  static defaultValue: Type;
Static members cannot reference class type parameters.
Static members cannot reference class type parameters.
}
```

Remember that types are always fully erased! At runtime, there’s only one Box.defaultValue property slot. This means that setting Box<string>.defaultValue (if that were possible) would also change Box<number>.defaultValue - not good. The static members of a generic class can never refer to the class’s type parameters.

#### **31 Parameter Properties**

TypeScript offers special syntax for turning a constructor parameter into a class property with the same name and value. These are called *parameter properties* and are created by prefixing a constructor argument with one of the visibility modifiers `public`, `private`, `protected`, or `readonly`. The resulting field gets those modifier(s):

```ts
class Params {
  constructor(
    public readonly x: number,
    protected y: number,
    private z: number
  ) {
    // No body necessary
  }
}
const a = new Params(1, 2, 3);
console.log(a.x);
             (property) Params.x: numberconsole.log(a.z);
Property 'z' is private and only accessible within class 'Params'.

```

#### **32 Class Expressions**

> Background Reading:
>
> [Class expressions (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/class)

Class expressions are very similar to class declarations. The only real difference is that class expressions don’t need a name, though we can refer to them via whatever identifier they ended up bound to:

```ts
const someClass = class<Type> {
  content: Type;
  constructor(value: Type) {
    this.content = value;
  }
};

const m = new someClass("Hello, world");
const m: someClass<string>;
```

#### **33 Constructor Signatures**

JavaScript classes are instantiated with the `new` operator. Given the type of a class itself, the [InstanceType](https://www.typescriptlang.org/docs/handbook/utility-types.html#instancetypetype) utility type models this operation.

```ts
class Point {
  createdAt: number;
  x: number;
  y: number;
  constructor(x: number, y: number) {
    this.createdAt = Date.now();
    this.x = x;
    this.y = y;
  }
}
type PointInstance = InstanceType<typeof Point>;

function moveRight(point: PointInstance) {
  point.x += 5;
}

const point = new Point(3, 4);
moveRight(point);
point.x; // => 8
```

#### **34 `abstract` Classes and Members**

Classes, methods, and fields in TypeScript may be *abstract*.

An *abstract method* or *abstract field* is one that hasn’t had an implementation provided. These members must exist inside an *abstract class*, which cannot be directly instantiated.

The role of abstract classes is to serve as a base class for subclasses which do implement all the abstract members. When a class doesn’t have any abstract members, it is said to be *concrete*.

Let’s look at an example:

```ts
abstract class Base {
  abstract getName(): string;

  printName() {
    console.log("Hello, " + this.getName());
  }
}

const b = new Base();
Cannot create an instance of an abstract class.Cannot create an instance of an abstract class.

```

We can’t instantiate `Base` with `new` because it’s abstract. Instead, we need to make a derived class and implement the abstract members:

```ts
class Derived extends Base {
  getName() {
    return "world";
  }
}

const d = new Derived();
d.printName();
```

Notice that if we forget to implement the base class’s abstract members, we’ll get an error:

```ts
class Derived extends Base {
Non-abstract class 'Derived' does not implement inherited abstract member getName from class 'Base'.Non-abstract class 'Derived' does not implement inherited abstract member getName from class 'Base'.  // forgot to do anything
}


```

Relationships Between Classes
In most cases, classes in TypeScript are compared structurally, the same as other types.

For example, these two classes can be used in place of each other because they’re identical:

```ts
class Point1 {
  x = 0;
  y = 0;
}
class Point2 {
  x = 0;
  y = 0;
}
// OK
const p: Point1 = new Point2();

//Similarly, subtype relationships between classes exist even if there’s no explicit inheritance:

class Person {
  name: string;
  age: number;
}
class Employee {
  name: string;
  age: number;
  salary: number;
}
// OK
const p: Person = new Employee();
```

This sounds straightforward, but there are a few cases that seem stranger than others.

Empty classes have no members. In a structural type system, a type with no members is generally a supertype of anything else. So if you write an empty class (don’t!), anything can be used in place of it:

```ts
class Empty {}
function fn(x: Empty) {
  // can't do anything with 'x', so I won't
}
// All OK!
fn(window);
fn({});
fn(fn);
```
