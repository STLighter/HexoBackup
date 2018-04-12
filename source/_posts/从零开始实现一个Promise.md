---
title: 从零开始实现一个Promise
date: 2018-04-12 11:35:49
categories: JavaScript
tags:
  - JavaScript
---

最近基于[Promises/A+](https://promisesaplus.com/)规范自己实现了一个`Promise`, 通过了[promises-aplus-tests](https://github.com/promises-aplus/promises-tests), 并额外使用`MutationObserver`确保`Promise`中绑定的处理方法作为`microtask`执行, 本文用来记录个人实现的思路.
Github: [https://github.com/STLighter/PromiseImpl](https://github.com/STLighter/PromiseImpl)

---

`Promise`是一个用来表示异步操作状态的对象, 通过`Promise`可以将传统回调式的异步操作变成链式的操作, 使代码更加简洁和易读. 关于`Promise`的具体用法可以参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise). 这里主要讲实现, 具体的用法这里不再赘述.

这里的实现主要分以下几步:
1. 实现构造函数和`.then`的绑定操作;
2. 实现`.then`的链式调用;
3. 处理外部操作返回`Promise`对象和`thenable`对象的情况;
4. 引入`microtask`;
5. 实现`catch`, `finally`和静态方法;
6. 打包以及测试.

其中前`4`项是核心部分.

<!-- more -->

---

### 实现构造函数和`.then`的绑定操作

`Promise`使用`.then`绑定处理方法实际和观察者模式有几分类似, 不同之处在于:

1. `Promise`中绑定的方法只能被执行一次
2. 异步操作完成后再绑定的方法也能执行

综合来看, 只需要将观察者中触发状态保持下来, 再次触发时直接忽略, 而后绑定方法时直接执行即可, 其他的实现与观察者类似即可.

既然要保持状态, 就可以将其表示成一个简单的状态机. 状态机中定义三种异步操作的状态: `pending`, `fulfilled`和`rejected`.  当异步操作成功完成后, `Promise`状态从`pending`转为`fulfilled`, 而如果操作失败, 则状态转为`rejected`. 一个`Promise`可以从`pending`状态转为`fulfilled`或`rejected`, 而在`fulfilled`或`rejected`状态时不能转为任何其他状态.

在构造`Promise`时初始化状态为`pending`, 并且建立数组用来存储`.then`传入的处理方法, 当异步操作完成后依次调用对应的处理方法. 如果`.then`在异步操作完成后才调用, 则直接调用`.then`中想要绑定的处理方法.

实现如下:
```javascript
const PENDING = 'Pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

function MyPromise (fn) {
  // 初始状态为pending
  this.status = PENDING;

  // onFulfilled时的处理函数
  this.onFulfilledCallbacks = [];

  // onRejected时的处理函数
  this.onRejectedCallbacks = [];

  const onFulfilled = value => {
    // 已经触发过回调了就直接忽略
    if(this.status !== PENDING) return;

    // 第一次触发, 存储要返回的结果以便传给后来才加入的处理函数
    this.value = value;

    // 修改当前状态
    this.status = FULFILLED;

    // 依次触发绑定的处理函数
    this.onFulfilledCallbacks.forEach(cb => cb());
  }

  // 与onFulfilled类似
  const onRejected = err => {
    if(this.status !== PENDING) return;
    this.error = err;
    this.status = REJECTED;
    this.onRejectedCallbacks.forEach(cb => cb());
  };

  try {
    // 同步执行传入的函数, 让异步操作通过onFulfilled和onRejected来改变内部状态
    fn(onFulfilled, onRejected);
  } catch (err) {
    onRejected(err);
  }
}

MyPromise.prototype.then = function then (onFulfilled, onRejected) {
  // 设置默认onFulfilled和onRejected方法
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : x => x;
  onRejected = typeof onRejected === 'function' ? onRejected : err => err;
  const onFulfilledCallback = () => {
    // 调用onFulfilled并传入存储的值
    onFulfilled(this.value);
  }
  const onRejectedCallback = () => {
    onRejected(this.error);
  }
  switch(this.status) {
    case FULFILLED:                                                       // 直接调用新加入的处理函数
      onFulfilledCallback();                  
      break;
    case REJECTED:
      onRejectedCallback();
      break;
    case PENDING:                                                         // 状态还为Pending说明异步操作还没完成, 将对应的回调加入等待队列中
      this.onFulfilledCallbacks.push(onFulfilledCallback);
      this.onRejectedCallbacks.push(onRejectedCallback);
      break;
    default:
      throw new TypeError('Unknow promise status.');
  }
}
```

---

### 实现`.then`的链式调用

`.then`的链式调用是为了给前一个`.then`绑定的处理方法绑定后续操作, 因此需要为绑定的处理方法创建一个`Promise`, 通过在这个`Promise`上调用`.then`方法绑定后续操作.

在`.then`方法中创建一个新的`Promise`, 并作为`.then`方法的返回值. 将`.then`中绑定操作的返回值作为新`Promise`的成功回调值, 将其抛出的异常作为新`Promise`的错误回调值处理即可.

实现如下:
```javascript
MyPromise.prototype.then = function then (onFulfilled, onRejected) {
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : x => x;
  
  // 默认onRejected方法中改为了抛出异常, 以便在没有传入错误处理方法时让后面链式调用绑定的错误处理方法来处理
  onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };
  
  // 创建一个新的Promise
  const promise = new MyPromise((resolve, reject) => {
    const onFulfilledCallback = () => {
      try {
        // 将返回值作为新Promise成功返回值
        const ret = onFulfilled(this.value);
        resolve(ret);
      } catch (err) {
        // 将抛出的异常作为新Promise失败返回值
        reject(err);
      }
    }
    const onRejectedCallback = () => {
      // 与onFulfilledCallback类似
      try {
        const ret = onRejected(this.error);
        resolve(ret);
      } catch (err) {
        reject(err);
      }
    }
    switch(this.status) {
      case FULFILLED: 
        onFulfilledCallback();
        break;
      case REJECTED:
        onRejectedCallback();
        break;
      case PENDING:
        this.onFulfilledCallbacks.push(onFulfilledCallback);
        this.onRejectedCallbacks.push(onRejectedCallback);
        break;
      default:
        throw new TypeError('Unknow promise status.');
    }
  });

  // 返回这个Promise
  return promise;
}
```

---

### 处理外部操作返回`Promise`对象和`thenable`对象的情况

上面的实现是没有考虑外部操作返回`Promise`对象的情况. 如果外部操作返回的是一个`Promise`对象, 那就需要等待这个`Promise`对象中的异步操作执行完后才能调用当前`Promise`上绑定的处理方法, 即在外部返回的`Promise`对象上绑定处理方法去改变当前`Promise`的状态.

能返回`Promise`的外部操作包括两部分:

1. 构造方法中的异步操作`resolve`一个`Promise`对象(这里只包含`resolve`而不提`reject`, 因为`reject`中应传递错误原因, 即使传入`Promise`对象也应该视为错误原因不做特殊处理);
2. 绑定的处理方法返回一个`Promise`对象.

也就是说, 在处理外部返回的`Promise`对象时有两个地方可以处理, 要么在异步操作调用`resolve`时(即在构造函数中传给`fn`的方法中)处理, 要么在`.then`中`onFulfilledCallback`和`onRejectedCallback`得到返回值`ret`时处理. 其中前者可以同时处理`1`,`2`(因为`.then`中最终也是将`ret`放入新`Promise`的`resolve`中), 而后者只处理`2`. [Promise/A+](https://promisesaplus.com/)规范只规定了处理`2`的情况, 实际上用后者的处理方式也可以通过测试用例, 但我们实际使用的`Promise`都是用前者的处理方式. 这个可以从下面的例子看出:

```javascript
const inner = new Promise(resolve => resolve(1));
const outer = new Promise(resolve => resolve(inner));

outer.then(v => console.log(v)); // 1
```

如果采用后者的处理, `v`将会是`inner`这个`Promise`对象.

本文将采用`1`的处理方式.

此外, [Promise/A+](https://promisesaplus.com/)规范中主要添加了对`thenable`的支持, 即使外部处理函数返回一个有`.then`方法的其他对象, 也像返回一个`Promise`对象一样处理. 但由于无法保证外部`thenable`对象的行为与`Promise`一样规范, 需要添加一些判断来约束不规范的行为(例如`thenable`中`.then`中`resolve`多次).

另外值得注意的是, 可能外部返回的`Promise`对象和`thenable`对象执行完成后再返回一个`Promise`或`thenable`, 也就是说需要对外部返回的结果递归去处理.

具体的操作已经在规范[The Promise Resolution Procedure](https://promisesaplus.com/#the-promise-resolution-procedure)部分描述的很清楚, 这里直接按照其逻辑实现:

```javascript
const resolution = (promise, x, resolve, reject) => {
  // 循环引用问题, 避免外部返回promise对象自己
  if(promise === x) {
    return reject(new TypeError('Circular references'));
  }

  // 记录被调用次数
  let called = false;

  const resolvePromise = y => {
    // 如果调用过就直接忽略, 避免resolvePromise或者rejectPromise多次调用
    if(called) return;
    called = true;
    // 递归处理返回值
    resolution(promise, y, resolve, reject);
  };

  const rejectPromise = r => {
    if(called) return;
    called = true;
    reject(r);
  }

  // 返回值是promise的情况
  if(x instanceof MyPromise) {
    // 处理完成后返回, 不会执行后面的if
    return x.then(resolvePromise, rejectPromise);
  }
  if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
    let then;
    try {
      then = x.then;
      if(typeof then === 'function') {
        // thenable的情况, 处理完以后返回
        return then.call(x, resolvePromise, rejectPromise);
      }
    } catch (err) {
      // 取x.then出错或者thenable执行时抛出错误, 处理完后返回
      if(called) return;
      called = true;
      return reject(err);
    }
  }
  // 非null, object或function, 或者不包含.then, 或者.then不是function的情况
  resolve(x);
}

function MyPromise (fn) {
  this.status = PENDING;
  this.onFulfilledCallbacks = [];
  this.onRejectedCallbacks = [];

  // 外部返回的promise或者thenable完成后才能执行这里的操作
  const onFulfilled = value => {
    if(this.status !== PENDING) return;
    this.value = value;
    this.status = FULFILLED;
    this.onFulfilledCallbacks.forEach(cb => cb());
  };
  const onRejected = err => {
    if(this.status !== PENDING) return;
    this.error = err;
    this.status = REJECTED;
    this.onRejectedCallbacks.forEach(cb => cb());
  };
  const resolve = value => {
    // 递归去处理外部返回值
    resolution(this, value, onFulfilled, onRejected);
  }
  const reject = err => {
    onRejected(err);
  }
  try {
    fn(resolve, reject);
  } catch (err) {
    reject(err);
  }
}

```

代码里面对于几种抛出错误情况都对`called`做了处理, 实际规范中没有要求`x.then`出错也限制`resolvePromise`或者`rejectPromise`的调用, 属于未定义的行为, 这里按自己的理解做的处理. 

---

### 引入`microtask`

规范[2.2.4](https://promisesaplus.com/#point-34)要求`.then`中绑定的`onFulfilled`或者`onRejected`只有当执行栈中只有平台代码时才能调用. 简单的说就是不能同步执行`onFulfilled`和`onRejected`, 只能当做单独的任务异步去执行.

通常异步执行可以直接扔到`setTimeout`里面, 事实上这样做也符合A+规范. 不过浏览器中的`Promise`中的`onFulfilled`比`setTimeout`中的代码更优先执行, 具体可以参考[Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)中对`microtasks`的介绍. 简单的说就是`setTimeout`中的任务属于`tasks`而`Promise`上绑定的方法属于`microtasks`, 只要有`microtasks`等待执行, "执行栈中只有平台代码"时都会去执行.

这里我参考[asap](https://github.com/kriskowal/asap)中部分代码使用`MutationObserver`实现了一个`microtask`函数(另外也有现成的跨浏览器的microtask实现[immediate](https://github.com/calvinmetcalf/immediate), 目前我还没去阅读其源码).

`MutationObserver`实现`microtask`的核心原理是在`dom`元素做改变后对应监听`MutationObserver`触发的回调是`microtask`, 因此需要通过改变`dom`触发回调, 并在回调中执行`onFulfilled`或者`onRejected`. 同时, 上面`Promise`实现中会将`onFulfilledCallbacks`或者`onRejectedCallbacks`数组中方法都加入到`microtask`队列中, 为了减少`dom`修改, 可以将加入的方法放在一个队列里面, 在一个`microtask`里面执行即可.

先来看看调用`onFulfilled`和`onRejected`时的变化:

```javascript
\\ Promise.prototype.then
\\ ...

const onFulfilledCallback = () => {
  // 套在 try catch 外保证异常能够正确捕获
  microtask(() => {
    try {
      const ret = onFulfilled(this.value);
      resolve(ret);
    } catch (err) {
      reject(err);
    }
  })
}
const onRejectedCallback = () => {
  microtask(() => {
    try {
      const ret = onRejected(this.error);
      resolve(ret);
    } catch (err) {
      reject(err);
    }
  })
}
```

其中`microtask`的实现:

```javascript
const getWebMicrotask = () => {
  const scope = window || self;
  if(scope) {
    const MutationObserver = scope.MutationObserver || scope.WebKitMutationObserver || scope.MozMutationObserver;
    const document = scope.document;
    if(MutationObserver && document) {
      // 保存在下次"执行栈中只有平台代码"前依次等待执行的任务
      const queue = [];
      const capacity = 1024;
      let index = 0;
      // 在监听触发时作为一个microtask执行的函数
      const run = () => {
        while(index < queue.length) {
          queue[index++]();
          // 边执行边移除避免过大的内存消耗
          if(index >= capacity) {
            queue.splice(0, index);
            index = 0;
          }
        }
        queue.length = 0;
        index = 0;
      }
      // 创建一个dom节点用于MutationObserver监听
      const target = document.createTextNode('');
      // 监听TextNode的数据变化
      const observerInitConfig = {
        characterData: true
      }
      let data = 1;
      observer = new MutationObserver(run);
      observer.observe(target, observerInitConfig);

      return function microtask (fn) {
        if(!queue.length) {
          // 当队列为空时需要改变data触发一次microtask
          target.data = data = - data;
        }
        // 否则只需要加入队列中, 与队列中其他任务在同一个microtask里面执行
        queue.push(fn);
      }
    }
  }

  // 当不支持MutationObserver时采用setTimeout将就一下
  return setTimeout;
}

module.exports = getWebMicrotask();

```

---

### 实现`catch`, `finally`和静态方法

`catch`, `finally`没有太多内容, 直接上代码:

```javascript
catch (onRejected) {
  return this.then(null, onRejected);
}
finally (onFinally) {
  return this.then(onFinally, onFinally);
}
```

`Promise.resolve`中对传入的对象为`Promise`对象时有[特殊要求](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve), 需要直接返回这个对象, 因此要特殊判定.

```javascript
MyPromise.resolve = value => {
  if(value instanceof MyPromise) {
    return value;
  } else {
    return new MyPromise(resolve => resolve(value));
  }
};

MyPromise.reject = err => new MyPromise((resolve, reject) => reject(err));
```

`Promise.all`需要一个数组存储中间结果, 直到最后一个结果返回再`resolve`. 而`Promise.race`只需要返回一个结果, 其他的结果再返回就忽略.

```javascript
MyPromise.all = list => new MyPromise((resolve, reject) => {
  let left = list.length;
  const values = [];
  // 空数组直接返回
  if(left === 0) return resolve(values);
  const resolverFactory = index => {
    return value => {
      --left;
      values[index] = value;
      if(left === 0) {
        // 全部返回完成
        resolve(values);
      }
    };
  };
  const rejector = err => reject(err);
  list.forEach((p, i) => {
    Promise.resolve(p).then(resolverFactory(i), rejector);
  });
});

MyPromise.race = list => new MyPromise((resolve, reject) => {
  let called = false;
  const resolver = value => {
    // 不是第一个返回就忽略
    if(called) return;
    // 第一个返回的
    called = true;
    resolve(value);
  };
  const rejector = err => {
    if(called) return;
    called = true;
    reject(err);
  }
  list.forEach(p => MyPromise.resolve(p).then(resolver, rejector));
});
```

---

### 打包以及测试

这块的代码模块化规范走的是`CMD`, 于是这里用`webpack`打了下包. 直接装了个`4.5`版本的.

这里需要提一下的是, 在`microtask`实现的代码中实际上我先检查了`process.nextTick`, 存在的时候就直接用`nextTick`, 否则再检查`MutationObserver`. 这样的代码在打包时遇到了问题.

一是默认`target: 'web'`, 然后`libraryTarget: "umd"`打出来的包会直接用`window`, 这样在`node`环境下会报错, 我这里直接分成两个单独的包去打, 一个设置`target: 'web'`, 另一个设置`target: 'node'`.

另一个问题是新版本`webpack`使用`target: 'web'`时默认`polyfill`了`node`中的一些方法, 就包括`process.nextTick`, 而且还是用`setTimeout`实现的...这样前面的`microtask`就无效了...于是用`node: false`屏蔽这些`polyfill`.

配置如下:
```javascript
var webpack = require('webpack');
const path = require('path');

const webConfig = {
  mode: "production",
  entry: './src/promise.js',
  output: {
    filename: 'promise.js',
    path: path.resolve(__dirname, 'dist'),
    library: "Promise",
    libraryTarget: "umd"
  },
  node: false,
  target: 'web',
  plugins: [
    new webpack.DefinePlugin({ TARGET: JSON.stringify('web')})
  ]
}

const nodeConfig = {
  mode: "production",
  entry: './src/promise.js',
  output: {
    filename: 'promise.node.js',
    path: path.resolve(__dirname, 'dist'),
    library: "Promise",
    libraryTarget: "commonjs2"
  },
  node: false,
  target: 'node',
  plugins: [
    new webpack.DefinePlugin({ TARGET: JSON.stringify('node')})
  ]
}

module.exports = [webConfig, nodeConfig];
```

既然加了`target`, 那`microtask`中平台判断就不那么重要了, 于是写成如下形式:
```javascript
const getWebMicrotask = () => {
  // ...
}

const getNodeMicrotask = () => {
  if(process && process.nextTick) return process.nextTick;
  return setTimeout;
}

if(TARGET === 'web') {
  module.exports = getWebMicrotask();
} else if (TARGET === 'node') {
  module.exports = getNodeMicrotask();
} else {
  module.exports = setTimeout;
}

```

在`package.json`中添加`build`命令:
```json
"scripts": {
  "build": "webpack",
  // ...
}
```

使用`npm run build`命令即可.

接着使用[promises-aplus-tests](https://github.com/promises-aplus/promises-tests)跑测试用例.

先按文档实现`adaptor`:

```javascript
// adapter.spec.js
const Promise = require('../dist/promise.js');
module.exports = {
  resolved: Promise.resolve,
  rejected: Promise.reject,
  deferred: function () {
    const defer = {};
    defer.promise = new Promise(function (resolve, reject) {
        defer.resolve = resolve;
        defer.reject = reject;
    });
    return defer;
  }
};
```

运行测试用例代码:

```javascript
// aplus-tests.spec.js
const adapter = require('./adapter.spec.js');

describe("Promises/A+ Tests", function () {
  require("promises-aplus-tests").mocha(adapter);
});

```

为了能测试浏览器中的效果还安装了`Karma`, `Karma`配置如下:

```javascript
module.exports = function(config) {
  config.set({
    basePath: '',
    frameworks: ['browserify', 'mocha'],
    files: [
      'test/**/*.spec.js'
    ],
    exclude: [
    ],
    preprocessors: {
      'test/**/*.spec.js': ['browserify']
    },
    browserify: {
      transform: ['brfs']
    },
    reporters: ['progress'],
    port: 9876,
    colors: true,
    logLevel: config.LOG_INFO,
    autoWatch: false,
    browsers: [!process.env.TRAVIS ? 'Chrome' : 'ChromeNoSandbox'],
    customLaunchers: {
      ChromeNoSandbox: {
        base: 'ChromeHeadless',
        flags: ['--no-sandbox']
      }
    },
    singleRun: true,
    concurrency: Infinity
  })
}

```

其中使用了`browserify`是因为`promises-aplus-tests`中使用了`fs`, 为了在浏览器中能运行需要转成内联文件.(另外这里含有`travis`相关的一些配置)

另外我还加入了一些`microtask`的测试用例, 但目前还没加静态方法的测试:

```javascript
// isolated-tests.spec.js

const Promise = require('../dist/promise.js');
const assert = require('chai').assert;

// more infomation check https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/
describe('Call callbacks as microtasks', function () {
  it('Should call onFulfilled before setTimeout', function (done) {
    const order = [];
    setTimeout(function() {
      order.push(2);
    }, 0);
    Promise.resolve().then(function() {
      order.push(1);
    });
    setTimeout(function () {
      assert.deepEqual(order, [1, 2]);
      done();
    }, 50);
  });
  it('Should call onRejected before setTimeout', function (done) {
    const order = [];
    setTimeout(function() {
      order.push(2);
    }, 0);
    Promise.reject().then(null, function() {
      order.push(1);
    });
    setTimeout(function () {
      assert.deepEqual(order, [1, 2]);
      done();
    }, 50);
  });
  it('Should run chain before setTimeout', function (done) {
    const order = [];
    setTimeout(function() {
      order.push(5);
    }, 0);
    Promise.resolve().then(function() {
      order.push(1);
    }).then(function() {
      order.push(2);
      throw '';
    }).then(null, function() {
      order.push(3);
      throw '';
    }).catch(function() {
      order.push(4);
    });
    setTimeout(function () {
      assert.deepEqual(order, [1, 2, 3, 4, 5]);
      done();
    }, 50);
  });
});

```

在`package.json`中添加跑测试的命令:
```json
"scripts": {
  "test": "webpack && karma start",
  // ...
}
```

执行`npm run test`即可运行测试.

![promise test](/uploads/promise.png)

另外配置了`travis`:

```
// .travis.yml
language: node_js
node_js:
  - "6"
dist: trusty
sudo: required
addons:
  chrome: stable
script:
  npm run test
```

到此为止, 整个`Promise`实现的小工程就搭建完成了, 目前还没有做的是:

1. 添加静态方法等的测试用例
2. 检查测试覆盖率
3. 没有测试node环境
4. ~~删库跑路~~

欢迎`pr`...