---
layout: post
title: tms下遇到的问题之一
thread: 27
categories: 记录
tags: javaEE
---

问题的罗列无序，别结合起来看，也须结合起来看

####问题一

         严重: Exception loading sessions from persistent storage java.io.EOFException


分析:EOFException表示输入过程中意外地到达文件尾或流尾的信号,导致从session中获取数据失败。异常是tomcat本身的问题，由于tomcat上次非正常关闭时有一些活动session被持久化（表现为一些临时文件），在重启时，tomcat尝试去恢复这些session的持久化数据但又读取失败造成的。此异常不影响系统的使用。

对应解决办法：删除Tomcat里面的work/Catalina/localhost下的内容即可解决

####问题二

 		FreeMarker template error!

 ![图1](/assets/java_web/FreeMarker template error.PNG)

网上看看基本有两种原因：
- 这种问题的引起往往是你在action中使用了addActionError这样的方法在汇报actionerror，而你在调用addActionError方法时传入了null值，这时候机会导致这样的问题发生，所以如果发生这样的情况请检查你的代码，看看是否因为某种问题导致给addActionError方法传入了null值，找到了解决这个问题就可以了。
- struts2中的标签漏了必要的属性。例如：用struts2中的checkBoxList标签时没有给标签加name属性，加上name属性后，问题消失。(我中于此情况)
- 参考链接：http://justsee.iteye.com/blog/755631

####问题三

         error:java.lang.ClassNotFoundException: org.apache.commons.lang.xwork.StringUtils
         

这个问题是我在基于SSH框架下引入json包出现的。我的包有点混搭，较新版本取消掉对某些东西的支持，而老版本却是必须的，读不到就报错了。所以最后到换成统一版本。最好不要混包。


####问题四

        HTTP Status 500 - java.lang.reflect.InvocationTargetException
        
        type Exception report
        
        message java.lang.reflect.InvocationTargetException
        
        description The server encountered an internal error that prevented it from fulfilling this request.
        
        exception
        
        net.sf.json.JSONException: java.lang.reflect.InvocationTargetException
        
        ...
        
        java.sql.SQLException: Positioned Update not supported.  

先看我的需求：我要在jsp页面中点击一级下拉选，异步查询其子选项内容（这里的查询是根据外键进行级联关系查询的）当有级联关系查询时，不能直接转化数json数组，要把查询得到的list放到map中，再转化为JSONArray.我的部分代码：

			List<Chapter> list = dao.findBySubjectId(id);
            
			// 有级联关系不能直接转化，要取出List放到map里面
			JsonConfig cfg = new JsonConfig();
            
			// 过滤关联，避免死循环net.sf.json.JSONException:
			// java.lang.reflect.InvocationTargetException
			cfg.setExcludes(new String[] { "handler","hibernateLazyInitializer" });
			// 转化为json字符窜
			Map<String, Object> result = new HashMap<String, Object>();
			result.put("result", list);
			JSONArray array = JSONArray.fromObject(result, cfg);
			chapter = array;
- 解决连接：http://yoyang.iteye.com/blog/651895
- 对于一般的，没有级联关系的查询: http://www.open-open.com/lib/view/open1328948611015.html

####问题五

>json传替数组的问题



JS中将JSON的字符串解析成JSON数据格式，一般有两种方式：http://blog.csdn.net/piaoxuan1987/article/details/8459233

对于json数组，主要要你的嵌套结构，我这里数组嵌套了两层。所以我都是这么访问的

		$.ajax({
			type : "post",
			dataType : "json",
			data : param,
			url : 'selectChapter',
			success : function(data) {				
				if(data != null){
					var chapters = data[0].result;  //注意
					var html = [];					
					for(var i=0;i<chapters.length;i++){
						html.push('<option value="'+chapters[i].id+'">'+chapters[i].name+'</option>');												
						}
					chValue.append(html.join(''));
					}				
			},
			error : function(xhr) {
				alert("有问题");
			}
		});
给张示例图：![图1](/assets/java_web/json.jpg)

####问题六

        HTTP Status 500 - tag 'select', field 'list': The requested list key 'subjectNames' could not be resolved as a collection/array/map/enumeration/iterator type. Example: people or people.{name} - [unknown location]

逻辑问题：我在struts.xml中toAddchapter里面没有写相关的action,没有输出到jsp页面的值，jsp页面中select的name属性没有相对应的list返回值，所以报错了

####问题七

        Unable to load configuration. - bean - jar:file:/D:/apache-tomcat-6.0.37/wtp：webapps/tms/WEB-INF/lib/struts2-dojo-plugin-2.3.16.jar!/struts-plugin.xml:29:136
        Caused by: Unable to load configuration. - bean - jar:file:/D:/apache-tomcat-6.0.37/wtpwebapps/tms/WEB-INF/lib/struts2-dojo-plugin-2.3.16.jar!/struts-plugin.xml:29:136
            ... 22 more
        Caused by: java.lang.NoClassDefFoundError: org/apache/struts2/views/TagLibraryDirectiveProvider
            ... 25 more
        Caused by: java.lang.ClassNotFoundException: org.apache.struts2.views.TagLibraryDirectiveProvider
            ... 34 more

对于这个异常：问题主要在后面。我们能看到：java.lang.ClassNotFoundException: org.apache.struts2.views.TagLibraryDirectiveProvider。  

这是包的问题：在项目之前一直都是用的struts2-2.1.8的jar包。但是在这个版本没有对应的接口跟实现类，所以我换到2.3.16。该版本最小包是（struts2）：

        commons-fileupload-1.3.jar
        commons-io-2.2.jar
        commons-lang-3.3.1.jar
        freemarker-2.3.19.jar
        javassit-3.11.0.GA.jar
        ognl-3.0.6.jar
        struts2-core-2.3.16.jar
        xwork-core-2.3.16.jar



