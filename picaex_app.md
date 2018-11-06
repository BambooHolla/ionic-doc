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

### <span id = "charts">`./bar | ./cnadlestick | ./ distanceline | ./liquid | ./pie | ./realtime-charts | ./smoothline | ./volumn `</span>  

这些组件都是对echarts的各类图标配置的封装, 项目中暂时没有用到, 可以用来作为参考, 建议重构时把这些组件删除.

### <span id = "rich-text">`./rich-text`</span>  

该组件是一个富文本展示组件,用于新闻公告的详情页. 为保持后端传来的富文本格式, 需要注意用了`<div class="content" [innerHTML]="dataSource?.content"></div>`直接加载服务端返回文本, 故不要饮用外部文本, 或是补充添加数据安全处理.  

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

`我的`页面, 四个一级页面之一. 需要登陆后才能查看. 个人信息相关的入口, 具体看app. 这里为了处理数据加载后页面header盖住content的布局问题. 调用`this.content.resize();`来手动触发content布局调整.  

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

登陆页面. 页面本身逻辑较为简单. 根据目前项目中的登陆逻辑, 都是在指定情况下弹出登陆框, 所以使用ionic的events订阅通知模式:  

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

    //其他地方通知弹出登陆界面
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

# app构建打包  

## <span id = "version">版本发布</span>  

# 遗留问题  

## git主要分支说明  

## 照相机持续优化

# 重构方案  

## 提取工具函数  

## 封装通用接口  

## [封装通用模块]  

### 利用`ng-container`简化数据获取和订阅.  