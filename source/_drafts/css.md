<!--
 * @Description: 
 * @Author: chenlong
 * @Date: 2020-07-23 13:18:04
 * @LastEditTime: 2021-01-18 19:36:26
 * @LastEditors: chenlong
-->
---
title: css
---

```css
//清除浮动
xxx:after {
  content: '.';
  clear: both;
  width: 0;
  height: 0;
  visibility: hidden;
  overflow: hidden;
  display: block;
}
```

### 外边距合并

https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing

### 垂直居中

top: 50%;
transform: translateY(-50%);

### 水平居中

left: 50%
transform: translateX(-50%);

### inline 类型的 flex 容器

    display: inline-flex;

## string

typeof new String('hello') // object

'11'.length = 2 // 是因为js做了装箱，将 '11'字符串转成了对象

## 焦点获取

 a 标签没有href属性是不能获取标签的
