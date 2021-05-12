# link-share 开发者文档

link 唤起技术和追踪模块 api 使用说明

为提高国内访问速度，以下仓库实时同步
* `gitee` https://gitee.com/umengplus/sharelink-jssdk-demo
* `github` https://github.com/umeng/sharelink-jssdk-demo
---

# 1.快速开始

通过 cdn 引入在 script 标签中使用

## 新版 api 示例

```html
<button id="xxl">点我唤起</button>
<script src="https://g.alicdn.com/jssdk/u-link/index.min.js"></script>
<script>
  ULink([
    {
      id: "linkidxxx",
      data: {
        a: 4,
        b: "xx",
      },
      selector: "#xxl",
    },
  ]);
</script>
```

## 旧版 api 示例

```html
<script src="https://g.alicdn.com/jssdk/u-link/index.min.js"></script>
<script>
  ULink.start({
    id: "xxx",
    data: {
      a: 4,
      b: "xx",
    },
  }).ready(function (ctx) {
    console.log(ctx.solution); // --> ISolutions;
    ctx.wakeup();
  });
</script>
```


# 2. API 文档

## 2.1 ULink

UMD 模块封装顶级对象

```typescript
/**
 * Ulink配置
 */
interface LinkConfig {
  id: string; // 必填参数,后台生成的linkid
  data?: object; // 自定义参数例如{a:1,b:2} 换起应用时会携带过去并映射成a=1&b=2
}
/**
 * 配置下发回调
 */
type ReadyCallback = {
  (ctx: LinkInstance): void;
};
/**
 * 配置下发内容
 */
interface ISolutions {
  wakeupUrl: string; // 唤起地址
  type: "scheme" | "universalLink"; // 唤起类型
  downloadUrl: string; // 下载地址
  appkey: string; // 对应appkey
  clipboardToken?:string; // 友盟后台开启剪切板能力时返回此字段，开启后可提高带参安装匹配成功率。
}
interface IWakeup {
  action?: "" | "load" | "click"; // 设置统计上报的唤起方式
  proxyOpenDownload?: IProxyOpenDownload; // 代理打开下载提示行为
  beforeOpenDownload?: ICallback;
  afterOpenDownload?: ICallback;
  timeout?: number; //触发弹窗等待超时时间单位毫秒，默认200毫秒，安卓中微信强制为0
}
type ICallback = {
  (ctx: LinkInstance): void;
};
type defaultActionCallback = {
  (extdata?: object): void;
};
type IProxyOpenDownload = {
  (defaultAction: defaultActionCallback, ctx: LinkInstance): any; // 如仍需执行默认弹窗行为可调用defaultActionCallback
};
/**
 * Ulink实例
 */
interface LinkInstance {
  ready(callback: ReadyCallback): void; // 配置下发
  wakeup(config: IWakeup): LinkInstance; // 唤起
  solution: ISolutions; // 配置下发内容
}
declare namespace ulink {
  /**
   * sdk版本
   */
  export const version: string;
  /**
   * 创建Link实例
   * @param config 实例配置
   */
  export function start(config: LinkConfig): LinkInstance;
  export interface tracker {}
}
type ProxyOpenInBrowerTips = {
  (): string;
};
type ProxyShowLoading = {
  (): void;
};
type ProxySHideLoading = {
  (): void;
};
/**
 * 自由拼接待写入剪切板内容，入参为服务端下发的token，返回值将被写入剪切板
 */
type SetClipboardText = {
  (clipboardToken:string): string;
}
type LinkOption = {
  id: string; // 必填参数,后台生成的linkid
  selector?: string; // 需要点击唤起的元素选择器（采用事件代理模式，不必等元素创建后绑定），示例 '#idxx,#idxxx',参考文档https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model/Locating_DOM_elements_using_selectors
  data?: object; // 自定义参数例如{a:1,b:2} 换起应用时会携带过去并映射成a=1&b=2
  proxyOpenDownload?: IProxyOpenDownload; // 自定义打开下载提示行为
  timeout?: number; // 触发弹窗等待超时时间单位毫秒，默认200毫秒，安卓中微信强制为0
  auto?: boolean; // 是否自动唤起,默认false,配置下发后不自动唤起应用(特别注意，部分web容器会限制自动唤起)
  lazy?: boolean; // 是否将配置下发延迟到点击时下发，默认false，如果需延迟到点击时下发配置应设置为true
  useOpenInBrowerTips?: string | ProxyOpenInBrowerTips; // 是否在微信和qq中使在浏览器中打开的提示，当值为string类型时，默认'default',值为function时，需要该函数返回蒙层html片段。
  useLoading?: string | [ProxyShowLoading, ProxySHideLoading]; // 即将支持 当值为string类型时，默认'default',启用自带loading,当值为数值时，数组第一个函数触发唤起时触发，第二个函数关闭loading时触发
  onready?:ReadyCallback; // 配置下发后触发
  useClipboard?:boolean | SetClipboardText; //默认 true, true 代表在唤起时用clipboardToken覆盖剪切板内容;false代表不会覆盖剪切板内容，开发者可以在onready后获取配置下发的token;如果是一个function，则将function返回的string写到剪切板，function的入参是配置下发的clipboardToken，当且仅当开发者需要自定义剪切板内容时使用。特别注意，若服务端剪切板功能关闭，则此配置完全失效。（2021.04.23上架生效）
};
declare class ulink {
  /**
   * ulink新版初始化函数
   * @param option 初始化参数
   */
  constructor(option: LinkOption);
  /**
   * ulink新版多参数初始化函数
   * @param options 多个linkid初始化参数
   */
  constructor(options: Array<LinkOption>);
}
export as namespace ULink;
export = ulink;

```

### start

初始化 Ulink 实例

### version

sdk 版本号 x.x.x

### tracker(追踪对象)

追踪器

#### setMetaInfo

设置追踪信息

```typescript
/** 设置追踪信息**/
export function setMetaInfo(MetaInfo: MetaInfo): MetaInfo;
export interface MetaInfo {
  /** openid */
  oid: string;
  /** 追踪码对应的urlkey */
  trackkey?: string;
  /** 来源追踪码 */
  trackcode?: string;
  /** 原始追踪码 */
  root_track_code?: string;
  /** 原始来源key */
  root_track_key?: string;
  /** 当前页面地址 */
  trackurl?: string;
  /** unionid */
  uid?: string;
  /** 昵称 */
  nickname?: string;
  /** 头像 */
  avator?: string;
}
```

#### getMetaInfo

获取设置的追踪信息

```typescript
/** 获取公共信息n */
export function getMetaInfo(): MetaInfo;
```

#### getNextTrackCode

获取追踪码

```typescript
/** 获取追踪码 */
export function getNextTrackCode(callback?: NtcCallback): void;
type NtcCallback = {
  (ntc: string): void; //新的追踪码
};
```

#### trackShare

发送分享事件

```typescript
/** 发送分享事件 */
export function trackShare(data: Shareinfo): void;
export interface Shareinfo {
  /** 分享标题 */
  title?: string;
  /** 分享描述 */
  desc?: string;
  /** 分享图片 */
  imgUrl?: string;
  /** 分享链接 */
  link: string;
  /** 分享场景 */
  scene?: string;
}
```

#### enter

发送 pv 事件

```typescript
/** pv事件 */
export function enter(config: PageConfig, callback?: Function): void;
interface PageConfig {
  /** 页面地址 */
  page: string;
  /** 页面名称 */
  page_name?: string;
}
```

# Live DEMO 演示程序

[https://share.umeng.com/demo/ulink/index.html](https://share.umeng.com/demo/ulink/index.html)

# 代码示例

## 1. 自定义打开下载提示行为
[演示demo](https://share.umeng.com/demo/ulink/index.html)
该代码效果为如未安装则直接跳转下载地址

```html
<button id="xxl">点我唤起</button>
<script src="https://g.alicdn.com/jssdk/u-link/index.min.js"></script>
<script>
  ULink([
    {
      id: "linkidxxx",
      data: {
        a: 4,
        b: "xx",
      },
      selector: "#xxl",
      
      useOpenInBrowerTips: "default",
      proxyOpenDownload: function (defaultAction, LinkInstance){
        if (LinkInstance.solution.type === "scheme"){
          // qq或者微信环境特殊处理下
          if (ULink.isWechat || ULink.isQQ) {
            // 在qq或者微信环境执行内置逻辑，具体内置逻辑为:当设置了useOpenInBrowerTips字段时，qq&&微信&&scheme时，启用蒙层提示去浏览器打开
            defaultAction();
          }else{
            window.location.href = LinkInstance.solution.downloadUrl;
          }
        }else if(LinkInstance.solution.type === "universalLink"){
          // universalLink 唤起应当由服务端提供一个带重定向到appstore的universallink地址。因此，此处不应写逻辑，友盟会在近期上线universalLink 重定向功能。
        }
      },
    },
  ]);
</script>
```

## 2. 自定义去浏览器中打开提示蒙层
在微信中打开
[体验在线demo](https://share.umeng.com/demo/ulink/index.html?tip=function)

```html
<button id="xxl">点我唤起</button>
<script src="https://g.alicdn.com/jssdk/u-link/index.min.js"></script>
<script>
  ULink([{
    id: "linkidxxx",
    data: {
      a:4,
      b:'xx'
    },
    selector:"#xxl",
    useOpenInBrowerTips:function(ctx){
      return `<div style="position:fixed;left:0;top:0;background:rgba(255,0,255,0.5);width:100%;height:100%;z-index:19910324;"></div>`；
    }
  }]);
</script>
```
## 3.自定义剪切板内容
特别注意，要去服务端开启剪切板功能
[体验在线demo](https://share.umeng.com/demo/ulink/index.html?clp=function)

```html
<button id="xxl">点我唤起</button>
<script src="https://g.alicdn.com/jssdk/u-link/index.min.js"></script>
<script>
  ULink([{
    id: "linkidxxx",
    data: {
      a:4,
      b:'xx'
    },
    selector:"#xxl",
    useClipboard:function(clipboardToken){
      // 如果H5以前用到了剪切板，且剪切板内容是 scheme://xxx/xx?key=value这种类型的,考虑到app升级，可以参考这种拼接方式`scheme://xxx/xx?key=value&um_clp=${clipboardToken}`
      return '用户自定义内容' + clipboardToken;
    }
  }]);
</script>
```

