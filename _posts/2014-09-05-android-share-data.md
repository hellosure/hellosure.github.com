---
layout: post
title: 一种通过单例模式共享数据的方法
category: Android
tags: [Android,Java,设计模式,单例,共享,可定制化复用类]
---

### 问题引出

不想通过`Intent`在Activity之间传递数据集。
目的是为了减少数据加载的次数，复用数据。

### 解决方法

#### 通过单例类来共享数据

{% highlight java %}

public class ContactListUtil {

	private static ContactListUtil instance = new ContactListUtil();
	
	private List<ContactBaseBO> listData = new ArrayList<ContactBaseBO>();
	private HashMap<String, Integer> tagLocation = new HashMap<String, Integer>();
	
	private ContactListUtil(){}
	
	public static ContactListUtil getInstance(){
		return instance;
	}
	
	public List<ContactBaseBO> getListData() {
		return listData;
	}

	public void setListData(List<ContactBaseBO> listData) {
		this.listData = listData;
	}

	public HashMap<String, Integer> getTagLocation() {
		return tagLocation;
	}

	public void setTagLocation(HashMap<String, Integer> tagLocation) {
		this.tagLocation = tagLocation;
	}

}
{% endhighlight %}

#### 何时注入数据

1. 数据产生的时候，将数据注入到单例类中。例子中的`ContactListUtil`是用来存放所有联系人的数据集，这个数据集的特点是含义固定，所以数据生成的位置是固定的，不随着所服务的界面而发生变化，而且生成了之后不考虑何时去使用它，仅需要知道肯定会被使用到。

{% highlight java %}

class LoadContactTask extends AsyncTask<Object, Void, Object> {
	@Override
	protected void onPreExecute() {
		progressBar.setVisibility(View.VISIBLE);
	}

	@Override
	protected Object doInBackground(Object... params) {
		initOrderData();
		return null;
	}

	@Override
	protected void onPostExecute(Object result) {
		progressBar.setVisibility(View.GONE);
		refreshContactList(structList.get(structList.size() - 1));
		//产生数据之后，将数据注入到单例类中
		ContactListUtil.getInstance().setListData(sortedData);
		ContactListUtil.getInstance().setTagLocation(tagLocation);
	}
}
{% endhighlight %}

2. 在需要使用该数据的时候，将数据注入到单例类中。例子中的`MultiChatUtil`的特点是当做传入`MultiChatContactActivity`的媒介，从不同的界面跳转到该Activity所需要注入的数据集是不同的，而且注入的数据集也是不同的。

{% highlight java %}

switch (menuID) {
	case CONTACT_MULTI_CHAT:
	if (null != ContactListUtil.getInstance().getListData()
			&& null != ContactListUtil.getInstance().getTagLocation()
			&& !ContactListUtil.getInstance().getListData().isEmpty()
			&& !ContactListUtil.getInstance().getTagLocation().isEmpty()) {
		//在需要跳转到MultiChatContactActivity时，注入所需要的数据集
		MultiChatUtil.getInstance().setListData(ContactListUtil.getInstance().getListData());
		MultiChatUtil.getInstance().setTagLocation(ContactListUtil.getInstance().getTagLocation());
		Intent contactIntent = new Intent(context,MultiChatContactActivity.class);
		contactIntent.putExtra("isRulerMode", true);
		contactIntent.putExtra("isTalkMode", false);
		context.startActivity(contactIntent);
	} else {
		new LoadContactTask().execute();
	}
	return true;
	...
}
{% endhighlight %}

这里再多说两句，`MultiChatContactActivity`的设计中，通过`mode`来实现**复用类的制定化**

{% highlight java %}
public void onCreate(Bundle savedInstanceState) {
	...
	//通过mode来实现定制化
	isRulerMode = getIntent().getBooleanExtra("isRulerMode", false);
	isTalkMode = getIntent().getBooleanExtra("isTalkMode", false);
	
	//使用的时候从单例类中获取数据集
	listData = MultiChatUtil.getInstance().getListData();
	tagLocation = MultiChatUtil.getInstance().getTagLocation();
	...
}
{% endhighlight %}

#### 何时销毁数据

1. `ContactListUtil`的数据可以认为整个应用存活期间都需要，因此不需要显示销毁。

2. `MultiChatUtil`的数据在使用完复用类`MultiChatContactActivity`之后就应该销毁。

{% highlight java %}

protected void onDestroy() {
	super.onDestroy();
	MultiChatUtil.getInstance().setListData(null);
	MultiChatUtil.getInstance().setTagLocation(null);
}
{% endhighlight %}

-EOF-
