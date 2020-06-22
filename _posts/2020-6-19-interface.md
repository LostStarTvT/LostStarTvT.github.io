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

 核心思想就是利用java的动态绑定， 即父类可以调用子类的方法。父类进行定义接口，然后在方法中只是调用一样接口中的方法， 具体的方法实现是在子类进行实现，但是需要在父类中定义一个接口注册函数，然后才能实现子类与父类之间的关联。

# 接口回调的作用 ？

可以简单的理解为，两个类之间进行数据的传输，就是相当于一个类调用另外一个类的方法，同时实现数据的传递。最常用的方法就是安卓中按钮点击事件的触发，异步的网路事件的调用。比如说netty如何实现nio。

# 实现的方法

例子：安卓上自定义键盘。通过回调函数获取用户每次按压的按键是哪个，并且调用相应的处理函数。

 NumberKeyboardView.java 中接口的定义。 接口获取到的数据。

1.在父类中进行定义接口和接口注册函数。

```java
NumberKeyboardView.java

//1 定义要用到的类为对象。
private OnNumberClickListener onNumberClickListener; 

//2 实现接口的注册方法，相当于进行绑定。
public void  setOnNumberClickListener(OnNumberClickListener onNumberClickListener){
    this.onNumberClickListener = onNumberClickListener;
}

// 3 进行调用改接口对象中的方法。
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
```

2.子类进行继承接口，并获取父类对象实例进行注册接口。

```java
//1 首先是继承接口。
public class MainActivity extends Activity implements NumberKeyboardView.OnNumberClickListener {

    private NumberKeyboardView mNkvKeyboard;
    private TextView mTvText;
    private TextView mPressure;
    private String str = "";
    private String preStr = "";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initView();
    }

    private void initView() {
        mTvText = (TextView) findViewById(R.id.am_tv_text);
        mPressure = (TextView) findViewById(R.id.am_tv_text_press);

        mNkvKeyboard = (NumberKeyboardView) findViewById(R.id.am_nkv_keyboard);
        
        //进行注册和绑定接口。 调用该类中的绑定接口方法。
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

 在父类中直接的调用一个虚的接口， 在子进行实现接口的内容， 根据java的多态性，会在运行时直接绑定子中实现的代码，也即是向上绑定。

所以说接口这个东西会很好用。

从以上的例子可以看出来，在使用netty的异步方法时候，例如Future。