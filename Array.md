---
typora-copy-images-to: ./img
---

# Typescript - Array

## Array constructor

Array's constructor will receive a number and create an Array full of empty items with that length.

an empty item is not equal to `undefined` or `null`, it's a non-initialized value, which means this value is not explicitly assigned, whereas `undefined` and `null` are explicitly assigned value.

For most of the time, you can barely do anything with empty items, you can use index to assign each value manually or use `Array.prototype.fill()` to quickly assign the same value to all the empty items.

```typescript
let arr = Array(5) // [<5 empty items>]
arr[0] = 'a' // ['a', <4 empty items>]
arr.fill('b') // ['b','b','b','b','b']
```

## Array forEach()

`Array.prototype.forEach(callBackFn)` is to implement a function to each item in this array.

`callBackFn(value:any, index:number, array:any[])` is the function we need to pass, it's invoked with 3 argments:

-  `value` means the current array item in the iteration;
-  `index` means iteration index; 
- `array` refers to the array where `callBackFn` is passed.

**forEach() will not modify the original array calls it, and it returns undefined**

**forEach() does not work on uninitialized values in the array**

**callBackFn() cannot break, if you want to break the iteration under specific circumstances, you should use the for loop**

```typescript
const arraySparse = [1,3,,7];
let numCallbackRuns = 0;

arraySparse.forEach(function(element){
  console.log(element);
  numCallbackRuns++;
});

console.log("numCallbackRuns: ", numCallbackRuns);

// 1
// 3
// 7
// numCallbackRuns: 3
```

`Array.prototype.forEach()` is always used to calculate, validate or print the items in an array, if you want to modify the value, you should use `Array.prototype.map()`

## Array map()

`Array.prototype.map()` is a function for modifying each item in an array using a call back.

Same as the `Array.prototype.forEach()`, `map()` also needs a `callBackFn()` argument:

`callBackFn(value:any, index:number, array:any[])`, the arguments of `callBackFn()` is the same as that in `forEach()`

**`map()` also does not work on uninitialized values, such as empty items**

**`map()` does not modify the original array and will return a new array object**

```typescript
let arr = Array(3);
let arr2 = arr.map(e => "a"); // [<3 empty items>]
let arr3 = arr.fill(null).map(e => "a"); // ["a", "a", "a"]
```

## Array unpack

When you use unpack to assign value(like you assign values from an array to multiple variables):

```typescript
const arr = Array(2).fill(undefined)
let [x, y] = arr // this is also the only way to assign values to multiple variables at the same time in TS.
```

When you want to unpack an array and pass the items within as function arguments, you can use spread syntax `...`

```typescript
let args = [1, 2, 3]
let result = sum(...args)
```

