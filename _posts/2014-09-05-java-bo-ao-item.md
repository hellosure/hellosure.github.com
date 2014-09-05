---
layout: post
title: 通过层层包装实体类实现复杂结构的展示
category: Java
tags: [Java,包装,enum]
---

### 问题引出

数据库中存了大量`type`字段，需要映射为字符串说明，然后还需要将这些数据按照规则分组聚集，之后再作为列表项显示。

### 问题解决

#### BO

创建`BO`来将数据库镜像到内存

`type`字段依然拿`int`或`short`来保存。

#### AO

创建`AO`来利用`Enum`完成`type`字段到字符串的映射，以及其他一些字段的处理

{% highlight java %}

public class NewInstCondAO {
	
	...
	private String limitStr;
	private String timeUnitTypeStr;
	
	public NewInstCondAO(){}
	
	public NewInstCondAO(NewInstCond nic){
		...
		String temp = String.valueOf(nic.getLimit());
		if(temp.endsWith(".0")){
			this.limitStr = temp.substring(0, temp.length()-2);
		}else{
			this.limitStr = temp;
		}
		
		this.timeUnitTypeStr = TimeUnitType.getName(nic.getTimeUnitType());
	}
	
	...
	
}

enum TimeUnitType {  
	ONE("天", 1), 
	TWO("小时", 2),
	THREE("分钟", 3),
	ELEVEN("紧急工作日", 11),
	TWELVE("紧急工作时", 12),
	THIRTEEN("紧急工作分钟", 13);
	
    private String name;
    private int type;

    private TimeUnitType(String name, int type) {
        this.name = name;
        this.type = type;
    }

    public static String getName(int type) {
        for (TimeUnitType c : TimeUnitType.values()) {
            if (c.getType() == type) {
                return c.name;
            }
        }
        return null;
    }

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
	public int getType() {
		return type;
	}
	public void setType(int type) {
		this.type = type;
	}
}  

{% endhighlight %}

#### Item

在`Item`类中完成对`AO`的分组聚集

**分组聚集算法**：先排序，然后遍历。

-EOF-
