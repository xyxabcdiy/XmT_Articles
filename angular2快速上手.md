目前angular2已经来到了beta版，这意味着它已经做好了开发出稳定应用的准备了。和angular1.x相比，它发生了许多颠覆性的变化，angular2的官方网站上，有一个5分钟快速开始教程(angular.io)，有兴趣的朋友可以去开一下，帮助你快速的了解angular2。在本文中，我将会通过制作一个简单的小网站，来为大家展示angular2的新特性，带领大家走入angular2的全新世界。

##开发环境
现在有两种主要的方法配置angular2的应用程序，一个是通过systemjs(https://github.com/systemjs/systemjs)，而另一个则是webpack（https://webpack.github.io/）。为了简单起见，在本文中将会使用systemjs。
你可以使用ES5、EcmaScript 2015 或者 TypeScript来开发你的angular2应用，angular2的团队的推荐是使用TypeScript开发。在使用TypeScript之前，需要完成一些配置工作，虽然你会感觉到有些繁琐，但是用起来你还是会感到得心应手的，你的代码也将会更加清晰明了。
我们先借用在angular2官网的教程(https://angular.io/docs/ts/latest/tutorial/)中使用的工程Tour of Heroes（https://github.com/johnpapa/angular2-tour-of-heroes）来开始我们的征程。
你可以通过git下载这个工程，当你下载下来之后，第一件事应该是用`npm i`命令来初始化项目。
###tsconfig.js
因为我们是用TypeScript编写我们的程序，所以我们先得在我们的工程中配置TypeScript，告诉编译器如何产生JavaScript文件。关于配置文件中的其他属性在这里我就不赘述了，我只介绍两个最重要的属性，一个是`"target":"ES5"`和`"module":"system"`。`target`属性将设置TypeScript编译出来的ECMAScript版本为<br>ES5</br>，`"module":"system"`以为着我们想用<br>system</br>的格式来生成我们的模块。

```
{
  "compilerOptions": {
    "target": "ES5",
    "module": "system",
    "sourceMap": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "moduleResolution": "node",
    "removeComments": false,
    "noImplicitAny": true,
    "suppressImplicitAnyIndexErrors": true
  },
  "exclude": [
    "node_modules"
  ]
}
```

###package.json
我们已经定义了该如何编译TypeScript文件，接下来我们需要为工程添加一个package.json文件。

```

```

##引导启动
Angular2中对于我来说最神秘的一点就是“我该如何运行我的应用？！”，第二神秘的一点就是“好吧，我该如何启动我的应用程序？！”

启动我们的应用的第一件事就是在工程的`index.html`文件中引用必要的资源。除了angular的资源的之外，最重要的一环就是`system.src.js`，用它来作我们的模块加载器。

```
<script src="node_modules/angular2/bundles/angular2-polyfills.js"></script>
<script src="node_modules/systemjs/dist/system.src.js"></script>
<script src="node_modules/rxjs/bundles/Rx.js"></script>
<script src="node_modules/angular2/bundles/angular2.dev.js"></script>
<script src="node_modules/angular2/bundles/router.dev.js"></script>
```

我们打算通过使用`systemjs`来输入我们的启动模块。

```
<script>
    System.config({ packages: { app: { format: 'register', defaultExtension: 'js' } } });
    System.import('app/boot')
        .then(null, console.error.bind(console));
</script>
```

在`boot.ts`文件中，我们输入了3格组件：`bootstrap`、`ROUTER_PROVIDERS`和`AppComponent`。然后实例化了我们的应用并指明了我们的根组件，`AppComponent`并注入了`ROUTER_PROVIDERS`作为一个子模块。

```
import {bootstrap} from 'angular2/platform/browser';
import {ROUTER_PROVIDERS} from 'angular2/router';
import {AppComponent} from './app.component';

bootstrap(AppComponent, [
  ROUTER_PROVIDERS
]);
```

回到之前的`index.html`文件中，`<app>Loading...</app>`这一行就是我们应用的入口标记。Angular已经实例化了`AppComponent`并且将它的模板载入到了`app`元素中。

```
<body>
    <app>Loading...</app>
</body>
```

在讲解了如何编译、配置和启动一个angular2的应用程序之后。让我们看一看angular2中组件的组成部分，以便我们能更好的理解`AppComponent`和`app`之间的联系。

##组件(Component)
Angular中的组件是angular中最基本的概念之一。一个组件控制一个视图（View）——一片为用户展示信息和响应用户反馈的网页。一般来说，一个组件就是一个用于控制视图模板的JavaScript类。
引用（有关于组件的更多内容，你可以去阅读我的《CIDER:创建一个Angular2组件》，你将会了解到如何创建和使用你的组件）

###App 组件
创建组建的第一件事就是先写一个类（class）,我们先做声明，一会再来改进它。

```
export class AppComponent {}
```

接下来，我们应该输入我们的依赖模块。在本例中，我们仅需要从`angular2/core`中输入`Component`。

```
import {Component} from 'angular2/core';
```

之后，我们需要装饰(decorate)我们的类。通过添加`@Component`的元数据来告诉我们的应用程序，我们想要一个怎么样的`AppComponent`。我们将使用`selector`属性为我们的组件取一个HTML元素的名字，这样就可以通过定义好的HTML元素的名字来使用组件了。同时还需要通过`templateUrl`和`styleUrls`为组件设置模板和样式。最后，我们将`ROUTER_DIRECTIVES`这个指令通过`directives`属性注入到组件中去。

```
@Component({
  selector: 'app',
  templateUrl: 'app/app.component.html',
  styleUrls: ['app/app.component.css'],
  directives: [ROUTER_DIRECTIVES]
})
export class AppComponent {}
```

在这个示例中，我们还想增加为我们的组件增加路由功能，因此我们将会使用`RouterConfig`模块，并用`@RouterConfig`来装饰我们的组件。为了能够路由，我们需要在文件中的最顶部输入`RouterConfig`、`ROUTER_DIRECTIVES`、`AboutComponent`、`ExperimentsComponent`和`HomeComponent`。

```
import {RouteConfig, ROUTER_DIRECTIVES} from 'angular2/router';
import {AboutComponent} from './about/about.component';
import {ExperimentsComponent} from './experiments/experiments.component';
import {HomeComponent} from './home/home.component';
```

我们需要路由模块来开启路由功能以及其他的组件作为路由的目标。接下来就需要向`@RouterConfig`中传入一组路由的定义，来告诉应用程序路由的路径、路由的名字和每个路由所对应的组件。我们为`home`路由的属性`useAsDefault`设置为`true`，将其作为默认路由。

```
@RouteConfig([
  {path: '/home',        name: 'Home',        component: HomeComponent, useAsDefault: true },
  {path: '/about',       name: 'About',       component: AboutComponent },
  {path: '/experiments', name: 'Experiments', component: ExperimentsComponent }
])
```

然后，我们将`StateService`和`ExperimentService`输入到我们的组件中并放入`component`修饰符的`providers`属性中。以下是整个 AppComponent 的代码。

```
import {Component} from 'angular2/core';
import {RouteConfig, ROUTER_DIRECTIVES} from 'angular2/router';
import {AboutComponent} from './about/about.component';
import {ExperimentsComponent} from './experiments/experiments.component';
import {HomeComponent} from './home/home.component';
import {StateService} from './common/state.service';
import {ExperimentsService} from './common/experiments.service';

@Component({
  selector: 'app',
  templateUrl: 'app/app.component.html',
  styleUrls: ['app/app.component.css'],
  directives: [ROUTER_DIRECTIVES],
  providers: [StateService, ExperimentsService],
})
@RouteConfig([
  {path: '/home',        name: 'Home',        component: HomeComponent, useAsDefault: true },
  {path: '/about',       name: 'About',       component: AboutComponent },
  {path: '/experiments', name: 'Experiments', component: ExperimentsComponent }
])
export class AppComponent {}
```

###Home 组件
在完成 App 组件之后，我们继续编写我们的 Home 组件。首先还是先定义一个 `HomeComponent` 类，同时在类中为组件的标题和内容定义两个属性。

```
export class HomeComponent {
  title: string = 'Home Page';
  body:  string = 'This is the about home body';
}
```

然后输入合适的依赖模块。

```
import {Component} from 'angular2/core';
```

之后装饰我们的类并设置`selector`和`templateUrl`属性。

```
@Component({
    selector: 'home',
    templateUrl: 'app/home/home.component.html'
})
```

然后我们将为我们的组件引用 StateService 并使用它来存储状态两个路由之间的状态。Angular2中的依赖注入发生在该类的构造函数中，因此我们将在构造函数中注入 StateService。Angular的每个组件都有有生命周期的钩子（lifecycle hooks），我们可以按顺序使用它们。在本组件中，我们想在组件初始化的时候从 StateService 中获取和设置我们的信息。这时候我们需要使用`ngOnInit`钩子。（有关于组件的生命周期的详细内容可以参考官网教程（https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html））

```
import {Component} from 'angular2/core';
import {StateService} from '../common/state.service';

export class HomeComponent {
  title: string = 'Home Page';
  body:  string = 'This is the about home body';
  message: string;

constructor(private _StateService: StateService) { }

ngOnInit() {
    this.message = this._StateService.getMessage();
  }

updateMessage(m: string): void {
    this._StateService.setMessage(m);
  }
}
```

##服务(Service)
许多组件可能会需要访问相同的一组数据，而我们并不想像傻瓜一样的一遍又一遍的复制粘贴同样的代码。这个时候，服务（Service）出现了。我们需要创建一个单一的且可以重复使用的数据服务，并学会将该服务注入到我们需要的组件中去。

将数据的存取重新包装成一个独立的服务让组件的重心倾向于对视图的支持。它能让组件的单元测试变得更加容易。

### State 服务
在上面的组件中，我们经常见到一个叫StateService的东西，它到底是什么呢？没错，它就是一个服务，接下来，让我们一步一步的创建它。首先我们需要声明一个叫 StateService 的类并将它暴露给我们的应用程序。

```
export class StateService {
  private _message = 'Hello Message';

getMessage(): string {
    return this._message;
  };

setMessage(newMessage: string): void {
    this._message = newMessage;
  };
}
```

该服务中定义了`_message`属性以及它的属性获取器(getter)和设置器(setter)。我们要想使StateService能够注入到其他组件中去，首先需要在我们的类中输入`Injectable`，并用`@Injectable`修饰我们的类。

```
import {Injectable} from 'angular2/core';
@Injectable()
export class StateService {
  private _message = 'Hello Message';

getMessage(): string {
    return this._message;
  };

setMessage(newMessage: string): void {
    this._message = newMessage;
  };
}
```

好了，现在服务模块也写好了，感觉我们的应用终于快完成了，最后也是最重要的一步，就是要描述应用的样子了。

##视图(Views)
我将通过`home.component.html`作为切入点来简单介绍一下 Angular2 中的升级版模板语法（template syntax）。

单向数据绑定与 Angular1.x 中的形式是一样的，就是通过字符串插入。

```
<h1>{{title}}</h1>

{{body}}
```

用户输入事件不再通过在我们的HTML标签中使用自定义的Angular指令来捕获，而是通过在插入语句中封装原生的DOM事件（https://developer.mozilla.org/en-US/docs/Web/Events）来捕获它们。我们可以在下面代码中看到，我通过`(click)="updateMessage(message)"`来为元素注册 click 事件，当事件触发时调用 updateMessage 方法。

```
<button type="submit" class="btn" (click)="updateMessage(message)">Update Message</button>
```

Angular2中的双向数据绑定基于单向数据绑定。之前我们看到单向数据绑定中使用了字符串插入的方法，其实在我们绑定一个元素的属性时，可以使用方括号语法。我们将属性绑定（组件到视图）和事件绑定（视图到组件）的方法组合起来就是双向数据绑定了。是不是很神奇？双向数据绑定的方法很简单，就是在 ngModel 用方括号和圆括号包起来变成`[(ngModel)]=”message`。

```
<input type="text" [(ngModel)]="message" placeholder="Message">
```

以下是整个 home.component.html 的内容。

```
<h1>{{title}}</h1>

{{body}}

<hr>

<div>
    <h2 class="text-error">Home: {{message}}</h2>
    <form class="form-inline">
      <input type="text" [(ngModel)]="message" placeholder="Message">
      <button type="submit" class="btn" (click)="updateMessage(message)">Update Message</button>
    </form>
</div>
```

###路由标记
接下来，我们需要回到 app.component.html 中，来探讨一下如何在模板语法中加入路由链接。在我们定义一个路由的时候，每个组件都有它对应的模板，那么问题来了，当路由到一个组件的时候，这个模板应该被放在哪里呢？这个时候，`router-outlet`出现了，它告诉应用程序路由中组件的摆放位置。

```
<div id="container">
    <router-outlet></router-outlet>
</div>
```

在定义了路由组件的摆放位置之后，那么我们如何从一个路由到另一个路由呢？RouterLink 就是完成了路由的指向问题。

```
<h1 id="logo">
  <a [routerLink]="['/Home']"></a>
</h1>

<div id="menu">
  <a [routerLink]="['/Home']" class="btn">Home</a>
  <a [routerLink]="['/About']" class="btn">About</a>
  <a [routerLink]="['/Experiments']" class="btn">Experiments</a>
</div>
```

整个 app.component.html 文件如下所示：

```
<header id="header">
  <h1 id="logo">
    <a [routerLink]="['/Home']"></a>
  </h1>

  <div id="menu">
    <a [routerLink]="['/Home']" class="btn">Home</a>
    <a [routerLink]="['/About']" class="btn">About</a>
    <a [routerLink]="['/Experiments']" class="btn">Experiments</a>
  </div>

  <div class="color"></div>
  <div class="clear"></div>
</header>

<div class="shadow"></div>

<div id="container">
  <router-outlet></router-outlet>
</div>
```

##总结
到此为止，我们的网页算是完成了，在你拿出去炫耀之前，让我们回顾一下我们的制作过程：

1、我们配置了tsconfig.json文件来说明TypeScript该如何编译。
2、我们学习了如何简单的使用systemjs来处理模块的载入并引导我们应用程序的启动。
3、我们了解了该如何创建 AppComponent 和 HomeComponent。
4、我们学习了如何使用 @Injectable() 创建一个可注入的服务。
5、我们稍微了解了一下Angular2中的绑定语法。
6、我们学习了如何用 router-outlet 在视图中定义路由组件存放的位置，并使用 routerLink 进行路由之间的跳转。
