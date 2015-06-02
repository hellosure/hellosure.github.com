---
layout: post
title: Android的viewHolder模式
category: Android
tags: [Android,viewHolder]
---

### 深入浅出（这部分总结的非常好）

ListView之BaseAdapter的基本使用以及ViewHolder模式。

话说开发用了各种Adapter之后感觉用的最舒服的还是BaseAdapter，尽管使用起来比其他适配器有些麻烦，但是使用它却能实现很多自己喜欢的列表布局，比如ListView、GridView、Gallery、Spinner等等。它是直接继承自接口类Adapter的，使用BaseAdapter时需要重写很多方法，其中最重要的当属getView，因为这会涉及到ListView优化等问题，其他的方法可以参考链接的文章

BaseAdapter与其他Adapter有些不一样，其他的Adapter可以直接在其构造方法中进行数据的设置，比如

{% highlight java %}

SimpleAdapter adapter = new SimpleAdapter(this, getData(), R.layout.list_item, new String[]{"img","title","info",new int[]{R.id.img, R.id.title, R.id.info}});

{% endhighlight %}

但是在BaseAdapter中需要实现一个继承自BaseAdapter的类，并且重写里面的很多方法，例如

{% highlight java %}

class MyAdapter extends BaseAdapter
    {
        private Context context;
        public MyAdapter(Context context)
        {
            this.context = context;
        }
        @Override
        public int getCount() {
            // How many items are in the data set represented by this Adapter.(在此适配器中所代表的数据集中的条目数)
            return 0;
        }
        @Override
        public Object getItem(int position) {
            // Get the data item associated with the specified position in the data set.(获取数据集中与指定索引对应的数据项)
            return null;
        }
        @Override
        public long getItemId(int position) {
            // Get the row id associated with the specified position in the list.(取在列表中与指定索引对应的行id)
            return 0;
        }
        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            // Get a View that displays the data at the specified position in the data set.
            return null;
        } 
    }

{% endhighlight %}

这里面没什么难度，但是这个getView方法必须好好处理，也是最麻烦的

第一种：没有任何处理，不建议这样写。如果数据量少看将就，但是如果列表项数据量很大的时候，会每次都重新创建View，设置资源，严重影响性能，所以从一开始就不要用这种方式

{% highlight java %}

@Override
        public View getView(int position, View convertView, ViewGroup parent) {
            View item = mInflater.inflate(R.layout.list_item, null);
            ImageView img = (ImageView)item.findViewById(R.id.img)
            TextView title = (TextView)item.findViewById(R.id.title);
            TextView info = (TextView)item.findViewById(R.id.info);
            img.setImageResource(R.drawable.ic_launcher);
            title.setText("Hello");
            info.setText("world");
                                                                                                                         
            return item;
        }

{% endhighlight %}

第二种ListView优化：通过缓存convertView,这种利用缓存contentView的方式可以判断如果缓存中不存在View才创建View，如果已经存在可以利用缓存中的View，提升了性能

{% highlight java %}

public View getView(int position, View convertView, ViewGroup parent) {
            if(convertView == null)
            {
                convertView = mInflater.inflate(R.layout.list_item, null);
            }
                                                                                             
            ImageView img = (ImageView)convertView.findViewById(R.id.img)
            TextView title = (TextView)convertView.findViewById(R.id.title);
            TextView info = (TextView)ConvertView.findViewById(R.id.info);
            img.setImageResource(R.drawable.ic_launcher);
            title.setText("Hello");
            info.setText("world");
                                                                                             
            return convertView;
        }
        
{% endhighlight %}

第三种ListView优化：通过convertView+ViewHolder来实现，ViewHolder就是一个静态类，使用 ViewHolder 的关键好处是缓存了显示数据的视图（View），加快了 UI 的响应速度。

当我们判断 convertView == null  的时候，如果为空，就会根据设计好的List的Item布局（XML），来为convertView赋值，并生成一个viewHolder来绑定converView里面的各个View控件（XML布局里面的那些控件）。再用convertView的setTag将viewHolder设置到Tag中，以便系统第二次绘制ListView时从Tag中取出。（看下面代码中）

如果convertView不为空的时候，就会直接用convertView的getTag()，来获得一个ViewHolder。

{% highlight java %}

//在外面先定义，ViewHolder静态类
static class ViewHolder
{
    public ImageView img;
    public TextView title;
    public TextView info;
}
//然后重写getView
@Override
public View getView(int position, View convertView, ViewGroup parent) {
    ViewHolder holder;
    if(convertView == null)
    {
        holder = new ViewHolder();
        convertView = mInflater.inflate(R.layout.list_item, null);
        holder.img = (ImageView)item.findViewById(R.id.img)
        holder.title = (TextView)item.findViewById(R.id.title);
        holder.info = (TextView)item.findViewById(R.id.info);
        convertView.setTag(holder);
    }else
    {
        holder = (ViewHolder)convertView.getTag();
    }
        holder.img.setImageResource(R.drawable.ic_launcher);
        holder.title.setText("Hello");
        holder.info.setText("World");
    }
                                                                                                 
    return convertView;
}

{% endhighlight %}

到这里，可能会有人问ViewHolder静态类结合缓存convertView与直接使用convertView有什么区别吗，是否重复了?

在这里，官方给出了解释

提升Adapter的两种方法

    To work efficiently the adapter implemented here uses two techniques:
    -It reuses the convertView passed to getView() to avoid inflating View when it is not necessary
    
    （译:重用缓存convertView传递给getView()方法来避免填充不必要的视图）

    -It uses the ViewHolder pattern to avoid calling findViewById() when it is not necessary
    
    （译：使用ViewHolder模式来避免没有必要的调用findViewById()：因为太多的findViewById也会影响性能）

ViewHolder类的作用

-The ViewHolder pattern consists in storing a data structure in the tag of the view
returned by getView().This data structures contains references to the views we want to bind data to,
thus avoiding calling to findViewById() every time getView() is invoked

（译：ViewHolder模式通过getView()方法返回的视图的标签(Tag)中存储一个数据结构，这个数据结构包含了指向我们要绑定数据的视图的引用，从而避免每次调用getView()的时候调用findViewById()）

实例：用BaseAdapter来自定义ListView布局
main.xml

{% highlight xml %}

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" >
    <ListView
        android:id="@+id/lv"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:fastScrollEnabled="true"
        />
</LinearLayout>

{% endhighlight %}

list_item.xml

{% highlight xml %}

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal" >
    <ImageView
        android:id="@+id/img"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        />
    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        >
        <TextView
            android:id="@+id/tv"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="20sp"
        />
        <TextView
            android:id="@+id/info"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="14sp"
            />
    </LinearLayout>
                                                             
</LinearLayout>

{% endhighlight %}

Activity

{% highlight java %}

public class Demo17Activity extends Activity {
    private ListView lv;
    private List<Map<String, Object>> data;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        lv = (ListView)findViewById(R.id.lv);
        //获取将要绑定的数据设置到data中
        data = getData();
        MyAdapter adapter = new MyAdapter(this);
        lv.setAdapter(adapter);
    }
                                                     
    private List<Map<String, Object>> getData()
    {
        List<Map<String, Object>> list = new ArrayList<Map<String, Object>>();
        Map<String, Object> map;
        for(int i=0;i<10;i++)
        {
            map = new HashMap<String, Object>();
            map.put("img", R.drawable.ic_launcher);
            map.put("title", "跆拳道");
            map.put("info", "快乐源于生活...");
            list.add(map);
        }
        return list;
    }
                                                     
    //ViewHolder静态类
    static class ViewHolder
    {
        public ImageView img;
        public TextView title;
        public TextView info;
    }
                                                     
    public class MyAdapter extends BaseAdapter
    {   
        private LayoutInflater mInflater = null;
        private MyAdapter(Context context)
        {
            //根据context上下文加载布局，这里的是Demo17Activity本身，即this
            this.mInflater = LayoutInflater.from(context);
        }
        @Override
        public int getCount() {
            //How many items are in the data set represented by this Adapter.
            //在此适配器中所代表的数据集中的条目数
            return data.size();
        }
        @Override
        public Object getItem(int position) {
            // Get the data item associated with the specified position in the data set.
            //获取数据集中与指定索引对应的数据项
            return position;
        }
        @Override
        public long getItemId(int position) {
            //Get the row id associated with the specified position in the list.
            //获取在列表中与指定索引对应的行id
            return position;
        }
                                                         
        //Get a View that displays the data at the specified position in the data set.
        //获取一个在数据集中指定索引的视图来显示数据
        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            ViewHolder holder = null;
            //如果缓存convertView为空，则需要创建View
            if(convertView == null)
            {
                holder = new ViewHolder();
                //根据自定义的Item布局加载布局
                convertView = mInflater.inflate(R.layout.list_item, null);
                holder.img = (ImageView)convertView.findViewById(R.id.img);
                holder.title = (TextView)convertView.findViewById(R.id.tv);
                holder.info = (TextView)convertView.findViewById(R.id.info);
                //将设置好的布局保存到缓存中，并将其设置在Tag里，以便后面方便取出Tag
                convertView.setTag(holder);
            }else
            {
                holder = (ViewHolder)convertView.getTag();
            }
            holder.img.setBackgroundResource((Integer)data.get(position).get("img"));
            holder.title.setText((String)data.get(position).get("title"));
            holder.info.setText((String)data.get(position).get("info"));
                                                             
            return convertView;
        }
                                                         
    }
}

{% endhighlight %}

### 目的

减少findViewById()使用，如果不用viewholder的话，So if you have 1000 rows in List and 990 rows will be out of View then 990 times will be called findViewById() again.

Holder design pattern is used for View caching - Holder (arbitrary) object holds child widgets of each row and when row is out of View then findViewById() won't be called but View will be recycled and widgets will be obtained from Holder.

### 定义为static

静态内部类不需要持有外部对象的引用，我理解这个好处和holder也最好为static内部类差不多，不然有可能造成外部对象的内存泄露。

### viewHolder的一种简洁写法

{% highlight java %}

        ViewHolder holder = null;  
        if(convertView == null){  
                convertView = mInflater.inflate(R.layout.xxx null);  
                holder = new ViewHolder();   
                holder.tvXXX = (TextView)findViewById(R.id.xxx);  
                //...一连串的findViewById  
        }  
        else{  
                holder = (ViewHolder) convertView.getTag();    
        }  
           
           
           
        private static class ViewHolder{  
                TextView tvXXX;  
                //很多view的定义  
        }  
        
{% endhighlight %}

这么写一次还行，但问题是总有很多很多的ViewAdapter要这么写，每次都repeat,repeat,repeat累啊。所以，有这么一种简洁的写法分享给大家，先声明，从国外网站上看的，不是自己原创的，但确实很喜欢这个简洁的设计。

ViewHolder这么写（只提供一个静态方法，其实可以加一个私有构造函数防止外部实例化），代码很简单，看过就明白了

{% highlight java %}

public class ViewHolder {  
    // I added a generic return type to reduce the casting noise in client code  
    @SuppressWarnings("unchecked")  
    public static <T extends View> T get(View view, int id) {  
        SparseArray<View> viewHolder = (SparseArray<View>) view.getTag();  
        if (viewHolder == null) {  
            viewHolder = new SparseArray<View>();  
            view.setTag(viewHolder);  
        }  
        View childView = viewHolder.get(id);  
        if (childView == null) {  
            childView = view.findViewById(id);  
            viewHolder.put(id, childView);  
        }  
        return (T) childView;  
    }  
}  

{% endhighlight %}

在getView里这样

{% highlight java %}

@Override  
public View getView(int position, View convertView, ViewGroup parent) {  
   
    if (convertView == null) {  
        convertView = LayoutInflater.from(context)  
          .inflate(R.layout.banana_phone, parent, false);  
    }  
   
    ImageView bananaView = ViewHolder.get(convertView, R.id.banana);  
    TextView phoneView = ViewHolder.get(convertView, R.id.phone);  
   
    BananaPhone bananaPhone = getItem(position);  
    phoneView.setText(bananaPhone.getPhone());  
    bananaView.setImageResource(bananaPhone.getBanana());  
   
    return convertView;  
}  

{% endhighlight %}

### RecyclerView.ViewHolder

提供了非常简便的写法，不用去判断是否需要使用viewholder，recyclerview会自动判断。

{% highlight java %}

@Override 
public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
   View root = LayoutInflater.from(mContext).inflate(myLayout, parent, false);
   return new ViewHolder(root);
}

@Override public void onBindViewHolder(ViewHolder holder, int position) {
   Item item = mItems.get(position);
   holder.title.setText(item.getTitle());
}

@Override public int getItemCount() {
   return mItems != null ? mItems.size() : 0;
}

{% endhighlight %}

-EOF-
