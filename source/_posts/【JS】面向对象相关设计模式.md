---
title: 【JS】面向对象相关设计模式
date: 2017-11-01 22:59:03
tags: 
  - oop
  - 设计模式
categories: 
  - js
  - oop
  - 设计模式
---


> 面向对象应该是一种思想，而不是代码的组织形式。

面向对象的特点：

* 封装
* 继承
* 多态

子类继承了父类的函数，同时覆盖实现了父类的某些行为。上面的setProgress函数即体现了多态又体现了继承

<!-- more -->

### 继承和组合

集成商为了实现复用，组合其实也是为了实现复用。继承是is-a的关系，而组合是has-a的关系。

可以把上面的ProgressBar改成组合的方式。 

```js
class ProgressBar {
  constructor($container) {
    this.fullWidth = $container.width()
    this.$bar = null
  }  
  // 设置宽度
  setProgress(percentage) {
    this.$bar.animate({
      width: this.fullWidth * percentage + 'px'
    })
  }
  // 完成
  finished() {
    this.$bar.hide()
  }
  // 失败
  failed() {
    this.addFailedText()
  }
  addFailedText() { }
}

class ProgressBarWithNumber extends ProgressBar {
  constructor($container) {
    super($container)
  }
  // 多态
  setProgress(percentage) {
    // 先借助继承父类的函数
    super.setProgress(percentage)
    this.showPercentageText(percentage)
  }
  showPercentageText(percentage) {}
}
```

在构造函数里面组合了一个progressBar的实例，然后在setProgress函数里面利用这个实例去设置进度条的百分比。

```js
class ProgressBarWithNumber {
  constructor($container) {
    this.progressBar = new ProgressBar($container)
  }
  setProgress(percentage) {
    this.progressBar.setProgress(percentage)
    this.showPercentageText(percentage)
  }
  showPercentageText(percentage) {}
}
```

**偏向于使用组合而非继承**

因为继承的耦合性要大于组合，组合更加灵活。继承是编译阶段就决定了关系，而组合是运行阶段才决定关系。

### 面向对象编程原则和设计模式

1. 单例模式

```js
var taskWorker = {
  tasks: [],
  draw() {},
  addTask(task) {
    Task.tasks.push(task)
  }
}
var mapTask = {
  add: function(task) {
    taskWorker.addTask(task)
  }
}
```

每次get的时候先判断mapTask有没有Task的新实例，如果没有则为第一次。先去实例化一个，并做些初始化工作，如果有则直接返回。然后执行mapTask.get()的时候就能够保证获取到的是一个单例。

* 破坏单例

        mapTask.aTask = null


2. 策略模式

场景：注册弹框，不同的弹框文字。把文案当做一个个的策略，使用时根据不同类型，映射到不同的策略。
这样比写if-else或者switch的好处在于：如果以后要增加或删除某种类型，只需要增删一个type就可以了，而不用去改动if-else逻辑。这就叫做开放封闭原则——对修改是封闭的，而对扩展是开放的。

```js
var popType = {
  userReg: {
    title: 'Create your account'
  },
  favHouse: {
    title: 'Add home to favorite'
  },
  saveSearch: {
    title: 'Save this search'
  }
}

var tpl = `<section><h1>{{title}}</h1></section>`

Mustache.render(tpl, popType['userReg'])

// 把回调操作封装成一个策略
var popCallback = {
  userReg: function() {},
  favHouse: function() {},
  saveSearch: function() {}
}

util.ajax('/register', function() {
  var popType = 'favHouse' // 获取popType
  popCallback[popType]
})
```

3. 观察者模式

观察者向消息的接受者订阅消息，一旦接受者收到消息后就把消息下发给它的观察者们。在一个回执搜索的应用里面，单击最后一个点关闭路径，要触发搜索。

```js
class Input {
  constructor(inputDom) {
    this.inputDom = inputDom
    this.visitors = {
      click: []
    }
  }

  // 添加访问者
  on(eventType, visitor) {
    this.visitors.push(visitor)
  }

  // 收到消息，把消息分发给访问者
  trigger(type, event) {
    if (this.visitors[type]) {
      for (let i = 0; i < this.visitors[type]; i++) {
        this.visitors[type]()
      }
    }
  }
}
```

4. 适配器模式

在一个响应式的页面里，假设小屏和大屏显示的分页样式不同，它们初始化和更新状态的函数都不同，如下：

```js
// 小屏
var pagination = new jqPagination({
})
pagination.showPage = function(curPage, totalPage) {
  pagination.setPage(curPage, totalPage)
}
// 大屏
var pagination = new Pagination({
})
pagination.showPage = function(curPage, totalPage) {
  pagination.showItem(curPage, totalPage)
}
```

如果每次调用都得判断一下不同的屏幕大小然后调用不同函数就很麻烦，所以用一个适配器，对外提供统一的接口：

```js
var screen = $(window).width() < 800 ? 'small' : 'large'
var paginationAdapter = {
  init: function() {
    this.pagination = screen === 'small' ? new jqPagination() : new Pagination()
    if (screen === 'large') {
      this.pagination.showItem = this.pagination.setPage
    }
  },
  showPage: function(curPage, totalPage) {
    this.pagination.showItem(curPage, totalPage)
  }
}
```

使用者只要调用 `paginationAdapter.showPage` 就可以更新分页状态，它不需要去关心当前是大屏还是小屏，由适配器去处理这些细节

5. 工厂模式

工厂模式是把创建交给一个工厂，使用者无需要关心创建细节，如下：

```js
var taskCreator = {
  createTask: function(type) {
    switch(type) {
      case 'map':
        return new MapTask()
      case 'search':
        return new SearchTask()
    }
  }
}
var mapTask = taskCreator.createTask('map')
```

需要哪种类型的Task时就传一个类型或者name给以个工厂，工厂根据名字去生产相应的产品，不用关心它是怎么创建的，要不要单例之类。

6. 外观/门面模式

在一个搜索逻辑里，为了显示搜索结果需要执行：

```js
hideNoResult() // 隐藏没有结果的显示
removeOldResult() // 删除老的结果
showNewResult() // 显示新结果
showPageItem() // 更新分页
resizePhoto() // 结果图片大小重置
```

于是考虑用一个模块把它包起来

```js
function showResult() {
  hideNoResult() // 隐藏没有结果的显示
  removeOldResult() // 删除老的结果
  showNewResult() // 显示新结果
  showPageItem() // 更新分页
  resizePhoto() // 结果图片大小重置
}
// 需要时调用 showResult()
```

把多个操作封装成一个模块，对外只提供一个门面叫showResult，使用者只要调一下该函数即可

7. 状态模式

实现一个类似微博的消息框，要求是当数字为0或者超过140时，发推按钮可单击，且剩余数字会跟着变

可用一个state来保存当前的状态，然后当用户输入时，这个state的数据会跟着变，同时更新按钮状态

```js
var tweetBox = {
  init() {
    // 初始化一个state
    this.state = {}
    tweetBox.bindEvent()
  },
  setState(key, val) {
    this.state[key] = val
  },
  changeSubmit() {
    // 通过获取当前的state
    $('#submit')[0].disabled = tweeetBox.state.text.length === 0
      || tweetBox.state.text.length > 140
  },
  showLeftTextCount() {
    $('#text-count').text(140 - this.state.text.length)
  },
  bindEvent() {
    $('.tweet-textarea').on('input', function() {
      // 改变当前的state
      tweetBox.setState('text', this.value)
      tweetBox.changeSubmit()
      tweetBox.showLeftTextCount()
    })
  }
}
```

用一个state保存当前状态，通过获取当前state进行下一步操作。

当然，上面的还有优化空间，代码如下：

```js
var tweetBox = {
  setState(key, val) {
    this.state[key] = val
    renderDom($('.tweet'))
  },
  renderDom($currentDom) {
    diffAndChange($currentDom, renderVirtualDom(tweetBox.state))
  }
}
// `<input type="submit" disabled={{this.state.text.length === 0 || this.state.text.length > 140}} />`
```

这其实就是React的原型，不同的状态有不同的表现行为，所以可以认为是一个状态模式，并且通过状态去驱动DOM更改

8. 代理模式

其实React不直接操作DOM，而是把数据给state，然后委托给state和虚拟DOM去操作真实DOM，所以它是一个代理模式

eventHandler -> state -> renderDom()

9. 状态模式的另一个例子

改变一个房源的状态：

```js
if (newState === 'sold') {
  if (currentState === 'building' || currentState === 'dold') {
    return 'error'
  } else if (currentState === 'ready') {
    currentState = 'sold'
    return 'ok'
  }
} else if (newState === 'ready') {
  if (currentState === 'building') {
    currentState = 'toBeSold'
    return 'ok'
  }
}
```

更改一个房源的状态之前先要判断一下当前的状态，如果当前状态不支持那么不允许修改。对上面的代码我们可以用代理模式重构一下，如下：

```js
var stateChange = {
  ready: {
    building: 'error',
    ready: 'error',
    sold: 'ok'
  },
  building: {
    building: 'error',
    ready: 'ok',
    sold: 'error'
  }
}
if (stateChange[currentState][newState] !== 'error') {
  currentState = newState
}
return stateChange[currentState][newState]
```

你会发现状态模式和策略模式是孪生兄弟，它们形式相同，只是目的不同。

策略模式封装成策略，状态模式封装成状态。这样的代码就比写很多个if-else强多了，特别是当切换关系比较复杂的时候

10. 装饰者模式

要实现一个贷款的计算器，点计算按钮后，除了要计算结果，还要把结果发给后端做一个埋点。所以写了一个calculateResult函数：

```js
function calculateResult(form) {
  var data = $(form).serializeForm()
  var l = data.rate / 1200
  var o = Math.pow(1 + l, data.term * 12)
  var e = data.price * (1 - data.payment / 100)
  var result = (e * l * o / (o - 1))
  var formatResult = util.formatMoney(result).toFixed(0))

  var $calResult = $('.loan-cal .cal-result-con')
  $calResult.find('.pi-result').text(formatResult)
  // 这个函数包含了两个功能，一个计算结果，一个改变DOM
  return result
}
// 计算按钮click回调
var result = calculateResult(form)
// 发送一个埋点请求
util.ajax('/cal-load', {result})
```

因为要把结果返回出来，所以这个函数有两个功能，一个是计算结果，第二个是改变DOM，这样写在一起感觉不太好。
于是我们把函数拆了，首先有一个LoanCalculator的类专门负责计算小数结果： 

```js
// 计算结果
class LoanCalculator {
  constructor(form) {
    this.form = form
  }
  calResult() {
    var result = 'xxx'
    this.result = result
    return result
  }
  getResult() {
    if (!this.result) this.result = this.calResult()
  }
}
```

它还提供了一个getResult的函数，如果结果没算过那先算一下保存起来，如果已经计算过了那就直接用算好的结果。
然后再写一个`NumberFormater`，它负责把小数结果格式化成带逗号的形式：

```js
class NumberFormater {
  constructor(calculator) {
    this.calculator = calculator
  }
  calResult() {
    var result = this.calculator.calResult()
    this.result = result
    return util.formatMoney(result)
  }
}
```

在它的构造函数里传一个calculator给它，这个calculator可以是上面的LoanCalculator，获取到它的计算结果然后格式化。
接着写一个DOMRenderer的类，它负责把结果下显示出来：

```js
// 显示结果
class DOMRenderer {
  constructor(calculator) {
    this.calculator = calculator
  }
  calResult() {
    let result = this.calculator.calResult()
    $('.pi-result').text(result)
  }
}
```

最后代码调用如下：

```js
let loadCalculator = new LoanCalculator(form)
let numberFormator = new NumberFormator(loadCalculator)
let domRenderer = new DOMRenderer(numberFormator)
domRenderer.calResult()
util.ajax('/cal-loan', {result: loadCalculator.getResult()})
```

可以看到它就是一个装饰的过程，一层一层地装饰：

DOMRenderer -> NumberFormator -> LoanCalculator

下一个装饰者调用上一个calResult函数，对它的结果进一步地装饰。如果这些装饰者的返回结果类型比较平行时，可以一层层地装饰下去。

使用装饰者模式，逻辑是清晰了，但是系统的复杂性增加了，有时候能用简单方式实现还是用简单方式。

---

## 总结

总结一下上文提到的面向对象的编程原则：

1. 把共性和特性或者会变和不变的部分分离出来
2. 少用继承，多用组合
3. 低耦合高聚合
4. 开放封闭原则（对修改封闭，对扩展开放）
5. 单一职责原则

