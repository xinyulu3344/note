# let、var和const

## 基本用法

`let`声明的变量只在它所在的代码块有效。

```js
{
  let a = 10;
  var b = 1;
}

console.log(b) // 1
console.log(a) // ReferenceError: a is not defined
```

for循环计数中，使用let，不要用var

```js
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function() {
    console.log(i)
  }
}
for (j = 0; j < a.length; j++) {
  a[j]() // 输出10个10
}
```

```js
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function() {
    console.log(i)
  }
}
for (j = 0; j < a.length; j++) {
  a[j]() // 输出0 1 2 3 ... 9
}
```

for循环条件是父作用域，循环体是一个自作用域

```js
for (let i = 0; i < 3; i++) {
  let i = "hello"
  console.log(i) // 输出3个hello
}
```

## 变量提升

var会变量提升，let不会

```js
// var 的情况
console.log(foo) // 输出undefined
var foo = 2

// let 的情况
console.log(bar) // ReferenceError: Cannot access 'bar' before initialization
let bar = 2
```

## 暂时性死区

如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

```js
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError: Cannot access 'tmp' before initializationReferenceError
  let tmp
}
```

```js
function bar(x = y, y = 2) {
  return [x, y]
}

bar(); // 报错
```



## 重复声明

`let`不允许在相同作用域内，重复声明同一个变量。

```js
// 报错
function func() {
  let a = 10
  var a = 1 // SyntaxError: Identifier 'a' has already been declared
}

// 报错
function func() {
  let a = 10
  let a = 1 // SyntaxError: Identifier 'a' has already been declared
}
```

```js
function func(arg) {
  let arg // SyntaxError: Identifier 'arg' has already been declared
}

function func(arg) {
  {
    let arg // 不报错
  }
}
```

## 块级作用域

```js
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}
```

# 模板字符串

```js
let pname = 'zhangsan'
let hi = `my name is ${pname}`
console.log(hi) // my name is zhangsan
```

# 函数



# 解构赋值

## 数组

```js
// 数组解构赋值
let [a, b, c] = [1, 2, 3]
console.log(a, b, c) // 1 2 3

let [d, [e], f] = [1, [2, 3], 4]
console.log(d, e, f) // 1 2 4

let [g, ...arr] = [2, 3, 4]
console.log(g, arr) // 2 [3, 4]

// 默认值
// 默认值只有当成员是undefined的时候才会生效
let [h, i = 2, ...j] = [1]
console.log(h, i, j) // 1 2 []
```

## 对象

```js
// 对象解构
let { bar } = { foo: 'aaa', bar: 'bbb' }
console.log(bar) // 'bbb'

let { log } = console
log('hello') // 'hello'

// 找到同名属性，赋值给一个变量
let { foo: baz} = { foo: 'aaa', bar: 'bbb'}
console.log(baz) // 'aaa'

// 默认值
let {x: y = 3} = {x: 5}
let {x = 3} = {}
```

