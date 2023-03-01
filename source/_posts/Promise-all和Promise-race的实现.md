---
title: Promise.all和Promise.race的实现
date: 2021-09-19 22:58:49
tags: [JavaScript]
categories: 前端
---

#### 前言

Promise.all()和Promise.race()都是使用频率非常高的俩方法，手动实现一下，对于理解Promise还是有点帮助的

#### Promise.all()的实现

我们把all()的功能拆解成几个步骤：

1. 首先接收一个数组，数组里可能是Promise，也<u>可能是普通的数据</u>
2. 构造一个新的Promise实例 **P**
3. 执行数组里的Promise，接收结果并等待数组里每一个Promise完成，这里分两种情况：
   <u>所有</u>Promise的状态都变成Fulfilled（成功），或者有<u>任意一个</u>Promise的状态变成Reject（失败）
4. 改变 **P** 的状态，这取决于第三步的情况：
   成功则返回值是一个数组，里面<u>按顺序</u>放着每一个Promise的返回值；失败则返回值是<u>第一个</u>失败的Promise的返回值

```javascript
function pAll(promises) {
    //创建一个新的Promise
    return new Promise((resolve, reject) => {
        //新建一个数组来保存结果
        const length = promises.length;
        if(length === 0){
            resolve([])
        }
        const results = new Array(length);
        let count = 0;
        for (let i = 0; i < length; i++) {
            // 每个Promise执行，把结果放进数组里
            Promise.resolve(promises[i]).then(function (res) {
                results[i] = res
                // 每一个Promise成功，就计数
                // 当所有Promise都成功，就resolve输出结果
                count++
                if (count === length) {
                    resolve(results)
                }
            }, function (err) {
                // 有任何一个Promise失败，就reject输出错误
                reject(err)
            })
        }
    })
}
```

我们先来测试一下，看看效果如何：

```javascript
// 模拟有延迟的Promise
var p1 = new Promise((resolve, reject) => {
    setTimeout(()=>{
        resolve(1)
    },800)
})
// 普通值
var p2 = 2
// 直接完成的Promise
var p3 = Promise.resolve(3)
// 失败的Promise
var p4 = Promise.reject(4)


//分别测试成功和失败
pAll([p1,p2,p3]).then((res)=>{
    console.log(res)  
    // [1,2,3]
}).catch((err)=>{
    console.log(err)
})

pAll([p1,p2,p3,p4]).then((res)=>{
    console.log(res)
}).catch((err)=>{
    console.log(err)
    // 4
})
```

如你所见，彳亍！

其实这里有不少需要注意的地方，需要我们考虑周全：

1. 首先是接收的参数，既可以是Promise，也可以是普通的数据。Promise我们不必多说，直接使用就行了，而对于普通数据，则要使用`Promise.resolve()`做处理
   `Promise.resolve()`对于Promise会直接返回这个Promise，对于普通值，则会将它包装成一个Promise。此外对于thenable，还会将它展开。
   所以才会有这么一句 `Promise.resolve(promises[i]).then(...)` 保证无论数组里放的是什么，都能正常运行
2. 然后是执行的过程，这个for循环里循环变量是let，绝对不能是var，当然你非要用var就得换个写法了。原因不用多说了吧，用var会导致每一个 `results[i] = res` 里的 i 是同一个
3. 最后是保存的结果，必须是按照接收参数的顺序来存放结果。因为并不能确定Promise完成的先后顺序，所以我们不使用push或是其它的方法，而是使用下标来存放。这个比较好理解
4. 还有这么一行代码 ` if(length === 0){resolve([])}`，其实啊，如果给Promise.all()传一个空数组[]，那么返回的Promise会直接完成，这里是和原生的all函数保持返回内容一致罢了

---

#### Promise.race

同样地，我们把race()的功能拆解一下：

1. 接收一个数组，可能是Promise，也可<u>能是普通数据</u>
2. 构造一个新的Promise实例 **P**
3. 执行数组里的Promise，等待数组里<u>最先完成</u>的Promise
4. 改变 **P** 的状态，根据第三步中最先完成的Promise，返回这个Promise的成功值或失败值

```javascript
function pRace(promises){
    return new Promise(((resolve, reject) => {
        for(let i = 0;i < promises.length;i++){
            Promise.resolve(promises[i]).then((res)=>{
                resolve(res)
            },(err)=>{
                reject(err)
            })
        }
    }))
}
```

这个相比 all() 简单多了，不需要考虑多个Promise的结果，只取第一个完成的Promise

看看效果如何：

```javascript
var p1 = new Promise((resolve, reject) => {
    setTimeout(()=>{
        resolve(1)
    },800)
})
var p2 = Promise.resolve(2)
var p3 = Promise.reject(3)

// 分别测试成功和失败
pRace([p1,p2]).then(res=>{
    console.log(res)
    // 2
}).catch(err=>{
    console.log(err)
})

pRace([p1,p3]).then(res=>{
    console.log(res)
}).catch(err=>{
    console.log(err)
    // 3
})
```

实践证明也彳亍！

同理要注意的地方，一个是对于普通数据的转换，使用 `Promise.resolve()`，执行Promise时的循环变量用 let

事实上，如果给原生的race()传入一个空数组 []，那这个Promise永远都不会完成。这里也一样就保持一致不多处理了。

