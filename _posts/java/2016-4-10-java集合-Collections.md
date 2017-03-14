---
layout: post
title: java集合-Collections
tags: [java]
author: 杨比轩
---

> 学习阶段，以下不考虑泛型

附继承关系图：


![简化版](http://upload-images.jianshu.io/upload_images/1156415-90200d16cf41ba75.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![完整版](http://upload-images.jianshu.io/upload_images/1156415-949fd9f70e33eb37.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> 1.集合接口：6个接口（短虚线表示），表示不同集合类型，是集合框架的基础。  
2.抽象类：5个抽象类（长虚线表示），对集合接口的部分实现。可扩展为自定义集合类。  
3.实现类：8个实现类（实线表示），对接口的具体实现。

## 一、java集合框架的学习目的

- 学习集合每个容器的特点，包括对象的存储和取出

## 二、顶层接口Collection的方法

集合中顶层接口中所有的抽象方法，下面的子类或者接口都是具备的，接口中的方法都是抽象的，找实现类ArrayList，也就是接口指向实现类，运行的方法都是子类重写后的方法。

### 1.以ArrayList为例

```java
add(Object o); //将元素添加到集合，存储到集合
addAll(collection c); //参数是一个集合，把一个集合存储到另一个集合
void clear(); //清楚集合中的所有对象，但是集合容器还在
int size(); //但会集合中存储元素的个数
boolean isEmpty(); //判断集合中元素个数是不是0，是返回True
boolean contains(Object o); //判断集合众是不是包含这个对象，如果包含返回True
boolean containsAll(Collection c); //一个集合是否完全包含另一个集合，是集合中存储的对象，如果完全包含，返回True
boolean remove(Object o);// 移除集合中的元素，移除成功返回True
//***********
//注意：如果集合中包含多个此元素，remove只移除第一个，也就是一次调用只能移除一个元素

boolean removeAll(Collection c);// 移除两个集合中的相同元素
boolean retainAll(Collection c);//保留两集合中相同的元素也就是取交集,只要原始集合没变，就返回False
toArry(); //集合专数组
```

### 2.方法运行Demo

```java
import java.util.*;
public class CollectionDemo {

	/*
	 * 练习collection中的方法
	*/
	public static void main(String[] args) {
		method();
	}
	
	public static void method(){
		Collection col = new ArrayList();
		Collection col1 = new ArrayList();
		
		col.add("a");
		col.add("b");
		col.add("c");
		col.add("a");
		col.add("a");
		
		col1.add("a");
		col1.add("b");
		col1.add("e");
		
		col.retainAll(col1);
		System.out.println(col.containsAll(col1));
		System.out.println(col.size());
		System.out.println(col);
		
	}
}
```

## 三、迭代器-Iterator

- 迭代器就是从集合中取出元素的方式
- 但是每种集合子类存储对象的方式不同，造成获取对象的方式也不同，因此规定了一个接口（iterator），所有子类的迭代器必须满足接口规则
- 每一个集合的子类中，都有一个方法`iterator()`，返回的就是自己的内部类的对象，但是这个内部类实现了`iterator`接口

```java
iterator iterator() //方法是获取迭代器对象的方法，返回值是一个接口，返回的就是这个接口的实现类
```

### 1.Iterator接口中的方法

```java
boolean hasNext();//判断集合中有没有下一个被取出的元素，有返回true
Object next();//获取集合中的从下一个元素
void remove();//移除元素，就是迭代过程中移除
```

### 2.迭代器的实现源码演示

使用ArrayList为例演示

```java
public class ArrayList extends XXX implements List{
  
  //用方法iterator来返回内部类对象
  public Iterator iterator(){
    return new Itr();
  }
  
  //用内部类来实现迭代器的方法，并设置为私有，不向外暴露
  private class Itr implements Iterator{
    public boolean hasNext();
	public Object next();
	public void remove();
  }
}

//外部调用的方式
Iterator it = new ArrayList对象.iterator();
```

### 3.迭代器取集合中元素的步骤

- 调用集合方法`iterator()`获取迭代器接口的实现类对象
- 调用接口`Iterator`方法`hasNext()`判断集合中有没有下一个元素
- 调用接口`Iterator`方法`next()`获取集合中的元素

```java
public static void main(String[] args) {
		//创建集合对象ArrayList
		Collection con =new ArrayList();
		
		con.add(1);
		con.add(2);
		con.add(3);
		con.add(4);
		
		//使用迭代器获取集合中的元素
		Iterator it = con.iterator();
		
		while(it.hasNext()){
			System.out.println(it.next());
			it.remove();
		}

		while(it.hasNext()){
			System.out.println(it.next());
		}
	}

/*输出结果
1
2
3
4
*/
```

### 4.迭代器注意事项

- 如果`hasNext()`判断为`false`，还用`next()`获取，出现没有元素被取出异常
- 在迭代的过程中，不允许使用集合的方法改变集合长度，出现并发修改异常
- 迭代器创建好以后，只能迭代一次
- 一次迭代中，不允许出现多次`next`方法

## 四、list接口派系

Lsit接口继承Collection接口，自己成立一个派系。特点（有序，重复，下标）：

- 存储是有序的。顺序：存储和取出的顺序相同
- 允许存储重复
- 元素都有下标

### 1.List接口特有方法

```java
void add(int index,Object o)//在列表的指定位置上插入元素
boolean addAll(int index,Collection c)//在列表的指定位置上插入另一个集合
Object get(int index)//返回指定索引上的元素！
Object remove(int index)//移除指定下标上的元素，返回移除前的对象
Object set(int index,Object o)//修改指定下标上的元素，返回修改前的元素
List subList(int start,int end)//获取集合中的一部分，包含头，不包含尾，返回新集合
```

demo:

```java
import java.util.ArrayList;
import java.util.List;

/*
 * List接口的特有方法
*/
public class ListDemo {
	public static void main(String[] args) {
		List list = new ArrayList();
		
		list.add("a");
		list.add("b");
		list.add("c");
		list.add("d");

		/*
		 * void add(int index,Object o)
		 * 在列表的指定位置前插入元素
		*/		
		list.add(2,"z");
		System.out.println(list);

		/*
		 *  void addAll(int index,Collection o)
		 * 在列表的指定位置插入另一个集合
		 * 插入的集合为空时，返回false
		*/
		
		List list2 = new ArrayList();
		
		list2.add("e");
		list2.add("f");
		list2.add("g");
		list2.add("h");
		
		list.addAll(5,list2);
		System.out.println(list);
		
		/*
		 * Object get(int index);
		 * 返回指定下标上的元素
		*/
		for(int i = 0;i<list.size();i++){
			System.out.print((String)list.get(i)+',');
		}
		System.out.print('\n');
		/*
		 * Object remove(int index)
		 * 移除指定下标上的元素
		*/
		list.remove(0);
		System.out.println(list);

		/*
		 * Object set(int index, Object o);
		 * 修改指定位置上的元素，并返回修改前的元素
		*/
		System.out.println(list2.set(0, "a"));
		System.out.println(list2);
		
		/*
		 * List subList(int start, int end)
		 * 获取集合一部分，有头无尾
		*/
		System.out.println(list.subList(1, 3));
	}
}
```

###2. List派系特有迭代器

迭代器只能应用于**List**集合，其他集合不能用。

**List**特有迭代器接口**ListIterator**，这个接口是**Iterator**的子接口。

**List**的派系的每一个子类，都有一个方法**listIterator**，返回**ListIterator**接口的实现类对象，这个实现类，就是集合的内部类。

 #### 2.1特点

迭代中，可以使用迭代器方法，进行添加，修改，删除集合中的元素，两个方向遍历

####2.2方法

```java
	 add(Object)//添加，迭代到哪个对象，添加到这个对象的后面
     set(Object)//修改，迭代到哪个对象，修改的就是哪个对象
     remove()//删除
     hasPrevious() == hasNext() //逆向使用
     previous() == next() //逆向使用

```

### 3.ArrayList类

- 容器的底层实现方式，可变长度数组(扩容，自动复制数组)，数组默认大小10个长度。
- 线程不安全集合，运行速度快。查询速度快，增删慢
- 每次增长50%。
- ArrayList使用的频率最高，速度快，查询也快
- 可以存储`NULL`

### 4.Vector类

开始版本JDK1.0 ，升级到JDK1.2的时候，才出现了集合框架的概念。类Vector在JDK1.2版本后，改为实现List接口。Vector类底层实现，也是可变数组，增长率100%，线程安全，运行速度慢。 从JDK1.2版本开始，Vector被ArrayList取代

### 5.LinkedList类

- 特点和List接口，一致，重复，有序，下标
- LinkedList底层实现原理：链表结构，查询慢，增删快，线程不安全，运行速度快
- 存储和取出，实现代码，和ArrayList完全一致

#### 5.1LinkedList特有方法

```java
addFirst(Object o);//头部添加元素
addLast(Object o);//尾部添加元素
Object getFirst();//获取头部
Object getLast();//获取尾部
Object removeFirst();//移除头部
Object removeLast();//移除尾部
  
//在LinkedList中，1.6版本开始，添加了几个新的方法，来操作集合
boolean offerFirst()  boolean offerLast() //替换的了 addFirst() addLast(),返回永真
Object peekFirst()  Object peekLast() //替换了 getFirst() getLast().如果集合中是空的，获取后返回null不会出现异常
Object pollFirst()  Object pollLast()// 替换了 removeFirst() removeLast().如果集是空的返回null,不会出现异常

```

#### 5.2LinkedList模拟队列和堆栈

- 堆栈：内存中的数据，先进去的，后出来
- 队列：内存中的数据，先进去的，先出来
- 特点：符合LinkedList类的特点，存储对象的时候，开头结尾，获取对象的时候，可以在开头和结尾获取。

```java
package collectionDemo;

import java.util.*;

/*
 * 通过LinkedList实现队列和栈
*/

public class LinkedListTest {
	public static void main(String[] args) {
		Heap<String> h = new Heap<String>();
		
		h.put("a");
		h.put("b");
		h.put("c");
		h.put("d");
		h.put("e");
		h.put("f");
		
		System.out.println(h);
		
		System.out.println(h.pop());
		System.out.println(h.pop());
		System.out.println(h.pop());
		System.out.println(h.pop());
		System.out.println(h.pop());
		System.out.println(h.pop());
		System.out.println(h.pop());
		
		Queue<Integer> q = new Queue<Integer>();
		
		q.put(1);
		q.put(2);
		q.put(3);
		q.put(4);
		q.put(5);
		System.out.println(q);
		System.out.println(q.pop());
		System.out.println(q.pop());
		System.out.println(q.pop());
		System.out.println(q.pop());
		System.out.println(q.pop());
		System.out.println(q.pop());

	}
	

}

//实现栈
class Heap<E>{ 
	private LinkedList<E> h ;
	
	Heap(){
		this.h =  new LinkedList<E>();
	}
	
	public void put(E o){
		this.h.addFirst(o);
	}
	
	public E pop(){
		
		return this.h.pollFirst();
	}
	
	public String toString(){
		
		return this.h.toString();
	}
}

//实现队
class Queue<E>{
	
	private LinkedList<E> q;
	
	Queue(){
		this.q = new LinkedList<E>();
	}
	
	public void put(E o){
		this.q.addFirst(o);
	}
	
	public E pop(){
		return this.q.pollLast();
	}
	
public String toString(){
		
		return this.q.toString();
	}

}
```
