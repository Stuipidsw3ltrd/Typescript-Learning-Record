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