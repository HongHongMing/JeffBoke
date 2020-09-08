# ### 事件循环 evet-loop

### 今天看了一下 javascript 事件循环，大概梳理一下记录下来

先明白几个概念
JavaScript 是单线程，所以异步

- JavaScript 代码按照函数的调用栈来决定执行顺序外，还有任务队列
  （队列就是先进先出，符合单线程）
- 一个线程事件循环是唯一的，当时任务队列可以有很多
- 任务队列分成两种 macro-task(宏任务)另一个是 micro-task(微任务)
- macro-task: script(整体代码),setTimeout,setInterval,setImmediate,I/O,UI rendering
- micro-task:process.nextTick,Promise,Object.observe,MutationObserver
- setTimeout/Promise 等我们称之为任务源。而进入任务队列的是他们指定的具体执行任务。

```
//意思就是进入任务队列的就是的他们的回调函数
 new Promise(function(resolve){
        //这里字符串不进任务队列
        console.log("immediate_promise");
        resolve();
    }).then(function(){
        //这里是回调就进任务队列
        console.log('immediate_then');
    })

setTimeout(function() {
  //这里回调进去队列
  console.log('xxxx');
})

```

例子 1:

```
setTimeout(function(){
    console.log("timeout1");
})

new Promise(function(resolve){
    console.log("promise1");
    for(var i =0; i< 100; i++) {
        i == 99 && resolve();
    }
    console.log("promise2");
}).then(function(){
    console.log("then1");
    new Promise(function(resolve){
        console.log("promise3");
        for(var i =0; i< 100; i++) {
            i == 99 && resolve();
        }
        console.log("promise4");
    })
    setTimeout(function(){
        console.log("timeout2");
    })
})
console.log("global1");

//运行结果
//promise1
//promise2
//global1
//then1
//promise3
//promise4
//timeout1
//timeout2
```

**首先，事件循环从宏任务队列开始，最开始的时候，宏任务队列中，只有一个 script(整体代码)任务。每一个任务的执行顺序，都依靠函数调用栈来搞定，就是自上到下，而当遇到任务源时，则会先分发任务到对应的队列中去，所以，上面例子的第一步执行如下图所示。**
![![image](http://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib3g6TGY1YsxUKkCPmA1grtXGvc26P4oIpmiaZY1MvzOE0Eic04ZamK7CQA7rsPOFIPcD14Sc9KA1fuQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://note.youdao.com/yws/public/resource/fc4ecb3e3f97270f2433e9abd6ced35c/xmlnote/2AD8E0828580487889C60E51BF6F2C31/4555)
首先 script 任务开始执行，全局上下文入栈

**第二步：script 任务执行时首先遇到了 setTimeout，setTimeout 为一个宏任务源，那么他的作用就是将任务分发到它对应的队列中。**

```
setTimeout(function(){
    console.log("timeout1");
})
```

![!image](https://note.youdao.com/yws/public/resource/fc4ecb3e3f97270f2433e9abd6ced35c/xmlnote/6097ACD6EC5D4C94AC80177A8A992115/4562)
宏任务 timeout1 进入 setTimeout 队列

**第三步：script 执行时遇到 Promise 实例。Promise 构造函数中的第一个参数，是在 new 的时候执行，因此不会进入任何其他的队列，而是直接在当前任务直接执行了，而后续的.then 则会被分发到 micro-task 的 Promise 队列中去。**

**因此，构造函数执行时，里面的参数进入函数调用栈执行。for 循环不会进入任何队列，因此代码会依次执行，所以这里的 promise1 和 promise2 会依次输出**

![image](https://note.youdao.com/yws/public/resource/fc4ecb3e3f97270f2433e9abd6ced35c/xmlnote/ADF97887F74A448EBDE3B0226B42F69B/4575)
resolve 在 for 循环中入栈执行

![image](https://note.youdao.com/yws/public/resource/fc4ecb3e3f97270f2433e9abd6ced35c/xmlnote/A6DD0810BBAE47958F6FF7B97E808C13/4581)
构造函数执行完毕的过程中，resolve 执行完毕出栈，promise2 输出，promise1 页出栈，then 执行时，Promise 任务 then1 进入对应队列

script 任务继续往下执行，最后只有一句输出了 globa1，然后，全局任务就执行完毕了。

**第四步：第一个宏任务 script 执行完毕之后，就开始执行所有的可执行的微任务。这个时候，微任务中，只有 Promise 队列中的一个任务 then1，因此直接执行就行了，执行结果输出 then1，当然，他的执行，也是进入函数调用栈中执行的。**

![image](https://note.youdao.com/yws/public/resource/fc4ecb3e3f97270f2433e9abd6ced35c/xmlnote/5E33DD250E784982BD0F76B40EDA86FF/4587)
执行所有的微任务

**第五步：当所有的 micro-tast 执行完毕之后，表示第一轮的循环就结束了。这个时候就得开始第二轮的循环。第二轮循环仍然从宏任务 macro-task 开始。**
![image](https://note.youdao.com/yws/public/resource/fc4ecb3e3f97270f2433e9abd6ced35c/xmlnote/CB36240F93EC47A7A7EA3B34B0231C51/4589)
微任务被清空

这个时候，我们发现宏任务中，只有在 setTimeout 队列中还要一个 timeout1 的任务等待执行。因此就直接执行即可。

![image](https://note.youdao.com/yws/public/resource/fc4ecb3e3f97270f2433e9abd6ced35c/xmlnote/F5C7EC1DB9DC4AB7A6C57792B8FD28A3/4591)

timeout1 入栈执行
这个时候宏任务队列与微任务队列中都没有任务了，所以代码就不会再输出其他东西了。

例子 2

```
console.log('golbal1');
setTimeout(function(){
    console.log('timeout1');
    process.nextTick(function(){
        console.log('timeout1_nextTick');
    })
    new Promise(function(resolve){
        console.log('timeout1_promise');
        resolve();
    }).then(function(){
        console.log('timeout1_then');
    })
})

setImmediate(function(){
    console.log('immediate1');
    process.nextTick(function(){
        console.log('immediate1_nextTick');
    })
    new Promise(function(resolve){
        console.log("immediate_promise");
        resolve();
    }).then(function(){
        console.log('immediate_then');
    })
})

process.nextTick(function(){
    console.log("global_nextTick");
})

new Promise(function(resolve){
    console.log("global1_promise");
    resolve();
}).then(function(){
    console.log('global1_then');
})

setTimeout(function(){
    console.log('timeout2');
    process.nextTick(function(){
        console.log('timeout2_nextTick');
    })
    new Promise(function(resolve){
        console.log('timeout2_promise');
        resolve();
    }).then(function(){
        console.log('timeout2_then');
    })
})


process.nextTick(function(){
    console.log('global2_nextTick');
})


new Promise(function(resolve){
    console.log("global2_promise");
    resolve();
}).then(function(){
    console.log('global2_then');
})

setImmediate(function(){
    console.log('immediate2');
    process.nextTick(function(){
        console.log('immediate2_nextTick');
    })
    new Promise(function(resolve){
        console.log("immediate2_promise");
        resolve();
    }).then(function(){
        console.log('immediate2_then');
    })
})
```

输出

```
golbal1
global1_promise
global2_promise
global_nextTick
global2_nextTick
global1_then
global2_then
timeout1
timeout1_promise
timeout2
timeout2_promise
timeout1_nextTick
timeout2_nextTick
timeout1_then
timeout2_then
immediate1
immediate_promise
immediate2
immediate2_promise
immediate1_nextTick
immediate2_nextTick
immediate_then
immediate2_then
```

这里有两个知识点第一个是执行过程遇到 setImmediate。setImmediate 也是一个宏任务分发器，将任务分发到对应的任务队列中。setImmediate 的任务队列会在 setTimeout 队列的后面执行，先执行 setTimeout 再执行 setImmediate

另一个是微任务中 process.nextTick 优先级比 Promise 的 then 回调的优先级高
