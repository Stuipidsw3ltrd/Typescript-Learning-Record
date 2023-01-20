# Object

JS/TS中的Object就像是Python当中的字典一样，是一种哈希表。Object的键类型必须是可哈希的类型，即字符串、数字、Symbol等，这些类型的键最后都会转化为字符串。

对于实例化的Object类型对象（假设名字为A），那么不能使用A.XXX来调用Object内建方法，因为A.XXX始终只能调用A内的属性和方法，必须使用Object.someMethod(A, *leftArgs) 来调用Object的内建方法并应用在对象A上

## 初始化

初始化的时候，可以给键名加双引号，也可以不加

```typescript
let obj = {
    keyA: 1,
    3:3
}

console.log(obj)
/*
{
  "keyA":1,
  "3":3
}
*/
```

## 格式化键名

如果要在object里面添加一个键，这个键的内容是来自其他变量，则需要将这个变量用方括号`[]`括起来才能完成正确的键值取值

```typescript
let x ={
    a:1,
    b:2,
}
const mock = {a:10, key:"a"}
let y = {
    ...x,
    [mock.key]: mock.a // 格式化键名
}
```

## Object.assign

object.assign(objectA, objectB) 可以把objectB的字段分配到objectA上面，对于A和B共有的字段，用B中的值去覆盖A中的值，如果B中有A中没有的字段，那么直接把B中的字段和对应的值加到A当中，如果A中有B中没有的字段，那么则维持原状

```javascript
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5 };

const returnedTarget = Object.assign(target, source);

console.log(target);
// expected output: Object { a: 1, b: 4, c: 5 }

console.log(returnedTarget === target);
// expected output: true
```

## Object的keys和values

object可以用keys和values方法获得对象的所有键名或者所有值

```javascript
let x = {
    a:1,
    b:2
}

console.log(Object.keys(x))
console.log(Object.values(x))

//[LOG]: ["a", "b"] 
//[LOG]: [1, 2] 
```

