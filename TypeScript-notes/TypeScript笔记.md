# TypeScript笔记

## 热编译

文件结构

```
project
│  gulpfile.js
│  package-lock.json
│  package.json
│  tsconfig.json
│  
├─dist
│      bundle.js
│      index.html
│      
├─node_modules
│
│
└─src
      greet.ts
      index.html
      main.ts
        
```

npm下包

```
npm install --save-dev browserify commonjs gulp gulp-typescript gulp-util tsify typescript vinyl-source-stream watchify
```

gulpfile.js

```js
var gulp = require("gulp");
var browserify = require("browserify");
var source = require('vinyl-source-stream');
var tsify = require("tsify");
// 热更新
var watchify = require("watchify");
var gutil = require("gulp-util");
var paths = {
  pages: ['src/*.html']
};

gulp.task("copy-js", function () {
  return browserify({
      basedir: '.',
      debug: true,
      entries: ['src/main.ts'],
      cache: {},
      packageCache: {}
    })
    .plugin(tsify)
    .bundle()
    .pipe(source('main.js'))
    .pipe(gulp.dest("dist"));
});

var watchedBrowserify = watchify(browserify({
  basedir: '.',
  debug: true,
  entries: ['src/main.ts'],
  cache: {},
  packageCache: {}
}).plugin(tsify));

gulp.task("copy-html", function () {
  return gulp.src(paths.pages)
    .pipe(gulp.dest("dist"));
});

var bundle = function () {
  return watchedBrowserify
    .bundle()
    .pipe(source('bundle.js'))
    .pipe(gulp.dest("dist"));
}

gulp.task(bundle)


gulp.task('default', gulp.series('copy-html', 'bundle'));
watchedBrowserify.on("update", bundle);
watchedBrowserify.on("log", gutil.log);
```

tsconfig.json

```js
{
    "files": [
        "src/main.ts",
        "src/greet.ts"
    ],
    "compilerOptions": {
        "noImplicitAny": true,
        "target": "es5"
    }
}
```

此时，在项目更目录使用`gulp`命令即可执行热编译。

## 基础类型

### 布尔值（boolean）

```ts
let isStr:boolean = false;
```

### 数字（number）

Typescript里所有的数字都是浮点数。支持十进制和十六进制的字面量，还支持ECMAScript 2015中引入的二进制和八进制字面量。

```ts
let decLiteral:number = 6;
let hexLiteral:number = 0xf00d;
let binaryLiteral:number = 0b1010;
let octalLiteral:number = 0o744;
```

### 字符串（string）

支持单、双引号以及模板字符串

```ts
let name:string = "bob";
let site:string = 'home';
let sentence:string =`My name is ${name}.I am at ${site} now.`
```

### 数组

```ts
// 第一种定义方法：在元素类型后面接上[]
let list:number[] = [1,2,3];

// 第二种定义方法：使用数组泛型
let arr:Array<number> = [1,2,3];
```

### 元组

先定义再使用

```ts
// 定义一个包含字符串和数字的元组
let x:[string,number,number];
// 初始化元组
x = ['hello',10,20];
// 初始化时必须要按照定义的时候写位置一一对应，否则报错。
x = [10,'hello',20]    //error
```

### 枚举

`enum`类型。使用枚举类型为一组数值赋予名字。

```ts
enum Color {red,green,blue}
let c:Color = Color.green;
```

默认情况下，元素编号从0开始，也可以设置为手动指定编号

```ts
enum Color {red=2,blue,green}
enum Color {red=2,blue=4,green=7}
```

枚举类型提供的一个便利是你可以由枚举的值得到它的名字。 例如，我们知道数值为2，但是不确定它映射到Color里的哪个名字，我们可以查找相应的名字：

```ts
enum Color {red = 1, green, bule}
let colorName: string = Color[2];
console.log(colorName);  // 'Green'
```

### any

使用`any`定义**不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查**

```ts
let notSure:any = 4;
notSure = "string";
notSure = false;
```

### void

`void`类型与`any`类型相反，表示没有任何类型。常见于函数没有返回值时，则其返回值类型是`void`。

```ts
function sayHi():void{
  console.log("Hi!~");
}
```

声明一个`void`类型的变量只能赋值`undefined`和`null`。

```ts
let unusable:void = undefined;
```

### null和undefined

`null`和`undefined`类型对应的值就是`unll`和`undefined`。

### never

`never`类型表示的是那些永不存在的值的类型。 例如， `never`类型是**那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型**； 变量也可能是 `never`类型，当它们被永不为真的类型保护所约束时。

```ts
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
    throw new Error(message);
}
```

### object

`object`表示非原始类型，也就是除`number`，`string`，`boolean`，`symbol`，`null`或`undefined`之外的类型。

```ts
declare function create(o: object | null): void;

create({ prop: 0 }); // OK
create(null); // OK

create(42); // Error
create("string"); // Error
create(false); // Error
create(undefined); // Error
```

### 类型断言

使用类型断言能让编译器知道你已经对类型进行了必须的检查，它不会再对数据进行检查和解构。

类型断言有两种形式。 其一是“尖括号”语法：

```ts
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;
```

另一个为`as`语法：

```ts
let someValue: any = "this is a string";

let strLength: number = (someValue as string).length;
```

两种形式是等价的。 至于使用哪个大多数情况下是凭个人喜好；

**然而，当你在TypeScript里使用JSX时，只有 `as`语法断言是被允许的。**

## 接口

TypeScript的核心原则之一是**对值所具有的结构进行类型检查**。

### 必选属性

使用`interface`定义一个接口，将这个接口作为类型传入函数，则对这个函数进行传参时需要有对应类型的值

```ts
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj); // Size 10 Object
```

### 可选属性

带有可选属性的接口与普通的接口定义差不多，只是在可选属性名字定义的后面加一个`?`符号。

```ts
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  let newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

let mySquare = createSquare({color: "black"});
```

### 只读属性

定义接口时，在属性吗赢钱加`readonly`来指定只读属性。

```ts
interface Point {
  readonly x:number;
  readonly y:number;
}
```

对象赋值后就不能再改变了

```ts
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!
```

TypeScript具有`ReadonlyArray<T>`类型，数组创建后就不能再改变了

```ts
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
ro.push(5); // error!
ro.length = 100; // error!
a = ro; // error!
```

上面代码的最后一行，可以看到就算把整个`ReadonlyArray`赋值到一个普通数组也是不可以的。 但是你可以用类型断言重写：

```ts
a = ro as number[];
```

#### `readonly` vs `const`

最简单判断该用`readonly`还是`const`的方法是看要把它做为变量使用还是做为一个属性。 做为变量使用的话用`const`，若做为属性则使用`readonly`。

## 类

类的实现

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
```

### 类的继承

```ts
class Animal {
    move(distanceInMeters: number = 0) {
        console.log(`Animal moved ${distanceInMeters}m.`);
    }
}

class Dog extends Animal {
    bark() {
        console.log('Woof! Woof!');
    }
}

const dog = new Dog();
dog.bark();
dog.move(10);
dog.bark();
```

类从基类中继承了属性和方法。 这里， `Dog`是一个 _派生类_，它派生自 `Animal` _基类_，通过 `extends`关键字。 派生类通常被称作 _子类_，基类通常被称作 _超类_。

### 类的属性私有化

TypeScript里，成员都默认为 `public`。即所有成员默认可见。

#### 理解 `private`

当成员被标记成 `private`时，它就不能在声明它的类的外部访问。比如：

```ts
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

new Animal("Cat").name; // 错误: 'name' 是私有的.
```

#### 理解 `protected`

`protected`修饰符与 `private`修饰符的行为很相似，但有一点不同， `protected`成员在派生类中仍然可以访问。

```ts
class Person {
    protected name: string;
    constructor(name: string) { this.name = name; }
}

class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name)
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
console.log(howard.getElevatorPitch());
console.log(howard.name); // 错误
```

#### readonly修饰符

你可以使用 `readonly`关键字将属性设置为只读的。 只读属性必须在声明时或构造函数里被初始化。

```ts
class Octopus {
    readonly name: string;
    readonly numberOfLegs: number = 8;
    constructor (theName: string) {
        this.name = theName;
    }
}
let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit"; // 错误! name 是只读的.
```

#### 存取器

TypeScript支持通过getters/setters来截取对对象成员的访问。 它能帮助你有效的控制对对象成员的访问。

```ts
let passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            console.log("Error: Unauthorized update of employee!");
        }
    }
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

#### 静态属性

当类被实例化的时候才会被初始化的属性。 创建类的静态成员，这些属性存在于**类本身**上面而不是**类的实例**上。

在这个例子里，我们使用 `static`定义 `origin`，因为它是所有网格都会用到的属性。 每个实例想要访问这个属性的时候，都要在 `origin`前面加上类名。 如同在实例属性上使用 `this.`前缀来访问属性一样，这里我们使用 `Grid.`来访问静态属性。

```ts
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

## 函数

### 函数类型

给函数添加类型检查

```ts
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x + y; };
```

```ts
let myAdd: (x: number, y: number) => number =
    function(x: number, y: number): number { return x + y; };
```

### 可选参数和默认参数

JavaScript里，每个参数都是可选的，可传可不传。 没传参的时候，它的值就是undefined。 在TypeScript里我们可以在参数名旁使用 `?`实现可选参数的功能。 比如，我们想让last name是可选的：

```ts
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");  // works correctly now
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");  // ah, just right
```

可选参数必须跟在必须参数后面。 如果上例我们想让first name是可选的，那么就必须调整它们的位置，把first name放在后面。

### 剩余参数

必要参数，默认参数和可选参数有个共同点：它们表示某一个参数。 有时，你想同时操作多个参数，或者你并不知道会有多少参数传递进来。 在JavaScript里，你可以使用 `arguments`来访问所有传入的参数。

在TypeScript里，你可以把所有参数收集到一个变量里：

```ts
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

剩余参数会被当做个数不限的可选参数。 可以一个都没有，同样也可以有任意个。 编译器创建参数数组，名字是你在省略号（ `...`）后面给定的名字，你可以在函数体内使用这个数组。

这个省略号也会在带有剩余参数的函数类型定义上使用到：

```ts
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let buildNameFun: (fname: string, ...rest: string[]) => string = buildName;
```

## 泛型

> [泛型](https://zh.wikipedia.org/wiki/%E6%B3%9B%E5%9E%8B)的定义主要有以下两种：
>
> 1. 在程序编码中一些包含**类型参数**的类型，也就是说泛型的参数只可以代表类，不能代表个别对象。（这是当今较常见的定义）
> 2. 在程序编码中一些包含参数的[类](https://zh.wikipedia.org/wiki/%E7%B1%BB\_\(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6\))。其参数可以代表类或对象等等。（现在人们大多把这称作[模板](https://zh.wikipedia.org/wiki/%E6%A8%A1%E6%9D%BF)）

一句话概况就是：使用泛型来创建可重用且支持多种类型的数据组件。

**定义一个泛型函数**

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

泛型的使用方法有两种：

第一种：传入所有参数，包含类型参数：

```typescript
let output = identity<string>("myString");// type of output will be 'string'
```

第二种方法更普遍。利用了**类型推论** ：

> 类型推论： 即编译器会根据传入的参数自动地帮助我们确定T的类型

```ts
let output = identity("myString");  // type of output will be 'string'
```

### 泛型类

将泛型定义在类中：

```ts
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

### 泛型约束

定义一个接口来描述约束条件，比如说泛型里需要处理带有`.length`属性的所有类型，只要传入有这个类型的属性，则不报错。

```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

### 泛型里使用类类型

```ts
class BeeKeeper {
    hasMask: boolean;
}

class ZooKeeper {
    nametag: string;
}

class Animal {
    numLegs: number;
}

class Bee extends Animal {
    keeper: BeeKeeper;
}

class Lion extends Animal {
    keeper: ZooKeeper;
}

function createInstance<A extends Animal>(c: new () => A): A {
    return new c();
}

createInstance(Lion).keeper.nametag;  // typechecks!
createInstance(Bee).keeper.hasMask;   // typechecks!
```

## 枚举

使用枚举我们可以定义一些带名字的常量。 使用枚举可以清晰地表达意图或创建一组有区别的用例。 TypeScript支持数字的和基于字符串的枚举。

### 数字枚举

定义了一个数字枚举， `Up`使用初始化为 `1`。 其余的成员会从 `1`开始自动增长。 换句话说，`Direction.Up`的值为 `1`， `Down`为 `2`， `Left`为 `3`， `Right`为 `4`

如果不对枚举成员使用初始化，则将会从`0`开始。

```ts
enum Direction {
    Up = 1,
    Down,
    Left,
    Right
}
```

### 字符串枚举

在一个字符串枚举里，每个成员都需要使用字符串字面量

```ts
enum Direction{
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT"
}
```

### 异构枚举

枚举可以混合字符串和数字成员，但是并不推荐这样做

```ts
enum isBook{
  No = 0,
  Yes = "YES",
}
```

## 类型兼容性

## 高级类型

### 交叉类型（Intersection Types）

### 联合类型（Union Types）

下面是一个联合类型的例子

```ts
function(x: string, y: string | number){
  return x + y
}
```

联合类型表示一个值可以是几种类型之一，使用`|`分隔每个类型。

如果一个值是联合类型，那只能访问这个类型里的成员

```ts
interface Bird {
    fly();
    layEggs();
}

interface Fish {
    swim();
    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // okay
pet.swim();    // errors
```

### 类型保护与区分类型（Type Guards and Differentiation Types）

套用上面的一段代码，当我们想判断`pet`是否为`Fish`时，我们需要在判断语句里额外添加**类型断言**。

```ts
let pet = getSmallPet();

if ((<Fish>pet).swim) {
    (<Fish>pet).swim();
}
else {
    (<Bird>pet).fly();
}
```

#### 用户自定义类型保护

要自定义一个类型保护，需要定义一个函数，返回一个**类型谓词**

> 类型谓词：`parameterName is Type`，其中`parameterName`必须是当前函数签名里的一个参数名。

```ts
function isFish(pet: Fish | Bird): pet is Fish {
    return (<Fish>pet).swim !== undefined;
}
```

#### `typeof`类型保护

```ts
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

**`typeof`类型保护**只有两种形式能被识别：`typeof v === "typename"`和 `typeof v !== "typename"`，

### 可以为null的类型

默认情况下，类型检查器认为 `null`与 `undefined`可以赋值给任何类型。`null`与 `undefined`是所有其它类型的一个有效值。 这也意味着，你阻止不了将它们赋值给其它类型，就算是你想要阻止这种情况也不行

`--strictNullChecks`标记可以解决此错误：当你声明一个变量时，它不会自动地包含 `null`或 `undefined`。 你可以使用联合类型明确的包含它们：

```ts
let s = "foo";
s = null; // 错误, 'null'不能赋值给'string'
let sn: string | null = "bar";
sn = null; // 可以
sn = undefined; // error, 'undefined'不能赋值给'string | null'
```

可选参数与可选属性

使用了 `--strictNullChecks`，可选参数和可选属性都会被自动地加上 `| undefined`:

```ts
function f(x: number, y?: number) {
    return x + (y || 0);
}
f(1, 2);
f(1);
f(1, undefined);
f(1, null); // error, 'null' is not assignable to 'number | undefined'

class C {
    a: number;
    b?: number;
}
let c = new C();
c.a = 12;
c.a = undefined; // error, 'undefined' is not assignable to 'number'
c.b = 13;
c.b = undefined; // ok
c.b = null; // error, 'null' is not assignable to 'number | undefined'
```

### 类型别名

类型别名会给一个类型起个新名字。 类型别名有时和接口很像，但是可以作用于原始值，联合类型，元组以及其它任何你需要手写的类型。

```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === 'string') {
        return n;
    }
    else {
        return n();
    }
}
```

### 可辨识联合

声明联合接口：

```ts
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
interface Circle {
    kind: "circle";
    radius: number;
}
```

首先我们声明了将要联合的接口。 每个接口都有 `kind`属性但有不同的字符串字面量类型。 `kind`属性称做 _可辨识的特征_或 _标签_。 其它的属性则特定于各个接口。 注意，目前各个接口间是没有联系的。 下面我们把它们联合到一起：

```ts
type Shape = Square | Rectangle | Circle;
```

现在我们使用可辨识联合:

```ts
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

### 索引类型（Index types）

通过 **索引类型查询**和 **索引访问**操作符：

```ts
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);
}

interface Person {
    name: string;
    age: number;
}
let person: Person = {
    name: 'Jarid',
    age: 35
};
let strings: string[] = pluck(person, ['name']); // ok, string[]
```

编译器会检查 `name`是否真的是 `Person`的一个属性。 本例还引入了几个新的类型操作符。 首先是 `keyof T`， **索引类型查询操作符**。 对于任何类型 `T`， `keyof T`的结果为 `T`上已知的公共属性名的联合。 例如：

```ts
let personProps: keyof Person; // 'name' | 'age'
```

## 迭代器与生成器

### 迭代器

#### `for..of` vs. `for..in` 语句

`for..of`和`for..in`均可迭代一个列表；但是用于迭代的值却不同，`for..in`迭代的是对象的 _键_ 的列表，而`for..of`则迭代对象的键对应的值。

下面的例子展示了两者之间的区别：

```ts
let list = [4, 5, 6];

for (let i in list) {
    console.log(i); // "0", "1", "2",
}

for (let i of list) {
    console.log(i); // "4", "5", "6"
}
```

### 生成器

#### 目标为 ES5 和 ES3

当生成目标为ES5或ES3，迭代器只允许在`Array`类型上使用。 在非数组值上使用 `for..of`语句会得到一个错误，就算这些非数组值已经实现了`Symbol.iterator`属性。

编译器会生成一个简单的`for`循环做为`for..of`循环，比如：

```ts
let numbers = [1, 2, 3];
for (let num of numbers) {
    console.log(num);
}
```

生成的代码为：

```js
var numbers = [1, 2, 3];
for (var _i = 0; _i < numbers.length; _i++) {
    var num = numbers[_i];
    console.log(num);
}
```

#### 目标为 ECMAScript 2015 或更高

当目标为兼容ECMAScipt 2015的引擎时，编译器会生成相应引擎的`for..of`内置迭代器实现方式。

## 模块

模块在其自身的作用域里执行，而不是在全局作用域里；这意味着定义在一个模块里的变量，函数，类等等在模块外部是不可见的，除非你明确地使用`export`形式之一导出它们。 相反，如果想使用其它模块导出的变量，函数，类，接口等的时候，你必须要导入它们，可以使用 `import`形式之一。

### 导出

#### 导出声明

任何声明（比如变量，函数，类，类型别名或接口）都能够通过添加`export`关键字来导出。

```ts
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

#### 导出语句

```ts
class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export { ZipCodeValidator };
export { ZipCodeValidator as mainValidator };
```

#### 重新导出

我们经常会去扩展其它模块，并且只导出那个模块的部分内容。 重新导出功能并不会在当前模块导入那个模块或定义一个新的局部变量。

```ts
export class ParseIntBasedZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && parseInt(s).toString() === s;
    }
}

// 导出原先的验证器但做了重命名
export {ZipCodeValidator as RegExpBasedZipCodeValidator} from "./ZipCodeValidator";
```

或者一个模块可以包裹多个模块，并把他们导出的内容联合在一起通过语法：`export * from "module"`。

```ts
export * from "./StringValidator"; // exports interface StringValidator
export * from "./LettersOnlyValidator"; // exports class LettersOnlyValidator
export * from "./ZipCodeValidator";  // exports class ZipCodeValidator
```

#### 默认导出

每个模块都可以有一个`default`导出。 默认导出使用 `default`关键字标记；并且一个模块只能够有一个`default`导出。 需要使用一种特殊的导入形式来导入 `default`导出。

类和函数声明可以直接被标记为默认导出。 标记为默认导出的类和函数的名字是可以省略的。

```ts
export default class ZipCodeValidator {
    static numberRegexp = /^[0-9]+$/;
    isAcceptable(s: string) {
        return s.length === 5 && ZipCodeValidator.numberRegexp.test(s);
    }
}
```

### 导入

模块的导入操作与导出一样简单。 可以使用以下 `import`形式之一来导入其它模块中的导出内容。

导入一个模块中的某个导出内容

```ts
import { ZipCodeValidator } from "./ZipCodeValidator";

let myValidator = new ZipCodeValidator();
```

可以对导入内容重命名

```ts
import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
let myValidator = new ZCV();
```

将整个模块导入到一个变量，并通过它来访问模块的导出部分

```ts
import * as validator from "./ZipCodeValidator";
let myValidator = new validator.ZipCodeValidator();
```

具有副作用的导入模块

尽管不推荐这么做，一些模块会设置一些全局状态供其它模块使用。 这些模块可能没有任何的导出或用户根本就不关注它的导出。 使用下面的方法来导入这类模块：

```ts
import "./my-module.js";
```

### `export =` 和 `import = require()`

CommonJS和AMD的环境里都有一个`exports`变量，这个变量包含了一个模块的所有导出内容。

CommonJS和AMD的`exports`都可以被赋值为一个`对象`, 这种情况下其作用就类似于 es6 语法里的默认导出，即 `export default`语法了。虽然作用相似，但是 `export default` 语法并不能兼容CommonJS和AMD的`exports`。

为了支持CommonJS和AMD的`exports`, TypeScript提供了`export =`语法。

`export =`语法定义一个模块的导出`对象`。 这里的`对象`一词指的是类，接口，命名空间，函数或枚举。

**若使用`export =`导出一个模块，则必须使用TypeScript的特定语法`import module = require("module")`来导入此模块。**
