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
   ler age = 20;
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

10. 