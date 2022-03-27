---
title: jQuery操作iframe页面DOM
author: 小谈
type: post
date: 2016-06-03T11:36:36+00:00
url: /2016/06/jquery-iframe/
duoshuo_thread_id:
  - 6291928656080012033
categories:
  - Develop
  - 前端

---
如图，左侧是一个表单，右侧是一个iframe嵌入的页面，要实现的效果是，左边表单选择不同的模板，右侧要展现不同的模板页面，当输入的标题，文案发送改变，右侧也要实时动态变化，提供预览的效果。

<a href="https://blog.tanteng.me/wp-content/uploads/2016/06/jquery_iframe.png" target="_blank"><img class="alignnone size-full wp-image-10135" src="https://blog.tanteng.me/wp-content/uploads/2016/06/jquery_iframe.png" alt="jquery_iframe" width="1092" height="514" /></a>

<!--more-->

### 1.jQuery改变iframe引用页面

选择不同模板，iframe展现不同的模板页面，只需在模板选项发生改变事件时，改变iframe的src属性值为新的模板路径即可。

<code class="lang:php decode:true ">//获取模板并在预览框展示
function previewTemplate(template_id) {
    $.post("/tipsadmin/ajaxGetTemplateName", {template_id: template_id}, function (data) {
        template_name = data.name;
        preview_url = data.previewUrl;
        if (template_name) {
            $("#preView").attr('src', preview_url + template_name + '/index.html');
        } else {
            $("#preView").attr('src', '');
        }
    }, 'json');
}</code>

### 2.jQuery操作iframe里的DOM

实现左侧表单文字改变，iframe模板页相应位置文字实时变化，需要jQuery操作iframe里的DOM元素。

如iframe页面标题的id为title，文案内容的id为content，改变iframe里的DOM的值，jQuery代码片段如下：

<code class="lang:php decode:true ">var $iframe = $("#preView");
var $rule_title = $("#rule_title");
var $rule_content = $("#rule_content");

//标题，内容文字变化预览效果
function contentChange() {
    $rule_title.change(function () {
        $iframe.contents().find("#title").html($(this).val());
    });

    $rule_content.change(function () {
        $iframe.contents().find("#content").html($(this).val());
    });
}</code>

这里主要是获取iframe的contents()属性，就可以操作iframe里的DOM了。