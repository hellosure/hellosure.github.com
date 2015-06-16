---
layout: post
title: 深入浅出Android Handler
category: Android
tags: [Android,Handler]
---

### Thread/Looper/Message/Handler的关系

1. 当一个Android应用程序第一次启动时,Android框架为应用程序的主线程创建一个Looper对象。一个Looper实现了一个简单的消息队列,在一个循环中处理Message对象。所有主要的应用程序框架事件(如活动生命周期方法调用,单击按钮,等等)都包含在Message对象,它被添加到Looper的消息队列然后一个个被处理。主线程的Looper在应用程序的整个生命周期中存在。
2. 当一个Handle在主线程被实例化,它就被关联到Looper的消息队列。被发送到消息队列的消息会持有一个Handler的引用,以便Android框架可以在Looper最终处理这个消息的时候，调用Handler#handleMessage(Message)。

### 深入浅出

#### Android同步机制的引入

很多初入Android或Java开发的新手对Thread、Looper、Handler和Message仍然比较迷惑，衍生的有HandlerThread、java.util.concurrent、Task、AsyncTask由于目前市面上的书籍等资料都没有谈到这些问题，今天就这一问题做更系统性的总结。我们创建的Service、Activity以及Broadcast均是一个主线程处理，这里我们可以理解为UI线程。但是在操作一些耗时操作时，比如I/O读写的大文件读写，数据库操作以及网络下载需要很长时间，为了不阻塞用户界面，出现ANR的响应提示窗口，这个时候我们可以考虑使用Thread线程来解决。

对于从事过J2ME开发的程序员来说Thread比较简单，直接匿名创建重写run方法，调用start方法执行即可。或者从Runnable接口继承，但对于Android平台来说UI控件都没有设计成为线程安全类型，所以需要引入一些同步的机制来使其刷新，这点Google在设计Android时倒是参考了下Win32的消息处理机制。
 
#### 更新View的方式

1. 对于线程中的刷新一个View为基类的界面，可以使用postInvalidate()方法在线程中来处理，其中还提供了一些重写方法比如postInvalidate(int left,int top,int right,int bottom) 来刷新一个矩形区域，以及延时执行，比如postInvalidateDelayed(long delayMilliseconds)或postInvalidateDelayed(long delayMilliseconds,int left,int top,int right,int bottom) 方法，其中第一个参数为毫秒

2. 当然推荐的方法是通过一个Handler来处理这些，可以在一个线程的run方法中调用handler对象的 postMessage或sendMessage方法来实现，Android程序内部维护着一个消息队列，会轮训处理这些，如果你是Win32程序员可以很好理解这些消息处理，不过相对于Android来说没有提供 PreTranslateMessage这些干涉内部的方法。
 
#### looper

Looper又是什么呢? ，其实Android中每一个Thread都跟着一个Looper，Looper可以帮助Thread维护一个消息队列，但是Looper和Handler没有什么关系，我们从开源的代码可以看到Android还提供了一个Thread继承类HanderThread可以帮助我们处理，在HandlerThread对象中可以通过getLooper方法获取一个Looper对象控制句柄，我们可以将其这个Looper对象映射到一个Handler中去来实现一个线程同步机制，Looper对象的执行需要初始化Looper.prepare方法就是昨天我们看到的问题，同时推出时还要释放资源，使用Looper.release方法。

#### message

Message 在Android是什么呢? 对于Android中Handler可以传递一些内容，通过Bundle对象可以封装String、Integer以及Blob二进制对象，我们通过在线程中使用Handler对象的sendEmptyMessage或sendMessage方法来传递一个Bundle对象到Handler处理器。对于Handler类提供了重写方法handleMessage(Message msg) 来判断，通过msg.what来区分每条信息。将Bundle解包来实现Handler类更新UI线程中的内容实现控件的刷新操作。相关的Handler对象有关消息发送sendXXXX相关方法如下，同时还有postXXXX相关方法，这些和Win32中的道理基本一致，一个为发送后直接返回，一个为处理后才返回 .

### 结合例子

#### 更新view

方法一：(java习惯，在android平台开发时这样是不行的，因为它违背了单线程模型）

刚刚开始接触android线程编程的时候，习惯好像java一样，试图用下面的代码解决问题   

{% highlight java %}

new Thread( new Runnable() {     
    public void run() {     
         myView.invalidate();    
     }            
}).start();

{% endhighlight %}

可以实现功能，刷新UI界面。但是这样是不行的，因为它违背了单线程模型：Android UI操作并不是线程安全的并且这些操作必须在UI线程中执行。

方法二：（Thread+Handler)

查阅了文档和apidemo后，发觉常用的方法是利用Handler来实现UI线程的更新的。

Handler来根据接收的消息，处理UI更新。Thread线程发出Handler消息，通知更新UI。

{% highlight java %}

Handler myHandler = new Handler() {  
          public void handleMessage(Message msg) {   
               switch (msg.what) {   
                    case TestHandler.GUIUPDATEIDENTIFIER:   
                         myBounceView.invalidate();  
                         break;   
               }   
               super.handleMessage(msg);   
          }   
     };  


class myThread implements Runnable {   
          public void run() {  
               while (!Thread.currentThread().isInterrupted()) {    
                       
                    Message message = new Message();   
                    message.what = TestHandler.GUIUPDATEIDENTIFIER;   
                      
                    TestHandler.this.myHandler.sendMessage(message);   
                    try {   
                         Thread.sleep(100);    
                    } catch (InterruptedException e) {   
                         Thread.currentThread().interrupt();   
                    }   
               }   
          }   
     }   

{% endhighlight %}

以上方法demo看:http://rayleung.javaeye.com/blog/411860
 
#### 定时

在Android里定时更新 UI，通常使用的是 java.util.Timer, java.util.TimerTask, android.os.Handler组合。实际上Handler 自身已经提供了定时的功能。 

{% highlight java %}

    private Handler handler = new Handler();  
  
    private Runnable myRunnable= new Runnable() {    
        public void run() {  
             
            if (run) {  
                handler.postDelayed(this, 1000);  
                count++;  
            }  
            tvCounter.setText("Count: " + count);  

        }  
    }; 

{% endhighlight %}

然后在其他地方调用

handler.post(myRunnable);

handler.post(myRunnable,time);

案例看：http://shaobin0604.javaeye.com/blog/515820

## Android中有关Handler的使用

一个Handler允许你发送和处理消息（Message）以及与一个线程的消息队列相关的Runnable对象。每个Handler实例都和单个线程以及该线程的消息队列有关。当你创建了一个新Handler，它就会和创建它的线程/消息队列绑定，在那以后，它就会传递消息以及runnable对象给消息队列，然后执行它们。
 
       需要使用Handler有两大主要的原因：
       （1）在将来的某个时间点调度处理消息和runnable对象；
       （2）将需要执行的操作放到其他线程之中，而不是自己的；
 
      调度处理消息是通过调用post(Runnable), postAtTime(Runnable, long), postDelayed(Runnable, long), sendEmptyMessage(int), sendMessage(Message), sendMessageAtTime(Message, long)和sendMessageDelayed(Message, long)等方法完成的。其中的post版本的方法可以让你将Runnable对象放进消息队列；sendMessage版本的方法可以让你将一个包含有bundle对象的消息对象放进消息队列，然后交由handleMessage(Message)方法处理。（这个需要你复写Handler的handleMessage方法）
      
andler在实际开发中是很常用的，主要是用来接收子线程发送的数据，然后主线程结合此数据来更新界面UI。
      Android应用程序启动时，他会开启一个主线程（也就是UI线程），管理界面中的UI控件，进行事件派发，比如说：点击一个按钮，Android会分发事件到Button上从而来响应你的操作。但是当你需要执行一个比较耗时的操作的话，例如：进行IO操作，网络通信等等，若是执行时间超过5s，那么Android会弹出一个“经典”的ANR无响应对话框，然后提示按“Force quit”或是“Wait”。解决此类问题的方法就是：我们把一些耗时的操作放到子线程中去执行。但因为子线程涉及到UI更新，而Android主线程是线程不安全的，所以更新UI的操作只能放在主线程中执行，若是放在子线程中执行的话很会出问题。所以这时就需要一种机制：主线程可以发送“命令/任务”给子线程执行，然后子线程反馈执行结果；
 
！你必需要知道的：
      若在主线程中实例化一个Handler对象，例如：
      Handler mHandler = new Handler();
      此时它并没有新派生一个线程来执行此Handler，而是将此Handler附加在主线程上，故此时若你在Handler中执行耗时操作的话，还是会弹出ANR对话框！
 
### post版本的Handler使用

{% highlight java %}

package com.dxyh.test;

import android.app.Activity;
import android.content.Context;
import android.os.Bundle;
import android.os.Handler;
import android.util.Log;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;

public class MainActivity extends Activity
			    implements OnClickListener {
	private final static String TAG = "HandlerTest";
	private final static int DELAY_TIME = 1000;
	
	private Button btnStart;
	private Button btnStop;
	
	Context mContext = null;
	
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        
        mContext = this;

        Log.i(TAG, "Main thread id = " + 
        		Thread.currentThread().getId());
        
        btnStart = (Button) findViewById(R.id.btn_start);
        btnStart.setOnClickListener(this);
        btnStop = (Button) findViewById(R.id.btn_stop);
        btnStop.setOnClickListener(this);
    }
    
    @Override
    public void onClick(View view) {
    	switch (view.getId()) {
    	case R.id.btn_start:
    		mHandler.postDelayed(workRunnable, DELAY_TIME);
    		break;
    	case R.id.btn_stop:
    		mHandler.removeCallbacks(workRunnable);
    		break;
    	}
    }
    
    Runnable workRunnable = new Runnable() {
    	int counter = 0;
    	
    	public void run() {
    		if (counter++ < 1) {
	    		Log.i(TAG, "workRunnable thread id = " + 
	    				Thread.currentThread().getId());
	    		mHandler.postDelayed(workRunnable, DELAY_TIME);
    		}
    	}
    };
    
    Handler mHandler = new Handler();
}

{% endhighlight %}

![1](http://hi.csdn.net/attachment/201105/5/0_1304610293515S.gif "1")

说明：发现thread id是相同的，这就说明：默认情况下创建的Handler会绑定到主线程上，你不能做太耗时的操作。

### sendMessage版本的Handler的使用

这里介绍几种模型:

#### 默认的Handler（消息处理队列挂在主线程上）

{% highlight java %}

package com.dxyh.test;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.util.Log;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;

public class MainActivity extends Activity
				  implements OnClickListener {
	private final static String TAG = "HandlerTest";
	
	private final static int TASK_BEGIN	= 1;
	private final static int TASK_1	= 2;
	private final static int TASK_2	= 3;
	private final static int TASK_END	= 4;
	
	private Button btnStart = null;
	private Button btnStop = null;
	
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        
        btnStart = (Button) findViewById(R.id.btn_start);
        btnStart.setOnClickListener(this);
        btnStop = (Button) findViewById(R.id.btn_stop);
        btnStop.setOnClickListener(this);
        
        Log.i(TAG, "[M_TID:" + Thread.currentThread().getId() + "]");
    }
    
    Handler mHandler = new Handler() {
    	// 注意：在各个case后面不能做太耗时的操作，否则出现ANR对话框
    	@Override
    	public void handleMessage(Message msg) {
    		switch (msg.what) {
    		case TASK_BEGIN:
    			Log.i(TAG, "[H_TID:" +
    				Thread.currentThread().getId() + "] Get TASK_BEGIN");
    			break;
    			
    		case TASK_1:
    			Log.i(TAG, "[H_TID:" +
    				Thread.currentThread().getId() + "] Get TASK_1");
    			break;
    			
    		case TASK_2:
    			Log.i(TAG, "[H_TID:" +
    				Thread.currentThread().getId() + "] Get TASK_2");
    			break;
    			
    		case TASK_END:
    			Log.i(TAG, "[H_TID:" +
    				Thread.currentThread().getId() + "] Get TASK_END");
    			finish();
    			break;
    		}
    		super.handleMessage(msg);
    	}
    };
    
    public void onClick(View view) {
    	switch (view.getId()) {
    	case R.id.btn_start:
    		// 启动任务（消息只有标识，立即投递）
    		mHandler.sendEmptyMessage(TASK_BEGIN);
    		Log.i(TAG, "Send TASK_BEGIN to handler.");
    		
    		// 开始任务1（在mHandler的消息队列中获取一个Message对象，避免重复构造）
    		Message msg1 = mHandler.obtainMessage(TASK_1);
    		msg1.obj = "This is task1";
    		mHandler.sendMessage(msg1);
    		Log.i(TAG, "Send TASK_1 to handler.");
    		
    		// 开启任务2（和上面类似）
    		Message msg2 = Message.obtain();
    		msg2.arg1 = 10;
    		msg2.arg2 = 20;
    		msg2.what = TASK_2;
    		mHandler.sendMessage(msg2);
    		Log.i(TAG, "Send TASK_2 to handler.");
    		break;
    		
    	case R.id.btn_stop:
    		// 结束任务（空消息体，延时2s投递）
    		mHandler.sendEmptyMessageDelayed(TASK_END, 2000);
    		Log.i(TAG, "Send TASK_END to handler.");
    		break;
    	}
}
}

{% endhighlight %}

![2](http://hi.csdn.net/attachment/201105/6/0_1304691667u3q3.gif "2")

#### 消息队列仍绑定在主线程上，但在子线程中发送消息

{% highlight java %}

package com.dxyh.test;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.util.Log;

public class MainActivity extends Activity {
	private final static String TAG = "HandlerTest";
	
	private final static int TASK_BEGIN	= 1;
	private final static int TASK_1	= 2;
	private final static int TASK_2	= 3;
	private final static int TASK_END	= 4;
	
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        
        Log.i(TAG, "[M_TID:" + Thread.currentThread().getId() + "]" +
        		"This is in main thread.");
        
        workThread.start();
    }
    
    Handler mHandler = new Handler() {
    	// 注意：在各个case后面不能做太耗时的操作，否则出现ANR对话框
    	@Override
    	public void handleMessage(Message msg) {
    		switch (msg.what) {
    		case TASK_BEGIN:
    			Log.i(TAG, "[H_TID:" +
				Thread.currentThread().getId() + "] Get TASK_BEGIN");
    			break;
    			
    		case TASK_1:
    			Log.i(TAG, "[H_TID:" +
				Thread.currentThread().getId() + "] Get TASK_1");
    			break;
    			
    		case TASK_2:
    			Log.i(TAG, "[H_TID:" +
				Thread.currentThread().getId() + "] Get TASK_2");
    			break;
    			
    		case TASK_END:
    			Log.i(TAG, "[H_TID:" +
				Thread.currentThread().getId() + "] Get TASK_END");
    			finish();
    			break;
    		}
    		super.handleMessage(msg);
    	}
    };
    
    Thread workThread = new Thread() {
    	// 你可以在run方法内做任何耗时的操作，然后将结果以消息形式投递到主线程的消息队列中
    	@Override
    	public void run() {
    		// 启动任务（消息只有标识，立即投递）
    		mHandler.sendEmptyMessage(TASK_BEGIN);
    		Log.i(TAG, "[S_TID:" + Thread.currentThread().getId() + "]" +
    				"Send TASK_START to handler.");
    		
    		// 开始任务1（在mHandler的消息队列中获取一个Message对象，避免重复构造）
    		Message msg1 = mHandler.obtainMessage(TASK_1);
    		msg1.obj = "This is task1";
    		mHandler.sendMessage(msg1);
    		Log.i(TAG, "[S_TID:" + Thread.currentThread().getId() + "]" +
    				"Send TASK_1 to handler.");
    		
    		// 开启任务2（和上面类似）
    		Message msg2 = Message.obtain();
    		msg2.arg1 = 10;
    		msg2.arg2 = 20;
    		msg2.what = TASK_2;
    		mHandler.sendMessage(msg2);
    		Log.i(TAG, "[S_TID:" + Thread.currentThread().getId() + "]" +
    				"Send TASK_2 to handler.");
    		
    		// 结束任务（空消息体，延时2s投递）
    		mHandler.sendEmptyMessageDelayed(TASK_END, 2000);
    		Log.i(TAG, "[S_TID:" + Thread.currentThread().getId() + "]" +
    				"Send TASK_END to handler.");
    	}
    };
}

{% endhighlight %}

![3](http://hi.csdn.net/attachment/201105/6/0_1304692156X4XD.gif "3")

#### 将消息队列绑定到子线程上，主线程只管通过Handler往子线程的消息队列中投递消息即可

{% highlight java %}

package com.dxyh.test;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.os.HandlerThread;
import android.os.Looper;
import android.os.Message;
import android.util.Log;

public class MainActivity extends Activity {
	private final static String TAG = "HandlerTest";
	
	private final static int TASK_BEGIN	= 1;
	private final static int TASK_1	= 2;
	private final static int TASK_2	= 3;
	private final static int TASK_END	= 4;
	
	private MyHandler mHandler = null;
	
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        Log.i(TAG, "[M_TID:" + Thread.currentThread().getId() + "]" +
        		"This is in main thread.");
        
        HandlerThread myLooperThread = new HandlerThread("my looper thread");
        myLooperThread.start();
        
        Looper looper = myLooperThread.getLooper();
        mHandler = new MyHandler(looper);
        
        // 启动任务（消息只有标识，立即投递）
		mHandler.sendEmptyMessage(TASK_BEGIN);
		Log.i(TAG, "[S_ID:" + Thread.currentThread().getId() + "]" +
				"Send TASK_START to handler.");
		
		// 开始任务1（在mHandler的消息队列中获取一个Message对象，避免重复构造）
		Message msg1 = mHandler.obtainMessage(TASK_1);
		msg1.obj = "This is task1";
		mHandler.sendMessage(msg1);
		Log.i(TAG, "[S_ID:" + Thread.currentThread().getId() + "]" +
				"Send TASK_1 to handler.");
		
		// 开启任务2（和上面类似）
		Message msg2 = Message.obtain();
		msg2.arg1 = 10;
		msg2.arg2 = 20;
		msg2.what = TASK_2;
		mHandler.sendMessage(msg2);
		Log.i(TAG, "[S_ID:" + Thread.currentThread().getId() + "]" +
				"Send TASK_2 to handler.");
		
		// 结束任务（空消息体，延时2s投递）
		mHandler.sendEmptyMessageDelayed(TASK_END, 2000);
		Log.i(TAG, "[S_ID:" + Thread.currentThread().getId() + "]" +
				"Send TASK_END to handler.");
    }
    
    class MyHandler extends Handler {
    	public MyHandler(Looper looper) {
    		super(looper);
    	}
    	
    	// 现在在每个case之后，你可以做任何耗时的操作了
    	@Override
    	public void handleMessage(Message msg) {
    		switch (msg.what) {
    		case TASK_BEGIN:
    			Log.i(TAG, "[H_TID:" +
    				Thread.currentThread().getId() + "] Get TASK_BEGIN");
    			break;
    			
    		case TASK_1:
    			Log.i(TAG, "[H_TID:" +
    				Thread.currentThread().getId() + "] Get TASK_1");
    			break;
    			
    		case TASK_2:
    			Log.i(TAG, "[H_TID:" +
    				Thread.currentThread().getId() + "] Get TASK_2");
    			break;
    			
    		case TASK_END:
    			Log.i(TAG, "[H_TID:" +
    				Thread.currentThread().getId() + "] Get TASK_END");
    			finish();
    			break;
    		}
    		super.handleMessage(msg);
    	}
    }
}

{% endhighlight %}

![3](http://hi.csdn.net/attachment/201105/6/0_1304692161OYTN.gif "3")

#### 自己创建新的线程，然后在新线程中创建Looper，主线程调用子线程中的发消息方法，将消息发给子线程的消息队列

{% highlight java %}

package com.dxyh.test;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.os.Message;
import android.util.Log;

public class MainActivity extends Activity {
	private final static String TAG = "HandlerTest";
	
	private final static int TASK_BEGIN	= 1;
	private final static int TASK_1	= 2;
	private final static int TASK_2	= 3;
	private final static int TASK_END	= 4;
	
	private Handler workHandler = null;
	
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        Log.i(TAG, "[M_TID:" + Thread.currentThread().getId() + "]" +
        		"This is in main thread.");
        
        LooperThread testThread = new LooperThread();
        testThread.start();
        
        // 注意，这里需要等待一下，防止出现空指针异常
        while (null == workHandler) {
        }
        
        testThread.sendMessageTodoYourWork();
    }
    
    class LooperThread extends Thread {
    	@Override
    	public void run() {
    		Looper.prepare();
    		
    		workHandler = new Handler() {
    			// 现在在每个case之后，你可以做任何耗时的操作了
    	    	@Override
    	    	public void handleMessage(Message msg) {
    	    		switch (msg.what) {
    	    		case TASK_BEGIN:
    	    			Log.i(TAG, "[H_TID:" +
    	    			   Thread.currentThread().getId() + "] Get TASK_BEGIN");
    	    			break;
    	    			
    	    		case TASK_1:
    	    			Log.i(TAG, "[H_TID:" +
    	    			   Thread.currentThread().getId() + "] Get TASK_1");
    	    			break;
    	    			
    	    		case TASK_2:
    	    			Log.i(TAG, "[H_TID:" +
    	    			   Thread.currentThread().getId() + "] Get TASK_2");
    	    			break;
    	    			
    	    		case TASK_END:
    	    			Log.i(TAG, "[H_TID:" +
    	    			   Thread.currentThread().getId() + "] Get TASK_END");
    	    			Looper.myLooper().quit();
    	    			finish();
    	    			break;
    	    		}
    	    		super.handleMessage(msg);
    	    	}
    		};
    		
    		Looper.loop();
    	}
    	
    	public void sendMessageTodoYourWork() {
    		Log.i(TAG, "[S_ID:" + Thread.currentThread().getId() + "]" +
				"Send TASK_START to handler.");
    		// 启动任务（消息只有标识，立即投递）
    		workHandler.sendEmptyMessage(TASK_BEGIN);

    		Log.i(TAG, "[S_ID:" + Thread.currentThread().getId() + "]" +
				"Send TASK_1 to handler.");
    		// 开始任务1（在workHandler的消息队列中获取一个Message对象，避免重复构造）
    		Message msg1 = workHandler.obtainMessage(TASK_1);
    		msg1.obj = "This is task1";
    		workHandler.sendMessage(msg1);
    		
    		Log.i(TAG, "[S_ID:" + Thread.currentThread().getId() + "]" +
				"Send TASK_2 to handler.");    		
    		// 开启任务2（和上面类似）
    		Message msg2 = Message.obtain();
    		msg2.arg1 = 10;
    		msg2.arg2 = 20;
    		msg2.what = TASK_2;
    		workHandler.sendMessage(msg2);

    		Log.i(TAG, "[S_ID:" + Thread.currentThread().getId() + "]" +
				"Send TASK_END to handler.");
    		// 结束任务（空消息体，延时2s投递）
    		workHandler.sendEmptyMessageDelayed(TASK_END, 2000);
    	}
    }
}

{% endhighlight %}

![4](http://hi.csdn.net/attachment/201105/6/0_1304692714WG0R.gif "4")

#### 主/子线程均有一个消息队列，然后相互传递消息（这种方式是最灵活的，双向传递，也不复杂）

{% highlight java %}

package com.dxyh.test;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.os.Message;
import android.util.Log;

public class MainActivity extends Activity {
	private final static String TAG = "HandlerTest";
	
	private final static int TASK_BEGIN		= 1;
	private final static int TASK_1		= 2;
	private final static int TASK_2		= 3;
	private final static int TASK_END		= 4;
	
	private final static int TASK_BEGIN_OVER	= 11;
	private final static int TASK_1_OVER	= 12;
	private final static int TASK_2_OVER	= 13;
	private final static int TASK_END_OVER	= 14;
	
	private Handler workHandler = null;
	
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        Log.i(TAG, "[M_TID:" + Thread.currentThread().getId() + "]" +
        		"This is in main thread.");
        
        LooperThread testThread = new LooperThread();
        testThread.start();
        
        // 注意，这里需要等待一下，防止出现空指针异常
        while (null == workHandler) {
        }
        
        testThread.sendMessageTodoYourWork();
    }
  
    Handler mainHandler = new Handler () {
    	// 在每个case之后，不能做耗时的操作
    	@Override
    	public void handleMessage(Message msg) {
    		switch (msg.what) {
    		case TASK_BEGIN_OVER:
    			Log.i(TAG, "[MH_TID:" +
    				Thread.currentThread().getId() + "] TASK_BEGIN_OVER");
    			break;
    			
    		case TASK_1_OVER:
    			Log.i(TAG, "[MH_TID:" +
    				Thread.currentThread().getId() + "] TASK_1_OVER");
    			break;
    			
    		case TASK_2_OVER:
    			Log.i(TAG, "[MH_TID:" +
    				Thread.currentThread().getId() + "] TASK_2_OVER");
    			break;
    			
    		case TASK_END_OVER:
    			Log.i(TAG, "[MH_TID:" +
    				Thread.currentThread().getId() + "] TASK_END_OVER");
    			finish();
    			break;
    		}
    		super.handleMessage(msg);
    	}
    };
    
    class LooperThread extends Thread {
    	@Override
    	public void run() {
    		Looper.prepare();
    		
    		workHandler = new Handler() {
    			// 现在在每个case之后，你可以做任何耗时的操作了
    	    	@Override
    	    	public void handleMessage(Message msg) {
    	    		switch (msg.what) {
    	    		case TASK_BEGIN:
    	    			Log.i(TAG, "[ZH_TID:" +
    	    			   Thread.currentThread().getId() + "] Get TASK_BEGIN");
    	    			// 做完之后报告信息给主线程
    	    			mainHandler.sendEmptyMessage(TASK_BEGIN_OVER);
    	    			break;
    	    			
    	    		case TASK_1:
    	    			Log.i(TAG, "[ZH_TID:" +
    	    			   Thread.currentThread().getId() + "] Get TASK_1");
    	    			// 做完之后报告信息给主线程
    	    			mainHandler.sendEmptyMessage(TASK_1_OVER);
    	    			break;
    	    			
    	    		case TASK_2:
    	    			Log.i(TAG, "[ZH_TID:" +
    	    			   Thread.currentThread().getId() + "] Get TASK_2");
    	    			// 做完之后报告信息给主线程
    	    			mainHandler.sendEmptyMessage(TASK_2_OVER);
    	    			break;
    	    			
    	    		case TASK_END:
    	    			Log.i(TAG, "[ZH_TID:" +
    	    			   Thread.currentThread().getId() + "] Get TASK_END");
    	    			Looper.myLooper().quit();
    	    			// 做完之后报告信息给主线程
    	    			mainHandler.sendEmptyMessage(TASK_END_OVER);
    	    			Looper.myLooper().quit();
    	    			break;
    	    		}
    	    		super.handleMessage(msg);
    	    	}
    		};
    		
    		Looper.loop();
    	}
    	
    	public void sendMessageTodoYourWork() {
    		Log.i(TAG, "[ZS_ID:" + Thread.currentThread().getId() + "]" +
				"Send TASK_START to handler.");
    		// 启动任务（消息只有标识，立即投递）
    		workHandler.sendEmptyMessage(TASK_BEGIN);

    		Log.i(TAG, "[ZS_ID:" + Thread.currentThread().getId() + "]" +
				"Send TASK_1 to handler.");
    		// 开始任务1（在workHandler的消息队列中获取一个Message对象，避免重复构造）
    		Message msg1 = workHandler.obtainMessage(TASK_1);
    		msg1.obj = "This is task1";
    		workHandler.sendMessage(msg1);
    		
    		Log.i(TAG, "[ZS_ID:" + Thread.currentThread().getId() + "]" +
				"Send TASK_2 to handler.");    		
    		// 开启任务2（和上面类似）
    		Message msg2 = Message.obtain();
    		msg2.arg1 = 10;
    		msg2.arg2 = 20;
    		msg2.what = TASK_2;
    		workHandler.sendMessage(msg2);

    		Log.i(TAG, "[ZS_ID:" + Thread.currentThread().getId() + "]" +
				"Send TASK_END to handler.");
    		// 结束任务（空消息体，延时2s投递）
    		workHandler.sendEmptyMessageDelayed(TASK_END, 2000);
    	}
    }
}

{% endhighlight %}

![5](http://hi.csdn.net/attachment/201105/6/0_1304692718t2rw.gif "5")

小结：Handler在Android中是很常用的，或是用来更新UI，或是派发任务给子线程去执行，也可以用来产生超时效果，比如用sendMessageDelayed(TASK_TIMEOUT, OUT_TIME)方法。


-EOF-
