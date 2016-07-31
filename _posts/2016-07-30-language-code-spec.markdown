---
layout: post
title:  "常用Code编码规范 "
date:   2016-07-30 16:37:11 +0800
categories: summary 
---

编程语言涉及到的命名主要包含filename、package、class、method（private、protected、public）、function、variable、constant、parameter

记录一下最近用到的语言规范，包含Go、Java、Scala、Python等
<pre> 

<table class="table table-bordered table-striped table-condensed">
   <tr>
      <td></td>
      <td><Strong>Go</Strong></td>
      <td><Strong>Java</Strong></td>
      <td><Strong>Scala</Strong></td>
      <td><Strong>Python</Strong></td>
   </tr>
   <tr>
      <td><Strong>package</Strong></td>
      <td>encoding/json</td>
      <td>net.frontfree.javagroup</td>
      <td>jxl.write.{WritableCell, Number, Label}</td>
      <td>socket</td>
   </tr>
   <tr>
      <td><Strong>class/struct</Strong></td>
      <td>ClassName</td>
      <td>UserReckoning</td>
      <td>-</td>
      <td>AdStats</td>
   </tr>
   <tr>
      <td><Strong>method/variable</Strong></td>
      <td>Name(public),Name(private)</td>
      <td>sendMessge() / userName</td>
      <td>-</td>
      <td>__private_var(private) </td>
   </tr>
   <tr>
      <td><Strong>function</Strong></td>
      <td>FuncName</td>
      <td>sendMessge()</td>
      <td>-</td>
      <td>get_name() / __get_name()(私有)</td>
   </tr>
   <tr>
      <td><Strong>variable</Strong></td>
      <td>驼峰</td>
      <td>globalID</td>
      <td>-</td>
      <td>this_is_a_var /_var (私有） / __doc__(专有）</td>
   </tr>
   <tr>
      <td><Strong>const</Strong></td>
      <td>大写+'_'</td>
      <td>MAX_VALUE</td>
      <td>-</td>
      <td>COLOR_WRITE</td>
   </tr>
   <tr>
      <td><Strong>type</Strong></td>
      <td>int 小写</td>
      <td>Int 大写</td>
      <td>-</td>
      <td></td>
   </tr>
   <tr>
      <td><Strong>parameter</Strong></td>
      <td>小写驼峰</td>
      <td>小写驼峰</td>
      <td>-</td>
      <td>下划线</td>
   </tr>
   <tr>
      <td><Strong>format tool</Strong></td>
      <td>go fmt</td>
      <td></td>
      <td></td>
      <td></td>
   </tr>
</table>
</pre>
