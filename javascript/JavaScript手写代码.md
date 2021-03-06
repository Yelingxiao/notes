## JavaScript手写代码



[TOC]



### 手写 call、apply 及 bind 函数

首先从以下几点来考虑如何实现这几个函数

- 不传入第一个参数，那么上下文默认为 `window`

- 改变了 `this` 指向，让新的对象可以执行该函数，并能接受参数

  

  那么我们先来实现 `call`

```javascript
Function.prototype.myCall = function(context = window) {
  if (typeof this !== 'function') throw new TypeError('Error')
  console.log(context, this)
  context.fn = this
  const arg = [...arguments].slice(1)
  const result = context.fn(...arg)
  delete context.fn
  return result
}
```

以下是对实现的分析：

- 首先 `context` 为可选参数，如果不传的话默认上下文为 `window`
- 接下来给 `context` 创建一个 `fn` 属性，并将值设置为需要调用的函数
- 因为 `call` 可以传入多个参数作为调用函数的参数，所以需要将参数剥离出来
- 然后调用函数并将对象上的函数删除

以上就是实现 `call` 的思路，`apply` 的实现也类似，区别在于对参数的处理，所以就不一一分析思路了

```javascript
Function.prototype.myApply = function(context = window) {
  if (typeof this !== 'function') throw new TypeError('Error')
  context.fn = this
  let result = null
  // 处理参数和 call 有区别
  arguments[1] ? result = context.fn(...arguments[1]) : result = context.fn()
  delete context.fn
  return result
}
```

`bind` 的实现对比其他两个函数略微地复杂了一点，因为 `bind` 需要返回一个函数，需要判断一些边界问题，以下是 `bind` 的实现

```javascript
Function.prototype.myBind = function (context = window) {
  if (typeof this !== 'function') throw new TypeError('Error')
  const _this = this
  const args = [...arguments].slice(1) 
  // 返回一个函数
  return function F() {
    // 因为返回了一个函数，我们可以 new F()，所以需要判断
    if (this instanceof F) {
      return new _this(...args, ...arguments)
    }
    return _this.apply(context, args.concat(...arguments))
  }
}
```

以下是对实现的分析：

- 前几步和之前的实现差不多，就不赘述了
- `bind` 返回了一个函数，对于函数来说有两种方式调用，一种是直接调用，一种是通过 `new` 的方式，我们先来说直接调用的方式
- 对于直接调用来说，这里选择了 `apply` 的方式实现，但是对于参数需要注意以下情况：因为 `bind` 可以实现类似这样的代码 `f.bind(obj, 1)(2)`，所以我们需要将两边的参数拼接起来，于是就有了这样的实现 `args.concat(...arguments)`
- 最后来说通过 `new` 的方式，在之前的章节中我们学习过如何判断 `this`，对于 `new` 的情况来说，不会被任何方式改变 `this`，所以对于这种情况我们需要忽略传入的 `this`

### 手写一个new操作符

*要创建一个新实例，必须使用new操作符。以这种方式调用构造函数实际上会经历以下4个步骤：*       

1. 创建一个新对象
2. 将构造函数的作用域赋给新对象（因此this就指向了这个对象） 
3. 执行构造函数中的代码（为这个新对象添加属性）
4.   返回新对象。



```javascript
function New(func) {
  let res = {}
  if (func.prototype !== null) res.__proto__ = func.prototype
  let ret = func.apply(res, Array.prototype.slice.call(arguments, 1));
  if ((typeof ret === "object" || typeof ret === "function") && ret !== null) return ret
  return res;
}
```



### 实现一个instanceOf



`instanceof` 可以正确的判断对象的类型，因为内部机制是通过判断对象的原型链中是不是能找到类型的 `prototype`。

```javascript
function instanceOf(left,right) {
    let proto = left.__proto__;
    let prototype = right.prototype
    while(true) {
        if(proto === null) return false
        if(proto === prototype) return true
        proto = proto.__proto__;
    }
}
```

