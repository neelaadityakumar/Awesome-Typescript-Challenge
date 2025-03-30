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
