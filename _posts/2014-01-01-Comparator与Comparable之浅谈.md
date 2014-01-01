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
					return arg1.length() - arg0.length();  
					}  
			     };

		  	System.out.println(list);  
		        Collections.sort(list, comparator);  
  	    		System.out.println(list);  
 		    }  
		 }  

