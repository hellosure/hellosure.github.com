---
layout: post
title: Android的viewHolder模式
category: Android
tags: [Android,viewHolder]
---

### 目的

减少findViewById()使用

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


-EOF-
