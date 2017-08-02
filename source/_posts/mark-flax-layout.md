---
title: Flax Layout Mark
date: 2017-07-27 16:27:10
tags: fount
---

最近在学习Flax Layout，发现几张图能轻松记住Flax的布局问题，顺便 Mark 下吧：

![flax_layout_header](https://raw.githubusercontent.com/dxyoo7/dxyoo7.github.io/hexo/flax_layout_header.jpg)

```Js
//flexDirection可以决定布局的主轴。子元素是应该沿着水平轴(row)方向排列，
//还是沿着竖直轴(column)方向排列呢？默认值是竖直轴(column)方向。
 
flaxDirection:	
'row',           //横向布局
'column',        //纵向布局
'row-reverse',   //横向反转（从又至左)
'column-reverse' //纵向反转（从下至上)
```
![flax_layout_direction](https://raw.githubusercontent.com/dxyoo7/dxyoo7.github.io/hexo/resources/flax_layout_direction.jpg)

```Js
//在组件的style中指定justifyContent可以决定其子元素沿着主轴的排列方式。
//子元素是应该靠近主轴的起始端还是末尾段分布呢？亦或应该均匀分布？

justifyContent:	//Flax Direction 有这些属性可选
'flex-start',           //从主轴起始位置开始布局（左 或 上)
'center',               //主轴居中显示
'flex-end',             //从主轴结束位置开始布局 (右 或 下)
'space-around‘          //平分主轴
'space-between'         //间隙平分
```
![flax_layout_justfy_content](https://raw.githubusercontent.com/dxyoo7/dxyoo7.github.io/hexo/resources/flax_layout_justfy_content.jpg)


```Js
//在组件的style中指定alignItems可以决定其子元素沿着次轴
//（与主轴垂直的轴，比如若主轴方向为row，则次轴方向为column）的排列方式。
//子元素是应该靠近次轴的起始端还是末尾段分布呢？亦或应该均匀分布？
alignItem:  //有这些属性
‘flex-start’,
'center',
'flex-end',
'stretch'
```
![flax_layout_align_item](https://raw.githubusercontent.com/dxyoo7/dxyoo7.github.io/hexo/resources/flax_layout_align_item.jpg)


![flax_layout_flax_wrap](https://raw.githubusercontent.com/dxyoo7/dxyoo7.github.io/hexo/resources/flax_layout_flax_wrap.jpg)

