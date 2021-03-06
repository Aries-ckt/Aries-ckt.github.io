---
layout: post
title: "“Java序列化”"
date: 2015-08-15 12:31:15 +0800
comments: true
categories: Java基础知识
---
#Java对象的序列化
当两个进程在进行远程通信时，彼此可以发送各种类型的数据。无论是何种类型的数据，都会以二进制序列的形式在网络上传送。发送方需要把这个Java对象转换为字节序列，才能在网络上传送；接收方则需要把字节序列再恢复为Java对象。
　　只能将支持 java.io.Serializable 接口的对象写入流中。每个 serializable 对象的类都被编码，编码内容包括类名和类签名、对象的字段值和数组值，以及从初始对象中引用的其他所有对象的闭包。
##1.概念
序列化：把Java对象转换为字节序列的过程。
反序列化：把字节序列恢复为Java对象的过程。
##2.用途
对象的序列化主要有两种用途：
　　1） 把对象的字节序列永久地保存到硬盘上，通常存放在一个文件中；
　　2） 在网络上传送对象的字节序列。
##3.对象的序列化
###3.1对象序列化API
java.io.ObjectOutputStream代表对象输出流，它的writeObject(Object obj)方法可对参数指定的obj对象进行序列化，把得到的字节序列写到一个目标输出流中。只有实现了Serializable和Externalizable接口的类的对象才能被序列化。

　　java.io.ObjectInputStream代表对象输入流，它的readObject()方法从一个源输入流中读取字节序列，再把它们反序列化为一个对象，并将其返回。

###3.2示例代码
<pre class="brush:java;gutter:true;">
import java.io.*;
import java.util.Date;

public class ObjectSaver {
    public static void main(String[] args) throws Exception {
        /*其中的  D:\\objectFile.obj 表示存放序列化对象的文件*/

        
        //序列化对象
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("D:\\objectFile.obj"));
        Customer customer = new Customer("王麻子", 24);    
        out.writeObject("你好!");    //写入字面值常量
        out.writeObject(new Date());    //写入匿名Date对象
        out.writeObject(customer);    //写入customer对象
        out.close();

        
        //反序列化对象
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("D:\\objectFile.obj"));
        System.out.println("obj1 " + (String) in.readObject());    //读取字面值常量
        System.out.println("obj2 " + (Date) in.readObject());    //读取匿名Date对象
        Customer obj3 = (Customer) in.readObject();    //读取customer对象
        System.out.println("obj3 " + obj3);
        in.close();
    }
}

class Customer implements Serializable {
    private String name;
    private int age;
    public Customer(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String toString() {
        return "name=" + name + ", age=" + age;
    }
}
</pre>