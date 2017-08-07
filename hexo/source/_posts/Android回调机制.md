---
title: Android回调机制
date: 2017-05-02 15:06:00
categories: Android
tags: 回调
---
回调就是A类中调用B类的方法b，然后B类中反过来调用A类的方法a。a这个方法就是回调方法。

<!-- more -->

## 概念分析

直接看回调概念，可能会是一头雾水，这个绕口令在说啥。那么就用一个现实生活中的场景，来模拟一下回调。

有一天，老板有一个活，需要员工来完成。老板很忙，有很多其他事，不能等着员工一直到这个活干完。于是他告诉员工，干完这个活后，就去办公室通知他。过了一天，员工打电话告诉老板，活干完了。通知老板这个行为就是一种回调。

## 实际意义

那么，为什么要用到回调呢？首先，编码过程中可能会出现像上面安排活一样的异步耗时需求。其次，回调增强了代码的可扩展性。

比如我们在使用框架的时候，直接调用框架提供的API就可以了。但如果我们想让框架来调用自己写的类中方法，该如何实现呢？难道要去改框架中的代码吗？所以，优秀的框架都会提供好相关的回调接口。我们只需要实现相关的接口，即可完成注册，让框架在合适的时候调用我们自己写的类。

可以反回头再读一下回调的定义：**A类中调用B类的方法b，然后B类中反过来调用A类的方法a**。这里我们写的类就是A,框架就是B。这样，不用去修改框架B，只去改自己的类A,就可以个性化定制框架所调用的方法，增强框架的代码扩展性。

## 代码实现

下面，以老板让员工干活，员工干完通知老板为例，用代码实现这一需求。

### 回调接口

首先，定义一个回调接口，里面定义一个回调函数。

``` java
//回调接口，老板要实现此接口
public interface CallBack {
    //提供联系方法（办公室地址），用来接收员工的汇报
    public void receive();
}
```

### 老板类

然后创建回调对象，即老板类。因为员工干完活要通知老板，所以老板要实现回调接口，提供联系方法，接收员工的汇报。

``` java
//老板类，实现回调接口
public class Boss implements CallBack {
	//员工对象的引用
    private Employee employee;
    public Boss(Employee employee) {
        this.employee = employee;
    }
    //分配活给员工
    public void assign() {
        employee.setCallBack(Boss.this);
        employee.work();
    }
    @Override
    public void receive() {
        System.out.println("我知道这个活已经做完了");
    }
}
```

### 员工类

接着创建控制对象，即员工类。他必须要知道老板的地址（持有回调接口），才能跟老板进行汇报。

``` java
//员工类
public class Employee {
    public CallBack callBack = null;
    //进行注册，获取老板的联系方式
    public void setCallBack(CallBack callBack) {
        this.callBack = callBack;
    }
    //员工干活
    public void work() {
        //1.开始干活
        for (int i=0;i<10;i++) {
            System.out.println("事情的第"+i+"步干完了");
        }
        //2.通知老板
        callBack.receive();
    }
}
```

### 测试类

最后创建测试类。

``` java
public class Client {
    public void test() {
        Employee employee = new Employee();
        Boss boss = new Boss(employee);
        boss.assign();
    }
}
```

## 步骤总结

1. 定义一个回调接口CallBack。
2. 创建一个回调对象Class A，实现回调接口CallBack。
3. 创建一个控制对象Class B，含有一个参数为CallBack的方法。 
4. 回调对象Class A中包含一个控制对象Class B的引用。
5. Class A调用Class B的方法b。
6. Class B就可以在b方法中调用Class A的方法a。

## Android实例

Android中View的点击事件就用到了回调机制。下面就用这个Andriod中最常见的代码做一下简单分析。

首先是有一个回调接口OnClickListener。

``` java
//这个是View的一个回调接口  
//Interface definition for a callback to be invoked when a view is clicked.  
public interface OnClickListener {  
	//Called when a view has been clicked. 
    void onClick(View v);  
}  
```

然后是有一个控制对象View，包含一个参数为OnClickListener的方法setOnClickListener。

``` java
public class View implements Drawable.Callback, KeyEvent.Callback, AccessibilityEventSource {   
    protected OnClickListener mOnClickListener;    
    public void setOnClickListener(OnClickListener l) {  
        if (!isClickable()) {  
            setClickable(true);  
        }  
        mOnClickListener = l;  
    }  
    public boolean performClick() {  
        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);  
        if (mOnClickListener != null) {  
            playSoundEffect(SoundEffectConstants.CLICK);    
            //B类调用A类的方法onClick()，这个onClick()就是回调方法  
            mOnClickListener.onClick(this);  
            return true;  
        }  
        return false;  
    }  
}  
```

接着是在回调对象MainActivity中实现回调接口OnClickListener，包含控制对象View的引用button，并调用控制对象View的方法setOnClickListener。

``` java
public class MainActivity extends Activity implements OnClickListener{  
    private Button button;  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        button = (Button)findViewById(R.id.button1);    
        button.setOnClickListener(this);  
    }  
    //用户点击Button时调用的回调函数，可以个性化定义 
    @Override  
    public void onClick(View v) {  
        Toast.makeText(getApplication(), "点击了", Toast.LENGTH_LONG).show();  
    } 
}  
```

可以对比上面总结出来的步骤，感受Android的点击事件是如何一步一步实现回调的。