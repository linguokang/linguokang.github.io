---
title: 【阅读笔记】javascript-questions收集
date: 2021-01-15 14:35:36
---
JavaScript 进阶问题列表
[Github链接](https://github.com/lydiahallie/javascript-questions/blob/master/zh-CN/README-zh_CN.md)

<!-- more -->

49. num的值是什么?

```javascript
const num = parseInt("7*6", 10);
```
- A: 42
- B: "42"
- C: 7
- D: NaN
<details>
<summary>答案</summary>
答案: C
只返回了字符串中第一个字母. 设定了 进制 后 (也就是第二个参数，指定需要解析的数字是什么进制: 十进制、十六机制、八进制、二进制等等……),parseInt 检查字符串中的字符是否合法. 一旦遇到一个在指定进制中不合法的字符后，立即停止解析并且忽略后面所有的字符。
*就是不合法的数字字符。所以只解析到"7"，并将其解析为十进制的7. num的值即为7.
</details>

---

59. 输出是什么?
```javascript
const numbers = [1, 2, 3, 4, 5];
const [y] = numbers;
console.log(y);
```

- A: [[1, 2, 3, 4, 5]]
- B: [1, 2, 3, 4, 5]
- C: 1
- D: [1]

<details>
<summary>答案</summary>
答案: C
我们可以通过解构赋值来解析来自对象的数组或属性的值，比如说：
```javascript
[a, b] = [1, 2];
```
a的值现在是1，b的值现在是2.而在题目中，我们是这么做的:
```javascript
[y] = [1, 2, 3, 4, 5];
```
也就是说，y等于数组的第一个值就是数字1.我们输出y， 返回1.
</details>

---

67. 输出什么?
```javascript
// index.js
console.log('running index.js');
import { sum } from './sum.js';
console.log(sum(1, 2));

// sum.js
console.log('running sum.js');
export const sum = (a, b) => a + b;
```

- A: running index.js, running sum.js, 3
- B: running sum.js, running index.js, 3
- C: running sum.js, 3, running index.js
- D: running index.js, undefined, running sum.js

<details>
<summary>答案</summary>
答案: B
import命令是编译阶段执行的，在代码运行之前。因此这意味着被导入的模块会先运行，而导入模块的文件会后执行。

这是CommonJS中require（）和import之间的区别。使用require()，您可以在运行代码时根据需要加载依赖项。 如果我们使用require而不是import，running index.js，running sum.js，3会被依次打印。
</details>

---

70. 输出什么?
```javascript
console.log("🥑" + "💻");
```

- A: "🥑💻"
- B: 257548
- C: A string containing their code points
- D: Error

<details>
<summary>答案</summary>
答案: A
使用+运算符，您可以连接字符串。 上述情况，我们将字符串“🥑”与字符串”💻“连接起来，产生”🥑💻“。
</details>

---

77. 以下是个纯函数么?
```javascript
function sum(a, b) {
  return a + b;
}
```

- A: Yes
- B: No

<details>
<summary>答案</summary>
答案: B
纯函数一种若输入参数相同，则永远会得到相同输出的函数。

在特定情况下，即使输入相同参数，也不能得到相同的返回值：
```javascript
var a = b = {}
a[Symbol.toPrimitive] = b[Symbol.toPrimitive] = () => Math.random()
sum(a, b) // Uncertain
```
所以它不是一个纯函数。
</details>

---

114. 将会发生什么?
```javascript
let config = {
  alert: setInterval(() => {
    console.log('Alert!')
  }, 1000)
}

config = null
```

- A: setInterval 的回调不会被调用
- B: setInterval 的回调被调用一次
- C: setInterval 的回调仍然会被每秒钟调用
- D: 我们从没调用过 config.alert(), config 为 null

<details>
<summary>答案</summary>
答案: C
一般情况下当我们将对象赋值为 null, 那些对象会被进行 垃圾回收（garbage collected） 因为已经没有对这些对象的引用了。然而，setInterval的参数是一个箭头函数（所以上下文绑定到对象 config 了），回调函数仍然保留着对 config的引用。只要存在引用，对象就不会被垃圾回收。因为没有被垃圾回收，setInterval 的回调每1000ms (1s)会被调用一次。
</details>

---

153. 哪个作为method的值可以打印{ name: "Lydia", age: 22 }?
```javascript
const keys = ["name", "age"]
const values = ["Lydia", 22]

const method = /* ?? */
Object[method](keys.map((_, i) => {
	return [keys[i], values[i]]
})) // { name: "Lydia", age: 22 }
```

- A: entries
- B: values
- C: fromEntries
- D: forEach

<details>
<summary>答案</summary>
答案: C
fromEntries 方法可以将二维数组转换为对象。在每个子数组的第一个元素是key，在每个子数组的第二个元素是value。在这个例子中，我们映射了 keys 数组，它返回了一个数组，数组的第一个元素为keys数组当前索引的值，第二个元素为values数组当前索引的值。

这样就创建了一个包含正确keys和values的子数组的数组，因此结果为{ name: "Lydia", age: 22 }。
</details>
