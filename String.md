# String

## includes

string也可以用includes来进行包含判断

```javascript
let x = "hello world!";
console.log(x.includes(hello)) // true
```

## 切片

js里的string不能像python一样用index来切片，js提供的字符串方法可以做到切片，slice、substring、substr都可以切片，这三个方法很类似但有细微差别

slice接收start index和end index，如果end index不提供，就默认一直切片至字符串末尾

```javascript
let x = "abcdefg";
console.log(x.slice(0,3)) // "abc"
```

substring和slice的参数list一样，但是会把所有小于0的index当作0看待

```javascript
let x = "abcdefg";
console.log(x.substring(-1,3)) // "abc"
```

substr的第一个参数是起始index，第二个参数是从起始index开始，切割字符串的长度。

```javascript
let x = "abcdefg";
console.log(x.substr(1,4)) // "abcd"
```

