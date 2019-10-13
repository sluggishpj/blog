---
title: JavaScript忍者秘籍笔记06之原型与面向对象
date: 2017-10-30 00:00:57
tags:
- js
- js忍者秘籍
- 原型
- 对象
categories:
- js
- js忍者秘籍
---

## 实例化和原型
### 对象实例化
#### 协调引用
JavaScript中的每个对象，都有一个名为constructor的隐式属性，该属性引用的是创建该对象的构造器，由于prototype是构造器的一个属性，所以每个对象都有一种方式可以找到自己的原型。
<!-- more -->
``` js
function Ninja() {
    this.swung = false;
    this.swingSword = function() {
        return !this.swung;
    };
}
Ninja.prototype.swingSword = function() {
    return this.swung;
};
var ninja = new Ninja();

// 控制台
> ninja.swingSword
ƒ () {
        return !this.swung;
    }
> ninja.__proto__.swingSword
ƒ () {
    return this.swung;
}
> ninja.constructor.prototype.swingSword
ƒ () {
    return this.swung;
}
```

### 通过构造器判断对象类型
``` js
function Ninja() {}
var ninja = new Ninja();
console.log(ninja instanceof Ninja); // true
console.log(ninja.constructor == Ninja); // true

var ninja2 = new ninja.constructor();
console.log(ninja2 instanceof Ninja); // true
```
> 不用知道原有的构造器函数也可以再次创建一个新实例，即便是原始的构造器在作用域内已经不存在了，也可以使用该引用。

### 继承与原型链
实现继承的最好方式，使用一个对象的实例作为另一个对象的原型：
`SubClass.prototype = new SuperClass();`
``` js
function Person() {}
Person.prototype.dance = function() {};

function Ninja() {}

Ninja.prototype = new Person(); // 继承

var ninja = new Ninja();

console.log(ninja instanceof Ninja); // true
console.log(ninja instanceof Person); // true
console.log(ninja instanceof Object); // true
console.log(typeof ninja.dance == "function"); // true
```

> 所有的内置对象，比如Array，包括其原型，都可以扩展它们，但不建议在原始对象上引入新的属性或方法，因为原生对象的原型只有一个实例，可能发生命名冲突

### HTML DOM原型
所有的DOM元素都继承于HTMLElement构造器，通过访问HTMLElement的原型，可以提供扩展任意HTML节点的能力。
``` html
<div id="parent">
    <div id="a">I'm going to be removed.</div>
    <div id="b">Me too!</div>
</div>
<script>
    HTMLElement.prototype.remove = function() {
        if (this.parentNode)
            this.parentNode.removeChild(this);
    };

    var a = document.getElementById("a");
    a.parentNode.removeChild(a);

    document.getElementById("b").remove();

    console.log(!document.getElementById("a")); // true
    console.log(!document.getElementById("b")); // true
</script>
```
> 处理HTMLElement原型时，不能在IE8之前的版本使用

### 扩展数字
``` js
Number.prototype.add = function(num) {
    return this + num;
};
var n = 5;
console.log(n.add(3) == 8); // true
console.log((5).add(3) == 8); // true

// 报错
console.log(5.add(3) == 8);
            ^^
SyntaxError: Invalid or unexpected token
```
> 报错原因，语法解析器不能处理字面量这种情况，最好避免在Number的原型做扩展

### 实例化问题
#### 判断函数是否是作为构造器进行调用的
``` js
function Test() {
    return this instanceof arguments.callee;
}

console.log(Test()); // false
console.log(new Test()); // Test {}
```
> * 通过arguments.callee可以得到当前执行函数的引用。
> * “普通”函数的上下文默认是全局作用域。
> * 利用instanceof操作符测试已构建对象是否构建于指定的构造器。

#### 在调用者上修复该问题
``` js
function User(first, last) {
    // 如果不按正确方法进行调用，则修复这个错误
    if (!(this instanceof arguments.callee)) {
        return new User(first, last);
    }
    this.name = first + '' + last;
}

var xiaoming = new User('小', '明');
console.log(xiaoming.name); // 小明

var xiaohong = User('小', '红');
console.log(xiaohong.name); // 小红
```
> arguments.callee严格模式下禁用！更通用的方法如下：（摘自Effective JavaScript）

``` js
function User(first, last) {
    if (!(this instanceof User)) {
        return new User(first, last);
    }
    this.name = first + '' + last;
}

var xiaoming = new User('小', '明');
console.log(xiaoming.name); // 小明

var xiaohong = User('小', '红');
console.log(xiaohong.name); // 小红
```

## 编写类风格的代码
### 检测函数是否可序列化
函数序列化（function serialization）就是简单接收一个函数，然后返回该函数的源码文本。可以用这种方法检查一个函数在某一个对象中是否存在引用。在大多数浏览器中，序列化会导致函数的toString()方法会被调用。可以使用如下表达式测试一个函数是否能够被序列化。
``` js
/xyz/.test(function() {xyz;}); // 返回true（能序列化）或false（反之）
```

### 子类方法
``` js
// 实现
(function() {
    var initializing = false,
        superPattern =
        /xyz/.test(function() { xyz; }) ? /\b_super\b/ : /.*/;

    Object.subClass = function(properties) {
        var _super = this.prototype;

        initializing = true;
        var proto = new this();
        initializing = false;

        for (var name in properties) {
            proto[name] = typeof properties[name] == "function" &&
                typeof _super[name] == "function" &&
                superPattern.test(properties[name]) ?
                (function(name, fn) {
                    return function() {
                        var tmp = this._super; // this引用的是当前的子类实例
                        this._super = _super[name];
                        var ret = fn.apply(this, arguments);
                        this._super = tmp;
                        return ret;
                    };
                })(name, properties[name]) :
                properties[name];
        }

        function Class() {
            // All construction is actually done in the init method
            if (!initializing && this.init) {
                this.init.apply(this, arguments);
            }
        }
        Class.prototype = proto;
        Class.constructor = Class;
        Class.subClass = arguments.callee;
        return Class;
    };
})();
```
> for...in 循环中的个人理解：
> 1. 要继承的属性是方法: `typeof properties[name] == "function"`
> 2. 自己的原型链中有同名方法: `typeof _super[name] == "function"`
> 3. 要继承的属性（方法）中含有`_super`字符串: `superPattern.test(properties[name])`
> 4. 若不能同时满足1,2,3点，则将要继承的属性作为自己实例的一个同名属性
> 5. 若同时满足1,2,3点，则返回一个新的函数，该函数中调用了要继承的属性（方法），并返回调用的结果。


``` js
// 使用
var Person = Object.subClass({
    init: function(isDancing) {
        this.dancing = isDancing;
    },
    dance: function() {
        return this.dancing;
    }
});

var Ninja = Person.subClass({
    init: function() {
        this._super(false);
    },
    dance: function() {
        // Ninja-specific stuff here
        return this._super();
    },
    swingSword: function() {
        return true;
    }
});

var person = new Person(true);
console.log(person.dance()); // true
console.log(person instanceof Person); // true

var ninja = new Ninja();
console.log(ninja.swingSword()); // true
console.log(ninja.dance()); // false
console.log(ninja instanceof Ninja && ninja instanceof Person); // true
```

### ES6继承
```js
class Animal {
    // 构造方法
    constructor(name) {
        this.name = name;
    }
    // 静态方法，只能透过类名调用
    static animalStaticMethod() {
        console.log('animal static method');
    }
    // 实例方法，只能通过实例对象调用
    sayHi() {
        console.log('sayHi, I am animal ' + this.name);
    }
}

class People extends Animal {
    constructor(name, gender) {
        super(name); // 调用父类的constructor(name)
        this.gender = gender;
    }
    sayHello() {
        super.sayHi(); // 调用父类的sayHi()
        console.log('sayHello, I am a ' + this.gender);
    }
}


Animal.animalStaticMethod(); // animal static method
// 继承父类静态方法
People.animalStaticMethod(); // animal static method


var dog = new Animal('旺旺');
dog.sayHi(); // sayHi, I am  animal 旺旺

var xiaoming = new People('小明', 'boy');
xiaoming.sayHi(); // sayHi, I am animal 小明
xiaoming.sayHello(); // sayHi, I am animal 小明
                     // sayHello, I am a boy
```
> 参考自：http://es6.ruanyifeng.com/#docs/class-extends
