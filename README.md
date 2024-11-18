
![](https://img2024.cnblogs.com/blog/468667/202411/468667-20241117051703893-956368840.gif)
 


【引言】（完整代码在最后面）


本文将通过一个简单的计数器应用案例，介绍如何利用鸿蒙NEXT的特性开发高效、美观的应用程序。我们将涵盖计数器的基本功能实现、用户界面设计、数据持久化及动画效果的添加。


【环境准备】


电脑系统：windows 10


开发工具：DevEco Studio 5\.0\.1 Beta3 Build Version: 5\.0\.5\.200


工程版本：API 13


真机：Mate60 Pro


语言：ArkTS、ArkUI


【项目概述】


本项目旨在创建一个多计数器应用，用户可以自由地添加、编辑、重置和删除计数器。每个计数器具有独立的名称、当前值、增加步长和减少步长。应用还包括总计数的显示，以便用户快速了解所有计数器的总和。


【功能实现】


1、计数器模型


首先，我们定义了一个CounterItem类来表示单个计数器，其中包含了计数器的基本属性和行为。



[?](https://github.com)

| 12345678910111213 | `@ObservedV2``class` `CounterItem {``id: number = ++Index.counterId;``@Trace name: string;``@Trace count: number = 0;``@Trace scale: ScaleOptions = { x: 1, y: 1 };``upStep: number = 1;``downStep: number = 1;` `constructor(name: string) {``this``.name = name;``}``}` |
| --- | --- |



2、应用入口与状态管理


应用的主入口组件Index负责管理计数器列表、总计数、以及UI的状态。这里使用了@State和@Watch装饰器来监控状态的变化。



[?](https://github.com)

| 12345678910111213141516 | `@Entry``@Component``struct Index {``static` `counterStorageKey: string =` `"counterStorageKey"``;``static` `counterId: number = 0;` `@State listSpacing: number = 20;``@State listItemHeight: number = 120;``@State baseFontSize: number = 60;``@State @Watch(``'updateTotalCount'``) counters: CounterItem[] = [];``@State totalCount: number = 0;``@State isSheetVisible: boolean =` `false``;``@State selectedIndex: number = 0;` `// ...其他方法``}` |
| --- | --- |



3、数据持久化


为了保证数据在应用重启后仍然可用，我们使用了preferences模块来同步地读取和写入数据。



[?](https://github.com)

| 1234567891011 | `saveDataToLocal() {``const saveData: object[] =` `this``.counters.map(counter => ({``count: counter.count,``name: counter.name,``upStep: counter.upStep,``downStep: counter.downStep,``}));` `this``.dataPreferences?.putSync(Index.counterStorageKey, JSON.stringify(saveData));``this``.dataPreferences?.flush();``}` |
| --- | --- |



4、用户界面


用户界面的设计采用了现代简洁的风格，主要由顶部的总计数显示区、中间的计数器列表区和底部的操作按钮组成。列表项支持左右滑动以显示重置和删除按钮。



[?](https://github.com):[西部世界官网](https://tianchuang88.com)

| 123456789101112131415 | `@Builder``itemStart(index: number) {``Row() {``Text(``'重置'``).fontColor(Color.White).fontSize(``'40lpx'``).textAlign(TextAlign.Center).width(``'180lpx'``);``}``.height(``'100%'``)``.backgroundColor(Color.Orange)``.justifyContent(FlexAlign.SpaceEvenly)``.borderRadius({ topLeft: 10, bottomLeft: 10 })``.onClick(() => {``this``.counters[index].count = 0;``this``.updateTotalCount();``this``.listScroller.closeAllSwipeActions();``});``}` |
| --- | --- |



5、动画效果


当用户添加新的计数器时，通过动画效果让新计数器逐渐放大至正常尺寸，提升用户体验。



[?](https://github.com)

| 123456789101112 | `this``.counters.unshift(``new` `CounterItem(`新计数项${Index.counterId}`));``this``.listScroller.scrollTo({ xOffset: 0, yOffset: 0 });``this``.counters[0].scale = { x: 0.8, y: 0.8 };` `animateTo({``duration: 1000,``curve: curves.springCurve(0, 10, 80, 10),``iterations: 1,``onFinish: () => {}``}, () => {``this``.counters[0].scale = { x: 1, y: 1 };``});` |
| --- | --- |



【总结】


通过上述步骤，我们成功地构建了一个具备基本功能的计数器应用。在这个过程中，我们不仅学习了如何使用鸿蒙NEXT提供的各种API，还掌握了如何结合动画、数据持久化等技术点来优化用户体验。希望本文能为你的鸿蒙开发之旅提供一些帮助和灵感！


【完整代码】



[?](https://github.com)

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232233234235236237238239240241242243244245246247248249250251252253254255256257258259260261262263264265266267268269270271272273274275276277278279280281282283284285286287288289290291292293294295296297298299300301302303304305306307308309 | `import` `{ curves, promptAction } from` `'@kit.ArkUI'` `// 导入动画曲线和提示操作``import` `{ preferences } from` `'@kit.ArkData'` `// 导入偏好设置模块` `@ObservedV2``// 观察者装饰器，监控状态变化``class` `CounterItem {``id: number = ++Index.counterId` `// 计数器ID，自动递增``@Trace name: string` `// 计数器名称``@Trace count: number = 0` `// 计数器当前值，初始为0``@Trace scale: ScaleOptions = { x: 1, y: 1 }` `// 计数器缩放比例，初始为1``upStep: number = 1` `// 增加步长，初始为1``downStep: number = 1` `// 减少步长，初始为1` `constructor(name: string) {` `// 构造函数，初始化计数器名称``this``.name = name``}``}` `@Entry``// 入口组件装饰器``@Component``// 组件装饰器``struct Index {``static` `counterStorageKey: string =` `"counterStorageKey"` `// 存储计数器数据的键``static` `counterId: number = 0` `// 静态计数器ID``@State listSpacing: number = 20` `// 列表项间距``@State listItemHeight: number = 120` `// 列表项高度``@State baseFontSize: number = 60` `// 基础字体大小``@State @Watch(``'updateTotalCount'``) counters: CounterItem[] = []` `// 计数器数组，监控总计数更新``@State totalCount: number = 0` `// 总计数``@State isSheetVisible: boolean =` `false` `// 控制底部弹出表单的可见性``@State selectedIndex: number = 0` `// 当前选中的计数器索引``listScroller: ListScroller =` `new` `ListScroller()` `// 列表滚动器实例``dataPreferences: preferences.Preferences | undefined = undefined` `// 偏好设置实例` `updateTotalCount() {` `// 更新总计数的方法``let` `total = 0;` `// 初始化总计数``for` `(``let` `i = 0; i <` `this``.counters.length; i++) {` `// 遍历计数器数组``total +=` `this``.counters[i].count` `// 累加每个计数器的count值``}``this``.totalCount = total` `// 更新总计数``this``.saveDataToLocal()` `// 保存数据到本地``}` `saveDataToLocal() {` `// 保存计数器数据到本地的方法``const saveData: object[] = []` `// 初始化保存数据的数组``for` `(``let` `i = 0; i <` `this``.counters.length; i++) {` `// 遍历计数器数组``let` `counter: CounterItem =` `this``.counters[i]` `// 获取当前计数器``saveData.push(Object({``// 将计数器数据添加到保存数组``count: counter.count,``name: counter.name,``upStep: counter.upStep,``downStep: counter.downStep,``}))``}``this``.dataPreferences?.putSync(Index.counterStorageKey, JSON.stringify(saveData))` `// 将数据保存到偏好设置``this``.dataPreferences?.flush()` `// 刷新偏好设置``}` `@Builder``// 构建器装饰器``itemStart(index: number) {` `// 列表项左侧的重置按钮``Row() {``Text(``'重置'``).fontColor(Color.White).fontSize(``'40lpx'``)``// 显示“重置”文本``.textAlign(TextAlign.Center)``// 文本居中``.width(``'180lpx'``)` `// 设置宽度``}``.height(``'100%'``)` `// 设置高度``.backgroundColor(Color.Orange)` `// 设置背景颜色``.justifyContent(FlexAlign.SpaceEvenly)` `// 设置内容均匀分布``.borderRadius({ topLeft: 10, bottomLeft: 10 })` `// 设置圆角``.onClick(() => {` `// 点击事件``this``.counters[index].count = 0` `// 重置计数器的count为0``this``.updateTotalCount()` `// 更新总计数``this``.listScroller.closeAllSwipeActions()` `// 关闭所有滑动操作``})``}` `@Builder``// 构建器装饰器``itemEnd(index: number) {` `// 列表项右侧的删除按钮``Row() {``Text(``'删除'``).fontColor(Color.White).fontSize(``'40lpx'``)``// 显示“删除”文本``.textAlign(TextAlign.Center)``// 文本居中``.width(``'180lpx'``)` `// 设置宽度``}``.height(``'100%'``)` `// 设置高度``.backgroundColor(Color.Red)` `// 设置背景颜色``.justifyContent(FlexAlign.SpaceEvenly)` `// 设置内容均匀分布``.borderRadius({ topRight: 10, bottomRight: 10 })` `// 设置圆角``.onClick(() => {` `// 点击事件``this``.counters.splice(index, 1)` `// 从数组中删除计数器``this``.listScroller.closeAllSwipeActions()` `// 关闭所有滑动操作``promptAction.showToast({``// 显示删除成功的提示``message:` `'删除成功'``,``duration: 2000,``bottom:` `'400lpx'``});``})``}` `aboutToAppear(): void {` `// 组件即将出现时调用``const options: preferences.Options = { name: Index.counterStorageKey };` `// 获取偏好设置选项``this``.dataPreferences = preferences.getPreferencesSync(getContext(), options);` `// 同步获取偏好设置``const savedData: string =` `this``.dataPreferences.getSync(Index.counterStorageKey,` `"[]"``) as string` `// 获取保存的数据``const parsedData: Array = JSON.parse(savedData) as Array` `// 解析数据``console.info(`parsedData:${JSON.stringify(parsedData)}`)` `// 打印解析后的数据``for` `(const item of parsedData) {` `// 遍历解析后的数据``const newItem =` `new` `CounterItem(item.name)` `// 创建新的计数器实例``newItem.count = item.count` `// 设置计数器的count``newItem.upStep = item.upStep` `// 设置计数器的upStep``newItem.downStep = item.downStep` `// 设置计数器的downStep``this``.counters.push(newItem)` `// 将新计数器添加到数组``}``this``.updateTotalCount()` `// 更新总计数``}` `build() {` `// 构建组件的UI``Column() {``Text(``'计数器'``)``// 显示标题``.width(``'100%'``)``// 设置宽度``.height(``'88lpx'``)``// 设置高度``.fontSize(``'38lpx'``)``// 设置字体大小``.backgroundColor(Color.White)``// 设置背景颜色``.textAlign(TextAlign.Center)` `// 文本居中``Column() {``List({ space:` `this``.listSpacing, scroller:` `this``.listScroller }) {` `// 创建列表``ForEach(``this``.counters, (counter: CounterItem, index: number) => {` `// 遍历计数器数组``ListItem() {` `// 列表项``Row() {` `// 行布局``Stack() {` `// 堆叠布局``Rect().fill(``"#65DACC"``).width(`${``this``.baseFontSize / 2}lpx`).height(``'4lpx'``)` `// 上方横条``Circle()``// 圆形按钮``.width(`${``this``.baseFontSize}lpx`)``.height(`${``this``.baseFontSize}lpx`)``.fillOpacity(0)``// 透明填充``.borderWidth(``'4lpx'``)``// 边框宽度``.borderRadius(``'50%'``)``// 圆角``.borderColor(``"#65DACC"``)` `// 边框颜色``}``.width(`${``this``.baseFontSize * 2}lpx`)` `// 设置宽度``.height(`100%`)` `// 设置高度``.clickEffect({ scale: 0.6, level: ClickEffectLevel.LIGHT })` `// 点击效果``.onClick(() => {` `// 点击事件``counter.count -= counter.downStep` `// 减少计数器的count``this``.updateTotalCount()` `// 更新总计数``})` `Stack() {` `// 堆叠布局``Text(counter.name)``// 显示计数器名称``.fontSize(`${``this``.baseFontSize / 2}lpx`)``// 设置字体大小``.fontColor(Color.Gray)``// 设置字体颜色``.margin({ bottom: `${``this``.baseFontSize * 2}lpx` })` `// 设置底部边距``Text(`${counter.count}`)``// 显示计数器当前值``.fontColor(Color.Black)``// 设置字体颜色``.fontSize(`${``this``.baseFontSize}lpx`)` `// 设置字体大小``}.height(``'100%'``)` `// 设置高度``Stack() {` `// 堆叠布局``Rect().fill(``"#65DACC"``).width(`${``this``.baseFontSize / 2}lpx`).height(``'4lpx'``)` `// 下方横条``Rect()``.fill(``"#65DACC"``)``.width(`${``this``.baseFontSize / 2}lpx`)``.height(``'4lpx'``)``.rotate({ angle: 90 })` `// 垂直横条``Circle()``// 圆形按钮``.width(`${``this``.baseFontSize}lpx`)``// 设置宽度``.height(`${``this``.baseFontSize}lpx`)``// 设置高度``.fillOpacity(0)``// 透明填充``.borderWidth(``'4lpx'``)``// 边框宽度``.borderRadius(``'50%'``)``// 圆角``.borderColor(``"#65DACC"``)` `// 边框颜色``}``.width(`${``this``.baseFontSize * 2}lpx`)` `// 设置堆叠布局宽度``.height(`100%`)` `// 设置堆叠布局高度``.clickEffect({ scale: 0.6, level: ClickEffectLevel.LIGHT })` `// 点击效果``.onClick(() => {` `// 点击事件``counter.count += counter.upStep` `// 增加计数器的count``this``.updateTotalCount()` `// 更新总计数``})``}``.width(``'100%'``)` `// 设置列表项宽度``.backgroundColor(Color.White)` `// 设置背景颜色``.justifyContent(FlexAlign.SpaceBetween)` `// 设置内容两端对齐``.padding({ left:` `'30lpx'``, right:` `'30lpx'` `})` `// 设置左右内边距``}``.height(``this``.listItemHeight)` `// 设置列表项高度``.width(``'100%'``)` `// 设置列表项宽度``.margin({``// 设置列表项的外边距``top: index == 0 ?` `this``.listSpacing : 0,` `// 如果是第一个项，设置上边距``bottom: index ==` `this``.counters.length - 1 ?` `this``.listSpacing : 0` `// 如果是最后一个项，设置下边距``})``.borderRadius(10)` `// 设置圆角``.clip(``true``)` `// 裁剪超出部分``.swipeAction({ start:` `this``.itemStart(index), end:` `this``.itemEnd(index) })` `// 设置滑动操作``.scale(counter.scale)` `// 设置计数器缩放比例``.onClick(() => {` `// 点击事件``this``.selectedIndex = index` `// 设置当前选中的计数器索引``this``.isSheetVisible =` `true` `// 显示底部弹出表单``})` `}, (counter: CounterItem) => counter.id.toString())``// 使用计数器ID作为唯一键``.onMove((from: number, to: number) => {` `// 列表项移动事件``const tmp =` `this``.counters.splice(from, 1);` `// 从原位置移除计数器``this``.counters.splice(to, 0, tmp[0])` `// 插入到新位置``})` `}``.scrollBar(BarState.Off)` `// 隐藏滚动条``.width(``'648lpx'``)` `// 设置列表宽度``.height(``'100%'``)` `// 设置列表高度``}``.width(``'100%'``)` `// 设置列宽度``.layoutWeight(1)` `// 设置布局权重` `Row() {` `// 底部合计行``Column() {` `// 列布局``Text(``'合计'``).fontSize(``'26lpx'``).fontColor(Color.Gray)` `// 显示“合计”文本``Text(`${``this``.totalCount}`).fontSize(``'38lpx'``).fontColor(Color.Black)` `// 显示总计数``}.margin({ left:` `'50lpx'` `})` `// 设置左边距``.justifyContent(FlexAlign.Start)` `// 设置内容左对齐``.alignItems(HorizontalAlign.Start)` `// 设置项目左对齐``.width(``'300lpx'``)` `// 设置列宽度` `Row() {` `// 添加按钮行``Text(``'添加'``).fontColor(Color.White).fontSize(``'28lpx'``)` `// 显示“添加”文本``}``.onClick(() => {` `// 点击事件``this``.counters.unshift(``new` `CounterItem(`新计数项${Index.counterId}`))` `// 添加新计数器``this``.listScroller.scrollTo({ xOffset: 0, yOffset: 0 })` `// 滚动到顶部``this``.counters[0].scale = { x: 0.8, y: 0.8 };` `// 设置新计数器缩放``animateTo({``// 动画效果``duration: 1000,` `// 动画持续时间``curve: curves.springCurve(0, 10, 80, 10),` `// 动画曲线``iterations: 1,` `// 动画迭代次数``onFinish: () => {` `// 动画完成后的回调``}``}, () => {``this``.counters[0].scale = { x: 1, y: 1 };` `// 恢复缩放``})``})` `.width(``'316lpx'``)` `// 设置按钮宽度``.height(``'88lpx'``)` `// 设置按钮高度``.backgroundColor(``"#65DACC"``)` `// 设置按钮背景颜色``.borderRadius(10)` `// 设置按钮圆角``.justifyContent(FlexAlign.Center)` `// 设置内容居中``}.width(``'100%'``).height(``'192lpx'``).backgroundColor(Color.White)` `// 设置行宽度和高度` `}``.backgroundColor(``"#f2f2f7"``)` `// 设置背景颜色``.width(``'100%'``)` `// 设置宽度``.height(``'100%'``)` `// 设置高度``.bindSheet(``this``.isSheetVisible,` `this``.mySheet(), {``// 绑定底部弹出表单``height: 300,` `// 设置表单高度``dragBar:` `false``,` `// 禁用拖动条``onDisappear: () => {` `// 表单消失时的回调``this``.isSheetVisible =` `false` `// 隐藏表单``}``})``}` `@Builder``// 构建器装饰器``mySheet() {` `// 创建底部弹出表单``Column({ space: 20 }) {` `// 列布局，设置间距``Row() {` `// 行布局``Text(``'计数标题：'``)` `// 显示“计数标题”文本``TextInput({ text:` `this``.counters[``this``.selectedIndex].name }).width(``'300lpx'``).onChange((value) => {` `// 输入框，绑定计数器名称``this``.counters[``this``.selectedIndex].name = value` `// 更新计数器名称``})` `}` `Row() {` `// 行布局``Text(``'增加步长：'``)` `// 显示“增加步长”文本``TextInput({ text: `${``this``.counters[``this``.selectedIndex].upStep}` })``// 输入框，绑定增加步长``.width(``'300lpx'``)``// 设置输入框宽度``.type(InputType.Number)``// 设置输入框类型为数字``.onChange((value) => {` `// 输入框变化事件``this``.counters[``this``.selectedIndex].upStep = parseInt(value)` `// 更新增加步长``this``.updateTotalCount()` `// 更新总计数``})` `}` `Row() {` `// 行布局``Text(``'减少步长：'``)` `// 显示“减少步长”文本``TextInput({ text: `${``this``.counters[``this``.selectedIndex].downStep}` })``// 输入框，绑定减少步长``.width(``'300lpx'``)``// 设置输入框宽度``.type(InputType.Number)``// 设置输入框类型为数字``.onChange((value) => {` `// 输入框变化事件``this``.counters[``this``.selectedIndex].downStep = parseInt(value)` `// 更新减少步长``this``.updateTotalCount()` `// 更新总计数``})` `}``}``.justifyContent(FlexAlign.Start)` `// 设置内容左对齐``.padding(40)` `// 设置内边距``.width(``'100%'``)` `// 设置宽度``.height(``'100%'``)` `// 设置高度``.backgroundColor(Color.White)` `// 设置背景颜色``}``}` |
| --- | --- |



　　


