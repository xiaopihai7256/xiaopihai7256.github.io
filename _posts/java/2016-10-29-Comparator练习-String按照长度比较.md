---
layout: post
title: Comparator练习-String按照长度比较
tags: [java]
author: 杨比轩
---

练习内容如题，自定义一个比较器StringLengthComparator ，使String按照长度进行排序，短的在前，长的在后，长度相等则使用String的compareTo()进行。

代码如下：
```
package SetDemo;

//调用StringComparator比较器进行比较

import java.util.*;
import java.util.Comparator;;

public class ComparatorDemo {
	public static void main(String[] args) {
		TreeSet<String> ts = new TreeSet<String>(new StringLengthComparator());
		ts.add("abcde");
		ts.add("abc");
		ts.add("bc");
		ts.add("ac");
		
		for(String s : ts){
			System.out.println(s);
		}
	}
}

/*
 * 实现String按照长度排序的比较器
 */
class StringLengthComparator implements Comparator<String>{
	public int compare(String s1, String s2){
		int num = s1.length()-s2.length();
		int num2 = s1.compareTo(s2);
		return num == 0 ? num2 : num;
	}
}
```
程序的输出结果为：
```
ac
bc
abc
abcde

```
