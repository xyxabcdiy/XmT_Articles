相信很多读者在制作自己网站的过程中，都会需要制作选项卡组件（tabs）。在Angular1中谈到指令控制器时，我们都会想到tabs组件。而angular2中没有指令控制器的概念，因为每个组件自己就是一个完全独立的模块。我们可以很简单的通过依赖注入来访问应用中其它的指令的组件。那么问题来了，你在angular1中制作选项卡组件的方法在angular2已经不能使用了，我们该如何在angular2中创建我们想要的选项卡组件呢？

没错，接下来就是见证奇迹的时刻! 让我们抛弃那些在指令和组件之间令你感到困惑不解的关系，去学习如何用angular2快速简单的制作一个选项卡组件。

##理想中的选项卡
在我们开始制作组件之前，让我们幻想一下我们理想中的组件应该会是什么样。在使用选项卡组件的时候，我们应该只要写下一串HTML代码就够了，代码里面包含了选项卡的标签，以及在每个标签中显示其对应的内容。如下段代码所示：

```
<tabs>
  <tab tabTitle="Tab 1">
    Here's some content.
  </tab>
  <tab tabTitle="Tab 2">
    And here's more in another tab.
  </tab>
</tabs>
```

我们有一个`<tab>`元素，来简单的表示单个有标题和内容的标签。而每一个`<tabs>`元素由包含了许多`<tab>`元素，将这些被包含的`<tab>`划分为一组选项卡。如果你已经理解了angular2的基本开发概念，你应该明白，组件中输入的值应该是由组件的使用者来管理和控制。而在angular2中，指令用来定义一个值是如何被传入到组件的范围中去，因此组件的使用者需要知道一个指令的内部工作方式。

如此一来，意味着当我们看到上面代码中`tabTitle`这个属性时，组件的使用者要么给组件的特性（attribute）赋值（如果属性存在的话），要么给组件的属性（ property）赋值。对于后者而言，可以允许组件的使用者给组件传递一个JS表达式，如下:

```
<tab tabTitle="This is just a String">
  ...
</tab>
<tab [tabTitle]="thisIsAnExpression">
  ...
</tab>
```

到此为止，我们大概明白了选项卡组件最终的使用方式，然后我们就可以开始动手编写代码了！

##创建一个组件
我们先从静态的HTML元素出发，创建一个`<tabs>`元素。如果你对angular2有一定的了解，应该知道我们需要一个`Component`修饰符来告诉angular我们组件的选择器和模板的样子。

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

当我们使用`Component`修饰符的时候，我们可以用`template`属性来说明模板的样子。大家可以发现我使用了反钩号（```）,这个语法是在ES2015中定义的，允许我们可以进行多行字符串的书写并且不需要使用如`+`这种恼人的连接符号。

正如你所看到的那样，组件模板已经由一个静态的选项卡列表组成。该列表将在稍后被一个动态指令所替换，目前我们就先这么用着。我们还需要一个地方来存放我们标签中的内容。我们在`ul`标签后放置一个`ng-content`元素。

```
@Component({
  selector: 'tabs',
  template: `
    <ul>
      <li>Tab 1</li>
      <li>Tab 2</li>
    </ul>
    <ng-content></ng-content>
  `
})
```

真棒，我们已经可以开始使用我们的`<tabs>`组件了！

```
<tabs>
  <p>Some random HTML with some random content</p>
</tabs>
```

当然，我们还想要在我们的`<tabs>`元素中使用`<tab>`元素，因此我们还需要创建一个`<tab>`组件。这个过程很简单，需要注意的是，我们应该为组件创建一个可以配置的`tabTitle`变量。

```
@Component({
  selector: 'tab',
  template: `
    <div>
      <ng-content></ng-content>
    </div>
  `
})
export class Tab {
  @Input() tabTitle;
}
```

上述代码中，我们为组件绑定了一个`tabTitle`的变量并将其输入给组件的`tabTitle`属性。最重要的一点是，我们的模板是一个被`div`所包裹的`ng-content`元素。

不是吧，选项卡做好？当然不是啦，还需要一点点的加工。但是我们先把以上两个小组件组装起来，看一下效果。

```
<tabs>
  <tab tabTitle="Foo">
    Content of tab Foo
  </tab>
  <tab tabTitle="Bar">
    Content of tab Bar
  </tab>
</tabs>
```

在浏览器中运行我们的程序，我们可以看到由`Tab 1`和`Tab 2`组成的列表出现了，但是应该出现在对应标签中的内容却同时出现了，这应该不是我们想要的选项卡组件吧？

##制作动态组件
到了这一步，我们首先先来完善一下我们`tabs`组件。我们需要动态的生成选项卡的标签列表而不是一个一个的手动填写，这样的话，就需要一个存放选项卡标签的数组。我们可以在组件的构造函数中初始化一个数组用来存放选项卡的所有标签。

```
export class Tabs {

  // typescript needs to know what properties will exist on class instances
  tabs: Tab[] = [];
}
```

看起来好像很不错的样子，但是我们如何让=将标签的标题放入到`tabs`这个数组中去呢？这个时候，在angular1中，指令控制器就该闪亮登场了。但是很遗憾，我们现在使用的是angular2，而且，在angular2中，这些操作将变得更加简单。首先，我们需要一个与外界连接接口，这样外界就可以为该组件中的内部属性传入参数。我们为组件增加一个`addTab(tab:Tab)`的方法，该方法中传入一个我们需要的`Tab`对象。

```
export class Tabs {
  ...
  addTab(tab:Tab) {
    this.tabs.push(tab);
  }
}
```

该方法仅仅是简单的将传入的`Tab`对象存入到我们的数组中去。接下来我们更新一下模板，让模板根据数组中的内容动态的生成一个标签列表。Angular2有一个叫做`ngFor`的指令可以对数组进行遍历，循环生成DOM，完美的解决了我们的需求。

```
@Component({
  ...
  template: `
    <ul>
      <li *ngFor="#tab of tabs">{{ tab.tabTitle }}</li>
    </ul>
  `
})
```

好了，现在我们有了一个`Tab`的数组，有了一个API可以扩充这个数组，在模板中还有一个基于该数组动态生成的列表。因为这个数组默认是一个空数组，所以默认情况下，产生的列表就是空的，选项卡标签的标题就不会显示出来，我们需要通过调用`addTab()`方法来添加选项卡标签的标题。

这个时候该`<tab>`组件登场了。在`<tab>`组件的年内不，我们可以通过强大的依赖注入系统来简单的调用父元素`Tabs`的依赖并获得它的实例。这样的话，我们就可以在子组件中使用父组件的API。

```
class Tab {
  constructor(tabs: Tabs) {
    tabs.addTab(this)
  }
}
```

看看上面这段代码，等一下，好像有什么不对劲的地方。`tabs:Tabs`不是 Typescript 中的类型注解么？在这里，angular2将其用在了依赖注入中。

根据 angular2 分等级的注射器（Hierarchical Injector）可以知道，我们首先通过从当前的组件向上遍历得到了一个`Tabs`的实例。在我们的应用中，当前的组件就是`<tab>`组件，它的注射器（Injector）需要一个`Tabs`的实例，如果当前组件中没有，注射器就会向父注射器申请`Tabs`。在这里，父注射器就是`<tabs>`组件并且它拥有`Tabs`实例，因此，将会返回正确的`Tabs`实例。

就目前而言，对于构造函数的参数中的类型注解让你可以访问父组件依赖这一点的理解非常重要。使用得到的实例，我们就可以轻松的调用`addTab()`方法了。

##使标签可切换
现在，我们已经可以正确的得到选项卡组件中的标签列表了，但是我们仍然可以同时看到每一个标签中的内容。而我们想达到的效果应该是点击对应标签，出现与之对应的内容。那么接下来我们该如何做呢？

首先，我们需要一个属性来激活或者注销一个标签，并通过该变量使标签的内容隐藏或者消失。我们需要扩展我们的`<tab>`组件。

```
@Component({
  template: `
    <div [hidden]="!active">
      <ng-content></ng-content>
    </div>
  `
})
class Tab { ... }
```

如果一个表情不处于激活状态，它应该被隐藏起来。我们没有在任何地方说明这个`active`属性，那么它的初始值就是`undefined`，在目前等于`false`。因此每一个标签在默认情况下都是未激活状态。为了保证至少有一个标签处于激活状态，我们可以稍微扩展一下`addTab()`方法。如下所示，默认激活第一个标签。

```
export class Tabs {
  ...
  addTab(tab:Tab) {
    if (this.tabs.length === 0) {
      tab.active = true;
    }
    this.tabs.push(tab);
  }
}
```

Perfect! 激活完成了，剩下的事就是让用户点击的标签激活。为了实现这个功能，我们还需要增加一个方法在用户点击标签时来设置`activate`属性。

```
export class Tabs {
  ...
  selectTab(tab:Tab) {
    this.tabs.forEach((tab) => {
      tab.active = false;
    });
    tab.active = true
  }
}
```

我们遍历了我们手头中所有的标签，并将它们注销，然后只激活被作为参数传进该方法的标签。然后，我们将该方法添加进我们的模板中，以后只要用户每一次点击，就会触发该方法。

```
@Component({
  ...
  directives: [NgFor],
  template: `
    <ul>
      <li *ngFor="#tab of tabs" (click)="selectTab(tab)">
        {{tab.tabTitle}}
      </li>
    </ul>
  `
})
```

我们看一下这里发生了什么，当`li`元素中的点击事件被触发时，就会调用`selectTab()`方法。

我们将最终代码组装一下。

```
@Component({
  selector: 'tabs',
  template: `
    <ul>
      <li *ngFor="#tab of tabs" (click)="selectTab(tab)">
        {{tab.tabTitle}}
      </li>
    </ul>
    <ng-content></ng-content>
  `,
})
export class Tabs {
  tabs: Tab[] = [];

  selectTab(tab: Tab) {
    this.tabs.forEach((tab) => {
      tab.active = false;
    });
    tab.active = true;
  }

  addTab(tab: Tab) {
    if (this.tabs.length === 0) {
      tab.active = true;
    }
    this.tabs.push(tab);
  }
}

@Component({
  selector: 'tab',
  template: `
    <div [hidden]="!active">
      <ng-content></ng-content>
    </div>
  `
})
export class Tab {

  @Input() tabTitle: string;

  constructor(tabs:Tabs) {
    tabs.addTab(this);
  }
}
```

到此为止，我们的一个比较粗糙的选项卡组件就完成了。这只是选项卡的一个最基本的实现方式，大家也可以以该组件为基础，制作一个更加理想的选项卡组件。



