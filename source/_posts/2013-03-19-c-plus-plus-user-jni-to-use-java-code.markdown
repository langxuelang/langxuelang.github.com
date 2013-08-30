---
layout: post
title: "c++利用jni调用java的代码"
date: 2013-03-19 10:07
comments: true
categories: 交叉编译
---
#起因#
在公司的一个项目中，需要在cocos2d-x中调用android中java的代码，于是自己想了想，还是学一下比较好，这样能更加加深我对于编程的理解。
#实践#
于是自己开始在百度与google上搜索，终于搜索一篇关于cocos2d-x关于jni调用的文章。于是还是延续我以前学习的方法，直接将代码抄一遍，然后等到抄完之后也就理解的差不多了。
[原文在这里](http://www.cnblogs.com/Androider123/archive/2012/10/19/2730644.html)
后来有搜索了一些，基本上就是抄这篇文章的。于是自己也开始写，将需要引用的一些库导入项目的库目录中。
接下来就是条件编译，只有在android平台上的时候，我们才进行编译这段代码。
<!-- more -->
为了方便，我将代码贴了出来，如下:

	#if(CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID)//条件编译语句
        {
                const char *message = "hello,xuelang";//需要传递到Java层的字符串
                JniMethodInfo methodInfo;
                bool isHave = JniHelper::getStaticMethodInfo(methodInfo,
                        "com/kengdie/test/Test",//需要调用的Java文件
                        "showMessageFromNative",//调用的方法名
                        "(Ljava/lang/String;)V");//参数
                if(isHave){
                        CCLog("the showMessage method is exits");
                        jstring StringArg = methodInfo.env->NewStringUTF(message);
                        methodInfo.env->CallStaticVoidMethod(methodInfo.classID, methodInfo.methodID, StringArg);
                }else{
                        CCLog("the showMessage method is not exits");
                }
        }
	#endif
在这里原文没有解释清楚一些东西，我下面给大家解释一下。
首先，我们会奇怪，为什么参数会是
>(Ljava/lang/String;)V

这种形式，分别代表什么含义，别着急，让我下面慢慢给大家说：
jni调用中，定义了一些关键字（类型）的签名（简写）

"I" 表示整数

"B" 表示byte

"C" 表示 char

"D" 表示 double

"F" 表示float

"J" 表示long

"S" 表示short

"V" 表示void

"Z" 表示boolean

"Ljava/lang/String;" 表示String
>(Ljava/lang/String;)V

中，括号里面代表参数类型，括号外面代表返回值，这下大家应该明白了，其实调用的java函数对应的类型应该是void showMessageFromNative(String message)这样的形式。
下面再写几种形式，让大家练习一下。
>(ILjava/lang/String;)V
>(Ljava/lang/String;Ljava/lang/String;)V
>(ILjava/lang/String;)V

提醒大家的是，参数与参数之间不需要任何分割符，但是Ljava/lang/String;中的分号是自带的不能省去。

了解完这个后，getStaticMethodInfo函数是用来初始化JniMethodInfo methodInfo;变量的，得到一些函数的基本信息，然后执行 methodInfo.env->CallStaticVoidMethod(methodInfo.classID, methodInfo.methodID, StringArg);便可以实现对java函数的调用了。有人说，为什么getStaticMethodInfo与CallStaticVoidMethod都是得到静态函数与执行静态函数，难道说，不能执行对象方法吗？
答案是当然可以，不过，这里便出现一个问题，我们该如何得到类的对象呢，只有这样才能用类的对象去执行非静态函数。（详细的大家可以跟踪到jni源代码中去一探究竟，等到里面你会发现，jni其实就已经实现了功能了，为啥还有一个jnihelper呢，其实jnihelper是cocos2d-x为大家封装好的一个类，让大家用起来更方便）

解决方法是这样的：
大家可以先在java里面写一个静态成员函数，然后让其返回这个类的一个实例，这样，我们就可以利用getStaticMethodInfo与CallStaticVoidMethod在c++中得到一个jobject类型的对象，然后调用
jdouble CallDoubleMethod(jobject obj, jmethodID methodID, ...)将之前得到的jobject传进去
于是便可以执行这个类对象的成员函数了。




