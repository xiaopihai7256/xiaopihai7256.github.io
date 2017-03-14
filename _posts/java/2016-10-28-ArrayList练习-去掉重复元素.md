---
layout: post
title: ArrayList练习-去掉重复元素
tags: [java]
author: 杨比轩
---

> 学习阶段，以下不考虑泛型

```
package collectionDemo;

import java.util.*;
/*
 * ArrayList去掉重复元素练习
 * 实现思路：新申请一个集合，每次取一个，判断有没有在新集合中，没有则存储进去，有则什么都不做
 * 
 * 输出结果：
 * [name:李四  age:5, name:赵三  age:8, name:科比  age:13]
 * --------------------------------------------------
 * [name:李四  age:5, name:赵三  age:8, name:科比  age:13]
 * 
*/
public class ArrayListDemo {

	public static void main(String[] args) {
		
		List<Student> list = new ArrayList<Student>();
		
		list.add(new Student("李四",5));
		list.add(new Student("赵三",8));
		list.add(new Student("科比",13));
		list.add(new Student("李四",5));
		list.add(new Student("赵三",8));
		list.add(new Student("科比",13));
		
		//迭代器实现
		List<Student> list2 = new ArrayList<Student>();
		
		Iterator<Student> it =  list.iterator();
		
		Student temp = null;
		while(it.hasNext()){
			temp = (Student)it.next();
			if(!list2.contains(temp)){
				list2.add(temp);
			}
		}
		
		System.out.println(list2);
		System.out.println("--------------------------------------------------");
		
		//不使用迭代器实现
		List<Student> list3 = new ArrayList<Student>();
		
		for(int i= 0; i<list.size(); i++){
			temp = (Student)list.get(i);
			if(!list3.contains(temp)){
				list3.add(temp);
			}
		}
		
		System.out.println(list3);
	}

	
}

class Student{
	private String name;
	private int age;
	
	Student(){
		
	}
	
	
	Student(String name, int age){
		this.name = name;
		this.age = age;
	}


	public String getName() {
		return name;
	}


	public void setName(String name) {
		this.name = name;
	}


	public int getAge() {
		return age;
	}


	public void setAge(int age) {
		this.age = age;
	}
	
	public String toString(){
		
		return "name:"+this.name+"  age:"+this.age;
	}
	
	public boolean equals(Object o){
		
		if(o == null){
			return false;
		}
		if(this == o){
			return true;
		}
		if(o instanceof Student){
			Student s = (Student)o;
			return this.name.equals(s.name) && this.age == s.age;
		}
		return false;
	}

}



```
