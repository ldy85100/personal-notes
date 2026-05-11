# ECMAScript 常用特性整理（ES6 — ES13）

> **版本称谓说明：** 本文以大家常说的 **ES6、ES7、ES8…** 这种叫法为主，读起来更顺口；括号里附上对应的 **ES20xx** 年份版名称（与官方、MDN 上的年份一致）。**两种叫法指的是同一版标准**，不是两套东西。文末写到 **ES13（ES2022）**；之后还有 **ES14（ES2023）**、**ES15（ES2024）** 等，可查 [TC39](https://tc39.es/process-document/) 或 MDN 以年份为准。

## 一、ES6（亦称 ES2015）

### 1. `let` 和 `const`

在 **ES6** 中，新增了 `let` 和 `const` 关键字，其中 `let` 主要用来声明变量，`const` 用来声明常量。`let`、`const` 相对于 `var` 关键字有以下特点：

| 特性 | var | let | const |
| :--- | :---: | :---: | :---: |
| 变量提升 | √ | × | × |
| 全局变量 | √ | × | × |
| 重复声明 | √ | × | × |
| 重新赋值 | √ | √ | × |
| 暂时性死区 | × | √ | √ |
| 块级作用域 | × | √ | √ |
| 只声明不初始化 | √ | √ | × |

**暂时性死区**

使用 `var` 声明的变量可以在声明之前进行使用，这种特性被称为变量提升：

```javascript
console.log(a); // 输出结果: undefined
var a = 1;
```

在 **ES6** 中，`let` 和 `const` 限制了变量提升，在变量声明之前不可使用，称为 **暂时性死区**：

```javascript
a = "a"; // Uncaught ReferenceError: Cannot access 'a' before initialization
let a;
console.log(a); // undefined
```

**块级作用域**

在引入 `let` 和 `const` 之前不存在块级作用域，会导致诸多问题。

- 内层变量覆盖外层的同名变量：

```javascript
var a = 1;
if (true) {
  var a = 2;
}
console.log(a); // 2
```

- 循环变量泄露为全局变量；`setTimeout` 与 `var` 的经典问题：

```javascript
var array = [1, 2, 3];
for (var i = 0; i < array.length; i++) {
  console.log(i); // 0 1 2
}
console.log(i); // 3

var array = [1, 2, 3];
for (var i = 0; i < array.length; i++) {
  setTimeout(function () {
    console.log(i); // 3 3 3
  }, 0);
}
```

使用 `let` 和 `const` 的变量存在块级作用域，可解决上述问题：

```javascript
let a = 1;
if (true) {
  let a = 2;
}
console.log(a); // 1

const array = [1, 2, 3];
for (let i = 0; i < array.length; i++) {
  console.log(i); // 0 1 2
}
console.log(i); // Uncaught ReferenceError: i is not defined

const array = [1, 2, 3];
for (let i = 0; i < array.length; i++) {
  setTimeout(function () {
    console.log(i); // 0 1 2
  }, 0);
}
```

### 2. 解构赋值

解构赋值语法是一种 JavaScript 表达式，可以将数组中的值或对象的属性取出，赋值给其他变量。

```javascript
// 数组
const array = [1, 2, 3, 4, 5];
const [a, b] = array; // a is 1, b is 2
const [a, , b] = array; // a is 1, b is 3
const [a, b, ...rest] = array; // a is 1, b is 2, rest is [3, 4, 5]

// 对象
const obj = { a: 1, b: 2, c: 3 };
const { a, b } = obj; // a is 1, b is 2
const { a: a1, b: b1 } = obj; // a1 is 1, b1 is 2
const { d: d1 = "ddefault", ...rest } = obj; // d1 is 'ddefault', rest is {a: 1, b: 2, c: 3}
```

### 3. 模板字符串

模板字符串是增强版字符串，用反引号 `` ` `` 标识，可定义单行/多行字符串，或在字符串中嵌入变量。优点：避免传统字符串拼接的繁琐写法。

```javascript
`string text`;

`string text line1
string text line2`;

`string text ${expression} string text`;
```

### 4. 函数默认参数

允许函数在没有值或 `undefined` 被传入时使用默认形参：

```javascript
function add(a, b = 1) {
  return a + b;
}
console.log(add(1)); // 2
console.log(add(1, 2)); // 3
```

### 5. 箭头函数

箭头函数语法更简洁，没有自己的 `this`、`arguments`、`super` 或 `new.target`，不能用作构造函数。

**语法：**

```javascript
(param1, param2, ..., paramN) => { statements };
(param1, param2, ..., paramN) => expression; // 相当于 return expression

// 只有一个参数时，圆括号可以省略
(singleParam) => { statements };
singleParam => { statements };

// 支持剩余参数
(...rest) => { statements };
```

**（1）更简洁**

```javascript
const array = ["Hydrogen", "Helium", "Lithium", "Beryllium"];
array.map(function (element) {
  return element.length;
});
// 可改写为
array.map((element) => element.length);
```

**（2）没有单独的 `this`**

箭头函数不会创建自己的 `this`，从作用域链上一层继承 `this`，在定义时确定、之后不变。

```javascript
// 普通函数
function Person() {
  var that = this;
  that.age = 0;
  setInterval(function growUp() {
    that.age++;
  }, 1000);
}

// 箭头函数
function Person() {
  this.age = 0;
  setInterval(() => {
    this.age++; // this 指向 p 实例
  }, 1000);
}
var p = new Person();
```

通过 `call`、`apply`、`bind` 不能改变箭头函数中 `this` 的指向：

```javascript
var id = "Global";
let fun1 = () => {
  console.log(this.id);
};
fun1(); // 'Global'
fun1.call({ id: "Obj" }); // 'Global'
fun1.apply({ id: "Obj" }); // 'Global'
fun1.bind({ id: "Obj" })(); // 'Global'
```

**（3）不绑定 `arguments`**

在箭头函数中访问 `arguments` 实际得到的是外层函数的 `arguments`。多数情况下更推荐使用剩余参数：

```javascript
function foo(arg) {
  var f = (...args) => args[0];
  return f(arg);
}
foo(1); // 1

function foo(arg1, arg2) {
  var f = (...args) => args[1];
  return f(arg1, arg2);
}
foo(1, 2); // 2
```

**（4）不可作为构造函数**

```javascript
var Foo = () => {};
var foo = new Foo(); // TypeError: Foo is not a constructor
```

### 6. 扩展运算符（Spread）

扩展运算符可在函数调用/数组构造时展开数组或字符串；对象上的展开在 **ES9** 中进一步规范（见「四、ES9」）。

```javascript
function sum(x, y, z) {
  return x + y + z;
}
const numbers = [1, 2, 3];
console.log(sum(...numbers)); // 6
console.log(sum.apply(null, numbers)); // 6
console.log(sum.call(null, ...numbers)); // 6
```

### 7. `Symbol`

**ES6** 引入新的基本数据类型，表示独一无二的值，可作对象属性键：

```javascript
const symbol1 = Symbol();
const symbol2 = Symbol(42);
const symbol3 = Symbol("foo");
console.log(typeof symbol1); // symbol
console.log(symbol2 === 42); // false
console.log(symbol3.toString()); // Symbol(foo)
console.log(Symbol("foo") === Symbol("foo")); // false
```

### 8. 集合 `Set`

`Set` 是值的集合，元素唯一。

| 属性和方法 | 概述 |
| :--- | :--- |
| `size` | 元素个数 |
| `add` | 添加元素，返回当前集合 |
| `delete` | 删除元素，返回布尔值 |
| `has` | 是否包含某元素 |
| `clear` | 清空 |

可利用唯一性去重：

```javascript
let arr = [1, 2, 3, 1, 2];
Array.from(new Set(arr)); // [1, 2, 3]
```

### 9. `Map`

`Map` 保存键值对，记住键的插入顺序；任意值可作键或值。

| 属性和方法 | 概述 |
| :--- | :--- |
| `size` | 元素个数 |
| `set` | 添加键值对 |
| `get` | 按键取值 |
| `has` | 是否包含 |
| `clear` | 清空 |

### 10. 模块化（ES Module）

**ES6** 起，原生 **ES Module** 将文件视为模块，核心是导出与导入。

**`export` 导出**

```javascript
// 方式一
export const first = "test";
export function func() {
  return true;
}

// 方式二
const first = "test";
const second = "test";
function func() {
  return true;
}
export { first, second, func };
```

使用 `as` 重命名导出：

```javascript
var first = "test";
export { first as second };
```

`export default`：默认导出，导入时可任意命名；默认导出只能使用一次。

```javascript
export default function () {
  console.log("foo");
}
// 导入
import defaultExport from "./module-name";
```

**`import` 导入**

```javascript
import { export1, export2, export3 } from "./module-name";
import { export1 as alias1 } from "./exportFile.js";
```

注意：`import` 会提升到模块顶部；不能使用表达式或变量作为模块说明符。

```javascript
import * as myModule from "/modules/my-module.js";
import "/modules/my-module.js"; // 仅执行副作用
import myDefault from "/modules/my-module.js";
```

### 11. 字符串方法

**（1）`includes()`** — 区分大小写查找子串：

```javascript
includes(searchString);
includes(searchString, position);
```

- `searchString`：要搜索的字符串（不能是正则）
- `position`：可选，起始位置，默认 `0`

```javascript
let str = "Hello world!";
str.includes("o"); // true
str.includes("z"); // false
str.includes("e", 2); // false
```

**（2）`startsWith()`** — 是否以给定子串开头：

```javascript
startsWith(searchString);
startsWith(searchString, position);
```

```javascript
let str = "Hello world!";
str.startsWith("Hello"); // true
str.startsWith("Helle"); // false
str.startsWith("wo", 6); // true
```

**（3）`endsWith()`** — 是否以指定字符串结尾：

```javascript
endsWith(searchString);
endsWith(searchString, endPosition);
```

- `endPosition`：可选，查找范围的结束位置（即 `searchString` 最后一个字符索引 + 1），默认 `str.length`

**（4）`repeat()`** — 重复拼接为新字符串：

```javascript
const mood = "Happy! ";
console.log(`I feel ${mood.repeat(3)}`); // "I feel Happy! Happy! Happy! "
```

### 12. 数组方法

**（1）`reduce()`**

对数组元素依次执行 reducer，将上一次结果传入下一次调用，最终得到一个返回值。

```javascript
reduce(callbackFn, [initialValue]);
```

`callbackFn` 接收：`previousValue`、`currentValue`、`index`、`array`。`initialValue` 作为首次调用的上一值。

无初始值时，从索引 `1` 开始执行回调；有初始值时从索引 `0` 开始。

```javascript
let arr = [1, 2, 3, 4];
let sum = arr.reduce((prev, cur, index, arr) => {
  console.log(prev, cur, index);
  return prev + cur;
});
console.log(arr, sum);
// 1 2 1
// 3 3 2
// 6 4 3
// [1, 2, 3, 4] 10

let sum2 = arr.reduce((prev, cur, index, arr) => {
  console.log(prev, cur, index);
  return prev + cur;
}, 5);
// 5 1 0
// 6 2 1
// 8 3 2
// 11 4 3
// [1, 2, 3, 4] 15
```

**常见用途：**

```javascript
// 对象数组求和（需提供 initialValue）
const objects = [{ x: 1 }, { x: 2 }, { x: 3 }];
const sum = objects.reduce((acc, cur) => acc + cur.x, 0);
console.log(sum); // 6

// 展开嵌套数组
const flattened = [[0, 1], [2, 3], [4, 5]].reduce(
  (acc, cur) => acc.concat(cur),
  []
);
// [0, 1, 2, 3, 4, 5]

// 统计出现次数
const names = ["Alice", "Bob", "Tiff", "Bruce", "Alice"];
const countedNames = names.reduce((allNames, name) => {
  const currCount = allNames[name] ?? 0;
  return { ...allNames, [name]: currCount + 1 };
}, {});
// { Alice: 2, Bob: 1, Tiff: 1, Bruce: 1 }
```

**（2）`filter()`**

用回调过滤，`true` 的元素进入新数组，不改变原数组。

```javascript
let arr = [1, 2, 3, 4, 5];
arr.filter((item) => item > 2);
```

移除假值（注意：`filter(Boolean)` 会去掉 `0`、`''`、`false` 等）：

```javascript
let arr = [1, undefined, 2, null, 3, false, "", 4, 0];
arr.filter(Boolean);
// 结果: [1, 2, 3, 4]
```

**（3）`Array.from()`**

从类数组或可迭代对象快速生成新数组；有迭代器即可转换。返回新数组，不修改原对象。

参数：① 类数组对象（必填）；② 映射函数（可选）；③ 映射函数的 `this`（可选）。

```javascript
Array.from("abc"); // ["a", "b", "c"]
Array.from(new Set(["abc", "def"])); // ["abc", "def"]
Array.from(
  new Map([
    [1, "ab"],
    [2, "de"],
  ])
); // [[1, 'ab'], [2, 'de']]
```

**（4）`fill()`**

用同一值填充数组的全部或部分区间。

```javascript
array.fill(value, start, end);
```

```javascript
const arr = [0, 0, 0, 0, 0];
arr.fill(5);
console.log(arr); // [5, 5, 5, 5, 5]
arr.fill(0);

arr.fill(5, 3);
console.log(arr); // [0, 0, 0, 5, 5]
arr.fill(0);

arr.fill(5, 1, 3);
console.log(arr); // [0, 5, 5, 0, 0]
arr.fill(0);

arr.fill(5, -1);
console.log(arr); // [0, 0, 0, 0, 5]
```

### 13. `Promise`

`Promise` 是异步编程的一种解决方案。

**特点：**

1. 状态不受外界影响：`pending`、`fulfilled`、`rejected` 仅由异步结果决定。
2. 状态一旦变为 `fulfilled` 或 `rejected` 即定型（resolved）；之后再注册回调也会立即得到结果（与事件不同）。

**用法：**

```javascript
const promise = new Promise(function (resolve, reject) {
  if (/* 异步操作成功 */) {
    resolve(value);
  } else {
    reject(error);
  }
});
```

**常用实例方法：**

- **`then`**：状态变为已解决时执行；返回**新**的 `Promise`，可链式调用。

```javascript
getJSON("/posts.json")
  .then(function (json) {
    return json.post;
  })
  .then(function (post) {
    // ...
  });
```

- **`catch`**：`rejected` 或抛错时执行。

- **`Promise.all([p1, p2, p3])`**：全部 `fulfilled` 才 `fulfilled`（结果为数组）；任一 `rejected` 则 `rejected`（取首个失败原因）。

```javascript
const p = Promise.all([p1, p2, p3]);
```

- **`Promise.race([p1, p2, p3])`**：任一率先落定，`p` 跟随其状态与结果。

---

## 二、ES7（亦称 ES2016）

### 1. `Array.prototype.includes()`

判断数组是否包含指定值，不改变原数组。

```javascript
arr.includes(searchElement, fromIndex);
```

- `searchElement`：必填。
- `fromIndex`：可选；负值时从 `array.length + fromIndex` 起算，仍向前搜索。默认 `0`。

```javascript
[1, 2, 3].includes(2); // true
[1, 2, 3].includes(4); // false
[1, 2, 3].includes(3, 3); // false
[1, 2, 3].includes(3, -1); // true
```

### 2. 指数运算符 `**`

与 `Math.pow()` 等效：

```javascript
Math.pow(2, 10); // 1024
2 ** 10; // 1024
```

---

## 三、ES8（亦称 ES2017）

### 1. `async` 和 `await`

`async` 函数中可使用 `await`，以更简洁的方式表达基于 `Promise` 的异步逻辑。

```javascript
function resolveAfter2Seconds() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve("resolved");
    }, 2000);
  });
}

async function asyncCall() {
  console.log("calling");
  const result = await resolveAfter2Seconds();
  console.log(result); // resolved
}
asyncCall();
```

### 2. `padStart()` 和 `padEnd()`

用于将字符串补齐到指定长度。

```javascript
"x".padStart(1, "ab"); // x
"x".padStart(2, "ab"); // ax
"x".padStart(4, "ab"); // abax
"x".padStart(5, "ab"); // ababx

"x".padEnd(1, "ab"); // x
"x".padEnd(2, "ab"); // xa
"x".padEnd(4, "ab"); // xaba
"x".padEnd(5, "ab"); // xabab
```

### 3. `Object.values()` 和 `Object.entries()`

与 `Object.keys()` 配套，遍历对象自身可枚举属性（不含继承与 `Symbol`），顺序与常规遍历一致。

- `Object.keys()`：键名数组  
- `Object.values()`：键值数组  
- `Object.entries()`：`[key, value]` 对组成的数组  

```javascript
let obj = { id: 1, name: "hello", age: 18 };
console.log(Object.keys(obj)); // ['id', 'name', 'age']
console.log(Object.values(obj)); // [1, 'hello', 18]
console.log(Object.entries(obj)); // [['id', 1], ['name', 'hello'], ['age', 18]]
```

注意：`Object.keys()` 返回的键均为字符串；结果均不含继承属性。

### 4. 尾后逗号

函数参数列表末尾允许逗号，便于 Git 协作时减少无意义 diff。

```javascript
function person(name, age, sex,) {}
```

---

## 四、ES9（亦称 ES2018）

### 1. `for await...of`

异步迭代器，用于遍历异步或同步可迭代对象（`String`、`Array`、类数组、`Map`、`Set` 及自定义异步/同步可迭代对象）。**仅能在 `async function` 内使用。**

```javascript
function Gen(time) {
  return new Promise((resolve, reject) => {
    setTimeout(function () {
      resolve(time);
    }, time);
  });
}

async function test() {
  let arr = [Gen(2000), Gen(100), Gen(3000)];
  for await (let item of arr) {
    console.log(Date.now(), item);
  }
}
test();
```

### 2. `Promise.prototype.finally`

无论成功或失败都会执行；**不接受参数**，通常做与状态无关的清理。

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("1");
  }, 1000);
});
promise
  .then(() => console.log("success"))
  .catch(() => console.log("fail"))
  .finally(() => console.log("finally"));
// fail
// finally
```

### 3. 对象的扩展运算符

**ES6** 中的扩展运算符主要针对数组；**ES9** 起可用于对象。

**（1）剩余属性：**

```javascript
const obj = { a: 1, b: 2, c: 3 };
const { a, ...rest } = obj;
console.log(rest); // {b: 2, c: 3}

(function ({ a, ...obj }) {
  console.log(obj); // {b: 2, c: 3}
})({ a: 1, b: 2, c: 3 });
```

**（2）对象展开：**

```javascript
const obj = { a: 1, b: 2, c: 3 };
const newObj = { ...obj, d: 4 };
console.log(newObj); // {a: 1, b: 2, c: 3, d: 4}
```

**（3）合并对象：**

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
const mergedObj = { ...obj1, ...obj2 };
console.log(mergedObj); // {a: 1, b: 2, c: 3, d: 4}
```

**（4）函数参数中的 rest：**

```javascript
function func({ param1, ...rest }) {
  return rest;
}
console.log(func({ param1: 1, b: 2, c: 3, d: 4 })); // {b: 2, c: 3, d: 4}
```

---

## 五、ES10（亦称 ES2019）

### 1. `trimStart()` 和 `trimEnd()`

移除字符串首或尾的空白（空格、`tab`、换行等）。`trimLeft` / `trimRight` 分别为别名。

```javascript
const s = "  abc  ";
s.trimStart(); // "abc  "
s.trimLeft(); // "abc  "
s.trimEnd(); // "  abc"
s.trimRight(); // "  abc"
```

### 2. `flat()` 和 `flatMap()`

**`flat(depth)`**：按指定深度递归扁平化，默认 `depth` 为 `1`。

```javascript
const arr = [1, 2, [[[3, 4]]]];
arr.flat(); // [1, 2, [[3, 4]]]
arr.flat(3); // [1, 2, [3, 4]]
arr.flat(-1); // 不展开
arr.flat(Infinity); // [1, 2, 3, 4]
```

**`flatMap()`**：映射后再扁平一层（深度固定为 1）。

```javascript
let arr = ["My name", "is", "", "Lisa"];
let newArr1 = arr.flatMap((e) => e.split(" "));
let newArr2 = arr.map((e) => e.split(" "));
console.log(newArr1); // ["My", "name", "is", "", "Lisa"]
console.log(newArr2); // [["My", "name"], ["is"], [""], ["Lisa"]]
```

### 3. `Object.fromEntries()`

将键值对列表转为对象，与 `Object.entries()` 互逆。

```javascript
const object = { key1: "value1", key2: "value2" };
const array = Object.entries(object);
Object.fromEntries(array); // { key1: 'value1', key2: 'value2' }
```

```javascript
Object.fromEntries([
  ["foo", "bar"],
  ["baz", 42],
]); // { foo: "bar", baz: 42 }

Object.fromEntries(
  new Map([
    ["foo", "bar"],
    ["baz", 42],
  ])
); // { foo: "bar", baz: 42 }
```

### 4. `Symbol.prototype.description`

创建 `Symbol` 时可传入描述字符串；**ES10** 起可直接通过 `.description` 读取。

```javascript
let dog = Symbol("dog");
String(dog); // "Symbol(dog)"
dog.toString(); // "Symbol(dog)"
dog.description; // dog
```

### 5. `Function.prototype.toString()` 的扩展

**ES10** 起尽可能保留注释、空格等，使输出更接近源码。

```javascript
function sayHi() {
  /* dog */
  console.log("wangwang");
}
sayHi.toString();
```

### 6. 可选的 `catch` 绑定

可省略 `catch` 的参数：

```javascript
try {
} catch (err) {
  console.log("err", err);
}

// ES10：可选 catch 绑定
try {
} catch {
}
```

---

## 六、ES11（亦称 ES2020）

### 1. `BigInt`

表示大于 \(2^{53}-1\) 的整数；可用字面量 `10n` 或 `BigInt(10)` 创建。

- 不能与 `Math` 的方法混用；与 `Number` 运算需先统一类型（`BigInt` 转 `Number` 可能丢精度）。
- 涉及小数的运算会向下取整。
- `0n === 0` 为 `false`，`0n == 0` 为 `true`；可与 `Number` 比较、在同一数组中排序。

```javascript
const mixed = [4n, 6, -12n, 10, 4, 0, 0n];
mixed.sort(); // [-12n, 0, 0n, 10, 4n, 4, 6]
```

### 2. 空值合并运算符 `??`

仅当左侧为 `null` 或 `undefined` 时返回右侧；与 `||` 不同，`||` 对所有假值都会回退右侧。

```javascript
const foo = null ?? "default string";
console.log(foo); // default string

const baz = 0 ?? 42;
console.log(baz); // 0

const baz2 = 0 || 42;
console.log(baz2); // 42
```

### 3. 可选链 `?.`

在链上遇到 `null` / `undefined` 时不报错，短路为 `undefined`。

```javascript
const adventurer = {
  name: "Alice",
  cat: { name: "Dinah" },
};
const dogName = adventurer.dog?.name;
console.log(dogName); // undefined
console.log(adventurer.someNonExistentMethod?.()); // undefined
```

**ES11** 之前常写作：

```javascript
const dogName =
  adventurer && adventurer.dog && adventurer.dog.name;
```

### 4. `Promise.allSettled`

入参为 Promise 可迭代对象；待全部落定后返回描述每个结果的对象数组。

```javascript
Promise.allSettled([
  Promise.resolve(33),
  new Promise((resolve) => setTimeout(() => resolve(66), 0)),
  99,
  Promise.reject(new Error("an error")),
]).then((values) => console.log(values));
// [
//   { status: 'fulfilled', value: 33 },
//   { status: 'fulfilled', value: 66 },
//   { status: 'fulfilled', value: 99 },
//   { status: 'rejected', reason: Error: an error }
// ]
```

### 5. `String.prototype.matchAll()`

返回包含所有匹配结果及捕获组的迭代器。

```javascript
const regexp = /t(e)(st(\d?))/g;
const str = "test1test2";
const array = [...str.matchAll(regexp)];
console.log(array[0]); // ["test1", "e", "st1", "1"]
console.log(array[1]); // ["test2", "e", "st2", "2"]
```

---

## 七、ES12（亦称 ES2021）

### 1. `String.prototype.replaceAll()`

返回新字符串，替换所有匹配项；模式可为字符串或正则。

```javascript
let string = "hello world, hello ES12";
string.replace(/hello/g, "hi");
string.replaceAll("hello", "hi"); // hi world, hi ES12
```

使用正则时必须带全局标志 `g`，否则抛错：

```javascript
string.replaceAll(/hello/, "hi");
// TypeError: String.prototype.replaceAll called with a non-global RegExp argument
```

### 2. 数字分隔符

使用 `_` 提高可读性：

```javascript
const money = 1_000_000_000;
// 等价于
const money = 1000000000;
```

### 3. `Promise.any()`

接受一组 `Promise`，封装为新的 `Promise`：任一 `fulfilled` 则整体 `fulfilled`；**全部** `rejected` 则整体 `rejected`（失败时常为 `AggregateError`）。

```javascript
const promises = [
  Promise.reject("ERROR A"),
  Promise.reject("ERROR B"),
  Promise.resolve("result"),
];
Promise.any(promises)
  .then((value) => console.log("value: ", value))
  .catch((err) => console.log("err: ", err));
// value: result
```

全部失败时：

```javascript
const promises = [
  Promise.reject("ERROR A"),
  Promise.reject("ERROR B"),
  Promise.reject("ERROR C"),
];
Promise.any(promises)
  .then((value) => console.log("value: ", value))
  .catch((err) => {
    console.log("err: ", err); // AggregateError: All promises were rejected
    console.log(err.message); // All promises were rejected
    console.log(err.name); // AggregateError
    console.log(err.errors); // ["ERROR A", "ERROR B", "ERROR C"]
  });
```

### 4. 逻辑赋值运算符

```javascript
a ||= b; // a = a || b
c &&= d; // c = c && d
e ??= f; // e = e ?? f
```

---

## 八、ES13（亦称 ES2022）

### 1. `Object.hasOwn()`

**ES13** 新增静态方法，比 `Object.prototype.hasOwnProperty.call(obj, prop)` 更简洁、可靠地判断自有属性。

```javascript
const example = { property: "123" };
console.log(Object.prototype.hasOwnProperty.call(example, "property"));
console.log(Object.hasOwn(example, "property"));
```

### 2. `Array.prototype.at()` / 字符串 `at()`

**ES13** 起，数组与字符串支持 `at()`，可用正负索引；负数从末尾计数。

```javascript
const array = [0, 1, 2, 3, 4, 5];
console.log(array[array.length - 1]); // 5
console.log(array.at(-1)); // 5
console.log(array[array.length - 2]); // 4
console.log(array.at(-2)); // 4

const str = "hello world";
console.log(str[str.length - 1]); // d
console.log(str.at(-1)); // d
```

### 3. `Error` 的 `cause`

**ES13** 起，可在 `new Error(message, { cause })` 中挂载原因，便于链式追溯。

```javascript
function foo() {
  try {
    // do something
  } catch (err0) {
    throw new Error("Connecting to database failed.", { cause: err0 });
  }
}

try {
  foo();
} catch (err1) {
  // 通过 err1.cause 可取回 err0
}
```

### 4. 顶层 `await`

在 **ES8** 中 `await` 仅能在 `async` 函数内使用；**ES13** 起在 ES 模块中可使用顶层 `await`，导入方会等待该模块完成异步初始化。

```javascript
const colors = fetch("../data/colors.json").then((response) =>
  response.json()
);
export default await colors;
```
