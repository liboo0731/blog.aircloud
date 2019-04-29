---
title: 【web】实现表格表头固定内容滚动效果
date: 2018-10-25 08:56:12
tags:
    - web 
    - html
    - css
---

> ## CSS table-layout 属性

## 浏览器支持

所有浏览器都支持 table-layout 属性。

注释：任何的版本的 Internet Explorer （包括 IE8）都不支持属性值 "inherit"。

## 定义和用法

tableLayout 属性用来显示表格单元格、行、列的算法规则。

#### 固定表格布局：

固定表格布局与自动表格布局相比，允许浏览器更快地对表格进行布局。

在固定表格布局中，水平布局仅取决于表格宽度、列宽度、表格边框宽度、单元格间距，而与单元格的内容无关。

通过使用固定表格布局，用户代理在接收到第一行后就可以显示表格。

#### 自动表格布局：

在自动表格布局中，列的宽度是由列单元格中没有折行的最宽的内容设定的。

此算法有时会较慢，这是由于它需要在确定最终的布局之前访问表格中所有的内容。

#### 说明

该属性指定了完成表布局时所用的布局算法。固定布局算法比较快，但是不太灵活，而自动算法比较慢，不过更能反映传统的 HTML 表。

| 默认值：          | auto                               |
| ----------------- | ---------------------------------- |
| 继承性：          | yes                                |
| 版本：            | CSS2                               |
| JavaScript 语法： | *object*.style.tableLayout="fixed" |

## 可能的值

| 值        | 描述                                         |
| --------- | -------------------------------------------- |
| automatic | 默认。列宽度由单元格内容设定。               |
| fixed     | 列宽由表格宽度和列宽度设定。                 |
| inherit   | 规定应该从父元素继承 table-layout 属性的值。 |

## 应用实例

```html
<!DOCTYPE HTML>
<html>  
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>转载自·威易网CSS教程</title>
<style>
table tbody {
	display:block;
	height:195px;
	overflow-y:scroll;
}
table thead,tbody tr {
	display:table;
	width:100%;
	table-layout:fixed;
}
table thead {
	width:calc( 100% - 1em )
}
table thead th {
	background:#ccc;
}		
</style>
</head>
<body>
	<table width="80%" border="1">
		<thead>
			<tr>
				<th>姓名</th>
				<th>年龄</th>
				<th>出生年月</th>
				<th>手机号码</th>
				<th>单位</th></tr>
		</thead>
		<tbody>
			<tr>
				<td>张三</td>
				<td>18</td>
				<td>1990-9-9</td>
				<td>13682299090</td>
				<td>阿里巴巴</td></tr>
			<tr>
				<td>张三封</td>
				<td>18</td>
				<td>1990-9-9</td>
				<td>13682299090</td>
				<td>阿里巴巴与四十大盗</td></tr>
			<tr>
				<td>张小三</td>
				<td>18</td>
				<td>1990-9-9</td>
				<td>13682299090</td>
				<td>腾讯科技</td></tr>
			<tr>
				<td>张三</td>
				<td>18</td>
				<td>1990-9-9</td>
				<td>13682299090</td>
				<td>浏阳河就业</td></tr>
			<tr>
				<td>张三疯子</td>
				<td>18</td>
				<td>1990-9-9</td>
				<td>13682299090</td>
				<td>阿里巴巴</td></tr>
			<tr>
				<td>张三</td>
				<td>18</td>
				<td>1990-9-9</td>
				<td>13682299090</td>
				<td>阿里巴巴</td></tr>
			<tr>
				<td>张大三</td>
				<td>18</td>
				<td>1990-9-9</td>
				<td>13682299090</td>
				<td>阿里巴巴</td></tr>
			<tr>
				<td>张三五</td>
				<td>18</td>
				<td>1990-9-9</td>
				<td>13682299090</td>
				<td>阿里巴巴</td></tr>
			<tr>
				<td>张刘三</td>
				<td>18</td>
				<td>1990-9-9</td>
				<td>13682299090</td>
				<td>阿里巴巴</td></tr>
			<tr>
				<td>张三</td>
				<td>18</td>
				<td>1990-9-9</td>
				<td>13682299090</td>
				<td>阿里巴巴</td></tr>
		</tbody>
	</table>
</body>
</html>

```

