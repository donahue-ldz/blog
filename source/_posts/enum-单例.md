title: "enum 单例"
date: 2015-04-07 16:42:37
categories: java
tags: java
---

实现单例方法很多，为什么enum实现是比较好的选择呢
1. Enum 构造函数是私用的，外部不能随意调用
2. Enum 对象是线程安全的
3. 原生态的序列化有jDK保证，一般方法实现单例模式的时候如果要出现序列化的需求的时候将调用对象的构造函数去反序列化，这个就打破了单例的初衷
但是可以重写

```
protected Object readResolve() throws ObjectStreamException {      
   return INSTANCE; 
}
```
