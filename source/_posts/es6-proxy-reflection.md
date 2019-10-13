---
title: ES6-Proxy-Reflection
date: 2018-09-25 23:24:45
tags:
- es6
- js
- proxy
- reflection
categories:
- js
- es6
---

## 说明

Proxy & Reflection

<!-- more -->

## 代理和反射

| proxy trap               | 覆盖的特性                                                   | 默认特性                           |
| ------------------------ | ------------------------------------------------------------ | ---------------------------------- |
| get                      | 读取一个属性                                                 | Reflect.get()                      |
| set                      | 写入一个属性                                                 | Reflect.set()                      |
| has                      | in 操作符                                                    | Reflect.has()                      |
| deleteProperty           | delete 操作符                                                | Reflect.deleteProperty()           |
| getPrototypeOf           | Object.getPrototypeOf()                                      | Reflect.getPrototypeOf()           |
| setPrototypeOf           | Object.setPrototypeOf()                                      | Reflect.setPrototypeOf             |
| isExtensible             | Object.isExtensible()                                        | Reflect.isExtensible               |
| preventExtensions        | Object.preventExtensions()                                   | Reflect.preventExtensions()        |
| getOwnPropertyDescriptor | Object.getOwnPropertyDescriptor()                            | Reflect.getOwnPropertyDescriptor() |
| defineProperty           | Object.defineProperty()                                      | Reflect.defineProperty()           |
| ownKeys                  | Object.keys()、<br />Object.getOwnPropertyNames()、<br />Object.getOwnPropertySymbols() | Reflect.ownKeys()                  |
| apply                    | 调用一个函数                                                 | Reflect.apply()                    |
| construct                | 用  new 调用一个函数                                         | Reflect.construct()                |



## 创建代理

### 没有陷阱的代理

```js
let target = {}
let proxy = new Proxy(target, {})

proxy.name = 'proxy'
console.log(proxy.name) // proxy
console.log(target.name) // proxy

target.name = 'target'
console.log(proxy.name) // target
console.log(target.name) // target
```

> 代理将所有操作直接 转发到目标。 proxy.name 和 target.name 引用的都是 target.name



### set(trapTarget, key, value, receiver)

* trapTarget: 用于接收属性（代理的目标）的对象
* key: 要写入的属性键（字符串 或 Symbol 类型）
* value: 被写入属性的值
* receiver: 操作发生的对象（通常是代理）

```js
let target = {
    name: 'target'
}

let proxy = new Proxy(target, {
    set(trapTarget, key, value, reciver) {
        // 忽略不希望受到影响的已有属性
        if(!trapTarget.hasOwnProperty(key)) {
            if(isNaN(value)) {
                throw new Error('属性必须是数字')
            }
        }

        // 添加属性
        return Reflect.set(trapTarget, key, value, reciver)
    }
})

proxy.count = 1
console.log(proxy.count) // 1
console.log(target.count) // 1

// 抛出错误
proxy.anotherName = 'proxy'
```



### get(trapTarget, key, receiver)

### has(trapTarget, key): 返回 false 

### deleteProperty(trapTarget, key)

* 直接返回 false 能阻止删除操作。返回 true 未必删除了，只有通过 Reflect.deleteProperty 才可能删除成功



### getPrototypeOf(trapTarget)

* 如果传入的参数不是对象，则 Reflect.getPrototypeOf() 方法会抛出错误，而 Object.getPrototypeOf() 方法则会在操作执行前先将参数强制转换为一个对象

### setPrototypeOf(trapTarget, proto)

* Reflect.setPrototypeOf() 方法返回一个**布尔值**来表示操作是否成功，成功时会返回 true，失败时返回 false。而 Object.setPrototypeOf() 方法一旦失败就会抛出一个错误
* Object.setPrototypeOf() 方法返回第一个参数作为它的返回值



### isExtensible(target) & preventExtensions(target)

* 传入非对象值时，Object.isExtensible() 返回 false，而 Reflect.isExtensible() 则抛出一个错误
* Object.preventExtensions() 的结果返回的是 其参数，Reflect.preventExtensions() 的参数如果不是对象会抛出错误，如果是一个对象，操作成功时 返回 true，失败时返回 false



### defineProperty(trapTarget, key, descriptor) & getOwnPropertyDescriptor(trapTarget, key)

* defineProperty 陷阱在操作成功后返回 true，否则返回 false
* Object.defineProperty() 返回 false 时会抛出错误。可以让陷阱返回 true 并且不调用 Reflect.defineProperty() 方法，可以让 Object.defineProperty() 方法静默失败
* getOwnPropertyDescriptor 的返回值必须是 null，undefined 或一个对象。如果返回对象，则对象自己的属性只能是 enumerable, configurable, value, writable, get 和 set，否则报错
* Object.defineProperty() 方法 和 Reflect.defineProperty() 方法只有返回值不同：Object.defineProperty() 方法返回第一个参数， Reflect.defineProperty() 与操作有关，成功返回 true，失败返回 false
* Object.getOwnPropertyDescriptor() 传入原始值作为第一个参数，内部会将其强制转换为一个对象。Reflect.getOwnPropertyDescriptor() 传入原始值会报错



### ownKeys(trapTarget)

通过返回一个数组的值可以覆写其行为。这个数组被用于 Object.keys()、Object.getOwnPropertyNames()、Object.getOwnPropertySymbols() 和 Object.assign()。Object.assign() 方法用数组来确定需要复制的属性

* Object.getOwnPropertyNames() 和 Object.keys() 方法返回的结果将 Symbol 类型的属性名排除在外
* Object.getOwnPropertySymbols() 方法返回的结果将字符串类型的属性名排除
* Object.assign() 方法支持字符串 和 Symbol 两种类型

```js
let proxy = new Proxy({}, {
    ownKeys(trapTarget) {
        return Reflect.ownKeys(trapTarget).filter(key => {
            return typeof key !== 'string' || key[0] !== '_'
        })
    }
})

let nameSymbol = Symbol('name')
proxy.name = 'proxy'
proxy._name = 'private'
proxy[nameSymbol] = 'symbol'

let names = Object.getOwnPropertyNames(proxy)
let keys = Object.keys(proxy)
let symbols = Object.getOwnPropertySymbols(proxy)

console.log(names) // [ 'name' ]
console.log(keys) // [ 'name' ]
console.log(symbols) // [ Symbol(name) ]
```



### apply & construct 陷阱

使用 **new** 操作符调用函数，则执行 [[Construct]] 方法；若不用，则执行 [[Call]] 方法此时会执行 apply 陷阱

* Reflect.apply(trapTarget, thisArg, argumentsList)
* Reflect.construct(trapTarget, argumentsList)



### 可撤销代理

可以使用 Proxy.revocable() 方法创建可撤销的代理，该方法与 Proxy 构造函数相同的参数：目标对象和代理处理程序。返回值是具有以下属性的对象：

* proxy 可撤销的代理对象
* revoke 撤销代理要调用的函数

当调用 revoke() 函数时，不能通过 proxy 执行进一步的操作。任何与代理对象交互的尝试都会触发代理陷阱抛出错误

```js
let target = {
    name: 'target'
}

let {proxy, revoke} = Proxy.revocable(target, {})

console.log(proxy.name) // target

revoke()

console.log(proxy.name) // TypeError: Cannot perform 'get' on a proxy that has been revoked
```



> 参考书籍：《UNDERESTANDING ECMASCRIPT 6》