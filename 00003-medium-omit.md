Implement the built-in `Omit<T, K>` generic without using it.

Constructs a type by picking all properties from `T` and then removing `K`

For example

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = MyOmit<Todo, "description" | "title">;

const todo: TodoPreview = {
  completed: false,
};
```

Solution:

```ts
1;

type MyOmit<T, K extends keyof T> = {
  [P in Exclude<keyof T, K>]: T[P];
};

2;
type MyOmit<T, K extends keyof T> = {
  [P in keyof T as P extends K ? never : P]: T[P];
};

3;
type MyOmit<T, K extends keyof T> = {
  [key in keyof T as Exclude<key, K>]: T[key];
};

4;
type MyOmit<T extends Object, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
```
