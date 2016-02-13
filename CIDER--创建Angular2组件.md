##前言
Angular2无论是在长相和语法中都与Angular1.x由很大的区别，2.x版本中废弃了1.x版本中很多的特性。说实话，Angular2就是一个全新的东西，哪怕你已经很了解Angular 1.x了。

在经历了一段时间的摸爬打滚之后，我总结出来了一套方法，可以让组件的创建变得更加简单。这套方法的关键字是 CIDER，接下来我通过一个示例为大家讲解一下如何简单的创建一个Angular2的组件。
 
##CIDER
1、创建一个类（Create）
2、输入依赖(Import)
3、装饰你的类(Decorate)
4、完善你的组件（Enhance）
5、在子组件中重复以上步骤（Repeat）

我借用一下Angular2官网教程的示例代码 Tour of Heroes Tutorial(https://angular.io/docs/ts/latest/tutorial/)来像大家展示一下组件的创建过程。

##创建一个类
制作组件的第一步，我们将要创建一个名为 AppComponent 的类。

```
class AppComponent {
  public title = 'Tour of Heroes';
}
```

##输入你的依赖
第二步我们要将所有的依赖输入到文件中。Component 是 AppComponent 这个组件中的唯一依赖。

```
import {Component} from 'angular2/angular2';

class AppComponent {
  public title = 'Tour of Heroes';
}
```

##装饰你的类
由于我们已经输入了最基本的Angular2依赖，所以是时候装饰一下我们类，让它起到一个Angular2组件该起的作用。我们打算告诉Angular2，将我们刚刚写好的 AppComponent 类作为一个组件使用，并将它与 `my-app` 元素绑定在一起。

```
import {Component} from 'angular2/angular2';
class Hero {
  id: number;
  name: string;
}
@Component({
  selector: 'my-app',
  templateUrl:'my-template.html'
})
class AppComponent {
  public title = 'Tour of Heroes';
  public hero: Hero = {
    id: 1,
    name: 'Windstorm'
  };
}
```

在本段代码中，我为了得到编译器代码格式的支持，使用了 `templateUrl` 属性。你也可以使用 `template` 属性，在该属性中定义模板的 HTML 结构。

```
<h1>{{title}}</h1>

<h2>{{hero.name}} details!</h2>

<div><label>id: </label>{{hero.id}}</div>

<div><label>name: </label>{{hero.name}}</div>
```

##完善你的组件
现在，我们已经奠定好一个组件的基础了，是时候来完善我们的组件了。我们打算使用我们刚刚定义好的模板文件，并在其中加入一个`input`元素，如下所示。

```
<h1>{{title}}</h1>

<h2>{{hero.name}} details!</h2>

<div><label>id: </label>{{hero.id}}</div>

<div>
  <label>name: </label>
  <div><input [(ng-model)]="hero.name" placeholder="name"></div>
</div>
```

为了对`input`元素进行双向数据绑定，我们需要输入一个`FORM_DIRECTIVES`指令来允许我们在模板中访问`ng-model`。接下来是很重要的一步，我们需要将`FORM_DIRECTIVES`放入修饰符中的`directives`属性中，该属性是一个数组类型。
 
```
import {Component, FORM_DIRECTIVES} from 'angular2/angular2';
class Hero {
  id: number;
  name: string;
}
@Component({
  selector: 'my-app',
  templateUrl:'my-template.html',
  directives: [FORM_DIRECTIVES]
})
class AppComponent {
  public title = 'Tour of Heroes';
  public hero: Hero = {
    id: 1,
    name: 'Windstorm'
  };
}
```

##在子组件中重复以上步骤
刚刚，我们出色的完善了我们的组件，为它添加了一个对表单输入框处理的功能，但是如果要作为一个产品，这显然还远远不够，我们还需继续为它开发。随着我们的组件变得越来越复杂，我们也可以将一个组件划分为成几个部分，并将每个部分抽象成一个子组件，这样又再一次开始了CIDER的过程。例如你可能有一个拥有复杂验证的表单，你想将表单元素从你的主组件中独立出来。你首先需要声明一个新的类来封装你的表单，并输入所有的依赖，于是新一轮的CIDER又开始了。

##补充：引导一个主组件启动
作为一个主组件，我们需要做一些额外的步骤来引导我们的应用程序以它为入口启动并持续运行。首先，我们需要输入一个名为 `bootstrap`的依赖，然后在类文件的底部调用 `bootstrap(AppComponent)`。你只需要在你的应用程序中调用一次就可以了，不需要为每个组件都调用 `bootstrap` 函数。
 
```
import {bootstrap, Component, FORM_DIRECTIVES} from 'angular2/angular2';
class Hero {
  id: number;
  name: string;
}
@Component({
  selector: 'my-app',
  templateUrl:'my-template.html',
  directives: [FORM_DIRECTIVES]
})
class AppComponent {
  public title = 'Tour of Heroes';
  public hero: Hero = {
    id: 1,
    name: 'Windstorm'
  };
}
bootstrap(AppComponent);
```

##尾声
OK，创建一个组件就是这么简单！希望以上创建组件的方法能对你在学习Angular2的道路上起到帮助作用，同时你也可以查看我其他有关Angular2的文章，希望对你有一些帮助。