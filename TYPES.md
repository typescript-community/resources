# Useful Types
I have written a list of types that are commonly used/may be useful.
Credits: @Robinlemon and @Micah

## Opaque
This constructs a type that is unique. This is very useful since it allows us to have different types for e.g.: a string and base64 data.
```ts
declare const Unique: unique symbol;
export type Opaque<T, K extends string> = T & { [Unique]: K };

/* Usage */
export type Base64Type = Opaque<string, '__base64__'>;
const DecodeBase64 = (B64String: Base64Type): string => Buffer.from(B64String, 'base64').toString('utf8');
```

## ResolvePromise

Returns the type `T` inside of a `Promise<T>`.

```ts
export type ResolvePromise<T> = T extends PromiseLike<infer R> ? R : never;

/* Usage */
const Result = Promise.resolve(0); // Promise<number>
type TResult = ResolvePromise<typeof Result> //number
```

## ClassType
Represents any class constructor.
```ts
/* eslint-disable-next-line @typescript-eslint/no-explicit-any */
export type ClassType = { new (...args: any[]): any };

/* Usage */
const ClassFactory = <T extends ClassType>(Ctor: T): InstanceType<T> => new Ctor();
```

## ClassMethodTypes
Returns a union of the **public** method types on a class.
```ts
export type ClassMethodTypes<T> = {
    [K in keyof T]: T[K] extends (...args: never[]) => void ? T[K] : never;
}[keyof T];

/* Usage */
class A {
    public a = (): void => {}
    public b(param: string): number { return 3; }
}

type MethodTypes = ClassMethodTypes<A>; // (() => void | (param: string) => number)
```

## ClassMethodNames
Returns a string union of **public** method names on a class.
```ts
export type ClassMethodNames<T> = {
    [K in keyof T]: T[K] extends (...args: never[]) => void ? K : never;
}[keyof T];

/* Usage */
class A {
    public a = (): void => {}
    public b(param: string): number { return 3; }
}

type MethodNames = ClassMethodNames<A>; // "a" | "b"
```

## ClassMethodTypesFiltered
Returns a union of the **public** method types on a class, but only where the method signature is assignable to U.
```ts
export type ClassMethodTypesFiltered<T, U> = {
    [K in keyof T]: U extends T[K] ? K : never;
}[keyof T];

/* Usage */
class A {
    public a = (): void => {}
    public b(param: string): number { return 3; }
}

type FilteredMethods = ClassMethodTypesFiltered<A, (...args: any[]) => number>; // (param: string) => number
```

## ClassStaticProps
Returns a type containing the static properties of a class.
```ts
export type ClassStaticProps<T> = Omit<T, 'prototype'>;

/* Usage */
class A {
    public static b: string = "abc";
}

/* typeof Important Here */
type StaticProps = ClassStaticProps<typeof A>; // { b: string }
```

## Expand
A debug type used to expand deep or messy types.
```ts
type Primitive = string | number | boolean | bigint | null | void | symbol | Function
type Expand<T> = T extends Primitive ? T : { [K in keyof T]: Expand<T[K]> }

/* Usage */
interface Foo<T> {
    foo: T
}
interface Bar {
    a: Foo<1>
    b: Foo<string>
}
interface Zip {
    c: boolean
}
type Collapsed = (Bar & Zip)[] // (Bar & Zip)[]
type Expanded = Expand<Collapsed> // type Expanded = { a: { foo: 1 }, b: { foo: string }, c: boolean }[]
```

## RemoveFirstParam
Remove the first parameter from a function signature.
```ts
export type RemoveFirstFromTuple<T extends any[]> = T['length'] extends 0 ? undefined : (((...b: T) => void) extends (a: any, ...b: infer I) => void ? I : []);
export type RemoveFirstParam<T extends (...args: any[]) => any> = (...params: RemoveFirstFromTuple<Parameters<T>>) => ReturnType<T>;

/* Usage */
const MyFn = (a: string, b: number): void => {};
const MyModifiedFunc: RemoveFirstParam<typeof MyFn> = (b: number): void => {};
```

## Extract Explicit Keys from Indexed Type
```ts
type ExplicitKeys<T> = { [K in keyof T]: string extends K ? never : number extends K ? never : K } extends { [_ in keyof T]: infer U } ? U : never
interface Apple {
    [prop: string]: boolean
    'hello': true
    'goodbye': false
}
type ExplicityAppleKeys = ExplicitKeys<Apple> // 'hello' | 'goodbye'
```

## Deep Readonly
```ts
export type DeepReadonly<T> = Readonly<{
    [k in keyof T]:
        T[k] extends unknown[] ? ReadonlyArray<DeepReadonly<T[k][number]>> :
        T[k] extends Function ? T[k] :
        T[k] extends object ? DeepReadonly<T[k]> :
            T[k];
}>
```

## Pair Array to Object
```ts
type PairToObject<T> = T extends readonly [string|number|symbol, unknown] ? { [index in T[0]]: T[1] } : never
type UnionToIntersection<T> = (T extends unknown ? (k: T) => void : never) extends (k: infer I) => void ? I : never
type PairArrayToObject<T extends readonly (readonly [string|symbol,unknown])[]> = UnionToIntersection<PairToObject<T[number]>>

/* Usage */
const fruitTuples = [
    ['apple', 'red'],
    ['banana', 'yellow'],
    ['cherry', 'black'],
] as const;
const fruitObject = fruitTuples.reduce((accumulator, [key, value]) => ({
    ...accumulator,
    [key]: value
}), {} as PairArrayToObject<typeof fruitTuples>)
fruitObject // type { apple: "red", banana: "yellow", cherry: "black" }
```

## Filter Object by Value Type
```ts
type ExtractObjectKeysByValueType<T extends object, V> = { [Key in keyof T]: T[Key] extends V ? Key : never }[keyof T]
type FilterObjectByValueType<T extends object, V> = { [Key in ExtractObjectKeysByValueType<T, V>]: T[Key] }

/* Usage */
interface Apple {
    color: 'red' | 'green'
    seeds: number
    shape: 'sphere'
}
type FilteredApple = FilterObjectByValueType<Apple, string> // type: { color: 'red' | 'green', shape: 'sphere' }
```

## Merge a union into a single type with unioned properties
```ts
type GetAllKeys<T> = T extends unknown ? keyof T : never
type GetAllValues<T, K extends string | number | symbol> = T extends object ? K extends keyof T ? T[K] : never : never
type Merge<T> = { [K in GetAllKeys<T>]: GetAllValues<T, K> }

/* Usage */
type Apple = Merge<{ a: number, c: 'hello' } | { a: string; b: boolean }> // { a: string | number, b: boolean, c: 'hello' }
```

## Get `keyof` recursively

```ts
type Primitive = string | number | boolean | bigint | null | void | symbol | Function
type RecursiveKeys<T, U extends string = ''> = T extends Primitive ? never : {
  [K in keyof T & string]: `${U}${K}` | RecursiveKeys<T[K], `${U}${K}.`>
}[keyof T & string]

type Out = RecursiveKeys<{ a: string, b: { c: number } }>
//   ^? - type Out = "a" | "b" | "b.c"
```

## Keyed object by property from union
```ts
type ToKeyedObject<T, U extends keyof T> = { [Key in T[U] & PropertyKey]: Extract<T, { [_ in U]: Key}> }

/* Usage */
interface Apple {
    kind: 'apple'
}
interface Banana {
    kind: 'banana'
}
type Fruit = Apple | Banana
type x = ToKeyedObject<Fruit, 'kind'>;
//   ^? - type x = {
//       apple: Apple;
//       banana: Banana;
//   }
```

## Type Safe Guard Clauses
```ts
// the check function passed to this should return the original value (after narrowing), or undefined (in the case of no match)
function createGuard<T, U extends T>(check: (maybe: T) => U | undefined): (maybe: T) => maybe is U {
    return (maybe: T): maybe is U => check(maybe) !== undefined
}

/* Usage */
type Apple = { size: 'large', color: 'red' }
type Banana = { size: 'large', color: 'yellow' }
type Cherry = { size: 'small', color: 'red' }
type Fruit = Apple | Banana | Cherry

declare const fruit: Fruit

const isBanana = createGuard((fruit: Fruit) => fruit.color === 'yellow' ? fruit : undefined)
if (isBanana(fruit)) {
    fruit
    // ^? const fruit: Banana
}

// naive way
function isCherryUnvalidated(fruit: Fruit): fruit is Cherry { return fruit.color === 'red' }
if (isCherryUnvalidated(fruit)) {
    fruit
    // ^? const fruit: Cherry
    // Uh oh, we didn't narrow enough but the compiler happily told us we have a Cherry when we could have an Apple!
}
// improved way
const isCherry = createGuard((fruit: Fruit) => fruit.color === 'red' ? fruit : undefined)
if (isCherry(fruit)) {
    fruit
    // ^? const fruit: Apple | Cherry
    // Since the return type of the guard is inferred, we correctly get the right type even if our function name is wrong
}

const isApple = createGuard((fruit: Fruit) => fruit.color === 'red' && fruit.size === 'large' ? fruit : undefined)
if (isApple(fruit)) {
    fruit
    // ^? const fruit: Apple
}
```
