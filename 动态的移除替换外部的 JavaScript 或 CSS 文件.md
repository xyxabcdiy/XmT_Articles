上一章中，我们简单介绍了如何动态加载外部 JavaScript 或 CSS 文件，接下来我们继续学习如何动态的移除/替换外部的 JavaScript 或 CSS 文件。

任何外部的 JavaScript 或 CSS 文件，无论是手动加载的还是动态加载的，都可以从页面中移除。然而移除之后的结果可能并不是你所期望的。

##动态移除一个外部的 JavaScript 或 CSS 文件

为了从页面中移除一个外部的 JavaScript 或 CSS 文件，关键的一步就是首先通过 DOM 操作的 `removeChild()` 方法来将它们从文档模型树中移除。一个通用的方法就是通过外部文件的文件名来移除它，当然还存在其它方法，例如通过 CSS 类名。考虑到这一点，接下来的函数将会基于文件名来移除任何 JavaScript 和 CSS 文件：

```
function removejscssfile(filename, filetype){
    var targetelement=(filetype=="js")? "script" : (filetype=="css")? "link" : "none" //确定元素类型
    var targetattr=(filetype=="js")? "src" : (filetype=="css")? "href" : "none" //确定元素中的相关属性
    var allsuspects=document.getElementsByTagName(targetelement)
    for (var i=allsuspects.length; i>=0; i--){ //从节点列表中向后搜索需要被移除的元素
    if (allsuspects[i] && allsuspects[i].getAttribute(targetattr)!=null && allsuspects[i].getAttribute(targetattr).indexOf(filename)!=-1)
        allsuspects[i].parentNode.removeChild(allsuspects[i]) //通过调用 parentNode.removeChild() 来移除元素
    }
}
 
removejscssfile("somescript.js", "js") //移除页面中所有出现的 “somescript.js” 的元素
removejscssfile("somestyle.css", "css") //移除页面中所有出现的 “somestyle.css” 的元素
```

该函数一开始就根据需要删除的文件类型，创建了一个页面中所有 "script" 或 "link" 元素的集合。检查相关属性时也会有相应的变化（`scr` 或 `href`）。之后，函数设置了一个循环，从后往前搜索集合中的元素，检查文件名是否匹配。对于这个从后往前的遍历顺序，我有一些要说明的地方：如果一个已经验证过的元素被删除了，那么整个集合每一次都会被这一个元素破坏，为了再继续正确的遍历新的集合，便用反向遍历耍了一个小手段（遍历时可能遇到 undefined 元素，因此首先要在 if 语句中检查 `allsuspects[i]`）。最后为了删除确定过的元素，就调用了 DOM 方法 `parentNode.removeChild()`。

那么，当你删除一个外部 JavaScript 或 CSS 文件时会发生什么事呢？可能完全不是你想象的那样。在 JavaScript 中，如果一个元素从文档结构树中删除时，所有已经加载过的外部 JavaScript 文件中的代码仍然存在于浏览器的内存中。也就是说，你仍然可以访问变量，函数等外部 JavaScript 文件中加载进来的东西。如果你想通过移除外部 JavaScript 文件清空浏览器内存，以上的方法是不可行的。对于外部的 CSS 文件，当你移除一个文件时，文件中所有的 CSS 规则也将被移除。

##动态替换一个外部的 JavaScript 或 CSS 文件

```
function createjscssfile(filename, filetype){
    if (filetype=="js"){ //如果文件名是外部 JavaScript 文件
        var fileref=document.createElement('script')
        fileref.setAttribute("type","text/javascript")
        fileref.setAttribute("src", filename)
    }
    else if (filetype=="css"){ //如果文件名是外部 CSS 文件
        var fileref=document.createElement("link")
        fileref.setAttribute("rel", "stylesheet")
        fileref.setAttribute("type", "text/css")
        fileref.setAttribute("href", filename)
    }
    return fileref
}
 
function replacejscssfile(oldfilename, newfilename, filetype){
    var targetelement=(filetype=="js")? "script" : (filetype=="css")? "link" : "none" //确定元素类型
    var targetattr=(filetype=="js")? "src" : (filetype=="css")? "href" : "none" //确定相关属性
    var allsuspects=document.getElementsByTagName(targetelement)
    for (var i=allsuspects.length; i>=0; i--){ //反向遍历节点列表中的元素并匹配要删除的元素
        if (allsuspects[i] && allsuspects[i].getAttribute(targetattr)!=null && allsuspects[i].getAttribute(targetattr).indexOf(oldfilename)!=-1){
            var newelement=createjscssfile(newfilename, filetype)
            allsuspects[i].parentNode.replaceChild(newelement, allsuspects[i])
        }
    }
}
 
replacejscssfile("oldscript.js", "newscript.js", "js") //用 "newscript.js" 文件替换所有存在 "oldscript.js" 文件的元素
replacejscssfile("oldstyle.css", "newstyle", "css") //用 "newscript.css" 文件替换所有存在 "oldscript.css" 文件的元素
```

注意一下函数 `createjscssfile()`，它其实就是我们上一章中所介绍的 `loadjscssfile()`，但是在返回值上做了修改，现在是返回新建的元素而不是将它增加到页面中。在 `replacejscssfile()` 中调用 `parentNode.replaceChild()` 来用新的元素替代旧的元素是非常简单和方便的。

##总结
在如今的大型 web 应用中，能够根据需求来异步加载相应的 JavaScript/CSS 文件，不仅仅是举手之劳，而且在某些情况下，已经是必备的技能了。希望你能从这项技术中找到你的乐趣。
