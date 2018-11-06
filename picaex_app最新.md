# 综述  

该文档基于项目v0.1.6版本, git commitid为 f211d77883bc00da6f2afed3541d9e55bd2998bd

# 项目结构  

开发文件主要放在`./src`目录下,根据功能分类为[app根文件](#app),[静态文件资源](#assets),[组件](#components),[页面](#pages),[指令](#directives),[管道](#pipes),[服务](#providers),[共享模块](#shared)等,下面根据文件夹整理.  

## <span id = "app">`./app`</span>  

app文件夹是app的根目录,  
`app.component`主要在app准备完成对app进行初始化设置,详见`afterPlatformReady()`; 预加载数据,如产品信息; 初始化服务,如给`keyboardservice`传入rander2实例.  

`app.module`文件主要配置组件/模块的引入, 以及路由的配置, 项目只在最初有尝试做模块划分, 故有存在引入共享模块以及直接引入组件两种模式, 后采用直接引入组件的模式, 若重构可以考虑从确定需求开始做功能模块划分. 对app功能模块解耦. 另一方面, 基于现在项目的复杂度判断, 不太需要进行模块解耦封装, 具体怎么取舍建议: 重构前保持当前全部注入appmodule的模式, 重构根据具体情况确定.  

`_mixin.scss & app.scss` app样式预设,mixin主要是一些项目中一些样式的处理函数, 若在项目中遇到搜不到的选择器, 可以翻看mixin检查. 这里再提一下另一个文件夹下的文件,`../theme/variables.scss` 此文件主要用来编辑样式变量以及自定义ionic的变量值的配置,在项目中大面积覆盖,需提前了解.  

`main.ts` 项目初始化文件之一, 与项目编译相关, 在项目aot编译时会有警告说该文件已被修改(~~不是我,我没有,别说瞎说啊~~),需要去官方项目上拷贝一个下来,可是其实是一模一样的,**而且照做了警告也依然存在, 照警告描述默认也替换成了正确的文件,故考虑不理他**.(重新建一个项目这问题不会存在...)  

## <span id = "assets">`./assets`</span>  

assets文件夹主要存放图片资源,包括icon,banner,background图,主要注意下面这个问题:  
1. assets文件夹在html,css,ts文件夹中相对路径要少一到两级的父级路径搜索, 如: 在开发中的css文件下, 常规情况是`'../../assets/yourasset'`, 而在项目里, 路径应为`'../assets/yourasset'`才能正常索引到资源.  

## <span id = "bnlc-framework">`./bnlc-framework`</span>  

这个文件夹由[@肇丰](#gaubee)引入的app底层框架, 对ionic做了一些封装. 由于这个框架是在肇丰参与该项目时候引入的, 故存在部分页面使用这个框架而其他的部分没使用的情况,想多了解该文件夹相关信息,参考[该文档](https://github.com/Gaubee/bnqkl/blob/master/bnqkl%20framework%E8%BF%99%E4%B8%AA%E6%96%87%E4%BB%B6%E5%A4%B9%E7%9A%84%E4%B8%80%E4%BA%9B%E4%BB%8B%E7%BB%8D.md) (ps.后面肇丰去做别的项目了而该项目里的框架没有更新,可能会有一些细微差别或者bug~)  

## <span id = "components">`./components`</span>  

该文件夹存放项目组件, 会重点介绍当前项目还在使用的组件  

### <span id = "echarts-base">`./echarts-base`</span>  

该组件是项目图表的基础, 将echart封装成一个基类, 包含基础的初始化和数据变化监听等.  

### <span id = "realtime-report">`./realtime-report`</span>  

该组件继承自[EchartsBaseComponent](#echarts-base), 主要封装配置项信息以及传入数据的处理,具体配置方法参考[echart官方配置项手册](http://echarts.baidu.com/option3.html)  

### <span id = "charts">`./bar | ./candlestick | ./ distanceline | ./liquid | ./pie | ./realtime-charts | ./smoothline | ./volumn `</span>  

这些组件都是对echarts的各类图标配置的封装, 项目中暂时没有用到, 可以用来作为参考, 建议重构时把这些组件删除.

### <span id = "rich-text">`./rich-text`</span>  

该组件是一个富文本展示组件,用于资讯新闻公告的详情页. 为保持后端传来的富文本格式, 需要注意用了`<div class="content" [innerHTML]="dataSource?.content"></div>`直接加载服务端返回文本, 故不要饮用外部文本, 或是补充添加数据安全处理.  

### <span id = "comments">`./comments`</span>  

该组件是个显示评论内容的组件, 用于资讯新闻公告的详情页. 通过流触发数据请求:  

```typescript  

  getComments() {
    this.getCommentsTermStream.next();
  }

  ngOnInit() {
    //TODO:管道和observable异步 有转换问题
    this.comments = this.getCommentsTermStream
      .debounceTime(300)
      .switchMap(() => this.commentService.getComments(this.newsId))
      .do((data) => {
        console.log('getted')
        this.commentListLength = data.length
        this.title = this.likedComment.length ? "热门评论" : "评论";
      })
      .catch((err) => this.commentService.handleError(err, true))
    setTimeout(() => this.getComments(), 0); 
  }

```  

这个地方有点问题, 使用setTimeout将获取评论的方法加入异步队列来解决当时存在的asyncpipe订阅不到comments的bug. 有时间需要重新验证.  

### <span id = "inputer">`./inputer`</span>  

该组件是输入框组件, 一开始不懂事, 组件粒度拆得过小, 不建议这么分配. 组件提供事件触发接口, 以及简单的rx监听逻辑, 可以作为骨架进行丰富定制需求. 初学rx时写的基础的代码, ~~命名比较随性, 将就着看~~  

```typescript  

    ngOnInit() {
      this.streamSubscription = this.sendTermStream
        .debounceTime(300)
        // .filter(value => value != undefined)
        .map(value => value && value.trim())
        .filter(value => !!value)
        .distinctUntilChanged()
        // .do(value => console.dir(value))
        .subscribe((term: string) => {
          this.sendMessageEmit(term)
          /**
           * 通过双向绑定清除输入框内容
           * TODO:根据发送成功后才清除.
           */
          this.inputValue = ''
        })
    }

```  

### <span id = "kjua-qrcode">`./kjua-qrcode`</span>  

肇丰完成的二维码组件, 项目中暂时没有用到, 如若需要使用可以问他. 

### <span id = "image-taker-controller">`./image-taker-controller`</span>  

该组件是对拍照功能的封装,使用方式是先通过controller创建一个imagetaker的实例,然后配置显示,参考[IdentificationPage](#identification)的以下代码:

```typescript  
    upload(name) {
      const imageTaker = this.imageTakerCtrl.create(name);
      const fid_promise = this.fs.getImageUploaderId(FileType.身份证图片);
      imageTaker.onDidDismiss((result, role) => {
          if (role !== 'cancel' && result) {
              const image = this.images.find(
                  item => item.name === result.name
              );
              // console.log('index: ', index, result);
              if (result.data) {
                  // 开始上传
                  this.updateImage(fid_promise, image, result);
              } else {
                  image.image = 'assets/images/no-record.png';
              }
              // console.log(this.images);
          }
      });
      imageTaker.present();
    }
```  

## <span id = "directives">`./directives`</span>  

该文件夹存放指令, 同样着重介绍在用的指令  

### <span id = "copy">`./copy`</span>  

该指令实现将制定文本复制到剪贴板, 通过注入剪贴板实例, 会覆盖注入元素原来的点击事件触发的复制文本, 实现在模板文件上配置剪贴板文本. (*我通过测试得出来的,具体为什么这么做我也不知道*)  

### <span id = "scroll-fix-keyboard">`./scroll-fix-keyboard`</span>  

该指令实现软键盘弹出时, 将指令附着的元素滚动到content可视区域的顶部的功能. 为了与ionic自身的滚动事件不冲突,采用content实例的scrollTo方法, 经测试可正常使用. 参考[TradeInterfaceV2Page](#trade-interface-v2)模板页的使用方式:  

```html  
  <!-- 限价交易 -->
	<ion-grid [hidden]="appDataService.show_onestep_trade" [scroll-fix-keyboard]="content">
  ...
  </ion-grid>	
```  

### <span id = "scroll-fix-x">`./scroll-fix-x`</span>  

该指令实现与前一个指令类似的效果, 作用在水平方向上, 当前项目中没有使用, 注释完整, 可以拿来参考编码.  

### <span id = "liked">`./liked`</span>  

该组件通过传入的值改变依附元素上的样式, 是指令比较基础的用法, 之前用在评论元素上, 项目中现在没用到, 重构时建议删除.  

### <span id = "directives.module">`./directives.module.ts`</span>  

该文件存储并导出需要用到的指令, 在appmodule里直接引入该模块, 不比每新增一个指令都到appmodule里添加, 一种可以参考的模块划分方式.  

## <span id = "modals">`./modals`</span>  

该文件夹存放模态窗(组件), 项目原来划分独立出来文件夹, 个人认为没有必要, 可以归在组件里. 或者按分类划分组件, 看你取舍. 目前这个文件夹里大部分组件没有用到.  

### <span id = "camera">`./camera`</span>  

该组件实现模拟相机模态窗, 是现在项目中唯一一个在使用的模态组件, 存在遗留问题, ios按下拍照后会卡主一会不能操作, 需要修复.  

## <span id = "pages">`./pages`</span>  

项目核心, 在经过项目初期尝试后, 大部分页面都独立开发, 也存在需求迭代中遗留废弃不用的页面, 本段文档尽量介绍全, 尽量尽量...  

### <span id = "about">`./about`</span>  

项目基本信息页面, 现放在 `我的-关于我们` 跳转. 包含公司信息联系方式和app版本号, 静态页面, 版本号每次发布需要手动修改. 版本号修改相关看[版本发布](#version)  

### <span id = "account-center">`./account-center`</span>  

用户账户中心页面, `我的-右上角齿轮` 跳转. 用户个人基本信息展示和设置的入口, 函数简单无难点.  

### <span id = "account-center-v2">`./account-center-v2`</span>  

已废弃的一版账户中心页面(是一级页面), 参考设计图 *毕加所app-账户中心.psd* , 只有页面布局, 无功能函数, 可不看, 重构时删除.  

### <span id = "add-address">`./add-address`</span>  

添加提现地址页面. 基于[bnlc-framework](#bnlc-framework)开发, 简单的表单页.  

### <span id = "auth-pending">`./auth-pending`</span>  

待审核状态页面, 已废弃.  

### <span id = "change-trade-password">`./change-trade-password`</span>  

修改交易密码页面. 基于[bnlc-framework](#bnlc-framework)开发, 简单的表单页.  

### <span id = "commission-list">`./commission-list`</span>  

已废弃的一版委托记录页面. 现在用[历史交易](#history-record)页面, 功能类似.  

### <span id = "contact">`./contact`</span>  

废弃页面, 无功能.  

### <span id = "create-account-confirm">`./create-account-confirm`</span>  

废弃注册条款确认页, 高交所遗留页面, 查看注册相关协议(弹出模态框)及选中确认注册. 简单.  

### <span id = "create-account-prompt">`./create-account-prompt`</span>  

废弃注册申明页面, 高交所遗留, 显示开户要求已经提供跳转注册功能. 只有页面跳转功能, 无参考价值, 可删除.  

### <span id = "create-account-step">`./create-account-step-first | ./create-account-step-second | ./create-account-step-third`</span>  

废弃注册页面, 高交所遗留, 分步注册, 功能类似, 都是表单提交页面跳转等简单功能, 个人信息表单验证工具函数可供提取.  
*create-account-step-first.ts* `ngAfterViewInit`方法里使用rxjs实现 *发送验证码倒计时* 写法可以改进封装成工具函数:  

```typescript  
const getCodeElement: HTMLElement = this.getCode.nativeElement || this.getCode._elementRef.nativeElement
this.getCode$ = Observable.fromEvent(getCodeElement, 'click')
    .debounceTime(300)
    // .do()
    .subscribe((event: MouseEvent) => {
        getCodeElement.setAttribute('disabled', 'disabled')
        let timer
        const countDown = (count: number) => {
            if (count <= 0) {
                getCodeElement.textContent = `获取验证码`
                getCodeElement.removeAttribute('disabled')
                return false
            }
            getCodeElement.textContent = `${count}s后重试`
            timer = setTimeout(() => {
                countDown(--count)
            }, 1e3);
        }
        countDown(60)
```  

*create-account-step-second* 银行卡号验证和邮箱地址简单验证工具函数:  

```typescript  
    luhmCheck(e: string) {
        if (!/^\d+$/.test(e)) {
            return false
        }
        if (e.length !== 16 && e.length !== 19) {
            return false
        }
        var x = e.slice(-1);
        // [...e] 在某些应用场景下会报错。
        // const A = [...e].reverse().slice(1);
        const A = Array.from(e.slice(0, -1)).reverse();
        var s = [];
        var a = [];
        var g = [];
        for (var v = 0; v < A.length; v++) {
            const num = +A[v];
            if (v % 2 == 0) {
                if (num < 5) {
                    s.push(num * 2)
                } else {
                    a.push(num * 2)
                }
            } else {
                g.push(num)
            }
        }
        var d = [];
        var c = [];
        for (var y = 0; y < a.length; y++) {
            d.push(a[y] % 10);
            c.push(Math.floor(a[y] / 10));
        }
        const sum = (a, b) => a + b;
        var z = s.reduce(sum);
        var u = g.reduce(sum);
        var l = d.reduce(sum);
        var f = c.reduce(sum);
        var total = z + u + l + f;
        var t = total % 10 || 10;
        var B = 10 - t;

        return +x === B;
    }
    queryHasEmail(emailValue) {
        return /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/.test(emailValue);
    }
```  

*create-account-step-second* 根据规则验证身份证号码工具函数:  

```typescript  
    checkIdCardNo(e) {
        //15位和18位身份证号码的基本校验
        var check = /^\d{15}|(\d{17}(\d|x|X))$/.test(e);
        if (!check) return false;
        //判断长度为15位或18位
        if (e.length == 15) {
            return this.check15IdCardNo(e);
        } else if (e.length == 18) {
            return this.check18IdCardNo(e);
        } else {
            return false;
        }
    }
    //校验15位的身份证号码
    check15IdCardNo(e) {
        //15位身份证号码的基本校验
        var check = /^[1-9]\d{7}((0[1-9])|(1[0-2]))((0[1-9])|([1-2][0-9])|(3[0-1]))\d{3}$/.test(
            e
        );
        if (!check) return false;
        //校验地址码
        var addressCode = e.substring(0, 6);
        check = this.checkAddressCode(addressCode);
        if (!check) return false;
        var birDayCode = '19' + e.substring(6, 12);
        //校验日期码
        return this.checkBirthDayCode(birDayCode);
    }

    //校验18位的身份证号码
    check18IdCardNo(e) {
        //18位身份证号码的基本格式校验
        var check = /^[1-9]\d{5}[1-9]\d{3}((0[1-9])|(1[0-2]))((0[1-9])|([1-2][0-9])|(3[0-1]))\d{3}(\d|x|X)$/.test(
            e
        );
        if (!check) return false;
        //校验地址码
        var addressCode = e.substring(0, 6);
        check = this.checkAddressCode(addressCode);
        if (!check) return false;
        //校验日期码
        var birDayCode = e.substring(6, 14);
        check = this.checkBirthDayCode(birDayCode);
        if (!check) return false;
        //验证校检码
        return this.checkParityBit(e);
    }

    checkAddressCode(e) {
        var check = /^[1-9]\d{5}$/.test(e);
        if (!check) return false;
        if (this.provinceAndCitys[parseInt(e.substring(0, 2))]) {
            return true;
        } else {
            return false;
        }
    }

    checkBirthDayCode(birDayCode) {
        var check = /^[1-9]\d{3}((0[1-9])|(1[0-2]))((0[1-9])|([1-2][0-9])|(3[0-1]))$/.test(
            birDayCode
        );
        if (!check) return false;
        var yyyy = parseInt(birDayCode.substring(0, 4), 10);
        var mm = parseInt(birDayCode.substring(4, 6), 10);
        var dd = parseInt(birDayCode.substring(6), 10);
        var xdata = new Date(yyyy, mm - 1, dd);
        if (xdata > new Date()) {
            return false; //生日不能大于当前日期
        } else if (
            xdata.getFullYear() == yyyy &&
            xdata.getMonth() == mm - 1 &&
            xdata.getDate() == dd
        ) {
            return true;
        } else {
            return false;
        }
    }
```  

### <span id = "entrance">`./entrance`</span>  

废弃入口页面, 高交所遗留, 只有跳转功能, 可直接删除.  

### <span id = "forget-pwd">`./forget-pwd`</span>  

忘记密码页面, 表单验证提交, 没什么特殊的.  

### <span id = "fund-statement">`./fund-statement`</span>  

资金变动查询页面, 高交所遗留, 暂时无用. 页面元素与设计参考 *高交所app-资金变动查询.psd* 和 *高交所app-资金变动查询-月度收益.psd* . 纯demo 展示静态样式.  

### <span id = "help">`./help`</span>  

帮助页面, 高交所遗留, 已废弃. 工单帮助功能目前为[我的工单](#work-order-list)页面作为入口.  

### <span id = "history-record">`./history-record`</span>  

历史成交页面, 三列式列表, 当前需求是展示后端提供的交割记录, 用到ionic的下拉刷新和无限滚动组件.  

### <span id = "home">`./home`</span>  

`我的`页面, 四个一级页面之一. 需要登录后才能查看. 个人信息相关的入口, 具体看app. 这里为了处理数据加载后页面header盖住content的布局问题. 调用`this.content.resize();`来手动触发content布局调整.  

### <span id = "identification">`./identification`</span>  

上传身份证照页面, 高交所遗留, 基于[bnlc-framework](#bnlc-framework)开发, 后更改需求设计后改用[实名认证页面](#submit-real-info)进行个人信息验证提交, 具体上传图片处理参考实名认证页面`ts`.  

### <span id = "iframepage">`./iframepage`</span>  

废弃页面...没有内容  

### <span id = "image-picker">`./image-picker`</span>  

独立的拍照页面, 高交所遗留, 废弃, 现今项目中使用[拍照组件](#image-taker-controller),  

### <span id = "info-tabs">`./info-tabs`</span>  

原高交所新闻资讯tab的头部新闻公告切换的tab, 一个页面两个tab导致一些不可预料的问题, 故废弃, 不建议使用.  

### <span id = "information">`./information`</span>  

原高交所资讯tab功能模块, 暂时无用. 是模块化开发的一个尝试, 涉及到功能模块, 共享模块的代码组织. 作为一个独立功能模块, 文件夹中包含 `information.module.ts` 文件, 和 `app.module.ts` 类似, 引入必要组件/模块和服务, 然后导出这个页面文件, 其他地方则是引入该模块即可使用.  

```typescript  
    //information.module.ts
    @NgModule({
        //定义使用的组件
        declarations: [
            InformationPage,
            NoticeListPage,
            NewsListPage,

            NewsContent,
            NoticePage,
        ],
        //手动指定二级页面,指引编译器编译,不然在跳转时会报错找不到页面
        entryComponents:[
            NewsContent,
            NoticePage,
        ],
        providers: [
            AlertService,
            NewsService,
        ],
        imports: [
            BaseSharedModule,//导入共享模块
            InfoSharedModule,//导入共享模块
            // CommonModule,
            // NewsContentModule,
            // NoticeModule,
            IonicPageModule.forChild(InformationPage),
        ],
        exports: [
            InformationPage,
        ]
    })
```  

```typescript  
    //app.module.ts
    @NgModule({
        ...
        imports: [
            ...
            InformationPage,            
        ],
    })
```  

```typescript  
    //tabs.ts
    export class TabsPage{
        infoTabsRoot: any = InformationPage;
    }
```  

当时认为, 以功能页面封装划分模块, 保证各个功能模块的独立性, 根据[Angular官方文档](#https://angular.cn/guide/ngmodules)提到的: *"对于那些只有少量组件的简单应用，根模块就是你所需的一切。 随着应用的成长，你要把这个根模块重构成一些特性模块，它们代表一组密切相关的功能集。 然后你再把这些模块导入到根模块中。"* 这是一种比较适用于大型项目的工程化实践. 事实上, 我们的项目由于人手和需求的问题, 可能并不需要将项目粒度分到这么细, ~~一个app根module一把梭就好~~, 后期重构, 长远考虑可能会想丰富app功能, 项目复杂度预想会增加很多的情况下, 可以考虑使用模块的方式组织项目各部分功能, 方便功能划分和分工, 各部分独立开发, 尽量降低共同修改某文件(如app.module.ts)的情况. 在项目管理上也能有些许帮助.吧.  

### <span id = "invite-commission">`./invite-commission`</span>  

邀请返佣的页面, 由陈峰完成. 主要用到指令注入推荐地址(参考[copy组件](#copy)).  

### <span id = "loading">`./loading`</span>  

废弃页面...  

### <span id = "login">`./login`</span>  

登录页面. 页面本身逻辑较为简单. 根据目前项目中的登录逻辑, 都是在指定情况下弹出登录框, 所以使用ionic的events订阅通知模式:  

```typescript  
    //app.component.ts
    constructor(){
        ...
        this.events.subscribe('show login', (status,cb?) => {
            this.events.unsubscribe('show login')
            if(cb){
                this.modalController
                .create(LoginPage, {cb})
                .present()
            }else{
                this.loginModal.present()
            }
            // console.log('events.subscribe:',page)
            // this.rootNav.push(page)      
        })
    }

    //其他地方通知弹出登录界面
    this.events.publish('show login', 'login');

    //login.ts (需要做注册硬件返回,以及在退出页面时重置硬件返回的注册和事件监听注册)
    ngOnInit() {
        ...
        this.unregisterBackButton = this.platform.registerBackButtonAction(() => {
            this.dismiss();
        })
    }

    dismiss(){
        this.unregisterBackButton();
        this.viewCtrl.dismiss()
        this.events.subscribe('show login', (status, cb?) => {
            this.events.unsubscribe('show login')
            const modal = cb ? this.modalController.create(LoginPage, { cb })
                : this.modalController.create(LoginPage)
            modal.present()
            // this.rootNav.push(page)      
        })
    }
```  

### <span id = "modify-pwd">`./modify-pwd`</span>  

修改密码页面, 围绕表单FormGroup开发, 表单页面可参考.  

### <span id = "news-content">`./news-content`</span>  

资讯内容页, 包含[富文本组件](#rich-text)和[评论显示组件](#comments), [input组件](#inputer), 评论和input组件. 直接使用模板语法联系起评论组件和input组件, 参考代码:  

```html  

<comments #commentList [newsId]="newsId"></comments>

<inputer (sendMessage)="commentList.postComment($event)"></inputer>

```  

### <span id = "news-list">`./news-list`</span>  

该页面现在的项目中是资讯新闻公告的一级页面, 由肇丰二次开发, 原本只是新闻列表页的二级页面. 通过控制ionic的slider组件控制切换显示的分类, 同时可以通过右上角搜索按钮调用后端资讯搜索接口, 因为可以在多个页面使用, 这部分可以考虑封装成组件, 具体代码参考以下部分:  

```typescript  

    @ViewChild('searchInputWrap', { read: ElementRef }) searchInputWrap;
    private showSearch = false;
    private query:string = '';
    private searchTermStream = new BehaviorSubject<string>('');
    search(term: string) {
        this.searchTermStream.next(term);
    }

    ngOnInit() {
        this.searchTermStream
            // .takeUntil(this.viewDidLeave$)
            .debounceTime(300)
            .map(str => str && str.trim())
            .distinctUntilChanged()
            .do(str => console.log('searchNews:Stream',str))
            .filter(str => str != this.query)
            // .switchMap((term: string) => Observable.of(term.trim().toLowerCase()))
            .subscribe(str=>this.searchNews(str))
    }

    toShowSearch() {
        this.showSearch = true;
        this.renderer.setElementStyle(this.searchInputWrap.nativeElement, 'width', 'unset');
    }

    cancelFilter() {
        this.showSearch = false;
        this.query = '';
        if (this.tabIndex == 0) {
            this.initNoticeList();
        } else if (this.tabIndex == 1) {
            this.initNewsList();
        }
    }

    async searchNews(str:string){
        this.query = str;
        if (this.tabIndex == 0) {
            this.noticeList.list = await this._getNoticeList();
        } else if (this.tabIndex == 1) {
            this.newsList.list = await this._getNewsList();
        }
    }

```  

```html  

    <ion-buttons right padding-horizontal [style.visibility]="showSearch? 'hidden':'visible'" (click)="toShowSearch()">
        <button class="button_self" ion-button>
            <ion-icon class="large-icon" name="search"></ion-icon>
        </button>
    </ion-buttons>
    <ion-toolbar [hidden]="!showSearch" class="searchbar">
        <div #searchInputWrap class="searchInput">
            <!-- <div class="icon"></div> -->
            <ion-item no-padding no-lines>
                <ion-input #searchInput type="text" clearInput (ionChange)="search(searchInput.value)">
                </ion-input>
            </ion-item>
        </div>
        <ion-buttons right (click)="cancelFilter()">
            <button ion-button navPop>取消</button>
        </ion-buttons>
    </ion-toolbar>

```  

```scss  

    .searchbar{
        position: absolute;
        left: 0%;
        bottom: 0%;
        font-size: 1.5rem;
        
        .searchInput{
            position: relative;
            margin-left: 2.6rem;
            transition:width 2s;
            width: 0;
            .clearInput{
                position: absolute;
                bottom: 0%;
                font-size: 1.5rem;
            }
            ion-input {
                border: 0;
                height: 3.2rem;
                width: 100%;
                border-radius: 1.6rem;
                background-color: rgba(255, 255, 255, 0.12);
                padding: 0 0 0 3.2rem;
                input {
                    margin: 0;
                    line-height: 3.2rem;
                }
            }
        }

        span {
            display: inline-block;
            width: 4.6rem;
            margin-left: .5rem;
            text-align: center;
            line-height: 3.2rem;
            vertical-align: middle;
        }
    }

```  

### <span id = "notice">`./notice`</span>  

废弃公告demo页, 没有有用的内容, 可直接删除.  

### <span id = "notice-list">`./notice-list`</span>  

高交所公告列表页, 也是常见的无限滚动和下拉刷新组件加上列表组成, 没有特殊的代码, 可以不看,删除.  

### <span id = "optional">`./optional`</span>  

持仓页(四个一级页面之一), 登录成功后请求`position/totals`, 目前返回的是所有的产品, 数据里包含用户持仓数据. 布局方面通过写死高度, 防止数据加载后header变高导致盖住content部分, 数据显示则是通过多个管道将源数据进行处理最终得出显示, 具体管道说明将在后面[pipe文件夹](#pipe)说明, 页面结构较为简单就不多赘述. 数据请求主要用rx来控制流, 逻辑比较简单, 包含请求数据以及刷新数据的方法.

### <span id = "quotation-detail">`./quotation-detail`</span>  

高交所遗留页面, 之前作为行情的二级页面存在, 已废弃, 经检查没有特殊的处理函数, 可直接删除.  

### <span id = "quotations">`./quotations`</span>  

币加所第一版行情页面, 基于高交所行情图表重写页面结构, 参考psd *毕加所app-行情界面* . 现在采用[v2版行情页面](#quotations-v2). 由于v2版本是在该版本CV大法后二次开发完善的, 相关内容参考v2版行情页说明即可. 页面结构差异较大  

### <span id = "quotations-v2">`./quotations-v2`</span>  

币加所现在的行情页面(四个一级页面之一), 页面与第一版本完全不同, 看到psd *行情页面已选择* 以及 *行情页面未选择* . 页面包含header上展示mainproducts的名称和换算法币价格, 搜索(通过关键字过滤实现, 和新闻不一样 不重新请求)功能, 点击mainproduct过滤交易对. content则是各个交易对的实时行情, 包括交易对名/ 实时价格/ 涨幅/ 最高最低价/ 换算法币价格/ 成交量/ 实时价格折线图. 所有数据都由websocket后端推送.  

搜索功能代码和[资讯新闻公告](#news-list)一样, 故将其抽为组件后可以简化大段代码, 其他主要代码就是搜索过滤, 现在采用的是 `traderList_show` 作为视图的源数据, 搜索过滤时候对该数组进行过滤, 取消则吧源数组赋值回来, 关键代码如下:  

```typescript  

	_filterProduct(product) {
		if (product) {
			product = product.trim().toLowerCase()
			this.traderList_show = this.traderList.filter((item: any, index, arr) => {
				return item.traderName.toLowerCase().indexOf(product) !== -1
			}).sort((a: any, b: any) => {
				return a.priceId - b.priceId
			})
		} else {
			this.traderList_show = this.traderList;
		}
	}

	cancelFilter(){
		this.showSearch = false;
		this.traderList_show = this.traderList;
		this.searchInputValue = '';
	}

```  

交易对行情的获取, 建议重构迁移到一个数据中心. 该函数在页面初始化时候调用, 将交易对的实时行情填入交易对对象中, 主要用到rx多播的功能来订阅同一条数据源, 处理后赋给各个交易对. 关键代码如下:   

```typescript  

	/**
	 * 获取多支行情数据
	 * TODO:迁移到数据中心处理
	 */
	async subscribeRealtimeReports(){
		await this.appDataService.traderListPromise;
		const srcTraderList = (this.appDataService.traderList.size ? this.appDataService.traderList
			: await this.tradeService.getTradeList()) as Map<string,AnyObject>
		// const traderList = [...this.appDataService.traderList.keys()]
		const traderIdList = []
		const traderList = []
		srcTraderList.forEach((value,key,map)=>{
			traderList.push(value)
			traderIdList.push(key)
		})
		this.traderList = traderList
		this.traderList_show = this.traderList
		this.realtimeReports$ = this.socketioService.subscribeRealtimeReports(traderIdList)
			// .takeUntil(this.viewDidLeave$)
		
		const refCount = this.realtimeReports$.multicast(new Subject()).refCount()

		srcTraderList.forEach((value, key, map) => {
			value.reportRef = refCount
				// .takeUntil(this.viewDidLeave$)
				.filter(({ type})=>{
					return type === value.traderId
				})
				.map(data=>data.data)
				.map(data => {
					//处理增量更新
					const srcArr = value.reportArr
					const length = srcArr.length
                    //这边因为socket根据订阅时间传了不同参数, 故做了临时特殊处理, 有时间请重新考虑.
					if(length == 0){
						srcArr.push(...data)//使用push+解构赋值,预期echarts动画实现
					}else{
						srcArr.splice(0,Math.min(length,data.length),...data)
					}
					if( length > this.appSettings.Charts_Array_Length){
						srcArr.splice(0, length - this.appSettings.Charts_Array_Length)
					}
					return srcArr.concat()
				})
			this.stockDataService
				.subscibeRealtimeData(value.traderId, 'price')
				.subscribe(value.marketRef)//, this.viewDidLeave$)
		})
	}

```  

### <span id = "recharge-detail">`./recharge-detail`</span>  

数字资产充值页面, 由肇丰完成, 继承自[bnlc-framework](#bnlc-framework), 主要是获取产品信息, 显示可用余额和冻结余额, 充值地址为请求后端提供, 复制地址功能使用了剪贴板插件, 参考[copy指令](#copy), 用装饰器修饰函数, 实现通用处理的方式值得借鉴:  

```typescript  

	@asyncCtrlGenerator.success('地址已经成功复制到剪切板')
	@asyncCtrlGenerator.error('地址复制失败')
	async copyCode() {
		if (!this.recharge_address.paymentAccountNumber) {
			throw new Error('无可用地址');
		}
		if (!navigator['clipboard']) {
			throw new Error('复制插件异常');
		}
		navigator['clipboard'].writeText(
			this.recharge_address.paymentAccountNumber,
		);
	}

```  

### <span id = "withdraw-detail">`./withdraw-detail`</span>  

提现页面, 与[充值页面](#recharge-detail)基本类似, 跳转添加地址需要传入一些数据以及在添加完成回到提现页面时需要做一些操作:  

```typescript  

	toAddWithdrawAddress() {
		// return this.routeTo("add-address", {
		// 	productInfo: this.productInfo,
		// });
		const selector = this.modalCtrl.create(AddAddressPage, {
			productInfo: this.productInfo,
		});
		selector.onDidDismiss(returnData => {
			this.selected_withdraw_address = returnData ? returnData : null;
			this.accountService
				.getWithdrawAddress(this.productInfo.productId)
				.then(data => {
					this.withdraw_address_list = data;
				});
		  });
		selector.present();
	}

```  

### <span id = "recharge-gateway">`./recharge-gateway`</span>  
### <span id = "withdraw-gateway">`./withdraw-gateway`</span>  

充值提现产品列表页, 展示产品列表, 提供跳转到相应产品的充值或提现页面, 功能简单不多赘述.  

### <span id = "withdraw-address-list">`./withdraw-address-list`</span>  

用户添加充值地址的页面, 简单表单页, 注意完成添加关闭页面传参即可:  

```typescript  

	finishSelect(send_selected_data?: boolean) {
		this.viewCtrl.dismiss({
			selected_data: send_selected_data
				? this.withdraw_address_list.find(
						v => v.id == this.formData.withdraw_address_id
					)
				: null,
			withdraw_address_list: this.is_withdraw_address_list_changed
				? this.withdraw_address_list
				: null
		});
	}

```  

### <span id = "recommend">`./recommend`</span>  

高交所推荐页, 几乎没有功能代码的废弃demo, 可删.  

### <span id = "register">`./register`</span>  

用户注册页面, 表单页面, 与[修改密码页面](#modify-pwd)一样, 参考一个就行.  

### <span id = "set-pay-pwd">`./set-pay-pwd`</span>  

设置密码页面, 表单页面, 与[修改密码页面](#modify-pwd)一样, 参考一个就行.  

### <span id = "search-item">`./search-item`</span>  

高交所遗留搜索股票的页面, 用到了三栏列表, [资讯新闻公告列表页](#news-list)的搜索组件更完善, 这个可删除.  

### <span id = "stock-detail">`./stock-detail`</span>  

高交所股票详情页, 功能基本全移植到币加所[交易页面](#trade-interface-v2).  

### <span id = "submit-real-info">`./submit-real-info`</span>  

实名认证页面, 表单页面, 与[修改密码页面](#modify-pwd)类似, 多了个图片上传以及图片预览. 涉及图片转换问题, 代码较多建议到文件中查看,这里贴下上传图片入口代码:    

```typescript  

    //上传图片功能入口
	@asyncCtrlGenerator.loading('图片上传中')
	@asyncCtrlGenerator.error('图片上传失败')
	async updateImage(
		fid_promise: Promise<any>,
		image: typeof SubmitRealInfoPage.prototype.images[0],
		result: any
	) {
		image.uploading = true;
		
		try {
			
			const fid = await fid_promise;
			const result_data = result.data;
			const blob = await this.minImage(result_data);
			const blob_url = URL.createObjectURL(blob);
			image.image = this.san.bypassSecurityTrustUrl(blob_url);
			const upload_res = await this.fs.uploadImage(fid, blob);
			image.fid = fid;
		}catch (e){
			//上传图片失败，展示失败图片
			console.log('图片上传失败',e)
			image.image = 'assets/images/no-record.png';
		} finally {
			image.uploading = false;
		}
	}

    //相机组件的使用:
	upload(name) {
		const imageTaker = this.imageTakerCtrl.create(name);
		const fid_promise = this.fs.getImageUploaderId(FileType.身份证图片);
		imageTaker.onDidDismiss((result, role) => {
			
			if (role !== 'cancel' && result) {
				const image = this.images.find(
					item => item.name === result.name
				);
				// console.log('index: ', index, result);
				if (result.data) {
					// 开始上传
					this.updateImage(fid_promise, image, result);

				} 
				// 隐藏没选图片的情况，这个图片提示用于图片上传失败
				// else {
				// 	image.image = 'assets/images/no-record.png';
				// }
				// console.log(this.images);
			}
			
		});
		imageTaker.present();
	}

```  

### <span id = "tabs">`./tabs`</span>  

app一级页面的tab栏, 用于切换四个一级页面的显示. `checkPersonalStockListIsNull` 函数为高交所遗留方法, 高交所需求为持仓页显示自选的股票, 如若没有则显示推荐的页面, 均为一级页面, 故做了一些处理, 现币加所则是显示所有产品用户的持仓情况, 具体参考[持仓页面](#optional).  

关于tab页的展示, 上个版本(在没有路由之前)是通过直接给root赋值导入的页面类来实现. 现在的版本由肇丰引入了路由, 故在实现上与[根文件](#app)定义的路由相关联, 不需要二次引入页面类. 

### <span id = "trade-interface">`./trade-interface`</span>  

基于高交所二次开发的第一版币加所交易页, 页面设计参考psd *毕加所app-交易*, *毕加所app-交易-快捷交易*, 功能方面参考[v2版交易页](#trade-interface-v2). 

### <span id = "trade-interface-v2">`./trade-interface-v2`</span>  

项目当前交易页, 设计参考psd *交易页面2*, *快捷交易1*. 页面逻辑较为繁多. 主要包括交易对下拉切换/ 图表显示/ 限价交易及快捷交易/ 显示当前资产/ 显示行情价格/ 显示买五卖五/ 以及当前交易对委托单等.  

用到ionic的 select组件/下拉刷新/无限滚动加载/拖动条组件, 主要改动是select元素上`style="width:initial"`用来解决元素宽度不合预期的问题, 以及一些sass样式的覆盖, 例如:  

```scss  

	ion-range{
		position: relative;
		top:-1.2rem;
		
		.range-slider .range-bar:first-child{
			border-radius: 2.5px;
			border: 1px solid color($colors,special);
			background: transparent;
		}
		&.issale .range-slider .range-bar:first-child{
			border: 1px solid color($colors,eye-catching);
		}
		
		&.issale .range-bar-active{
			background:color($colors,eye-catching);
		}
	}

    ion-popover{
        .item-ios{
            padding-right: 18px;//因为左边有2padding 故这里是16+2=18px
        }
        .label{
            margin-left: 52px !important;
            text-align: center;
        }
    }
        
    ion-popover .item.item-block .item-inner{
        border-bottom: 0.5px solid rgba(193, 177, 127, .6);
    }
    .item.item-block.item-radio-checked .item-inner .radio-inner{
        border-color: color($colors,special);
    }

    .item.item-block.item-radio-checked .item-inner .label{
        color:color($colors,special);
    }

    .alert-message a{
        color: unset;
    }
    .alert-wrapper .alert-checkbox-icon{
        background-color: unset;
    }

```  

样式覆盖主要采用检查元素实际类名后尽量通过提高优先级方式来覆盖, 不得已情况使用`!important`关键字, 使用类名优先级覆盖注意避免'ios'/'md' 这种区分设备的字符, 除非就是要更改指定一种设备的样式.  

由于我们在app启动后锁定了屏幕旋转, 这边大图表采用的是通过css动画实现的模拟横屏, 旋转的代码放在了 **app.scss** 文件中, 代码如下:  

```scss  

    .sim-landscape {
        width: 100vh;
        height: 100vw;
        position: relative;
        left: calc(50vw - 50vh);
        top: calc(50vh - 50vw);
        transform: rotate(0);
        transition: all .4s;
        opacity: 0;
        z-index: -1;

        &.show {
            transform: rotate(90deg);
            opacity: 1;
            z-index: 100;
        }
    }

```  

交易页面中, 跟输入数值有关的, 最终基本上会通过`changeByStep()`方法, 用来处理输入数值的格式化, 做了一些hack处理, ps. 后面的版本少勇引入了bignumber库更好地实现该功能, 下面代码仅供参考:  

```typescript  

    changeByStep(target: string, sign: string = '+', step?: any, precision: number = -8) {
        const invBase = Math.pow(10, -(precision));
        // 浮点数四则运算存在精度误差问题.尽量用整数运算
        // 例如 602 * 0.01 = 6.0200000000000005 ，
        // 改用 602 / 100 就可以得到正确结果。

        let length = 0
        //step参数初始化, 若没传或传的参数有问题, 会根据当前数值的小数位长度设置step
        if (isNaN(step)) {
            length = this[target].split('.')[1] ? this[target].split('.')[1].length : length
            step = Math.pow(10, -length)
            step = sign + step
        }
        //原来的方法遇到 “0.0048” *10^8,会丢失精度
        // const result = Math.max(0, Math.floor(+this[target] * invBase + step * invBase) / invBase);
        //新方法,区分价格跟数量,价格用新的，数量用旧方法
        // '11.12' -> ['11','12'] -> (11 * 10^8 * 10^(arr[1].length) + 12 * 10^8 ) / 10^8
        let result;
        if(typeof this[target] == "string" ){
            result = this[target].split('.');
            if(result.length == 2){
                result = Math.max(0, Math.floor(result[0] * invBase *  Math.pow(10,result[1].length) + result[1] * invBase + step * invBase * Math.pow(10,result[1].length)) / (invBase * Math.pow(10,result[1].length)));
            }else{
                result = Math.max(0, Math.floor(result[0] * invBase + step * invBase) / invBase);
            }
        } else {
            result = Math.max(0, Math.floor(+this[target] * invBase + step * invBase) / invBase);
        }
        //强制刷新数据hack处理
        this[target] = length ? result.toFixed(length) : result.toString()
        this.platform.raf(()=>{
            this[target] = length ? result.toFixed(length) : result.toString()//.toFixed(Math.max(0, -precision));
        })
    }

```  

### <span id = "transfer">`./transfer`</span>  

高交所遗留银行转账页面, 代码简单且老旧, 删了吧...  

### <span id = "work-order-add">`./work-order-add`</span>  

工单提交页面, 主要功能为表单提交和图片拍照上传, 上面都有提到不多做赘述.  

### <span id = "work-order-detail">`./work-order-detail`</span>  

工单详情页面, 实现工单内容显示以及类似聊天页面.

### <span id = "work-order-list">`./work-order-list`</span>  

工单列表页面, 若没有工单显示欢迎页, 有则会在欢迎页动画完成后显示工单列表.  

## <span id = "pipes">`./pipes`</span>  

该文件夹存放自定义的管道文件, 一般用于处理显示相关的问题而不会去影响到源数据, 大部分逻辑比较简单, 不多做介绍. 就挑几个说:  

### <span id ="number-unit-format">`number-unit-format`</span>  

该管道对number进行处理, 显示指定位数, 是否保留小数后0等自定义需求. 限定位数时候, 可能存在整数部分超过限定位数, 故通过循环计算位数来修改数值单位, 问题是各国计量单位有差, 这方面没做适配, 需要优化:  

```typescript  

    unitArray = ['', '万', '亿', '万亿', '亿亿'];


    let count = 0;
    const replacer = retainTailZeroAfterDigit ? /\.$/ : /(?:\.0*|(\.\d+?)0+)$/
    for ( ; value >= 1e4 && count < this.unitArray.length; count++) {
      value /= 1e4;
    }

    // Math.min() 是为了处理循环变量越界（超出数组长度）的情况。
    count = Math.min(count, this.unitArray.length - 1);

```  

### <span id ="price-conversion">`price-conversion`</span>  
### <span id ="quantity-conversion">`quantity-conversion`</span>  

这两个都是根据业务需要, 对数值进行换算以及保留指定位数小数的处理. 就几行代码...  

### <span id ="timespan-pipe">`timespan-pipe`</span>  

用来处理新闻时间的显示, 按'刚刚' '分钟' '小时' '天' '月'这样一直下去. 项目中现在貌似没用, 看着玩吧.  

## <span id = "providers">`./providers`</span>  

该文件夹存放项目的服务提供商, 项目在开发过程中尝试了各种对于**provider**的定位, 故各个文件用途会比较混乱, 着重说明现在项目中重要的:  

### <span id ="account-service">`account-service`</span>  

账号相关的数据请求服务提供商. 由肇丰完成, 基于[本能理财](#bnlc-framework)开发, 使用框架封装的请求方法. 可以作为http请求实例学习, 包括普通请求以及是否做缓存等...  

```typescript  

	getRechargeAddress(productId: string) {
		return this.fetch
			.autoCache(true)
			.get<CryptoCurrencyModel[]>(this.GET_CRYPTO_CURRENCY, {
				search: { productId },
			});
	}

```  

### <span id ="fs">`fs`</span>  

当前项目上传图片相关的逻辑. `file.service`弃用可直接删除.  

### <span id ="identification-number-checker">`identification-number-checker`</span>  

身份证号码验证的服务, 与注册页面用到的代码段一样. 由服务提供检查代码, 在项目中去引用. 将工具函数封装成服务, 减少项目中重复代码段. 代码注释说明完善, [create-account-step](#create-account-step)中有列出.  

### <span id ="app-data-service">`app-data-service`</span>  

app数据缓存服务, 使用ionic storage. 服务初始化时将服务中的`_data`变量进行初始化封装, 更改变量中的各参数的getter和setter, 实现外部简易操作缓存.  

```typescript  

  //对_data进行初始化封装
  initProperties() {
    this._dataReady = this.storage.ready();
    Object.keys(this._data).forEach(key => {
      if (this._in_storage_keys.indexOf(key) === -1) {
        var storage_set = localStorage.setItem.bind(localStorage);
        var storage_remove = localStorage.removeItem.bind(localStorage);
      } else {
        storage_set = (k, v) =>
          this.storage.ready().then(() => this.storage.set(k, JSON.stringify(v)));
        storage_remove = k =>
          this.storage.ready().then(() => this.storage.remove(k));
      }
      Object.defineProperty(this, key, {
        get: () => {
          return this._data[key];
        },
        set: value => {
          this._data[key] = value;
          if (value !== null && value !== undefined) {
            storage_set(this.APPDATASERVICE_PREIX + key, JSON.stringify(value));
          } else {
            storage_remove(this.APPDATASERVICE_PREIX + key);
          }
        },
        // 确保在 defineProperty 之后不允许再更改属性的设置。
        configurable: false,
        // 允许枚举
        enumerable: true
      });
    });
  }

```  

### <span id ="app-settings">`app-settings`</span>  

app配置项配置服务, 提供一些基础配置, 高交所遗留代码较多. 如`tradingTime`相关代码, 其他一些简单配置为币加所配置. 主要配置服务器地址, 汇率参数, 数据限制, ~~登录/模拟数据开关等~~(现在弃用). 没什么复杂逻辑, 主要是配置, 一看就懂~  

这里要提到[bnlc-framework](#bnlc-framework)框架中也有一个app-settings文件, 同样是一些app配置, 为了统一一个地方配置地址, 故在原来的**app-settings**文件中使用`getter`获取框架中配置的服务器地址, 故直接修改该文件中的即可(重构可以同一起来), 还做了便于开发的配置代码:  

```typescript  

const server_host =
  getQueryVariable('SERVER_HOST') || localStorage.getItem('SERVER_HOST') || '';
if (location.hostname === 'dev-bnlc.bnqkl.cn') {
  AppSettingProvider.SERVER_URL = 'http://dev-bnlc.bnqkl.cn:40001';
} else if (server_host.startsWith('HOME')) {
  let home_ip = location.hostname;
  if (server_host.startsWith('HOME:')) {
    home_ip = server_host.replace('HOME:', '').trim();
  }
  AppSettingProvider.SERVER_URL = `http://${home_ip}:40001`;
} else if (location.hostname === 'wzx-bnlc.bnqkl.cn' || server_host === 'WZX') {
  AppSettingProvider.SERVER_URL = 'http://192.168.16.216:40001';
}

```  

### <span id ="app-service">`app.service`</span>  

封装http请求的服务, 包含数据初始化处理以及错误处理. 同目录下另一个`http-service`实现类似封装, 为高交所遗留下来修改, 可以直接弃用. 同样[bnlc-framework](#bnlc-framework)框架中也有类似服务`app-fetch`, 基本思路类似, 多做了配置缓存代码, 重构考虑整合.  

### <span id ="image-picker-service">`image-picker-service`</span>  

将image-picker插件封装成服务以供调用, 一些简单配置不多赘述.  

### <span id ="keyboard-service">`keyboard-service`</span>  

软键盘服务, 处理软键盘弹出出现的布局问题.  

### <span id ="login-service">`login-service`</span>  

登录相关服务, 包括登录注销, 获取token等基础服务, 以及账户相关的密码修改或重置方法(常规http请求).  

### <span id ="register-service">`register-service`</span>  

注册, 常规http请求.  

### <span id ="socketio-service">`socketio-service`</span>  

websocket连接服务, 使用`socketAPIs`存储管理socket链接, `connectSocket`方法初始化ws连接, 由于深度和实时价格的sorce都是'/transition', 而且返回数据都是监听`'data'`事件, 故连接时需要配置option的`forceNew:true`.  

通过rxjs创建observable监听websocket数据流:  

```typescript  

subscribeEquity(equityCodeWithSuffix: string, api: string): Observable<any> {
    const observable = new Observable(observer => {
      // 对于所有订阅都已取消的 refCount 重新进行订阅时，
      // 这个函数会被重新调用一次，并传入新的 observer 。
      this.socketReady(api)
        //socketReady包含前面提到的connectSocket方法, 初始化连接
        .then(() => {
          //处理股票参数
          if (equityCodeWithSuffix.indexOf('-') === -1) {
            equityCodeWithSuffix = '-' + equityCodeWithSuffix
          }
          //getObservableFromMap方法中做了缓存处理,若存在api,则返回对应api,不存在则创建保存并返回.
          this.getObservableFromMap(api, `${equityCodeWithSuffix}`)
            .subscribe(observer)
          if (api == 'price' || api == 'depth') {
            this.socketAPIs.get(api).socket.emit('watch', [`${equityCodeWithSuffix}`])
          } else {
            //else 是兼容写法, 目前已经可以删去. 
            this.socketAPIs.get(api).socket.emit('watch', `${equityCodeWithSuffix}`)
          }
        })
        .catch(err => {
          // console.log(err)
        });

      // 在 multicast refCount 上的所有订阅都取消时，
      // 会调用此方法取消 observer 的订阅。
      return () => {
        // this.removeFromSocketioSubscribeList(subscribeData);
        // this._socketioSubscribeSet.delete(subscribeData);
        if (api == 'price' || api == 'depth') {
          this.socketAPIs.get(api).socket.emit('unwatch', [`${equityCodeWithSuffix}`])
        } else {
          //else 是兼容写法, 目前已经可以删去. 
          this.socketAPIs.get(api).socket.emit('unwatch', `${equityCodeWithSuffix}`)
        }
      }
    });

    return observable;
  }

```  

因为有mainproduct price(行情首页头部的三个产品实时法币价格), 以及report数据在参数上和返回数据上略有不同, 故可以看到三段相似代码, 这部分需要重构整理.  

### <span id ="stock-data-service">`stock-data-service`</span>  

股票数据相关的服务, 用于请求股票的各种相关数据, 高交所遗留代码占大多数, 还在使用的代码包括但不限于: `resetData`, `requestProducts`, `requestProductById`, `parseStockListData`, `getProduct`等,  
为了解决请求全部产品列表和请求单个产品在app初始化时重复获取的问题, 通过引入promise尝试解决:  

```typescript  

  public requestProducts(platformType: string = this.appSettings.Platform_Type): Promise<any> {

    if(this.productRequestPromise){
      return this.productRequestPromise
    }

    const path = `/product/product`
    const params = {
      instid: platformType,
    }
    return this.productRequestPromise = this.appService.request(RequestMethod.Post, path, params)
      .then(async data => {
        console.log('requestProducts: ', data)
        await this.parseStockListData(data)
        this.productRequestPromise = void 0
      })
      .catch(err => {
        console.log('requestProducts error: ', err.message || err);
        this.productRequestPromise = void 0        
        return Promise.reject(err);
      });
  }

```  

最终解决方法应该还是封装缓存与请求关系, 由于我们采用restfulapi, 故可以对请求做*在请求全体时候搁置个体请求,等全体请求到后再检查是否需要再获取个体*这种模式.  

### <span id ="trade-service">`trade-service`</span>  

获取交易对和交易相关数据服务, 关键函数`getTradeList`, 添加注释方便理解:  

```typescript  

  public getTradeList() {

    const path = `/transactionengine/traders`;
    
    const requestTime = new Date()
    const shouldUpdate = (+requestTime - this.last_time_getTradeList) > 3e2

    return this.appService
      .request(RequestMethod.Get, path, undefined)
      .then(async data => {
        console.log('getTradeList: ', data);
        const traderList = await this.appDataService.traderList;

        if (!data) {
          return Promise.reject(new Error('data missing'));
        } else if (data.error) {
          return Promise.reject(new Error(data.error));
        } else {
          await Promise.all(
            (data as any[])
            // .filter(item =>
            //   item
            // )
              .map(async ({ priceId, productId, buyFee, saleFee },index) => {
                const products = await this.appDataService.productsPromise;
                const product = await this.stockDataService.getProduct(productId)//products.get(productId);
                const price = !priceId ? undefined : await this.stockDataService.getProduct(priceId)//products.get(priceId);
                // if (product) {
                if (!traderList.has(`${priceId}-${productId}`) || shouldUpdate ){  
                  traderList.set(`${priceId}-${productId}`, {
                    traderId: `${priceId}-${productId}`,
                    traderName: !priceId ? `${product ? product.productName : '--'}` :
                      `${product ? product.productName : '--'} / ${product ? price.productName : '--'}`,
                    reportRef: new Observable(), //用来存放报表中间管道
                    reportArr: [], //用来存放图表数据
                    marketRef: new BehaviorSubject(undefined), //用来存放交易中间管道
                    buyFee,
                    saleFee,
                    priceId,
                    productId,
                    index, //由于map不保存数组的顺序,故需要这个参数保存item在数组中的顺序,由此源数据转为数组时候,不使用push方法,而是直接对每个数组项进行赋值,保证数组项顺序与后端传过来的一致
                  });
                }
                // }
              })
          )
        }
        return Promise.resolve(traderList);
      })
      .catch(err => {
        console.log('getTradeList error: ', err);
        // return Promise.reject(err);
      });
  }

```  

## <span id = "shared">`./shared`</span>  

共享模块文件夹, 通过引入和导出组件/指令等, 将功能分模块封装, app或是其他子组件只需引入共享模块即可使用模块中包含的组件, 项目模块划分的一次尝试, 用法有待商榷.  

# app构建打包  

## <span id = "version">版本发布</span>  

版本发布前需要做修改两个文件:  
1. 'config.xml'中的`version`参数, 如: `<widget id="cn.bnqkl.picasso" version="0.1.6" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0">`

1. 'about.html'中的显示, `<ion-col>币加所：v0.1.6</ion-col>`  

然后打个tag提交到远程分支.  

在公司的Macbook上打包.

# 遗留问题  

## git主要分支说明  

git上主要有三个分支:  
1. master为一开始从高交所项目复制过来提交的代码.  
1. develop分支为行情和交易页面重做前的版本, 两个页面重做完成后, 切到了v2-develop上开发.  
1. v2-develop现今的开发在该分支上执行, 主要是bug修复和需求些微调整.  

## 照相机持续优化  

相机插件在ios上表现不尽人意, 仍会有拍照卡顿, 反应时间长的问题. 估计插件也不够完善或是与Angular兼容性差, 需要持续关注.

## 一些解决方案细节  

### header加载数据后高度被撑开盖住content问题解决:  

页面元素若有数据是获取后显示, 且没有写死元素高度, 导致没数据时候元素没有高度的, 会出现header被撑高盖住content的问题, 可以调用ionic的`content.resize`方法, 手动触发页面布局的计算(注意这个很消耗性能,慎重使用).  

### 利用`ng-container`简化数据获取和订阅.  

`ng-container`可以包裹页面元素而不会在页面上多留下一层dom结构, 可以用来将`async`管道和源数据组合起来当个变量, 包裹的里面直接使用变量进行赋值, 解决每个async管道都需要订阅一次的问题.  

如:`<ng-container *ngFor="let a of [(b|async)]">{{a.name}}</ng-container>` 这种方式以试验过, 符合预期.  

[官方源码](#https://github.com/angular/angular/blob/master/packages/examples/common/ngIf/ts/module.ts#L76)提到的另一种例子实验失败, 不清楚是bug还是版本问题, 留给后面检验了:  

```typescript  

@Component({
  selector: 'ng-if-let',
  template: `
    <button (click)="nextUser()">Next User</button>
    <br>
    <div *ngIf="userObservable | async as user; else loading">
      Hello {{user.last}}, {{user.first}}!
    </div>
    <ng-template #loading let-user>Waiting... (user is {{user|json}})</ng-template>
`
})

```  

相关参考: [ngIf源码](#https://github.com/angular/angular/blob/master/packages/common/src/directives/ng_if.ts#L67)

### 合理使用通过ChangeDetectionStrategy/ChangeDetectorRef/runOutsideAngular管理脏检查提高性能.  

Angular2的脏检查机制允许一定配置, 通过减少不必要的检查减少重绘而提高性能. 相关内容较多需要一段时间学习, 算是性能优化的一个方向, 具体就不展开了...  

# 重构方案  

项目累积至今, 存在太多无用代码和耦合的逻辑, 对于新上手或持续开发都很不利, 需要花费大量时间去了解和熟悉代码, 代码过于耦合混乱也给继续开发带来很大阻碍, 故建议腾出时间重构甚至是重写代码. 重写有个好处, 不需要完全缕清现有的代码, 只需复用需要的部分, 然后进行组合即可. 重构则需要完全搞清每个文件和代码块间的联系, 工作量甚至可能会比重写大...  

## 准备  

开搞之前, 为了标准化流程以及明确app各项参数. 最好能有以下材料提供:  

### 设计稿提供主要配色, 配色数量尽量少  

为了尽量降低设计稿与app实际界面的差异, 同时提高开发效率, 希望能提供App项目设计稿的主要色号, 然后我们在**variables.scss**文件中定义颜色变量集合, 在具体页面中定义样式时, 使用变量而不是具体色号, 方便随时修改颜色主题.(现在大部分页面也是采用这种方式, 可以参考文件中的`$colors`定义, 有少部分因为用的地方比较少或者历史遗留问题, 故直接写死, 需要统一.)  

关于颜色变量名称的定义, 参考[抽像命名Sass变量](#https://www.w3cplus.com/preprocessor/better-sass-variables.html), 根据颜色名以及用处各自定义一份颜色, 在需要的地方使用合适的变量.    

## 提取app统一配置  

app中可能存在一些需要配置的变量, 方法等. 比如服务器地址, 模拟数据(币加所放弃了模拟数据的兼容, 主要通过谷歌云, 公司服务器, 自己跑服务等来获取数据.), 交易深度, 前后端数据换算比例等,  可以参考[app-settings](#app-settings)获取更多信息.  

## 提取工具函数  

工具函数包括很多包括各种表单验证, 一些与业务无关的转换之类的函数. 将工具函数集中到一个或几个**service**进行管理, 在app顶层`app.component.ts`中作为provider引入, 供各个有需要的页面调用.  

## 封装ionic提供的组件  

ionic提供了很多组件, 如`alert`,`promote`,`toast`等等的弹窗组件, 引入原生插件的`keyboard`,`imagepicker`等, 建议作为全局单例使用, 故需要封装成服务作为服务提供商, 方便做一些类似*重复弹出*的处理.  

如`toast`是一种简单的半透明文字提示框, 可以在服务中存储需要弹出的队列, 设置一定的延迟以及弹出的替换来保证每次都只弹出一个不会互相覆盖, 同时还能把所有信息呈现给用户. `alert`, `promote`类似.  

还有诸如`keyboard`,`imagepicker`这种原生插件, 建立服务对一些方法进行封装, 方便进行参数配置, 扩展方法以及必要的hack, 而组件部分专注交互和展示即可.  

## 函数优化  

函数优化不多展开, 基础的做到尽量别在多处写重复的函数, 可以提取到service中作为服务提供商. 第二, 较长的业务逻辑, 分解处理函数, 保证代码简练可读. 还有更多更深入的解决方案视情况而定. 个人认为, 代码可读性是第一目的.  

## 封装通用接口  

### http请求  

http请求的封装已经有做, 可以参考看[app.service](#app-service).  

### websocket  

socket封装参考[socketio-service](#socketio-service), 现有的封装可能还不够完善, 没有与后端授权相关的处理, 直接连上了就当成授权通过, 现有代码有授权的监听以及断线重连的说明, 详情看文件中的`connectSocket`方法内的代码和注释, 后期完善可参考:  

```typescript  

    socket.on('connect', () => {
      this._authenticated.next(true)
    });

    socket.on('connect_error', (...args) => {
      // console.log(`${api}Socket connect_error:`, args);
      // 在连接出错的情况下（后期可以考虑在连续多次出错时才进行处理），
      // 使用 disconnect() 可以保证不再做无谓的连接尝试。
      // 同时将 _authenticated 设为 undefined ，
      // 以便后续的订阅请求可以等待网络重连后重新认证的结果，
      // 并且可用于区分网络问题导致的 disconnect 与认证失败的 disconnect 。
      socket.disconnect();
      this._authenticated.next(undefined);
    });

    // connect_timeout 时，会同时触发 connect_error ，
    // 似乎不需要额外处理，因此姑且注释掉。
    // socket.on('connect_timeout', (...args) => {
    //  // console.log('connect_timeout:', args);
    //   // socket.disconnect();
    //   // if (this._authenticated.getValue() !== undefined) {
    //   //   this._authenticated.next(false);
    //   // }
    // });

    // 重连成功也会触发 connect 事件，
    // 因此似乎没有必要再重复进行后续处理。
    // socket.on('reconnect', (retryTimes) => {
    //   // reconnect 回调函数的参数只有一个，
    //   // 表示重连成功之前的重试次数。
    //  // console.log('reconnect, retryTimes: ', retryTimes);
    // });

```  

现有授权还存在另一个问题, 没有区分开不同api, 需要将授权也加入`socketAPIs`中管理可能会比较合适.  

### 关于数据缓存  

数据缓存部分, 参考现有的[数据缓存文件](#app-data-service), 建议是例如产品信息这种数据, 不需要经常更新的, 通过ionic的storage缓存到本地, 便于获取. 注意要和后端配合做过期处理等数据安全性操作(这部分项目目前没有)  

## [封装通用模块]  

将通用组件和独立的(非全局需要)的服务封装到`xx.module.ts`文件中, 而app引入使用, 便于分类管理, 目前这个项目是否需要这样做有待商榷. 是分工开发比较合适的实践.  