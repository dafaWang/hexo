---
title: python学习-装饰器
date: 2018-04-02 16:35:42
tags: 
     - python
     - 设计模式
categories: 
     - Python
     
---
装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。

这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。

<!-- more -->

#### python的装饰器是为了实现开闭原则而生的
#### 一旦投入使用的代码就不能做修改了(封闭原则)

比如:   
　　我写了几个方法,现在老板让我在这几个方法的基础上,计算一下方法执行消耗的时间,因为现在这几个方法,有很多客户已经在使用了,所以不能对原函数做修改(封闭原则),而且又得能计算消耗的时间,因为可能要对很多函数做修改,所以最好找一个通用简单的方法
#### 这时候,装饰器就是最好的选择了
比如,我已经写了两个func():

        def func1():
            print("func1  is  running.....")
            time.sleep(1)
        def func2():
            print("func2  is  running.....")
            time.sleep(2)

现在我要用装饰器对现在已有的这两个函数做修改,增加其原有功能
##### 在使用之前,必须要知道python的三个知识点
##### 1.作用域
##### 2.高阶函数
##### 3.闭包

补充：闭包

	def outer(x):
	    def inner(y):
	        nonlocal x
	        x += y
	        print(x)
	    return inner

	a = outer(5)
	a(3)
	a(8)

	输出结果： 8 ， 16 
	inner内部函数，拿到了外部函数outer的局部变量x的引用，在执行到a = outer(5)后，outer函数弹栈，
	但是因为inner有outer函数临时变量的引用，outer将x变量绑定给内置函数inner，所以即使outer函数不在内存中了，
	还是可以使用其局部变量x，x就为 闭包变量（外函数绑定给内函数的局部变量）
	而且根据上述操作可知，闭包变量实际上只有一份，每次开启内函数都在使用同一份闭包变量

	所以，闭包可以用来做单例
	


首先,我们定义一个专门用来计算函数消耗时间的函数spend_time,传入的参数就是我们要计算时间的函数x

        def spend_time(x):  #x为我们要计算的函数---(高阶函数特性  函数可以作为参数)
            start = time.time() #记录开始时间
            x() #要计算的函数执行
            end = time.time()#记录结束时间
            print("spend    %   秒"%(end - start))


        spend_time(func1)
        spend_time(func2)
      #执行结果:
        #func1  is  running.....
        #spend   1.0000572204589844 秒
        #func2  is  running.....
        #spend   2.0001144409179688  秒

这样写确实可以实现功能了,而且也不会太麻烦,但是只能调用spend_time这个方法才可以计算出消耗时间,有什么办法可以让我们调用func1()或者func2()就可以实现呢???

我们对spend_time函数做一下修改

        def spend_time(func):
            def inner():
                start = time.time() #记录开始时间
                x() #要计算的函数执行
                end = time.time()#记录结束时间
                print("spend    %   秒"%(end - start))
            return inner #最后把inner这个函数返回(高阶函数特性,闭包)

这时,我们就可重新给func1和func2赋值

        func1 = spend_time(func1)#这里的spend_time(func1)  实际上是inner函数
        func2 = spend_time(func2)#同上

这个时候直接使用func1和func2函数的结果就和之前使用spend_time()是一样的了

        func1()
        func2()
        #执行结果:
        #func1   is   running...
        #spend   1.0000574588775635 秒
        #func2   is   running...
        #spend   2.0001144409179688 秒

## 这就是装饰器....↑
然而,每次在使用装饰器的时候都要给原有函数重新赋值是不是相当麻烦?
每次都要调用
func1 = spend_time(func1)
func2 = spend_time(func2)
是一件必要且有规律可寻的重复的工作
python已经很好的替我们做了这个工作

### 在我们定义了spend_time函数的基础上,在要修改的函数上面加上@spend_time 这个注解,就可以实现类似
### func1 = spend_time(func1)这样的赋值代码了

    def spend_time(fun):
        def inner():
            starttime = time.time()
            fun()
            end_time = time.time()
            print('spend   %s ' % (end_time - starttime))
        return inner


    @spend_time
    def fun1():
        print('func1   is   running...')
        time.sleep(1)


    @spend_time
    def fun2():
        print('func2   is   running...')
        time.sleep(2)


    # fun1 = spend_time(fun1)
    # fun2 = spend_time(fun2)

    fun1()
    fun2()

    执行结果
    func1   is   running...
    spend   1.0000574588775635 
    func2   is   running...
    spend   2.0001144409179688 



### 回到最初的起点:
**我们要给每个函数增加一些功能的话只需要将功能定义出来,在要修改的函数上加上**
**@装饰器名称就可以了,简单粗暴**


##### 拓展--[装饰设计模式](https://baike.baidu.com/item/%E8%A3%85%E9%A5%B0%E6%A8%A1%E5%BC%8F/10158540?fr=aladdin)


## 带参数的装饰器
### 我们用一个例子来学习
#### 需求:我们需要做一个登陆系统,如果用户没有登录的话就跳转登录并可以选择微博登录或者微信登录,如果已经登录了,就跳转目标页面(主页或者产品页)

**需求本质内容实际上是跳转目标页面,在这个基础上增加登录功能,并在登录功能基础上增加登录类型选择**

首先我们需要一个全局变量来存储用户的登录状态,

    isLogin = False

其次,定义主要跳转函数

    def turn_to_home():
        print('跳转主页面.....')
    def turn_to_prod():
        print("跳转产品页......")

在类似的方法中增加功能,我们使用装饰器

    def check_login(func): #传入具体的跳转操作
        def inner():
            global isLogin #在这里我们需要声明这里的登录判断flag是全局的变量
            if isLogin:
                print("用户已经登录了....")
                func() #执行具体的跳转操作
            else:
                print("用户还未登陆,去登陆........")
                print("登录成功")
                func() #登录成功,执行具体跳转操作
                isLogin = True  #将用户登录状态修改为已登录
        return inner
这样,一个带有登录判断功能的装饰器就写好了
我们重新定义具体登录操作的函数

    @check_login
    def turn_to_home():
        print('跳转主页面.....')
    @check_login
    def turn_to_prod():
        print("跳转产品页......")

    turn_to_home()
    turn_to_prod()


    #运行结果:
    #用户还未登录,去登陆......
    #登录成功
    #跳转主页面......
    #用户已经登录了....
    #跳转产品页

现在,登录判断功能已经添加了,还需要添加一个登录类型区分

我们再定义一个函数,并且将之前的装饰器作为他的局部函数和返回参数放进去,这个函数的参数就可以被我们用来做登录类型区分来使用了

    def check_login_type(login_type):
        def check_login(func): #传入具体的跳转操作
            def inner():
                global isLogin #在这里我们需要声明这里的登录判断flag是全局的变量
                if isLogin:
                    print("用户已经登录了....%s登录" % login_type)
                    func() #执行具体的跳转操作
                else:
                    print("用户还未登陆,去登陆........跳转%s登录"%login_type)
                    print("登录成功")
                    func() #登录成功,执行具体跳转操作
                    isLogin = True  #将用户登录状态修改为已登录
            return inner
        return check_login

我们再重新定义具体跳转方法的时候就可以将登录类型作为装饰器的参数传入

    @check_login_type("微博")
    def turn_to_home():
        print('跳转主页面.....')
    @check_login_type("微信")
    def turn_to_prod():
        print("跳转产品页......")
    turn_to_home()
    turn_to_prod()

    #执行结果:
    #用户还未登录,去登陆......跳转微博登录
    #登录成功
    #跳转主页面......
    #用户已经登录了....
    #跳转产品页
