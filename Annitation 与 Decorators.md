2014年，Angular团队发布了它的 ECMAScript 语言的扩展版本 AtScript。为了使 AtScript 能够更方便程序员使用，拥有良好的开发体验，angular团队在里面增加了类型检测和注释语法(annotations)。半年多过去了，该团队宣布将 AtScript 转变成支持 annotations 和 decorators 两大特性的 TypeScript.

但是，annotations 和 decorators 究竟是什么呢？它们在 TypeScript 中又起着怎样的作用呢？接下来我们一起看一下 annotations 的转变过程以及它与 decorators 的不同之处。

##Annotation

我们先从 Annotations 开始。前文提到，Angular 团队开发出了 JavaScript 的语法糖 AtScript，引入了新的特性例如 Type Annotations（类型注释）, Field Annotations（字段注释） and MetaData Annotations（元数据注释）。我们现先从 metadata annotations 说起。下面是一个 Angular2 组件的例子，我们通过该例子看一下 metadata annotations 到底长什么样：

```
@Component({
  selector: 'tabs',
  template: `
    <ul>
      <li>Tab 1</li>
      <li>Tab 2</li>
    </ul>
  `
})
export class Tabs {

}
```

该例子中申明了一个 `Tabs` 的类。该类拥有一个 annotations —— `@Component`。如果我们移除所有的 annotations，仅仅留下一个空的类是不是就没有任何特殊意义了呢？因此，`@Component` 就是为 `Tabs` 类增加一些 metadata，从而使该类带有了一些特殊的意义。其实这就是 annotations 的作用，它们以一种声明的方式，为代码增加了 metadata。

`@Component` 作为一个 annotations，它告诉 Angular，紧挨着它的那个类 `Tabs` 是一个组件。annotations 就是这么简单的一个东西，虽然很简单，但是仍然有一些新的问题：

1、这些 annotations 从哪里来的？这些应该不是 JavaScript 中自带的东西吧？
2、谁定义了它的名字 `@Component`？
3、如果这是 AtScript 中的一部分，那么它被转化成 JavaScript 后是什么样子的？我们能在浏览器中使用它们么?

以上的问题我们一个一个的解决。首先，annotations 从哪里来的？回到这个问题之前，我们需要完成一段简单代码。`@Component` 需要我们从 Angular2 的文件中输入进来了：

```
import {Component} from "angular2/core";
```

这样第一个问题就解决了，所有的 annotations 都来自于 Angular2。我们再来看一下这个 annotations 在 Angular2 代码中的样子：

```
export class Component extends DirectiveMetadata {

  constructor() {
    ...
  }
}
```

我们可以看到 `Component` 实际上是 Angular 架构中的一个类，那么第二个问题也解决了。但是等一下，annotations 也是一个类，那么一个简单的类是如何改变另一个类的行为的呢？为什么我们只要在这些类之前加上 `@` 标记就能使用这些类作为 annotations 了呢？其实，我们并不可以。annotations 在目前的浏览器中是无效的，我们需要将它转化成其它的样子才能在浏览器中运行。

尽管我们有许多编译器可以选择，如 Babel, Traceur, TypeScript 等等，但是只有 Traceur 实现了 annotations。我们来看一下以上组件代码用 Traceur 编译之后的样子：

```
var Tabs = (function () {
  function Tabs() {}

  Tabs.annotations = [
    new Component({...})
  ];

  return Tabs;
})
```

最终，类被编译成了一个函数，也可以说是一个对象，所有的 annotations 最终都会变成一个实例并且放入类的 `annotations` 属性中。刚才我说所有 annotations，其实这种说法并不严谨。当 annotations 在类的参数中时，它就会被放入类的 `parameters` 属性中：

```
class MyClass {

  constructor(@Annotation() foo) {
    ...
  }
}
```

编译成 JavaScript 之后：

```
var MyClass = (function () {
  function MyClass() {}

  MyClass.parameters = [[new Annotation()]];

  return MyClass;
})
```

在这里 annotation 被放入了一个嵌套数组中，这是因为一个参数可能有不止一个 annotation。

好了，现在我们知道了这些 metadata annotations 是什么和它们转变之后的样子，但是我们仍然不知道在 Angular2 中， `@Component` 是如何将一个正常的类转化成一个组件的。实际上，这些工作都是 Angular 自己完成的。annotations 就是一段被添加进代码中的 metadata，这也就是为什么 `@Component` 是可以在 Angular2 中找到实现的源码的。Angular2 中还有很多其它的 annotation，但是也只有 Angular 才明白这些 annotation 的具体作用。

还有另一件非常有意思的事，Angular 希望将 metadata 放入类中的 `annotations` 和 `parameters` 属性中。如果 Traceur 不将 annotation 转化到这些特定的属性中，Angular2 就不会知道从哪获取 metadata。AtScript Annotations 就是一个非常特殊的实现，使 annotation 能够起到实际的作用。

如果作为程序猿的你可以选择在 metadata 在你代码中的位置，这种体验是不是更好一些呢？接下来我们来说一下 decorator 扮演着一个怎样的角色。

##Decorator
Decorator 是由 Yehuda Katz 为 ECMAScript 2016 提出的一个建议标准，目的就是为了要在设计应用程序的时候注释和修饰类和属性。这听起来是不是有些像 annotation 干的事？好像是有那么一点像...我们先来看一下 decorator 长什么样：

```
@decoratorExpression
class MyClass { }
```

可以看到，这完全和 AtScript 中的 annotations 机会一样！它们确实很像，但其实它们还是有明显的差别。从使用者的角度来看，decorator 的长相就是跟 "AtScript annotation" 差不多的。我们先来看一下上面一段代码转化成 JavaScript 之后的样子：

```
function decoratorExpression(target) {
   // Add a property on target
   target.annotated = true;
}
```

现在可以很清楚的看到，decorator 是一个函数，为 `target` 增加了一个 `annotated` 的属性，并将其设置为 `true`。它将一个需要被修饰的 `target` 的可访问权限给你了。是不是看出了一些端倪？现在不是编译器来决定你的 annotation 放在哪，而是我们自己来掌握在哪定义 decoration 或 annotation。其实，用一句话总结：有了 decorator，我们就能构建一个 annotation。

虽然关于 decorator 还有很多内容，但是这已经超出了本文的范围了。我建议有兴趣的读者可以看一下 Yehuda 的建议（https://github.com/wycats/javascript-decorators）来了解更多的有关于该特性的内容。

##总结
正如你所知道的那样，Angular 团队已经宣布他们放弃 “AtScript” 转而支持 “TypeScript”，因为这两种工具都是解决同一个问题。另外，TypeScript 已经在 1.5 版本中就已经支持 annotation 和 decorator 了。“AtScript Annotation” 和 decorator 几乎是一样的。从一个使用者的角度来说，它们拥有相同的语法。它们之中唯一的不同就在于我们不能控制如何将 annotation 作为 metadata 添加进我们的代码。然而 decorator 则是一个允许构建 annotation 接口。所以，我们只需要将注意力放在 decorator 上，因为它已经成为了一种标准。我希望本文能让那些在 annotation 和 decorator 之间迷茫的读者得到一些帮助。

