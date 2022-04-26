interface：接口，TS 设计出来主要用于定义【对象类型】，可以对【对象】的形状进行描述。

type ：类型别名，为类型创建一个新名称。它并不是一个类型，只是一个别名。
主要区别在于 type 一旦定义就不能再添加新的属性，而 interface 总是可扩展的。

很多时候 interface 和 type 是相同的，但有一个明显区别在于 interface 可以重复定义，类型注解会累加，而 type 重复定义会报错


```
interface User {
  name: string;
}
interface User {
  age: number
}

//合并后

interface User {
  name: string;
  age: number;
}
```

type

```
type StringType = string
let s:StringType
s = 'hello'
```

联合类型

```
type paramType = number | string
let param: paramType
param =1;
param = 'txt'
```

> 联合类型（Union Types），表示取值可以为多种类型中的一种。使用 | 分隔每个类型。
> 这里的 string | number 的含义是，允许param的类型是 string 或者 number，但是不能是其他类型
> 如果需要被 extends 或者 implements, 则尽量使用接口

# 接口

> 对象类型
```
type UserType = { name: string; age?: number };
const user: UserType = {
  name: "kiki",
  age: 18,
};

interface IUserType { name: string; age?: number }
const person: IUserType = {
  name: 'alice',
  age: 20
}
```

> 索引类型
```
interface ILanguage {
  [index: number]: string;
}
const language: ILanguage = {
  0: "html",
  1: "css",
  2: "js",
};

type Score = {
  [name: string]: number;
}
const score: Score = {
  Chinese: 120,
  math: 95,
  Englist: 88,
}
```

> 函数类型
```
interface ISelfType {
  (arg: string): string;
}
type LogType = (arg: string) => string;

function print(arg: string, fn: ISelfType, logFn: LogType) {
  fn(arg);
  logFn(arg);
}
function self(arg: string) {
  return arg;
}
console.log(print("hello", self, self))
```

> 继承
接口可以实现多继承，继承后的接口具备所有父类的类型注解
extends

```
interface ISwim {
  swimming: () => void;
}
interface IEat {
  eating: () => void;
}
interface IBird extends ISwim, IEat {}
const bird: IBird = {
  swimming() {},
  eating() {},
}
```

> 交叉类型
交叉类型其实是与的操作，用 & 符号，将接口进行与操作后，实质上需要满足所有与操作接口的类型注解
```
interface ISwim {
  swimming: () => void;
}
interface IEat {
  eating: () => void;
}
type Fish = ISwim | IEat;
type Bird = ISwim & IEat;

const fish: Fish = {
  swimming() {},
};
const bird: Bird = {
  swimming() {},
  eating() {},
};
export {}
```

> 接口实现
接口可以通过类使用 implements 关键字来实现，类只能继承一个父类，但是可以实现多个接口
class Fish extends Animal implements ISwim, IEat

```
interface ISwim {
  swimming: () => void
}
interface IEat {
  eating: () => void
}
class Animal {}
class Fish extends Animal implements ISwim, IEat {
  swimming(){}
  eating(){}
}
class Person implements ISwim {
  swimming(){}
}
function swimAction(iswim: ISwim){
  iswim.swimming()
}
swimAction(new Fish())
swimAction(new Person())
```

# 字面量赋值
```
interface IPerson{
  name: string;
  age:string;
  hobbies: string[];
}

const person: IPerson = {
  name: 'alice',
  age: 20,
  hobbies: ['swimming'],
  friends: 'kiki' // error  无friends字端 所以报错
}

const user = {
  name: 'alice',
  age: 20,
  hobbies: ['swimming'],
  friends: 'kiki' // error  无friends字端 所以报错
}
const person: IPerson = user 
// 将对象的引用赋值的话，会进行 freshness 擦除操作，类型检测时将多余的属性擦除，如果依然满足类型就可以赋值
```

# 泛型类
```
class Point<T> {
  x: T;
  y: T;
  z: T;
  constructor(x: T, y: T, z: T) {
    this.x = x;
    this.y = y;
    this.z = z;
  }
}
new Point("1.55", "2.34", "3.67");
new Point(1.55, 2.34, 3.67);
```
