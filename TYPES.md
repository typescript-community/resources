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
Returns the type T inside of a Promise<T>.
```ts
export type RemovePromise<T> = T extends PromiseLike<infer R> ? R : never;

/* Usage */
const Result = Promise.resolve(0); // Promise<number>
type TResult = RemovePromise<typeof Result> //number
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
    [K in keyof T]: T[K] extends (...args: unknown[]) => void ? T[K] : never;
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
    [K in keyof T]: T[K] extends (...args: unknown[]) => void ? K : never;
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
type PairToObject<T> = T extends readonly [string, string] ? { [index in T[0]]: T[1] } : never
type UnionToIntersection<T> = (T extends any ? (k: T) => void : never) extends (k: infer I) => void ? I : never
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
