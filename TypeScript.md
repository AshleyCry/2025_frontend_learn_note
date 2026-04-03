## TypeScript

### TypeScript的理解，与JavaScript的区别

TypeScript是JavaScript的类型的超集，支持ES6语法，支持面向对象编程的概念，如类、接口、继承、泛型等

它是一种静态类型检查的语言，提供了类型注解，在代码编译阶段就可以检查出数据类型的错误，同时扩展了JavaScript的语法，所有的JS程序都可以在不加改变的在TypeScript中工作

为了保证兼容性，TS在编译阶段需要编译器编译成纯JS来运行，是为大型应用开发而设计的语言

#### 特性

- 类型批注和编译时类型检查：在编译时批注变量类型
- 类型推断：ts中没有批注变量类型会自动推断变量的类型
- 类型擦除：编译过程中批注的内容和接口会在运行时利用工具擦除
- 接口：ts中用接口来定义对象类型
- 枚举：用于取值被限定在一定范围内的场景
- mixin：可以接受任意类型的值
- 泛型编程：写代码时使用一些以后才指定的类型
- 名字空间：名字只在该区域内有效，其他区域可重复使用该名字而不冲突
- 元组：元组合并了不同类型的对象，相当于一个可以装不同类型数据的数组
- 。。。

### TypeScript数据类型

- boolean
- number
- string
- array
- tuple
- enum
- any
- null undefined
- void
- never
- object

#### array

方式1：元素类型后面跟上[]

```jsx
 let arr:string[] = ['12', '23'];
 arr = ['45', '56'];
```

方式2：使用数组泛型，Array<元素类型>

```jsx
let arr: Array<number> = [1, 2]
```

#### tuple

元组类型，允许表示一个已知元素数量和类型的数组，各元素的类型不必相同

```jsx
let tupleArr:[number, string, boolean];
 tupleArr = [12, '34', true]; //ok
 typleArr = [12, '34'] // no ok
```

#### enum

对JS标准数据类型的一个补充，使用枚举类型可以为一组数值赋予友好的名字

```jsx
enum Color {Red, Green, Blue}
let c: Color = Color.Green
```

#### null undefined

默认情况下null和undefined是所有类型的子类型，可以把null和undefined赋给number类型的变量

但是ts配置了--strictNullChecks标记的话，null和undefined就只能付给void和他们各自

#### void

用于标识方法返回值的类型，表示该方法没有返回值

```jsx
function hello(): void {
  alert("Hello Runoob");
}
```

#### never

never是其他类型（包括null和undefined）的子类型，可以赋值给任何类型，代表从不会出现的值

没有类型是never的子类型，never的变量只能被never类型所赋值

never类型一般用来指定那些总是会抛出异常·无限循环

```jsx
let a: never;
a = 123; // 错误
a = (() => {
  // 正确
  throw new Error(" ");
})();
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
  throw new Error(message);
}
```

### 高级类型

- 交叉类型
- 联合类型
- 类型别名
- 类型索引
- 类型约束
- 映射类型
- 条件类型

#### 交叉类型

通过&将多个类型合并为一个类型，包含了所需要的所有类型的特性，本质上是一种并的操作：T & U

适合于对象合并场景，将两个对象合并成一个对象返回

```tsx
function extend<T , U>(first: T, second: U) : T & U {
  let result: <T & U> = {}
  for (let key in first) {
    result[key] = first[key]
  }
  for (let key in second) {
    if(!result.hasOwnProperty(key)) {
      result[key] = second[key]
    }
  }
  return result
}
```

#### 联合类型

语法逻辑和或一致，表示其类型为连接的多个类型中的任意一个，本质上是一个交的关系： T | U

```jsx
function formatCommandline(command: string[] | string) {
  let line = "";
  if (typeof command === "string") {
    line = command.trim();
  } else {
    line = command.join(" ").trim();
  }
}
```

#### 类型别名

类型别名会给一个类型起一个新名字，类型别名有时候和接口很像，但是可以用作于原始值、联合类型、元组以及其他任何你需要手写的类型

可以使用 type SomeName = someValidTypeAnnotation的语法来创建类型别名

```jsx
 type some = boolean | string
 const b: some = true // ok
 const c: some = 'hello' // ok
 const d: some = 123 // 不能将123 分配给类型“some”
```

此外类型别名可以是泛型

```jsx
type Container<T> = { value: T }
```

也可以使用类型别名在属性中引用自己

```jsx
type tree<T> = {
  value: T;
  left: Tree<T>;
  right: Tree<T>;
}
```

类型别名和接口使用十分类似，都可以描述一个对象或者函数

最大的区别在于interface只能用于定义对象类型，type处了对象之外还可以定义交叉、联合、原始类型等，类型声明的方式适用范围更加广泛

#### 类型索引

keyof类似于Object.keys，用于获取一个接口中Key的联合类型

```jsx
interface Button {
  type: string;
  text: string;
}

type ButtonKeys = keyof Button
type ButtonKeys = 'type' | 'text'
```

#### 类型约束

通过关键字extend进行约束，不同于在class后使用extends的继承作用，泛型内使用的主要作用是对泛型加以约束

```jsx
type BaseType = string | number | boolean
// 这里表示copy的参数只能是字符串、数字、布尔
function copy<T extends BaseType>(arg: T): T {
  return arg
}
```

类型约束通常和类型索引一起使用，比如有一个方法专门用来获取对象的值，但是这个对象并不确定，我们就使用extends和keyof进行约束

```jsx
 function getValue<T, K extends keyof T>(obj: T, key: K) {
 	 return obj[key]
 }
 const obj = { a: 1 }
 const a = getValue(obj, 'a')
```

#### 映射类型

通过in关键字做类型的映射，遍历已有的接口的key或者遍历联合类型

```jsx
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
interface Obj {
  a: string;
  b: string;
}
type ReadOnlyObj = Readonly<Obj>;
```

- keyof T: 通过类型索引keyof得到联合类型a | b
- P in keyof T等同于P in a | b，相当于执行了一次forEach逻辑，遍历a | b

最终ReadOnlyObj为：

```jsx
interface ReadOnlyObj {
  readonly a: string;
  readonly b: string;
}
```

#### 条件类型

语法和三元表达式一致，经常用于一些不确定情况

```jsx
T extends U ? X : Y
```

如果T是U的子集，则是类型X，否则是类型Y

### 接口

接口是一系列抽象方法的声明，是一些方法特征的集合，这些方法都应该是抽象的，需要由具体的类去实现，然后第三方就可以通过这组抽象方法调用，让具体的类执行具体的方法

简单来讲，一个接口描述的是一个对象相关的属性和方法，但并不提供具体创建此对象实例的方法

typescript的核心功能之一就是对类型做检测，虽然这种检测方式是鸭式辩形法，而接口的作用就是为这些类型命名和为代码或第三方代码定义一个约定

```jsx
interface User {
   name: string
   age: number
 }
 const getUserName = (user: User) => user.name
```

上述传入的对象必须拥有name和age属性，否则typescript在编译阶段会报错。如果不想要age属性的话可以采用可选属性：

```jsx
 interface User {
   name: string
   age?: number
 }
```

这时候age属性就可以使number或者undefined

有时候想要一个属性为只读，可以使用readonly声明

```jsx
interface User {
   name: string
   age?: number
   readonly isMale: boolean
 }
```

- 类型推断

  ```jsx
   interface User {
      name: string
      age: number
   }
   const getUserName = (user: User) => user.name
   getUserName({color: 'yellow'} as User)
  ```

- 给接口添加字符串索引签名

  ```jsx
  interface User {
     name: string
     age: number
     [propName: string]: any;
   }
  ```

- 接口继承

  ```jsx
   interface Father {
   	 color: String
   }
   interface Mother {
     height: Number
   }
   interface Son extends Father,Mother{
     name: string
     age: Number
   }
  ```

### 类

类是面对对象程序设计实现信息封装的基础，类是一种用户定义的引用数据类型，也称类类型

传统的面向对象语言基本都是基于类的，JS基于原型的方式让开发者多了很多理解成本。在ES6之后，JavaScript拥有了class关键字，虽然本质依然是构造函数，但是使用起来已经方便了很多

但是JS的calss依然有些特性还没有加入，比如修饰符和抽象类

TS的class支持面向对象的所有特性，比如类、接口等

#### 使用方式

定义类的关键字为class，后面紧跟类名，类可以包含以下几个模块：

- 字段：类里面声明的变量。字段表示对象的有关数据
- 构造函数：类实例化时调用，可以为类的对象分配内存
- 方法：方法为对象要执行的操作

```jsx
class Car {
  // 字段
  engine: string;
  // 构造函数
  constructor(engine: string) {
    this.engine = engine;
  }
  // 方法
  disp():void {
    console.log(" :   "+this.engine)
  }
}
```

##### 继承

类的继承使用extends关键字

```jsx
class Animal {
  move(distanceInMeters: number = 0) {
    console.log(`Animal moved ${distanceInMeters}m.`);
  }
}
class Dog extends Animal {
  bark() {
    console.log("Woof! Woof!");
  }
}
const dog = new Dog();
dog.bark();
dog.move(10);
dog.bark();
```

Dog是一个派生类，派生自Animal基类，派生类通常被称作子类，基类通常被称作超类

Dog继承了Animal类，因此实例Dog也能够使用Animal类的move方法

类继承后，子类可以对父类的方法重新定义，这个过程称之为方法的重写，通过super关键字是对父类的直接引用，该关键字可以引用父类的属性和方法

```jsx
class PrinterClass {
  doPrint(): void {
    console.log(" doPrint() ");
  }
}
class StringPrinter extends PrinterClass {
  doPrint(): void {
    super.doPrint(); // 调用父类的构造函数
    console.log(" doPrint() ");
  }
}
```

##### 修饰符

typescript在此基础上添加了三种修饰符

- 公共public：可以自由的访问类程序里定义的成员
- 私有private：只能够在该类的内部进行访问
- 受保护protect：除了在该类的内部可以访问，还可以在子类中仍然可以访问

###### 私有修饰符

只能够在该类的内部进行访问，实例对象并不能够访问

```jsx
class Father {
  private name: string
  constructor(name: string) {
    this.name = name
  }
}

const father = new Father('huihui')

father.name // 会报错，name为私有属性，只能在类Father中访问
```

并且继承该类的子类并不能访问，如下图所示：

```jsx
class Father {
  private name: String
  constructor(name: String) {
    this.name = name
  }
}

class Son extends Father {
  say() {
    console.log(`my name is ${this.name}`) // 属性name为私有属性，只能在类Father中访问
  }
}
```

###### 受保护修饰符

跟私有修饰符很相似，实例对象同样不能访问受保护的属性，但protected成员在子类中仍然可以访问：

```jsx
class Father {
  protected name: String
  constructor(name: String) {
    this.name = name
  }
}

class Son extends Father {
  say() {
    console.log(`my name is ${this.name}`)
  }
}
```

除了上述修饰符之外，还有只读修饰符

###### 只读修饰符

通过readonly关键字进行声明，只读属性必须在声明时或构造函数里被初始化

```jsx
class Father {
  readonly name: String
  constructor(name: String) {
    this.name = name
  }
}

const father = new Father('huihui')

father.name = 'change' // 无法分配到name，因为它是只读属性
```

除了实例属性之外，同样存在静态属性

readonly可以防止变量的属性被修改，比如readonly Array声明的数组不能使用push，pop等方法

##### 静态属性

这些属性存在于类本身上面而不是类的实例上，通过static进行定义，访问这些属性需要通过 类型.静态属性 这种形式访问：

```jsx
class Square {
  static width = '100px'
}

console.log(Square.width) // 100px
```

上述的类都能发现一个特点，都能够被实例化，在typescript中，还存在一种抽象类

##### 抽象类

抽象类作为其他派生类的基类使用，他们一般不会直接被实例化，不同于接口，抽象类可以包含成员的实现细节

abstract关键字是用于定义抽象类和在抽象类内部定义的抽象方法，如下所示：

```jsx
abstract class Animal {
  abstruct makeSound(): void
  move(): void {
    console.log('roaming the earch...')
  }
}
```

这种类并不能被实例化，通常需要我们创建子类去继承，如下：

```jsx
class Cat extends Animal {
  makeSound() {
    console.log("miao miao");
  }
}
const cat = new Cat();
cat.makeSound(); // miao miao
cat.move(); // roaming the earch...
```

#### 应用场景

除了日常借助类的特性完成日常业务代码，还可以将类(class)作为接口，尤其在React工程中是很常用的，如下：

```jsx
export default class Carousel extends React.Component<Props, State> {}
```

由于组件需要传入props的类型Props，同时还需要设置默认props即defaultProps，这时候更加适合使用class作为接口

先声明一个类，这个类包含组件props所需的类型和初始值：

```jsx
// props的类型
export default class Props {
  public children: Array<React.ReactElement<any>> | React.ReactElement<any> | never[] = []
	public speed: number = 500
	public height: number = 160
	public animation: string = 'easeInOutQuad'
	public isAuto: boolean = true
	public autoPlayInterval: number = 4500
	public afterChange: () => {}
  public beforeChange: () => {}
  public selectedColor: string
  public showDots: boolean = true
}
```

当我们需要传入props类型的时候直接将Props作为接口传入，此时Props的作用就是接口，当我们需要设置defaultProps初始值的时候，我们只需要：

```jsx
public static defaultProps = new Props()
```

Props的实例就是defaultProps的初始值，这就是class作为接口的实际应用，我们用一个class起到了接口和设置初始值两个作用，方便统一管理，减少了代码量

### 枚举

枚举是一个被命名的整型常数集合，同于声明一组命名的常数，当一个变量有几种可能的取值时，可以将它定义为枚举类型

通俗来说，枚举就是一个对象的所有可能取值的集合

在日常生活中也很常见，例如表示星期的SUNDAY，MONDAY，TUESDAY，WEDNESDAY。。。就可以看作一个枚举

枚举的说明与结构和联合相似

```jsx
enum 枚举名 {
  标识符1 = 枚举值1,
  标识符2 = 枚举值2,
  ...
  标识符N = 枚举值N,
}
```

#### 数字枚举

当我们声明一个枚举类型时，虽然没有给他们赋值，但是它们的值其实是默认的数字类型，而且默认从0开始依次累加：

```jsx
enum Direction {
  Up, //  默认值为0
  Down, //  默认值为1
  Left, //  默认值为2
  Right, //  默认值为3
}
console.log(Direction.Up === 0); // true
console.log(Direction.Down === 1); // true
console.log(Direction.Left === 2); // true
console.log(Direction.Right === 3); // true
```

如果将第一个值赋值后，后面的值也会根据前一个值累加1：

```jsx
enum Direction {
  Up = 10,
  Down,
  Left,
  Right,
}
console.log(Direction.Up, Direction.Down, Direction.Left, Direction.Right);
// 10 11 12 13
```

#### 字符串枚举

```jsx
enum Direction {
  Up = "Up",
  Down = "Down",
  Left = "Left",
  Right = "Right",
}
console.log(Direction["Right"], Direction.Up); // Right Up
```

如果设定了一个变量为字符串后，后续的字段也需要赋值字符串，否则报错

#### 异构枚举

即将数字枚举和字符串枚举混合起来使用

```jsx
enum BooleanLikeHeterrogenousEnum {
  No = 0
  Yes = 'YES'
}
```

#### 枚举本质

```jsx
enum Direction {
  Up,
  Down,
  Left,
  Right,
}
```

编译后的结果：

```jsx
var Direction;
(function (Direction) {
  Direction[(Direction["Up"] = 0)] = "Up";
  Direction[(Direction["Down"] = 1)] = "Down";
  Direction[(Direction["Left"] = 2)] = "Left";
  Direction[(Direction["Right"] = 3)] = "Right";
})(Direction || (Direction = {}));
```

Direction[Direction["Up"] = 0] = "Up"可以分成

- Direction['up'] = 0
- Direction[0] = 'up'

定义枚举类型后，可以通过正反映射拿到对应的值

```jsx
 console.log(Direction.Up === 0); // true
 console.log(Direction[0]); // Up
```

多处定义枚举时可以进行合并操作

```jsx
enum Direction {
  Up = "Up",
  Down = "Down",
  Left = "Left",
  Right = "Right",
}
enum Direction {
  Center = 1,
}
```

编译后：

```jsx
var Direction;
(function (Direction) {
  Direction["Up"] = "Up";
  Direction["Down"] = "Down";
  Direction["Left"] = "Left";
  Direction["Right"] = "Right";
})(Direction || (Direction = {}));
(function (Direction) {
  Direction[(Direction["Center"] = 1)] = "Center";
})(Direction || (Direction = {}));
```

可以看到Direction对象属性会叠加

### 函数

typescript为javascript函数添加了额外的功能，丰富了更多应用场景

在没有提供函数实现的情况下，有两种声明函数类型的方式，如下所示：

```jsx
type LongHand = {
  (a: number): number
}

type ShortHand = (a: number) => number
```

#### 可选参数

在参数后面加上？代表参数可能不存在

```jsx
const add = (a: number, b?: number) => a + (b ? b : 0)
```

#### 剩余类型

剩余参数与JS的语法相似，需要使用...来表示剩余参数

如果剩余参数rest是由number类型组成的数组，则如下表示：

```jsx
const add = (a: number, ...rest: number[]) => rest.reduce(((a, b) => a + b), a)
```

#### 函数重载

允许创建数项名称相同但输入输出类型或个数不同的子程序，它可以简单地称为一个单独功能可以执行多项任务的能力

关于typescript函数重载，必须要把精确的定义放在前面，最后函数实现时，需要使用|操作符或者？操作符，把所有可能的输入类型包含进去，用于具体实现

这里的函数重载也只是多个函数的声明，具体的逻辑还需要自己去写，typescript不会真的将多个重名function的函数体进行合并

例如我们有一个add函数，它可以接受string类型的参数进行拼接，也可以接受number类型的参数进行相加

```jsx
function add (arg1: string, arg2: string): string
function add (arg1: number, arg2: number): number
function add (arg1: string | number, arg2: string | number) {
  if (typeof arg1 === 'string' && typeof arg2 === 'string') {
    return arg1 + arg2
  } else if (typeof arg1 === 'number' && typeof arg2 === 'number') {
    return arg1 + arg2
  }
}
```

#### 区别

- 从定义的方式而言，typescript声明函数需要定义参数类型或者声明返回值类型
- typescript在参数中，添加可选参数供使用者选择
- typescript增添函数重载功能，使用者只需要通过查看函数声明的方式，即可知道函数传递的参数个数以及类型

### 泛型

泛型允许我们在强类型程序设计语言中编写代码时使用一些以后才指定的类型，在实例化时作为参数指明这些类型

```tsx
function returnItem<T>(para: T): T {
  return para
}
```

#### 使用方式

泛型通过<>的形式进行表述，可以声明：

- 函数
- 接口
- 类

##### 函数声明

声明函数的形式如下：

```tsx
function returnItem<T>(para: T): T {
  return para
}
```

定义泛型的时候，可以一次定义多个类型参数，比如我们可以同时定义泛型T和泛型U

```tsx
function swap<T, U>(tuple: [T, U]): [U, T] {
  return [tuple[1], tuple[0]];
}
swap([7, "seven"]); // ['seven', 7]
```

##### 接口声明

```tsx
 interface ReturnItemFn<T> {
 	 (para: T): T
 }
```

##### 类声明

使用泛型声明类的时候，既可以作用于类本身，也可以作用于类的成员函数

```tsx
class Stack<T> {
  private arr: T[] = [];
  public push(item: T) {
    this.arr.push(item);
  }
  public pop() {
    this.arr.pop();
  }
}
```

使用方式如下：

```tsx
const stack = new Stack<number>()
```

如果上述只能传递string和number类型，这时候就可以使用<T extends xx>的方式来实现**约束泛型**

```tsx
type Params = string | number
class Stack<T extends Params> {
  private arr: T[] = [];
  public push(item: T) {
    this.arr.push(item);
  }
  public pop() {
    this.arr.pop();
  }
}
```

如果要设计一个函数，这个函数接受两个参数，一个参数为对象，另一个参数为对象上的属性，我们通过这两个参数返回这个属性的值，这时候就涉及到泛型的索引类型和约束类型共同实现

##### 索引类型、约束类型

索引类型keyof T把传入的对象的属性类型取出，生成一个联合类型，这里的泛型U被约束在这个联合类型中

```tsx
function getValue<T extends object, U extends keyof T>(obj: T, key: U) {
  return obj[key]
}
```

##### 多类型约束

```tsx
interface FirstInterface {
  doSomething(): number
}

interface SecondInterface {
  doSomethingElse(): string
}
```

可以创建一个接口继承上述两个接口，如下

```tsx
interface ChildInterface extends FirstInterface, SecondInterface {}
```

```tsx
class Demo<T extends ChildInterface> {
  private genericProperty: T;
  constructor(genericProperty: T) {
    this.genericProperty = genericProperty;
  }
  useT() {
    this.genericProperty.doSomething();
    this.genericProperty.doSomethingElse();
  }
}
```

### 装饰器

装饰器时一种特殊类型的声明，它能够被附加到类声明，方法，访问符，属性或参数上。是一种在不改变原类和使用继承的情况下，动态地扩展对象功能

装饰器就是一个普通的函数，@expression的形式其实是Object.defineProperty语法糖

expression求值后必须是一个函数，他会在运行的时候被调用，被装饰的声明信息作为参数传入

使用前需要在tsconfig.json文件启动

```tsx
{
  compilerOptions: {
    target: "ES5",
    experimentalDecorators: true,
  },
};

```

ts装饰器使用与js基本一致，可以修饰：

- 类
- 方法/属性
- 参数
- 访问器

#### 类装饰

声明一个函数addAge去给Class的属性age添加年龄

```tsx
function addAge(constructor: Function) {
  constructor.prototype.age = 18;
}

@addAge
class Person {
  name: string;
  age!: number;
  constructor() {
    this.name = 'huihui'
  }
}

let person = new Person()
console.log(person.age) // 18
```

上述代码等同于：

```tsx
Person = addAge(function Person() { ... });
```

当装饰器修饰类的时候，会把构造器传递进去。constructor.prototype.age就是在每个实例化的对象上面添加一个age属性

#### 方法/属性装饰

同样，装饰器可以用于修饰类的方法，这时候装饰器函数接受的参数变成了：

- target: 对象的原型
- propertyKey：方法的名称
- descriptor：方法的属性描述符

可以看到，这三个属性实际就是Object.defineProperty的三个参数，如果是类的属性，则没有传递第三个参数

```tsx
// 声明装饰器修饰方法/属性
function method(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  console.log(target)
  console.log('prop' + propertyKey)
  console.log('desc' + JSON.stringify(descriptor) + '\n\n')
  descriptor.writable = false
}

function property(target: any, propertyKey: string) {
  console.log('target', target)
  console.log('prop' + propertyKey)
}

class Person {
  @property
  name: string;
  constructor() {
    this.name = 'huihui'
  }
  
  @method
  say() {
    return 'instance method'
  }
  
  @method
  static run() {
    return 'static method'
  }
}

const xmz = new Person()

xmz.say = function() {
  return 'edit'
}
```

输出如下

![img](https://s4.51cto.com/oss/202109/10/ebabb8722cabbeab3cb3f6787fea6de6.jpg)

#### 参数装饰

接受三个参数，分别是：

- target：当前对象的原型
- propertyKey：参数的名称
- index：参数数组中的位置

```tsx
function logParameter(target: Object, propertyName: string, index: number) {
  console.log(target);
  console.log(propertyName);
  console.log(index);
}
class Employee {
  greet(@logParameter message: string): string {
    return `hello ${message}`;
  }
}
const emp = new Employee();
emp.greet("hello");

```

#### 访问器装饰

使用起来和方法装饰一致

```tsx
function modification(
  target: Object,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  console.log(target);
  console.log("prop " + propertyKey);
  console.log("desc " + JSON.stringify(descriptor) + "\n\n");
}
class Person {
  _name: string;
  constructor() {
    this._name = "huihui";
  }
  @modification
  get name() {
    return this._name;
  }
}

```

#### 装饰器工厂

如果想要传递参数，使装饰器变成类似工厂函数，只需要在装饰器函数内部再返回一个函数即可

```tsx
function addAge(age: number) {
  return function (constructor: Function) {
    constructor.prototype.age = age;
  };
}
@addAge(10)
class Person {
  name: string;
  age!: number;
  constructor() {
    this.name = "huihui";
  }
}
let person = new Person();
```

#### 执行顺序

当多个装饰器应用于一个声明上，将由上至下依次对装饰器表达式求值，求职的结果会被当作函数，由下至上依次调用

```tsx
function f() {
  console.log("f(): evaluated");
  return function (
    target,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log("f(): called");
  };
}
function g() {
  console.log("g(): evaluated");
  return function (
    target,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log("g(): called");
  };
}
class C {
  @f()
  @g()
  method() {}
}

```

#### 应用场景

- 代码可读性增强了，装饰器命名相当于一个注释
- 在不改变源代码的情况下，对原来的功能进行扩展

### 命名空间

#### 模块

TS和ES2015一样，任何包含顶级import或者export的文件都被当成一个模块

相反的，如果一个文件不带有顶级的import或者export声明，那么它的内容被视为全局可见的

例如我们在一个TS工程下建立文件1.ts，声明变量a

```tsx
const a = 1
```

然后在另一个文件同样声明一个变量a，这时候就会出现错误信息，提示重复a变量，但是所处空间是全局的

如果需要解决这个问题，则通过import或者export引入模块系统即可

```tsx
const a = 10
export default a
```

### 命名空间

命名空间的最明确目的就是解决重名问题

命名空间定义了标识符的可见范围，一个标识符可在多个名字空间中定义，他在不同名字空间的含义是不相干的

```tsx
namespace SomeNameSpaceName {
  export interface ISomeInterfaceName {}
  export class SomeClassName {}
}
```

以上定义了一个命名空间SomeNameSpaceName，如果我们需要在外部调用SomeNameSpaceName中的类和接口，则需要在类和接口添加export，使用的时候使用SomeNameSpaceName.SomeClassName

命名空间本质上是一个对象，作用是将一系列相关的全局变量组织到一个对象的属性

```tsx
namespace Letter {
  export let a = 1;
  export let b = 2;
  export let c = 3;
  // ...
  export let z = 26;
}

```

编译成js会变为：

```tsx
var Letter;
(function (Letter) {
  Letter.a = 1;
  Letter.b = 2;
  Letter.c = 3;
  // ...
  Letter.z = 26;
})(Letter || (Letter = {}));

```

#### 区别

- 命名空间是位于全局命名空间下的一个普通的带有名字的Jacascript对象，使用起来十分容易。但就像其他全局空间污染一样，很难去识别组件之间的依赖关系，尤其在大型应用中
- 像命名空间一样，模块可以包含代码和声明。不同的是模块可以声明它的依赖
- 正常TS开发不推荐使用命名空间，但通常通过d.ts文件标记js库类型的时候使用命名空间，主要作用是给编译器编码的时候参考用

### 工具类型

#### 核心工具类型

##### Partial<T>

将类型T的所有属性变成可选

```tsx
interface User {
  id: number;
  name: string;
}

type PartialUser = Partial<User>;
/* 等价于
{
  id?: number;
  name?: string;
}
*/
```

##### Required<T>

将类型T的所有属性变为必选

```tsx
type RequiredUser = Required<PartialUser>;
// 恢复为原始 User 接口
```

##### Readonly<T>

将类型T的所有属性变成只读

```tsx
type ReadonlyUser = Readonly<User>;
/* 等价于
{
  readonly id: number;
  readonly name: string;
}
*/
```

##### Record<K, T>

构造键类型为K，值的类型为T的对象类型

```tsx
type UserMap = Record<string, User>;
// { [key: string]: User }
```

##### Pick<T, K>

从类型T中选取属性K（字符串字面量或者联合类型）

```tsx
type UserName = Pick<User, 'name'>;
// { name: string }
```

##### Omit<T, K>

从类型T中排除属性K

```tsx
type UserWithoutId = Omit<User, 'id'>;
// { name: string }
```

#### 类型操作工具

##### Exclude<T, U>

从类型T中排除可赋值给U的类型

```tsx
type T = string | number | boolean;
type NumbersAndBools = Exclude<T, string>;
// number | boolean
```

##### Extract<T, U>

从类型T中提取可赋值给U的类型

```tsx
type StringsAndNumbers = Extract<T, string | number>;
// string | number
```

##### NonNullable<T>

从T中排除null和undefined

```tsx
type T = string | null | undefined;
type ValidString = NonNullable<T>;
// string
```

##### Parameters<T>

获取函数类型T的参数类型元组

```tsx
type Fn = (a: number, b: string) => void;
type Params = Parameters<Fn>;
// [a: number, b: string]
```

##### ReturnType<T>

获取函数类型T的返回值类型

```tsx
type Result = ReturnType<typeof Math.random>;
// number
```

#### 高级工具类型

##### Awaited<T>

获取Promise的解析值类型（递归处理嵌套Promise）

```tsx
type P = Promise<Promise<string>>;
type Resolved = Awaited<P>;
// string
```

##### ContructorParamrters<T>

获取构造函数类型的参数类型元组

```tsx
class Person {
  constructor(name: string, age: number) {}
}
type PersonParams = ConstructorParameters<typeof Person>;
// [name: string, age: number]
```

##### InstanceType<T>

获取构造函数类型的实例类型

```tsx
type PersonInstance = InstanceType<typeof Person>;
// Person
```

##### ThisParameterType<T>

```tsx
function sayHello(this: { name: string }) {}
type Context = ThisParameterType<typeof sayHello>;
// { name: string }
```

#### 字符串处理工具

##### Uppercase<S>

将字符串字面量类型转为大写