---
layout: post
title: "jQuery中prop和attr的区别"
date: 2013-02-19 21:53
comments: true
categories: [jQuery, JavaScript]
---

一直都以为jQuery中获取html标签属性就是用attr方法，今天碰巧看到一篇文章，才发现还有一个prop方法，就想着查一下两个方法有什么区别。
结果还真挖出来个坑。

先做个实验，把以下代码保存成html文件，跑一遍（写这篇博客时jQuery版本为1.9.1）

```html HTML
<!DOCTYPE html>
<html>
<head>
  <style>
  p { margin: 20px 0 0 }
  b { color: blue; }
</style>
  <script src="http://code.jquery.com/jquery-latest.js"></script>
</head>
<body>

<input id="check1" type="checkbox" checked="checked">
<label for="check1">Check me</label>
<p></p>

<script>
$("input").change(function() {
  var $input = $(this);
  $("p").html(".attr('checked'): <b>" + $input.attr('checked') + "</b><br>"
              + ".prop('checked'): <b>" + $input.prop('checked') + "</b><br>"
              + ".is(':checked'): <b>" + $input.is(':checked') ) + "</b>";
}).change();
</script>

</body>
</html>
```

这是jQuery官方文档的对prop方法做的一个例子，展示prop和attr的区别。先说结果：

``` javascript JavaScript
attr('checked')    // 不管checkbox选中与否，始终返回'checked'，注意不是true和false
prop('checked')    // 根据checkbox的状态返回true或false
```

为什么会有这种结果？jQuery为什么要提供attr和prop两个类似的方法？这得从html说起。

<!-- more -->


## html中的attribute和property

虽然在中文里这两个词差不多的意思，但html中对此的定义却有严格区别。先大致解释下：

* attribute是html文本层面的，我们在写html时看到的 key="value" 就是attribute。其值始终是html元素定义时的值，除非html文本改变。
* property是dom层面的，可以理解成用原生JavaScript函数获取的dom对象的属性。并且会随着我们对dom树的操作而改变。
  有的property是attribute在dom对象中的一种呈现。所以property和attribute有一定的交集。


看个例子：

``` html HTML
<input type="checkbox" id="cb" class="cb" name="key" checked="checked"/>
```

这是一个checkbox，默认是勾选状态的。它的attribute有id，type，class，name和checked。这些attribute的值都不会改变。
即使取消了勾选状态，其checked属性也是"checked"

再看dom层面，id，type，name和checked都有同名的property。class这个attribute对应property的叫className。另外，作为一个dom元素，它还有nodeName这个property。
上面那个checkbox元素的dom对象可以通过JavaScript获取。你可以把property看成JavaScript里面dom对象的属性。

``` javascript JavaScript
var el = document.getElementById('cb');
el.className   // "cb"        property的名称跟attribute的名称并不是完全对应的。
el.class       // undefined
el.nodeName    // "INPUT"     property也不一定就是attribute的照搬，它只跟dom有关
el.checked     // true        注意这里不是"checked"
```

明白了这些后，就不难理解jQuery的prop和attr方法的区别了。但事情还没完。


## jQuery中的attr和prop

为了遵守html规范，jQuery对attribute和property分别提供了封装方法attr和prop。但因为prop是jQuery 1.6版本才引进的，而且jQuery 1.9之前的版本为了考虑老代码的兼容性问题，
对attr和prop的界限划得有点模糊。

还是拿上面那个checkbox举例子，看看jQuery中各版本的区别，先拿checked属性做个实验

```javascript JavaScript
$('.cb').prop("checked")    // true  会随着checkbox状态改变而改变
$('.cb').attr("checked")    // (pre-1.6) true  会随着checkbox状态而改变
                            // (1.6) "checked"  始终获取checkbox的初始状态，不会改变
                            // (1.6.1 - 1.8)	"checked"  会随着checkbox状态改变而改变，这是出于对老代码的兼容性考虑
                            // (1.9.1) "checked"  始终获取checkbox的初始状态，跟1.6一样
```

然后看看其他属性，attr和prop获取的值是否有区别：

```javascript JavaScript
$('.cb').prop('name', "aaa")   // 这里不管用prop还是attr对实验结果都没有影响
$('.cb').prop('name')          // "aaa"  返回的是改变后的状态
$('.cb').attr('name')          // "aaa"  同上
```

可以看到这里并没有什么区别，为什么？因为attribute针对的是html文本改变，以上代码，改变name的时候，相应的html文本也产生了变化（用开发工具可以看到）。
所以这里attr和prop获取的都是改变后的值。


最后个人总结一下什么时候适合用什么方法：

* 改变property就会改变html的，不管用prop还是attr都可以。大多数的property都属于此列，比如type，name，src。
  你可以用JavaScript改变property后再用开发工具看看html是否产生了变化。
* 改变property不会改变html的，适合用prop，或者jQuery针对该属性专门提供的方法
  举个例子，文本框里的value属性，可以通过用户输入，或者用JavaScript改变，prop和val都可以获取正确的值，但优先推荐val。
  这种情况attr返回的值就完全不对了。
* 不属于attribute对应的property，适合用prop，比如nodeName。
* 表示true/false含义的attribute，适合用prop，比如checked，disabled，方便做判断。


## 参考文档

* [jQuery: Test/check if checkbox is checked](http://jquery-howto.blogspot.jp/2013/02/jquery-test-check-if-checkbox-checked.html)  
  这篇文章列举了几种判断checkbox是否被选中的方法，例子挺多。

* [HTML: The difference between attribute and property](http://jquery-howto.blogspot.jp/2011/06/html-difference-between-attribute-and.html)  
  这篇文章讲了attribute和property的区别，但对attribute的定义并不正确，attribute并不是始终以在html定义时的初始状态为准，而是始终跟html文本的变化保持同步。

* [jQuery API: prop](http://api.jquery.com/prop/)  
  官方API，说明了为什么要加入prop这个方法，和在不同版本中的差异。
