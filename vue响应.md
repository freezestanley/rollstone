# vue的响应式
定义一个数组收集所有的依赖
定义全局变量reactiveFns用来保存所有的函数
定义方法watchFn，入参为函数，代码体为将入参保存进全局变量中
修改对象的值，遍历全局变量，执行每一个函数
```
let user = {
  name: "alice",
};
let reactiveFns = [];
function watchFn(fn) {
  reactiveFns.push(fn);
}
watchFn(function () {
  console.log("哈哈哈");
});
watchFn(function () {
  console.log("hello world");
});
function foo(){
  console.log('普通函数')
}
user.name = "kiki";
reactiveFns.forEach((fn) => {
  fn();
});
```
此时通过响应式函数 watchFn 将所有需要执行的函数收集进了数组中，然后当变量的值发生变化时，手动遍历执行所有的函数

https://blog.csdn.net/bingbing1128/article/details/121845075?spm=1001.2014.3001.5502