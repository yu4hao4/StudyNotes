# Serializable接口

## 1. 概念

java的序列化是 jdk1.1 时引入的特性，用于将 Java 对象转换为字节数组，便于存储或传输。并且，仍然可以将字节数组转换回 Java 对象原有的状态。

序列化的思想是 “冻结” 对象状态，然后写到磁盘或者在网络中传输；反序列化的思想是 “解冻” 对象状态，重新获得可用的 Java 对象

![1569599653299](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569599653299.png)

可以看到 **Serializable** 是一个空接口

## 2.演示

创建一个 **User** 实体类

![1569599870159](C:\Users\Lenovo\Desktopimages\Serializable\1569599870159.png)

创建测试类，通过 ObjectOutputStream 将 user 对象写入到文件当中（序列化）；再通过 ObjectInputStream 将 user 对象从文件中读取出来（反序列化）

**测试类**

![1569625181689](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569625181689.png)

**运行结果**

![1569625644418](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569625644418.png)

当我们点进报错信息里面

![1569625417726](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569625417726.png)

可以看到，最后是判断了当前对象，是不是 Serializable 这个接口的实现类。如果不是就会抛出 NotSerializableException 这个异常。

**接下来，我们实现 Serializable 之后**

![1569625720199](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569625720199.png)

运行结果:

![1569625823212](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569625823212.png)

可以看到，我们成功的将对象存储到文件中后，又将其取了出来

### static和transient关键字

添加 static 和transient 修饰的变量

![1569634938888](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569634938888.png)

再次测试方法：

![1569635148617](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569635148617.png)

可以看到，transient 修饰的属性，在保存的时候并没有将其值保存下来。（在反序列化之后，会将其修饰的属性，变为默认值）

而 static 修饰的字段 属于类的状态，而序列化则是保存对象的状态，所以，序列化并不会保存 static 修饰的字段。

![1569635516447](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569635516447.png)

![1569635526020](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569635526020.png)

![1569635539789](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569635539789.png)

### Externalizable接口

![1569635658079](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569635658079.png)

序列化还可以选择实现 Externalizable接口 ，但是必须要实现 writeExternal 和 readExternal 方法。

接下来，我们再次测试序列化。

![1569635863229](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569635863229.png)

可以看到结果是：

![1569635976771](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569635976771.png)

原因是，我们必须要重写 Externalizable接口 让我们实现的方法

![1569636086134](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569636086134.png)

就是这样。

再次测试：

![1569636111518](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569636111518.png)

结果正常

### serialVersionUID

首先，给对象 加一条属性

![1569638851421](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569638851421.png)

然后，运行测试方法

![1569638879739](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569638879739.png)

![1569638927062](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569638927062.png)

结果如上。

这时，我们把 User1 类里面的 serialVersionUID 进行修改。进行反序列化操作

![1569641193792](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569641193792.png)

![1569641208117](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569641208117.png)

报错显示 serialVersionUID 不一致：

![1569641251314](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569641251314.png)

跟进源码：

![1569641346706](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569641346706.png)

大概意思就是，在反序列化的时候，会比较文件中的  serialVersionUID   和当前类中的  serialVersionUID  如果不一致，则会抛出 stream classdesc serialVersionUID = 1, local class serialVersionUID = 2 异常



如果我们不对类的 serialVersionUID  进行设置，JVM会使用自己的算法生成默认的 serialVersionUID  。



**将 serialVersionUID  改回后：**

![1569641577988](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569641577988.png)

![1569641599711](C:\Users\Lenovo\Desktop\StudyNotes\images\Serializable\1569641599711.png)

和上文一样