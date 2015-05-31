---
layout: post
title: 说说Android的GLSurfaceView
category: Android
tags: [Android, OpenGL, SurfaceView]
---

Android游戏当中主要的除了控制类外就是显示类View。SurfaceView是从View基类中派生出来的显示类。

### view、SurfaceView和GLSurfaceView的区别
 
View：显示视图，内置画布，提供图形绘制函数、触屏事件、按键事件函数等；必须在UI主线程内更新画面，速度较慢

SurfaceView：基于view视图进行拓展的视图类，更适合2D游戏的开发；是view的子类，类似使用双缓机制，在新的线程中更新画面所以刷新界面速度比view快。

GLSurfaceView：基于SurfaceView视图再次进行拓展的视图类，专用于3D游戏开发的视图；是SurfaceView的子类，openGL专用。

在2D游戏开发中，大致可以分为两种游戏框架，View和SurfaceView。

#### View和SurfaceView区别：

View：必须在UI的主线程中更新画面，用于被动更新画面。

surfaceView：UI线程和子线程中都可以。在一个新启动的线程中重新绘制画面，主动更新画面。

UI的主线程中更新画面 可能会引发问题，比如你更新画面的时间过长，那么你的主UI线程会被你正在画的函数阻塞。那么将无法响应按键，触屏等消息。
当使用surfaceView 由于是在新的线程中更新画面所以不会阻塞你的UI主线程。但这也带来了另外一个问题，就是事件同步，涉及到线程同步。

#### 选择View还是SurfaceView：

所以基于以上，根据游戏特点，一般分成两类。

1. 被动更新画面的。比如棋类，这种用view就好了。因为画面的更新是依赖于 onTouch 来更新，可以直接使用 invalidate。 因为这种情况下，这一次Touch和下一次的Touch需要的时间比较长些，不会产生影响。

2. 主动更新。比如一个人在一直跑动。这就需要一个单独的thread不停的重绘人的状态，避免阻塞main UI thread。所以显然view不合适，需要surfaceView来控制


### SurfaceView的使用

下面是使用sufaceView一个基本的框架：

使用的SurfaceView的时候，一般情况下要对其进行创建，销毁，改变时的情况进行监视，这就要用到 `SurfaceHolder.Callback`. 

<% highlight java %>

    public class TestSurfaceView extends Activity {  
        @Override  
        public void onCreate(Bundle savedInstanceState) {  
            super.onCreate(savedInstanceState);  
            setContentView(new MyView(this));  
        }  
     }
 
<% endhighlight %>

<% highlight java %>

    class MyView extends SurfaceView implements SurfaceHolder.Callback,Runnable{  
        SurfaceHolder holder=null;  
        Paint paint;  
        public MyView(Context context) {  
            super(context);  
            // TODO Auto-generated constructor stub  
            holder=getHolder();  
            holder.addCallback(this);  
            paint=new Paint(Paint.ANTI_ALIAS_FLAG);  
            paint.setColor(Color.RED);  
              
            this.setFocusable(true);  
        }  
        
        @Override  
        public void surfaceChanged(SurfaceHolder holder, int format, int width,  
                int height) {  
            // TODO Auto-generated method stub  
              
        }  
  
        @Override  
        public void surfaceCreated(SurfaceHolder holder) {  
            // TODO Auto-generated method stub  
            Thread t=new Thread(this);  
            t.start();  
        }  
  
        @Override  
        public void surfaceDestroyed(SurfaceHolder holder) {  
            // TODO Auto-generated method stub  
            isRunning=false;  
        }  
  
        @Override  
        protected void onDraw(Canvas canvas) {  
            // TODO Auto-generated method stub  
            canvas=holder.lockCanvas();  
            //刷屏    
            canvas.drawColor(Color.BLACK);  
              
            canvas.drawCircle(x, y, 10, paint);  
              
            holder.unlockCanvasAndPost(canvas);  
        }  
        private void paint(Paint paint) {  
            Canvas canvas=holder.lockCanvas();  
            //刷屏    
            canvas.drawColor(Color.BLACK);  
              
            canvas.drawCircle(x, y, 10, paint);  
              
            holder.unlockCanvasAndPost(canvas);  
        }  
          
        boolean isRunning=true;  
        @Override  
        public void run() {  
            // TODO Auto-generated method stub  
            while (isRunning) {  
                paint(paint);  
                move();  
                try {  
                    Thread.sleep(50);  
                } catch (InterruptedException e) {  
                    // TODO Auto-generated catch block  
                    e.printStackTrace();  
                }  
                  
            }  
        }  
          
        private int x,y;  
        private void move(){  
            x+=2;  
            y+=2;  
        }  
    }  
      

<% endhighlight %>

-EOF-
