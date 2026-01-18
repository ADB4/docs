# Module 1: TypeScript Foundations — Question Bank

**Covers Weeks 1–4:** TypeScript fundamentals, advanced TypeScript, TypeScript in React, TypeScript project configuration.

**Scoring note (EECS 381 style):** For multiple choice and true/false, wrong answers are subtracted from right answers. Do not guess. Leave blank if unsure.

---

## Part A: Multiple Choice (24 Questions)

Choose the single best answer for each question.

---

**MC-1.** What is the inferred type of `x` in the following code?

```ts
const x = "hello";
```

(a) `string`
(b) `"hello"`
(c) `any`
(d) `String`

---

**MC-2.** Which of the following is a key difference between `interface` and `type` in TypeScript?

(a) Interfaces can describe object shapes; type aliases cannot.
(b) Type aliases can represent union types; interfaces cannot.
(c) Interfaces are erased at compile time; type aliases are preserved at runtime.
(d) Type aliases support generic parameters; interfaces do not.

---

**MC-3.** Given the following code, what error does TypeScript report?

```ts
function greet(name: string) {
  console.log("Hello, " + name);
}
greet(42);
```

(a) `42` is not assignable to parameter of type `string`
(b) Cannot find name `greet`
(c) Function `greet` expects 0 arguments but got 1
(d) No error; TypeScript will coerce `42` to `"42"`

---

**MC-4.** What does TypeScript's `typeof` type operator do when used in a type position?

```ts
const config = { debug: true, level: 5 };
type Config = typeof config;
```

(a) Returns `"object"` as a string literal type
(b) Extracts the type of a value so it can be used as a type annotation
(c) Checks the runtime type and narrows within a conditional block
(d) Creates a branded type from the variable

---

**MC-5.** Which utility type produces a type where all properties of `T` are optional?

(a) `Pick<T, K>`
(b) `Required<T>`
(c) `Partial<T>`
(d) `Omit<T, K>`

---

**MC-6.** What does the following type evaluate to?

```ts
type Result = Omit<{ a: number; b: string; c: boolean }, "b">;
```

(a) `{ a: number; c: boolean }`
(b) `{ b: string }`
(c) `{ a: number; b: string; c: boolean }`
(d) `{ a: number }`

---

**MC-7.** What is a discriminated union in TypeScript?

(a) A union type where each member has a common literal property that TypeScript can use to narrow the type
(b) A union type that excludes `null` and `undefined`
(c) A special union type created by the `discriminate` keyword
(d) A union type where all members must share the exact same properties

---

**MC-8.** What is the purpose of a type guard function?

(a) To prevent a variable from ever being reassigned
(b) To tell TypeScript's type system that a value is a specific type within a conditional block
(c) To enforce that a class implements an interface
(d) To catch runtime errors before they reach production

---

**MC-9.** What does `satisfies` do differently from a type annotation?

```ts
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
} satisfies Record<string, string | number[]>;
```

(a) It widens the type of `palette` to `Record<string, string | number[]>`
(b) It validates that `palette` matches the target type while preserving the narrower inferred type
(c) It converts `palette` to a readonly type
(d) It is identical to using `as const`

---

**MC-10.** What does `as const` do when applied to a value?

```ts
const directions = ["north", "south", "east", "west"] as const;
```

(a) Makes the array mutable but locks the element types to `string`
(b) Infers the narrowest possible literal types and makes the value deeply readonly
(c) Converts the array to a tuple of type `[string, string, string, string]`
(d) Creates a new const variable; it is equivalent to another `const` declaration

---

**MC-11.** In a generic function, what does a constraint accomplish?

```ts
function getLength<T extends { length: number }>(item: T): number {
  return item.length;
}
```

(a) It restricts `T` to only arrays
(b) It requires that any type argument for `T` has at least a `length` property of type `number`
(c) It sets a default value for `T`
(d) It makes `T` optional

---

**MC-12.** Which of the following correctly types a function that accepts either a `string` or `number` and returns a `boolean`?

(a) `function check(val: string & number): boolean`
(b) `function check(val: string | number): boolean`
(c) `function check(val: string, number): boolean`
(d) `function check<T = string | number>(val: T): boolean`

---

**MC-13.** What is the type of `value` inside the `if` block?

```ts
function process(value: string | number) {
  if (typeof value === "string") {
    // what is the type of value here?
  }
}
```

(a) `string | number`
(b) `string`
(c) `number`
(d) `unknown`

---

**MC-14.** What is the purpose of `.d.ts` files?

(a) They contain compiled JavaScript output from TypeScript
(b) They contain only type information and produce no JavaScript output
(c) They are configuration files for the TypeScript compiler
(d) They are debug files generated during development builds

---

**MC-15.** If a library does not bundle its own types and has no `@types` package on DefinitelyTyped, what is the recommended approach?

(a) Use `// @ts-ignore` on every import
(b) Disable `strict` mode in tsconfig
(c) Write your own `.d.ts` declaration file for the module
(d) Switch to a different programming language

---

**MC-16.** Why does the syllabus prefer plain function declarations over `React.FC` for typing components?

(a) `React.FC` does not work with TypeScript
(b) `React.FC` implicitly includes `children` in props (in older typings), obscures the return type, and prevents generic components
(c) Plain function declarations are faster at runtime
(d) `React.FC` is only available in class components

---

**MC-17.** How should you type the props of a React component according to the syllabus?

```ts
// Which pattern is preferred?
```

(a) `const Button: React.FC<{ label: string }> = ({ label }) => ...`
(b) `function Button({ label }: { label: string }) { ... }`
(c) `class Button extends React.Component<{ label: string }> { ... }`
(d) `const Button = (props: any) => ...`

---

**MC-18.** What does `useState` infer when called like this?

```ts
const [count, setCount] = useState(0);
```

(a) `count` is typed as `any`
(b) `count` is typed as `number` (inferred from the initial value)
(c) `count` is typed as `number | undefined`
(d) `count` is typed as `0` (the literal type)

---

**MC-19.** When should you provide an explicit type argument to `useState`?

(a) Always, for every `useState` call
(b) Never; TypeScript always infers correctly
(c) When the initial value doesn't represent the full range of possible state (e.g., `useState<User | null>(null)`)
(d) Only when using `useState` inside class components

---

**MC-20.** What does `noUncheckedIndexedAccess` do in tsconfig?

(a) Prevents using bracket notation on objects entirely
(b) Adds `undefined` to the type of values accessed by index, forcing you to handle the possibly-missing case
(c) Disables array destructuring
(d) Requires all object keys to be declared in an interface before use

---

**MC-21.** What is a branded type (also called a nominal or opaque type)?

(a) A type alias that uses a phantom property to make structurally identical types incompatible
(b) A type that is enforced at runtime using Symbol
(c) A type that can only be created by a specific class constructor
(d) A type that is automatically generated by the TypeScript compiler

---

**MC-22.** In a tsconfig with `"strict": true`, which of the following is NOT enabled?

(a) `strictNullChecks`
(b) `noImplicitAny`
(c) `noUncheckedIndexedAccess`
(d) `strictFunctionTypes`

---

**MC-23.** What is the relationship between `import type` and regular `import` in TypeScript?

(a) They are identical; `import type` is purely cosmetic
(b) `import type` imports only type information and is completely erased in the JavaScript output, ensuring no runtime side effects
(c) `import type` imports values that can only be used in type annotations but still appear in the JS output
(d) `import type` is deprecated in favor of inline `type` imports

---

**MC-24.** How does Vite handle TypeScript during development?

(a) It runs `tsc` on every file change for both type checking and transpilation
(b) It uses esbuild to strip types for fast transpilation but does NOT perform type checking
(c) It uses Babel exclusively for TypeScript compilation
(d) It compiles TypeScript to WebAssembly for faster execution

---

## Part B: True / False (32 Questions)

Circle T or F. Wrong answers are subtracted from right answers — do not guess.

---

**TF-1.** TypeScript's type system is structural: two types are compatible if their structures match, regardless of their declared names.

**TF-2.** The `any` type disables type checking for the value it is applied to.

**TF-3.** The `unknown` type allows you to perform arbitrary operations on a value without narrowing it first.

**TF-4.** Type inference means TypeScript can often determine a variable's type without an explicit annotation, based on the assigned value.

**TF-5.** In TypeScript, `string` and `String` are interchangeable and refer to the same type.

**TF-6.** An intersection type (`A & B`) produces a type that has all properties from both `A` and `B`.

**TF-7.** A union type (`A | B`) means a value must satisfy both `A` and `B` simultaneously.

**TF-8.** When you use `let x = "hello"`, TypeScript infers the type of `x` as `string`, not as the literal `"hello"`.

**TF-9.** The `readonly` modifier on an array type (`readonly string[]`) prevents the array from being reassigned and also prevents mutations like `push`.

**TF-10.** In TypeScript, an interface can extend another interface using the `extends` keyword.

**TF-11.** You can add new properties to an existing interface by declaring it again with the same name (declaration merging). You cannot do this with `type` aliases.

**TF-12.** The `never` type represents a value that should never occur, such as the return type of a function that always throws.

**TF-13.** TypeScript generics are resolved at runtime, and the generic type parameter is available for reflection during execution.

**TF-14.** `Partial<T>` makes all properties of `T` required.

**TF-15.** `Record<string, number>` describes an object type where every key is a `string` and every value is a `number`.

**TF-16.** `Pick<T, "a" | "b">` creates a type with only the `a` and `b` properties from `T`.

**TF-17.** A mapped type iterates over the keys of a type and produces a new type with transformed properties.

**TF-18.** The `satisfies` operator changes the type of the variable it is applied to, widening it to the target type.

**TF-19.** A type guard using `in` (e.g., `if ("name" in obj)`) can narrow a union type based on whether a property exists.

**TF-20.** Conditional types in TypeScript (`T extends U ? X : Y`) are evaluated at runtime to determine which branch to take.

**TF-21.** The `ReturnType<T>` utility type extracts the return type of a function type `T`.

**TF-22.** A custom hook in React must start with the prefix `use` for the Rules of Hooks linter to recognize it.

**TF-23.** When typing a custom hook's return value, you should use `as const` on the return array to preserve the tuple type rather than letting TypeScript widen it to a union array.

**TF-24.** `React.FC` is the preferred way to type functional components in 2026, as recommended by the React team.

**TF-25.** The `children` prop in React should be typed as `React.ReactNode` when you want to accept any renderable content.

**TF-26.** The `tsconfig.json` `strict` flag is a shorthand that enables a collection of stricter type-checking options.

**TF-27.** Setting `"moduleResolution": "bundler"` in tsconfig is appropriate for projects using Vite, as it aligns with how bundlers resolve modules.

**TF-28.** `as const` assertions make values mutable but lock their types to the widest possible inference.

**TF-29.** Template literal types allow you to construct string types by combining literal types with interpolation syntax.

**TF-30.** Branded types provide runtime type safety by adding a hidden property to the JavaScript output.

**TF-31.** `.d.ts` files produce JavaScript output when compiled.

**TF-32.** TypeScript automatically discovers type definitions in `node_modules/@types` without any additional tsconfig configuration.

---

## Answer Key

### Part A: Multiple Choice

| # | Answer | Explanation |
|---|---|---|
| MC-1 | **(b)** `"hello"` | `const` declarations infer literal types. If this were `let x = "hello"`, the type would be `string`. |
| MC-2 | **(b)** | Type aliases can represent unions, intersections, primitives, and tuples. Interfaces are limited to object shapes (though they can extend other interfaces). Both support generics. Both are erased at compile time. |
| MC-3 | **(a)** | TypeScript reports that `number` is not assignable to `string`. This is a compile-time type error. |
| MC-4 | **(b)** | The `typeof` operator in a type position extracts the TypeScript type from a value-level variable. This is distinct from JavaScript's runtime `typeof`. |
| MC-5 | **(c)** `Partial<T>` | `Partial<T>` makes every property in `T` optional. `Required<T>` does the opposite. |
| MC-6 | **(a)** `{ a: number; c: boolean }` | `Omit` removes the specified keys. Here it removes `"b"`, leaving `a` and `c`. |
| MC-7 | **(a)** | A discriminated union uses a common literal property (the discriminant) that TypeScript can use in `switch` or `if` statements to narrow to a specific member. |
| MC-8 | **(b)** | A type guard (either built-in like `typeof`/`instanceof` or a custom `is` predicate) tells TypeScript to narrow the type within a conditional scope. |
| MC-9 | **(b)** | `satisfies` validates against a type without widening. A type annotation (`const x: T = ...`) would widen to `T`; `satisfies` preserves the narrower inferred type while checking compatibility. |
| MC-10 | **(b)** | `as const` infers the narrowest possible types (literal values, readonly tuples/arrays) and makes the entire structure deeply readonly. |
| MC-11 | **(b)** | `T extends { length: number }` constrains `T` so that any type argument must have at least a `length` property of type `number`. Strings, arrays, and custom objects with `length` all qualify. |
| MC-12 | **(b)** | `string | number` is the union type for accepting either. `string & number` is an intersection (effectively `never` for primitives). |
| MC-13 | **(b)** `string` | The `typeof` check narrows the union. Inside the `if` block, TypeScript knows `value` must be `string`. |
| MC-14 | **(b)** | `.d.ts` files contain only type declarations and produce no `.js` output. They exist solely for type checking. |
| MC-15 | **(c)** | Writing your own `.d.ts` file is the correct approach. You can also use `declare module "name"` as a quick escape hatch. |
| MC-16 | **(b)** | Older `React.FC` typings implicitly included `children`, its return type annotation was redundant, and it didn't support generic components cleanly. Plain function declarations avoid these issues. |
| MC-17 | **(b)** | A plain function declaration with destructured, typed props is the preferred pattern. |
| MC-18 | **(b)** `number` | TypeScript infers the state type from the initial value. `0` is a `number`, so `count` is `number`. |
| MC-19 | **(c)** | Provide an explicit type argument when the initial value is a subset of the possible states, such as starting with `null` for a value that will later be a `User`. |
| MC-20 | **(b)** | This flag makes indexed access types include `undefined`, forcing explicit handling of potentially missing values. It is NOT enabled by `strict`. |
| MC-21 | **(a)** | Branded types use a phantom/tag property (never actually set at runtime) to create nominal distinctions between structurally identical types like `UserId` vs. `PostId`. |
| MC-22 | **(c)** `noUncheckedIndexedAccess` | This flag is not part of the `strict` family. It must be enabled separately. |
| MC-23 | **(b)** | `import type` is fully erased during compilation. It guarantees no runtime module side effects and signals to bundlers that the import can be safely removed. |
| MC-24 | **(b)** | Vite uses esbuild to strip type annotations for speed during dev. Type checking must be done separately via `tsc --noEmit` or an IDE. |

### Part B: True / False

| # | Answer | Explanation |
|---|---|---|
| TF-1 | **T** | TypeScript uses structural typing (duck typing). Two types with the same shape are compatible regardless of name. |
| TF-2 | **T** | `any` effectively opts out of type checking. It accepts any operation without complaint. |
| TF-3 | **F** | `unknown` requires narrowing before any operations can be performed. That is the key difference from `any`. |
| TF-4 | **T** | Type inference is one of TypeScript's core features. It reduces the need for verbose annotations. |
| TF-5 | **F** | `string` is the primitive type. `String` is the wrapper object type. You should almost always use lowercase `string`. |
| TF-6 | **T** | An intersection combines all properties from both types into a single type. |
| TF-7 | **F** | A union means the value can be one of the types, not both. You must narrow before accessing type-specific properties. |
| TF-8 | **T** | `let` allows reassignment, so TypeScript widens to `string`. `const` would infer the literal `"hello"`. |
| TF-9 | **T** | `readonly` on an array type prevents both reassignment and mutating methods like `push`, `pop`, and `splice`. |
| TF-10 | **T** | Interfaces support `extends` for inheritance from one or more other interfaces. |
| TF-11 | **T** | Declaration merging is a feature unique to interfaces. Declaring the same `type` alias twice is a compile error. |
| TF-12 | **T** | `never` represents the bottom type: functions that never return, exhausted union branches, and impossible states. |
| TF-13 | **F** | Generics are fully erased at compile time. There is no generic type information at runtime. |
| TF-14 | **F** | `Partial<T>` makes all properties optional, not required. `Required<T>` makes them required. |
| TF-15 | **T** | `Record<K, V>` creates an object type with keys of type `K` and values of type `V`. |
| TF-16 | **T** | `Pick` creates a type by selecting the specified keys from the original type. |
| TF-17 | **T** | Mapped types use the `[K in keyof T]` syntax to transform each property of an existing type. |
| TF-18 | **F** | `satisfies` validates without changing the type. The variable keeps its narrower inferred type. |
| TF-19 | **T** | The `in` operator is a valid type guard that narrows based on property existence. |
| TF-20 | **F** | Conditional types are resolved at compile time, not runtime. They are part of the type system, not executable code. |
| TF-21 | **T** | `ReturnType<typeof fn>` extracts the return type of a function. |
| TF-22 | **T** | The `use` prefix is required for React to recognize custom hooks and enforce the Rules of Hooks. |
| TF-23 | **T** | Without `as const`, TypeScript infers `(string | Dispatch<...>)[]` instead of the intended `[string, Dispatch<...>]` tuple. |
| TF-24 | **F** | `React.FC` is not recommended. The React team and community prefer plain function declarations. |
| TF-25 | **T** | `React.ReactNode` is the correct type for children that can be strings, numbers, elements, fragments, or null. |
| TF-26 | **T** | `strict` is a meta-flag that enables `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, and several others. |
| TF-27 | **T** | `"moduleResolution": "bundler"` was introduced for tools like Vite, esbuild, and webpack that resolve modules differently from Node.js. |
| TF-28 | **F** | `as const` does the opposite: it makes values readonly and infers the narrowest possible literal types. |
| TF-29 | **T** | Template literal types use backtick syntax at the type level, e.g., `` type Route = `/${string}` ``. |
| TF-30 | **F** | Branded types are a compile-time-only technique. The brand property does not exist at runtime; it is erased like all type information. |
| TF-31 | **F** | `.d.ts` files contain only type declarations. They produce no JavaScript output. |
| TF-32 | **T** | TypeScript automatically includes `node_modules/@types` in the type resolution path by default. |