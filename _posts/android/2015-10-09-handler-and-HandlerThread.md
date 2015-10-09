---
layout: post
title: Handler与HandlerThread
category: 安卓
tags: android
---

## 1.什么是Handler


**SDK**中关于**Handler**的说明如下：

A Handler allows you to sendand process Messageand Runnable objects associated with a thread's MessageQueue.Each Handler instance is associated with a single thread and that thread'smessage queue. When you create a new Handler, it is bound to the thread /message queue of the thread that is creating it -- from that point on, it willdeliver messages and runnables to that message queue and execute them as theycome out of the message queue.

### 1.1 Handler的作用

There are two main uses for aHandler: (1) to schedule messages and runnables to be executed as some point inthe future; and (2) to enqueue an action to be performed on a different threadthan your own.

#### 1.1.1 发送和处理消息

下面是一段常见的代码：


```
Private MainHandler mMainHandler = new MainHandler();

private class MainHandler extends Handler {
    public voidhandleMessage(android.os.Message msg) {
        switch (msg.what) {
        case MSG_MAIN_HANDLER_TEST:
            Log.d(TAG, "MainHandler-->handleMessage-->thread id =" +     Thread.currentThread().getId());
            break;
        }
    }
 };
```

这样就可以通过对象mMainHandler发送消息给Handler处理了：

```
Message msg = mMainHandler.obtainMessage(MSG_MAIN_HANDLER_TEST);
mMainHandler.sendMessage(msg);
```

#### 1.1.2 在handler中使用runnable对象

 1. 在主线程中定义Handler对象
 2. 构造一个runnable对象，为该对象实现runnable方法
 3. 在子线程中使用Handler对象post(runnable)对象

**注意：**

handler.post这个函数的作用是把Runnable里面的run方法里面的这段代码发送到消息队列中，等待运行。如果handler是以UI线程消息队列为参数构造的，那么是把run里面的代码发送到UI线程中，等待UI线程运行这段代码。如果handler是以子线程线程消息队列为参数构造的，那么是把run里面的代码发送到子线程中，等待子线程运行这段代码。

一段示例代码：

```
public class TestActivity extends Activity 
implements OnClickListener {
    /** Calledwhen the activity is first created. */
   
    private Button mBtnTest=null;
    private Handler myHandler=null;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
       
        mBtnTest=(Button)findViewById(R.id.btn_test);
        mBtnTest.setOnClickListener(this);
       
        myHandler=new Handler();
    }
    @Override
    public void onClick(View v) {
        //注意:此处UI线程被阻塞，因为myHandler是在UI线程中创建的
       myHandler.post(new Runnable() {
           public void run() {
                long i=0;
                while(true){
                i++;
                }
              }
           });
    }
}
```

#### 1.1.3 线程间通信

在一个线程中向另一个线程所拥有的handler发送消息，即可实现线程间通信。例如在worker线程中修改ui

### 1.2 职责与关系

熟悉Windows编程的朋友可能知道Windows程序是消息驱动的，并且有全局的消息循环系统。而Android应用程序也是消息驱动的，按道理来说也应该提供消息循环机制。实际上谷歌参考了Windows的消息循环机制，也在Android系统中实现了消息循环机制。 Android通过Looper、Handler来实现消息循环机制，Android消息循环是针对线程的（每个线程都可以有自己的消息队列和消息循环）。

下面就简单的介绍下各个对象的职责：
Message：消息，其中包含了消息ID，消息处理对象以及处理的数据等，由MessageQueue统一列队，终由Handler处理。

Handler：处理者，负责Message的发送及处理。使用Handler时，需要实现handleMessage(Messagemsg)方法来对特定的Message进行处理，例如更新UI等。

MessageQueue：消息队列，用来存放Handler发送过来的消息，并按照FIFO规则执行。当然，存放Message并非实际意义的保存，而是将Message以链表的方式串联起来的，等待Looper的抽取。

Looper：消息泵，不断地从MessageQueue中抽取Message执行。因此，一个MessageQueue需要一个Looper。

Thread：线程，负责调度整个消息循环，即消息循环的执行场所。

Handler，Looper和MessageQueue就是简单的三角关系。Looper和MessageQueue一一对应，创建一个 Looper的同时，会创建一个MessageQueue。而Handler与它们的关系，只是简单的聚集关系，即Handler里会引用当前线程里的特定Looper和MessageQueue。这样说来，多个Handler都可以共享同一Looper和MessageQueue了。当然，这些Handler也就运行在同一个线程里。

**【疑问】**我们写的Activity中没有看见looper，也没有看见什么Thread去调度整个消息循环？

Activity是一个UI线程，运行于主线程中，Android系统在启动的时候会为Activity创建一个消息队列和消息循环（Looper），详细实现请参考ActivityThread.java文件。

### 1.3 消息发送与处理的过程

由1.2中的分析，结合代码，我们可以总结消息发送与处理的过程为：

#### 1.3.1 消息的生成

```
       Message msg =mHandler.obtainMessage();
       msg.what = what;
       msg.sendToTarget();
```

#### 1.3.2 消息的发送

```
       MessageQueue queue= mQueue;
        if (queue != null){
            msg.target =this;
            sent =queue.enqueueMessage(msg, uptimeMillis);
        }
```

在Handler.java的sendMessageAtTime(Messagemsg, long uptimeMillis)方法中，我们看到，它找到它所引用的MessageQueue，然后将Message的target设定成自己（目的是为了在处理消息环节，Message能找到正确的Handler），再将这个Message纳入到消息队列中。

#### 1.3.3 消息的提取

```
        Looper me =myLooper();
        MessageQueue queue= me.mQueue;
        while (true) {
            Message msg =queue.next(); // might block
            if (msg !=null) {
                if(msg.target == null) {
                    // Notarget is a magic identifier for the quit message.
                   return;
                }
                msg.target.dispatchMessage(msg);
               msg.recycle();
            }
        }
```

在Looper.java的loop()函数里，我们看到，这里有一个死循环，不断地从MessageQueue中获取下一个（next方法）Message，然后通过Message中携带的target信息，交由正确的Handler处理（dispatchMessage方法）

#### 1.3.4 消息的处理

```
        if (msg.callback!= null) {
           handleCallback(msg);
        } else {
            if (mCallback!= null) {
                if(mCallback.handleMessage(msg)) {
                    return;
                }
            }
           handleMessage(msg);
        }
```
在Handler.java的dispatchMessage(Messagemsg)方法里，其中的一个分支就是调用handleMessage方法来处理这条Message，而这也正是我们在职责处描述使用Handler时需要实现handleMessage(Messagemsg)的原因。
至于dispatchMessage方法中的另外一个分支，我将会在后面的内容中说明。
至此，我们看到，一个Message经由Handler的发送，MessageQueue的入队，Looper的抽取，又再一次地回到Handler的怀抱。而绕的这一圈，**帮助我们将同步操作变成了异步操作**，同时解决了线程数据同步的问题。

### 1.4 在工作线程中使用handler

使用handler必须有对应的messageQueue以及looper，在工作线程中必须自己创建，如：

```
       class LooperThreadextends Thread {
                publicHandler mHandler;
                   publicvoid run() {
                      Looper.prepare();
                       mHandler= new Handler() {
                           public void handleMessage(Message msg) {
                           // process incoming messages here
                            }
                        };
                       Looper.loop();   //不能在这个后面添加代码，程序是无法运行到这行之后的
                   }
      }
```

在创建Handler之前，为该线程准备好一个Looper（Looper.prepare），然后让这个Looper跑起来（Looper.loop），抽取Message，这样，Handler才能正常工作。


### 1.5 从非ui线程更新ui的几种方法

 1. Activity.runOnUiThread(Runnable)
 2. View.post(Runnable)
 3. View.postDelayed(Runnable, long)
 4. 向在UI线程中创建的Handler发消息

## 2 HandlerThread

HandlerThread本质还是一个Thread，只不过使得使用Handler更加方便，看一个例子：

```
       //生成一个HandlerThread对象，实现了使用Looper来处理消息队列的功能，
       //这个类由Android应用程序框架提供
       mHandlerThread = new HandlerThread("handler_thread");
      
       //在使用HandlerThread的getLooper()方法之前，必须先调用该类的start();
       mHandlerThread.start();
       //即这个Handler是运行在mHandlerThread这个线程中
       mMyHandler = new MyHandler(mHandlerThread.getLooper());
      
       mMyHandler.sendEmptyMessage(1);
    }
   
    private class MyHandler extends Handler {
      
       public MyHandler(Looper looper) {
           super(looper);
       }
 
       @Override
       public void handleMessage(Message msg) {
           Log.d(TAG, "MyHandler-->handleMessage-->threadid = " + Thread.currentThread().getId());
           super.handleMessage(msg);
       }
    }

```  


