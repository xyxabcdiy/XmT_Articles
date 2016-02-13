JavaScript 中的闭包经常会把人弄得比较迷茫。我已经亲身体验过程序员之间在许多场合中因为这些概念而引起争论，一旦他们开始感到在使用闭包轻松自如的时候，往往都会陷入一个同样的陷阱：在 for 循环内部使用闭包。

最常见的情况：在用循环遍历 DOM 元素时，为每一个 DOM 元素都注册一个事件处理函数。在下面的例子中，我尝试让每个列表元素在被点击时，弹出警告框，显示它的序列号：

```
function attachEventsToListItems( ) {
    var oList = document.getElementById('myList');
    var aListItems = oList.getElementsByTagName('li');
    for(var i = 0; i < aListItems.length; i++) {
        var oListItem = aListItems[i];
        // Here I try to use the variable i in a closure:
        oListItem.onclick = function() {
            alert(i);
        }
    }
}
```

点击这些列表元素，查看结果：

<ul id="myList1">
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
    <li>Fourth item</li>
</ul>
<script type="text/javascript">
// <!--
function attachEventsToListItems1( ) {
    var oList = document.getElementById('myList1');
    var aListItems = oList.getElementsByTagName('li');
    for(var i = 0; i < aListItems.length; i++) {
        var oListItem = aListItems[i];
        oListItem.onclick = function() {
            alert(i);
        }
    }
}
attachEventsToListItems1();
// -->
</script>

奇怪的事情发生了。所有的这些列表元素在被点击弹出的警告框时，都只会显示 4。这个原因有一点复杂。我们将一个匿名函数作为事件处理器，它继承了 `attachEventsToListItems` 作用域中的变量 `i` 的值，而不是 for 循环中的变量 `i` 的值。然而，等到事件处理器执行的时候，for 循环已经完成了迭代，该函数中变量 `i` 的值已经变成了 4。这个问题的出现是因为，我们用作事件处理器的函数在被执行之前，并没有为变量 `i` 创造一个新的作用域。
  
解决这个问题的最好的办法是通过在 for 循环执行一个函数为当前的变量 `i` 创造一个新的作用域：

```
function attachEventsToListItems( ) {
    var oList = document.getElementById('myList');
    var aListItems = oList.getElementsByTagName('li');
    for(var i = 0; i < aListItems.length; i++) {
        var oListItem = aListItems[i];
        // Watch this:
        oListItem.onclick = (function(value) {
            return function() {
                alert(value);
            }
        })(i);
    }
}
```

<ul id="myList2">
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
    <li>Fourth item</li>
</ul>
<script type="text/javascript">
// <!--
function attachEventsToListItems2( ) {
    var oList = document.getElementById('myList2');
    var aListItems = oList.getElementsByTagName('li');
    for(var i = 0; i < aListItems.length; i++) {
        var oListItem = aListItems[i];
        oListItem.onclick = (function(value) {
            return function() {
                alert(value);
            }
        })(i);
    }
}
attachEventsToListItems2();
// -->
</script>

让我们来看一下上面这段代码中发生了什么：我并没有直接传入事件处理器，而是创建了一个中间函数 (1)，它有一个参数 (2)，并将该参数传递进了内部的函数。然后我马上调用了函数 (1)，并将变量 `i` 作为传入参数 (3)。

```
oListItem.onclick = (·function·1(·value·2) {
    return function() {
        alert(value);
    }
})(·i·)3;
```

通过调用该函数，为每一次的迭代创建了一个新的变量作用域，因为变量作用域是在函数执行时创建的。这样的话，我们就能为这个新的作用域传入变量 `i`。

现在，如果我们想要使用变量的值，我还需要完成一些事情。记住一点，我们在尝试传入事件处理器时，执行了中介函数并返回了一个可以作为事件处理器的函数。该函数继承了中介函数的变量作用域，并且能放心的使用在循环期间变量的值。

为了理解这个小技巧，我们需要掌握 JavaScript 的内部的很多工作原理，这些原理比我们在正常使用时用到的概念还要丰富。但是它却非常实用，希望你能喜欢。