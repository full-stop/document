`Promise` 是ES6的特性之一，采用的是 `Promise/A++` 规范，它抽象了异步处理的模式，是一个在JavaScript中实现异步执行的对象。
按照字面释意 `Promise` 具有“承诺”的含义，它承诺当异步处理完成后，回馈一个结果给你！或者你可以将其认为是一个状态机，一旦状态发生了改变，便会触发对应的行为。

`Promise` 最早出现于E语言中(一种给予并列/并行处理设计的编程语言)，JavaScript 引入这一特性，旨在为了规范异步的操作和避免陷入回调地狱。

---

**目录**

* 如何使用？
* Promise的状态
* Promise的简单示例
* 链式调用&&值的传递
* 链式调用&&状态处理
* Promise.all
* Promise.race
* Promise的同步调用
* Promise && callback
* $.Deferred
* 初探Promise的基本实现
* Promise的常见问题

## 如何使用？ ##

`Promise`的使用主要有两种方式，一种是对象实例化操作，它具有固定的使用格式：

```
new Promise(exector);
```

具体示例：

```
var promise = new Promise(function(resolve,reject){
    if(success){
        resolve();
    }else{
        reject();
    }
})
```

`exector` 是一个作为参数的匿名函数，它接收两个参数，一个是 `resolve` ，另一个则是 `reject`,这两个参数都是方法，通过执行两个方法我们可以修改 `Promise` 实例对象的状态，使其再触发对应的行为。

另一种则是静态调用，这些方法本身就是 `Promise` 对象的静态实现：

```
Promise.resolve().then(resolve);
Promise.reject().then(undefined,rejected);
Promise.reject().catch(rejected);
```

静态调用常用于快速执行一个异步操作，例如在我们的程序功能中有一个耗时很长的循环，这个循环的目的只是为了计算一个结果并显示，但是若直接放在程序的上下文的地方，会导致阻塞，常用的方式是将其加入到一个定时器中进行异步操作：

```
setTimeout(function(){
    for(;;){;}
},16);
```

但是学习了静态调用 `Promise`，我们完全可以将这个操作放入到要给 `Promise`的异步回调中。

```
Promise.resolve().then(function(){

    for (var i = 0; i < 100000; i++) {
        if(i  === 100000/2){
            console.log('loop end2');
        }
    }
});
```


对比这两种方式，不难发现通过对象实例的方式，我们可以为实例对象赋予更多的功能，可以根据自身的需要手动的改变 `Promise` 的状态，而使用静态调用的方式，则可以快速的进行异步操作。
需要注意的是，不论是静态的方式还是实例化对象，根据Promise的状态被调用的方法都是以异步方式执行的。但是 `Promise` 对象实例化的过程却依然是同步的。


## Promise的状态 ##


`Promise` 有三种状态：`pendding`、`rejected`、`fulfilled`;

* `pendding`  : 表示初始化状态
* `rejected` : 表示失败状态
* `fulfilled`: 表示成功完成状态

而状态的变化，则需要通过执行对应的方法来完成，

* pendding -> fulfilled 通过 `resolve()` 方法来完成。
* pendding -> rejected ： 通过 `reject()` 方法来完成。

`Promise` 默认的状态是 `pendding` 状态，这种状态出现在实例对象刚刚初始化的情况，结束于 `resolve()` 或者是 `reject()`  方法调用之后。一旦状态发生改变，便无法再修改，也因此说明，状态改变后执行的回调操作 `then` 也只会执行一次。除此之外，在 `Promise` 中主动使用 `throw new Error()` 也可以使 promise的状态改变为 `rejected`。


## Promise的简单示例 ##


`Promise` 的实例对象一旦创建好后，会大致具有以下的操作：

* 初始化状态为`pendding` 。
* 附加 `then`、`catch` 等异步处理方法
* 执行 `exector` 方法(同步的方式)，根据具体的行为来决定是否变更 promise实例对象的状态。

实际上通过静态调用的方式来执行 Promsie，除了不具有 `exector` 其它的都是相同的。
需要着重说明的是 `then` 与 `catch` 这两个方法，它们都是 `Promise`对象的状态回调函数，一旦 promise的状态发生改变，便会对应的进行触发。

```
var promise = new Promise(function(resolve, reject) {
    var num = Math.random() * 5;
    setTimeout(function() {
        if (num >= 2.5) {
            reject('数值过大');
        } else {
            resolve(num);
        }
    }, 1000)
});


promise.then(function(v) {
    console.log('success:' + v);
}, function(v) {
    console.log(v)
});
```

`then` 方法有两个参数：`promise.then(onResolved,onRejected)`，其中 `onResolved` 表示成功（状态变更为 fulfilled）的回调函数，而 `onRejected` 则表示失败（状态变更为 rejected）情况下的回调函数，一般来说第二个参数可以忽略不写，只保留成功的回调方法 `then(onResolved)`，但是如果你只想处理失败的回调函数，那么 `onResolved` 并不能被省去，`promise.then(undefined,onRejected)`。

或者将 `rejected` 的处理单独提取出来是更好的办法：

```
promise.then(function(v) { console.log('success:' + v); }).catch(function(v) { console.log(v) });
```

Promsie 支持这种类似`JQ`的链式调用，并且可以同时连续调用多个 `then` 方法，而这里的 `catch` 方法与我们的 `try..catch` 功能相同，都是用于捕获错误。而且还可以将错误单独的提取出来，这便为我们带来一个非常大的优势那就是哪怕我`then`方法中的 `onResolved` 方法执行错误，也不会阻塞其它代码的执行。

而能够以链式连续多次执行 `then` 方法的原因就在于我们调用 `then` 方法的时候，该方法会返回一个新的 promise 对象，同样的，对于 `catch` 方法而言道理也是相同的，只是 `catch` 不能做到多次调用。
通过这个简单的实例，我们可知一个promise实例对象的执行是同步的，但是根据状态改变的句柄方法是异步执行的，同时这些句柄方法还会再次返回一个新的 `Promise`对象，用于进行链式调用。



## 链式调用&&值的传递 ##

`Promsie`实例对象中的 `then`或者是 `catch` 方法不仅可以接受 `exector` 中通过 `resolve(value)` 或者是 `reject(value)` 传递而来的值，还可以在其回调函数中通过 `return` 语句将值传递给调用链的下一个 `then` 或者是 `catch` 方法。

```
Promise.resolve(1).then(function(v){return v+1}).then(function(v){return v+1}).then(function(v){console.log(v)}) // 3
```

实际上当 `then` 方法执行完成后，会返回一个新的 `Promise` 对象，并且将自己接收的值附加到这个promise对象上作为一个参数值，供调用链上的下一个 `then`或者是`catch`方法读取并处理。

如果去详细的讨论 `resolve(value)` 或者是 `reject(value)`值的传输的话，它主要有以下几种情况：

* resolve(promsieObj) ： 如果接收的是一个promise对象作为参数，则返回的promise对象便是这个作为参数的promsie对象。
* resolve(like-promise)：如果接收的参数是一个类promise对象，则将其转换并返回（带有then）一个新的promsie对象。
* resolve(value)：如果参数只是一个普通的js数据类型值，则返回一个新的promise对象，并且该promise对象的值就是这个参数。

```
Promise.resolve(Promise.resolve(3)).then(function(v){console.log(v)});
Promise.resolve('123').then(function(v){console.log(v)})
```

`resolve` 与 `reject` 基本相同，不同的只是，如果reject接收到的是另一个promise作为参数，则返回的并不是新的promise对象，依然是其本身。


## 链式调用&&状态处理 ##

与ES3中我们会用 `try..catch..finally` 来进行异常的处理，那么在Promise中，异常都会有何种的流程呢？

```
Promise.reject(1).then(function(v) {
    console.log('success:' + v);
    return v
}).catch(function(v) {
    console.log('error:' + v);
        return v
}).then(function(v) {
    console.log(v);
    return v;
}).then(function(v) {
    console.log(v)
});

/*
 * error:1
 * 1
 * 1
 * /
```

从运算的结果上我们可以看出，`then..catch..then` 的结构就类似于ES3中的 `try...catch..finally`;

再看下面的示例，

```
Promise.reject(1).then(function(v) {
    console.log('success:' + v);
    return v
}).catch(function(v) {
    console.log('error:' + v);
        return v
}).then(function(v) {
    console.log(v);
    return v;
}).catch(function(v) {
    console.log('error2：'+v);
    return v;
}).then(function(v){
    console.log(v)
});

/*
 * error:1
 * 1
 * 1
 * /
```
从这个实例中我们就可以得知多个`catch`只会有一个会被触发，并且是最早的那个。

## Promise.all ##

`Promise.all` 可以执行一个由众多 promise对象组成的数组，并返回一个新的 promise对象，新返回的 promise对象其状态会根据所执行的 promise对象数组的状态而定，如果数组中的所有promise对象都是resolved状态，`Promise.all`返回的 promise对象才会触发 resolved状态，否则停止 `Promise.all` 的执行，并返回一个rejected 状态的promsie对象。

```
var p1 = Promise.resolve(1);
var p2 = Promise.resolve(2);
var p3 = Promise.resolve(3);

Promise.all([
    p1,
    p2,
    p3
]).then(function(vs){
    console.log(vs)
})
```

`Promise.all` 这种以执行最慢的那个异步为准的特性，可以使用它来做网页资源的预加载。

## Promise.race ##

与 `Promise.all` 相同，race也可以执行众多promise对象组成的数组，只是不同的是，只要在这个数组有一个Promise状态发生了改变，（resolved或者是rejected）就会使race返回一个新的 promise对象，而且这个对象的状态也是基于 promise数组执行时的状态。

```
var p1 = Promise.resolve(1);
var p2 = Promise.resolve(2);
var p3 = Promise.resolve(3);

Promise.race([
    p1,
    p2,
    p3
]).then(function(vs){
    console.log(vs)
})
```

如果说 `Promise.all` 会以Promise数组中最慢的为准，那么 race 则会以数组中最先执行的那个Promise对象为基准，因此利用这一个特性，可以用 race来设置超时时间。

```
var p1 = new Promise(function(resolve, reject) {
    setTimeout(function() {
        reject('超时')
    }, 5000)
    setTimeout(function() {
        resolve('success')
    }, 10000)
});
var p2 = new Promise(function(resolve, reject) {

    setTimeout(function() {
        resolve('success')
    }, 6000)
});
var p3 = new Promise(function(resolve, reject) {

    setTimeout(function() {
        resolve('success')
    }, 6000)
});

Promise.race([p1, p2, p3]);
```


## Promise的同步调用 ##

`Promise` 本身是一个异步对象，当异步对象状态更改时触发对应状态的handle方法。因此如果想让多个promise对象同步执行，必须将具有依赖关系的 promise对象放置到对应的另一个promise对象的回调中。

```
function getAsync(v){

    return new Promise(function(resolve,reject){
        resolve(v);
    });
}

function main(){

    return getAsync(1000).then(pushValue).then(function(){return getAsync(500).then(pushValue)}).then(function(){return getAsync(1500).then(pushValue)})
}

main().then(function(){
    console.log('全部执行完成!');
})
```

## Promise && callback ##
Promise的本质是维护状态，侦测状态，根据状态进行响应。而`callback` 则是将自身作为参数传入到另一个方法中，作为别的方法体的一部分去调用，因此 Promsie要比 Callback灵活的很多。
如果用代码来做对比的话，promise是这样的：

```
Promise.resove(2).then(function(v){return v}).then(function(v){return v})
```

而 callback的方式则是这样的：

```
doAsync1(function () {
  doAsync2(function () {
    doAsync3(function () {
      doAsync4(function () {
    })
  })
})
```

总的来说，Promsie的优势更体现与链式调用，而callback（相比较Promise的劣势）就体现在嵌套使用

## $.Deferred ##

对于 `Promsie` 的实现，`Jquery`有着自己的一套实现方式，那就是JQ的 `$.Deferred` 方法。
通过调用 `$.Deferred` 我们可以获得一个JQ版的异步对象实例。

简单实例：

```
var def = $.Deferred(); //获得一个JQ的异步对象实例。
def.resolve('success').then(function(v){console.log(v)}); // success
```

是不是与ES6的 `promise` 完全一样？如果你真的这样认为那就错了，继续看下面的示例:

```
var def = $.Deferred();
def.then(function(v){console.log(v)});
def.resolve('success');
```

是不是发现了一个很大的不同之处，异步对像实例 `def` 竟然可以通过`resolve()` 方法自己修改自己的状态！而ES6中的`Promsie`标准规定的是异步对象的状态不能手动改变，虽然两者不同，但是也无需大惊小怪，毕竟JQ的 `$.Deferred` 有着自己的实现标准。而且，JQ也提供了另一种受限的异步对象，这个受限的异步对象，基本上就与ES6的 `Promise`基本一致了。

但是这个受限的异步对象必须要配合一定的写法，才能避免被手动更改状态。

```
function getAsync() {
    var def = $.Deferred();
    setTimeout(function() {
        def.resolve('success');
    }, 1000);
    return def.promise(); //返回一个受限制的异步对象实例。
}

var dep = getAsync();

dep.then(function(v) {
    console.log(v);
    return v
}).then(function(v) {
    console.log(v);
    return v
});

```

我们可以通过比较受限与不受限的两种异步对象实例，从而更直观的了解这这两者的区别：

```
console.log($.Deferred());
console.log($.Deferred().promise());
```

通过打印这两种异步对象，我们明显可以看到受限的对象其含有的方法要远远少于没有受限的实例对象，而其中最明显的就是受限的实例对象并不具有 `resolve` 方法，这也就直接的说明了受限的异步对象是无法直接修改自己的状态。

由于使用最多的还是受限的异步对象，所以这里我们就大致的说下受限的异步对象具有的一些方法。


**then**

JQ中异步对象实例的 `then` 方法与ES6的`Promise` 对象实例的 `then` 方法使用格式与功能基本相同，唯一不同的就是JQ对 `then` 方法的回调处理进行了扩展，加入了 `pedding` 状态时的回调。

```
function getAsync() {
    var def = $.Deferred();
    def.notify('loading'); //指定pedding时触发回调，并传入参数。
    setTimeout(function() {
        def.resolve('success');
    }, 1000);
    return def.promise(); //返回一个受限制的异步对象实例。
}

var dep = getAsync();

dep.then(function(v) {
    console.log(v)
}, function() {}, function(v) {
    console.log(v)
});
```

**done/fail/progress**

`done()`、`fail()`、`progress()` 等方法都是对 `then（）` 方法的功能包装。
`done()` 表示`resolved`状态时的处理方法，`fail()` 表示 `rejected` 状态时的处理方法，`progress()`  表示`pendding` 状态时的处理方法。

```
function getAsync() {
    var def = $.Deferred();
    def.notify('loading')
    setTimeout(function() {
        def.reject('fail');
    }, 1000);
    return def.promise(); //返回一个受限制的异步对象实例。
}

var dep = getAsync();

dep.done(function(v){
    console.log(v);
});
dep.fail(function(v){
    console.log(v);
});
dep.progress(function(v){
    console.log(v);
})
```

**always**
通过JQ `Deferred().promise()` 方法返回的受限的异步对象实例中，`always` 方法类似于 `try..catch..finally` 中的 `finally`,不论异步对象的状态是成功还是失败，都会触发该方法。

```
function getAsync() {
    var def = $.Deferred();
    setTimeout(function() {
        def.resolve('success');
    }, 1000);
    return def.promise(); //返回一个受限制的异步对象实例。
}

var dep = getAsync();

dep.always(function(v){console.log(v)});
```

**state**

通过调用 `state()` 方法可以获得当前对象实例的状态。

```
$.Deferred().promise().state(); //pending
```

## 初探Promise的基本实现 ##

学习一门技术或者是一个工具，最好的办法，莫非于了解它们的大致实现，这里我以自己的方式，编写一个简单的 `Promise` 对象。
现在只是一个简单的示例，功能还非常简陋只有 `then、resolve、reject`等功能，而且还有很多bug，但是也足够让我对 `promise` 有更进一步的认识。

```
function Deferred(fn) {

    var _this = this;

    var doneList = []; // 用于保存 then方法中 resolved状态时的回调函数.
    var failCallbck = []; // 用于保存 then方法中 rejected状态时的回调函数.

    this.PromiseValue = undefined; //promsie的值
    this.PromiseStatus = 'pending'; //promise的状态

    function resolve(v) { //resolve状态的执行函数

        setTimeout(function() {    //脱离同步代码，以异步的方式执行 then 方法中的代码。
            _this.PromiseStatus = 'resolved';
            _this.PromiseValue = v;
            doneList.forEach(function(self, index) {
                _this.PromiseValue = self(_this.PromiseValue); //保存then方法中回调的return值，以供链式调用时下个then回调函数使用。
            });
        }, 0);

    }

    function reject(v) { //reject状态的执行函数

        var idx; //定位failCallbck中最后的失败处理函数的索引。 eg: [1,...,n] 1表示最早最后，n表示最先最近的
        setTimeout(function() { //脱离同步代码，以异步的方式执行 then 方法中的代码。

            _this.PromiseStatus = 'rejected';
            _this.PromiseValue = v;
            failCallbck.forEach(function(f, i) {
                if (typeof f != 'undefined' && typeof f == 'function') {
                    idx = i;
                    _this.PromiseValue = f(_this.PromiseValue);
                    return; //对于异常处理函数，只会执行最后的那一个。
                }
            });

            _this.PromiseStatus = 'resolved';

            //遍历执行doneList中 resolved 状态时的回调函数，但是忽略rejected处理函数之前的所有resolve 状态的回调函数
            for (var i = idx + 1; i < doneList.length; i++) {
                _this.PromiseValue = doneList[i](_this.PromiseValue);
            }

        }, 0)

    }

    this.then = function(done, faill) {

        if (this.PromiseStatus === 'pending') {
            doneList.push(done);
            failCallbck.push(faill);
        } else if (this.PromiseStatus === 'resolved') {
            done();
        } else {
            failCallbck = faill;
        }

        return this;

    }

    try {
        fn(resolve, reject);
    } catch (e) {
        throw new Error(e);
    }
}

```
调用：

```
var def1 = new Deferred(function(resolve, reject) {
    resolve(1);
});
var def2 = new Deferred(function(resolve, reject) {
    reject(2);
});

def1.then(function(e) {
    console.log('success:' + e);
    return e + 1;
}, function(e) {
    console.log('error:' + e);
    return e + 1;
}).then(function(v) {
    console.log(v);
});

def2.then(function(e) {
    console.log('success:' + e);
    return e + 1;
}, function(e) {
    console.log('error:' + e);
    return e + 1;
}).then(function(v) {
    console.log(v);
});
```

## Promise的常见问题 ##

### Promise的同步与异步执行 ###

```
var p = new Promise(function(resolve,reject){
    console.log(1);
    resolve(2);
    console.log(4)
});

p.then(function(){
    console.log(3);
})
console.log(5)
```
在实例化对象的时候，代码的执行依然是同步执行，而实例化对象的状态回调函数 `then,catch` 才是异步执行。

### 状态改变 ###

```
var p = new Promise(function(resolve,reject){
    resolve('success1');
    reject('error1');
    resolve('success2');
});

p.then(function(e){console.log(e)}).catch(function(v){console.log(v)})
```

`Promise` 的状态一旦确定将无法改变。

### 异常情况下的Promise ###

```
Promise.resolve(1).then(function(v) {
    console.log(v);
    return new Error('error!!!');
}).then(function(v) {
    console.log(v);
}).catch(function(e) {
    console.log(e);
});
```

`resolve()` 进入了第一个`then`的回调，虽然返回了一个 `error` 类型，但是是通过 `return` 返回的，所以它将会被作为第二个 `then` 的值接收，并不会改变promsie的状态，所以后面的 `catch` 便不会被触发。
如果想触发`catch`也很简单，只需要使用下面两种方式的任何一种即可。

* `return new Promise().reject(1)`
* `throw new Error('xxx')`

```
Promise.resolve(1).then(function(v) {
    console.log(v);
    return new Error('error!!!');
}).then(function(v) {
    return Promise.reject(1);
}).catch(function(e) {
    console.log(e);
});
```

虽然触发了最后的 `catch` 回调，但是否与 `Promise` 定义的标准相悖呢？毕竟 promsie对象的状态一经发生，便无法改变的...实际上并不是如此，因为我们之前已经说过， `then、catch` 等都会返回要给新的 `promise` 对象，而且这里 `return Promise.reject(1)` 返回的本身就是一个新的对象。


### 值的穿透 ###

```
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log)
```

`.then` 或者 `.catch` 的参数期望是函数，传入非函数则会发生值穿透。


---

> 参考
> http://liubin.org/promises-book/ (开源Promise 迷你书)
> https://zhuanlan.zhihu.com/p/30797777  Promise 必知必会（十道题）