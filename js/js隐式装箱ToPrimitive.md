# js隐式装箱ToPrimitive

把对象转换成数字，会先进行ToPrimitive(input,Number)，然后把得到的值进行转换

ToPrimitive(input [, PreferredType])

1.如果没有传入PreferredType参数，则让hint的值为'default'
2.否则，如果PreferredType值为String，则让hint的值为'string'
3.否则，如果PreferredType值为Number，则让hint的值为'number'
4.如果input对象有@@toPrimitive方法，则让exoticToPrim的值为这个方        法，否则让exoticToPrim的值为undefined
5.如果exoticToPrim的值不为undefined，则
	a.让result的值为调用exoticToPrim后得到的值
	b.如果result是原值，则返回
	c.抛出TypeError错误
6.否则，如果hint的值为'default'，则把hint的值重新赋为'number'
7.返回 OrdinaryToPrimitive(input,hint)

OrdinaryToPrimitive(input,hint)

1.如果hint的值为'string',则
	a.调用input对象的toString()方法，如果值是原值则返回
	b.否则，调用input对象的valueOf()方法，如果值是原值则返回
	c.否则，抛出TypeError错误
2.如果hint的值为'number',则
	a.调用input对象的valueOf()方法，如果值是原值则返回
	b.否则，调用input对象的toString()方法，如果值是原值则返回
	c.否则，抛出TypeError错误


  ```
  Array.prototype[Symbol.toPrimitive] = function(hint){
	switch(hint){
		case 'number' :
			return 123;
		case 'string' :
			return 'hello world!';
		case 'default' : 
			return 'default';
		default :
			throw new Error();
	} 
}
```
```
Array.prototype[Symbol.toPrimitive] = function(hint){
	switch(hint){
		case 'number' :
			return 123;
		case 'string' :
			return 'hello world!';
		case 'default' : 
			return 'default';
		default :
			throw new Error();
	} 
}

var arr = [];
arr + 2;//"default2"
arr * 2;//"246"
String(arr);//"hello world!"
```