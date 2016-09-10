---
layout: post
title: "InnerClass"
date: 2015-09-20 19:24:08 +0800
comments: true
categories: "java inner class"
---
##内部类
内部类是定义在一个类内部的类，特点：
1.内部类可以直接访问外部内中的成员变量。内部类要访问局部变量必须将局部变量用final修饰。
2.内部类有两种定义方式，一种定义在方法体内，另外一种定义在方法体外。区别在于在方法体外了一定以3内部类的访问类型，可以使public,protected以及默认的。而不能在方法体内定义内部类的访问类型。
3.内部类不能定义静态成员。
4.静态内部类，在方法外部定义的内部类前面可以加上static关键字，从而成为Staic Nested Class,不再具备内部类的特征了。
简单写一个匿名内部类例子(线程)
<pre class="brush:java;gutter:true;">
	public class Outer{
	pubic void start(){
	new Tread(
	new Runnable(){
	public void run(){}
	}).start();
}
}
</pre>
