---
layout: post
title: "jquery_ujs小技巧，使用ajax回调"
date: 2013-02-27 19:42
comments: true
categories: [jQuery, JavaScript, Rails]
---

最近在优化admin后台的几个CRUD页面，要把删除数据的DELETE请求的变成ajax的。为了重用客户端的JavaScript代码，研究了一下jquery_ujs文件。

<!-- more -->

每个CRUD页面要求都是一样的：

1. 删除按钮都放在列表页面，每行数据一个。
2. 点击删除后发起ajax请求。
3. 删除成功后用JavaScript把该行数据删掉。

## 老的做法

按照RailsCasts介绍的方式（我当时只会这一种），link_to加上remote参数，destroy方法里render js返回一段JavaScript，在JavaScript中处理页面相关的操作（这里用的隐藏被删除的行）。

```erb destroy.js.erb
$('#item_<%= @item.id %>').hide('slow');    // 假设每一行起一个id叫item_xxx，这里做的隐藏效果
```

#### 上面那种做法有几个麻烦的地方：

1. 服务器端返回的JavaScript没有上下文环境，想找到需要删除的行就必须通过指定id之类的方式。这样页面上每行还要加一个id。
2. 每个CRUD页面，删除后执行的JavaScript几乎都是这样，用erb写就要每个controller的destroy方法都写一个（Rails的模板继承应该可以解决此问题，没研究过）。

那怎么重构呢？ 我们可以给ajax请求绑定回调函数，为了让所有CRUD页面都使用相同的回调，我们可以写一个初始化函数，检查CRUD的列表，找到所有删除链接，绑定回调。

但Rails采用的是unobtrusive JavaScript，ajax请求的代码是写在jquery_ujs里的，这种情况下要添加ajax回调怎么做呢？

看了下jquery_ujs，它对ajax请求的代码放在__handleRemote__方法里，而且很智能的提供了几个回调，其中一个就是__ajax:success__，ajax请求成功后调用，这样我们就只用对删除按钮绑定__ajax:success__事件就行了。

## 使用jquery_ujs回调的做法

CRUD列表的页面结构简单说下，为了方便找CRUD的列表，我给列表加上了一个crud-list的class，删除按钮是用link_to生成的a标签

```javascript JavaScript
  // 在dom ready事件中绑定这个方法就行了
  function initCrudList() {
    // 想兼容性更好可以加上条件[data-remote=true]，这样只有声明了data-remote的链接会绑定回调，普通链接还是直接发起DELETE请求
    $('.crud-list a[data-method=delete]').on('ajax:success', function() {
      $(this).parents('tr:first').hide('slow');
    });
  }
```

这样就好了。以后如果哪个页面的ajax请求比较特殊，可以使用off把这段回调删了再添加自己想要的逻辑。
