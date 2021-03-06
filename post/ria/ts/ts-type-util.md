---
title: typescript type utils
date: 2020-06-12
private: true
---
# types utils
## 强制类型（Type Assertion）
有时编译器不知道用什么类型，可以用来手动指定一个值的类型。
Note: 断言不是类型转换

    let c = (a as number).toExponential()
    let d = (<number>a).toExponential()
    process.env.APP_ENV as APPENV || 'dev';

有时编译器类型不确定：

    refEl.current as unknown as HTMLDivElement

## ts类型别名
常用于联合类型

    type Name = string;
    type NameResolver = () => string;
    type NameOrResolver = Name | NameResolver;
    function getName(n: NameOrResolver): Name {
        if (typeof n === 'string') {
            return n;
        } else {
            return n();
        }
    }

## 获取类型
### typeof 获取数据类型
    const data = {
        value: 123,
        text: 'text'
    };
    type Data = typeof data;
    // type Data = {
    //   value: number;
    //   text: string;
    // }

获取数据元素类型: via type key 

    const data = ['text 1', 'text 2'] as const;
    type Data = typeof data[number];
    // type Data = "text 1" | "text 2"

via data key

    const locales = [
        {
            locale: 'se',
            language: 'Swedish',
        },
        {
            locale: 'en',
            language: 'English',
        }
    ] as const;
    type Locale = typeof locales[number]['locale'];
    // type Locale = "se" | "en"


### ReturnType获取类型
https://stackoverflow.com/questions/36015691/obtaining-the-return-type-of-a-function

    const foo = (): FooReturnType => { }
    
    // 获取函数类型
    typeof foo

    // 获取函数返回类型
    type returnType = ReturnType<typeof foo>; 

更多示例

    type T10 = ReturnType<() => string>;  // string
    type T11 = ReturnType<(s: string) => void>;  // void
    type T12 = ReturnType<(<T>() => T)>;  // {}
    type T13 = ReturnType<(<T extends U, U extends number[]>() => T)>;  // number[]


## keyof
### keyof T
    interface Person {
        name: string;
        age: number;
        location: string;
    }

    type K1 = keyof Person; // "name" | "age" | "location"
    type K2 = keyof Person[]; // "length" | "push" | "pop" | "concat" | ...
    type K3 = keyof { [x: string]: Person }; // string

### keyof data

    const data = {K:1}
    type Keys = keyof typeof data


## valueof

    Person[keyof Person];
    type ValueOf<T> = T[keyof T];

which gives you

    type Foo = { a: string, b: number };
    type ValueOfFoo = ValueOf<Foo>; // string | number
    // or 
    type sameAsString = Foo['a']; // lookup a in Foo
    type sameAsNumber = Foo['b']; // lookup b in Foo


## extends(eleOf)
https://stackoverflow.com/questions/49285864/is-there-a-valueof-similar-to-keyof-in-typescript

下例中key 与value 类型配对

    type JWT = { id: string, token: string, expire: Date };
    const obj: JWT = { id: 'abc123', token: 'tk01', expire: new Date(2018, 2, 14) };

    function onChange(key: keyof JWT, value: JWT[keyof JWT]) {
        obj[key] = value //value match key type?
    }

用`extends` 推断泛型的, The idea is that the `key` parameter allows the compiler to infer the `generic K` parameter.
 Then it requires that `value` matches `JWT[K]`, the lookup type you need.

    declare function onChange<K extends keyof JWT>(key: K, value: JWT[K]): void; 

### in union type
in 类似于extends

    type Foo = 'a' | 'b';
    type Bar = {[key in Foo]: any};

## Map key type

    type M = {[key:string]: any}

partial key with `?`

    type M = {[key in 'age'| 'name']: any}
    type M = {[key in 'age'| 'name']?: any}

## Partial

    interface Todo {
        title: string;
        description: string;
    }
    function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
        return { ...todo, ...fieldsToUpdate };
    }

## `Record<K,T>`
Constructs a type with a set of properties K of type T. This utility can be used to map the properties of a type to another type.

    interface PageInfo {
        title: string;
    }

    type Page = 'home' | 'about' | 'contact';

    const x: Record<Page, PageInfo> = {
        about: { title: 'about' },
        contact: { title: 'contact' },
        home: { title: 'home' },
    };

## `Pick<T,K>`
Constructs a type by picking the set of properties K from T.

    interface Todo {
        title: string;
        description: string;
        completed: boolean;
    }

    type TodoPreview = Pick<Todo, 'title' | 'completed'>;

    const todo: TodoPreview = {
        title: 'Clean room',
        completed: false,
    };

## Exclude
    type T0 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
    type T1 = Exclude<"a" | "b" | "c", "a" | "b">;  // "c"
    type T2 = Exclude<string | number | (() => void), Function>;  // string | number

## Extract
    type T0 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
    type T1 = Extract<string | number | (() => void), Function>;  // () => void

## NonNullable
    type T0 = NonNullable<string | number | undefined>;  // string | number
    type T1 = NonNullable<string[] | null | undefined>;  // string[]

## `Parameters<T>`
    declare function f1(arg: { a: number, b: string }): void
    type T0 = Parameters<() => string>;  // []
    type T1 = Parameters<(s: string) => void>;  // [string]
    type T2 = Parameters<(<T>(arg: T) => T)>;  // [unknown]
    type T4 = Parameters<typeof f1>;  // [{ a: number, b: string }]
    type T5 = Parameters<any>;  // unknown[]
    type T6 = Parameters<never>;  // never
    type T7 = Parameters<string>;  // Error
    type T8 = Parameters<Function>;  // Error

## ConstructorParameters
The `ConstructorParameters<T>` type lets us extract all parameter types of a constructor function type. 
t produces a tuple type with all the parameter types (or the type never if T is not a function).

    type T0 = ConstructorParameters<ErrorConstructor>;  // [(string | undefined)?]
    type T1 = ConstructorParameters<FunctionConstructor>;  // string[]
    type T2 = ConstructorParameters<RegExpConstructor>;  // [string, (string | undefined)?]

# 官方的type utils
https://www.typescriptlang.org/docs/handbook/utility-types.html#partial