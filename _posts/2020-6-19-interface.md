---
layout: post
title: Java：接口回调
tags: java  
---


> 在nio网络模型中大量使用异步模式进行编程，其实使用的也就是接口回调的技术，接口回调即实现两个对象之间的通信。以下为在安卓中实现的接口回调实例。

#  目录
* 目录
{:toc}
# 什么叫做接口回调 ？ 

 核心思想就是利用java的动态绑定，利用多态的特性去调用接口的实现方法。 使用用法：在A类中定义接口方法， 在B类中实现A中的接口方法，然后将自己的实现方法注册到A类中，这样在A类中调用自己的接口就相当与调用B类中定义的方法，这样便能够实现AB类中的数据交互，因为方法是在B类中进行实现的，也相当于把A类接口中的参数传递给B类，这样B类便可以进行处理A中的参数，并且还能将处理后的参数传递给A类。实现双向的数据交互。

# 接口回调的作用 ？

可以简单的理解为，两个类之间进行数据的传输，就是相当于一个类调用另外一个类的方法，同时实现数据的传递。最常用的方法就是安卓中按钮点击事件的触发，异步的网路事件的调用。比如说netty如何实现nio。

# 实现的方法

例子：安卓上自定义键盘。通过回调函数获取用户每次按压的按键是哪个，并且调用相应的处理函数。

 NumberKeyboardView.java 中接口的定义。 接口获取到的数据。

1.在A类中定义接口和接口注册函数。

```java
public class NumberKeyboardView {
    //1 定义接口类。
    private OnNumberClickListener onNumberClickListener; 

    //2 实现接口的注册方法，相当于进行绑定。
    public void  setOnNumberClickListener(OnNumberClickListener onNumberClickListener){
        this.onNumberClickListener = onNumberClickListener;
    }

    // 3 调用接口中的方法。
    if (onNumberClickListener != null) {
        if (number != null) {
            onNumberClickListener.getUserPressure(event.getPressure()); //就是说在这个类中会自动的调用这个接口，自己没有实现内容，但是继承的类会实现，之后调用实例化中的代码即可。
            if (number.equals("delete")) {
                onNumberClickListener.onNumberDelete(); //相同的调用这个接口。

            } else {
                onNumberClickListener.onNumberReturn(number);
            }
        }
    } 

    // 定义接口对象。
    public interface OnNumberClickListener {
            //回调点击的数字
            public void onNumberReturn(String number);

            //删除键的回调
            public void onNumberDelete();

            //获取按键压力的回调
            public void getUserPressure(float pre);
        }
}

```

2.在B类中继承并且实现接口，并获取父类对象实例进行注册接口。

```java
//1 首先是继承接口。
public class MainActivity extends Activity implements NumberKeyboardView.OnNumberClickListener {
	
    // 获取A类实例。
    private NumberKeyboardView mNkvKeyboard;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        initView();
    }

    private void initView() {
        
        // 调用A类中的注册函数
        mNkvKeyboard.setOnNumberClickListener(this);
    }
 
	// 2 实现的接口。
    @Override   //接口文件
    public void onNumberReturn(String number) {
        str += number;
        setTextContent(str);
    }

    @Override  //接口文件
    public void onNumberDelete() {  //这个就是自定义的代码会自动的绑定在 NumberKeyboardView 这个类中。在这个进行调用。
        if (str.length() <= 1) {
            str = "";
        } else {
            str = str.substring(0, str.length() - 1);
        }
        setTextContent(str);
    }

    @Override  //接口文件
    public void getUserPressure(float pre) {  //pre 是接口传递出来的数据， 之后自己进行操作即可。
        preStr = pre + "";
        mPressure.setText(preStr);
    }

    private void setTextContent(String content) {
        mTvText.setText(content);
    }
}
```

通过以上的方式便能实现在A类中进行调用接口方法的时候其实是调用的B类中的函数，实现数据的传输。这样其实也算是一种异步的数据传输，不需要B类去一直检查A类有没有数据需要传输。 

从以上的例子可以看出来，在使用netty的异步方法时候，例如Future，也是使用的相同的思想去实现，即Channel中处理还数据以后就会调用Future函数。