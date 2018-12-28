# demo4

## 混合强调样式

基本强调样式

- 加粗
**加粗**
- 倾斜
*倾斜*
- 删除
~~删除~~

混合强调样式
- 加粗倾斜
***加粗倾斜***
- 倾斜删除
*~~倾斜删除~~*
- 倾斜删除
*~~倾斜删除~~*

- 加粗倾斜删除
***~~加粗倾斜删除~~***

## 图片链接

基本文字链接
<!--[链接文字](URL)-->
[百度](http://www.baidu.com)

基本图片
<!--![alt](url text)-->
![GitHub](https://avatars1.githubusercontent.com/u/46068099?s=460&v=4)

![GitHub][header]

图片链接
[![GitHub](https://avatars1.githubusercontent.com/u/46068099?s=460&v=4)](http://www.baidu.com)  

[![GitHub][header]][baidu]

## 引用链接

基本引用链接的用法：

good:
[百度][baidu]
[百度网站][baidu]

low:
[百度]
[百度网站]


<!--以下是引用式链接 -->

[baidu]: http://www.baidu.com
[header]: https://avatars1.githubusercontent.com/u/46068099?s=460&v=4
[百度]: http://www.baidu.com
[百度网站]: http://www.baidu.com


## 多级列表
- 问题1：如何打断：空行不行，需要文字段落
- 问题2：打断的列表，如何续上

无序列表
- item1
	- item1.1
	- item1.2
- item2

	asdasd（在段落中续上列表）

- item3

有序列表
1. item1
	1. item1.1
	2. item1.2
2. item2

asdasd（在段落中添加文字，可打断列表）

3. item3
4. item4