---
title: python学习-列表生成式/生成器
date: 2018-05-09 22:10:04
tags: 
     - python
     
categories: 
     - Python
     
---


列表生成式即List Comprehensions，是Python内置的非常简单却强大的可以用来创建list的生成式。

通过列表生成式，我们可以直接创建一个列表。但是如果数据过大会造成内存的严重浪费，如果可以根据某种算法，在循环过程中推算出后面的列表元素，按需取用，就会极大的节省内存空间。

在Python中，这种一边循环一边计算的机制，称为生成器：generator。
<!-- more -->

### 列表生成式：
	
需求：生成一个包含0-9 10个元素的列表：
	
	a = range(10)
	print(list（a))
	打印输出：[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

需求 生成一个0\*0、1\*1、2\*2…………9\*9 10个元素的列表

	a = [x*x for x in range(10)]
	print(a)
	打印输出：[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
需求 生成一个0+5-2、1+5-2…………9+5-2 10个元素的列表

	def f(x):
		return x+5-2
	a = [f(x) for x in range(10)]
	print(a)
	打印输出：[3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
需求 生成1-10中，所有偶数的立方的列表

	a = [x**3 for x in range(1,11) if x%2 == 0]
	print(a)
	打印输出：[8, 64, 216, 512, 1000]
需求 输出“abc”和“123”中字母或数字两两组合的所有可能性，如a1,a3,b2,c1

	a = [x + y for x in "abc" for y in "123"]
	print(a)
	打印输出：['a1', 'a2', 'a3', 'b1', 'b2', 'b3', 'c1', 'c2', 'c3']


### 生成器

　　生成器实际上就是一个可迭代对象（Iterable），可以用for循环遍历

	
##### 创建方式一：
　　将上面的生成式[]改为（）就是生成器了
	
	a = (x for x in range(10))
	print(type(a))
	打印结果：<class 'generator'>   a为生成器
	在取值的时候可以用for循环
	for i in a:
		print(i)
	打印结果：0123456789

	因为是一个一个的取，当i指向1的地址时，之前的0就没有被指向（引用），就会被python内存回收机制给回收，这样就有效的避免了内存浪费

##### 创建方式二：
　　通过yield关键字实现

	def fun1():
		return 1
	def fun2():
		yield 1
	print(type(fun1))
	print(type(fun2))
	输出结果：
	<class 'int'>
	<class 'generator'>

	可见普通函数执行之后生成返回数据，yield返回生成器

使用yield关键字实现输出0-9

	def fun():
		i = 0
		while i <= 9 :
			yield i
			i += 1
	for i in fun():
		print(i)

	输出打印：0123456789

生成器可以用next（生成器对象）来取值

#### 生成器 send（value）方法
send（）方法和next一样，同样可以从生成器中取值，不同的是它可以向生成器中传输数据

**注意：send之前如果没调用next（）方法，第一次send需要传None---send(None)**

	def fun():                   #步骤描述
		print("step1")			# 2
		data = yield 1			#  3  # 6
		print("step2")			# 7
		yield data				# 8
	f = fun()
	print(f.send(None))                #  1    # 4
	print(f.send("lalala"))				#  5   # 9

	输出结果：step1
			1
			step2
			lalala

