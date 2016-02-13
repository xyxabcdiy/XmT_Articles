##大量的闭包示例

上一章，我们简单说明了一下闭包的概念，本章你将继续深入理解闭包。

我个人认为，当你通过阅读来了解闭包的时候，它们是非常苦涩且难以理解的，但是在你学习了一些闭包的示例之后，你就能通过点击几个按钮了解到闭包是如何工作的。我建议你在了解闭包如何工作之前，好好的消化一下本文提供的例子。如果你在没有完全掌握闭包之前就开始使用必爆了，你可能会遇到很多意想不到的 bug！

###示例一

本例说明了，本地变量不是复制出来的，它们是一种引用。这是一种类似于在外部函数存在的时候，将堆栈帧保存在内存中的方法。

```
function say667() {
    // Local variable that ends up within closure
    var num = 666;
    var sayAlert = function() { alert(num); }
    num++;
    return sayAlert;
}
```

<input value="var sayNumba = say667();
sayNumba();" id="btn3" class="codeButton" onclick="var sayNumba = say667();sayNumba();window['say3x'] = sayNumba;" type="button">
<script>
    // Ignore this: it just sets the text for the button: Done this way so button has two lines of text.
    document.getElementById('btn3').value = "var sayNumba = say667();\nsayNumba();" 
</script>

###示例二

以下示例中的三个全局函数对同一个闭包拥有一个公共的引用，这是因为它们都在调用 `setupSomeGlobals()`之后被申明。
 
```
function setupSomeGlobals() {
    // Local variable that ends up within closure
    var num = 666;
    // Store some references to functions as global variables
    gAlertNumber = function() { alert(num); }
    gIncreaseNumber = function() { num++; }
    gSetNumber = function(x) { num = x; }
}
```

<input class="codeButton" value="setupSomeGlobals();" onclick="setupSomeGlobals();document.getElementById('example4setup').style.display='';" type="button">

该例中的三个函数享有对统一闭包的访问权限，当三个函数被定义的时候，该闭包就是 setupSomeGlobals() 的本地变量。

有一点需要注意的是，如果你再一次点击 setupSomeGlobals()，会有创建一个新的闭包。老的 gAlertNumber, gIncreaseNumber, gSetNumber 变量会用带有新的闭包的新函数重写。（在 JavaScript 中，无论何时你在一个函数内部声明了一个函数，在外部函数的每一次调用时，内部的函数会被重新声明。）

###示例三

本例可以让许多初学者明白闭包，所以你需要用心去理解它。如果你在一个循环中定义了一个函数：来自闭包的本地变量会让你意想不到。

```
function buildList(list) {
    var result = [];
    for (var i = 0; i < list.length; i++) {
        var item = 'item' + list[i];
        result.push( function() {alert(item + ' ' + list[i])} );
    }
    return result;
}
 
function testList() {
    var fnlist = buildList([1,2,3]);
    // using j only to help prevent confusion - could use i
    for (var j = 0; j < fnlist.length; j++) {
        fnlist[j]();
    }
}
```

<input class="codeButton" onclick="testList();" value="testList();" type="button">
<script>
    function buildList(list) {
        var result = [];
        for (var i = 0; i < list.length; i++) {
          var item = 'item' + list[i];
          result.push( function() {alert(item + ' ' + list[i])} );
        }
        return result;
    }
    function testList() {
        var fnlist = buildList([1,2,3]);
        // using j only to help prevent confusion - could use i
        for (var j = 0; j < fnlist.length; j++) {
          fnlist[j]();
        }
    }
    </script>

`result.push( function() {alert(item + ' ' + list[i])}` 这一行循环了3次，每一次都将一个匿名函数的引用添加至 `result` 数组中。如果你不熟悉匿名函数，可能觉得过程是这样的：

```
pointer = function() {alert(item + ' ' + list[i])};
result.push(pointer);
```

注意，当你运行示例的时候，警告内容“item3 undefined”弹出了3次。这是因为，就像前一个例子一样，buildList 函数中只有一个本地变量的闭包。当匿名函数在这一行 `fnlist[j]();` 中被调用的时候，它们使用了同一个闭包，并且它们使用了当前闭包中 `i` 变量和 `item` 变量的值（变量 `i` 的值是3，是由于当循环完成的时候，变量 `i` 的值是3，且变量 `item` 的值是 `item3`）。

###示例四
本例说明了闭包中包含任何在外部函数退出之前，外部函数内声明的本地变量。有一点要注意的就是，变量 `alice` 实际上是在匿名函数后被声明的。首先是匿名函数被声明：当该匿名函数被调用的时候，它是可以访问变量 `alice` 的，因为变量 `alice` 在闭包中。那么 `sayAlice()();` 就是直接调用了从函数 `sayAlice()` 中返回的函数引用。

```
function sayAlice() {
    var sayAlert = function() { alert(alice); }
    // Local variable that ends up within closure
    var alice = 'Hello Alice';
    return sayAlert;
}
```

<input class="codeButton" value="sayAlice()();" onclick="sayAlice()();" type="button">
<input class="codeButton" value="alert(alice); // 注意: 由于 alice 不是一个全局变量，所以不会有任何反应" onclick="alert(alice);" type="button">
<script>
    function sayAlice() {
      var sayAlert = function() { alert(alice); }
      // Local variable that ends up within closure
      var alice = 'Hello Alice';
      return sayAlert;
    }
</script>
    
小提示：注意变量 `sayAlert` 也在闭包内部，并且可以被其他在函数 `sayAlice` 中声明的其他函数访问，或者可以被内部函数递归的访问。
    
###示例五
最后一个示例说明了每一次函数调用都会为本地变量创造一个单独的闭包。不是在每一次函数声明中有一个闭包，而是在每一次函数调用的时候才会产生一个闭包。

```
 function newClosure(someNum, someRef) {
    // Local variables that end up within closure
    var num = someNum;
    var anArray = [1,2,3];
    var ref = someRef;
    return function(x) {
        num += x;
        anArray.push(num);
        alert('num: ' + num + 
            '\nanArray ' + anArray.toString() + 
            '\nref.someVar ' + ref.someVar);
    }
}
```

<input value="closure1 = newClosure(40, {someVar : 'closure 1'});
closure2 = newClosure(1000, {someVar : 'closure 2'});" id="btn6" class="codeButton" onclick="closure1 = newClosure(40, {someVar : 'closure 1'});closure2 = newClosure(1000, {someVar : 'closure 2'});document.getElementById('example6setup').style.display='';" type="button">
<script>
    // Ignore this: it just sets the text for the button: Done this way so button has two lines of text.
    document.getElementById('btn6').value = "closure1 = newClosure(40, {someVar : 'closure 1'});\nclosure2 = newClosure(1000, {someVar : 'closure 2'});" 
</script>
<div id="example6setup" style="display: none;">
        <input class="codeButton" value="closure1(5);" onclick="closure1(5);" type="button">
        <br>
        <input class="codeButton" value="closure2(-10);" onclick="closure2(-10);" type="button">
</div>
<script>
    function newClosure(someNum, someRef) {
      // Local variables that end up within closure
      var num = someNum;
      var anArray = [1,2,3];
      var ref = someRef;
      return function(x) {
          num += x;
          anArray.push(num);
          alert('num: ' + num + 
               '\nanArray ' + anArray.toString() + 
               '\nref.someVar ' + ref.someVar);
        }
    }
</script>