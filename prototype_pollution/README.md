JavaScript is very well known for its weirdness with types and that stuff. One thing that makes the language as it is, are prototypes.

A prototype of an object is basically, as its name suggests, a "skeleton" for the objects that can be derived from it. For example, the prototype of `Object` (the base class in JavaScript) contains these methods and properties:

```js
> let t = {}
undefined
> t.__proto__.
t.__proto__.__proto__             t.__proto__.constructor           t.__proto__.hasOwnProperty        t.__proto__.isPrototypeOf
t.__proto__.propertyIsEnumerable  t.__proto__.toLocaleString        t.__proto__.toString              t.__proto__.valueOf
```

Every object that you create will have those properties, as they are inherited from the prototype:


```js
> t.
t.__proto__             t.constructor           t.hasOwnProperty        t.isPrototypeOf         t.propertyIsEnumerable  t.toLocaleString
t.toString              t.valueOf
```

A thing that makes one wonder is what happens if the prototype (`__proto__`) of the object is edited... what will happen is that every new object will have the edited property:


```js
> t.__proto__.t = "f(t)"
'f(t)'
> let x = {}
undefined
> x.t
'f(t)'
```

Not only you can create new properties, but also edit the previously created ones. For example, you can replace the `toString` method, and replacing it will make the application crash whenever something needs to be represented as a string:

```js
> Object.prototype.toString = "xd"
'xd'
> y.toString()
Uncaught TypeError: y.toString is not a function
```

Some JavaScript functions accept objects as input and do things if a property is set. With this, you could trigger that. As an example, consider this function:

```js
function f(X){
    if(X.isAdmin){
        return "here's my cock daddy";
    }

    return "What are you looking at?";
}
```

If somehow, you manage to edit the Object prototype to set an attribute `isAdmin` with value `true`, you will be able to get the cock.

With this, you should see where things are going if you can *pollute* prototypes.

> The `Object.prototype` property is also a prototype but for all created objects, this means that if you edit it, every object already created and those which will be created will have the edit that you did.