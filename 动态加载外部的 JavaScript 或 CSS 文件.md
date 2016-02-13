传统的在页面中加载外部的 JavaScript 文件（如：.js）和 CSS 文件（如：.css）的方法是在页面的 head 标签中添加引用。

```
<head>
<script type="text/javascript" src="myscript.js"></script>
<link rel="stylesheet" type="text/css" href="main.css" />
</head>
```

用这种方法为页面引用文件会与页面的资源一起同步加载。在很多情况下，这种方法能很好的满足我们的需求，但是在异步 Ajax 的设计模式中，加载 JavaScript/ CSS 文件的方式也变得越来越灵活。在本教程中，我们来看一下如何动态的加载  JavaScript/ CSS 文件。

简而言之，为了在一个页面中加载一个 .js 或 .css 文件，就意味着要使用 DOM 方法，首先创建一个新的 “script” 或 “link” 元素，为它们设置合适的属性，最后，使用 `element.appendChild()` 来在文档结构树中合适的位置增添元素。这听起来似乎有些玄乎，我们来看一下具体的示例：

```
function loadjscssfile(filename, filetype){
    if (filetype=="js"){ //如果文件名是外部的 JavaScript 文件
        var fileref=document.createElement('script')
        fileref.setAttribute("type","text/javascript")
        fileref.setAttribute("src", filename)
    }
    else if (filetype=="css"){ //如果文件名是外部的 CSS 文件
        var fileref=document.createElement("link")
        fileref.setAttribute("rel", "stylesheet")
        fileref.setAttribute("type", "text/css")
        fileref.setAttribute("href", filename)
    }
    if (typeof fileref!="undefined")
        document.getElementsByTagName("head")[0].appendChild(fileref)
}
 
loadjscssfile("myscript.js", "js") //动态加载和添加该 .js 文件
loadjscssfile("javascript.php", "js") //将 “javascript.php” 文件作为 JavaScript 文件动态加载
loadjscssfile("mystyle.css", "css") ////动态加载和添加该 .css 文件
```

因为外部的 JavaScript 和 CSS 文件在技术上是可以以任何自定义的文件扩展名结尾的（例如：“`javascript.php`”），函数的参数 `filetype` 让你告诉函数你想要加载文件类型。在知道文件类型之后，函数会使用合适的 DOM 方法来创建 Html 元素，并为它们分配合适的属性，最终将它添加到 head 元素内。现在，我们需要解释一下，我们如何在 head 元素中放置文件的引用：

```
document.getElementsByTagName("head")[0].appendChild(fileref)
```

首先通过引用页面的 head 元素，然后调用 `appendChild()`，将新创建的元素增加到 head 标签的末尾处。此外，你应该意识到已经存在的元素是不会对新创建的元素产生任何影响的。也就是说，如果你两次调用 `loadjscssfile("myscript.js", "js")` ，现在在 head 元素的末端将会有两个新的 script 元素，都指向同一个 JavaScript 文件。这里仅仅从效率角度来看的话，你为你的页面添加一个多余的元素时，并浪费了进程中浏览器的内存。这里有一个简单方法来阻止为页面多次添加相同的文件，该方法就是记录每一次通过 `loadjascssfile()` 来增加文件，只增加新的文件：

```
var filesadded="" //已经加载的文件的列表
 
function checkloadjscssfile(filename, filetype){
    if (filesadded.indexOf("["+filename+"]")==-1){
        loadjscssfile(filename, filetype)
        filesadded+="["+filename+"]" //将已经加载的文件名存放入已加载文件列表，形式是 “[filename1],[filename2],etc”
    }
    else
        alert("file already added!")
}
 
checkloadjscssfile("myscript.js", "js") //成功加载
checkloadjscssfile("myscript.js", "js") //多余的文件，所以文件不会被加载
```

我刚刚粗略的做了检测，一个文件在被加载之后是否会存在于一个用于存放已存在文件列表的变量 `filesadded` 中。

在本章中，我仅仅介绍了一下如何动态的加载 JavaScript 和 CSS 文件。但是有些情况下，可能要求你移除或替换一个已经添加进页面的 .js 或 .css 文件，这该怎么办呢？在下一章我会继续我的教程，敬请期待！