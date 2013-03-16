---
layout: post
title: "jQuery中不同的事件绑定方法的区别"
date: 2013-02-24 12:55
comments: true
categories: [jQuery, JavaScript]
---

这段时间处理事件绑定方法比较多，突然想到jQuery里绑定方法挺多的，有on, bind, delegate，还有以前记得的live和one。以前都只会用，
没具体研究过区别在哪里。这篇文章算是把知识梳理一下，做个总结。

<!-- more -->

## on

这个是jQuery 1.7加入的方法，它提供了所有其他事件绑定方法的功能，统一了事件绑定的API。1.7以后的版本推荐都用此方法。
看看例子就知道怎么用了，详细的可以去查官方API。

相应的解除绑定的方法是__off__。

```javascript JavaScript
$('table a.delete').on('click', function() {});             // 绑定click方法
$('table a.delete').on('click.myPlugin', function() {});    // 绑定click方法，加上命名空间myPlugin，方便解除绑定
$('table').on('click', 'a.delete', function() {});          // delegate的写法，绑定
$('table').on('click dblclick', function() {});             // 同时绑定click和dblclick方法
$('table').on({                                             // 比较少见的用法，一般都用链式调用做了
  click: function() {},
  dblclick: function() {}
});
```

## bind

给html元素直接绑定事件，1.7以前的版本没有on方法，只能用这个。

相应的解除绑定的方法是__unbind__。

```javascript JavaScript
$('table a.delete').bind('click', function() {});    // 给a绑定click事件
$('table a.delete').on('click', function() {});      // on的写法
$('table a.delete').click(function() {});            // 只是绑定dom事件一般都这样写了
```

## delegate

指定一个根元素，给它下面符合选择器要求的元素绑定事件。
它利用的是事件冒泡特性，回调函数实际上绑定给跟元素，
当下面的元素触发了事件，冒泡上来被跟元素捕获，jQuery检查它符合选择器的要求，就会执行回调函数。
当要绑定事件的元素不存在，或者有频繁的变动时（比如被删除了或者新添加了），适合使用这个方法。

相应的解除绑定的方法是__undelegate__。

```javascript JavaScript
$('table').delegate('a.delete', 'click', function() {});    // delegate的写法
$('table').on('click', 'a.delete', function() {});          // on的写法
```

如果用bind的话，当table中新加了一个a.delete元素时，你必须记得调用bind方法给它绑定click事件，
元素被删除之前为了性能考虑，最要调用一下unbind解除绑定。不仅麻烦，而且给太多元素绑定事件也会影响性能。

这种应用最典型的两个例子就是jquery-rails中的jquery_ujs和Bootstrap的各种插件。
jquery_ujs对document元素绑定了各种事件，处理data-remote="true"的元素等等。
Bootstrap中可以通过在html中声明data-xxx属性来使用JavaScript插件，也是插件在一开始就绑定回调到document元素的关系。

## live

这个方法在jQuery 1.7中被废弃，1.9中就被移除了。不建议使用（可用delegate替代）。
它跟delegate提供一样的功能，只是事件直接元素直接绑定到document，省去了跟元素的定义，看看例子就知道了。

相应的解除绑定的方法是__die__，同样已被废弃。

```javascript JavaScript
$('a.delete').live('click', function() {});                  // 跟元素为document
$(document).delegate('a.delete', 'click', function() {});    // delegate的写法
$(document).on('click', 'a.delete', function() {});          // on的写法
```

## one

这个方法跟on差不多，就是对于每个绑定的元素和事件，回调只执行一次，之后自动unbind。
它的方法签名跟on完全一样，你会用on就会用这个。

```javascript JavaScript
$('table a.delete').one('click', function() {});       // 通常的写法
$('table').one('click', 'a.delete', function() {});    // delegate的写法
```

## 参考文档

好吧我标题党了……直接去看官方API就可以了。一个on的API就能让你了解jQuery事件绑定的方方面面。也许还有些平时你不知道的知识。 比如：

1. 你知道回调函数里return false和stopPropagation和preventDefault的区别吗？
2. 你知道stopPropagation和stopImmediatePropagation的区别吗？
3. 你知道trigger和triggerHandler的区别吗？
