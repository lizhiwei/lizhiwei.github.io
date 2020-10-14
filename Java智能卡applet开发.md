<!--
author: lizhiwei
head: 
date: 2020-1-14
title: Java智能卡applet开发
tags: sim,java
images: 
category: sim
status: publish
summary: Java智能卡applet开发
-->



每个Java卡的Applet都继承自 javacard.framework.Applet类，Applet类为抽象类，也就是
只定义了Applet执行过程中所需要的方法，而没有具体实现。


       package javacard.framework;
       
       import com.sun.javacard.impl.PrivAccess;
       
       public abstract class Applet {
         private PrivAccess thePrivAccess = PrivAccess.getPrivAccess();
         
         public static void install(byte[] bArray, short bOffset, byte bLength) throws ISOException {
           ISOException.throwIt((short)27265);
         }
         
         public boolean select() {
           return true;
         }
         
         public void deselect() {}
         
         public Shareable getShareableInterfaceObject(AID clientAID, byte parameter) {
           return null;
         }
         
         protected final void register() throws SystemException {
           this.thePrivAccess.register(this);
         }
         
         protected final void register(byte[] bArray, short bOffset, byte bLength) throws SystemException {
           if (bLength < 5 || bLength > 16)
             SystemException.throwIt((short)1); 
           this.thePrivAccess.register(this, bArray, bOffset, bLength);
         }
         
         protected final boolean selectingApplet() {
           return this.thePrivAccess.selectingApplet();
         }
         
         public abstract void process(APDU paramAPDU) throws ISOException;
       }



按照Applet建立和执行过程中被JCRE调用的顺序，分别为install register select process和deselect方法。

终端 首先发送 Select AID  APDU到Java卡，JCRE将先取消当前激活的Applet(如果用了逻辑通道，其他逻辑通道的Applet不会被取消，只取消本通道的), 并调用该Applet的deselect方法。再根据Select AID APDU命令中的AID，查找已经注册的Applet, 如果存在，则调用对应Applet的select方法，并将该Applet设置为当前激活的Applet.

当Applet激活后，终端发送的除select之外的的所有APDU 都会有JCRE转送给Applet执行。Applet将调用process来处理这些APDU.

Applet类将process声明为 抽象方法， 因此，每个继承 javacard.framework.Applet 的Applet实例，都必须要Override（重写）

install,select,deselect和process方法都是JCRE的入口点方法，将由JCRE负责调用。同时，由于Applet类只实现了默认的功能（除了process外，process方法是个抽象方法), 所以，Applet需要增加其他功能或者对其修改，可以通过 重写(override) 特定的方法来实现。


### install()方法

用于安装一个Applet实例。在方法执行过程中，首先将创建Applet所需要的对象，然后进行初始化，为数据赋初始值;
最后调用 register方法，将Applet注册到Java卡平台中。只有成功执行了register方法， Applet install()方法才算是执行成功。
如果在install方法中，没有调用regiter或者register执行错误，则该Applet安装失败。

如果Applet安装失败，JCRE会自动执行相关操作，使Java智能卡平台恢复到install前的状态。

Applet install成功结束后，该Applet被JCRE将如到注册表中，并标记为可选择。
在此状态下，终端可以通过Select AID APDU命令，来激活已经安装的Applet


某些Applet安装可能会需要输入相关参数，用于确定创建对象的大小或者初始值。该输入参数将由 bArray[] 数组负责传递。
install()中定义的bArray数组为全局数组， 可以被任意的Applet所访问。

该数组为LV结构，格式如下：

       bArray[0]      =  Applet实例的AID长度
       bArray[1:Li]   =  Applet实例的AID
       bArray[Li+1]          =  Applet控制信息的长度
       bArray[Li+2:Li+Lc+1]  = Applet控制信息
       bArray[Li+Lc+2]            = Applet Install方法参数的个数
       bArray[Li+Lc+3:Li+Lc+La+1] = Applet Install方法输入参数

以上的LV格式中， Li, Lc, La都可能为0
如果应用需要保存bArray[]中的数据，将该数据复制到自身Applet实例中即可， 但bArray[]数组作为全局数组，该数组的引用，不能被存储到类变量或者对象实例中。


#### 注册到javacard

  提供了两个register方法

       protected final void register() throws SystemException
       pretected final void register(byte[] bArray, short bOffset, byte bLength) throws SystemException

前者 使用 Applet的Class AID作为Applet的Instance AID, 注册到 JCRE.
   Class AID 是 Applet的class文件 转换为  cap文件时，由用户指定。


后者 则使用特定的数据作为 Instance AID，将Applet注册到JCRE. 所带的参数用于传递AID数据， 其中bArray为包含AID的数组
bOffset为AID在数组中的偏移， bLength为AID数据的长度



### select()方法

在Applet被选中之前，一直处于挂起(suspend)，当JCRE收到一个SELECT命令， 且该SELECT命令的数据段，和某个Applet实例化AID一致
该应用将被选中， JCRE将会调用Applet的select方法。当 Manage Channel Open命令调用时， 在该通道上激活的Applet的select方法，也会被调用。


在调用 select 方法前， JCRE将会 取消选择当前活动的Applet, JCRE通过调用Applet.deselect或者MultiSelectable.deselect来取消当前应用。

如果Applet需要在执行SELECT命令是，做某些初始设置，Applet可以重写 Applet.select()方法，在重写的实现里，进行相关状态设置和判断。
若当前状态满足要求， 则select方法返回true. 否则返回false 或者抛出异常。


如果select 方法返回false 或者抛出 异常， JCRE将向终端 返回 0x6999, 表示Applet选择失败。
     如果返回成功， 则表示Applet可以接收终端发出命令， 并且该条选择命令也会送到 Applet.process()方法执行。

有一个API方法：Applet.selectingApplet()，功能是，如果当前应用已经被选择，  则该方法返回true, 否则返回false


### process()方法

当Applet被选择后， 由终端发送的命令 将会交付给 Applet.process()方法执行。Applet.process()方法为虚方法，每一个Applet
必须要 重写(override)该方法，在方法中定义自己的命令执行流程，以此来完成引用和同终端的命令交互过程。
通常， process()方法将会首先检查输入命令结构是否正确，然后根据INS字节来进入不同的处理流程，最后返回执行结果。

process()执行过程中，Applet会主动抛出ISOException，该异常会JCRE自动不或，并将ISOException.Reason作为状态字返回给终端。
对于其他异常， JCRE不能自动捕获。
  如果Applet没有自己去捕获， JCRE会返回ISO7816.SW_UNKOWN，即 6F00给终端，表示应用执行过程中出现了 未知异常。



### deselect() 方法

   在一个新的Applet被选中之前， JCRE将调用 Applet.deselect方法 或者 MultiSelect.deselect来取消选择。有时候，将要选择的应用
   同当前已选应用是同一个应用， JCRE仍然abuilding先 取消选择，再重新选择它。

   Applet被取消的时候，做一些清空操作，就需要重写deselect方法。






