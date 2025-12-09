---
title: JavaScript入门
date: 2025-12-07 20:34 
description: JavaScript入门笔记
categories:
- JavaScript笔记
tags:
---
<head>
  <meta name="referrer" content="no-referrer" />
</head>



# JavaScript入门

## 数据类型和变量

1. Number

   JavaScript不区分整数和浮点数，统一用Number表示。例如：

   ```javascript
   123; // JS可表示的最大整数是2^53
   123.5;
   -99;
   NaN; // NaN表示Not a Number，当无法计算结果时用NaN表示
   Infinity; // Infinity表示无限大，当数值超过了JavaScript的Number所能表示的最大值时，就表示为Infinity
   ```

2. 字符串

   用单引号或双引号括起来的任意文本是字符串。

   如果字符串本身包含`'`或`"`，那么可以用`\` 转义：

   ```js
   'I\'m \"OK\"!'; I'm OK!
   ```

   多行字符串则是用反引号表示``

   ```js
   `这
   是
   多行`;
   ```

   字符串之间的连接可以用`+`，也可用反引号和${}就可以自动替换：

   ```js
   let name = '小明';
   let age = 20;
   let message = `你好，${name}，你今年${岁}了！`;
   ```

   需要注意，JS中的字符串一旦创建就是***不可变的***，这点和Java有点类似：

   ```js
   let a = 'Test';
   a[0] = 'X';
   console.log(a); // 'Test'
   ```

   

3. 布尔值，布尔运算

   `true`或`false`，`&&`是与运算，`||`是或运算，`|`是非运算。

4. 比较运算符

   绝大部分比较运算符都与Java一致，需要特别注意三个**例外**：

   - 相等运算符：JS在设计时，有两种相等运算符：

     - `==`，它会自动转换数据类型再比较，有时会得到错误结果

       ```js
       false == 0; // true
       `5` == 5; // true
       ```

     - `===`，不会自动转换数据类型，如果数据类型不一致，会返回`false`，如果一致再进行值比较

       ```js
       false === 0; // false
       `5` === 5; // false
       ```

     > [!TIP]
     >
     > 由于JS的最初设计问题，我们应***坚持使用***`===`比较，不要用`==`

   - NaN与其他所有值都不相等，包括它自己

     ```js
     NaN === NaN; //false
     ```

     所以要判断`NaN`的唯一方法是通过：

     ```js
     isNaN(NaN); // true
     ```

   - JavaScript的浮点数运算会出现误差

     ```js
     1 / 3 === (1 - 2 / 3); //false
     ```

     正确的比较方式是计算二者的误差是否在允许范围内：

     ```js
     Math.abs(1 / 3 - (1 - 2 / 3)) < 0.00000001; //true
     ```

     

5. BigInt

   如果要表示比2^53还要大的数字，可以在整数的后面加上n，例如`12345678912345679098764n`，也可以用`BigInt()`把Number和字符串转换成BigInt。

   BigInt之间支持正常的加减乘除运算，结果仍然是一个BigInt，但不能把一个BigInt和Number放在一起运算。

6. null和undefined

   `null`表示一个空的值，类似Java中的`null`，Python中的`None`

7. 数组

   JavaScript与Python类似，数组可以包括任意数据类型。例如：

   ```js
   [1, 2, 3.14, 'Hello', null, true];
   ```

   索引的起始值为0。

   JS的数组是动态的数组具有很强的灵活性：如果对数组的长度赋一个新的值，也会改变数组的长度，多余长度的元素是`undefine`。同样，如果对数组的索引超过了范围，也会引起数组长度的变化。

   `slice()`方法，对数组进行切片，类似Python中对数组的切片操作：

   Python：arr[0:3]等价于JavaScript：arr.slice(0, 3);

   `push()`向数组末尾添加若干元素，`pop()`则把最后一个元素删除掉。

   `unshift()`向数组头部添加如干元素，`shift()`则是把数组的第一个元素删掉。

   `splice(start, deleteCount, item1, item2, ..., itemN)`方法：表示从start开始更改的位置索引，要删除deleteCount个元素，然后从start位置插入item1, item2, ..., itemN个元素。

   

8. 对象

   JS的对象有点类似Python的字典，但是JS的对象的键只能是字符串类型，值则可以是任意数据类型。例如：

   ```js
   var person = {
       name: 'Bob',
       age: 20,
       tags: ['js', 'web', 'mobile'],
       city: 'Beijing',
       hasCar: true,
       zipcode: null
   };
   ```

   要获取一个对象的属性，我们可以用`对象变量.属性名`的方式：

   ```js
   person.name; // Bob
   person.zipcode; // null
   ```

   

9. 变量

   与其他编程语言类似，唯一需要注意的是，声明一个变量需要用`var`语句：

   ```js
   var a = 1;
   ```

   JS是动态语言，变量的类型无需在声明时指定，并且在运行时可以改变。

   现代JS中，还推荐用`let`来声明变量：

   ```js
   let a = 1;
   ```
   > JavaScript的`'use strict'`模式是用来检查变量是否有用`var`或者`let`申明，否则会报错

10. 字符串

字符串是不可变的，如果对字符串的某个索引赋值，不会产生任何报错，也不会有任何效果。
```js
let s = 'Text';
s[0] = 'X';
console.log(s); // s仍然为'Test'
```

11. 数组

JavaScript的`Array`可以包含任意数据类型，并通过索引来访问每个元素
```js
let arr = [1, 2, 3, 'Alice', null, true]
```
JavaScript对数组的索引如果超出范围，会直接改变数组的大小，但是不会报错，元素值为`undefined`。
- `slice(start, end)`函数：
截取数组的片段，表示截取数组[start, end)，start默认为0，end默认为数组长度。
- `push(element, ...)`函数：
表示向数组的末尾添加若干元素。
- `pop()`函数：
表示把数组的最后一个元素删除。
- `unshift(element, ...)`函数：
表示向数组的开头添加若干元素。
- `shift()`函数：
表示把数组的第一个元素删除。
- `concat()`函数：
把数组连接起来，注意concat()并没有修改当前`Array`，而是返回了一个新的Array。
```js
let a = ['A', 'B', 'C'];
let arr = a.concat([1, 2, 3]); // 数组a没有改变
```
- `join(str)`函数：
表示把当前数组的所有元素用指定的字符串连接起来。然后返回连接后的字符串。
12. 对象
JavaScript的对象，是一个无序的集合数据类型，它由若干键值对组成。
例如：
```js
let xiaoming = {
    name: '小明',
    birth: 1990,
    school: 'No.1 Middle School',
    height: 1.70,
    weight: 65,
    score: null
};
xiaoming.name; // '小明'
xiaoming.birth; // 1990
```
对象的属性必须是一个有效的变量名。如果属性包含特殊字符，必须用`''`括起来。访问属性必须用`['XXX']`来访问。
```js
let xiaohong = {
    name: '小红',
    'middle-school': 'No.1 Middle School'
};
xiaohong['middle-school'];
```
13. 条件判断
```js
if {
   ...
} else if {
   ...
} else {
   ...
}
```
JavaScript把`null`、`undefined`、`0`、`NaN`和空字符串`''`视为`false`，其他值一概视为`true`。

14. 循环
- 初始条件+结束条件+递增条件
```js
for(i = 1; i<=1000; i++) {
   ...
}
```
- `for.. in...`
将一个对象的所有属性遍历出来
```js
let o = {
    name: 'Jack',
    age: 20,
    city: 'Beijing'
};
for (let key in o) {
    console.log(key); // 'name', 'age', 'city'
}
```
要过滤掉对象继承的属性，用hasOwnProperty()来实现：
```js
let o = {
    name: 'Jack',
    age: 20,
    city: 'Beijing'
};
for (let key in o) {
    if (o.hasOwnProperty(key)) {
        console.log(key); // 'name', 'age', 'city'
    }
}
```
由于Array也是对象，而它的每个元素的索引被视为对象的属性，因此，for ... in循环可以直接循环出Array的索引：
```js
let a = ['A', 'B', 'C'];
for (let i in a) {
    console.log(i); // '0', '1', '2'
    console.log(a[i]); // 'A', 'B', 'C'
}
```
- `for... of ...`

`for... of ...`的提出是为了解决Map和Set数据结构没有下标，不能使用下标访问元素，因此ES6标准引入了新的`iterable`对象，`Array`，`Map`和`Set`
都是`iterable`对象。

```js
let a = ['A', 'B', 'C'];
let s = new Set(['A', 'B', 'C']);
let m = new Map([[1, 'x'], [2, 'y'], [3, 'z']]);
for (let x of a) { // 遍历Array
    console.log(x);
}
for (let x of s) { // 遍历Set
    console.log(x);
}
for (let x of m) { // 遍历Map
    console.log(x[0] + '=' + x[1]);
}
```
> 注意，`for... in ...`是遍历对象的**属性**，而`for ... of ...`是遍历iterable的**元素**。

- `forEach`

遍历`iterable`对象还可以使用内置的`forEach`方法：
```js
// Array对象
let a = ['A', 'B', 'C'];
a.forEach(function (element, index, array) {
    // element: 指向当前元素的值
    // index: 指向当前索引
    // array: 指向Array对象本身
    console.log(`${element}, index = ${index}`);
});
// Set对象
let s = new Set(['A', 'B', 'C']);
s.forEach(function (element, sameElement, set) {
    console.log(element);
});
// Map对象
let m = new Map([['A', 1], ['C', 2]]);
m.forEach(function (value, key, map) {
   console.log(value); // 1, 2
});
```

15. `Map`

由于对象的数据结构中，属性只能是字符串，在ES6规范中，引入了`Map`数据结构，其具有极快的查找速度。
```js
let m = new Map([['Michael', 95], ['Bob', 75], ['Tracy', 85]]);
m.get('Michael'); // 95，查找
m.set('Adam', 100); // 添加新的键值对
```

16. `Set`

`Set`数据结构中，也是一组key的集合，但是不存储value，在`Set`中，没有重复的key。

## 函数

- 定义方式

  ```js
  function abs(x) {
    if (x >= 0)
      return x;
    else
      return -x;
  }
  ```

  由于JavaScript允许传入任意个参数而不影响调用，因此传入的参数比定义的参数多也没有问题，函数内部并不会处理多余的参数。

  传入的参数比定义的少也不会报错。

- arguments

  JavaScript还有一个免费赠送的关键字`arguments`，它只在函数内部起作用，永远指向当前函数调用者传入的所有参数。

  ```js
  function foo(x) {
    for (let i = 0; i<arguments.length; i++) {
      console.log('arg ' + i + ' = ' + arguments[i]); // 10, 20, 30
    }
  }
  foo(10, 20, 30)
  ```

- rest参数

  rest参数有点类似Python中的**kwargs，代表除了已明确定义的其余任何函数参数。

  ```js
  function foo(a, b, ...rest) {
    console.log(a); // 1
    console.log(b); // 2
    console.log(rest); // [3, 4, 5]
  }
  
  foo(1, 2, 3, 4, 5)
  ```

- 方法

  在对象中绑定的函数，称为对象的方法。

- 高阶函数

  - `map/reduce`函数

    Map函数的作用是将函数作用在Array的每一个元素上

    ```js
    function pow(x) {
      return x * x;
    }
    let arr = [1, 2, 3, 4, 5];
    let results = arr.map(pow);
    console.log(results);
    ```

    Reduce函数必须接受两个参数，作用在整个Array所有元素上，并返回一个结果，如果数组元素只有1个，还需要提供一个额外的初始参数：

    ```js
    let arr = [123];
    arr.reduce(function (x, y) {
      return x + y;
    }, 0);
    ```

    

  - `filter`函数

    filter函数即使把`Array`的某些元素过滤掉：

    ```js
    let arr = [1, 2, 4, 5, 6];
    let r = arr.filter(fuction (x) {
        return x % 2 !== 0;      
    });
    console.log(r); // [1, 5]
    ```

    

  - `sort`函数

    Sort函数默认是把所有元素转换成String再排序，然后比较俩俩元素的ASCII码的大小，由小到大排序。

    sort函数当然也支持自定义，通常规定对于两个元素，返回负数表示认为`x<y`，返回`0`表示认为`x=y`，返回正数表示认为`x>y`。

- 箭头函数

  箭头函数的作用是简化常用函数的撰写

  ```js
  x => {
    if (x > 0) {
      return x * x;
    }
    else {
      return -x * x;
    }
  }
  ```

- 闭包

  JavaScript中，函数可以直接作为返回值返回。

  ```js
  function lazy_sum(arr) {
    let sum = function() { // 嵌套函数
      return arr.reduce(function (x, y) {
          return x + y;                  
      });
    }
    return sum;
  }
  
  let f = lazy_sum([1, 2, 3, 4, 5]); // function sum()
  f(); // 15
  ```

  

## 面向对象编程

- 原型继承

  JavaScript的面向对象编程概念中最初的继承方式是通过原型（prototype）`__prototype__`变量

- class继承

  class写一个类的时候，注意定义在类里的函数，无`function`关键字。

  ```js
  class Student {
    constructor(name) {
      this.name = name;
    }
    hello() {
      alert('Hello, ' + this.name + '!');
    }
  }
  ```

  继承：

  ```js
  class PrimaryStudent extends Student {
    constructor(name, grade) {
      super(name); // 记得super调用父类的构造方法！
      this.grade = grade;
    }
    myGrade() {
      alert('I am at grade ' + this.grade);
    }
  }
  ```

## 浏览器

- DOM树

  HTML文档经过浏览器解析后就是一颗DOM树，JavaScript通过操作DOM树，来操作HTML文档

  <img src="https://gitee.com/Marches7/piture-bed/raw/master/img/image-20251112145355734.png" alt="image-20251112145355734" style="zoom:50%;" />

- JavaScript的异步操作

  - AJAX

    即Asynchronous JavaScript and XML，意思是用JavaScript执行异步网络请求。

    应用场景：网页向后端提交表单时，传统的同步做法，表达开始提交时，浏览器就开始刷新界面，在新的页面告诉你提交成功还是失败。而AJAX的方法可以让用户停留在当前界面，同时用JavaScript发送HTTP请求，接受到数据后，再用JavaScript更新页面。

  - Promise

    Promise即代码承诺将来会根据状态执行某段代码。

    典型案例：`setTimeout()`函数，指定一段时间后执行某段代码。`setTimeout(printTime, 1000);`指的是1000毫秒后执行自定义的printTime函数。

  - async函数

    `async function`可以定义一个异步函数，函数内部，使用`await`关键字调用一个Promise，这时的函数看起来和同步函数是一样的。大大提高代码可读性。

    ```js
    async function myDisplay() {
      let myPromise = new Promise(function(resolve) {
        setTimeout(function() {resolve("I love You!!!");}, 1000);
      });
      document.getElementById("demo").innerHTML = await myPromise;
    }
    
    myDisplay();
    ```

- JavaScript错误处理

  与Java类似，使用`try...catch...finally`

  但是注意**异步**函数的错误处理需要定义在执行时，调用时不一定会执行所以可能看不到错误捕捉。

## jQuery库

jQuery是一款用于简化JavaScript对HTML文档操作的函数工具库，它的优势是：

1. 消除浏览器差异：兼容不同的浏览器版本

2. 操作DOM的方法简洁高效：例如用`$('#test')`就能代替`document.getElementById('test')`

   - `$`符号

     jQuery把所有功能全部封装在一个全局变量`jQuery`中，而`jQuery`的别名是`$`符号。

     ```js
     $ === jQuery; // true
     ```

3. 轻松实现动画、修改CSS等各种操作

### 选择器

jQuery的选择器可以帮助我们快速定位到一个或多个DOM节点。

1. 按tag查找

   例如`<div id="abc">...</div>`，查找方式是直接写上`tag`名：

   ```js
   let div = $('div');
   ```

2. 按属性查找

   1. 按ID查找，如果DOM节点有`id`属性，例如`<div id="abc">...</div>`，查找方式是以`#`开头：

      ```js
      let div = $('#abd');
      ```

   2. 按class查找，例如`<div class="red">...</div>`，查找方式是直接在class名称前加一个`.`：

      ```js
      let div = $('.red');
      ```

      如果节点有多个class，例如`<div class="red green">...</div>`，可以查找同时包含`red`和`green`的节点：

      ```js
      let a = $('.red.green'); // 注意没有空格
      ```

   3. 属性名查找

      一个DOM节点除了`id`和`class`外还可以有很多属性，很多时候按属性查找会非常方便。

      例如：`<div type="password">`

      ```js
      let ps = $('[type=password]');
      ```

      再例如：`<p name="A B">`

      ```js
      let a = $('[items="A B"]'); // 如果属性的值包含空格等特殊字符，需要用双引号括起来。
      ```

      还支持前缀查找或者后缀查找：

      ```js
      let i = $('[name^=icon]'); // 前缀,找出所有<... name="icon*">
      let n = $('[name$=with]'); // 后缀,找出所有<... name="*with">
      ```

3. 组合查找

   组合tag和属性查找：

   例如：`<input name="email">...</input>`

   ```js
   let emailInput = $('input[name=email]');
   ```

   组合tag和class来查找：

   例如：`<tr class="red ...">...</tr>`

   ```js
   let tr = $('tr.red');
   ```

4. 层级查找

   层级选择器的好处在于缩小了选择范围，避免了页面不相关的元素。

   1. 后代选择器：`$('ancestor descendant dedescendant ...')`

      如果多个DOM元素有层级关系，可以用`$('ancestor descendant dedescendant ...')`来选择，例如：
      
      ```html
      <div class="testing">
          <ul class="lang">
              <li class="lang-javascript">JavaScript</li>
              <li class="lang-python">Python</li>
              <li class="lang-lua">Lua</li>
          </ul>
      </div>
      ```
      
      要选出JavaScript所在的DOM，可以用层级选择器，下面三种方法都可以选出`[<li class="lang-javascript">JavaScript</li>]`：
      
      ```js
      $('div.testing ul.lang li.lang-javascript'); // 三个层级均指定
      $('ul.lang li.lang-javascript'); // 指定父子两个层级
      $('div.testing li.lang-javascript'); // 指定祖孙两个层级
      ```

   2. 子选择器：`$(parent>child)`
   
      仅匹配parent的**直接子元素**。例如对上面的HTML例子，想要选出JavaScript所在的DOM，只能写成：
   
      ```js
      $('ul.lang>li.lang-javascript');
      ```

5. 过滤器

   过滤器一般不单独使用，它通常附加在选择器上，例如：

   ```js
   $('ul.lang li'); // 选出JavaScript、Python和Lua 3个节点
   $('ul.lang li:first-child'); // 仅选出JavaScript
   ```

### 操作DOM

- 修改text和HTML

  - .html()用于读取（函数不带参数）和修改（函数带参数）元素的HTML标签，对应JS中的`innerHTML`。

  - .text()用于读取（函数不带参数）和修改（函数带参数）元素的纯文本内容，对应JS中的`innerText`。

  - .val()用于读取（函数不带参数）和修改（函数带参数）元素的value值，.val()只能用在表单元素上。

    > .html()和.val()用在多个元素上时，只能读取**第一个**元素。.text()可以返回所有选中元素的文本内容。

- 修改CSS

  `.css('name', 'value')`

  ```js
  let div = $('#test-div');
  div.css('color'); // '#000033', 获取CSS属性
  div.css('color', '#336699'); // 设置CSS属性
  div.css('color', ''); // 清除CSS属性
  ```

- 显示和隐藏DOM

  ```js
  let a = $('a[target=_blank]');
  a.hide();
  a.show();
  ```

- 添加节点：`append()`

  > 如果一个要添加的DOM节点已经存在于HTML文档中，它会首先从文档中移除，然后再添加。

- 删除节点：`remove()`

### 事件

所谓事件即异步执行的JavaScript代码，jQuery获取对象后，可以给对象绑定不同的事件。

例如用户点击超链接时弹出提示框，可以这样写：

```js
let a = $('#test-link');
a.click(function () {
  alert('Hello');
});
```

jQuery能够绑定的事件包括但不限于：

- 鼠标事件

  - click：鼠标单机时触发
  - dbclick：鼠标双击时触发
  - mouseenter：鼠标进入时触发；
  - mouseleave：鼠标移出时触发；
  - mousemove：鼠标在DOM内部移动时触发；

- 键盘事件

  - keydown：键盘按下时触发；
  - keyup：键盘松开时触发；
  - keypress：按一次键后触发。

- 其他事件

  - change：当`<input>`、`<select>`或`<textarea>`的内容改变时触发；

  - ready：当页面被载入并且DOM树完成初始化后触发。

    > `ready`仅作用于`document`对象，由于`ready`事件在DOM完成初始化后触发且仅出发一次，所以适合用来写初始化代码。
    >
    > 例如我们想在提交表单时弹出警示框，可以这么写：
    >
    > ```js
    > $(document).ready(function() {
    >   $('#testForm').submit(function() {
    >     alert('submit!');
    >   });
    > });
    > ```
    >
    > 还可以简化成：
    >
    > ```js
    > $(function () {
    >   $('#testForm').submit(function() {
    >     alert('submit');
    >   });
    > });
    > ```

### 动画

略

## underscore

## Node.js

Node.js是JavaScript的一个运行环境，并为其提供了诸如网络设置，文件管理等多个环境包。

- 模块化

  在Node.js环境中，一个.js文件就被称为一个模块（module），因此模块之间的变量函数不冲突，还可以相互引用。例如：

  `module01.js`

  ```js
  const aaa = '111';
  const bbb = '222';
  
  module.exports = {
    a: aaa, // 这里填写输出的变量，可以是任意的对象、函数、数组等
    b: bbb,
  };
  ```

  `module02.js`

  ```js
  const foo = require('./module01');
  console.log(foo.a); // '111'
  console.log(foo.b); // '222'
  ```

- 内置模块

  Node.js也内置了一些模块，用户可以直接调用，为其程序提供的基本功能

  导入基本模块的示例：

  ```js
  import { readFile } from 'node:fs'; // 从fs模块中导入异步读文件的函数
  ```

  

  - `fs`模块

    `fs`模块负责文件系统管理的功能。

  - `stream`模块

    `stream`模块负责处理'流'这种数据结构。

  - `http`模块

    提供网络处理功能。

  - `crypto`模块

    提供加密算法等功能。



