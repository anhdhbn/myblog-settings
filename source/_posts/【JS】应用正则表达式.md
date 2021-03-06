---
title: 【JS】应用正则表达式
date: 2018-02-26 12:11:03
tags:
  - js
  - 正则表达式
categories:
  - js
---

> 之前也写过正则学习相关的，取这标题只是因为最近读忍者秘籍第2版又讲到这个话题，作为学习、回顾把一些重要的知识点和方法记录下来。正则表达式需要长时间的积累及不断实践才能提高，虽然工作中碰不到太多正则使用场景，简单的记录也算作一种学习、应用吧

<!-- more -->

## 我们可以使用正则解决各种任务

* 操作HTML节点中的字符串
* 使用CSS选择器表达式定位部分选择器
* 判断一个元素是否具有指定的类名
* 输入校验
* 其他任务

---

## 带着问题学习

1. 什么时候使用正则字面量，而不是使用正则对象
2. 什么是粘连匹配，如何开启粘连匹配
3. 使用全局和非全局正则表达式的区别是什么

---

## 正则相关

创建正则的方式：

* 使用正则字面量
* 通过创建 RegExp 对象的实例

回答1： 当正则表达式在开发环境是明确的，推荐优先使用字面量语法；当需要在运行时动态创建字符串来构建正则表达式时，则使用构造函数的方式。

正则修饰符：

* i 对大小写不敏感
* g 查找所有匹配项
* m 多行匹配
* y 开启粘连匹配
* u 允许使用 Unicode点转义符

### 术语和操作符

**精确匹配**

/test/ 这类，/^test$/

**匹配字符集**

[abc] 我们匹配的是 a b 或 c，只是abc其中一个字符

[^abc] 字符集里的 ^ 表示除了abc意外的任意字符，这要和 ^ 以开始作区别

**转义**

$ . ^ [ / ] \ { } ' 这些特殊字符都是需要 添加 `\` 来进行转义的

**起止符号**

尖括号()用于匹配字符串的开始，如 /^test/就是匹配test出现在字符串的开头。（注，这只是字符的重载，^还可以表示非）

美元符号$表示字符串的结束，如/test$/ 就是匹配 test出现在字符串的末尾

**重复出现**

* ? 0次或1次
* + 1次或多次
* * 0次或多次
* {n} 表重复次数
* {2, 4} 重复范围 出现2到4次
* {4, } 匹配4次或更多

这些字符串都可以是贪婪的或非贪婪的。默认是贪婪模式，可以匹配所有可能的字符。在运算符后添加?，例如 a+?，使得运算符为非贪婪模式，只进行最小限度的匹配。
例如，对于字符串aaa，正则表达式/a+/会匹配全部3个字符，而非贪婪模式/a+?/则匹配一个字符a，因为一个字符a足以满足a+术语。

**预定义字符集**只列常用的

* \t 水平制表符
* \r 回车
* \h 换行符
* . 匹配除换行意外的任意字符
* \d 匹配十进制数字 [0-9]
* \D 匹配非十进制数字外任意字符 [^0-9]
* \w 匹配任何数字、字母、下划线 [0-9a-zA-Z_]
* \W 匹配除了数字、字母、下划线以外的任意字符 [^0-9a-zA-Z_]
* \s 匹配任意空白字符 空格、制表符、换行等
* \S 匹配除空白符以外的任意字符
* \b 匹配单词边界
* \B 匹配非单词边界

**分组**

操作符，如 + 和 * 只影响其前面的术语。如果对一组术语使用操作符，可以使用**圆括号**()进行分组，这与数学表达式类似。例如，/(ab)+/匹配一个或多个连续的ab

**或操作符**

使用竖线 | 表示或。例如， /a|b/表示可以匹配a或者b，/(ab+)|(cd+)/表示可以匹配一个或多个ab或cd

**反向引用**

反向引用可引用正则中定义的捕获。稍后会详细介绍。

现在只需要把捕获看作待匹配的字符串，也就是前面匹配的字符串。反向引用分组中捕获的内容，使用反斜线加上数字表示引用，该数字从1开始。
例如 /^([dtn]a)\1/ 匹配任意从d t或n开始的连续有a的字符串，连续匹配第一个分组中捕获的内容。

### 编译正则表达式

处理正则经历多个阶段，其中两个主要的阶段是编译和执行。
编译阶段发生在正则表达式被创建的时期。执行发生在使用编译之后的正则表达式进行匹配字符串的时期。

在编译过程中，表达式经过JS引擎的解析，转换为内部代码。解析和转换的过程发生砸死正则表达式创建时期（浏览器会进行内部优化处理）。通常的，浏览器会判断使用哪条正则并缓存表达式的编译结果。但对于复杂表达式，我们通过预编译正则，使得性能得到明显提升。

编译正则并将其保存在变量是个很重要的优化过程。
每个正则都有以个独特的对象表示，每次创建一个正则都会创建一个新的正则表达式对象。这与原始类型不同，因为每个正则都是独一无二的。使用构造函数创建正则可以在运行时使用字符串创建正则。对于构建可以重复使用的复杂正则很有用。

在运行时编译一个稍后使用的正则，如下：

```html
<div class="samurai ninja"></div>
<div class="ninja samurai"></div>
<div></div>
<span class="samurai ninja ronin"></span>
<script>
function findClassInElements(className, type) {
  // 根据标签类型查找元素
  const elems = document.getElementsByTagName(type || '*')
  // 使用传入的class编译正则
  const reg = new RegExp(`(^|\\s)${className}(\\s|$)`)
  // 储存最终结果
  const results = []
  for (let i = 0, len = elems.length; i < length; i++) {
    // 检测是否与正则匹配
    if (reg.test(elems[i].className)) {
      results.push(elems[i])
    }
  }
  return results
}
</script>
```

这种方式可以大大提高性能。一旦正则被编译后，就可以通过该正则的test方法收集匹配的元素。
推荐使用预创建和预编译的正则表达式，这对性能有很大提升。在正则表达式中使用圆括号，不仅可以用于术语，还可以创建捕获。

### 捕获匹配的片段

执行简单捕获，代码示例如下:

```html
<div id="square" style="transform:translateY(15px);"></div>
<script>
function getTranslate(elem) {
  const transformValue = elem.style.transform
  if (transformValue) {
    const match = transformValue.match(/translate[x|y|z|3d]\(([^\)]+)\)/)
    console.log(match)
    return match ? match[1] : ''
  }
  return ''
}
const square = document.getElementById('square')
getTranslate(square)
</script>
```

match方法匹配结果通过第一个索引返回，然后每次捕获结果索引递增。所以第0个匹配的是整个字符串translateY(15px)，第二个位置是15px。
在正则表达式中使用圆括号定义捕获。因为我们在正则表达式中仅定义了一个捕获，因此当匹配变换值时，其值存储在[1]中。

### 使用全局表达式进行匹配

使用局部正则表示可以返回数组，该数组中包含全部匹配的内容以及操作中的全部匹配结果。
但当使用全局正则表达式g返回全部的匹配结果，但不会返回捕获结果。代码如下：

```js
const html = `<div class="test"><b>hello</b> <i>world!</i></div>`
const reg1 = /<(\/?)(\w+)([^>]*?)>/
const reg2 = /<(\/?)(\w+)([^>]*?)>/g
const result = html.match(reg1)
console.log(result) // ["<div class="test">", "", "div", " class="test"", index: 0, input: "<div class="test"><b>hello</b> <i>world!</i></div>"]
const all = html.match(reg2)
console.log(all) // ["<div class="test">", "<b>", "</b>", "<i>", "</i>", "</div>"]
```

当局部匹配时只有一个实例被匹配，并返回匹配中的捕获结果。但当使用全局匹配时，返回的是匹配的全部内容列表。
若捕获结果重要，可以在全局匹配中使用正则的exec方法，可多次对一个正则调用exec方法，每次调用都可以返回下一个匹配的结果，代码如下：

```js
const html = `<div class="test"><b>hello</b> <i>world!</i></div>`
const reg = /<(\/?)(\w+)([^>]*?)>/g
let match, num = 0
while (match = reg.exec(html) !== null) {
  num++
}
console.log(num) // 5
```

### 捕获的引用

对捕获结果进行引用有两种方式：

1. 在自身匹配
2. 字符串替换

下示例为使用反向匹配html标记的内容

```js
const html = `<b class="hello">hello</b> <i>world!</i>`
const reg = /<(\w+)([^>]*)>(.*?)<\/\1>/g //使用反向引用
let match = reg.exec(html)
console.log(match) // ["<b class="hello">hello</b>", "b", " class="hello"", "hello"]
match = reg.exec(html)
console.log(match) // ["<i>world!</i>", "i", "", "world!"]
```

使用 \1 指向表达式中的第一个捕获，在本例中捕获的是标记的名称。使用第一个捕获的内容匹配对应的结束标记。
此外，可以调用字符串的replace方法，对替换字符串内获取捕获。不使用反向引用，可以用$1、$2、$3等标记捕获序号，如下：

    'fontFamily'.replace(/([A-Z])/g, '-$1').toLowerCase()

上述代码中，第一个捕获值 F 通过替代字符$1进行引用，从而替换。由于捕获和分组都可以使用圆括号表示，对于正则处理器来说，无法区别所添加的是捕获还是分组。对于这种情况我们继续介绍

### 未捕获的分组

圆括号有两项职责：

1. 定义分组
2. 指定捕获

对于存在大量分组的正则表达式来说，可能会产生太多不必要的捕获，这会导致处理捕获变得频繁

    const pattern = /((ninja-+))sword/

创建正则时，允许前缀 ninja-单词在sword之前出现1次或多次，并捕获整个前缀。该正则使用了两个括号：

1. 定义捕获的圆括号，字符串sword之前的全部内容
2. 定义ninja-和+操作符分组的操作符

返回结果中不止一个捕获，因为含有用于分组的圆括号。说明有一组括号不应该产生捕获，正则表达式语法可以在起始圆括号之后使用符号 ?: ，这就是所谓的`被动子表达式`。

    const pattern = /((?:ninja-)+)sword/

只有外层括号会创建捕获，内层括号会变成一个被动子表达式，我们来看以个具体的示例：

```js
const reg1 = /((ninja-+))sword/
const reg2 = /((?:ninja-+))sword/
const ninjas1 = 'ninja-ninja-sword'.match(reg1) // ["ninja-sword", "ninja-", "ninja-"]
const ninjas2 = 'ninja-ninja-sword'.match(reg2) // ["ninja-sword", "ninja-"]
```

在不需要捕获情况下，尽量使用非捕获分组代替捕获，表达式引擎不需要记忆和返回捕获结果，这利于提高性能。

### 利用函数进行替换

replace最重要的特征不仅支持替换值，而且支持替换函数作用参数：

* 全文匹配
* 匹配时的捕获
* 在原始字符串匹配的索引
* 源字符串
* 从函数返回的值作为替换值

下面来看示例，将短横线连接的字符串转换为驼峰式字符串

```js
function upper(all, letter) {return letter.toUpperCase()} // 转换为大写的函数
'border-bottom-width'.replace(/-(\w)/g, upper) // borderBottomWidth
```

由于全局匹配，对每一个匹配的字符串都会执行替换函数。我们再来看一个实例加深印象

假设需要将查询字符串换为符合我们所需格式的字符串，需将 foo=1&foo=2&blah=a&blah=b&foo=3转换为 foo=1,2,3&blah=a,b

```js
function compress(source) {
  const keys = {}
  source.replace(
    /([^=&]+)=([^&]*)/g,
    function(full, key, value) { // 提取键值对信息
      keys[key] = (keys[key] ? keys[key] + ',' : '') + value
      return ""
    }
  )
  const result = []
  for (let key in keys) {
    if (keys.hasOwnProperty(key)) {
      result.push(key + '=' + keys[key])
    }
  }
  return result.join('&')
}
```

### 几个实例

匹配所有字符，包括换行符

```js
const html = '<b class="hello">hello</b> \n<i>world!</i>'
// 不匹配换行
/.*/.exec(html) // ["<b class="hello">hello</b>"]
// 使用空白符匹配所有字符
/[\S\s]*/.exec(html) //["<b class="hello">hello</b>\n<i>world!</i>"]
// 第二种匹配所有字符的方法
/(?:.|\s)*/.exec(html)
```

在css选择器中匹配转义字符

```js
// 该正则允许任意字符序列组成的单词，包括一个反斜线紧跟任意字符
const reg = /^((\w+)|(\\.))+$/
const tests = [
  'formUpdate',
  'form\\.update\\.whatever',
  'form\\:update',
  '\\f\\o\\r\\m\\u\\p\\d\\a\\t\\e',
  'form:update'
]
for (let i = 0; i < tests.length; i++) {
  console.log(reg.test(tests[i]))
}
```

## 总结

* 创建正则可以使用正则字面量 /test/ 或正则构造函数 new RegExp('test')。对于在开发环境明确的推荐使用正则字面量，在运行时则推荐使用构造函数
* 使用section指定一组待匹配的字符
* 理解掌握5种模式gimyu，各种术语和操作符：匹配字符集[] 转义\ 起止^ 结束$ 重复 预定义字符等
* 使用圆括号对多个术语进行分组，使用竖线 | 表示或
* 通过反斜线加数字 \1 \2 可以对匹配的字符串进行反向引用
* 字符串的正则方法，match test replace
