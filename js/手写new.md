```
function (Fn, ...args) {
  const result = {} //一个新对象过
  if (Fn.prototype != null) { // 该对象的__proto__属性指向该构造函数的原型
    Object.setPrototypeOf(result, Fn.prototype)
  }
  // 将执行上下文（this）绑定到新创建的对象中
  const returnResult = Fn.apply(result, args);
  // 如果构造函数有返回值（对象或函数），那么这个返回值将取代第一步中新创建的对象。
    if ((typeof returnResult === 'object' || typeof returnResult === 'function') && returnResult !== null) {
        return returnResult;
    }
    return result;
}
```

> 手写实现 typeof

```
function typeof(obj){
  return Object.prototype.toString.call(obj).slice(8,-1).toLowerCase();
}
```

> 手写实现 instanceof

```
function instanceOf(left, right) {
  let proto = left.__proto__
  while (true) {
    if (proto === null) return false
    if (proto === right.prototype) {
      return true
    }
    proto = proto.__proto__
  }
}
```

| | typeof 操作符 | instanceof 运算符 | Object.prototype.toString.call() 方法 |
| 优点 | 可识别主要的基本数据类型 | 可识别引用数据类型 | 准确检测所有类型 |
| 缺点 | 不能正确识别 null 与具体的引用数据类型 | 不能判断基本数据类型、无法正确处理多全局对象的类型判断 | IE6 以下无法正确检测 null 与 undefined、无法判断某实例具体由哪种自定义类生成 |
