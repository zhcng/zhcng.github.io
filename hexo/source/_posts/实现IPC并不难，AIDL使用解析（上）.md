---
title: 实现IPC并不难，AIDL使用解析（上）
date: 2017-07-31 16:46:40
categories: Android
tags: AIDL
---
IPC是 Inter-Process Communication 的缩写，含义为跨进程通信。通俗来说就是再两个进程之间进行数据交换的过程。而AIDL是 Android Interface Define Language 的缩写，含义为安卓接口定义语言， 是IPC进程间通信方式的一种。

<!-- more -->

## 概述

我们通过一个最简单的AIDL的示例，来了解怎么实现跨进程通信。假设有一个服务端和一个客户端，我们需要实现这样一个效果，在服务端储存管理图书，在客户端可以获取并能够修改服务端图书信息。在实现示例的过程中，讲解相应的概念信息。

## 添加Book类

我们新建一个项目，在里面new出两个Module，选择Phone & Tablet Module类型的，分别命名为BookManagerSevice和BookManagerClient作为我们的服务端和客户端。首先，我们为服务端里面增添一个Book类，作为图书信息的Bean类。

### AIDL可用文件类型

这里，我们需要明确一点，在AIDL文件中，只有如下的文件类型才可以使用。

- **基本数据类型：**int、long、char、boolean、double等
- **String和CharSequece**
- **List：** 只支持ArrayList，里面的每个元素都必须被AIDL支持
- **Map：** 只支持HashMap，里面的每个元素都必须被AIDL支持，包括Key和Value
- **List：** 只支持ArrayList，里面的每个元素都必须被AIDL支持
- **Parcelable：** 实现了Parcelable接口的对象
- **AIDL：** AIDL接口本身也可以在AIDL文件中使用

所以，我们需要在两个进程中传递Book信息，这个Book类就必须要实现Parcelable接口。

### Parcelable序列化

在Android中，实现序列化的方式通常有两种。一种是实现Serializable接口，它是Java中的序列化方式，使用简单但是开销很大，序列化和反序列化都需要大量的I/O操作。另一种是实现Parcelable接口，它是Android中的序列化方式，因此更适合用在Android平台上，它的缺点是使用起来比较麻烦，但是效率很高。一般来说，Parcelable主要用在内存序列化上，Serializable主要用在序列化到存储设备或网络传输上。而在AIDL中，如上面所说，我们只能选择Parcelable。

以下为Book.java的代码，基本可作为Parcelable序列化的模板代码。

``` java
package com.zhcng.bookmanagerservice;

import android.os.Parcel;
import android.os.Parcelable;

public class Book implements Parcelable {
    public int bookId;
    public String bookName;

    public Book(int bookId, String bookName) {
        this.bookId = bookId;
        this.bookName = bookName;
    }

    /**
     * 内容描述功能
     * @return 仅当当前对象存在文件描述符时返回1，否则返回0。
     */
    @Override
    public int describeContents() {
        return 0;
    }

    /**
     * 将当前对象按顺序写入序列化结构中
     */
    @Override
    public void writeToParcel(Parcel parcel, int i) {
        parcel.writeInt(bookId);
        parcel.writeString(bookName);
    }

    public static final Creator<Book> CREATOR = new Creator<Book>() {
        /**
         * 从序列化的对象中创建原始对象
         */
        @Override
        public Book createFromParcel(Parcel in) {
            return new Book(in);
        }

        /**
         * 创建指定长度的原始对象数组
         */
        @Override
        public Book[] newArray(int size) {
            return new Book[size];
        }
    };

    /**
     * 按顺序读取原始对象
     */
    protected Book(Parcel in) {
        bookId = in.readInt();
        bookName = in.readString();
    }

    @Override
    public String toString() {
        return bookId + " : " + bookName;
    }
}
```

从代码中我们可以看出，在Parcelable序列化过程中，需要实现的功能有内容描述、序列化和反序列化。内容描述由describeContents()方法完成，几乎所有情况都返回0，仅当当前对象存在文件描述符时返回1。序列化功能由writeToParcel()方法完成，通过Parcel中的一系列write方法将当前对象按顺序写入序列化结构中。反序列化功能由CREATOR来完成，其内部标明了如何创建序列化对象和数组，并通过Parcel中的一系列read方法将当前对象按顺序读取出来，完成反序列化过程。

## 创建AIDL文件

如果AIDL文件中用到了自定义的Parcelable对象，那么必须新建一个和它同名的AIDL文件，并在其中声明它为Parcelable类型。所以，我们必须要创建一个Book.aidl，添加如下代码。

``` java
// Book.aidl
package com.zhcng.bookmanagerservice;

parcelable Book;
```

接下来，再创建一个IBookManager.aidl的文件，添加如下代码。

``` java
// IBookManager.aidl
package com.zhcng.bookmanagerservice;

import com.zhcng.bookmanagerservice.Book;

interface IBookManager {
    List<Book> getBookList();
    void addBook (in Book book);
}
```

这里注意，自定义的Parcelable对象和AIDL对象必须要显式的import进来，不管它们是否和当前的AIDL文件位于同一个包内。所以这里我们要加上import com.zhcng.bookmanagerservice.Book。另外，AIDL中除了基本数据类型，其他类型的参数必须标上方向：in、out或者inout，in表示输入型参数，out表示输出型参数，inout表示输入输出型参数。

接下来，将Book.java以及AIDL文件连同包名一起复制到客户端中。注意报的结构一定要保持一致，否则会出错。这是因为客户端需要反序列化服务端中和AIDL接口相关的所有类，如果类的完整路径不一样，就无法成功反序列化。

最后，在Android Studio中，点击Tools-->Android-->Sync Project with Gradle Files进行同步。同步完成后，会看到在build包里面生成了IBookManager.java文件。

## 服务端

定义完AIDL接口，接下来就要在服务端实现这个接口了。在服务端工程下创建一个Service，命名为BookManagerService，代码如下。

``` java
package com.zhcng.bookmanagerservice;

import android.app.Service;
import android.content.Intent;
import android.os.Binder;
import android.os.IBinder;
import android.os.RemoteException;

import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class BookManagerService extends Service {
    private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<Book>();

    private Binder mBinder = new IBookManager.Stub() {
        @Override
        public List<Book> getBookList() throws RemoteException {
            return mBookList;
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            mBookList.add(book);
        }
    };

    @Override
    public void onCreate() {
        super.onCreate();
        mBookList.add(new Book(1, "Android"));
        mBookList.add(new Book(2, "Ios"));
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
}
```

这是一个服务端Service的典型实现。首先创建了一个mBinder对象并在onBind中返回它，这个对象继承自IBookManager并实现了它内部的AIDL方法。实现过程比较简单，需要注意的是这里我们采用了CopyOnWriteArrayList，它支持并发读写。AIDL方法是在服务端的Binder线程池中执行的，因此当多个客户端同时连接时，会存在多个线程同时访问的情形，所以我们要在AIDL方法中处理线程同步。前面我们提到，AIDL中能够使用的List只有ArrayList，那为什么这里可以使用CopyOnWriteArrayList呢？因为这里虽然服务端返回的是CopyOnWriteArrayList，但是在Binder中会按照List的规范去访问数据并最终形成一个新的ArrayList传递给客户端，关于这点会在后面的程序中给予验证。

最后，不要忘记了在AndroidManifest.xml中注册该Service。

``` xml
<service
    android:name=".BookManagerService"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="com.zhcng.bookmanagerservice.BookManagerService" />
    </intent-filter>
</service>
```

## 客户端

客户端的实现就比较简单了。首先要绑定远程服务，绑定成功后将服务端返回的Binder对象转换成为AIDL接口，然后就可以通过这个接口去调用服务端的远程方法了。

``` java
package com.zhcng.bookmanagerclient;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;
import android.os.RemoteException;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;

import com.zhcng.bookmanagerservice.Book;
import com.zhcng.bookmanagerservice.IBookManager;

import java.util.List;

public class BookManagerActivity extends AppCompatActivity {
    private static final String TAG = "BookManagerActivity";
    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            IBookManager bookManager = IBookManager.Stub.asInterface(iBinder);
            try {
                List<Book> list = bookManager.getBookList();
                Log.i(TAG, "query book list, list type:" + list.getClass().getCanonicalName());
                Log.i(TAG, "query book list:" + list.toString());
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {

        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_book_manager);
        bindService();
    }

    private void bindService() {
        Intent intent = new Intent();
        intent.setAction("com.zhcng.bookmanagerservice.BookManagerService");
        intent.setComponent(new ComponentName("com.zhcng.bookmanagerservice", "com.zhcng.bookmanagerservice.BookManagerService"));
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onDestroy() {
        unbindService(mConnection);
        super.onDestroy();
    }
}
```

我们在onCreate()方法里面通过action绑定Service，绑定成功后，通过bookManager去调用getBookList方法，获取到服务端图书的信息。要注意的是，服务端的方法有可能很久才能执行完毕，所以真实运用中这里需要处理避免ANR。接下来我们先启动服务端功能，再运行客户端工程，可以看到如下Log。

> I/BookManagerActivity: query book list, list type:java.util.ArrayList
I/BookManagerActivity: query book list:[1 : Android, 2 : Ios]

可以看到，虽然服务端返回的是CopyOnWriteArrayList类型，但是客户端收到的仍然是ArrayList类型，证实了前面的结论。同时，在客户端成功获取到了服务端的图书数据。

接下来，我们再调用另外一个接口addBook，在客户端给服务端添加一本书，看能否添加成功。修改onServiceConnected的代码如下。

``` java
public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
    IBookManager bookManager = IBookManager.Stub.asInterface(iBinder);
    try {
        List<Book> list = bookManager.getBookList();
        Log.i(TAG, "query book list, list type:" + list.getClass().getCanonicalName());
        Log.i(TAG, "query book list:" + list.toString());
        Book newBook = new Book(3, "Web");
        bookManager.addBook(newBook);
        Log.i(TAG, "add book:" + newBook);
        List<Book> newList = bookManager.getBookList();
        Log.i(TAG, "query new book list:" + newList.toString());
    } catch (RemoteException e) {
        e.printStackTrace();
    }
}
```

重新运行后查看Log，发现添加成功~

> I/BookManagerActivity: query book list, list type:java.util.ArrayList
I/BookManagerActivity: query book list:[1 : Android, 2 : Ios]
I/BookManagerActivity: add book:3 : Web
I/BookManagerActivity: query new book list:[1 : Android, 2 : Ios, 3 : Web]

最后，附上整个项目的代码目录结构。
![项目目录](http://opzh3oodr.bkt.clouddn.com/aidl/project_directory.png?imageMogr2/auto-orient/thumbnail/500x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)
