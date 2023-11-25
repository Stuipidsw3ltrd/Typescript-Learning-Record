# JavaScript/TypeScript

## JS中的事件循环(Event Loop)

JS是单线程的语言，因此，JS会通过将不同函数的执行上下文压入执行调用栈中来保证代码的有序执行。在执行同步代码的时候，如果遇到异步事件，JS引擎并不会一直等待异步事件响应，而是先将异步事件挂起，转而去执行栈中的其他任务。JS会将异步任务或其回调函数加入到一个任务队列当中等待执行。

这个任务队列根据任务类型的不同分为两类：一种是 **微任务队列**；另一种是 **宏任务队列**；

常见的微任务：Promise.then()/Promise.catch(); node中的process.nextTick; 对DOM变化监听的MutationObserver

常见的宏任务：Script脚本的执行; setTimeout(), setInterval(), setImmediate() 之类的定时事件; I/O操作; UI渲染

事件循环的过程：

1. 从宏任务队列中取出头部的宏任务执行（第一次的话这个宏任务就是执行Script脚本）
2. 按照调用栈的顺序，执行该宏任务中的所有同步代码，期间如果遇到微任务，就将其加入微任务队列，如果遇到宏任务，就加入到宏任务队列；
3. 该轮宏任务执行完毕，检查微任务队列，如果存在微任务，则按照入队顺序执行这些微任务；执行这些微任务的期间，如果又进一步发现微任务，则将其加入到微任务队列，如果进一步发现宏任务则也加入到宏任务队列
4. 重复 3. 直到微任务队列当中不再存在微任务
5. 一轮事件循环循环结束，如果宏任务队列中存在宏任务，则返回1. ，否则该次执行结束。

简而言之，事件循环中的任务执行顺序为：本轮宏任务中的同步代码 -> 本轮宏任务中的所有微任务 -> 下一轮宏任务 ...

下面看几个例子：

```typescript
async function async1 () {
	console.log('async1 start')
	await async2()
	console.log('async1 end')
}

async function async2 () {
	console.log('async2 start')
}

console.log('script start')

setTimeout(function() {
	console.log('setTimeout 1')
}, 2000)

setTimeout(function() {
	console.log('setTimeout 2')
}, 0)

async1()

new Promise(function (resolve) {
	console.log('proimse1')
	resolve()
}).then(function() {
	console.log('promise2')
})
console.log('script end')

```

这段代码的输出应该是：

```javascript
script start
async1 start
async2 start
proimse1
script end
async1 end
promise2
setTimeout 2
setTimeout 1
```

首先，JS会执行第一个宏任务，就是这个脚本本身。

JS会优先执行所有同步代码，而异步事件则将其或者其回调放入到对应的队列当中：

```javascript
1. 执行到的第一句同步输出代码就是 console.log('script start')
2. 往下发现setTimeout 1 和 setTimeout 2，这是两个宏任务，直接将其加入到宏任务队列当中
3. 再往下走到async1()的调用当中，进入async1()函数，首先执行同步代码 console.log('async1 start')
4. 然后进入到await async2(), 注意，这里调用栈中的顺序是：会先执行async2()中的所有同步代码，不会看到await就直接将其挂起，栈执行顺序就类似深度优先（它不先进去怎么知道该这个异步函数是该fulfill还是走到更深的调用栈才能fulfill呢？）
5. console.log('async2 start') 被执行，然后 return， 这里由于async2是异步函数，所以return 相当于 return Promise.resolve()
6. async2 resolve 之后，出来看到自己被await修饰（函数调用语句从右到左执行），所以自己下面的语句就是被自己阻塞住的语句，也就是它的回调（await就是Promise的语法糖，被await阻塞的语句就相当于是Promise.then(), 在await过程被catch后要执行的语句就相当于是Promise.catch()）
7. 将console.log('async1 end')加入到微任务队列， async1() 执行完毕
8. 来到new Promise 这里，注意，new Promise 一旦被定义，传入其中的函数将会被立即执行，所以这里直接执行同步代码 console.log('proimse1')
9. 然后 new Promise 被 resolve；一旦resolve，就可以进入到.then回调当中，发现then回调是一个微任务，于是将其放入微任务队列
10.执行脚本最后一行console.log('script end')
11.此时当前轮宏任务的同步代码已经执行完毕。而微任务队列当前有：1） console.log('async1 end') 2）console.log('promise2') 宏任务队列当中有两个setTimeOut
12.因此我们需要清空微任务队列，按照入队顺序依次执行1）和2）
13.微任务执行完毕，现在开始执行下一个宏任务setTimeout1，执行完毕。注意，是setTimeout这个计时函数本身是一个宏任务，而不是其回调函数。这里执行就执行了，JS并不会去等待setTimeout时间结束触发回调。
14.执行最后一个宏任务setTimeout2,执行完毕。
15.setTimeout2回调触发，执行console.log('setTimeout 2')
16.两秒后，setTimeout1回调触发，执行console.log('setTimeout 1')
```

上述例子非常详细了，触类旁通可以按照事件循环的原理推敲出所有异步代码的执行顺序，下面再看一个小例子：

```javascript
new Promise((resolve, reject) => {
    setTimeout(() => {console.log('setTimeout'); resolve('')}, 1000)
})
.then(() => console.log('resolve 1'))
.then(() => console.log('resolve 2'))
```

执行顺序为`setTimeout -> resolve 1 -> resolve 2`

```javascript
原理一样很简单：
1. 执行Promise中的函数，发现是个宏任务，将其加入到宏任务队列
2. 此时已经没有可以执行的同步代码，且微任务队列为空(Promise没有被resolve，进不到then当中去)
3. 一轮事件循环结束，从宏任务队列中拿出setTimeout执行，执行完毕。
4. 等待一秒后，回调触发（现在这个回调就相当于一个宏任务了），按调用栈顺序执行同步代码：打印setTimeout，然后resolve
5. 因为resolve了，所以进入到第一个then当中，发现是个then微任务，将其加入到微任务队列当中，没有可以执行的同步代码，现在微任务队列中有一个then回调，所以执行它，打印resolve 1，然后return（相当于return Promise.resolve())
6. 因为resolve了，所以走到第二个then，发现是微任务，加入微任务队列。没有同步代码可以执行了，微任务队列有then回调，执行它，打印resolve 2。
7. 微任务队列清空。宏任务队列没有新任务了。执行结束。
```



## TS中的深浅拷贝

### 深拷贝和浅拷贝

编程语言中的数据类型可按照存储方式分为两种：

- 不可变类型（基础类型）
- 可变类型（引用类型）

同时，在操作系统中有两种不同的内存种类：

- 栈内存
- 堆内存

![image-20220912163256226](C:\Users\Hongyu Lin\Projects\Typescript-Learning-Record\img\image-20220912163256226.png)

从上图可见，栈内存是一个类似于哈希表的结构，所有的基础数据类型都是直接存储在栈内存当中，当我们初始化一个变量`a`并为其赋值1，这个值就会以键值对的形式存储在栈内存当中。栈内存当中的name是唯一的，所以当你创建一个变量b，并且让其等于a，这个时候b会在栈内存当中拥有自己的键值对，因此修改b，是不会影响a的。因此直接变量名和值直接存储在栈内存当中的数据类型就是不可变类型。典型的不可变类型有：

- number
- string
- boolean

![image-20220912163713462](C:\Users\Hongyu Lin\Projects\Typescript-Learning-Record\img\image-20220912163713462.png)

而可变类型是值存储在堆内存当中的数据类型。堆内存没有类似栈内存的一样的哈希表结构，它只能靠堆地址来进行索引。由上图可见，可变类型的数据在栈内存中存储的是其变量名和堆地址，而堆地址所对应的堆内存中才存储的是其实际的值。

所以如果我们先创建一个名字为a的可变类型变量，再创建一个名字为b的可变类型变量，并且让b等于a。在栈内存中，他们当然是拥有不相同的键值，但是，由于它们的值不再是一个直接可读写的数值，而是一个地址，因此，这两个变量将始终指向同一堆地址。因此，每当修改变量a中的数组，变量b的数组也会跟着改变，因为它们指向的就是堆内存中的同一数组。

#### 浅拷贝

当我们已经有a这一个数组变量之后，我们将a的堆地址1中指向的值复制一份，到一个新的堆地址2，并且将这个堆地址2赋给变量b，这个过程就是浅拷贝。

这种拷贝看似非常的正常，a和b都指向了不同的堆地址，不同的堆地址里面都有自己的那个值。但问题在于，如果堆内存中存储的数据中的子项里面有可变类型数据，那么对a的这些可变子项修改，仍然会造成b中的对应改变

```typescript
let a = [[1,2],[3]]
let b = [...a] //shallow copy
a[1].push(4)
console.log(b) // [[1,2],[3,4]]
```

在很多情况下，这很明显不是我们想要的结果。发生这种结果的原因就是，浅拷贝只是单纯复制了堆内存中的数据，这样复制出来，并不会改变数据当中子项可变数据的堆地址。

浅拷贝的方法有很多，对于数组来说，spread syntax以及绝大部分能够返回新的数组的数组方法都是一种浅拷贝。

#### 深拷贝

深拷贝就是连根拔起的一种拷贝，对于要拷贝的数据本身以及其可能包含的recursive reference，全部都会复制一份新的数据并分配新的堆内存，这样的话，由a深拷贝得到b，再修改a的话不会对b造成任何影响。

TS当中的深拷贝方法非常有限：

- 原生TS的方法：`JSON.parse(JSON.stringify())`；本质就是先将其转换为JSON字符串再解析，非常的畸形
- Lodash：Lodash是一个非常有用的工具库，对于深拷贝，可以直接`lodash.clonedeep()`

## TS类的构造函数

TS当中，对于类的属性有两种实例化方法：

- 先在类中定义属性，再在构造函数中为该属性赋值

```typescript
class Test{
    private a;
    constructor(a:number){
        this.a = a
    }
}
```

- 直接将构造函数的参数作为类的属性

```typescript
class Test{
    constructor(
      private a: number,
      private b: number
    ){
        this.someFunction(this.a, this.b)
    }
}
```

## TS类中的方法调用链

TS当中，可以直接让方法`return this`来返回类的当前实例，从而可以达到方法调用链的效果

```typescript
class Test{
    function printA(){
        console.log("A")
        return this
    }
    
    function PrintB(){
        console.log("B")
        return this
    }
}

let test = new Test()
test.A().B() // function chain
```

## TS中的相等判断

TS当中:

三等号`===`是等同判断，会同时判断等式两边变量的类型和值，必须要类型和值同时相同才会为true，其对应的不等判断为`!==`

双等号判断是等值判断，只会判断等式两边的值，如果等式两边的变量类型不相同，则会先进行类型转换：

- 对于数字，字符串或者布尔值，会优先转化成数字进行比较
- null和undefined只能和自身以及对方相等
- NaN和任何对象包括自身都不相等

双等号的不等判断为`!=`

双等号会带来很多意想不到的判断结果，如果不是有特殊需求，尽量不要使用。

## TS中的特殊类型

- any: 表示这个变量可以是任何类型，any是一个非常不安全的类型，在typescript中尽量避免使用

```typescript
let x:any
x = 1;
x = "a";
x = true;
x.toString(); // 不会报错
```

any可以让一个变量拥有任何类型，可以使用任意方法而不会报语法错误

- unknown: typescript提供的any的安全替代版，unknown类型的变量，也是可以指代任何数据类型，可以被任意种不同的类型赋值，但是！unknown类型的变量，在真正的类型被确定之前，是不能够调用任何方法，或者被赋值给其他非unknown或any类型的变量的。

```typescript
let x:unknown
x = 1;
x = true;
x = "a";
x.toString(); // 报错
x.length(); // 报错，虽然x现在确实是string，但它的定义类型仍然是unknown
let y: string = x // 报错，原因同上
```

- never: 这个类型就是代表那种永远不会有值的类型，一般在会抛出异常的函数中作为返回值声明类型使用

## TS中会让真假判断结果为false的值

空字符串，数字0，undefined和null

```typescript
const not = val => !val
// 以下全为true
console.log(not(""))
console.log(not(0))
console.log(not(undefined))
console.log(not(null))
```

## JS/TS 中的 bind()

参考链接：[JavaScript Function bind() Method (w3schools.com)](https://www.w3schools.com/js/js_function_bind.asp)

bind主要有两个作用：

1. 为函数绑定作用域，防止this的context丢失
2. 函数借调 - 让一个对象A使用另一个对象B的函数，并把B的函数作用域绑定至A

### JS/TS 当中的 this 丢失 context 问题

上图说明了this的指向。this的指向总结成一句话：跟this写在哪无关，跟谁调用它有关

通常，当一个函数被作为回调函数，this会丢失它原本的上下文

```javascript
const person = {
  firstName:"John",
  lastName: "Doe",
  display: function () {
    let x = document.getElementById("demo");
    x.innerHTML = this.firstName + " " + this.lastName;
  }
}

setTimeout(person.display, 3000); // "undefined undefined" 因为setTimeout是一个定义在全局的函数，因此this会指向window
```

解决方案：使用bind方法，为函数手动绑定作用域：

```typescript
const person = {
  firstName:"John",
  lastName: "Doe",
  display: function () {
    let x = document.getElementById("demo");
    x.innerHTML = this.firstName + " " + this.lastName;
  }
}
display = person.display.bind(person)
setTimeout(display, 3000); // "undefined undefined" 因为setTimeout是一个定义在全局的函数，因此this会指向window
```

在嵌套函数当中，this同样会丢失上下文

```typescript
class TestClass {
    private a: number
    constructor(){
        this.a = 100
    }
    public nestFunction(){
        function inner(){
            console.log(this.a) // this is undefined
        }
        const inner = 
        inner()
    }
}

let a = new TestClass()
a.nestFunction()
```

解决方案：使用箭头函数。箭头函数没有this，如果在箭头函数中用this，这个this会自动绑定到距离箭头函数层级最近的那个非箭头函数对象，在这里就是TestClass对象

## 函数借调

```javascript
const person = {
  firstName:"John",
  lastName: "Doe",
  fullName: function () {
    return this.firstName + " " + this.lastName;
  }
}

const member = {
  firstName:"Hege",
  lastName: "Nilsen",
}

let fullName = person.fullName.bind(member);
console.log(fullName()) // "Hege Nilsen"
```

借调的时候，同时可以设置第二个函数给借来的函数一个参数默认值

```javascript
const person = {
  firstName:"John",
  lastName: "Doe",
  fullName: function (greeting: string, mark: string) {
    return greeting + this.firstName + " " + this.lastName + mark;
  }
}

const member = {
  firstName:"Hege",
  lastName: "Nilsen",
}

let fullName = person.fullName.bind(member,"Hello: ", "!"); // 给借来的函数传递默认的参数
console.log(fullName()) // "Hello: Hege Nilsen!"
```

```javascript
class A {
    name: string = 'A';
    go() {
        console.log(this.name);
    }
}

class B {
    name: string = 'B';
    go() {
        console.log(this.name);
    }
}

const a = new A();
a.go();
const b = new B();
b.go = b.go.bind(a);
b.go(); // A
```



## const / let 和块级作用域

Reference: [let & const: block scope in javascript](https://blog.csdn.net/weixin_46334272/article/details/114090340)

块级作用域（block scope）是JavaScript ES6引入的新概念

在之前，JS只有全局变量和函数局部变量的概念。ES6引入块级作用域的同时，也引入了let和const两个声明关键字。

ES6之前：

```javascript
if (true){
    var a = 10
}

function foo(){
    var a = 20
}

console.log(a) // 不会报错，因为ES6之前，只有函数的作用域内是局部变量，if内部的a是全局变量，所以这里会输出10，非常反人类
```

```javascript
var i = 5
for (var i = 0; i < 10; i++){
    //pass
}
console.log(i) //输出10，因为for循环作用域中的i也是全局变量
```

ES6之后，引入了块级作用域，所有花括号包裹起来的部分，都是一个独立的作用域了，因此在ES6中，if和for循环也将成为一个独立的作用域，而值得注意的是：**只有新引入的let和const关键字声明的变量，才能够触发块级作用域！** var关键字在非函数的块中声明出的变量依然是被视作全局变量（因此现在尽量少使用var）

```javascript
let i = 5
for (let i = 0; i < 10; i++){
    //pass
}
console.log(i) //输出5，因为for循环现在是一个独立的作用域，let在独立作用域中可以redeclare重名的变量
```

```javascript
const x = 5
{
    const x = 10
}
console.log(x) //输出5，因为花括号内部是独立作用域
```

### const和let的异同

let：不能重复声明，但可以重新给变量赋值，声明时不需要赋初始值

const：不能重复声明，也不能重新赋值，声明时必须赋初始值

```javascript
let x = 10;
x = 20 // legal
let x = 30 // illegal
const y = 10;
y = 20; // illegal
const y = 30; // illegal

```

注意，**const并非真正的常量**，const并不能保持一个变量的值不变，它是保持变量所指向的栈内存地址中的值不变。在深浅拷贝中已经写过，对于不可变类型的数据，其值就直接存在栈内存当中，对于可变数据类型，其在栈内存中保存的是一个指向堆地址的指针。所以，如果const声明的是一个可变类型（对象或者数组），那么修改其内部的值是不会报错的

```javascript
const x = []
x.push(1)
console.log(x) // [1]
```

## Asynchronous 异步

一般来说，我们写的代码都是同步的，也就是从上到下依次执行，如果有A，B，C三个语句，那么B必须在A执行完了之后才能执行，C必须在B执行完了之后才能执行。

这种在当前线程中代码执行的序列又叫callstack。

如果现在，有A B C三个语句，其中B是异步的，那么，C可以不用等待B执行完毕就马上执行。如果B是很耗时的操作，C又需要马上执行，那么异步操作就变得相当有用。

```javascript
// 1
console.log('Let's begin.');
// 2
setTimeout(() => {
    console.log('I waited and am done now.');
}, 3000);
// 3
console.log('Did I finish yet?');

//输出
Let's begin.
Did I finish yet?
I waited and am done now.
```

ES7推出了async和await关键字，这两个关键字可以更好的处理异步操作，但在此之前，异步操作都是由Promise对象来完成的。

A **Promise** is an object with a delayed completion at some indeterminate future time.

Promise是一个在未来某个事件延迟完成和确定的对象，由一个异步函数组成，我们需要把要异步执行的函数体，放在promise内部，并定义好在何种情况下是处理成功，何种情况是处理失败即可

```javascript
async function sayWordsAfterWhile(){
    return new Promise<string>((resolve, reject) => {
        setTimeout(()=>{
            const randomNum = (Math.random() * 100 ) % 10
            if (randomNum % 2 === 0){
                resolve("I complete the async function!")
            }else{
                reject("encountered error")
            }
            
        }, 3000)
    })
}

let promiseResult = sayWordsAfterWhile()
promiseResult.then(res => {
    console.log("success: ", res)
}).catch(err => {
    console.log("error: ", err)
})
```

可以看到，promise支持泛型，我们可以通过定义其泛型为string，从而声明了这个函数的返回值是`Promise<string>`, 同时，我们可以看到promise内部的函数，这个函数有两个参数，resolve和reject，这个是由promise自身提供的函数，如果函数体执行成功，那么就可以用resolve来返回处理结果，resolve返回的结果会作为then中回调函数的参数；如果promise函数体执行出错，就用reject返回对应的信息，reject返回的数据会作为catch的回调函数的参数。

当我们拿到一个promise后，我们需要使用then和catch来注册相应的回调函数，代表当这个promise处理成功和失败的时候我们应该干什么。then和catch又可以嵌套新的promise，这样层层嵌套下去非常丑陋，代码很难读懂。

```javascript
let somePromise = someAsyncFunction()
somePromise
.then(res => {
    let anotherPromise = anotherAsyncFunction(res)
    anotherPromise
    .then(secondRes => {
        ...
    })
})
```

 所以，async和await关键字的出现缓解了这一问题。

async关键字用于函数声明，表明这是一个异步的函数，而await关键字则用于调用异步函数的时候。注意：await关键字只能用于有async关键字声明的函数当中

```javascript
async foo(){
    let x = await anotherAsyncFunction()
} 
```

下面看一段代码例子：

```javascript
import fetch from "@data-fetch"
import constants from "@utility-constants"

(async function getData() {
    let response = await fetch(constants.pictureApi) // 这里不用try catch是因为fetch内部不会抛出reject，而是把响应的结果定义在resolve的数据当中
    if (response.ok){
        console.log("success")
        let data = await response.json()
        renderPicture(data)
    }else{
        throw new Error("encounter error!")
    }
})();
```

这里可以看到，我们通过await字段，可以直接等待这个异步函数的完成，因为后续的逻辑需要这个异步函数的返回值。

如果我们用promise来写，那就是两个then的嵌套，第一个then我们得到response，判断它的状态，如果是ok的话，又会用then来处理response.json()这个请求，阅读起来非常的麻烦。而async和await可以把代码逻辑变得非常清楚明了，就像是同步的代码一样。

上述代码片中的(async function getData(){...})()的写法叫做Immediately Invoked Function Expression(IIFE)，它的作用是可以同时声明和执行这个函数，当你需要在toplevel去使用await关键字的时候，就需要这么写。

刚刚已经说过，await可以等待一个异步函数执行完毕，如果我们不使用这个字段，那么这个异步函数后续的代码将马上被执行，如果后续的语句依赖该异步函数的返回值，则这些语句将拿到一个空的Promise

```javascript
async function sayWordsAfterWhile(){
    return new Promise<string>((resolve, reject) => {
        setTimeout(()=>{
            const randomNum = (Math.random() * 100 ) % 10
            if (randomNum % 2 === 0){
                resolve("I complete the async function!")
            }else{
                reject("encountered error")
            }
        }, 1000)
    })
}

function test() {
    console.log("step 1")
    let res = sayWordsAfterWhile()
    console.log(res)
    console.log("step 3")
}

async function test2() {
    console.log("step 1")
    try{
        const res = await sayWordsAfterWhile()
        console.log(res)
    }catch(err){ // 注意必须要handle exception，不然无法处理reject的情况，导致callstack被彻底阻塞
        console.log(err)
    }
    console.log("step 3")
}

test()

test2()

=============================
[LOG]: "step 1" 
[LOG]: Promise: {} 
[LOG]: "step 3" 

[LOG]: "step 1" 
[LOG]: "encountered error" 
[LOG]: "step 3" 
```

