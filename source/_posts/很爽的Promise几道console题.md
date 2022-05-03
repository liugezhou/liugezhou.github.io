---
layout: post
title:  很爽的Promise几道console题
author: liugezhou
date: 2022-04-04
updated: 2022-04-04
category: 
    前端小技
tags:
    Promise
---
> 本篇博文是学习的掘金的一篇博文：[【建议星星】要就来45道Promise面试题一次爽到底(1.1w字用心整理)](https://juejin.cn/post/6844904077537574919)，经过自己一个个问题的学习，整理如下：

## 一.Promise的几道基础题

---

### 题目一 ✨
```javascript
const promise1 = new Promise((resolve,reject)=>{
  console.log('promise1')
})
console.log('1',promise1)
// promise1
// 1 Promise {<pending>}
```
> 先执行构造函数中的代码promise1，然后执行同步代码 1，由于没有resolve或者reject，此时状态为pending

### 
### 题目二 ✨✨
```javascript
const promise = new Promise((resolve,reject)=>{
  console.log(1)
  resolve('success')
  console.log(2)
})
promise.then(()=>{
  console.log(3)
})
console.log(4)
// 1 2 4 3
```
> 1. 首先执行同步代码，打印出1
> 1. 接着resolve的出现，只是将promise的状态改变成了resolved，将success这个值保存了下来。
> 1. 会接着执行同步代码，输出2
> 1. promsie.then是一个微任务，放入到微任务列表，等待宏任务执行完毕后，来执行微任务列表
> 1. 继续执行本轮的宏任务，输出4
> 1. 之后本轮宏任务执行完毕，检查微任务列表发现了这个promise..then，执行输出3

### 
### 题目三 ✨
```javascript
const promise = new Promise((resolve, reject) => {
  console.log(1);
  console.log(2);
});
promise.then(() => {
  console.log(3);
});
console.log(4);
// 1 2 4
```
> 在promise中没有resolve或者reject，因此promise.then不会执行


### 题目四 ✨
```javascript
const promise1 = new Promise((resolve, reject) => {
  console.log('promise1')  // 1 执行构造函数中代码，打印输出
  resolve('resolve1')      // 2 将promise1状态改为resolved，并将结果保存下来
})
const promise2 = promise1.then(res => { // 3. 将这个微任务，放入到微任务队列
  console.log(res)  // 6.执行微任务，打印出结果 resolve1
})
console.log('1', promise1);  //4 打印出promise1的状态为 resolved
console.log('2', promise2);  //5 打印出promise2的状态为 pending，宏任务执行完毕，寻找微任务队列,去到步骤6
// promise1
// 1 Promise{<resolved>}
// 2 Promise{<pending>}
// resolve1
```

### 题目五 ✨
```javascript
const fn = () => (new Promise((resolve, reject) => {
  console.log(1);   //2.执行同步代码，打印1
  resolve('success') //3.更新fn的状态为resolved的Promise，且保存其值
}))
fn().then(res => { // 1.执行fn函数，.then为微任务列表，等待本轮宏任务执行完毕来检查
  console.log(res)  // 4执行完，发现有微列表，打印success
})
console.log('start') // 4 本来宏任务，打印4
// 1 
// start
// success
```
> fn是一个函数且直接返回的是一个Promise，主要是因为fn的调用在start之前的，所以fn里面的代码先执行


### 题目六 ✨
```javascript
const fn = () =>
new Promise((resolve, reject) => {
  console.log(1);     // 3.执行fn函数同步代码，打印1
  resolve("success"); // 4.将fn的结果Promise状态改为resolved，且保存这个值
});
console.log("start"); // 1.直接打印 start
fn().then(res => {		// 2.先去执行调用的fn函数，且将.then存入到微任务列表
  console.log(res);		// 5.打印保存的这个值 success
});
// start
// 1
// success
```
> 这个fn的调用是在start之后才开始执行的，所以先打印start，再去执行fn这个函数


## 二、Promise结合setTimeout

---

### 题目七 ✨
```javascript
console.log('start')  // 1.本轮宏任务开始，打印出 start
setTimeout(() => {		// 2. setTimeout是宏任务，会放到下一轮中去执行
  console.log('time')  // 6.第二轮宏任务开始，打印出 time
})
Promise.resolve().then(() => {  //3.then为微任务，等到本轮宏任务执行完毕，来检查	
  console.log('resolve')		// 5.检查到本轮微任务，打印出resolve，开始执行下一个宏任务
})
console.log('end') 			// 4.本轮宏任务打印出end，然后去检查本轮微任务
// start
// end
// resolve
// time
```

### 题目八  ✨✨
```javascript
const promise = new Promise((resolve, reject) => {
  console.log(1);  // 1.执行同步代码，打印出1
  setTimeout(() => { // 2.下一轮宏任务才来执行，我们继续走我们本轮的宏任务
    console.log("timerStart"); // 7.第二次宏任务开始执行，打印出 timeStart
    resolve("success"); // 8.将promise状态改为resolved，且将结果保存
    console.log("timerEnd"); // 9.继续执行同步代码，打印出timeEnd
  }, 0);
  console.log(2); // 3.本轮的宏任务，直接打印出 2
});
promise.then((res) => { //4. .then是个微任务，等本轮宏任务执行完毕，再来检查，进入5
 //6.来检查了，但是promise还是pending状态，不执行，直接开始下一轮宏任务
  console.log(res);     宏任务    // 10 打印出success
});
console.log(4); // 5.本轮宏任务，打印出4，然后去检查本轮微任务
// 1
// 2
// 4
// timerStart
// timerEnd
// success
```

### 题目九 ✨✨
```javascript
// 1.本轮宏任务1开始
setTimeout(() => {  // 2.遇到setTimeout，放入到宏任务2去
  console.log('timer1'); // 4.宏任务2开始，打印出timer1
  setTimeout(() => {		// 5.遇到setTimeout，放入到宏任务3后去，定义为4且宏任务2结束
    console.log('timer3') // 6. 宏任务4开始，打印出timer3
  }, 0)
}, 0)
setTimeout(() => {	//3. 遇到setTimeout，放入到宏任务2之后去，定义为宏任务3
  console.log('timer2') // 6.宏任务3开始，打印出 timer2，宏任务3结束
}, 0)
console.log('start') // 4.宏任务1执行完毕，打印出start
// start
// timer1
// timer2
// timer3
```
加个Promise 
```javascript
setTimeout(() => {  // 1.加入到宏任务2中
  console.log('timer1'); // 4.宏任务2开始，打印出 timer1
  Promise.resolve().then(() => { //5.遇到.then，那么放入到宏任务2的微任务队列
    console.log('promise')   // 6.没有其它宏任务，直接打印 promise
  })
}, 0)
setTimeout(() => { // 2.加入到宏任务2之后，定义为宏任务3
  console.log('timer2') //7.开始执行宏任务3，打印出 timer2
}, 0)
console.log('start') // 3.宏任务1执行完毕，打印出start
// start
// timer1
// promise
// time2
```
> 之所以先打印出promise是因为，第一个setTimeout为下一个宏任务队列，第二个setImeout为下下一个宏任务队列，因此在第二个宏任务队列执行完毕之后，会先去本轮的微任务队列中去查找是否有微任务。


### 题目十  ✨
```javascript
Promise.resolve().then(() => { // 1.遇到.then，放入到本轮微任务列表中去
  console.log('promise1');    // 4.查到本轮微任务列表，打印 promise1
  const timer2 = setTimeout(() => { //5.遇到setTimeout，放入到第三轮宏任务列表中去
    console.log('timer2')  //7.第三轮宏任务开始，打印出timer2
  }, 0)
  });
const timer1 = setTimeout(() => { //2.遇到setTimeout，放入到第二轮宏任务列表中去
  console.log('timer1')	//5.打印出timer1
  Promise.resolve().then(() => { //6.宏任务2执行完毕后，发现微任务列表，打印出promise2
    console.log('promise2')
  })
}, 0)
console.log('start'); //3.打印出 start,查找本轮微任务列表
// start
// promise1
// timer1
// promise2
// timer2
```

### 题目十一 ✨✨✨

```javascript
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => { // 1.放到下一轮宏任务中去
    resolve('success') //7.将promise1的状态修改为resoled，且保存值，查找还有没有微任务，查找到了.then
  }, 1000)
})
const promise2 = promise1.then(() => { //2.放到本轮微任务列表中去
  // 6.此时，promise1还没返回状态，不执行
  throw new Error('error!!!')  // 8.throw相当于给了promise2状态为rejected,且打印出异常，开始执行下一轮宏任务
})
console.log('promise1', promise1) // 3.此时打印的promise1为pending状态
console.log('promise2', promise2) // 4.此时打印的promise2为pending状态
setTimeout(() => {     // 5.放入到下一轮宏任务中去,找到到.then，去执行本轮微任务
  console.log('promise1', promise1) // 9.此时打印出promise1的状态为resolved，且值为success
  console.log('promise2', promise2) // 10. promise2此时的状态为rejected
}, 2000)
// promise1 Promise{pending}
// promise2 Promise{pending}
// 102 Uncaught (in promise) Error: error!!! 
// promise1 Promise{resoled:'success'}
// promise2 Promise{rejected:'error!!!'}
```

### 题目十二 ✨✨
```javascript
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {  // 1.下一轮宏任务
    resolve("success"); //7.promise微resoled，且拿到了这个值进行保存
    console.log("timer1");	//8.打印4:打印出timer1
  }, 1000);
  console.log("promise1里的内容"); // 2.本轮同步代码，打印1 
});
const promise2 = promise1.then(() => { //3.本轮微任务队列
  throw new Error("error!!!"); // 9.打印出5异常，且promise2为rejected状态，保存值
});
console.log("promise1", promise1); //4.打印2:promise1此时为pending状态
console.log("promise2", promise2); //5.打印3：此时promise2位pending状态
setTimeout(() => { // 6.下下一轮宏任务
  console.log("timer2"); // 10.打印出timer2
  console.log("promise1", promise1); // 11 打印6出success
  console.log("promise2", promise2); // 12 打印7出error！！！！
}, 2000);
// promise1里的内容
// promise1 Promise{pending}
// promise2 Promise{pending}
// timer1
// throw error
// timer2
// promise1 Promise{resolved}
// promise2 Promise{rejected}
```

## 三、Promise中的then、catch、finally

---

### 题目十三 ✨
```javascript
const promise = new Promise((resolve, reject) => {
  resolve("success1");
  reject("error");
  resolve("success2");
});
promise
  .then(res => {
  console.log("then: ", res);
}).catch(err => {
  console.log("catch: ", err);
})
// then success1
```
> Promise中的状态一经改变，就不能再次进行改变了。


### 题目十四 ✨✨
```javascript
const promise = new Promise((resolve, reject) => {
  reject("error");
  resolve("success2");
});
promise
  .then(res => {
  console.log("then1: ", res);
}).then(res => {
  console.log("then2: ", res);
}).catch(err => {
  console.log("catch: ", err);
}).then(res => {
  console.log("then3: ", res);
})
// catch error
// then3 undefined
```
> 之所以最后是undefined，是因为catch()也会返回一个Promise，且由于这个Promise没有返回值


### 题目十五 ✨
```javascript
Promise.resolve(1)
  .then(res => {
  console.log(res);
  return 2;
})
  .catch(err => {
  return 3;
})
  .then(res => {
  console.log(res);
});
// 1 2
```
> return 2 会被包装成 Promise(2)


### 题目十六 ✨
```javascript
Promise.reject(1)
  .then(res => {
  console.log(res);
  return 2;
})
  .catch(err => {
  console.log(err);
  return 3
})
  .then(res => {
  console.log(res);
});
// 1 3
```

### 题目十七 ✨ ✨
```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('timer')
    resolve('success')
  }, 1000)
})
const start = Date.now();
promise.then(res => {
  console.log(res, Date.now() - start)
})
promise.then(res => {
  console.log(res, Date.now() - start)
})
// timer
// success 1000
// success 10001
```
> 这里的.then之所以都会执行，是因为Promise中的.then或者.catch可以被调用多次。


### 题目十八  ✨ ✨
```javascript
Promise.resolve().then(() => {
  return new Error('error!!!')
}).then(res => {
  console.log("then: ", res)
}).catch(err => {
  console.log("catch: ", err)
})
// then error
```

> 返回任意一个非promise的值都会包裹成promise对象，这里的return new Error('error!!!')被包装成 return Promise.resolve(new Error('error')),如果需要抛出错误，可以使用Promise.reject()或者throw


### 题目十九  ✨ ✨
```javascript
const promise = Promise.resolve().then(() => {
  return promise;
})
promise.catch(console.err)
// 结果报错：Uncaught (in promise) TypeError: Chaining cycle detected for promise #<Promise>
```
> Promise的.then或者.catch中返回的值不能是promise本身，否则会造成死循环、报错


### 题目二十  ✨ ✨
```javascript
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log)
// 1
```
> .then或者.catch中的参数期望是**函数**，如果传入非函数那么就会发生**值穿透**，第一个then和第二个then中传入的都不是函数，因此发生了值的透传，将resolve(1)传入到了最后一个then中。


### 题目二十一  ✨
```javascript
Promise.reject('err!!!')
  .then((res) => {
  console.log('success', res)
}, (err) => {
  console.log('error', err)
}).catch(err => {
  console.log('catch', err)
})
// error,err!!!
```

> then函数中会有两个参数，第一个为传入resolve结果的，第二个为reject，可以理解为.catch为.then的简写.then(undeifined,err)


```javascript
Promise.resolve()
  .then(function success (res) {
  throw new Error('error!!!')
}, function fail1 (err) {
  console.log('fail1', err)
}).catch(function fail2 (err) {
  console.log('fail2', err)
})
// fail2 error!
```

### 题目二十一  ✨ ✨ ✨
```javascript
Promise.resolve('1')
  .then(res => {
  console.log(res)
})
  .finally(() => {
  console.log('finally')
})
Promise.resolve('2')
  .finally(() => {
  console.log('finally2')
  return '我是finally2返回的值'
})
  .then(res => {
  console.log('finally2后面的then函数', res)
})
// 1
// finally2
// finally
// finally2后面的then函数 2
```

> 为什么finally要在finally2中，看下面代码


```javascript
Promise.resolve('1')
  .finally(() => {
  console.log('finally1')
  throw new Error('我是finally中抛出的异常')
})
  .then(res => {
  console.log('finally后面的then函数', res)
})
  .catch(err => {
  console.log('捕获错误', err)
})
// finally1
// 捕获错误 我是finally中抛出的异常
```

## 四、Promise中的all和race

### 题目二十二-all  ✨
```javascript
function runAsync (x) {
  const p = new Promise(r => setTimeout(() => r(x, console.log(x)), 1000))
  return p
}
Promise.all([runAsync(1), runAsync(2), runAsync(3)])
  .then(res => console.log(res))
// 1
// 2
// 3
// [1,2,3]
```
> 有了all，可以并行执行多个异步操作，并且在一个回调中处理所有的返回数据，.all后面的then接收的是所有异步操作的结果。


### 题目二十三-all  ✨ ✨
```javascript
function runAsync (x) {
  const p = new Promise(r => setTimeout(() => r(x, console.log(x)), 1000))
  return p
}
function runReject (x) {
  const p = new Promise((res, rej) => setTimeout(() => rej(`Error: ${x}`, console.log(x)), 1000 * x))
  return p
}
Promise.all([runAsync(1), runReject(4), runAsync(3), runReject(2)])
  .then(res => console.log(res))
  .catch(err => console.log(err))
// 1
// 3
// 2
// Error 2
// 4
```

### 题目二十四-race ✨
```javascript
function runAsync (x) {
  const p = new Promise(r => setTimeout(() => r(x, console.log(x)), 1000))
  return p
}
Promise.race([runAsync(1), runAsync(2), runAsync(3)])
  .then(res => console.log('result: ', res))
  .catch(err => console.log(err))
// 1
// result 1
// 2
// 3
```

> 虽然race是赛跑且只会获取最先执行的那个结果，但是 2和3还是会去执行的。
应用场景是：为异步请求设置超时时间。


### 题目二十五-race✨
```javascript
function runAsync(x) {
  const p = new Promise(r =>
      setTimeout(() => r(x, console.log(x)), 1000)
     );
  return p;
}
function runReject(x) {
  const p = new Promise((res, rej) =>
    setTimeout(() => rej(`Error: ${x}`, console.log(x)), 1000 * x)
   );
  return p;
}
Promise.race([runReject(0), runAsync(1), runAsync(2), runAsync(3)])
  .then(res => console.log("result: ", res))
  .catch(err => console.log(err));
// 0
// result :Error:0
// 1
// 2
// 3
```
> all与race小结
> - Promise.all的作用是接收一组异步任务，然后并行执行异步任务，在所有异步任务执行完毕后才执行回调。
> - Promise.race也是接收一组异步任务，然后并行执行异步任务，只保留第一个先完成的异步执行结果，其它的结果会抛弃不用。
> - Promise.all.then中的结果与传入到all中的顺序是一致的。
> - all和race传入的数组如果有抛出异常的错误，那么只会抛出第一个错误。


## 五、async和await

---

> 在很多时候，async和await的解法和Promise差不多，但是有些又不一样。

### 题目二十六 ✨✨
```javascript
async function async1() {
  console.log("async1 start");
  await async2();   //会阻塞后面代码的执行
  console.log("async1 end");
}
async function async2() {
  console.log("async2");
}
async1();
console.log('start')
// async1 start
// async2
// start
// async1 end
```
> await后面的语句相当于放到了Prmise中，下一行以及之后的语句相当于放到了.then中。

```javascript
async function async1() {
  console.log("async1 start");
  new Promise(resolve => {
    console.log("async2")
    resolve()
  }).then(res => console.log("async1 end"))
}
async function async2() {
  console.log("async2");
}
async1();
console.log("start")
// async1 start
// async2
// start
// async1 end
```

```javascript
async function async1() {
  console.log("async1 start");
  new Promise(resolve => {
    console.log('promise')
  })
  console.log("async1 end");
}
async1();
console.log("start")
// async1 start
// promise
// async1 end
// start
```

### 题目二十七 ✨✨

```javascript
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}
async function async2() {
  setTimeout(() => {
    console.log('timer')
  }, 0)
  console.log("async2");
}
async1();
console.log("start")
// async1 start
// async2
// async1 end
// start
// timer
```

### 题目二十八 ✨✨✨

```javascript
async function async1() {
  console.log("async1 start");  // 2.打印1 async1 start
  await async2();								// 3.相当于new Promise，去里面执行同步代码
  console.log("async1 end");    // 8.打印4 async1 end
  setTimeout(() => {  					//9. 放入到下下下一轮宏任务
    console.log('timer1')       // 11 打印7  timer1
  }, 0)
}
async function async2() {
  setTimeout(() => {         //4. 放到下一步宏任务中去
    console.log('timer2')    //9. 打印5  timer2
  }, 0)
  console.log("async2");     //5. 打印2  async2
}
async1();     //1.开始执行async1函数
setTimeout(() => {    	    // 6. 放入到下下一轮宏任务中去
  console.log('timer3')			//10. 打印6 timer3
}, 0)
console.log("start")      //7.打印3  start，然后检查本轮微任务
// async1 start
// async2
// start
// async1 end
// timer2
// timer3
// timer1
```

### 题目二十九 ✨
```javascript
async function fn () {
  // return await 1234
  // 等同于
  return 123
}
fn().then(res => console.log(res))
// 123
```
> 正常情况下，await后面是一个Promise对象，会返回改对象的结果

### 
### 题目三十 ✨✨✨
```javascript
async function async1 () {
  console.log('async1 start');  // 2
  await new Promise(resolve => {
    console.log('promise1')   // 3
  })
  console.log('async1 success'); 
  return 'async1 end'
}
console.log('srcipt start')   // 1
async1().then(res => console.log(res)) 
console.log('srcipt end')   // 4
// srcipt start
// async1 start
// promise1
// srcipt end
```
> 在async1中await后面的Promise是没有返回值的，也就是它的状态始终是pending状态，因此相当于一直在await，await，await却始终没有响应...
> 所以在await之后的内容是不会执行的，也包括async1后面的 .then。

### 
### 题目三十一 ✨✨
```javascript
async function async1 () {
  console.log('async1 start'); // 2
  await new Promise(resolve => {
    console.log('promise1') // 3
    resolve('promise1 resolve')
  }).then(res => console.log(res)) //5
  console.log('async1 success'); //6
  return 'async1 end'
}
console.log('srcipt start') // 1
async1().then(res => console.log(res)) //7
console.log('srcipt end') // 4
// srcipt start
// async1 start
// promise1
// srcipt end
// promise1 resolve
// async1 success
// async1 end
```

### 题目三十二 ✨✨
```javascript
async function async1 () {
  console.log('async1 start');  // 2
  await new Promise(resolve => {
    console.log('promise1') // 3
    resolve('promise resolve')
  })
  console.log('async1 success'); // 5
  return 'async1 end'
}
console.log('srcipt start')  // 1
async1().then(res => {
  console.log(res) // 6
})
new Promise(resolve => {
  console.log('promise2') // 4
  setTimeout(() => {
    console.log('timer') // 7
  })
})
// srcipt start
// async1 start
// promise1
// promise2
// async1 success
// async1 end
// timer
```

### 题目三十四  ✨✨
```javascript
async function async1() {
  console.log("async1 start"); // 2
  await async2();
  console.log("async1 end"); // 6
}

async function async2() {
  console.log("async2"); // 3
}

console.log("script start"); // 1

setTimeout(function() {  
  console.log("setTimeout");//8
}, 0);

async1();

new Promise(function(resolve) {
  console.log("promise1"); // 4
  resolve();
}).then(function() {
  console.log("promise2"); //7
});
console.log('script end') // 5
// script start
// async1 start
// async2
// promise1
// script end
// async1 end
// promise2
// setTimeout
```

### 题目三十五  ✨✨
```javascript
async function testSometing() {
  console.log("执行testSometing"); // 2
  return "testSometing";
}
async function testAsync() {
  console.log("执行testAsync"); // 6
  return Promise.resolve("hello async");
}
async function test() {
  console.log("test start..."); // 1
  const v1 = await testSometing();
  console.log(v1); //5
  const v2 = await testAsync();
  console.log(v2); // 8
  console.log(v1, v2);//9
}
test();
var promise = new Promise(resolve => {
  console.log("promise start..."); // 3
  resolve("promise");
});
promise.then(val => console.log(val)); // 7
console.log("test end..."); // 4
// test start...
// 执行testSometing
// promise start...
// test end...
// testSometing
// 执行testAsync
// promise
// hello async
// testSometing hello async
```

## 六、async处理错误

---

### 题目三十六  ✨
```javascript
async function async1 () {
  await async2();
  console.log('async1');
  return 'async1 success'
}
async function async2 () {
  return new Promise((resolve, reject) => {
    console.log('async2') // 1
    reject('error')
  })
}
async1().then(res => console.log(res))
// async2
// Uncaught (in promise) error
```
> await后面跟着一个rejected的promise，那么就会抛出错误，不再往下执行


```javascript
async function async1 () {
  console.log('async1');
  throw new Error('error!!!')
  return 'async1 success'
}
async1().then(res => console.log(res))
```
> 同上


### 题目三十七 ✨✨
```javascript
async function async1 () {
  try {
    await Promise.reject('error!!!')
  } catch(e) {
    console.log(e) // 2
  }
  console.log('async1'); // 3
  return Promise.resolve('async1 success')
}
async1().then(res => console.log(res)) //4
console.log('script start')//1
// script start
// error!!!
// async1
// async1 success
```

```javascript
async function async1 () {
  await Promise.reject('error!!!')
    .catch(e => console.log(e))
  console.log('async1');
  return Promise.resolve('async1 success')
}
async1().then(res => console.log(res))
console.log('script start')
// 同上
```

## 七、综合题

---

### 题目三十八 ✨✨
```javascript
const first = () => (new Promise((resolve, reject) => {
  console.log(3); //1
  let p = new Promise((resolve, reject) => {
    console.log(7); //2
    setTimeout(() => { 
      console.log(5); //6
      resolve(6);
      console.log(p) //7-1
    }, 0)
    resolve(1); 
  });
  resolve(2); 
  p.then((arg) => { 
    console.log(arg); // 4-1
  });
}));
first().then((arg) => {
  console.log(arg); //5-2
});
console.log(4); //3
// 3
// 7
// 4
// 1
// 2
// 5
// Promise{<resolved>: 1}
```

### 题目三十九 ✨✨
```javascript
const async1 = async () => {
  console.log('async1'); // 2-async1
  setTimeout(() => {
    console.log('timer1') //7-timer1
  }, 2000)
  await new Promise(resolve => {
    console.log('promise1') //3-promise1
  })
  console.log('async1 end')
  return 'async1 success'
} 

console.log('script start'); // 1-script start
async1().then(res => console.log(res));
console.log('script end'); //4-script end
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .catch(4)
  .then(res => console.log(res)) //5-1
setTimeout(() => {
  console.log('timer2') //6-timer2
}, 1000)
// script start
// async1
// promise1
// script end
// 1 
// timer2
// timer1
```
> 这里需要注意的一点是：async函数中await的new Promise要是没有返回值的话则不执行后面的内容。


### 题目四十 ✨✨✨
```javascript
const p1 = new Promise((resolve) => {
  setTimeout(() => {
    resolve('resolve3');
    console.log('timer1') //3-timer1
  }, 0)
  resolve('resolve1');
  resolve('resolve2');
}).then(res => {
  console.log(res) //1-resolve1
  setTimeout(() => {
    console.log(p1) //4-p1返回的是finally最终的值，所以也是undefined
  }, 1000)
}).finally(res => {  //finally中是接收不到Promise结果的，这个res是个迷惑项
  console.log('finally', res) // 2-finally-undefined
})
// resolve1
// finally,undefined
// timer1
// Promise {resolve:undefined}
```

## 八、大厂面试题

### 题目四十一--使用Promise每秒输入1 2 3

```javascript
const arr = [1, 2, 3]
arr.reduce((p, x) => {
  return p.then(() => {
    return new Promise(r => {
      setTimeout(() => r(console.log(x)), 1000)
    })
  })
}, Promise.resolve())
```

### 题目四十二--使用Promise实现红绿灯交替重复亮

```
// 红灯3秒亮一次，黄灯2秒亮一次，绿灯1秒亮一次
function red {
  console.log('red')
}
function yellow(){
  console.log('yellow')
}
function green(){
  console.log('green')
}
const light = function (timer, cb) {
  return new Promise(resolve => {
    setTimeout(() => {
      cb()
      resolve()
    }, timer)
  })
}
const step = function () {
  Promise.resolve().then(() => {
    return light(3000, red)
  }).then(() => {
    return light(2000, green)
  }).then(() => {
    return light(1000, yellow)
  }).then(() => {
    return step()
  })
}

step();
```

### 题目四十三：封装一个异步加载图片的方法

```
function loadImg(url){
  return new Promise((resolve,reject)=>{
    const img = new Image();
    img.onload = function(){
      console.log('一张图片加载完成')
      resolve(img)
    }
     img.onerror = function() {
    	reject(new Error('Could not load image at' + url));
    };
    img.src = url;
  })
}
```
