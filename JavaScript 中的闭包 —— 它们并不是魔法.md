相信很多 JavaScript 的初学者在理解闭包的问题上，多多少少都会遇到一些困难。我想通过这篇文章，简单的说一下什么是闭包，帮助初学者解答一下心中的疑惑。

一旦你掌握了 JavaScript 的核心概念，那么闭包其实并不难理解。然而，通过阅读一些学术性的文章或者带有学术性方向的信息，你是很难理解闭包是什么的，你只会感到苦涩难懂。

本文希望通过一些 JavaScript 的具体函数示例，例如下面的例子，来让大家感受一下闭包。

```
function sayHello(name){
    var text = 'Hello ' + name;
    var sayAlert = function(){ alert(text); }
    sayAlert();
}
```

<input class="codeButton" value="sayHello('Bob');" onclick="sayHello('Bob');" type="button">
<script>
function sayHello(name){
    var text = 'Hello ' + name;
    var sayAlert = function(){ alert(text); }
    sayAlert();
}
</script>

##闭包的示例

两句话总结：
1、闭包本地的函数变量 —— 在函数返回之后仍然起作用；
2、闭包是一个堆栈帧，它在函数返回后并不会被释放。

下面的代码中返回了一个函数的引用：

```
function sayHello2(name){
    var text = 'Hello ' + name; // local variable
    var sayAlert = function(){ alert(text); }
    return sayAlert;
}
```

<input value="var say2 = sayHello2('Jane');
say2();" id="btn2" class="codeButton" onclick="var say2 = sayHello2('Jane');say2();window['say2x'] = say2;" type="button">
<script>
    // Ignore this: it just sets the text for the button: Done this way so button has two lines of text.
    document.getElementById('btn2').value = "var say2 = sayHello2('Jane');\nsay2();" 
</script>
<script>
    function sayHello2(name) {
        var text = 'Hello ' + name;
        var sayAlert = function() { alert(text); }
        return sayAlert;
    }
</script>

大多数的 JavaScript 的程序员能理解上面的代码中一个函数的引用是如何返回一个变量的。如果你不知道的话，那么你需要在学习闭包之前了解一下。一个C语言的程序员可能会认为这个函数返回了一个指向函数的指针，并且变量 `sayAlert` 和 `say2` 都是一个指向函数的指针。

在C语言的指向的指针和 JavaScript 的函数引用两者中，其实有很明显的不同。在 JavaScript 中，你能认为一个函数引用变量同时作为一个指向函数的指针和一个隐藏的指向闭包的指针。

上面的代码中存在闭包，因为匿名函数 `function() { alert(text); }` 是在另一个函数 `sayHello2()` 内部声明的。在 JavaScript 中，如果你在另一个函数内部使用 `function` 关键字，你就创造了一个闭包。

在C语言和其他语言中，在函数返回之后，所有的本地变量都不再可以访问，因为它们的堆栈帧已经被破坏了。

而在 JavaScript 中，如果你在另一个函数内部申明了一个函数，那么该本地函数变量在你调用函数并返回之后是仍然可以访问的。我们再来看一下上面的例子，在我们已经返回了 `sayHello2()` 之后，我们调用了 `function say2();`。需要注意的是，我们调用函数引用来显示变量文本的代码，是函数 `sayHello2()` 的本地变量。

```
function() { alert(text); }
```

<input class="codeButton" value="alert(say2.toString()); // show code" onclick="if(!window['say2x']) {alert('Click the above button 1st!')} else alert(say2x.toString()); // show code" type="button">

点击上面的按钮来获取打印出来的匿名函数的 JavaScript 代码。你可以看到指向变量文本的代码。该匿名函数能引用值为 `Jane` 的文本，因为 `sayHello2()` 的本地变量被保存在一个闭包当中。

JavaScript 的神奇之处在于，一个函数的引用同时拥有一个隐秘的对闭包的引用。
