
- “>”表示在列表中使用引用
	> 这是一个引用
	> 可以有多行，但只需要在第一行敲“>”即可
	> > 还可以进行嵌套
	> > - 同时也支持嵌套列表
	> >> 1. 包括有序、无序列表
	>
	> 详细信息请参考 [官方文档](https://example.com)
	> 
	> &#x2139;&#xfe0f; **提示**  
	> 首次使用需要进行账户验证，验证邮件已发送到您的邮箱。

---

- 代码区块使用 **4 个空格**或者一个**制表符（Tab 键）**。

正常文本

	这里是代码块

正常文本

或者也可以使用三个反引号表示代码块

```python
import numpy
def func():
	print("hello")
```

也可以显示代码差异

```diff
function calculateTotal(items) {
-   let total = 0;
+   let total = 0.0;
    
    for (let item of items) {
-       total += item.price;
+       total += parseFloat(item.price);
    }
    
+   // 保留两位小数
+   total = Math.round(total * 100) / 100;
    return total;
}
```

---

- 链接的常用表示如下
```
[链接名称](链接地址)
[链接文字](链接地址 "可选的标题")
<链接地址>
[发送邮件](mailto:example@email.com)
[拨打电话](tel:+86-138-0013-8000)
```
[图片](https://www.runoob.com/wp-content/uploads/2019/03/49E6CB42-F780-4DA6-8290-DC757B51FB9A.jpg)
[图片](https://www.runoob.com/wp-content/uploads/2019/03/49E6CB42-F780-4DA6-8290-DC757B51FB9A.jpg "这是一张图片")
<https://www.runoob.com>
[发送邮件](mailto:example@email.com)
[拨打电话](tel:+86-138-0013-8000)
- 参考式链接将链接定义与使用分离
```
markdown[链接文字][参考标签]
[参考标签]: URL "可选标题"
```
这里使用1作为[参考标签][1]
我喜欢使用 [GitHub][] 来管理代码。

---
- [锚点链接](#锚点链接介绍)

# 锚点链接介绍
锚点链接用于在同一文档内跳转，特别适合长文档的导航

---
- 图片
```
![替代文字](图片路径)
![替代文字](图片路径 "图片标题")
```
![示例图片](https://img.cdn1.vip/i/6a0aaa7c35357_1779083900.jpg)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

---
- [表格语法直接看RUNOOB教程](https://www.runoob.com/markdown/md-table.html)

| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |
| he | ll | o |

---
- [html技巧](https://www.runoob.com/markdown/md-advance.html)、[图表绘制](https://www.runoob.com/markdown/md-draw.html)、[数学公式](https://www.runoob.com/markdown/md-math.html)

[GitHub]: https://github.com
[1]: https://www.runoob.com "这是一个网站"