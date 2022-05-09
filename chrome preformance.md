DCL, L
DCL(DOMContentLoaded) 表示HTML加载完成事件， L(onLoad) 表示页面所有资源加载完成事件
Navigation Timing API
window.performance.timing属性获取这些事件的具体时间戳，进而对页面性能进行分析
![avatar](https://github.com/freezestanley/rollstone/blob/main/preformance.png)

# FP, FCP, FMP, LCP
FP(First Paint): 页面在导航后首次呈现出不同于导航前内容的时间点。
FCP(First Contentful Paint): 首次绘制任何文本，图像，非空白canvas或SVG的时间点
window.performance.getEntriesByType('paint') 获取两个时间点的值

# FMP & LCP
FMP(First Meaningful Paint): 首次绘制页面“主要内容”的时间点。

CLS(Cumulative Layout Shift): 表示用户经历的意外 layout 偏移的频率

TBT(Total Blocking Time): 表示从 FCP 到 TTI 之间，所有 long task 的阻塞时间之和

1、首屏时间 VS 白屏时间
这两个完全不同的概念，白屏时间是小于首屏时间的 白屏时间：首次渲染时间，指页面出现第一个文字或图像所花费的时间

2、为什么 performance 直接拿不到首屏时间

随着 Vue 和 React 等前端框架盛行，Performance 已无法准确的监控到页面的首屏时间
因为 DOMContentLoaded 的值只能表示空白页（当前页面 body 标签里面没有内容）加载花费的时间
浏览器需要先加载 JS , 然后再通过 JS 来渲染页面内容，这个时候单页面类型首屏才算渲染完成

window.performance.getEntriesByType('navigation')[0].startTime，即开始记录性能时间

初始化 MutationObserver 监听

# LCP 最大內容繪製
計算網頁可視區域(viewport)中最大的內容元件載入所需要的時間。換句話說，這項指標標記了頁面上的主要內容被使用者看到所花的時間，相當於網頁給人的第一印象 2.5秒时标准

# FP, FCP, FMP, LCP
FP(First Paint): 页面在导航后首次呈现出不同于导航前内容的时间点。
FCP(First Contentful Paint): 首次绘制任何文本，图像，非空白canvas或SVG的时间点
window.performance.getEntriesByType('paint') 获取两个时间点的值

# FMP & LCP
FMP(First Meaningful Paint): 首次绘制页面“主要内容”的时间点。

CLS(Cumulative Layout Shift): 表示用户经历的意外 layout 偏移的频率

TBT(Total Blocking Time): 表示从 FCP 到 TTI 之间，所有 long task 的阻塞时间之和

# FID
計算使用者第一次與網頁互動(點擊連結、點開下拉選單、填表格…etc)時的延遲時間。這項指標代表了網頁的回應性，在使用者嘗試與網頁互動時是否能馬上回應

瀏覽器會因為主線程(main thread)正在忙碌中而沒辦法即時回應用戶的輸入(user input)，導致使用者可能點了一個按鈕，卻在一些延遲後才出現預期的效

TTI(Time to Interactive)

# CLS，Cumulative Layout Shift或「累計版面配置轉移」，計算了網頁元件在載入時的非預期移位

<link rel="preconnect" href="https://example.com">
<link rel="dns-prefetch" href="https://example.com">
preconnect一般只配置一个，dns-prefetch可以配置多个，所以一般把最关键的资源配置成preconnect，比如js或者css所在的cdn域名

在浏览器渲染任何内容之前，需要解析html，并生成dom树。html解析器会被任何样式文件 <link rel="stylesheet"> 以及同步脚本阻塞并暂停解析 