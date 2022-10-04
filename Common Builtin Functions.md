# Common Built-in Functions

## Control the number of digit displayed after the decimal point

```typescript
let x = 3.99999999999999999
console.log(x.toFixed(2)) // "4.00" this function will return a string
```

## Use spread syntax to pass props

```react
export default function testComponent() {
	const props = {
        a:1,
        b:2
    }
    return <childrenComponent {...props}></childrenComponent>
    // childrenCompnent has props interface
    /* 
    	{
    		a: number,
    		b: number
    	}
    */
}
```

