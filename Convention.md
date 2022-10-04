# Convention

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

![image-20221003162045282](C:\Users\Hongyu Lin\AppData\Roaming\Typora\typora-user-images\image-20221003162045282.png)

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

