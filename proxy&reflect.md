
```
const user = {
  name: "alice",
  age: 18
};
const proxy = new Proxy(user, {
  get(target, key) {
    console.log("执行了get方法");
    return target[key];
  },
  set(target, key, value) {
    target[key] = value;
    console.log("执行了set方法");
  },
  has(target, key) {
    return key in target;
  },
  deleteProperty(target, key){
    console.log('执行了delete的捕获器')
    delete target[key]
  },
  ownKeys(target){
    console.log('ownKeys')
    return Object.keys(target)
  },
  defineProperty(target, key, value){
    console.log('defineProperty', target, key, value)
    return true
  },
  getOwnPropertyDescriptor(target, key){
    return Object.getOwnPropertyDescriptor(target, key)
  },
  preventExtensions(target){
    console.log('preventExtensions')
    return Object.preventExtensions(target)
  },
  isExtensible(target){
    return Object.isExtensible(target)
  },
  getPrototypeOf(target){
    return Object.getPrototypeOf(target)
  },
  setPrototypeOf(target, prototype){
    console.log('setPrototypeOf', target, prototype)
    return false
  }
});

console.log(proxy.name);
proxy.name = "kiki";
console.log("name" in proxy);
delete proxy.age
console.log(Object.keys(proxy))
Object.defineProperty(proxy, 'name', {
  value: 'alice'
})
console.log(Object.getOwnPropertyDescriptor(proxy, 'name'))
console.log(Object.preventExtensions(proxy))
console.log(Object.isExtensible(proxy))
console.log(Object.getPrototypeOf(proxy))
Reflect.setPrototypeOf(proxy, {})
```
Proxy里对应捕获器与普通对象的操作和定义是一一对应的

> handler.getPrototypeOf()
Object.getPrototypeOf 方法的捕捉器。

> handler.setPrototypeOf()
Object.setPrototypeOf 方法的捕捉器。

> handler.isExtensible()
Object.isExtensible 方法的捕捉器。

> handler.preventExtensions()
Object.preventExtensions 方法的捕捉器。

> handler.getOwnPropertyDescriptor() Object.getOwnPropertyDescriptor 方法的捕捉器。

> handler.defineProperty()
Object.defineProperty 方法的捕捉器。

> handler.has()
in 操作符的捕捉器。

> handler.get()
属性读取操作的捕捉器。

> handler.set()
属性设置操作的捕捉器。

> handler.deleteProperty()
delete 操作符的捕捉器。

> handler.ownKeys()
Object.getOwnPropertyNames 方法和 Object.getOwnPropertySymbols 方法的捕捉器。

> handler.apply()
函数调用操作的捕捉器。

> handler.construct()
new 操作符的捕捉器

# Reflect

Reflect中的方法与Proxy中是一一对应的

Reflect.apply(target, thisArgument, argumentsList)
对一个函数进行调用操作，同时可以传入一个数组作为调用参数。和 Function.prototype.apply() 功能类似。

Reflect.construct(target, argumentsList[, newTarget])
对构造函数进行 new 操作，相当于执行 new target(…args)。

Reflect.defineProperty(target, propertyKey, attributes)和 Object.defineProperty() 类似。如果设置成功就会返回 true

Reflect.deleteProperty(target, propertyKey)
作为函数的delete操作符，相当于执行 delete target[name]。

Reflect.get(target, propertyKey[, receiver])
获取对象身上某个属性的值，类似于 target[name]。

Reflect.getOwnPropertyDescriptor(target, propertyKey)
类似于 Object.getOwnPropertyDescriptor()。如果对象中存在该属性，则返回对应的属性描述符, 否则返回 undefined.

Reflect.getPrototypeOf(target)
类似于 Object.getPrototypeOf()。

Reflect.has(target, propertyKey)
判断一个对象是否存在某个属性，和 in 运算符 的功能完全相同。

Reflect.isExtensible(target)
类似于 Object.isExtensible().

Reflect.ownKeys(target)
返回一个包含所有自身属性（不包含继承属性）的数组。(类似于 Object.keys(), 但不会受enumerable影响).

Reflect.preventExtensions(target)
类似于 Object.preventExtensions()。返回一个Boolean。

Reflect.set(target, propertyKey, value[, receiver])
将值分配给属性的函数。返回一个Boolean，如果更新成功，则返回true。

Reflect.setPrototypeOf(target, prototype)
设置对象原型的函数. 返回一个 Boolean， 如果更新成功，则返回true。

```
const user = {
  name: "alice",
  age: 18
};
const proxy = new Proxy(user, {
  get(target, key) {
    console.log("执行了get方法");
    return Reflect.get(target, key)
  },
  set(target, key, value) {
    Reflect.set(target, key, value)
    console.log("执行了set方法");
  },
  has(target, key) {
    return Reflect.has(target, key)
  },
  deleteProperty(target, key){
    Reflect.deleteProperty(target, key)
  },
  ownKeys(target){
    console.log('ownKeys')
    return Reflect.ownKeys(target)
  },
  defineProperty(target, key, value){
    console.log('defineProperty', target, key, value)
    return true
  },
  getOwnPropertyDescriptor(target, key){
    return Reflect.getOwnPropertyDescriptor(target, key)
  },
  preventExtensions(target){
    console.log('preventExtensions')
    return Reflect.preventExtensions(target)
  },
  isExtensible(target){
    return Reflect.isExtensible(target)
  },
  getPrototypeOf(target){
    return Reflect.getPrototypeOf(target)
  },
  setPrototypeOf(target, prototype){
    console.log('setPrototypeOf', target, prototype)
    return false
  }
});
console.log(proxy.name);
proxy.name = "kiki";
console.log(Reflect.has(proxy, 'name'));
delete proxy.age
console.log(Reflect.ownKeys(proxy))
Reflect.defineProperty(proxy, 'name', {
  value: 'alice'
})
console.log(Reflect.getOwnPropertyDescriptor(proxy, 'name'))
console.log(Reflect.preventExtensions(proxy))
console.log(Reflect.isExtensible(proxy))
console.log(Reflect.getPrototypeOf(proxy))
Reflect.setPrototypeOf(proxy, {})
```

# Reflect的receiver

Reflect中在进行get/set捕获器操作的时候，还有一个入参是receiver，指的是代理对象，用于改变this指向
```
const user = {
  _name: 'alice',
  get name(){
    return this._name
  },
  set name(value){
    this._name = value
  }
}
const proxy = new Proxy(user, {
  get: function(target, key, receiver){
    console.log('get操作', key)
    return Reflect.get(target, key, receiver)
  },
  set: function(target, key, receiver){
    console.log('set操作', key)
    return Reflect.set(target, key, receiver)
  },
})
console.log(proxy.name)
proxy.name = 'kiki'
```
(1) 如果没有receiver，那么当修改name属性时，objProxy先执行key为name时的get操作
(2) 然后代理到obj里的get方法，读取this的_name属性，此时的this是obj，会直接修改
obj._name，不会再经过objProxy
(3) 增加了receiver之后，执行obj的get方法，读取this的_name属性，此时this是proxy
对象，所以会再次到get的捕获器中


```
const user = {
  _name: 'alice',
  get name(){
    return this._name
  },
  set name(value){
    this._name = value
  }
}
const proxy = new Proxy(user, {
  get: function(target, key, receiver){
    console.log('get操作', key)
    debugger
    return Reflect.get(target, key, receiver)
  },
  set: function(target, key, receiver){
    console.log('set操作', key)
    debugger
    return Reflect.set(target, key, receiver)
  },
})
console.log(proxy.name)
proxy.name = 'kiki'
```

# Reflect中的constructor
```
function Person(){
}
Person.prototype.doit = function () {
  alert('doit')
}
function Student(name, age){
  this.name = name;
  this.age = age
}
const student = Reflect.construct(Student, ['aclie', 18], Person)

console.log(student)
console.log(student.__proto__ === Person.prototype)
```
