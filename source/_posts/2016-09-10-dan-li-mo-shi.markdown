---
layout: post
title: "单例模式"
date: 2016-09-10 14:40:19 +0800
comments: true
categories: 
---
单例模式7种写法

1.饿汉式

public class Singleton{
    private static Singleton instance=new Singleton();
    private Singleton{
    }
    
    public static Singleton getInstance(){
        return instance;
    }
}
2.懒汉式(非线程安全)

public class Singleton{
    private static Singleton;
    private Singleton{
    }
    
    public static Singleton getInstance(){
        if(instance==null){
            instance=new Singleton();
        }
        return instance;
    }
}
不能在多线程下工作

3.懒汉式（线程安全）

public class Singleton{
    private static Singleton;
    private Singleton{
    }
    
    public static synchronized Singleton getInstance(){
        if(instance==null){
            instance=new Singleton();
        }
        return instance;
    }
}

public class Singleton{
    private static Singleton;
    private Singleton{
    }
    
    public static  Singleton getInstance(){
        synchronized(Singeleton.class){
        if(instance==null){
            instance=new Singleton();
        }
        }
        return instance;
    }
}
效率低下

4.饿汉式（变种）

public class Singleton {  
    private Singleton instance = null;  
    static {  
    instance = new Singleton();  
    }  
    private Singleton (){}  
    public static Singleton getInstance() {  
    return this.instance;  
    }  
}
静态内部类

public class Singleton {  
    public static class SingletonHolder{
        private static final Singeleton INSTANCE=new Singleton();
    }
    public Singleton(){
    }
    public static Singleton getInstance(){
        return SingletonHolder.INSTANCE;
    }
}
双重校验锁

public class Singleton{
    private  volatile static Singleton instance=null;
    
    private Singleton(){
    }
    
    public static Singleton getInstance(){
        if(instance==null){
            synchronized(Singleton.class){
                if(instance==null){
                    instance=new Singleton();
                }    
            }
        }
        return instance;
    }
}
第一个条件是说，如果实例创建了，那就不需要同步了，直接返回就好了。 不然，我们就开始同步线程。 第二个条件是说，如果被同步的线程中，有一个线程创建了对象，那么别的线程就不用再创建了。

使用 volatile 有两个功用: 1）这个变量不会在多个线程中存在复本，直接从内存读取。

2）这个关键字会禁止指令重排序优化。也就是说，在 volatile 变量的赋值操作后面会有一个内存屏障（生成的汇编代码上），读操作不会被重排序到内存屏障之前。

枚举

public enum Singleton{
   INSTANCE;
}
