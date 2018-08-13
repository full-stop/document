`script` 标记是 netspace 公司最早为在 html中引入 javascript代码而创造的HTML元素，并最终被 HTML规范采纳。
`script` 标记有四个比较重要的属性：

* src
* type
* defer
* async

**defer**
添加了`defer` 属性的脚本会等待页面加载并解析完成后执行（即在浏览器解析到`</html>`之后）。并且多个延迟脚本会根据顺序执行。

```html
<head>
    <script defer="defer" src="1.js"></script>
    <script defer="defer" src="2.js"></script>
</head>
```

> 兼容性：IE6.0+ 、FF3.5、SF5、CH1+

**async**
与延迟脚本 `defer` 相同，异步脚本 `async` 也会等待页面加载并解析完成后执行，只是多个异步脚本其执行顺序并不相同。
```
<head>
    <script async src="1.js"></script>
    <script async src="2.js"></script>
</head>
```
> 兼容性：IE10.0+ 、FF3.6、SF5、CH1+