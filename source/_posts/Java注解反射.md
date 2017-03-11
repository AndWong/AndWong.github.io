---
title: Java注解反射
date: 2016-06-17 21:10:16
tags: Java
---
一.3种方式获取加载指定的类
Class cla = 类名.class
Class cla = 对象.getClass();
Class cla = Class.forName("name");

Demo demo = (Demo)cla.newInstance(); //加载指定类的对象
demo.a(); //调用对象的方法
<!--more-->
```
public class Test {
	public static void main(String[] args) {
		try {
			Class c = MyDemo.class;
			MyDemo demo = (MyDemo) c.newInstance();
			System.out.println("--->c: " + demo.c); //如果是在同一个包下,则可以调用protected属性和方法,否则只能操作public
			demo.f();
			System.out.println("成员变量个数 : " + c.getDeclaredFields().length); //能取到所有的属性
			System.out.println("公开方法个数 : " + c.getMethods().length);//能取到公开方法,包括父类的
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```
输出:
--->c: 0
--->f
成员变量个数 : 3
公开方法个数 : 10
```
package test2;
public class MyDemo {
	private int a;
	protected int b;
	public int c;

	private void d() {
		System.out.println("--->d ");
	}

	protected void e(){
		System.out.println("--->e ");
	}

	public void f() {
		System.out.println("--->f ");
	}
}
```
二.Field 成员变量
Class c = MyDemo.class;
Field[] fieldArray = c.getFields(); //得到Field数组
```
Class c = MyDemo.class;
MyDemo demo = (MyDemo) c.newInstance();
Field[] fieldArray = c.getFields();
for(int i = 0; i<fieldArray.length;i++){
		Field field = fieldArray[i];
		System.out.println("变量名称 : " + field.getName());
    System.out.println("变量值 : " + field.get(demo));
}
```
输出:
变量名称 : c
变量值 : 0
(只打印出公开成员)

修改访问限制:
```
//获取所有的成员,包含private
Field[] fieldArray = c.getDeclaredFields();
field[0].setAccessible(true);
```
三.Method 方法
Class c = MyDemo.class;
Method[] ma = c.getMethods();
```
Method[] mo = c.getMethods();
for(int i = 0; i<mo.length;i++){
  Method method = mo[i];
  System.out.println("方法名 : " + method.getName());
  System.out.println("方法返回类型 : " + method.getReturnType());
	Parameter[] parames = method.getParameters();//获取参数列表
}
```
invoke()的使用:
```
//getMethod,分别传入方法名,参数类型
//invoke,分别传入对象,参数,没有则为null
c.getMethod("f", null).invoke(demo, null);
```
修改访问限制:
```
//获取所有的成员变量,包含private
Method[] mo = c.getDeclaredMethods();
AccessibleObject.setAccessible(mo, true);
mo[2].invoke(demo, null);
```
四.Constructor 构造器
