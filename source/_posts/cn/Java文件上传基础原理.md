---
layout: post
title: Java文件上传基础原理
catalog: true
typora-root-url: Java文件上传基础原理
author: Futari
date: 2024-09-14 22:28:59
---

**POST请求**前端form表单在最初只能提交文本数据，即为<font style="color:rgb(68, 68, 68);background-color:rgb(245, 245, 245);">application/x-www-form-urlencoded(默认值)</font>

<font style="color:rgb(68, 68, 68);background-color:rgb(245, 245, 245);">在1995年multipart/form-data，在form表单的enctype属性进行指定</font>



在一个HTML文档中，一个表单的标准格式如下：

```html
<form action="http://localhost:8080/repositories" method="get">
  <input type="text" name="language" value="go" />
  <input type="text" name="since" value="monthly" />
  <input type="submit" />               
</form> 
```

:::success
这样的一个Form被加载到浏览器中后会呈现为一个表单的样式，当在两个文本框中分别输入文本(或以默认的文本作为输入)后，点击“提交(submit)”，浏览器会向[http://localhost:8080](http://localhost:8080)发出一个HTTP请求，由于Form的method属性为get，因此该HTTP请求会将表单的输入文本作为查询字符串参数(Query String Parameter，在这里即是?language=go&since=monthly)。服务器端处理完该请求后，会返回一个HTTP承载的应答，该应答被浏览器接收后会按特定样式呈现在浏览器窗口中。上述这个过程可以用总结为下面这幅示意图：

:::

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1726283927946-8c09fc1d-ae2d-4fed-8475-e9d143b84685.png)



Form中的method也可以使用post，就像下面这样：

```html
<form action="http://localhost:8080/repositories" method="post">
   <input type="text" name="language" value="go" />
   <input type="text" name="since" value="monthly" />
   <input type="submit" />
</form>
```

:::success
改为post的Form表单在点击提交后发出的http请求与method=get时的请求有何不同呢？不同之处就在于在method=post的情况下，表单的参数不会再以查询字符串参数的形式放在请求的URL中，而是会被写入HTTP的BODY中。我们也将这一过程用一幅示意图的形式总结一下：

:::

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1726283937364-431d4e94-dfad-483b-81e7-745975d0fca1.png)



:::success
由于表单参数被放置在HTTP Body中传输(body中的数据为：language=go&since=monthly)，因此在该HTTP请求的headers中我们会发现新增一个header字段：Content-Type，在这里例子中，它的值为application/x-www-form-urlencoded。我们可以在Form中使用enctype属性改变Form传输数据的内容编码类型，该属性的默认值就是application/x-www-form-urlencoded(即key1=value1&key2=value2&...的形式)。

:::

修改<font style="color:rgb(68, 68, 68);background-color:rgb(245, 245, 245);">enctype属性</font>，得到以下：

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1726284040903-87d6eca9-c2b0-49b9-ab79-c75d7e2ec970.png)

> <font style="color:rgb(77, 77, 77);">enctype=text/plain时，这种编码格式也称为raw,即将数据内容原封不动的放入Body中传输，保持数据的原先的编码方式(通常为utf-8)；</font>

当enctype=multipart/form-data时，HTTP Body中的数据以多段(part)的形式呈现，段与段之间使用指定的随机字符串分隔，该随机字符串也会随着HTTP Post请求一并传给服务端(放在Header中的Content-Type的值中，与multipart/form-data使用分号相隔)，如：

```html
Content-Type: multipart/form-data; boundary=--------------------------399501358433894470769897
```

服务端会根据分割符号，对被分割的每段根据content-type进行解析，如下：

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1726284334199-1ccbc467-2f56-4040-a60f-32564dc13d11.png)

:::success
<font style="color:rgb(77, 77, 77);">对于无法识别的文件类型（比如：没有扩展名），文件分段的Content-Type通常会设置为</font>**<font style="color:rgb(77, 77, 77);">application/octet-stream</font>**<font style="color:rgb(77, 77, 77);">。</font>

:::

