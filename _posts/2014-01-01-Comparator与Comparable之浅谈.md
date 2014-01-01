---
layout: post
title: Comparator 与 Comparable 之浅谈
thread: 14
categories: 记录
tags: java
---

    故事引之： Comparator 与 Comparable 在听的时候没听出所以然，做题云里雾里
             于是查阅资料，整理得此笔记
   
   在对可排序的集合排序时，我们常用 Comparable 进行排序，当要对某些集合按照自己指定的排序方式排序时，我们就使用 Comparator ,此两个接口都须实现相应的方法
   
   - Comparable

         public class ComparatorDemo {  
   	      public static void main(String[] args) {  
        		List<String> list = new LinkedList<String>();  
	        	list.add("killer");  
		        list.add("adm");  
		        list.add("Marry");  
		        list.add("Boss");  
		        /*
		 		 * 匿名类的方式创建  
		 		 * Comparator 接口中还有一个抽象方法 equals  
		 		 * 但我们不需要实现，因为我们的子类默认继承自Object，而Object 己经实现了equals方法  
		 		 */
		      Comparator<String> comparator = new Comparator<String>() {  
				@Override  
				public int compare(String arg0, String arg1) {  
				        	return arg1.length() - arg0.length(); //以字符串长度进行排序 
					}  
			     };
				System.out.println(list);  
				Collections.sort(list, comparator);  
				System.out.println(list);  
 		    }  
		 }  

  - Comparable 接口
>Comparable接口的定义如下：

    		public  interface  Comparable<T>{
      			 public  int compareTo(T  o);
			}

compareTo方法 与 compare 都返回一个int类型的值，此值不关心具体数值，只关心取值范围，即int的值是一下三种(用 compareTo 做推论，compare 类似)：
 * 返回值 > 0 ：当前对象比给定的对象大
 * 返回值 < 0：当前对象比给定的对象小
 * 返回值 = 0：两个对象相等

  对于两种用法方式简单述之
 当我们需要排序的元素自身已经实现了Comparable接口，并且定义好了比较规则，但这种比较规则不能满足我们对于排序的需求是，我们就需要提供一种额外的比较规则，这时候我们就需要使用Comparator


  以上纯是知识点的复述，现在开始我说迷惑的问题
  
  在给出的例子中，我们能测试能够看出这是一个降序排序的方式，但当我们写成下面方式，则变成升序方式
  
      return arg0.length() - arg1.length();  

一直很疑惑是怎样的一种机制，改变两个值的顺序，就能实现反序排列

在查阅资料跟看源码中有已下心得（用 Comparator 做说明）：

在Comparator和Comparable的接口实现时仅需比较出两个对象的大小，而排序的实现是由Collections.sort()和Arrays.sor()提供的。Collections.sort() 是调用了 Arrays.sort() 方法排序后的效果就是要保证[i-1].compare([i])<=0,就是在[i-1].compare([i]) >0 时交换顺序。换个角度理解就是他总是保持某种意义上的升序排列方式，让前一个对象与后一个对象相比是小于零的（注：虽然在比较的时候参数顺序可调换，但是元素在排序前是没有换顺序的，即按照给定的顺序进行比较）

 * 参考链接： <a href="http://www.blogjava.net/fastunit/archive/2008/04/08/191533.html" target="_blank">点击这里</a>

  于 2014 年 1 月 1 号 完结
