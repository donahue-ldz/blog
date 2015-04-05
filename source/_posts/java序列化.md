title: "java序列化"
date: 2015-03-13 20:29:02
categories: java
tags: [序列化]
---


Java 提供了一种对象序列化的机制，该机制中，一个对象可以被表示为一个字节序列，该字节序列包括该对象的数据、有关对象的类型的信息和存储在对象中数据的类型。

将序列化对象写入文件之后，可以从文件中读取出来，并且对它进行反序列化，也就是说，对象的类型信息、对象的数据，还有对象中的数据类型可以用来在内存中新建对象。

<span style="color: #ff00ff;">由于序列化都是由jvm实现 所以可以跨平台在不同的地点序列化</span>

java的序列化有两种方法
##Serializable达到自动序列化
记住有些东西是不能或者不方便实例话的（密码，银行卡等敏感信息），此时要是自动序列化的话将是不安全的表现，但是java提供了一个关键字transient，``被transient修饰的变量将不会被序列话，也就不能被反序列化`。

`被static修饰的变量也不会被序列化`，但是反序列化时候，如果当前内存中存在静态变量的值的时候，反序列化的时候被修饰的类成员将还是会看见值，此时的值就是现在内存中的值

---

##Externalizable手动序列化
在里面重写相关的方法实现自定义序列化，此时不管是否被transient修饰都使可以哦～～～

```
public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(content);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException,
            ClassNotFoundException {
        content = (String) in.readObject();
    }
```

本文只是介绍jdk自带的序列化，还有更多更好的序列化方法将比jdk带的更加方便和效率高