Ionic2 使用说明（ v 0.2.0 ）

[TOC]

# 安装

1. 首先全局安装依赖包

	```
	npm install -g ionic cordova
	```

	如果安装速度慢，**可以使用代理或者淘宝镜像。淘宝镜像的使用方法：命令行中输入**

	```
	npm config set registry https://registry.npm.taobao.org
	```

2. 创建一个 Ionic2 的项目

	进入一个目录，在命令行执行：

	```
	ionic start <项目名称> <预设模板> --v2 --skip-npm
	```

	* 项目名称可以自由决定；
	* 预设模板会自动创建一个范例 app ，可用的有 `tabs` 、 `sidemenu` 、 `blank` 、 `super` 、 `tutorial` ，可以通过观察官方所给的范例进行学习；
	* `--v2` 参数代表需要使用 Ionic2 （ **ionic 3.x 之后， `--v2` 参数可以不用指定了**）；
	* `--skip-npm` 代表不要自动安装所需的扩展包（因为 npm 安装可能比较慢，可以先把这一步跳过，先让项目文件夹初始化完成）。

	命令执行成功后，会创建一个项目文件夹，由刚才输入的项目名称决定。进入此文件夹，使用

	```
	npm install
	```

	安装项目所依赖的扩展包。**其中有的扩展包使用了 node-sass ，这个包安装可能很慢，甚至于可能失败，需要手动为其指定淘宝镜像**。可以先在命令行中输入：

	```
	npm config set sass_binary_site https://npm.taobao.org/mirrors/node-sass/
	```

	然后再执行 `npm install` 。

3. 使用已建立好的项目

	高交所 APP 的项目文件已放到 git 的仓库上，可以直接 clone 下来：

	```
	git clone ssh://git@130.211.241.191/home/work/git/repo/venus/gjs-app.git
	```

	clone 完成后，进入 `gjs-app` 目录，可以跳过上面第二步中的 `ionic start` 部分，直接执行 `npm install` 。

	仓库上的项目文件不包含 node_modules 目录，所以需要自行执行 `npm install` 来安装依赖包，记住最好要为 node-sass 指定淘宝镜像源。

# 开发

进入项目目录，在命令行中执行 `ionic serve -b` ，Ionic 的命令行工具就会自动将项目文件进行编译打包，并在默认情况下会打开 8100 端口进行监听，在浏览器中输入网址 `http://localhost:8100` 可以看到构建出来的网页版 app 。

**`-b` 代表不要自动打开浏览器页面**。第一次执行时可以使用 `-b` 参数，但之后建议不带此参数，因为有时可能需要中断监视后再执行 serve 指令，使用 `-b` 可以避免每次都打开一个新的浏览器页面。

`ionic serve` 指令执行后，构建过程并不会退出，它会自动监视源文件，在发现文件变化时会自动执行构建过程。

Ionic2 与以前的版本不同，由于大量使用了 TypeScript （ ES6 的超集）的语法，为了保证代码的可用性、以及模块划分过细后资源加载的问题，因此编译与打包过程是必要的，不像以前版本那样可以即改即用。

Ionic2 的项目打包主要是使用 webpack ，速度较慢，暂时未找到优化的方法。 html 或 scss 文件修改时，构建过程很快；但 ts 文件修改时，构建过程基本会达到 5 ～ 20 秒。

## Macbook 开发报错的问题

在 Macbook 中执行 `ionic serve` ，可能会报告一个 `/node_modules/@types/jquery/index.d.ts` 的错误。暂时的解决方法是打开这个文件，将文件最后部分的：

```js
declare var $: JQueryStatic;
```

修改为：

```js
declare var $: any;
```

>此问题在项目使用了 jquery 的情况下会出现，但高交所的 APP 后期抛弃了 jquery 。

## 在 WebStorm 中开发

使用 WebStorm 打开项目目录，然后点击 Run 菜单中的第四项 `Run...` ，再点击 `Edit Configurations` ，对运行参数进行配置。原先设置的 `ionic build` 中填写的 Javascript file 是我电脑上的 Ionic 安装位置，需要将其修改为个人电脑上全局安装 Ionic 所在的位置。

配置完成后，就可以在 WebStorm 中运行这个 Ionic2 项目了。

但目前发现这种运行方式相当慢，除了 webpack 构建过程比较慢之外，在 WebStorm 中输入代码或者选择代码块、 html 标签块都会比较慢，有明显的卡顿现象。推测是由自动完成、代码提示等智能化功能造成的，暂时未找到优化的方法。

# 项目文件结构说明

## src 文件夹

`src` 文件夹中存放的是项目核心的源代码。其中：

1. `src/index.html`

	项目的起始文件。源码中包含了需要额外引用的 js 、 css 等文件（此项目早期版本对 Simditor 这个可视化编辑器进行了封装，使用时需要引入 jquery 与 Simditor 有关的各个文件。此封装可以作为包装第三方插件、数据双向绑定的参考。后期已删除，有需要可查阅旧版本的代码）。

	此外可以看到 app 的根元素 `<ion-app>` ，而源码中所有模块的 ts 文件与 html 模板文件都会被编译打包成 `build/main.js` 文件，所有的 Sass 文件会被合并打包成 `build/main.css` 文件。

1. `src/declarations.d.ts`

	`.d.ts` 文件是 TypeScript 的声明文件，用于存放 class 、 interface 等声明定义。可以在此文件中放置自定义声明或引入的第三方库的声明，声明后在整个项目中可用。 TypeScript 是 js 的预处理语言，是一种静态类型语言。若在此文件中缺少必要的声明，项目构建时有可能会报错。

1. `src/app`

	项目根组件所在目录。

	`app.module.ts` 是项目模块的声明文件，凡是新增了模块，都应当到这个文件里进行声明，否则项目编译或执行时将会报错。服务要声明在 providers 属性中，页面元素控件模块要同时在 declarations 与 entryComponents 属性中进行声明， imports 属性中声明了 Angular 、 Ionic 或引入的第三方组件， bootstrap 属性中则定义了项目启动所用的控件（在 Ionic 项目中基本固定为 `IonicApp` ）。

	>新增模块也可以不在 `app.module.ts` 中进行声明，可以声明在共享模块（ shared module ）或子模块中， `app.module.ts` 负责导入共享模块或子模块，这样可以将一部分局部使用的模块按照功能分离出去，让结构清晰，同时避免 `app.module.ts` 过于冗长。
	>从 Ionic 3.x 版本开始，使用命令行 g （ generate ）指令添加的 指令（ directive ）、组件（ component ）与管道（ pipe ），其声明引用会被自动添加到 `app.module.ts` 中；但页面（ page ）则不会，需要手动添加。

	`app.component.ts` 与 `app.html` 定义了根组件的 ts 代码与 html 模板。

1. `src/assets`

	项目所使用的资源文件。

1. `src/components`

	用于存放自定义组件的文件夹，将一些公用的组件（例如顶部横条、菜单、股票列表等等）放在此处。

	>由于之前 Ionic 命令行的 g 指令功能有错，生成自定义指令（ directive ）时也会将所生成文件放到 `components` 文件夹中，因此高交所 APP 早期版本曾经将自定义组件放在 `compolets` 文件夹中，但后期放弃了这种做法。

1. `src/i18n`

	多语言支持的名词定义文件。

	>多语言支持在高交所 APP 后期版本中已被删除，需要的话可参考旧版本代码。

1. `src/modals`

	一些模态对话框（ modal ）的定义。模态对话框原本也属于组件（ components ），但为了区分作用，特地将模态对话框的组件文件放在 `modals` 文件夹中，这是本项目的一个约定。

1. `src/pages`

	页面级别模块的定义。

1. `src/pipes`

	管道的定义。管道用于在 html 模板页上进行数据的过滤与格式转换等处理。

1. `src/plugins`

	自定义插件。暂时只放了一个可视化编辑器 simiditor 。

	>项目后期删除了此文件夹。

1. `src/providers`

	全局服务的定义，可用于放置全局变量、设置、无界面的公用方法（包括向服务器端提交请求并获取返回数据）等。由于没有界面，每个模块只需要一个 ts 文件，因此没必要额外创建文件夹（而 pages 、 components 等文件夹中，每个模块会单独创建一个文件夹，用于放置 ts 、 html 与 scss 文件）。

1. `src/theme`

	项目风格定义，在 `variables.scss` 中可以对 Ionic 预设的变量进行修改（覆盖预设的定义值）。

## 构建 APP 时生成的文件夹

以下几个文件夹并未包含在仓库中，会在构建 APP （例如： `ionic build android` ）时自动产生。

1. `hooks` 文件夹用于存放 Cordova 的钩子代码。 Ionic 构建出来的 app 是通过 Cordova 在手机上执行的，在此处书写代码可以调用或修改 Cordova 的功能，但是我们目前暂未使用。

1. `platforms` 文件夹中是构建手机 app 所依赖与产生的文件。

1. `plugins` 文件夹里存放的是 Ionic 与 Cordova 的一些插件，一般情况下不需要去改动。

1. `resources` 文件夹中存放的是构建 app 时使用的一些资源文件。

## 构建页面时生成的文件夹

`www` 文件夹是使用 `ionic serve` 或 `ionic build` 指令时自动构建出的，此文件夹也未包含在仓库中。

# 框架说明

## Angular

### 简介

AngularJS 从第二版起改名为 Angular （后文提到的 Angular 都是指 2.0 或更高版本），各项特性与第一版有很大的差别。

[Angular2 的官方文档](https://v2.angular.io/docs/ts/latest/)

[Angular 的中文版文档](https://angular.cn/docs/ts/latest/)，但此文档是基于最新版本的（目前是 4.0 ，与 2.x 差异不太大），而且尚未完全翻译，2.x 版本则没有官方中文文档。

Angular 主流代码基于 TypeScript （也有基于 js 或 Dart 的写法），代码比较明显的特征是 ES6 的模块导入/导出、 class 定义、装饰器（ decorator ）。

### TypeScript

TypeScript 是 js 的一种预编译语言，是一种强类型的语言，代码文件的扩展名为 ts 。它是 ES6 的超集，也就是说所有的 ES6 特性都可以使用，此外还有一些更新版本以及尚在 ES 草案中的特性也可以使用，最重要的是装饰器、类属性的访问性（公用、私有等）、类属性的快捷定义与赋初值、对象的扩展运算符等等。

[TypeScript 官方文档](https://www.typescriptlang.org/docs/tutorial.html)

TypeScript 可以对变量、函数参数、函数返回值等等进行类型约束。一般的方式是在变量（或参数等要素）后面添加冒号与类型说明，例如：

```ts
let ready:boolean = false;
```

此代码声明了一个布尔型的变量，并使用 false 对其进行赋值。

变量如果可能会有多种类型，可以在声明时使用 `|` 来分隔类型，例如： `:string | boolean` 。

如果变量可能是任意类型，声明时可以使用 `any` 作为其类型。声明变量、属性时未赋予初值，并且也没有进行类型声明，则默认类型为 `any` 。

函数可以声明参数类型与返回值类型。

TypeScript 除了提供对新的 ES 语法的支持外，最重要的就是通过类型与接口（ interface ）声明，规范了代码。这样在项目变得庞大时，各种方法调用、参数使用都能够得到类型约束，并且也可以配合代码编辑器或 IDE 实现代码提示、自动完成等功能，有助于项目代码的编写与协调工作。而最终实际执行的代码是 ts 编译过后产生的 js 文件（一般会编译为 ES5 的代码），也就是对实际运行没有影响，只影响代码编写、项目构建的阶段。

### RxJS

Rx 是一种基于可观察对象、观察者（订阅者）的事件处理框架，有多种不同的语言实现，能为异步处理提供强大的支持。

它相对于 Promise 的优势是订阅的事件（可观察对象）可以取消、重做、出错重试、节流（设定一定时间内只允许执行一个操作、有效防止重复提交）、映射、归并，一个可观察对象上可以有多个订阅，也可以将一系列可观察对象合并为一个可观察对象，以便进行统一处理。

Angular 在 Http 部分使用了 RxJS 进行异步处理。完整的 RxJS 比较庞大，因此程序只会载入 RxJS 的核心功能，需要额外载入其他功能时，应在代码中指定（例如 `import 'rxjs/add/operator/map';` ）。

>RxJS 额外功能（操作符）的载入只需要在项目中载入一次，项目中其他代码文件就可以直接使用相应的功能，因此为了直观与统一管理，建议将 RxJS 的额外载入全部书写在 `app.module.ts` 中。

最基本的范例可以参看 `src/providers/login-service.ts` ，此代码中的 `doLogin()` 方法中被注释的部分使用了 `.toPromise()` 方法将 RxJS 的操作结果转换为 Promise 。也可以不转换为 Promise ，直接使用 Observable ，参考 `/src/providers/http-service.ts` 中的 `getObservableWithToken()` 方法。

除了 Http 之外， RxJS 在 Angular 中的应用还很广泛。例如可以监视数据的变化，在变化发生后执行特定操作；可以结合 async 管道操作在页面上显示异步数据，等等。

### 装饰器

装饰器可以用于装饰类、类的属性或方法、函数参数，是 ES 草案中的特性，但现在已经有不少类库在使用它。写法是在所装饰对象的前面使用 `@` 符号，并带上装饰函数的名称与调用，例如 `@Injectable()` 或 `@Component({templateUrl: 'app.html'})` 。装饰器本质上是一个函数调用，它会将所装饰的对象（类、属性、方法或参数）作为参数去调用装饰器函数，并返回被装饰后的对象。在 Angular 中，装饰器的最主要用途是装饰类，为其注入元数据（ MetaData ）以及各种预定义的行为。这些可以参看项目中的各个 ts 文件。

在一些类中可以看到， `constructor()` 方法带有类型声明的参数，其顺序是不重要的，可以自由调整。这是 `@Injectable()` 装饰器所起的作用，它会根据 `constructor()` 方法的参数声明传入对应的参数值。并且若参数上带有 `public` 或者 `private` 关键字修饰，则会创建对应的类属性（共有或私有），未带有 `public` 或 `private` 的参数则只能在构造器内部使用。

`@Component()` 装饰器是 `@Injectable()` 的一个扩展派生，也拥有参数依赖注入的特性。

### TS 的类属性

在许多代码文件中（例如 `src/pages/tabs/tabs.ts` ），可以看到与 ES6 class 的一些区别。 在 `TabsPage` 这个 class 中声明了一些类属性，例如 `homeRoot: any = HomePage;` ，这种写法是 ES6 所不允许的（ ES6 的类属性只能在类方法中使用 `this.<属性名>` 的方式进行读取，不能直接写在类声明中）。此外，在 `constructor()` 构造函数中，参数声明时使用了 `public` 关键字进行修饰，这种写法会在类实例构造时同时创建类的公用属性（或使用 `private` 声明私有属性）。

```ts
class TabsPage {
  constructor(public navCtrl: NavController) {

  }
}
```

相当于

```ts
class TabsPage {
  public navCtrl;

  constructor(navCtrl: NavController) {
	this.navCtrl = navCtrl;
  }
}
```

这种写法也是 ES 的未来特性。

此代码若按照 ES6 的特性，应当写为：

```ts
class TabsPage {
  constructor(navCtrl) {
	this.navCtrl = navCtrl;
  }
}
```

### Angular 模板语法

Angular 的模板语法可以采用类似旧版本的写法，但推荐的是新语法。可以查阅 [官方中文文档](https://angular.cn/docs/ts/latest/guide/template-syntax.html) 。

#### 属性与事件的绑定

关于属性与事件的绑定，主要有：

* 属性值单向绑定使用方括号
* 事件调用使用圆括号
* 双向绑定使用方括号内嵌套圆括号

例如 `src/modals/input/input.html` ，底部按钮 `<button ion-button small item-right [disabled]="!newInput" (click)="submit()">提交</button>` 单向绑定了一个属性值 `disabled` ，并且定义了一个事件调用 `click` 。而在此上方有个输入框 `<ion-input placeholder="Title Input" clearInput [(ngModel)]="newInput"></ion-input>` 使用了双向绑定，类中预设的 `newInput` 属性值会被用于该输入框的初始化（或者在必要时响应代码执行中对于 `newInput` 属性值的修改），而输入框内的值变化时也会同步更新类的 `newInput` 属性值。

属性值的单向绑定也可以使用原先的插值写法（ `{{ }}` ），例如 `<button ion-button disabled="{{!newInput}}" (click)="submit()">提交</button>` 与 `<button ion-button [disabled]="!newInput" (click)="submit()">提交</button>` 一般情况下是等效的，官方推荐使用方括号的写法。

对于输入项来说，一般直接使用 `[(ngModel)]` 即可实现输入值的双向绑定。而对于自定义的控件，可以自行定义括号内的属性名称，参见 `src/pages/editor/editor.html` 中的 `[(content)]` ，在 `src/plugins/simditor/ng-simditor.ts` 中可以看到代码是如何定义与使用 `content` 这个属性的。

>`editor.html` 在后期已被删除，需要的话可查阅旧版本代码。

#### 绑定 html 属性

在 Angular 模板中为 html 标签或自定义标签绑定的属性（带有方括号或圆括号的），都是指 html 元素的**属性**（ property ），而不是**特性**（ attribute ）。二者的区别在于：特性是在一般 html 页面上直接写在 html 标签中的，而属性是通过 js 去访问 DOM API 获得的节点上的 js 属性。例如：

* 对于普通 html 页面中的 `<input type="text" disabled="disabled" />` 来说， `disabled` 是该 html 元素的特性，其值为 `disabled` （也可以写为 `<input type="text" disabled />` ，一样有效，作用是禁用该输入框）。而在 js 代码中，获取了该 html 元素的引用对象后，对象的 `.disabled` 的值不是一个字符串（ `disabled` 或空字符串 `` ），而是一个布尔值，值为 true 时代表该输入框被禁用。

* 直接在一般 html 页面的元素上书写 innerHTML 是无效的，因为 `innerHTML` 是一个属性，而不是 html 特性。例如： `<span id="test" innerHTML="test content"></span>` ，这种写法是无效的，需要改为写 `<span id="test">test content</span>` 。但在 js 代码中访问该元素的时候，则要使用 `innerHTML` 属性，例如：

	```js
	document.querySelector('#test').innerHTML = "test content new";
	```

在查看或修改 Angular 的模板时，必须牢记标签上绑定的是属性（使用了方括号、圆括号或在属性值上使用了 `{{ }}` 插值），而不是 html 特性。因此在模板文件 `src/pages/editor/editor.html` 中的 `<div [innerHTML]="simditorContent"></div>` 的写法才是正确的（也可以写为 `<div innerHTM]="{{simditorContent}}"></div>` ，但一般推荐前一种写法）。这里若写为 `<div>{{simditorContent}}</div>` ，则 simditorContent 的值会被 Angular 的引擎转义，导致其中包含的 html 标签代码被转换为文本，不能按预期显示。

#### style 绑定

在 Angular 的模板中， style 特性除了可以直接写成不绑定的方式（例如 `style="exam"` ）外，还可以针对某个样式属性这样写：

```html
<ion-nav
	[style.display]="loginService.status && loginService.ready ? 'block' : 'none'"
></ion-nav>
```

这段代码单独操纵了 `display` 样式属性。

>注：控制元素是否显示也可以写为：
>
>```html
><ion-nav [hidden]="!loginService.status || !loginService.ready"></ion-nav>
>```
>
>元素的 hidden 特性是 html5 的特性，只控制元素是否可见，而不会将元素从页面上移除。

#### class 绑定

设置元素的 class 也可以用类似的写法：

```html
<ion-nav [class.example]="loginService.status"></ion-nav>
```

这样，若 `loginService.status` 的值为真值，则实际生成的 html 标签的内容是：

```html
<ion-nav class="example"></ion-nav>
```

否则会生成

```html
<ion-nav class=""></ion-nav>
```

也就是可以使用开关变量来切换实际使用的 class 特性。有多个 class 需要处理时，可以按前面的写法重复书写，每个 class 写为一个 Angular 的属性，例如：

```html
<ion-nav
	[class.example]="loginService.status"
	[class.important]="loginService.ready"
></ion-nav>
```

当 `loginService.status` 与 `loginService.ready` 均为真值时，所生成的实际元素内容为：

```html
<ion-nav class="example important"></ion-nav>
```

#### Angular 指令

Angular 指令语法是模板语法的一部分，也有与旧版本类似的写法，但也推荐使用新语法。例如：

```html
<page-loading *ngIf="!loginService.ready"></page-loading>
```

这里使用了一个 `*ngIf` 来控制该元素是否渲染，后面表达式的结果为假值时，该元素会被移除（而不仅仅是隐藏）。

`*ngFor` 指令的用例可以参看 `src/pages/optional/optional.html` ，里面有如下代码：

```html
<ion-row *ngFor="let data of optionalListData; trackBy data?.id"
	class="more-dark-transparent bottom-hr">
  <ion-col padding>
    <div>
      <h5>{{data.name}}</h5>
      持仓 <span class="color-important">{{data.count}}</span>
    </div>
  </ion-col>
  <ion-col padding>
    <h5 class="{{data.changeRate | riseOrFall}}">
      {{data.price | number:'.2-2'}}
    </h5>
  </ion-col>
  <ion-col padding>
    <h5 class="{{data.changeRate | riseOrFall}}">
      {{data.changeRate | percent:'.2-2' | positiveSign}}
    </h5>
  </ion-col>
</ion-row>
```

注意 Angular 的模板循环指令使用的是 `let` 结合 `of` ，在这后面还可以使用分号分隔一些微指令语句，例如[官方文档](https://angular.cn/docs/ts/latest/guide/structural-directives.html#!#microsyntax)中的例子：

```html
<div *ngFor=
	"let hero of heroes; let i=index; let odd=odd; trackBy: trackById"
	[class.odd]="odd">
  ({{i}}) {{hero.name}}
</div>
```

其中 `trackBy` 微指令用于对循环的项目进行追踪，在数组元素可能会改变的场合，使用项目追踪可以改善渲染的性能。

>范例中提到的代码，在项目开发过程中已经被变更或调整了位置，请参阅项目最新的实际代码。

此外还有一个 NgSwitch 指令（由 `[ngSwitch]` 与 `*ngSwitchCase` 、 `*NgSwitchDefault` 配合使用），有待在今后项目中进行实践。

#### 安全导航操作符

在上面的代码中可以看到 `trackBy: data?.id` ，在小数点前面使用问号，表示需要使用安全导航。此处由于 `optionalListData` 数组的内容有可能为空，对于空对象访问其 `contryId` 属性将会导致错误。而 `?.` 安全导航操作符会在对象为空时不继续去获取其成员的值，保证属性值的获取不会出现异常。

安全导航操作符可以连续使用，例如 `item?.props?.name?.firstname` ，这么写大约相当于 `item && item.props && item.props.name ? item.props.name.firstname : undefined` ，使用 `?.` 避免了类型的安全检查，大大简化了代码。

### 父子组件通信

在 `src/plugins/simditor/ng-simditor.ts` 代码中，定义了一个私有变量 `_content` ，并定义了访问器属性 `content` 对这个私有属性的读写进行代理。

在 `content` 属性的读取访问器方法 `get content() {}` 上使用了 `@Input()` 装饰器，表示该属性可以（并且必须）从父组件传递一个值到子组件的 `content` 属性；而在 `content` 属性的写入访问器方法 `set content(val) {}` 中，调用了 `contentChange` 事件触发器，该触发器声明处使用了 `@Output()` 装饰器，表示该事件触发时会将所用的参数值传递给父组件。

这样，在 `src/modals/input/input.html` 中的 `[(content)]="simditorContent"` 就实现了 `EditorPage` 类实例的 `simditorContent` 属性与 `SimditorPlugin` 类实例的 `content` 属性的双向绑定。

>`ng-simditor` 在项目开发中已被移除。关于父子组件的通信，可参阅 echarts 相关组件的代码，例如 `/src/components/realtime-charts/realtime-charts.ts` 。

### 页面元素引用

在对象属性上使用 `@ViewChild('<目标元素的模板变量名>')` 可以获取对于目标元素的引用。例如在 `src/plugins/ng-simditor.html` 中，在 `<textarea>` 元素上使用 `#texterea` 定义该元素的模板变量名为 `texterea` ，然后在 `src/plugins/ng-simditor.ts` 中定义属性 `@ViewChild('texterea') texterea;` ，即可获得对页面上这个 `<textarea>` 元素的引用。

类似的装饰器还有 `@ViewChildren` 、 `@ContentChild` 与 `@ContentChildren` 。

## Ionic

Ionic2 基于 Angular2 ，而 Ionic3 基于 Angular4 。 Ionic2 与 Ionic3 的差别较小，却与 Ionic1 有巨大差异。

Ionic2+ 提供了许多预设的控件与样式，一般使用自定义标签形式（例如 `<ion-item></ion-item>`）。一些样式可以使用 html 标签属性的方式（例如 `<button ion-button>Submit</button>` ）。

相关文档可查阅 [Ionic Components](http://ionicframework.com/docs/components/) 以及 [Ionic API Docs](http://ionicframework.com/docs/api/) 。

在 `gjs-app` 这个项目中使用了 `<ion-nav>` 、 `<ion-header>` 、 `<ion-footer>` 、 `<ion-navbar>` 、 `<ion-title>` 、 `<ion-content>` 、 `<ion-list>` 、 `<ion-item>` 、 `<ion-label>` 、 `<ion-select>` 、 `<ion-option>` 、 `<ion-input>` 、 `<ion-menu>` 等等控件（其中有些控件暂时用作示例，以后可能会被移除），在少数地方书写了注释或对照代码。此外，对于打开新页面的方式，做了 modal 方式与 nav 方式的对比(参见“我的”页，仅作为示例)。表单的提交可以参看登录页，推荐使用 `FormGroup` 的方式。

Angular 的风格是将每个模块都写为单独的 ts 文件，而在 Ionic 中自定义模块有可能会带有 html 模板与 scss 样式表文件，因此习惯上每个模块单独用一个文件夹（ providers 与 pipes 除外）。

### Sass

Sass 是 css 的一种预处理语言（同类的还有 less 与 stylus 等）。它可以用编程语言的方式书写 css 代码，可以定义变量、进行循环判断等操作，也可使用 mixin 、 @extend 等方式对 css 的类声明进行混入（扩展）处理。

Sass 文件的扩展名一般是 `.scss` ，在文件中可以直接书写 css 代码，可以不包含 Sass 的任何特性。也就是说，从 css 到 scss 可以平滑过渡。 scss 一般会在项目构建时被转换为 css 文件，项目实际运行时使用的就是编译出来的 css 文件。

文件名前面带有下划线的 scss 文件不会被直接编译，它是用于嵌入其他 scss 文件的（使用 `@import` 导入）。

Sass 的变量使用 `$` 符号来识别，变量值允许是数值、字符串或元组（列表）等。

Ionic2 的样式使用 Sass 进行声明，预定义了许多 Sass 变量。可以通过修改这些变量的值来改变 app 的风格（ `src/theme/variables.scss` ）。相关资料可以查询 Ionic 的 [CSS Utilities](http://ionicframework.com/docs/theming/css-utilities/) 、 [Sass Variables](http://ionicframework.com/docs/theming/sass-variables/) 、 [Overriding Ionic Sass Variables](http://ionicframework.com/docs/theming/overriding-ionic-variables/) 等文档。

需要注意的是，修改 Ionic 预设的 Sass 变量，有可能每个改动都要写三次，针对于三种不同的平台（ ios 、 md 、 wp ）。

此外，如果修改可能影响到全局的样式，可以在 `src/app/app.scss` 这个文件上进行修改；而只影响到某个页面的样式，可以写在该页面所在目录的 scss 文件中。

### Ionic2 图标

Ionic2 预制了许多图标，在 html 元素或自定义元素上使用图标名称就可以使用，例如在 `src/pages/tabs/tabs.html` 模板文件中，可以试着把“自选”标签的图标进行修改：

```html
<ion-tab [root]="optionalRoot" color="lightfont" tabTitle="自选" tabIcon="optional"></ion-tab>
```

修改为：

```html
<ion-tab [root]="optionalRoot" color="lightfont" tabTitle="自选" tabIcon="home"></ion-tab>
```

这样就可以看到底部 tab 栏的第一个标签项的图标变为了小房子形状的图标。

Ionic2 的完整图标列表可以查阅 [IONICONS](http://ionicframework.com/docs/ionicons/) 。同一个图标基本上都有三种样式， Ionic 会根据 app 所运行的平台决定要使用哪一种。

需要注意的是， Ionic2 的图标并不是以图片文件的形式存在的，而是合并到一个字体文件中（ `/assets/fonts/ionicons.woff2` ），此字体的原文件被包含在 `ionicons` 扩展包中。

观察构建后页面的对应元素，可以看到 Ionic2 通过样式表为该元素指定了一个 `:before` 伪元素，具体样式为：

```css
.ion-ios-home:before {
    content: "\f448";
}
```

为 `:before` 伪元素指定了内容 unicode 字符 `"\f448"` ，而该字符对应的字形在 `ionicons.woff2` 字体文件中（由继承样式 `ion-icon { font-family: "Ionicons"; }` 所指定）。

如果今后我们需要创建自己的图标，一种方式是制作我们自己的 woff2 字体文件（参见[官方说明](http://blog.ionic.io/building-ionicons/)），另一种方式是创建图片文件，为自定义的 `:before` 伪元素指定 `background-image` 与 `width` 、 `height` 等属性（可以参见 `src/app/app.scss` 中的 `$ext-icons` 部分，它所调用的 `icons` 混入函数的定义在 `/src/app/_mixin.scss` 文件中）。

### Ionic 命令行

前面的内容涉及了 Ionic 命令行的 `ionic start` 、 `ionic serve` 指令，还稍微提及了 `ionic build` 指令。

除此之外，我们可能会比较常使用到的 Ionic 命令行指令还有 `ionic generate` （简写为 `ionic g` ），这个指令可以快速创建所需的代码文件。例如：

```
ionic g provider test-service
```

这行命令会在 `providers` 文件夹下创建一个 `test-service.ts` 代码文件，该代码会创建一个 `TestService` 类，使用 `@Injectable()` 装饰器进行修饰，这样就创建了一个名为 `TestService` 的服务组件。

注意：

* 命令行中使用单数形式（ provider ），而所创建的文件会在复数形式的 `providers` 文件夹中。
* 组件的名称在命令行中可以使用 PASCAL 命名（每个单词的首字母都大写，例如 TestService ）、驼峰命名（第一个单词首字母小写，后面每个单词首字母都大写，例如 testService ）或连字符命名（单词全部小写，相互之间使用连字符、也就是减号分隔，例如 test-service ）。三种形式的命名在命令行中都可以使用，效果没有区别。
* 对于实际生成的代码文件，文件名会使用连字符命名方式（如 `test-service.ts` ），而代码中的类名则会使用 PASCAL 命名方式（如 `TestService` ）。
* 若指定的组件名称的后缀与组件类别相同，则所生成的子文件夹与文件会自动去掉后缀。例如： `ionic g component test-component` ，则生成的文件是 `/src/components/test.ts` 等。

Ionic 命令行的 generate 指令的用于指示模块类型的参数主要有：

* `page` ：页面，会在 `src/pages` 文件夹下创建一个子文件夹，包含 `.ts` 、 `.module.ts` 、 `.html` 与 `.scss` 共四个文件。
* `provider` ：服务。
* `component` ：控件（自定义 html 元素）。
* `directive` ：指令（生成的文件会在 `components` 文件夹中，同时还会生成子文件夹）。
* `pipe` ：管道（可以参见 `src/pages/optional/optional.html` ，模板内使用了两个自定义管道 `riseOrFall` 与 `positiveSign` ，在 `src/pipes` 文件夹中可以找到对应的代码文件）。

## 第三方插件

### ECharts

安装依赖包：

```
npm install echarts --save
npm install @types/echarts --save
```

在使用 ECharts 的模块 ts 文件中引入：

```ts
import * as echarts from 'echarts';
```

使用：

在模块的 html 元素上定义模板变量名（使用井号），然后在 ts 文件中使用 `@ViewChild()` 装饰器获得对元素的引用。初始化此元素使用 `var myChart = echarts.init(this.echartsELem.nativeElement);` ，其中 `echartsELem` 应替换为实际使用的对模板元素的引用。

### 引入插件的额外方法

在前面介绍 RxJS 时已经有所涉及。导入 RxJS 的 Observable 时使用：

```ts
import { Observable } from 'rxjs/Observable';
```

但是这只是导入了 Observable 对象，没有同时导入一些相关的操作符（ operator ），例如 `Observable.debounceTime()` 、 `Observable.toPromise()` 等。这些方法是 Observable 对象的额外成员方法，需要另外导入。注意导入的并不是对象/类，而是对象/类上的方法，因此不能使用 `import { <SomeObject> } from '<path>';` 这样的方式，而是要使用：

```ts
import 'rxjs/add/operator/debounceTime';
import 'rxjs/add/operator/toPromise';
```

使用的是 ES6 模块的“无绑定导入”语法。

类似的还有 ECharts 的 `echarts-liquidfill` 插件，导入时使用：

```ts
import 'echarts-liquidfill';
```

这样就可以使用 Echarts `type='liquidFill'` 的图表了（参见 `/src/components/liquid/liquid.ts` ）。

# 编码风格

1. 原先的代码、包括网上找到的各种示范用例，基本上使用两个空格进行缩进。我们可以考虑保留这种风格。

1. 对于数组或是对象成员，如果分为多行书写，可以在最后一个元素后面使用逗号。这样在新增元素时，不需要改动两行。例如：

	```js
	  providers: [
	    StatusBar,
	    SplashScreen,
	    {provide: ErrorHandler, useClass: IonicErrorHandler},
	    LoginService,
	  ]
	```

	新增一个元素 `AppSettings` ，代码改为：

	```js
	  providers: [
	    StatusBar,
	    SplashScreen,
	    {provide: ErrorHandler, useClass: IonicErrorHandler},
	    LoginService,
	    AppSettings,
	  ]
	```

	使用 git 进行代码管理时，文件差异只会显示新增一行，而不会显示出修改一行同时新增一行。

	函数的参数如果比较多，可以改为一行写一个参数的形式，并在最后一个参数后面加逗号。数组或对象成员的可以在最后一个成员后多写一个逗号，这是 ES6 的语法；而函数参数最后多一个逗号则是 ES2017 的特性，但 TypeScript 已经支持了这种写法。
