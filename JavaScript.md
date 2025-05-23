## 变量

- 大写命名的常量仅用作“硬编码（hard-coded）”值的别名

## 数据类型

- JS中有8中基本的数据类型（7种原始类型和1种引用类型）
  - Number
    - 任何对NaN的进一步数学运算都会返回NaN
  - BigInt
    - 用于表示任意长度的整数，（普通number只有+-2^53-1）
    - 数字尾部为n代表为BigInt类型
    - IE不兼容
  - String
  - Boolean
  - null
    - 对不存在的object的引用活着null指针
  - undefined
    - 未被赋值
  - Object和Symbol
    - 其他所有类型为“原始类型”，object不是
  - typeof
    - 返回参数的类型
    - typeof null = 'object'(历史错误)

## alert, prompt, confirm

- alert
  - 显示一条信息，并等待用户按下OK
  - alert('Hello');
- prompt
  - 显示一个带有文本消息的模态窗口，带有input和确认取消按钮
  - result = prompt(title, [default])
  - 返回的值为字符串
  - IE 在不给第二个参数时会带入undefined
- confirm
  - 显示一个带有question以及确认取消按钮的模态窗口
  - let isBoss = confirm("Are you the boss?");
- 这些方法都是模态的，暂停脚本的执行，不允许用户与页面其余部分进行交互，知道窗口被解除
- 模态窗口的确切位置由浏览器决定，通常在页面中心
- 窗口的确切外观也取决于浏览器，不能被修改

## 类型转换（除对象）

- 数字转换

| 值              | 变成……                                                       |
| :-------------- | :----------------------------------------------------------- |
| `undefined`     | `NaN`                                                        |
| `null`          | `0`                                                          |
| `true 和 false` | `1` and `0`                                                  |
| `string`        | 去掉首尾空白字符（空格、换行符 `\n`、制表符 `\t` 等）后的纯数字字符串中含有的数字。如果剩余字符串为空，则转换结果为 `0`。否则，将会从剩余字符串中“读取”数字。当类型转换出现 error 时返回 `NaN`。 |

- 遇到"+"时，优先连接字符串

  +运算中，只要任意一个运算元时字符串，另一个运算元也将被转化为字符串

  - alert(2 + 2 + '1') // '41'

  可用+来做数字转换

- 布尔类型转换

  - 0, '', null, undefined, NaN将会变成false，其他变成true

## 运算符

- 一元运算符的优先级大于二元运算符
- +=， -=， /=， *= 优先级与=相同，只有2
- ++和--只适用于变量
  - let counter = 1;
  - let a = ++ counter // 2 返回的是++后的新值
  - let counter = 1
  - let a = counter++ // 1 返回的是++前的旧值
  - 如果++后的值不会被立刻赋值，仅做运算，则二者表现相同
  - ++的优先级比绝大部分运算符高
    - let counter = 1;
    - 2 * ++ counter // 4
    - 2 * counter++ // 2
- 位运算符
  - 把运算元当作32为整数，在他们的二进制表现形式上操作
  - 按位与 &
  - 按位或 |
  - 按位异或 ^
  - 按位非 ~
  - 左移 <<
  - 右移 >>
  - 无符号右移 >>>
- 逗号运算符
  - 被用来写更简短的代码，能处理多个表达式，使用','将它们分开，每个表达式都运行了，但只有最后一个的结果会被返回
    - let a = (1 + 2, 3 + 4) // 7(3+4)的结果
  - 逗号运算符的优先级非常低只有1
  - 可用来把几个行为放在一行进行复杂的运算
    - for (a = 1, b = 3, c =a * b; a < 10; a++){}

## 值的比较

- 所有比较运算符均返回布尔值

### 字符串比较

- 按字符逐个进行比较

  - 'Glow' > 'Glee'
  - 'Bee' > 'Be'

- 首先比较两个字符串的首位字符大小

  如果一方字符较大（或较小），则该字符串大于（或小于）两一个字符串。算法结束

  否则，如果两个字符串的首位字符相等，则继续去除两个字符串各自的后一位字符进行比较

  重复上述步骤进行比较，知道比较完成某字符串的所有字符位置

  如果两个字符串的字符同时用完，那么判定他们相等，否则未结束的字符串更大

- 在'Glow'和'Glee'中

  - G和G相等（继续）
  - l和l相等（继续）
  - o比e大，算法停止，第一个字符串大于第二个

### 不同类型间的比较

- js会首先将其转化为数字再判定大小
  - '2' > 1
  - '01' == 1
  - true == 1
  - false == 0
- null与undefined的比较 
  - null === undefined // false
  - null == undefined // true
  - 当使用数学式或其他比较方法（> < >= <=）时
    - null被转化为0，undefined被转化为NaN
    - null > 0 // false
    - null == 0 // false，因为==不会做任何类型转换
    - null >= 0 // true
    - undefined > 0 // false 被转换为了NaN
    - undefined < 0 // false 同上
    - undefined == 0 // false 被转换为了NaN

## 逻辑运算符

### ||

- 从左到右依次计算操作数
- 处理每一个操作数时，都将其转换为布尔值，如果结果为true，停止计算，返回这个操作数的初始值
- 如果所有的操作数都被计算过，则返回最后一个操作数
- 或运算可以寻找第一个真值

### &&

- 从左到右以此计算操作数
- 在处理每一个操作数时，都将其转化为布尔值，如果结果是false，就停止计算，并返回这个操作数的初始值
- 如果所有的操作数都被计算过（假如都是真值），则返回最后一个操作数
- 与运算可以寻找第一个假值
- 与运算优先级比或运算高

### ！

- 将操作数转化为布尔值再取反
- !!可以将某个值转换为布尔值
- ！的优先级在所有运算符里面最高

## 空值合并运算符??

- a ?? b的结果是：

  - 先设定，如果一个值不是null也不是undefined，则称之为已定义的

  - 如果a是已定义的，则结果为a

  - 如果a不是已定义的，则结果为b

  - ````js
    let height = 0;
    alert(height || 100); // 100
    alert(height ?? 100); // 0
    ````

  - ??的优先级与||相同，都为3

  - ??禁止与||和&&一起使用，但可以使用括号解决这个问题

    - ````
      let x = (1 && 2) ?? 3; // 可以正常工作
      ````

### 循环

- break / continue不允许放入三元表达式或者 ? 的右面

##### break / continue标签

对于双层循环

~~~js
for (let i = 0; i < 3; i++) {

​	for (let j = 0; j < 3; j++) {

​		let input = prompt(`Value at coords (${i}, ${j})`, '');

​	}

}
~~~

如果想在用户取消输入后立刻break只会终止内部循环

可以使用标签来终止外部循环 break <labelName>

~~~js
outer: for (let i = 0; i < 3; i++) {

  for (let j = 0; j < 3; j++) {

    let input = prompt(`Value at coords (${i},${j})`, '');

    // 如果是空字符串或被取消，则中断并跳出这两个循环。
    if (!input) break outer; // (*)

    // 用得到的值做些事……
  }
}

alert('Done!');
~~~

此时break outer向上寻找名为outer的标签并跳出找到的循环

还可以将标签单独写一行

~~~js
outer:
for (let i = 0; i < 3; i++) { ... }
~~~

continue同理，这时候会执行跳转到标记循环的下一次迭代

- break指令必须在代码块内

~~~js
break label;  // 跳转至下面的 label 处（无效）

label: for (...)
~~~

~~~js
label: {
  // ...
  break label; // 有效
  // ...
}
~~~

- continue只有在循环内部才可行

### Switch

- switch中的比较是严格相等 ===

### 函数

- 当一个参数没有被给到函数中，会变成undefined
- 只有空return或者没有return的函数将会返回undefined
- 不要在return与返回值之间添加新行，JS会默认在return之后加上分号发哦之返回空值。想要这样需要加上括号

### 函数表达式

~~~js
let sayHi = function() {
  alert( "Hello" );
};
~~~

- 函数声明与函数表达式-创建函数的时机

  - 函数表达式在赋值时-代码执行到等号右侧开始创建函数，并且从现在开始使用

  - 函数声明在被定义之前就可以被调用

  - ~~~js
    sayHi("John"); // Hello, John
    
    function sayHi(name) {
      alert( `Hello, ${name}` );
    }
    ~~~

  - 而函数表达式不行

  - ~~~js
    sayHi("John"); // error!
    
    let sayHi = function(name) {  // (*) no magic any more
      alert( `Hello, ${name}` );
    };
    ~~~

- 函数声明与函数表达式的块级作用域

  - 如果函数声明在一个代码块内，严格模式下他在该代码块内的任何位置都是可见的，在代码块外不可见。

  - ~~~js
    let age = prompt("What is your age?", 18);
    
    // 有条件地声明一个函数
    if (age < 18) {
    
      function welcome() {
        alert("Hello!");
      }
    
    } else {
    
      function welcome() {
        alert("Greetings!");
      }
    
    }
    
    // ……稍后使用
    welcome(); // Error: welcome is not defined
    ~~~

  - 原因：

  - ~~~~js
    let age = 16; // 拿 16 作为例子
    
    if (age < 18) {
      welcome();               // \   (运行)
                               //  |
      function welcome() {     //  |
        alert("Hello!");       //  |  函数声明在声明它的代码块内任意位置都可用
      }                        //  |
                               //  |
      welcome();               // /   (运行)
    
    } else {
    
      function welcome() {
        alert("Greetings!");
      }
    }
    
    // 在这里，我们在花括号外部调用函数，我们看不到它们内部的函数声明。
    
    
    welcome(); // Error: welcome is not defined
    ~~~~

  - 如果想在if外也可用，可以用函数表达式

  - ~~~~js
    let age = prompt("What is your age?", 18);
    
    let welcome;
    
    if (age < 18) {
    
      welcome = function() {
        alert("Hello!");
      };
    
    } else {
    
      welcome = function() {
        alert("Greetings!");
      };
    
    }
    
    welcome(); // 现在可以了
    ~~~~

    

### 箭头表达式

- 如果函数没有参数，括号必须保留

