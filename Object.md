# Object

JS/TS中的Object就像是Python当中的字典一样，是一种哈希表。Object的键类型必须是可哈希的类型，即字符串、数字、Symbol等，这些类型的键最后都会转化为字符串。

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
