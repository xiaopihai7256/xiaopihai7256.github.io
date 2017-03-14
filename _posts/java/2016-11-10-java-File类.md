---
layout: post
title: java-File类
tags: [java]
author: 杨比轩
---

File类将文件和文件夹以及路径封装成对象，以提供更多的方法和属性来操作这些对象。File类不属于流对象，不能读写文件，但是可以作为参数传递给流对象的构造方法。

File学习的核心要点就是要熟练掌握其方法，配合其它对象，实现更多的功能。

## 一、File类中的静态常量

1. Windows下：

   `public static String separator`，这是windows中的目录分隔符：`\\`

   `public static String separatorChar `和上面一样，类型不一样

   `public stativ String pathSeparator`，路径分隔符：`;`

   `public stativ char pathSeparatorChar char`类型的路径分隔符

2. Linux下名字和类型都一样，只是值不一样。目录分隔符为：`/ `路径分隔符为：`:`

## 二、File类下的构造方法

- `File(String pathname) `传递一个字符串类型的路径（路径可以目录的绝对地址，也可以是相对地址，更可以是文件，下同，都称之为路径），将次路径封装成为一个File对象
- `File(String parent, String child) `传递字符串形式的子父路径，注意子和父是相对的概念
- `File(File parent, String child)` 传递字符串形式的子和文件对象的父

## 三、File类的创建方法

以下方法，当目标路径存在时，均会返回false，创建成功返回true

1. `boolean createNewFile()` 创建文件，注意也只能创建文件，并不能创建目录
2. `boolean mkdir() `创建文件夹，注意只能创建一级目录
3. `boolean mkdirs() `创建多级目录

## 四、File类的删除方法

1. `boolean delete()` 删除指定目录或者文件，成功返回true，注意此删除不走回收站，文件夹不为空时没有办法删除
2. `void deleteOnExit()` jvm退出前删除，个人感觉用处不是很广泛

## 五、File类的判断方法

1. `boolean exists() `判断构造方法中封装的路径或者文件是否存在
2. `boolean isAbsolute()` 判断构造方法中封装的路径是不是绝对路径，是返回真
3. `boolean isDirectory() `判断是不是路径，是返回true
4. `boolean isFile()` 判断是不是文件，是返回true
5. `boolean isHidden()` 判断封装的路径（包括文件）是不是隐藏属性是返回true

## 六、File类的获取方法

1. `String getName()` 获取封装路径最后部分的名字，此方法不会判断路径是否存在，使用之前最好判断，如果是文件的话当然获取到的也就是文件的名字了
2. `String getParent()` 返回封装路径的父路径，注意封装的是相对路径时如果相对路径没有父路径，就会出错
3. `File getParentFile() `返回封装路径的父路径的File对象
4. `String getAbsolutePath()` 返回封装路径的绝对路径
5. `File getAbsoluteFile()` 返回封装路径的绝对路径的File对象

## 七、File类的其他方法

1. `boolean renameTo(File f)`重命名，File封装一个路径，这个对象调用`renameTo`(修改后的路径的File对象)，修改成功返回真，失败返回假。修改名字后路径不同，方法`renameTo`带有剪切效果
2. `long lastModified()`返回File构造方法封装的路径中文件的最后修改时间的毫秒值
3. `boolean setLastModified(long time) `能查就能改

## 八、File的list方法

1. `static File[] listRoots()` 静态的方法，列出系统所以的根，光驱和移动设备存储也会算进去
2. `String[] list()`返回File构造方法封装的路径下的所有文件和文件夹
3. `File[] listFiles()`返回File构造方法封装的路径下的所有文件和文件夹，返回的File的对象，带全路径。
> 注意：
如果空目录调用了listFiles()，会返回一个长度为0的File数组，但是如果是文件调用了此方法，则会返回null。前后两者是有实质性区别的。空目录是存在长度为0的数组，文件是不存在，也就是null；

## 九、list方法的文件过滤器

一般遍历目录时，只想要获取特定类型的文件，其他的文件都必须过滤掉，这个就是文件过滤器的用处。

FileFilter只是一个接口，使用的时候需要自己实现来达到过滤文件的目的。FileFilter接口下只有一个accept()方法，所以使用的时候可以考虑使用匿名内部类来实现。具体用法就是讲FileFilter接口的实现对象传入list方法中即可。

> 一般实现accept方法的时候，建议将封装了纯路径的对象返回为true，否则如果剔除掉子目录的话就没有办法递归进入到子目录下。



**以上一到九标题下的所有练习全部提供在一个demo中：**

```java
package file;
import java.io.File;
import java.io.FileFilter;
import java.io.IOException;
import java.util.Date;
import java.text.DateFormat;
/*
 * file的练习
 */
public class FileDemo {

	public static void main(String[] args) throws Exception {
		mehtod_8();
	}
	
	/*
	 *  file类的四个静态常量
	 *  public static String separator 与系统有关的目录分隔符 Windows中 :\ 
	 *	public static char separatorChar Linux中: /
	 * 	public static String pathSeparator 与系统有关的路径分隔符 Windows中  ; 分号
	 *	public static char pathSeparatorChar Linux中  : 冒号
	 */
	public static void method(){
		File f = new File("E:\\");
		String s = f.separator;
		System.out.println(s);
		s = f.pathSeparator;
		System.out.println(s);
	}
	
	/*
	 * file 类的构造方法
	 * 1. File(String pathName)
	 * 2. File(String parent, String child)
	 * 3. File(File parent, String child)
	 */
	public static void method_1(){
		File f = new File("E:\\gama");
		File f1 = new File("E:\\","game");
		File f2 = new File(f,"gama");
		System.out.println(f);
		System.out.println(f1);
		System.out.println(f2);
	}
	
	/* File类的创建方法 ,目标文件或者路径存在时，均会返回false
	 *  1. boolean createNewFile() 创建文件，成功返回true
	 *  2. boolean mkdir() 创建文件夹，成功返回true，只能创建一级目录
	 *  3. boolean mkdirs() 创建多级文件夹
	 */
	public static void method_2() throws IOException{
		File f = new File("E:\\TEST.md");
		System.out.println(f.createNewFile());
		
		File f1 = new File("E:\\TEST\\TEST\\TEST\\");
		System.out.println(f1.mkdirs());
	}
	
	/* File类的删除方法
	 * 1. boolean delete() 删除指定目录或者文件，成功返回true，注意此删除不走回收站，文件夹不为空时没有办法删除
	 * 2. void deleteOnExit() jvm退出前删除，
	 */
	public static void method_3() throws Exception{
		File f = new File("E:\\1.txt");
		//System.out.println(f.delete());
		f.deleteOnExit();
		Thread.sleep(5000);
	}
	
	/* File类的判断方法
	 * 1. boolean exists() 判断构造方法中封装的路径或者文件是否存在
	 * boolean isAbsolute() 判断构造方法中封装的路径是不是绝对路径，是返回真
	 * boolean isDirectory() 判断是不是路径，是返回true
	 * boolean isFile() 判断是不是文件，是返回true
	 * boolean isHidden() 判断封装的路径（包括文件）是不是隐藏属性是返回true
	 */
	public static void method_4() throws Exception{
		File f = new File("E:\\a7d03c85ed46e9d87d8f766ec0be65de");
		
		System.out.println(f.exists());
		
		System.out.println(f.isAbsolute());
	}
	
	/*
	 * File类的获取方法
	 * 1.String getName() 获取封装路径最后部分的名字，此方法不会判断路径是否存在，使用之前最好判断
	 * 2.String getParent() 返回封装路径的父路径
	 * 3.File getParentFile() 返回封装路径的父路径的File对象
	 * 4.String getAbsolutePath() 返回封装路径的绝对路径
	 * 5.File getAbsoluteFile() 返回封装路径的绝对路径的File对象
	 */
	public static void method_5() {
		File f = new File("src");
		System.out.println(f.getAbsoluteFile().getParent());
	}
	
	/*File的其他方法
	 * 1.boolean renameTo(File f)重命名，File封装一个路径，这个对象调用renameTo(修改后的路径的File对象)，
	 *   修改成功返回真，失败返回假。修改名字后路径不同，方法renameTo带有剪切效果
	 * 2.long lastModified()返回File构造方法封装的路径中文件的最后修改时间的毫秒值
	 * 3.boolean setLastModified(long time) 能查就能改
	 */
	public static void method_6() {
		File f = new File("H:\\project\\Demo\\src\\io\\BufferedReaderDemo.java");
		
		long time = f.lastModified();
		DateFormat df = DateFormat.getDateTimeInstance(DateFormat.LONG,DateFormat.LONG);
		String date = df.format(new Date(time));
		
		System.out.println(date);
		
		f.setLastModified(new Date().getTime());

		date = df.format(new Date(f.lastModified()));
		System.out.println(date);
	}
	
	/*
	 * File的list方法
	 * 1. static File[] listRoots() 静态的方法，列出系统所以的根，光驱和移动设备存储也会算进去
	 * 2. String[] list()返回File构造方法封装的路径下的所有文件和文件夹
	 * 3. File[] listFiles()返回File构造方法封装的路径下的所有文件和文件夹，返回的Filed对象，带全路径
	 */
	public static void method_7(){
		
		File[] roots = File.listRoots();
		
		for(File f: roots){
			System.out.println(f);
		}
	}
  
	/*
	 * 文件过滤打印
	 */
	public static void mehtod_8(){
		File dir = new File("G:\\");
		FilesFilter(dir);
	}
	
	//递归过滤方法的实现，选出目录下的*.java文件
	//直接使用匿名内部内实现了FileFilter接口
	public static void FilesFilter(File file) {
		File[] files = file.listFiles(
			new FileFilter(){
				public boolean accept(File pathname){ //使用匿名内部类实现文件过滤器的接口
					if(pathname.getName().endsWith(".java") || pathname.isDirectory()){
						return true;
					}else{
						return false;
					}
				}
			});

		if (files != null) {
			for (File f : files) {
				if (f.isFile()) {
					System.out.println(f);
					
				} else
					FilesFilter(f);
			}
		}
	
	}
}
```



## 十、递归方法

所谓递归就是方法自身调用自身，层层调用，知道最底层的方法完成后一层层的退出知道最外层方法结束。比较的常见的就是文件的遍历和实现斐波那契数列等等。

递归可以一定程度上减少代码量，提高代码的可读性，也容易实现。但是缺点就是不能递归太多次，否则方法调用太多会占用太多的系统资源。

下面给出一个递归遍历全盘的demo：

```java
package file;
/*
 * 一个练习，统计电脑里面文件的总数
 * 思路:递归
 * 
 * 输出（感觉C盘部分文件没有统计到，有点少）：
 * C:\目录下有文件：205263个
 * D:\目录下有文件：0个
 * E:\目录下有文件：34746个
 * F:\目录下有文件：3025个
 * G:\目录下有文件：5616个
 * H:\目录下有文件：141762个
 * 390412
 */
import java.io.File;

public class FileTest {
	public static void main(String[] args) {

		File[] files = File.listRoots();
		long filesNum = 0;
		long temp = 0;
		
		for (File f : files) {
			filesNum = filesNum + countFiles(f);
			System.out.println(f + "目录下有文件：" + (filesNum-temp) + "个");
			temp = filesNum;
		}
		System.out.println(filesNum);
	}

	public static long countFiles(File file) {

		long FilesNum = 0;

		File[] files = file.listFiles();

		if (files != null) {

			for (File f : files) {
				if (f.isFile()) {
					FilesNum++;
					
				} else
					FilesNum = FilesNum + countFiles(f);
			}
		}
		return FilesNum;
	}

  	/*
	 * 斐波那契数列实现
	 */
	public static int getSum(int months){ 
		if(months == 1 || months ==2)
			return 1;
		return getSum(months-1) + getSum(months-2);
	}
}
```
