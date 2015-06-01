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

{% highlight java %}

    public class TestSurfaceView extends Activity {  
        @Override  
        public void onCreate(Bundle savedInstanceState) {  
            super.onCreate(savedInstanceState);  
            setContentView(new MyView(this));  
        }  
     }
 
{% endhighlight %}

{% highlight java %}

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
      

{% endhighlight %}


### GLSurfaceView

如果是非交互式的应用，可以直接使用GLSurfaceView。如果需要交互式的行为，则需要继承
GLSurfaceView并重写一些方法。

#### 非交互

{% highlight java %}

/** 
 * 本示例演示OpenGL ES开发3D应用 
 * 该Activity直接使用了GLSurfaceView 
 * 这是因为GLSurfaceView可以直接使用，除非需要接受用户输入，和用户交互，才需要重写一些GLSurfaceView的方法 
 * 如果开发一个非交互式的OpenGL应用，可以直接使用GLSurfaceView。参照本示例 
 * @author Administrator 
 * 
 */  
public class NonInteractiveDemo extends Activity {  
      
    private GLSurfaceView mGLView;  
      
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
          
        mGLView = new GLSurfaceView(this);  
        //这里需要指定一个自定义的渲染器  
        mGLView.setRenderer(new DemoRenderer());  
        setContentView(mGLView);  
          
    }  
      
      
    public void onPause(){  
        super.onPause();  
        mGLView.onPause(); //当Activity暂停时，告诉GLSurfaceView也停止渲染，并释放资源。  
    }  
      
    public void onResume(){  
        super.onResume();  
        mGLView.onResume(); //当Activity恢复时，告诉GLSurfaceView加载资源，继续渲染。  
    }  
      
      
      
      
}  
class DemoRenderer implements Renderer{  
    @Override  
    public void onDrawFrame(GL10 gl) {  
        //每帧都需要调用该方法进行绘制。绘制时通常先调用glClear来清空framebuffer。  
        //然后调用OpenGL ES其他接口进行绘制  
        gl.glClear(GL10.GL_COLOR_BUFFER_BIT|GL10.GL_DEPTH_BUFFER_BIT);  
          
    }  
    @Override  
    public void onSurfaceChanged(GL10 gl, int w, int h) {  
        //当surface的尺寸发生改变时，该方法被调用，。往往在这里设置ViewPort。或者Camara等。   
        gl.glViewport(0, 0, w, h);  
    }  
    @Override  
    public void onSurfaceCreated(GL10 gl, EGLConfig config) {  
        // 该方法在渲染开始前调用，OpenGL ES的绘制上下文被重建时也会调用。  
        //当Activity暂停时，绘制上下文会丢失，当Activity恢复时，绘制上下文会重建。  
          
        //do nothing special  
    }  
      
}  

{% endhighlight %}

#### 交互

{% highlight java %}

/** 
 * 本示例演示OpenGL ES开发3D应用 
 * 该Activity使用了自定义的GLSurfaceView的子类 
 * 这样，我们可以开发出和用户交互的应用，比如游戏等。 
 * 需要注意的是：由于渲染对象是运行在一个独立的渲染线程中，所以 
 * 需要采用跨线程的机制来进行事件的处理。但是Android提供了一个简便的方法 
 * 我们只需要在事件处理中使用queueEvent(Runnable)就可以了. 
 *  
 * 对于大多数3D应用，如游戏、模拟等都是持续性渲染，但对于反应式应用来说，只有等用户进行了某个操作后再开始渲染。 
 * GLSurfaceView支持这两种模式。通过调用方法setRenderMode()方法设置。 
 * 调用requestRender()继续渲染。 
 *  
 *  
 * @author Administrator 
 * 
 */  
public class InteractiveDemo extends Activity {  
      
    private GLSurfaceView mGLView;  
      
    public void onCreate(Bundle savedInstanceState){  
        super.onCreate(savedInstanceState);  
        mGLView = new DemoGLSurfaceView(this); //这里使用的是自定义的GLSurfaceView的子类  
        setContentView(mGLView);  
    }  
      
      
    public void onPause(){  
        super.onPause();  
        mGLView.onPause();  
    }  
      
    public void onResume(){  
        super.onResume();  
        mGLView.onResume();  
    }  
}  
class DemoGLSurfaceView extends GLSurfaceView{  
    DemoRenderer2 mRenderer;  
      
    public DemoGLSurfaceView(Context context) {  
        super(context);  
        //为了可以激活log和错误检查，帮助调试3D应用，需要调用setDebugFlags()。  
        this.setDebugFlags(DEBUG_CHECK_GL_ERROR|DEBUG_LOG_GL_CALLS);  
        mRenderer = new DemoRenderer2();  
        this.setRenderer(mRenderer);  
    }  
      
    public boolean onTouchEvent(final MotionEvent event){  
        //由于DemoRenderer2对象运行在另一个线程中，这里采用跨线程的机制进行处理。使用queueEvent方法  
        //当然也可以使用其他像Synchronized来进行UI线程和渲染线程进行通信。  
        this.queueEvent(new Runnable() {  
              
            @Override  
            public void run() {  
                  
                //TODO:  
                mRenderer.setColor(event.getX()/getWidth(), event.getY()/getHeight(), 1.0f);  
            }  
        });  
          
        return true;  
    }  
      
}  
/** 
 * 这个应用在每一帧中清空屏幕，当tap屏幕时，改变屏幕的颜色。 
 * @author Administrator 
 * 
 */  
class DemoRenderer2 implements GLSurfaceView.Renderer{  
      
    private float mRed;  
    private float mGreen;  
    private float mBlue;  
    @Override  
    public void onDrawFrame(GL10 gl) {  
        // TODO Auto-generated method stub  
        gl.glClearColor(mRed, mGreen, mBlue, 1.0f);  
        gl.glClear(GL10.GL_COLOR_BUFFER_BIT|GL10.GL_DEPTH_BUFFER_BIT);  
    }  
    @Override  
    public void onSurfaceChanged(GL10 gl, int w, int h) {  
        // TODO Auto-generated method stub  
        gl.glViewport(0, 0, w, h);  
    }  
    @Override  
    public void onSurfaceCreated(GL10 gl, EGLConfig config) {  
        // TODO Auto-generated method stub  
          
    }  
      
    public void setColor(float r, float g, float b){  
        this.mRed = r;  
        this.mGreen = g;  
        this.mBlue = b;  
    }  
      
}  

{% endhighlight %}

-EOF-
