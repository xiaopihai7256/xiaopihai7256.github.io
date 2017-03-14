---

layout: post
title: JavaScript函数
tags: [JavaScript]
author: 杨比轩
---


## 函数定义和调用

**简单的函数定义**

```javascript
//求绝对值
function abs(x){
    if(x < 0)
      return -x;
    else
    return x;
}
```

和java语法类似：

- `function` 函数关键字申明
- `abs` 函数名
- `(x)` 参数
- `{....} `函数体

> JS如果没有return语句，函数执行完毕会返回`undefined`

本着万物皆对象的原则，function也是对象，所以：

```javascript
var abs = function(x){
  ....
};
```

此时函数是一个匿名函数，变量abs指向匿名函数，本质上和上面那种方式没有区别，但是这里有一点要注意，此处其实是一个赋值语句，所以末尾要加上`;`

**函数的调用**

正常调用不做赘述，说说比较特殊的地方。

JS函数允许传递多个参数或者不传递参数，所以：

```javascript
abs(1,2,“123”,4,5);//后面的参数不会生效，结果返回 1
abs(); //结果返回 NaN
```

> 返回NaN解释：函数传递的x变成`undefined`，计算结果变成NaN

所以，为了避免这种情况的产生，可以做健壮性处理：

```javascript
function abs(x){
  if(x !== 'number'){
    throw 'not a number';
  }
  
  if(x > 0)
    return -x;
  else
    return x;
}
```

**arguments**

JS的函数默认会有一个`arguments`参数，此参数指向函数的参数集，有点类似于array，但是不是array。

可以使用arguments的length属性，来判断参数的个数和对参数进行访问

demo:

```javascript
function getSum(x,y,z){
    var num = 0;
    console.log("arguments length:" + arguments.callee);
    for(var i = 0 ; i < arguments.length ; i++){
        console.log(arguments[i]);
        num = num + arguments[i];
    }
    return num;
}
console.log(getSum(1,2,3,4,5,6,7,8,9,10));
//输出：1,2,3，4,5,6,7,8,9,10,55
```

**rest**

ES6引入的新标准，用于处理多余传入函数内的参数，用法：

```javascript
function restDemo(x, y, ...rest){
    console.log("x="+x);
    console.log("y="+y);
    console.log("rest=" + rest);
}
restDemo(1,2,3,4,5,6);
//输出：
//x=1
//y=2
//rest=3,4,5,6
```

**关于return**

JS有自动在行末加`;`的机制，所以要小心return语句

demo:

```javascript
//return了一个匿名对象，这种是常规情况，可以正常运行
function returnDemo(){
    return {name:'luke'};
}

//return的代码过长时，需要换行，此时js会自动加`;`在行末，所以返回undefined
function returnDemo(){
    return
    { name:'luke'};
}

//这种的也正常，{ 表示此行未结束，会阻断添加 ; 
function returnDemo(){ 
    return {
      name:'luke'
    };
}
```

## 变量的作用域

JS的函数可以嵌套，所有当两个嵌套的函数的变量重名后，内部可以调用外部变量，外部则不能访问内部。

```javascript
'use strict';

function foo() {
    var x = 1;
    function bar() {
        var y = x + 1; // bar可以访问foo的变量x!
    }
    var z = y + 1; // ReferenceError! foo不可以访问bar的变量y!

```

此时如果两个参数名字相同，则内部参数相当于重写了外部参数，会达到屏蔽的效果。

```javascript
'use strict';

function foo() {
    var x = 1;
    function bar() {
        var x = 'A';
        console.log('x in bar() = ' + x); // 'A'
    }
    console.log('x in foo() = ' + x); // 1
    bar();
}
```

**变量的自动提升**

在JS中可以引用稍后申明的变量而不会引发异常就是因为有自动变量提升（variable hoisting），但是提升也会带来一些问题。

```javascript
console.log(x === undefined);//logs true
var x = 3;

//上述代码相当于
var x;
console.log(x === undefined);
x = 3;
```

分析：
> `var x = 3;`相当于一个申明变量语句和一个赋值语句，但是自动提升只会提升申明语句`var x;`仅此而已。所以log输出就是`true`

所以，一段代码或者函数中的var语句应该尽可能的放在接近代码段的顶部位置,一避免产生麻烦。

**全局变量**

全局变量相当于全局对象的属性，在网页中全局对象是`window`，所以可是使用`window.variable`来访问和设置全局变量。
所以在函数体之用`var`或者`const`申明的变量既是全局变量，也是`window`对象的属性，。
无论是function还是var声明的变量，在js中都是变量，也就是window的属性，当前前提是function和var都全局的。

**局部变量**
在`for`或者`if`中声明的变量其实在`{}`也可以引用，因为js的变量作用域其实是函数内部，而`for`等不是函数。
ES6引入了新标准，`let`关键字可以替代`var`可以申明块级的作用域的变量。
```javascript
{
    let abc = "qweqweqweqweqweqweq";
}
console.log(abc);//logs:Uncaught ReferenceError: abc is not defined(…)
```

**常量**
ES6添加的新特性，`const`关键字申明一个固定值的常量。

**命名空间**
javascript实际上只有一个全局作用域。任何变量如果在当前函数作用域中找到，就会继续往上找（上指的是父类），最后再全局作用中也没有找到，就汇报ReferenceError错误。

全局变量会绑定到`window`上，不同的js文件如果使用了相同的全局变量，就会造成明明冲突，并且难以发现。

避免这个问题的方法就是把所有的变量和函数全部绑定到一个全局变量中，如：

```javascript
//全局变量
var MMP = {};

//其它变量
MMP.a='a';
MMP.b = function(){
    console.log("this is function");
}
```

这样就可以将需要的变量申明在`MMP`下。

**对象方法**

在对象内部定义的函数也叫方法，和普通的函数其实没啥区别，但是此种方法可以使用`this`关键字来指向当前调用方法的对象。

```javascript
var person = {
    name:"bixuan",
    age:23
    //此方法就是返回对象的age属性的value
    getAge:function(){

        return this.age;
    }
}
```

如果此时`getAge()`方法写在person对象之外时，当`person.getAge()`种调用时，可以获得和上面一样的结果，但是要是单独的调用`getAge()`

```javascript
var person = {
    name:"bixuan",
    age:23,
    getage:getAge
}
function getAge(){
    return this.age;
}

person.getage();// 结果就是23
getAge();//结果是 undefined
```

`person.getage`指向的就是函数`getAge()`，函数中的`this`指向的就是`person`对象，所以返回的就是23，而直接运行`getAge()`时`this`指向的是`window`对象，因为没有`age`属性，所以结果是`undefined`。

还有一种情况就是函数的多层嵌套，也会导致this的调用出现问题。

```javascript
var person = {
    name:"bixuan",
    age:23,
    
    getage:function(){

        function getAge(){
            return this.age;
        }
    }
}

person.getage();//结果是 undefined
```
原因就是`this`又重新指向了`undefined`了，当然如果处于非`strict`模式下，thi又会指向`window`。为了避免这种情况的发生，可以在外层加上一句：

```javascript
...
var that = this; //用that来存储this变量，供内层调用
function getAge(){
    return that.age;
}
...
```
**apply()**

函数对象都会有一个`apply()`方法，此方法接收两个参数，一个是需要绑定的this对象，第二个参数是一个`Array`,存储的是函数本来的参数。

```javascript
function sum(x, y){
    console.log(this.age);
    console.log( x + y);
}

sum.apply({age:23},[1,2]);
//输出 23,3
```

`call()`和apply类似，不同的是`call`第二个参数不是array，而是按顺序将原函数所有的参数顺序传入。`call({},参数1,参数2,...)

利用`apply`的这个特性，可以用来实现装饰器。

> 装饰器：对已有的函数方法进行装饰，在实现原有功能的基础上，添加额外的功能，或者增强原有功能。

```javascript
function pri(){
    console.log("this is a demo");
}

var count = 0;
(function newPro(){
    count++;

    pri.apply(null,[]);
    
})();
```

上述例子就实现了对函数`pri`的装饰，在执行原有代码的同时，对其运行的次数进行了统计。

**高阶函数**

普通函数接受的参数都是数据类型，高阶函数就是把函数对象当作参数进行传递。

```javascript
function sum(x, y){
    return x + y;
}

function useSum(x, y, sum){

    return sum(x, y);
}
```

1. map/reduce

`map()`方法相当于对`arr`中的数据按照传入的参数函数进行一次迭代。

```javascript
function pow(x){
    return x*x;
}

var arr = [1, 2, 3, 4, 5];

arr.map(pow);// [1,2,9,16,25]
```

`reduce()`其实也是对`arr`中的数据进行迭代，但是和`map`不同的是，`reduce()`相当于嵌套迭代。

> 需要注意的是：`reduce()`必须接收两个参数的函数

```javascript
function sum(x, y){
    return x + y;
}

var arr = [1, 2, 3, 4, 5];

arr.reduce(sum);//15

```

**2. filter **

不知道官方的叫法是什么，应该翻译做过滤器吧。使用`arr`调用，并传入返回`true/false`来对arr中的元素进行判定从而进行过滤。

```javascript
var arr = [1,2,3,4,5];
var newArr = arr.filter(function(x){
    if(x%2 === 0){
        return true;
    }else{
        return false;
    }
})

console.log(arr);//[1,2,3,4,5]
console.log(newArr);//[2,4]
```

`filter()`不会更改调用数组的值，而是返回一个新的`arr`对象。

**回调函数**

`filter`接受回调函数。

- 当回调函数一个参数时，代表数组的元素
- 当对调函数三个参数时，分别为：数组的元素，元素索引，数组本身

```javascript
arr.filter(function(a,b,c){
    console.log(a);
    console.log(b);
    console.log(c);
})
```

所以可以利用`filter`来实现对arr的去重复，去空元素等等操作。

```javascript
//去掉arr中的重复元素
//这里利用的就是indexOf返回的是数组中该元素第一次出现的索引
//来实现对arr的去重读
var strs = ['a','b','c','d','e','b','c','e','g','a'];

var newStrs = strs.filter(function(element, index, arr){
    if(arr.indexOf(element) == index){
        return true;
    }
    else{
        return false;
    }
})

console.log(newStrs);
```

**3.`sort`**

`sort`应该可以翻译为排序器吧，类似java中的`comparable`,只需要传入一个排序的方法，就能对调用的数组进行排序。

> sort默认是将数组的元素转换为`String`对象后进行排序，排序的依据是ASCII值的大小。

```javascript
//按从小到大的循序排序
var nums = [12312,345345,24654356,234234,2342,4564,2342,6786,34,85,24558,62,48,4,512,5,95623];

console.log(nums.sort(function(x,y){
    if(x > y){
        return 1;
    }else{
        return -1;
    }
    return 0;
}))
```
通过传入的比较方法的不同，可实现不同的实现效果，所以重点在于传入的比较函数。

**闭包**

一般的方法和函数都是返回计算的结果，也就是基本数据类型和对象。当函数返回的是包含参数的函数对象时，这种的就称为闭包。

```javascript
var nums = [12312,345345,24654356,234234,2342,4564,2342,6786,34,85,24558,62,48,4,512,5,95623];
function lazy_sum(arr){
    var sum = function(){
        return arr.reduce(function(x,y){
            return x + y;
        })
    }
    return sum;
}

var result = lazy_sum(nums);

console.log(result);//[Function: sum]
console.log(result());//25383212
```
打印结果说明，`result`此时是一个函数，调用此函数，输出结果为`sum()`函数执行的结果。
若此时继续执行代码：
```javascript
nums.push(123123);
console.log(result());//25506335
```
但需要注意的是，此时因为传入的引用数据类型，所以当此数据的值发生变化时，最终的结果也会受到影响。

**箭头函数**

箭头函数有点类似于java 8中的新特性。

```javascript

var f = x => x+1;
//和下面的写法实现了相同的功能

var f = function(x){
    return x+1;`
}
```

此处由于函数体比较简单，所以省略了`{...}  return `，并且当参数不止一个时，也需要使用`(...)`括起来。所以，箭头函数也有以下几种写法： 

```javascript
(x,y) => x + y; //多个参数需要用括号括起来
(x,y) => {  //实现复复杂逻辑体时，需要使用{}括起来
    if(x > y)
        return true;
    else
        return false;
}

/*空参也需要括号。像这种直接返回对象的时候，需要使用（）把括号
把对象括起来，否则引起函数体和对象体的语法冲突而报错
*/
() => ({name:"bixuan"}) 。
```

箭头函数同时也修复了this的历史遗留问题，不是在多层调用时指向window或者undefined了，而是指向词法作用域，也就是外层调用这的OBJ。

```javascript
//以前的写法会出现undefined或者指向的了window
var obj = {
    birth: 1990,
    getAge: function () {
        var b = this.birth; // 1990
        var fn = function () {
            return new Date().getFullYear() - this.birth; // this指向window或undefined
        };
        return fn();
    }
};

//而箭头函数则不会有这个问题出现
var obj = {
    birth: 1990,
    getAge: function () {
        var b = this.birth; // 1990
        var fn = () => new Date().getFullYear() - this.birth; // this指向obj对象
        return fn();
    }
};
obj.getAge(); // 25
```

但是有一点也需要注意，就是箭头函数因为绑定了`this`,所以再调用调用`call()`或者`apply()`的时候，传入的第一个参数无法对`this`进行绑定，还是会和上面一样，绑定在外层调用者上，所以`call({...},X,Y)`的第一个对象参数会被忽略掉。

```javascript
var obj = {
    birth: 1990,
    getAge: function(year){
        var b = this.birth;//1990
        var fn = (y) => y - this.birth;//此时，this.birth还是1990

        return fn.call({birth:3000},year);
    }
};
var obj2 = {
    birth: 1990,
    getAge: function(year){
        var b = this.birth;//1990
        // var fn = (y) => y - this.birth;//此时，this.birth还是1990
        var fn  = function(y){
            return y - this.birth;
        };

        return fn.call({birth:3000},year);
    }
};


console.log(obj.getAge(2016));//结果是26
console.log(obj2.getAge(2016));//-984
```
