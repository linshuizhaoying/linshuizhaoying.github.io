---
layout: post
title: 记某次项目的中期重构工作 - 学习篇
category: 技术
tags: [学习,开发,前端,Node,React,React,Mock,重构,typescript,rxjs,redux]
keywords: 学习,开发,前端,RxJS,React,React,Mock,重构,typescript,rxjs,redux
description: 
---

# 前言

  之前开发的时候做过技术评估，然后开发到一半发现一些问题，比如 Typescript 的一些应用场景并没有很好的使用。比如一些组件的设计，项目的一些为了进度而写的丑代码。
  其实起因还是因为 Typescript 的 any 问题，当时微信群有很多热心的小伙伴引导，后来我说那我项目完成后再重构吧，然后小伙伴齐刷刷的说一般项目结束后没人会重构的，当时心想你们说的好有道理我竟然无法反驳。。。
  于是在前端交互完成，后端基本架构都完成了的情况下，我打算对现有的一些项目进行重构，当然在开发的时候已经重构了部分我就简单带过。
 
# 目标

作为在中期重构我们肯定需要抓紧时间，我给自己分配的大概是3天重构的时间(ps: 不包括学习时间)，因此需要明确目标，然后一心怼。

1. Typescript的应用问题，比如 any, 接口等等
2. 项目中重复代码需要精简处理
3. 一些丑代码的更改。
4. 必要的注释。

## Typescript的应用问题

在解决问题之前我需要重新回顾一些基础的知识点，然后需要去看一些成熟项目的源码来学习这些思维。

首先我很久以前学习 Typescript 的时候已经做过笔记。然后之前在自己的咨询应用中看到一系列的 Typescript 的文章，我将其标记过。这样我可以边看自己的然后和别人的文章相对照。


### 规范提升学习

#### 接口学习

常用的接口定义一般是这么写的:

```

interface Project{
  _id : String,
  projectName?: String,
  projectUrl?: String,
  projectDesc?: String,
  version?: String,
  transferUrl?: String,
  status?: String,
  type?: String,
  teamMember?: Array<any>,
  interfaceList?: Array<any>,
}


```

这在约定好接口字段的时候很管用，但是有时候字段没约定好，比如有时候可能会传入多余的数据，这个时候编译前会自动报错提醒。

此时可以考虑这么写

```

interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}

SquareConfig可以有任意数量的属性，并且只要它们不是color和width，那么就无所谓它们的类型是什么。

```




#### 泛型学习

经过学习，除去经常用的接口定义 Interface, 以及常用的基本类型定义。想要更优写法肯定需要应用上泛型。

> 泛型来创建可重用的组件，一个组件可以支持多种类型的数据。 这样用户就可以以自己的数据类型来使用组件。

> 泛型（Generics）是指在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性。 

> 泛型就是指定一个表示类型的变量，用它来代替某个实际的类型用于编程，而后通过实际调用时传入或推导的类型来对其进行替换，以达到一段使用泛型程序可以实际适应不同类型的目的。

`HandBook` 上有几个例子，比如当需要一种方法使返回值的类型与传入参数的类型是相同的情况下。

```

function identity<T>(arg: T): T {
    return arg;
}

给identity添加了类型变量T。 T帮助我们捕获用户传入的类型（比如：number），之后我们就可以使用这个类型。 之后我们再次使用了T当做返回值类型。现在我们可以知道参数类型与返回值类型是相同的了。 这允许我们跟踪函数里使用的类型的信息。

```

还有就是泛型类的应用。

泛型类其实多数时候是应用于容器类。假设我们需要实现一个 FilteredList，我们可以向其中 add()(添加) 任意数据，但是它在添加的时候会自动过滤掉不符合条件的一些，最终通过 get all() 输出所有符合条件的数据(数组)。而过滤条件在构造对象的时候，以函数或 Lambda 表达式提供。

```

// 声明泛型类，类型变量为 T
class FilteredList<T> {
    // 声明过滤器是以 T 为参数类型，返回 boolean 的函数表达式
    filter: (v: T) => boolean;
    // 声明数据是 T 数组类型
    data: T[];
    constructor(filter: (v: T) => boolean) {
        this.filter = filter;
    }

    add(value: T) {
        if (this.filter(value)) {
            this.data.push(value);
        }
    }

    get all(): T[] {
        return this.data;
    }
}

// 处理 string 类型的 FilteredList
const validStrings = new FilteredList<string>(s => !s);

// 处理 number 类型的 FilteredList
const positiveNumber  = new FilteredList<number>(n => n > 0);

```

这个可以应用于维护本地数据的地方，以及ajax提交数据的一些地方。主要是用于筛选数据。我们完全可以基于这个思想做一些多条件的过滤。在很多对数据需要处理的地方这个还是很有用的。

#### 命名空间与模块

 “内部模块”现在称做“命名空间”。 “外部模块”现在则简称为“模块”。
 
 常用的 `helper函数`，封装的 `until` 系列都是习惯用 export 导出模块然后引用的。但是一些验证函数如果也这么使用就不稳妥了。
 
 随着更多验证器的加入，我们需要一种手段来组织代码，以便于在记录它们类型的同时还不用担心与其它对象产生命名冲突。 因此，我们把验证器包裹到一个命名空间内，而不是把它们放在模块下。
  我们来看一个例子。在之前我们是这么写的:
  
  ```
  
  interface StringValidator {
    isAcceptable(s: string): boolean;
}

let lettersRegexp = /^[A-Za-z]+$/;
let numberRegexp = /^[0-9]+$/;

class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}

class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: StringValidator; } = {};
validators["ZIP code"] = new ZipCodeValidator();
validators["Letters only"] = new LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
    for (let name in validators) {
        let isMatch = validators[name].isAcceptable(s);
        console.log(`'${ s }' ${ isMatch ? "matches" : "does not match" } '${ name }'.`);
    }
}
  
  ```

然后改成命名空间的写法：

```

namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    const lettersRegexp = /^[A-Za-z]+$/;
    const numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: Validation.StringValidator; } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
    for (let name in validators) {
        console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
    }
}

```

很明显这个组织格式差别不大，但是在一个命名空间内可以避免一些不必要的冲突。

然后我们可以将一些不同逻辑的代码进行物理分割，但是在逻辑上，在同一个命名空间内，即使它们分离两地但是还是心连心。

```

Validation.ts
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }
}
LettersOnlyValidator.ts
/// <reference path="Validation.ts" />
namespace Validation {
    const lettersRegexp = /^[A-Za-z]+$/;
    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }
}
ZipCodeValidator.ts
/// <reference path="Validation.ts" />
namespace Validation {
    const numberRegexp = /^[0-9]+$/;
    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}
Test.ts
/// <reference path="Validation.ts" />
/// <reference path="LettersOnlyValidator.ts" />
/// <reference path="ZipCodeValidator.ts" />

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: Validation.StringValidator; } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
    for (let name in validators) {
        console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
    }
}

```

模块的基本都了解，在 Typescirpt 中，我们可以用更好的方式组织模块。

比如我们常常引用自己的模块是通过相对导入。

比如 `import { DefaultHeaders } from "../constants/http";`

我们引用的一些外部模块是非相对的.

比如 `import * as $ from "jQuery";`

对于一些比较成熟的依赖库我们可以通过将其作为外部模块载入。

>可以使用顶级的 export 声明来为每个模块都定义一个 .d.ts 文件，但最好还是写在一个大的 .d.ts 文件里。 我们使用与构造一个外部命名空间相似的方法，但是这里使用 module 关键字并且把名字用引号括起来，方便之后 import.

```

node.d.ts (simplified excerpt)
declare module "url" {
    export interface Url {
        protocol?: string;
        hostname?: string;
        pathname?: string;
    }

    export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
}

declare module "path" {
    export function normalize(p: string): string;
    export function join(...paths: any[]): string;
    export let sep: string;
}

```

我们可以 `/// <reference> node.d.ts` 并且使用 `import url = require("url");` 或 `import * as URL from "url"` 加载模块

```

/// <reference path="node.d.ts"/>
import * as URL from "url";
let myUrl = URL.parse("http://www.typescriptlang.org");

```
 
#### 书写声明文件

针对模块有三种可用的模块， `module.d.ts`, `module-class.d.ts` 和 `module-function.d.ts`. 

如果模块不能被调用或构造，使用module.d.ts文件。

`module.d.ts`里一般这么写:

```

// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ This is the module template file. You should rename it to index.d.ts
 *~ and place it in a folder with the same name as the module.
 *~ For example, if you were writing a file for "super-greeter", this
 *~ file should be 'super-greeter/index.d.ts'
 */

/*~ If this module is a UMD module that exposes a global variable 'myLib' when
 *~ loaded outside a module loader environment, declare that global here.
 *~ Otherwise, delete this declaration.
 */
export as namespace myLib;

/*~ If this module has methods, declare them as functions like so.
 */
export function myMethod(a: string): string;
export function myOtherMethod(a: number): number;

/*~ You can declare types that are available via importing the module */
export interface someType {
    name: string;
    length: number;
    extras?: string[];
}

/*~ You can declare properties of the module using const, let, or var */
export const myField: number;

/*~ If there are types, properties, or methods inside dotted names
 *~ of the module, declare them inside a 'namespace'.
 */
export namespace subProp {
    /*~ For example, given this definition, someone could write:
     *~   import { subProp } from 'yourModule';
     *~   subProp.foo();
     *~ or
     *~   import * as yourMod from 'yourModule';
     *~   yourMod.subProp.foo();
     */
    export function foo(): void;
}

```


```

使用 module-class.d.ts 如果模块能够使用new来构造：
var x = require("bar");
// Note: using 'new' operator on the imported variable
var y = new x("hello");

```

`module-class.d.ts` 的模板写法如下:

```

// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ This is the module template file for function modules.
 *~ You should rename it to index.d.ts and place it in a folder with the same name as the module.
 *~ For example, if you were writing a file for "super-greeter", this
 *~ file should be 'super-greeter/index.d.ts'
 */

/*~ Note that ES6 modules cannot directly export callable functions.
 *~ This file should be imported using the CommonJS-style:
 *~   import x = require('someLibrary');
 *~
 *~ Refer to the documentation to understand common
 *~ workarounds for this limitation of ES6 modules.
 */

/*~ If this module is a UMD module that exposes a global variable 'myFuncLib' when
 *~ loaded outside a module loader environment, declare that global here.
 *~ Otherwise, delete this declaration.
 */
export as namespace myFuncLib;

/*~ This declaration specifies that the function
 *~ is the exported object from the file
 */
export = MyFunction;

/*~ This example shows how to have multiple overloads for your function */
declare function MyFunction(name: string): MyFunction.NamedReturnType;
declare function MyFunction(length: number): MyFunction.LengthReturnType;

/*~ If you want to expose types from your module as well, you can
 *~ place them in this block. Often you will want to describe the
 *~ shape of the return type of the function; that type should
 *~ be declared in here, as this example shows.
 */
declare namespace MyFunction {
    export interface LengthReturnType {
        width: number;
        height: number;
    }
    export interface NamedReturnType {
        firstName: string;
        lastName: string;
    }

    /*~ If the module also has properties, declare them here. For example,
     *~ this declaration says that this code is legal:
     *~   import f = require('myFuncLibrary');
     *~   console.log(f.defaultName);
     */
    export const defaultName: string;
    export let defaultLength: number;
}

```



```

使用 module-function.d.ts，如果模块能够作为函数调用。
var x = require("foo");
// Note: calling 'x' as a function
var y = x(42);

```

`module-function.d.ts` 的模板写法如下:

```

// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ This is the module template file for class modules.
 *~ You should rename it to index.d.ts and place it in a folder with the same name as the module.
 *~ For example, if you were writing a file for "super-greeter", this
 *~ file should be 'super-greeter/index.d.ts'
 */

/*~ Note that ES6 modules cannot directly export class objects.
 *~ This file should be imported using the CommonJS-style:
 *~   import x = require('someLibrary');
 *~
 *~ Refer to the documentation to understand common
 *~ workarounds for this limitation of ES6 modules.
 */

/*~ If this module is a UMD module that exposes a global variable 'myClassLib' when
 *~ loaded outside a module loader environment, declare that global here.
 *~ Otherwise, delete this declaration.
 */
export as namespace myClassLib;

/*~ This declaration specifies that the class constructor function
 *~ is the exported object from the file
 */
export = MyClass;

/*~ Write your module's methods and properties in this class */
declare class MyClass {
    constructor(someParam?: string);

    someProperty: string[];

    myMethod(opts: MyClass.MyClassMethodOptions): number;
}

/*~ If you want to expose types from your module as well, you can
 *~ place them in this block.
 */
declare namespace MyClass {
    export interface MyClassMethodOptions {
        width?: number;
        height?: number;
    }
}

```

一些规范之前没有注意，在项目需要重新审视。

>不要使用如下类型Number，String，Boolean或Object。 这些类型指的是非原始的装盒对象，它们几乎没在JavaScript代码里正确地使用过。

```

/* 错误 */
function reverse(s: String): String;

/* OK */
function reverse(s: string): string;

```

一些例子:

**全局变量** :  `declare var foo: number;`

**全局变量** : `declare function greet(greeting: string): void;`

**带属性的对象** :

```

全局变量myLib包含一个makeGreeting函数， 还有一个属性numberOfGreetings指示目前为止欢迎数量。

代码

let result = myLib.makeGreeting("hello, world");
console.log("The computed greeting is:" + result);

let count = myLib.numberOfGreetings;
声明

使用declare namespace描述用点表示法访问的类型或值。
declare namespace myLib {
    function makeGreeting(s: string): string;
    let numberOfGreetings: number;
}

```

**函数重载** :

```

getWidget函数接收一个数字，返回一个组件，或接收一个字符串并返回一个组件数组。

代码

let x: Widget = getWidget(43);

let arr: Widget[] = getWidget("all of them");
声明

declare function getWidget(n: number): Widget;
declare function getWidget(s: string): Widget[];

```

**组织类型** :

```

greeter对象能够记录到文件或显示一个警告。 你可以为.log(...)提供LogOptions和为.alert(...)提供选项。
代码

const g = new Greeter("Hello");
g.log({ verbose: true });
g.alert({ modal: false, title: "Current Greeting" });
声明

使用命名空间组织类型。
declare namespace GreetingLib {
    interface LogOptions {
        verbose?: boolean;
    }
    interface AlertOptions {
        modal: boolean;
        title?: string;
        color?: string;
    }
}
你也可以在一个声明中创建嵌套的命名空间：
declare namespace GreetingLib.Options {
    // Refer to via GreetingLib.Options.Log
    interface Log {
        verbose?: boolean;
    }
    interface Alert {
        modal: boolean;
        title?: string;
        color?: string;
    }
}

```

### 开源项目学习

针对项目的技术栈是 `react全家桶 + typescript`

需要找一些成熟的项目学习一下它们的写法。

简单的确定下目标，一个是.d.ts文件的管理，一个是各种 any的解决方案。

#### d.ts文件的管理

1. 直接在 模块顶部定义。
2. 建立  `typings` 目录，统一根据模块名管理
   
  `global.d.ts` 用于全局变量和方法管理
  `modules.d.ts` 用于管理引用的模块
  
3. 在模块的同级目录下建立新的ts文件用于接口定义等。(可以统一管理，也可以分模块名-types.ts分别管理)
   比如建立 `index.d.ts` 统一 或者 `login-types.ts` 分割模块。
   
4. 建立 `interfaces` 目录，分割多个ts小文件定义用 `export`暴露出来，通过一个 `index.d.ts` 统一引用管理


#### Any 的一些解决方案

1. 通过利用 `redux.d.ts` 定义的 `Action`，可以将 `reducer` 里的 `any` 给去除

   ` action: Action<Test> `
   
   当然也可以不引用 `Action`,而是自己定义一个 `interface`
   
  比如
 
```
   
interface Loading{
  type: string,
  data: object
}
   
const loading = (state = initialState, action: Loading) => {}
   
```

2. 在 `tsx` 文件中,  `Props` 可以引用 `Redux` 的 `ActionCreator`。

```

interface IProps {
  counter: ICounter;
  increment: Redux.ActionCreator<ICounterAction>;
  decrement: Redux.ActionCreator<ICounterAction>;
}

@connect(
  (state) => ({ counter: state.counter }),
  (dispatch) => ({
    decrement: () => dispatch(decrement()),
    increment: () => dispatch(increment()),
  }),
)

class Counter extends React.Component<IProps, {}> {}

```

# 结尾

接下来就是根据需求对前端项目进行更改了。

# 资料

> [typescript-handbook](https://zhongsp.gitbooks.io/typescript-handbook/)


> [官方英文文档](http://www.typescriptlang.org/docs/handbook/jsx.html)


> [官方中文文档](https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook/tutorials/TypeScript%20in%205%20minutes.html)

> [非官方文档](https://ts.xcatliu.com/basics/primitive-data-types.html)

> [写d.ts参考](http://definitelytyped.org/)

> [边城 Typescript 系列文章](https://segmentfault.com/a/1190000010774159)


