title: FLEX 面试题总结
date: 2014-3-3 15:39:31 
tags: [Flex面试题]
categories: Flex
---
*作者： 王小虎*
*日期：2014-3-3*

1.  请问用什么方法自定一个事件呢？说下具体的方式

 	先用as创建一个event类，dispatchEvent(创建的对象），然后可以用addEventListener直接响应，也可以使用Event元标签声明，在Mxml标签中使用事件名称来响应  

2.  Flex内存回收原则

	被删除对象在外部的所有引用一定要被删除干净才能被系统当做垃圾回收

3.  内存泄露的解决方法
	
	- 在组件的Remove_from_stage事件回掉中做垃圾处理操作	
	- 利用FLEX性能优化工具profile来对项目进行监控，知道目前那些对象没有进行删除

4.  有没有遇到什么效率问题？ 如何提升FLEX运行效率

	- 避免容器的多级嵌套
	- 使用轻量级容器
	- 处理数据时多用分页的方式

5.  Flex生成的文件都很大，用什么方法进行缩小？

6.  HBox、VBox、Canvas有什么区别？

7.  说一下FLEX组件的生命周期

8.  FLEX中能把CSS编译成SWF嘛？

9.  请列举一下，你认为不错的网站或者blog 

10.  FLEX 里面调用JS方法

11. Flex和后台交互的几种方法，以及适用场合

12. Flex常用的控件  及容器

13. FLEX页面跳转的几种方式

	- 使用viewStack组件
	- 通过state实现跳转
	- NavigateToURL
	- 

 
   
