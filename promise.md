# js处理异步请求进化论

[场景描述](#yinyan)

[1、 地狱回调](#dyht)

[2、 地狱回调优化](#htyh)

[3、Promise](#promise)

[4、Generator](#generator)

[5、Async](#async)

<p id="yinyan">场景描述:

我们常常会遇到这样的业务场景：后一步异步请求依赖前一步异步请求的结果，如果这样的依赖层数比较多，那么初学者很容易写出地狱回调的代码。

以下代码均以setTimeout模拟异步请求。
</p>

<div id="dyht">
<br>

>地狱回调
```javascript
setTimeout(() => {
    console.log('step1');
    setTimeout(() => {
        console.log('step2');
        setTimeout(() => {
            console.log('step3');
        }, 1000);
    }, 1000);
}, 1000);
```
</div>
<div id="htyh">

<br>
以上代码看起来不利于阅读和维护，我们可以稍加优化。

>地狱回调优化
```javascript
function step1() {
    setTimeout(() => {
        console.log('step1');
        step2('step1');
    }, 1000);
}
function step2(data) {
    setTimeout(() => {
        console.log('step2');
        step3('step2');
    }, 1000);
}
function step3(data) {
    setTimeout(() => {
        console.log('step3');
    }, 1000);
}
step1();
```

以上的代码虽然看上去利于阅读了，但是还是没有解决根本问题。而地狱回调的根本问题就是：
1. 嵌套函数存在耦合性，一旦有所改动，就会牵一发而动全身
2. 嵌套函数一多，就很难处理错误

   当然，回调函数还存在着别的几个缺点，比如不能使用try catch捕获错误，不能直接return。

</div>

<div id="promise">

<br>

如何解决回调地狱呢？
* 使用ES6中的Promise,Promise有三种状态：pending/reslove/reject 。pending就是未决，resolve可以理解为成功，reject可以理解为拒绝。
同时Promise常用的三种方法:
    * then 表示异步成功执行后的数据状态变为reslove 
    * catch 表示异步失败后执行的数据状态变为reject 
    * all表示把多个没有关系的Promise封装成一个Promise对象使用then返回一个数组数据。

下面一起来看看如何使用Promise：
>promise
```javascript
var step1 = function () {
    return new Promise(function (resolve, reject) {
        let st = setTimeout(() => {
            resolve('step1');
        }, 1000);
    });
}

var step2 = function () {
    return new Promise(function (resolve, reject) {
        let st = setTimeout(() => {
            resolve('step2');
        }, 1000);
    });
}

var step3 = function () {
    return new Promise(function (resolve, reject) {
        let st = setTimeout(() => {
            resolve('step3');
        }, 1000);
    })
}
step1().then(function (data) {
    console.log(data);
    return step2();
}).then(data => {
    console.log(data);
    return step3();
}).then(data => {
    console.log(data);
})
```
</div>

<div id="generator">

<br>

* Generator 函数是 ES6 提供的一种异步编程解决方案，语法行为与传统函数完全不同。形式上，Generator 函数是一个普通函数，但是有两个特征:
    1. function关键字与函数名之间有一个星号（*）；
    2. 函数体内部使用yield表达式，定义不同的内部状态
>Generator
```javascript
var step1 = function () {
    return new Promise((resolve, reject) => {
        setTimeout(function () {
            resolve("step1");
        }, 1000);
    })
}
var step2 = function () {
    return new Promise((resolve, reject) => {
        setTimeout(function () {
            resolve("step2");
        }, 1000);
    })
}
var step3 = function () {
    return new Promise((resolve, reject) => {
        setTimeout(function () {
            resolve("step3");
        }, 1000);
    })
}
function* g() {
    yield step1();
    yield step2();
    yield step3();

}
var step = g();
step.next().value.then(data => {
    console.log(data);
    return step.next().value;
}).then(data => {
    console.log(data);
    return step.next().value;
}).then(data => {
    console.log(data);
});
```

</div>

<div id="async">

<br>

 * 当然生成器不是最完美的，它的语法让人难以理解，所以ES7推出了async/await (异步等待)。
 >async
```javascript
var step1 = async function () {
    return new Promise(function (resolve, reject) {
        let st = setTimeout(() => {
            resolve('step1');
        }, 1000);
    });
}

var step2 = function () {
    return new Promise(function (resolve, reject) {
        let st = setTimeout(() => {
            resolve('step2')
        }, 1000);
    });
}

var step3 = function () {
    return new Promise(function (resolve, reject) {
        let st = setTimeout(() => {
            resolve('step3')
        }, 1000);
    })
}
async function test() {
    var data1 = await step1();
    console.log(data1);
    var data2 = await step2();
    console.log(data2);
    var data3 = await step3();
    console.log(data3)
}
test();
```
</div>

<p>
最后，期待大家一同交流，共同进步。
</p>

