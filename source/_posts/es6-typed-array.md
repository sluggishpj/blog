---
title: ES6-TypedArray
date: 2018-07-28 15:36:56
tags:
  - es6
  - js
  - array
categories:
  - js
  - es6
---

## 定型数组

用于处理**数值类型**数据的专用数组

<!-- more -->

### 数值数据类型

JS 一个数字用 64 位存储

int8, uint8, int16, uint16, int32, uint32, float32, float64

### 数组缓冲区 ArrayBuffer

```js
let buffer = new ArrayBuffer(10);
console.log(buffer.byteLength); // 10

let buffer2 = buffer.slice(4, 6);
console.log(buffer2.byteLength); // 2
```

### 通过视图 DataView 操作数组缓冲区

DataView 类型是一种通用的数组缓冲区视图。

创建 size 大小的数组缓冲区：`new ArrayBuffer(size)`

#### 获取视图信息

- buffer：视图绑定的数组缓冲区
- byteOffset：DataView 构造函数的第 2 个参数，默认是 0，只有传入参数时才有值
- byteLength：第 3 个参数，默认是缓冲区的长度 byteLength

```js
let buffer = new ArrayBuffer(10);
let view1 = new DataView(buffer);
let view2 = new DataView(buffer, 5, 2); // 包含位于索引 5 和 6 的字节

console.log(view1.buffer === buffer); // true
console.log(view2.buffer === buffer); // true

console.log(view1.byteOffset); // 0
console.log(view2.byteOffset); // 5

console.log(view1.byteLength); // 10
console.log(view2.byteLength); // 2
```

#### 读取和写入数据

`getInt(byteOffset, littleEndian)`：读取位于 byteOffset 后的 int8 类型数据

`setInt(byteOffset, value, littleEndian)`：在 byteOffset 处写入 int8 类型数据

对于其他类型同理。 littleEndian 布尔值，表示是否按照小端序进行读取（小端序是指**最低有效字节位于字节 0** 的字节顺序，详细参考下一小节）

```js
let buffer = new ArrayBuffer(2);
let view = new DataView(buffer);

view.setInt8(0, 5);
view.setInt8(1, -1);

console.log(view.getInt16(0)); // 1535，即 101 1111 1111
console.log(view.getInt8(0)); // 5
console.log(view.getInt8(1)); // -1
```

#### 大小端（Endianness）

arr 的映射是按照运行 JavaScript 的平台的大小端设置（大端或小端）进行的。如果二进制数据的构造是基于某个大小端配置，而解释平台的大小端配置正相反，就会出现问题

大小端的意思是多字节数字（比如前面代码片段中创建的 16 位无符号整型）中的低字节（8 位）位于这个数字字节表示中的右侧还是左侧。

举个例子，设想一个十进制数字 3085，我们需要用 16 位来表示它。如果只是用一个 16 位数字容器，不管大小端配置如何，它会被表示为二进制 00001100 00001101（十六进制 0c0d）

但是如果用两个 8 位数组表示数字 3085，那么大小端设置就会明显影响它在内存中的存储表示：

- 低地址    高地址
- 00001100 00001101 / 0c0d（大端）
- 00001101 00001100 / 0d0c (小端）

如果接收到一个来自于小端系统的表示 3085 的位序列 00001101 00001100，而在大端系统中为其建立视图，得到的值将是 3340（十进制）或者 0d0c（十六进制）。

目前 Web 上最常用的是小端表示方式，但是肯定存在不采用这种方式的浏览器。根据 MDN，这里有一个快速检测 JavaScript 大小端的方法：

```js
var littleEndian = (function () {
  var buffer = new ArrayBuffer(2);
  new DataView(buffer).setInt16(0, 256, true);
  return new Int16Array(buffer)[0] === 256;
})();
```

> 不要把数组 buffer 的底层二进制存储的大小端和给定数字在 JavaScript 程序中如何表示搞混。比如， `(3085).toString(2)` 返回 "110000001101"，加上前面 4 个隐去的 "0" 看起来似乎是大端表示。实际上，这个表示法是基于 16 位视图，而不是两个 8 位字节的表示。要确定 JavaScript 环境的大小端，最好的方法就是前面的 DataView 测试。

#### 定型数组是视图

ES6 定型数组实际上是用于数组缓冲区的**特定类型**的视图。通过每个实例的 `BYTES_PER_ELEMENT` 属性可以查看每个元素的字节数

| 构造函数名称      | 元素尺寸（字节） |
| ----------------- | ---------------- |
| Int8Array         | 1                |
| Uint8Array        | 1                |
| Unit8ClampedArray | 1                |
| Int16Array        | 2                |
| Uint16Array       | 2                |
| Int32Array        | 4                |
| Uint32Array       | 4                |
| Float32Array      | 4                |
| Float64Array      | 8                |

> Unit8ClampedArray 与 Uint8Array 大致相同，唯一的区别在于数组缓冲区的值如果小于 0 或大于 255，Unit8ClampedArray 分别将其转换为 0 或 255

```js
console.log(Uint8Array.BYTES_PER_ELEMENT); // 1
```

#### 创建特定类型的视图

- **方法 1**，传入 DataView 构造函数可接受的参数来创建新的定型数组，可接受的参数：数组缓冲区，可选的比特偏移量、可选的长度值

```js
let buffer = new ArrayBuffer(10);

let view1 = new Int8Array(buffer);
let view2 = new Int8Array(buffer, 5, 2);

console.log(view1.buffer === buffer); // true
console.log(view2.buffer === buffer); // true

console.log(view1.byteOffset); // 0
console.log(view2.byteOffset); // 5

console.log(view1.byteLength); // 10
console.log(view2.byteLength); // 2
```

- **方法 2**，调用构造函数时传入一个数字，表示分配给数组的元素数量，构造函数将创建一个新的缓冲区，并按照数组元素的数量来分配合理的字节数量

```js
let ints = new Int16Array(2);
let floats = new Float32Array(5);

console.log(ints.byteLength); // 4
console.log(ints.length); // 2

console.log(floats.byteLength); // 20
console.log(floats.length); // 5
```

> 如果不传参数，会按照传入 0 来处理

- **方法 3**，将以下任一对象作为唯一的参数传入：
  - 一个定型数组，数组中的每个元素会作为新的元素被复制到新的定型数组中
  - 一个可迭代对象
  - 一个数组
  - 一个类数组对象

### 定型数组与普通数组区别

- **定型数组**的 **length** 属性是一个不可写属性，不能修改，严格模式下会报错
- 通过 `Array.isArray()` 检查**定型数组**返回的是 **false**
- 给 **定型数组**中**不存在的数组索引** 赋值会被忽略

```js
let ints = new Int16Array([25, 50]);

console.log(ints.length); // 2
ints[2] = 5;

console.log(ints.length); // 2
console.log(ints[2]); // undefined
```

- 定型数组会检查数据类型的**合法性**，**0 被用于代替所有非法值**。所有**修改定型数组值的方法执行时都会受到相同的限制**，e.g. 给 `map()` 方法传入的函数返回非法值，最终会用 0 来代替

```js
let ints = new Int16Array(['hi']);
console.log(ints.length); // 1
console.log(ints[0]); // 0
```

- 相对数组，**定型数组缺失的方法**：`concat()`, `shift()`, `pop()`, `splice()`, `push()`, `unshift()`
- 相对数组，**定型数组附加的方法**：`set()` 和 `subarray()`
  - `set()`：将其他数组复制到已有的定型数组。接收两个参数：一个是数组，一个是可选的偏移量，表示开始插入数据的位置，默认 0
  - `subarray()`：提取已有定型数组的一部分作为一个新的定型数组。接收两个参数：可选的开始位置，可选的结束位置（不包含当前位置的数据）

```js
let ints = new Int16Array(4);

ints.set([25, 50]);
ints.set([75, 100], 2);

console.log(ints.toString()); // 2
```

```js
let ints = new Int16Array([25, 50, 75, 100]);

let subints1 = ints.subarray();
let subints2 = ints.subarray(2);
let subints3 = ints.subarray(1, 3);

console.log(subints1.toString()); // 25,50,75,100
console.log(subints2.toString()); // 75,100
console.log(subints3.toString()); // 50,75
```

> 参考书籍：《UNDERESTANDING ECMASCRIPT 6》&《You Don't Know JS》
