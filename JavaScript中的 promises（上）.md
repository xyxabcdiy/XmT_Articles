JavaScript Promises 是 ECMAscript 6 中新增加特性，目的在提供一个更加清楚，更加直观的方法来处理异步任务的结束。一直到 JavaScript Promises 的出现为止，处理异步任务的结束都是交给事件处理器（例如：`image.onload`）和回调函数完成的。事件处理器在独立的元素中颇有成效，但是如果你想在一个组图片完全被加载之后收到通知，或者按每幅图加载顺序收到通知，这该如何做呢？你传递给方法中的最后一个参数作为回调函数（例如 jQuery 中的 `animate()`函数）在任务完成之后能很好运行自定义的代码，但是如果自定义代码中也调用了异步方法（如 `animate()`）并不断嵌套调用回调函数呢？你可能知道这叫做“回调地狱”。

JavaScript Promises 提供了一个机制，通过更多的鲁棒性和更少的混乱性来记录异步任务的状态。

##JavaScript Promises 支持

JavaScript Promises 是 ECMAscript 6 标准中的一部分，并且它最终应该在所有浏览器上都得到支持。目前，promise 已经在最近版本的 Chrome、FF、Safari 已经一些移动浏览器中得到支持，但是习惯性的，IE 并不支持这个特性。因为 IE 不支持 promise，你可以使用如 es6-promise.js 来使 IE 兼容 promise。

##语法
好的，接下来我们正式进入 promise 的学习。JavaScript Promises 中的核心部分就是 `Promise` 构造函数：

```
var mypromise = new Promise(function(resolve, reject){
 // 异步代码运行处
 // 调用 resolve() 来表示任务成功完成
 // 调用 reject() 来表示任务失败 
})
```

该构造函数中传递进了一个匿名函数，该匿名函数有两个参数，一个是 `resolve()` 方法，该方法在 promise 的状态被设置成 fulfilled 的时候被调用，另一个参数 `reject()` 在 promise 的状态被设置为 rejected 的时候被调用。一个 Promise 对象的起始状态是 pending，表示异步代码的监控既没有完成(fulfilled)也没有失败(rejected)。让我们先看一看 JavaScript Promises 在实际中是如何应用的，下面例子中，基于 image 元素中的 URL 动态加载图片：

```
function getImage(url){
    return new Promise(function(resolve, reject){
        var img = new Image()
        img.onload = function(){
            resolve(url)
        }
        img.onerror = function(){
            reject(url)
        }
        img.src = url
    })
}
```

`getImage()`函数返回一个 Promise 对象来记录图片加载的状态。当你调用：

```
getImage('doggy.gif')
```

它的 promise 对象最终将从最初的状态 pending 转化成 fulfilled 或 rejected，最终的状态取决于最后的图片是否加载成功。请注意我们已经将图片的 URL 传递给了 Promise 的两个方法 `resolve()` 和 `reject()`。这可以是其它任意你希望能在任务结束后被处理的数据。
 
就目前看来，Promises 设置对象的状态来表示任务的状态看起来好像是没有意义的。但是正如你看到的那样，Promise 能让我们更简单、更直接的定义在任务结束后该做的事情。

## then() 和 catch()
无论什么时候你实例化了一个 Promise 对象，then() 和 catch() 这两个方法都是有效的，它们用来决定异步任务结束后下一步的规划。看一下下面的例子：

```
getImage('doggy.jpg').then(function(successurl){
    document.getElementById('doggyplayground').innerHTML = '<img src="' + successurl + '" />'
})
```

在本段代码中，一旦 doggy.jpg 加载完成，我们会让图片在 `doggyplayground` 的 div 中显示。最开始的 `getImage()` 函数返回了一个 Promise 对象，因此我们可以调用 `then()` 来表示请求状态已经是 resolved。我们传递进 `resolve()` 函数中的图片的 URL 在 `then()` 函数中作为参数。

如果图片加载失败又会发生什么呢？`then()` 方法能够接受第二个函数来处理 rejected 状态的 Promise 对象：

```
getImage('doggy.jpg').then(
    function(successurl){
        document.getElementById('doggyplayground').innerHTML = '<img src="' + successurl + '" />'
    },
    function(errorurl){
        console.log('Error loading ' + errorurl)
    }
)
```

在上述结构中，如果加载图片成功，会执行 `then()` 中的第一个函数，如果加载失败的话，就会执行第二个函数。我们也可以使用 `catch()` 方法来处理错误：

```
getImage('doggy.jpg').then(function(successurl){
    document.getElementById('doggyplayground').innerHTML = '<img src="' + successurl + '" />'
}).catch(function(errorurl){
    console.log('Error loading ' + errorurl)
})
```

调用 `catch()` 等于调用 `then(undefined,function)`，因此上述代码等价于：

```
getImage('doggy.jpg').then(function(successurl){
    document.getElementById('doggyplayground').innerHTML = '<img src="' + successurl + '" />'
}).then(undefined, function(errorurl){
    console.log('Error loading ' + errorurl)
})
```

##使用递归按顺序载入和显示图片

如果我们有一组图片，我们想要按顺序载入和显示它们，也就是说，第一个载入和显示的是图片1，一旦完成了，就轮到图片2了，以此类推。一会我们将会谈到 chaining promises，但在此之前，我们先来看下另一种实现的方法。首先使用递归得到图片列表，然后每一次都调用 `getImage()` 函数在之后用 `then()` 方法在下一次调用 `getImage()` 之前来显示目前的图片，如此循环反复，直到所有图片都被处理过了：
 
```
var doggyplayground = document.getElementById('doggyplayground')
var doggies = ['dog1.png', 'dog2.png', 'dog3.png', 'dog4.png', 'dog5.png']
 
function displayimages(images){
    var targetimage = images.shift()
    if (targetimage){
        getImage(targetimage).then(function(url){
            var dog = document.createElement('img')
            dog.setAttribute('src', url)
            doggyplayground.appendChild(dog)
            displayimages(images)
        }).catch(function(url){
            console.log('Error loading ' + url)
            displayimages(images)
        })
    }
}
 
displayimages(doggies)
```

`displayimages()` 函数中，通过调用 `images.shift()` 按顺序的获取了一组图片。对于每一幅图片，我们首先调用 `getImage()` 来获取图片，然后在返回后 Promise 对象的 `then()` 方法中进一步说明下一步的动作，在本例中就是在调用 `displayimages()` 之前调用将图片放入 div `doggyplayground` 之中。在图片加载失败的时候，用 `catch()` 方法处理这种情况。当 `doggies` 数组为空时，递归停止。

带着 JavaScript Promises 使用递归，是一个按顺序处理一系列异步任务的方法。另一个更加通用的方法就是通过学习 chaining promises 的技巧。 

##Chaining Promises
我们已经知道 `then()` 方法可以被一个 Promise 的实例调用来说明任务完成之后该发生的事。然而，事实上我们可以将多个 `then()` 方法链接起来，将多个 promises 轮流链接到一起，来指定每一个 promise 转化为 resolved 状态之后该发生的事。使用 `getImage()` 函数举例说明如何在获取另一个图片之前获取一个图片：

```
getImage('dog1.png').then(function(url){
    console.log(url + ' fetched!')
    return getImage('dog2.png')
}).then(function(url){
    console.log(url + ' fetched!')
})

//Console log:
// dog1.png fetched
// dog2.png fetched!
```

那么这里发什么了什么呢？注意第一个 `then()` 方法中：

```
return getImage('dog2.png')
```

这行代码获取图片“dog2.png”并返回一个 Promise 对象。通过在 `then()` 中返回一个 Promise 对象，下一个 `then()` 就会等着那个 promise 在运行之前转化状态为 resolve，同时接受新的 Promise 对象传入的数据作为参数。通过在 `then()` 中返回另一个 promise 对象，就是多个 promises 链接到一起的关键一步。

注意我们可以在 `then()` 中简单的返回一个静态的值，该值将会被传入下一个 `then()` 方法并且作为它的参数。

在看了上述的示例之后，我们仍然想知道当一个图片没有成功加载时，发什么了什么：

```
getImage('baddog1.png').then(function(url){
    console.log(url + ' fetched!')
}).catch(function(url){
    console.log(url + ' failed to load!')
}).then(function(){
    return getImage('dog2.png')
}).then(function(url){
    console.log(url + ' fetched!')
}).catch(function(url){
    console.log(url + ' failed to load!')
})

//Console log:
// baddog1.png failed to load!
// dog2.png fetched!
```

`catch()` 和 `(undefined, functionref)` 是相同的意思，因此在 `catch()` 之后的的下一个 `then()` 仍然执行。注意，我们将下一个 promise 对象的返回放入了它的 `then()` 方法中，在前一个 promise 完成之后需要通过 `then()` 和 `catch()` 方法来处理。

如果你想要连续载入3幅图片，我们仅仅只需要为上述代码增加一系列的 `then()` `then()` `catch()`。

##创造一系列的 Promises

我们现在知道了 chaining promises 的基本的概念，但是手动地将所有 promises 链接到一起很快就将变得不可管理。对于一条很长的链，我们只需要一个从空 Promise 对象开始的方法和通过一堆用编程方式写出来的 `then()` 和　`catch()` 函数直到最后一个 promises。在 JavaScript Promises 中，我们能够创造出一个空白的 Promise 对象，并且转化状态为 resolved：

```
var resolvedPromise = Promise.resolve()
```

这里也可以用 `Promise.reject()` 来创建一个空白的且状态已经是 rejected 的 Promise 对象。那么为什么我们想要一个这样的 Promise 对象呢？对于一个已经与其他 promises 链接在一起的空白的 Promise 对象来说，因为状态已经是 resolved 的 Promise 对象将会自动跳转到第一个 `then()` 方法并开始启动事件链。

我们可以通过堆砌 `then()` 和 `catch()`，来使用状态是 resolved 的 Promise 地想来创造一系列的 promise。

```
var sequence = Promise.resolve()
var doggies = ['dog1.png', 'dog2.png', 'dog3.png', 'dog4.png', 'dog5.png']
 
doggies.forEach(function(targetimage){
    sequence = sequence.then(function(){
        return getImage(targetimage)
    }).then(function(url){
        console.log(url + ' fetched!')
    }).catch(function(err){
        console.log(err + ' failed to load!')
    })
})
 
//Console log:
// dog1.png fetched
// dog2.png fetched!
// dog3.png fetched!
// dog4.png fetched!
// dog5.png fetched!
```

首先，我们创建了一个状态为 resolved 的 Promise 对象并称其为 `sequence`，然后用 froEach() 遍历 `doggies[]` 数组中的元素，同时在 `then()` 和 `catch()` 中处理每一幅已经加载完成的图片。最终的结果是一连串的 `then()` 和 `catch()` 方法附加在 `sequence` 的后面，同时完成了载入每幅图片的理想的时间轴。

在另一种情况下，使用 `for` 循环来替代 `forEach()`：

```
var sequence = Promise.resolve()
var doggies = ['dog1.png', 'dog2.png', 'dog3.png', 'dog4.png', 'dog5.png']
 
for (var i=0; i<doggies.length; i++){
    (function(){
        var capturedindex = i
        sequence = sequence.then(function(){
            return getImage(doggies[capturedindex])
        }).then(function(url){
            console.log(url + ' fetched!')
        }).catch(function(err){
            console.log('Error loading ' + err) 
        })
    }())
}
 
//Console log:
// dog1.png fetched
// dog2.png fetched!
// dog3.png fetched!
// dog4.png fetched!
// dog5.png fetched!
```

在 `for` 循环的内部，为了在每一次循环正确的获得 `i` 的值并传递进 `then()` 中，我们需要创建一个外部的闭包来获取 `i` 的值。没有外部的闭包，每一次传入 `then()` 中的 `i` 的值仅仅只是最后一次循环 `i` 的值或者说 `doggies.length-1`。

##创建一组 promises
我们可以用一组 promises 来替代将多个 promises 链接到一起。这会使在所有异步任务已经完成之后的事情变得简单，而不是每一个任务完成之后的事。例如，下面的代码中使用 `getImage()` 来获取两个图片并将它们存放在　promises 的数组里。

```
var twodoggypromises = [getImage('dog1.png'), getImage('dog2.png')]
```

`getImage()` 调用的时候返回一个 promise，`twodoggypromises` 目前包含了两个 promises 对象。那么我们对这一组 promises 做什么呢？我们可以使用静态方法 `Promise.all()`：

```
Promise.all(twodoggypromises).then(function(urls){
    console.log(urls) // logs ['dog1.png', 'dog2.png']
}).catch(function(urls){
    console.log(urls)
})
```

`Promise.all()` 中传入了一个可以迭代的（数组或类数组对象） promise 对象，在进入随后的 `then()` 方法之前，需要等待所有 promises 完成。`then()` 方法中将会传入一个由所有 promise 返回值组成的数组参数。

那么如果中间某一个 promise 的状态变成了 reject 时，会发生什么情况呢？在该情况下，所有的 `then()` 部分将会被忽略，只会执行 `catch()` 方法。因此上面的情况中，如果一个或多个图片加载失败，只会调用 `catch()` 方法打印一组加载失败的图片的 url。

##在得到所有图片后显示图片

现在到了该显示图片的时候了。


