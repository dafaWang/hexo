---
title: python学习-作用域
date: 2018-05-07 22:24:20
tags: 
     - python
categories: 
     - Python
     
---
## python中的作用域分四种

作用的范围，指它在函数在哪些范围内可以用，而在其他部分不可以，要用就得重新定义。

<!-- more -->

### L：local，局部作用域  -- 函数中定义的变量
### E：enclosing，嵌套的父级函数的局部作用域，即包含此函数的上级函数的局部作用域，但不是全局的 
### G：global,全局变量，模块级别定义的变量
### B：built-in,系统固定模块里面的变量，比如int，bytearray等

## 优先级：
### L > E > G > B


例：

	count = 10
	def outer():
		
		count += 1   #这里报错
	结果报错----------
	原因，全局变量在局部作用域里面是不能被修改的，只能看，不能修改
	换言之：局部作用域不能修改全局作用域的变量

如何在局部作用域修改全局作用域的变量？

	count = 10
	def outer():
		global count
		count += 1
		print(count)
	
	在局部作用域中使用global 关键字声明这是一个全局变量

如何在函数的内置函数中修改父级函数中的变量？

	count = 10
	def outer():
		global count
		count += 1
		print(count)
		num = 5

		def inner():
			num += 3
			print(num)

	这样会报错，在内置函数中不能用global 去声明父级函数中的变量了
	在python3中新增 nonlocal 可以在子级函数中声明父级函数的变量

	count = 10
	def outer():
		global count
		count += 1
		print(count)
		num = 5

		def inner():
			nonlocal num
			num += 3
			print(num)

总结：

	count = 10
	def outer():
	    global count
	    print(count)
	    count += 1
	    print(count)
	    num = 5
	    def inner():
	        nonlocal num
	        num += 1
	        print(num)
	        global count
	        count += 1
	        print(count)
	    inner()
	outer()