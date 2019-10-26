---
layout: post
title: html-mailto
date: 2019-10-26
tags: html
---

## MailTo 属性

> mailto 属性可以设置到`a` 标签和`form` 标签中

例如：
```html
<a href="mailto:****@qq.com">send mail</a>

--或者

<form action="mailto:*****@qq.com">
....
</form>
```

mailto后跟的事收件人的信息
| to      | 收件人 |
| :------ | -----: |
| subject |   主题 |
| cc      |   抄送 |
| bcc     |   暗送 |
| body    |   内容 |

![mailto](/images/posts/2019/2019-10-26-mailto.png)

----

a链接方式：

```html
<a href="mailto:****@qq.com?subject=test&cc=***@qq.com&body=use mailto sample">send mail</a>
```

form方式：

```html
<form name='sendmail' action="mailto:****@qq.com">
    <input name='cc' type='text' value='***@qq.com'>
    <input name='subject' type='text' value='test send mail'>
    <input name='body' type='text' value='use mailto sample'>

</form>
```
----

**当然主题和内容也可为中文，但是要urlencode编码一下，用于格式解析，否则会出现乱码；**


