---

layout: post
title: JavaScript基础
categories: 
 - JavaScript
author: 杨比轩
---

> 申明：本篇文章内所有内容都是在参考学习[此教程](http://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000)时所作的笔记，如有兴趣可参考原博。

----

## 一、基本语法

js的语法和java类似，每句以`;`结尾，代码块用`{...}`

```javascript
if(i > 2){
  x = 1;
  y = 2;
}
```

和java语法完全相同的地方：

- 单行和多行注释
- 严格区分大小写
- 代码块的嵌套、逻辑判断等等

## 二、JS中的变量

Number：用于存储数值了类型

```javascript
123; // 整数123
0.456; // 浮点数0.456
1.2345e3; // 科学计数法表示1.2345x1000，等同于1234.5
-99; // 负数
NaN; // NaN表示Not a Number，当无法计算结果时用NaN表示
Infinity; // Infinity表示无限大，当数值超过了JavaScript的Number所能表示的最大值时，就表示为Infinity
```

### 字符串

用`'..'`和`"..."`括起来都可以。

### 布尔值

true或者false，也可以是运算的返回结果，比如：与或非逻辑运算或者大小比较。

### 比较运算符

有这几种：>,<,>=,<=,\=\=,\=\=\=

> 1. 由于js允许对任意数据类型进行比较，所以要区别\=\=和\=\=\=：
>
> 第一种是`==`比较，它会自动转换数据类型再比较，很多时候，会得到非常诡异的结果；
>
> 第二种是`===`比较，它不会自动转换数据类型，如果数据类型不一致，返回`false`，如果一致，再比较。
>
> 2. 这两种比较方式都不能比较NaN，NaN使用`isNaN()`方法判断

由于js的设计缺陷，比较浮点型应使用下列方式：

```javascript
Math.abs(1 / 3 - (1 - 2 / 3)) < 0.00001; // true
//而不是
1 / 3 === (1 - 2 / 3); // false
```

### 数组

特殊点在于同一数组内可以存储多种数据类型，有下标

```javascript
new Array(1, 2, 3); // 创建了数组[1, 2, 3]
[1, 2, 3.14, 'Hello', null, true];
var arr = [1, 2, 3.14, 'Hello', null, true];
arr[0]; // 返回索引为0的元素，即1
arr[5]; // 返回索引为5的元素，即true
arr[6]; // 索引超出了范围，返回undefined
```

### 对象

js的对象时有键值对组成的无序集合。

```javascript
var student = {
    name: 'Bixuan',
    age: 22,
    tags: ['js', 'web', 'mobile'],
    school: 'High School',
    hasGirlFriend: true,
    zipcode: null
};

//获取也很简单，直接  对象名.键  获取
student.name;
```

### 变量

建议变量不要使用中文命名

- 申明一个变量使用`var`关键字
- 变量名使用数字大小写字母和符号`$`、`_`的组合，数字不能开头
- 变量名不能是js中的关键字
- 由于js是动态语言的原因，所以变量可以存储任意类型的数据

```javascript
//这样的代码也是完全可以的 
var i = 10;
i = "lalal";
```

strict模式：javascript允许变量的申明不使用var关键字，但是此变量将变成全局变量，如果要禁止此种行为，可以开启strict模式，开启后不使用var关键字申明的变量被使用时将会报错。启用strict模式的方法是在JavaScript代码的第一行写上：

```javascript
'use strict';
// 如果浏览器支持strict模式
```

## 三、字符串

JS的字符串就是用`'...'`,`"..."`括起来的字符表示。如果`'`本身也是一个字符时，那就使用`"..."`括起来，也可以使用转义字符：`\'`

转义字符也可以用来表示十六进制或者Unicode编码等等

```javascript
'\x41'; // 完全等同于 'A'
'\u4e2d\u6587'; // 完全等同于 '中文'
```

在JS中有多行字符串的概念：

```javascript
//此时，解释器并不会忽略换行，而是换行符也会存储在字符串中
//当然了，使用\n显示标注也是可以的

var s = `这是一个
多行
字符串`;
```

要把多个字符串连接起来，可以使用`+`或者模板字符串

```javascript
var name = '杨比轩';
var age = 22;
var message = name + "今年" + age + "岁了";
var message = "${name}今年${age}岁了";
//当然了，有的老的浏览器可能不支持ES6模板字符串
```

操作字符串，直接见demo;

```javascript
var s = 'Hello World';
s.length; // 11
//字符串是不可变的，如果对字符串的某个索引赋值
//不会有任何错误，但是，也没有任何效果：
s[0];//H
//toUpperCase把一个字符串全部变为大写
s.toUpperCase();//HELLO WORLD
//toLowerCase把一个字符串全部变为小写
s.toLowerCase();//hello world
//indexOf返回字符串的出现位置
s.indexOf('world'); // 没有找到指定的子串，返回-1.
s.indexOf('World'); // 返回6
//subtring，切割字符串，参数为开始到结束，都是下标，
s.substring(0,3);// Hel
```

## 四、数组

JavaScript的数组可以包含任意的数据类型，并通过数组下标来访问单个元素。

要获取数组长度，直接访问length属性，直接给length属性赋值，会改变数组长度：

```javascript
var arr = [1,2.3,4,"abc",null];
arr.length;//5
arr.length = 7; //arr = [1,2.3,4,"abc",null,undefined,undefined];
//如果给超出索引长度的元素赋值，会使数组发生变化
arr[9] = 20;
//arr = [1,2.3,4,"abc",null,undefined,undefined,undefined,20];
```

### indexOf()

返回该元素在数组中的位置，没有找到返回-1

```javascript
var arr = [12,34,45,56,67];
arr.indexOf(12);// 0 
arr.indexOf(45);// 2
```

### slice()

和string中的substring基本类似，截取并返回一个新的数组，传递一个参数则是从指定位置到结束，不传递任何参数则是直接复制一个新的一模一样的数组

```javascript
var arr = [1,2,3,4,5];
arr.slice(0,3);//[1,2,3]
var arrCopy = arr.slice();
arrCopy === arr;//true
```

### push()和pop()方法

`push()`是向array的末尾添加元素，`pop()`则是把最后一个元素删除

```javascript
var s = ['a','b','c','d'];
s.push('e','f');//s =  ['a','b','c','d','e','f'];
s.pop();s.pop();s.pop();
alert(s);// ['a','b','c']
```

### unshift()和shift()

和上面的两个方法对应，操作arr的头部

```javascript
var s = ['a','b','c','d'];
s.unshift('e','f');//s =  ['e','f','a','b','c','d'];
s.shift();s.shift();s.shift();
alert(s);// ['b','c','d'];
//如果全部pop或者shift完，则再输出就会报出undefined
```

### sort()

对array进行排序，顺序为默认的顺序`arr.sort()`

### reverse()

翻转array

### splice()

修改arr的万能方法，删除指定开始位置和指定个数的元素，移位添加指定的元素

```javascript
var s = ['a','b','c','d','e','f'];
//添加元素的个数没有限制
s.splice(2,3,'qwe','qwe')
//只删除 ，不添加
s.splice(2,2);
//只添加， 不删除
s.splice(3,0,"insert");
```
### concat()

`concat()`方法把当前的`Array`和另一个`Array`连接起来，并返回一个新的`Array`，实际上concat可以添加任意个元素，并且会把其中的arr全部拆开，添加进新的arr中

```javascript
var arr = ['A', 'B', 'C'];
var added = arr.concat('x','y',[1, 2, 3]);
added; // ['A', 'B', 'C', 'x','y',1, 2, 3]
arr; // ['A', 'B', 'C']
```

### join()

`join()`方法是一个非常实用的方法，它把当前`Array`的每个元素都用指定的字符串连接起来，然后返回连接后的字符串。join并不会更改数组值，而是返回一个字符串。

```
var arr = [1,2,3,4];
arr.join('~');// [1~2~3~4]
```

### 多维数组

JS是支持多维数组操作，具体格式和其他语言类似。

```javascript
'use strict';
var arr = [1,[2,3],[4,5,6],[7,8,9,10]];

var x = arr[0];//x为1
x = arr[1][1];//x为3
```

练习：

arr中存储了学生姓名，请排序后按照如下格式输出：

`欢迎XXX，XXX，...，XXX和XXX同学！`

```javascript
'use strict';
var arr = ['小明','小刚', '小花', '小红', '大军', '阿黄'];
var s = "欢迎" + arr.sort().slice(0,arr.length-1).join("，")+ "和" + arr[arr.length-1] + "同学";
alert(s);
```

## 五、对象

JS的对象是一种无序的集合数据类型，由若干键值对组成。

具体形式如下：

```javascript
//对象的键值对用{}括起来，键值对的形式为：XXX:XXX 
//键值对之间用 ‘,’ 隔开，最后一个不用加
var student = {
  name:'杨比轩',
  age:20,
  school:'ynu',
  
  //键中含有特殊字符时，用''括起来
  //此时，此属性就不是有效属性了
  'high-shcool':'guozhen high school'
}

alert(student.name);//输出的就是：杨比轩
//非有效属性访问不用 . 直接['...']访问
student['high-shcool'];//guozhen high school
```

### 对象属性相关

- 访问不存在的属性时返回`undefined`

- JS的对象时动态类型，属性可以后加

  ```javascript
  var student = {
      name: 'bixuan'
  };
  student.age;//undefined
  student.age = 10; //后加的属性
  delete student.age;//删除该属性，删除不存在的属性不会报错
  ```

- 检测某一对象是否包含某一属性可以使用 in 操作符

- 使用in时，如果父类包含，也会返回true，所有仅判断对象本身是否包含某一属性时，可以用`hasOwnProperty()`方法

  ```javascript
  var student = {
      name: 'bixuan'
  };
  student.hasOwnProperty('name'); // true
  student.hasOwnProperty('toString'); // false
  ```

## 六、条件判断

条件判断就是`if{...}esle{...}`或者`if{...}else if{...}else{...}`，如果之前学过C\C++或者java的话，基本不用看了，全部是一样的，这里也就没有任何笔记。

## 七、循环

循环这里基本上也都是一样的，有一个`for in`循环值的来说一下。

基本语法：

```javascript
//这样就可以遍历对象内的所有属性或者元素，无论对象内的元素时什么类型的值，value永远是String类型的。
for(var value in object){
  value....;
}
```

其他的`for`、`while`、`do...while`循环全部和java等一样。

## 八、map和set

JS中的默认对象表达方式{}其实和java中的map数据类型比较相似，都是键值对的形式。但是JS的默认对象有一个缺陷，key只能是String类型，更像java中的`Properties`。于是**ES6**版本中新增了数据类型Map。

### Map

`Map`是一组键值对的结构，具有相对更快的查找速度。

语法如下：

```javascript
//创建一个map数据类型
var m = new Map([['Michael', 95], ['Bob', 75], ['Tracy', 85]]);
m.get('Michael'); // 95
m.set('bixuan', 100); // 添加新的key-value
m.delete('bixuan');
m.has('bixuan');//false
```

特性和java中的也比较相似，但是一些小区别

- 添加新元素使用`set`，不是`put`
- 判断是否存在某个key使用`has()`，不是`containsKey`
- 删除元素使用`delete()`，不是`remove()`
- 重复set进key相同的键值对，后进去的会覆盖之前的，和java相同

### Set

创建set

```javascript
var s1 = new Set(); // 空Set
var s2 = new Set([1, 2, 3, 3]); // 含1, 2, 3,重复元素会被过滤掉
```

添加和删除元素

```javascript
var set = new Set();
//添加是add()
set.add('1');
set.delete('1');
```

### iterable

如同java集合中的继承体系一样，JS中的`Map`和`Set`也有类似的继承体系，只是比较简单而已。

Map和Set都继承自`iterable`（感觉就像java中的迭代器`Iterator`)，而属于iterable类型的集合可以使用`for ..of`来进行遍历（就像java中的增强for循环的感觉类似），同样这些都是ES6中新增的内容。

```javascript
var set = new Set([1,2,3,4,5,6]);
for(var value of set){
  console.log(value);
}
```

关于`for...in`和`for...of`

其实`for...of`是用来修复`for...in`的历史遗留问题的。首先来看一个demo

```javascript
//for of 也可以遍历Array
var arr = [1,2,3,4,5,6];
for(var value of arr){
  console.log(value + 1);
}
//for of 输出结果为：2,3,4,5,6,7
for(var value in arr){
  console.log(value + 1);
}
//for of 输出结果为：01,21,31,41,51,61
```

这个例子说明，`for...of`遍历的是对象的值，而`for...in`可以说遍历的是对象的属性名称，再看下面这个demo

```javascript
var arr = [1,2,3];
arr.name = 'hello';
for(var value in arr){
  console.log(value); //输出为：1,2，3，name
}
```

`for...in`遍历给对象添加的额外的属性，但是却没有遍历对象本身的属性length。所以这就是问什么引进行的`for...of`，就是为了修复这些历史遗留问题，`for...of`只遍历集合或者对象本身的元素。

除了`for...of`循环之外，也可以使用ES5.1中新增的`forEach()`来进行遍历集合。

遍历Array：

```javascript
var a = ['A', 'B', 'C'];
a.forEach(function (element, index, array) {
    // element: 指向当前元素的值
    // index: 指向当前索引
    // array: 指向Array对象本身
    console.log(element);
});
```

遍历set，由于set没有下标，所以第二个参数也指向了元素

```javascript
var s = new Set(['A', 'B', 'C']);
s.forEach(function (element, sameElement, set) {
    console.log(element);
});
```

遍历map，则为 value，key，map

```javascript
var m = new Map([[1, 'x'], [2, 'y'], [3, 'z']]);
m.forEach(function (value, key, map) {
    console.log(value);
});
```

JS的函数调用其实不要求参数必须一致，所以可以进行简写，例如遍历Array的element

```javascript
var a = ['A', 'B', 'C'];
a.forEach(function (element) { //此处的element是写给自己看的，其实些什么都是获取的是元素的值
    console.log(element);
});

//set也一样
var set = new Set(['A', 'B', 'C']);
set.forEach(function (BALALALA) {
    console.log(BALALALA);
});

//map输出的是value
var map = new Map([['A',1], ['B',2], ['C',3]]);
map.forEach(function (BALALALA) {
    console.log(BALALALA);
});
```
