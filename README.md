
![](https://img2024.cnblogs.com/blog/468667/202412/468667-20241213161200409-990760759.gif)
 


【引言】


本案例将展示如何使用鸿蒙NEXT框架开发一个简单的世界时钟应用程序。该应用程序能够展示多个城市的当前时间，并支持搜索功能，方便用户快速查找所需城市的时间信息。在本文中，我们将详细介绍应用程序的实现思路，包括如何获取时区信息、更新城市时间、以及如何实现搜索高亮功能。


【环境准备】


• 操作系统：Windows 10


• 开发工具：DevEco Studio NEXT Beta1 Build Version: 5\.0\.3\.806


• 目标设备：华为Mate60 Pro


• 开发语言：ArkTS


• 框架：ArkUI


• API版本：API 12


【实现思路】


1\. 组件结构设计


我们的应用程序主要由两个核心组件构成：CityTimeInfo类和WorldClockApp组件。CityTimeInfo类用于存储每个城市的名称、当前时间和时区信息。WorldClockApp组件则负责管理城市时间列表、搜索功能以及用户界面。


2\. 获取时区信息


在应用程序启动时，我们需要获取可用的时区信息。通过调用i18n.TimeZone.getAvailableIDs()方法，我们可以获取所有可用的时区ID。接着，我们使用这些ID创建CityTimeInfo实例，并将其添加到城市时间列表中。为了确保用户能够看到当前时间，我们还添加了北京的时间信息作为默认城市。


3\. 更新时间逻辑


为了实时更新城市的当前时间，我们在updateAllCityTimes方法中实现了时间更新逻辑。通过获取系统的语言环境和相应的日历对象，我们可以根据城市的时区ID获取当前的年、月、日、时、分、秒，并将其格式化为字符串。这个方法会在页面显示时每秒调用一次，确保时间信息的准确性。


4\. 搜索功能实现


为了提升用户体验，我们实现了搜索功能，允许用户通过输入关键词来筛选城市。在highlightSearchText方法中，我们对城市名称进行分段处理，将匹配的关键词高亮显示。通过这种方式，用户可以快速找到所需的城市，并且高亮的文本能够提供更好的视觉反馈。


5\. 用户界面构建


最后，我们使用build方法构建用户界面。界面包括一个搜索框、城市名称和时间的显示区域。我们使用了Column和Row组件来布局，并通过设置样式属性来美化界面。滚动区域的实现使得用户可以方便地浏览多个城市的信息。


【完整代码】



[?](https://github.com)

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189 | `import` `{ i18n } from` `'@kit.LocalizationKit'` `// 导入国际化模块，用于处理多语言``import` `{ inputMethod } from` `'@kit.IMEKit'` `// 导入输入法模块` `@ObservedV2``// 观察者装饰器，用于观察状态变化``class` `CityTimeInfo {` `// 定义城市时间信息类``@Trace cityName: string =` `""``;` `// 城市名称，初始为空字符串``@Trace currentTime: string =` `""``;` `// 当前时间，初始为空字符串``timeZone: i18n.TimeZone;` `// 时区属性` `constructor(cityName: string, timeZone: i18n.TimeZone) {` `// 构造函数，接收城市名称和时区``this``.cityName = cityName;` `// 设置城市名称``this``.timeZone = timeZone;` `// 设置时区``}` `@Trace isVisible: boolean =` `true``;` `// 是否可见，初始为true``}` `@Entry``// 入口组件装饰器``@Component``// 组件装饰器``struct WorldClockApp {` `// 定义世界时钟应用组件``@State` `private` `searchText: string =` `''``;` `// 搜索文本，初始为空字符串``@State` `private` `cityTimeList: CityTimeInfo[] = [];` `// 城市时间信息列表，初始为空数组``private` `lineColor: string =` `"#e6e6e6"``;` `// 边框颜色``private` `titleBackgroundColor: string =` `"#f8f8f8"``;` `// 标题背景色``private` `textColor: string =` `"#333333"``;` `// 文字颜色``private` `basePadding: number = 4;` `// 内边距``private` `lineWidth: number = 2;` `// 边框宽度``private` `rowHeight: number = 50;` `// 行高``private` `ratio: number[] = [1, 1];` `// 列宽比例``private` `textSize: number = 14;` `// 基础字体大小``private` `updateIntervalId = 0;` `// 更新间隔ID` `updateAllCityTimes() {` `// 更新所有城市的时间``const locale = i18n.System.getSystemLocale();` `// 获取系统语言环境``for` `(const cityTime of` `this``.cityTimeList) {` `// 遍历城市时间列表``const timeZoneId: string = cityTime.timeZone.getID();` `// 获取时区ID``const calendar = i18n.getCalendar(locale);` `// 获取日历对象``calendar.setTimeZone(timeZoneId);` `// 设置日历的时区` `// 获取当前时间的各个部分``const year = calendar.get(``"year"``).toString().padStart(4,` `'0'``);` `// 年``const month = calendar.get(``"month"``).toString().padStart(2,` `'0'``);` `// 月``const day = calendar.get(``"date"``).toString().padStart(2,` `'0'``);` `// 日``const hour = calendar.get(``"hour_of_day"``).toString().padStart(2,` `'0'``);` `// 小时``const minute = calendar.get(``"minute"``).toString().padStart(2,` `'0'``);` `// 分钟``const second = calendar.get(``"second"``).toString().padStart(2,` `'0'``);` `// 秒` `// 更新城市的当前时间字符串``cityTime.currentTime = `${year}年${month}月${day}日 ${hour}:${minute}:${second}`;``}``}` `onPageShow(): void {` `// 页面显示时的处理``clearInterval(``this``.updateIntervalId);` `// 清除之前的定时器``this``.updateIntervalId = setInterval(() => {` `// 设置新的定时器``this``.updateAllCityTimes();` `// 每秒更新所有城市的时间``}, 1000);``}` `onPageHide(): void {` `// 页面隐藏时的处理``clearInterval(``this``.updateIntervalId);` `// 清除定时器``}` `private` `highlightSearchText(cityTime: CityTimeInfo, keyword: string) {` `// 高亮搜索文本``let` `text = cityTime.cityName` `// 获取城市名称` `if` `(!keyword) {` `// 如果没有关键词``cityTime.isVisible =` `true` `// 设置城市可见``return` `[text]` `// 返回城市名称``}``let` `segments: string[] = [];` `// 存储分段文本``let` `lastMatchEnd: number = 0;` `// 上一个匹配结束的位置``while` `(``true``) {` `// 循环查找关键词``const matchIndex = text.indexOf(keyword, lastMatchEnd);` `// 查找关键词位置``if` `(matchIndex === -1) {` `// 如果没有找到``segments.push(text.slice(lastMatchEnd));` `// 添加剩余文本``break``;` `// 退出循环``}` `else` `{``segments.push(text.slice(lastMatchEnd, matchIndex));` `// 添加匹配前的文本``segments.push(text.slice(matchIndex, matchIndex + keyword.length));` `// 添加匹配的关键词``lastMatchEnd = matchIndex + keyword.length;` `// 更新最后匹配结束位置``}``}``cityTime.isVisible = (segments.indexOf(keyword) != -1)` `// 设置城市可见性``return` `segments;` `// 返回分段文本``}` `aboutToAppear() {` `// 组件即将出现时的处理``const timeZoneIds: Array = i18n.TimeZone.getAvailableIDs();` `// 获取可用时区ID列表` `this``.cityTimeList.push(``new` `CityTimeInfo(``'北京 (中国)'``, i18n.getTimeZone()));` `// 添加北京的城市时间信息` `for` `(const id of timeZoneIds) {` `// 遍历时区ID``const cityDisplayName = i18n.TimeZone.getCityDisplayName(id.split(``'/'``)[1],` `"zh-CN"``);` `// 获取城市显示名称``if` `(cityDisplayName) {` `// 如果城市名称存在``this``.cityTimeList.push(``new` `CityTimeInfo(cityDisplayName, i18n.getTimeZone(id)));` `// 添加城市时间信息``}``}` `this``.updateAllCityTimes();` `// 更新所有城市的时间``}` `build() {` `// 构建组件的UI``Column({ space: 0 }) {` `// 创建一个垂直列``Search({ value: $$``this``.searchText })``// 创建搜索框``.margin(``this``.basePadding)``// 设置边距``.fontFeature(``"\"ss01\" on"``)` `// 设置字体特性``Column() {` `// 创建一个列``Row() {` `// 创建一行``Text(``'城市'``)``// 显示“城市”文本``.height(``'100%'``)``// 高度占满``.layoutWeight(``this``.ratio[0])``// 设置布局权重``.textAlign(TextAlign.Center)``// 文本居中``.fontSize(``this``.textSize)``// 设置字体大小``.fontWeight(600)``// 设置字体粗细``.fontColor(``this``.textColor)` `// 设置字体颜色``Line().height(``'100%'``).width(``this``.lineWidth).backgroundColor(``this``.lineColor)` `// 创建分隔线``Text(``'时间'``)``// 显示“时间”文本``.height(``'100%'``)``// 高度占满``.layoutWeight(``this``.ratio[1])``// 设置布局权重``.textAlign(TextAlign.Center)``// 文本居中``.fontSize(``this``.textSize)``// 设置字体大小``.fontWeight(600)``// 设置字体粗细``.fontColor(``this``.textColor)` `// 设置字体颜色``}.height(``this``.rowHeight).borderWidth(``this``.lineWidth).borderColor(``this``.lineColor)` `// 设置行高和边框``.backgroundColor(``this``.titleBackgroundColor)` `// 设置背景色``}.width(`100%`).padding({ left:` `this``.basePadding, right:` `this``.basePadding })` `// 设置列宽和内边距` `Scroll() {` `// 创建可滚动区域``Column() {` `// 创建一个列``ForEach(``this``.cityTimeList, (item: CityTimeInfo) => {` `// 遍历城市时间列表``Row() {` `// 创建一行``Text() {` `// 创建文本``ForEach(``this``.highlightSearchText(item,` `this``.searchText), (segment: string, index: number) => {` `// 高亮搜索文本``ContainerSpan() {` `// 创建容器``Span(segment)``// 创建文本段``.fontColor(segment ===` `this``.searchText ? Color.White : Color.Black)``// 设置字体颜色``.onClick(() => {` `// 点击事件``console.info(`高亮文本被点击：${segment}`);` `// 输出点击的文本``console.info(`点击索引：${index}`);` `// 输出点击的索引``});``}.textBackgroundStyle({``// 设置文本背景样式``color: segment ===` `this``.searchText ? Color.Red : Color.Transparent` `// 根据是否匹配设置背景色``});``});``}``.height(``'100%'``)` `// 高度占满``.layoutWeight(``this``.ratio[0])` `// 设置布局权重``.textAlign(TextAlign.Center)` `// 文本居中``.fontSize(``this``.textSize)` `// 设置字体大小``.fontColor(``this``.textColor)` `// 设置字体颜色` `Line().height(``'100%'``).width(``this``.lineWidth).backgroundColor(``this``.lineColor)` `// 创建分隔线``Text(item.currentTime)``// 显示当前时间``.height(``'100%'``)``// 高度占满``.layoutWeight(``this``.ratio[1])``// 设置布局权重``.textAlign(TextAlign.Center)``// 文本居中``.fontSize(``this``.textSize)``// 设置字体大小``.fontColor(``this``.textColor)` `// 设置字体颜色``}``.height(``this``.rowHeight)` `// 设置行高``.borderWidth({ left:` `this``.lineWidth, right:` `this``.lineWidth, bottom:` `this``.lineWidth })` `// 设置边框宽度``.borderColor(``this``.lineColor)` `// 设置边框颜色``.visibility(item.isVisible ? Visibility.Visible : Visibility.None)` `// 根据可见性设置显示状态``})` `}.width(`100%`).padding({ left:` `this``.basePadding, right:` `this``.basePadding })` `// 设置宽度和内边距``}``.width(``'100%'``)` `// 设置宽度占满``.layoutWeight(1)` `// 设置布局权重``.align(Alignment.Top)` `// 对齐方式``.onScrollStart(() => {` `// 滚动开始事件``this``.onPageHide()` `// 页面隐藏处理``})``.onScrollStop(() => {` `// 滚动停止事件``this``.onPageShow()` `// 页面显示处理``})``.onTouch((event) => {` `// 触摸事件``if` `(event.type == TouchType.Down) {` `// 如果是按下事件``inputMethod.getController().stopInputSession()` `// 停止输入会话``}``})``}``}``}` |
| --- | --- |



　　


 本博客参考[wgetcloud全球加速器服务](https://wgetcloud6.org)。转载请注明出处！
