---
layout: post
title:  重新认识c++（一）虚表
date:   2016-03-13 12:55:11
category: "C++"
description: 这篇博客简单介绍自己对虚表的理解。
---

### 先谈多态

> 
-  
>  多态就是一个接口，多个方法。多态性允许给父对象赋予子对象的值，父对象可以根据当前赋值给他的子对象的特性以不同的方式运作。把子对象当成父对象来看，可以屏蔽不同对象之间的差异，写出通用的代码。
-  
>  多态函数的调用是在**运行期间**确定的，而函数重载的函数调用是在**编译期间**确定的。 

### 多态是怎么实现的

> 多态是通过虚函数实现的。虚函数允许子类重新定义成员函数。

###  虚函数怎么实现的

> 通过虚函数表。此表位于对象实例中最前沿的部分(不同编译器可能不同)。这个表用来记录此类的虚函数的函数指针，依次为父类的函数指针、子类的函数指针（按声明的顺序排列）。FATHER1_FUTHER2_FUTHER3_SUN1_SUN2_SUN3_end。如果SUN3为对FATHER1的重新定义，那虚表就变成这样的：SUN3_FUTHER2_FUTHER3_SUN1_SUN2_end

###  以撸为证
- **测试一 单继承 不重写基类虚函数** 

{% highlight cpp %}

    class father
    {
    public:
    	father(){};
    	virtual void  father1(){cout << "father::father1" << endl;}
    	virtual void  father2(){cout << "father::father2" << endl;}
    	virtual ~father(){};
    };
    class sun:public father
    {
    public:
    	sun(){};
    	virtual void sun1(){ cout << "sun::sun1" << endl; }
    	virtual void sun2(){ cout << "sun::sun2" << endl; }
    	virtual ~sun(){};
    };
    int main()
    {
    	father f;
    	sun s;	
    	system("pause");
    	return 0;
    }

{% endhighlight %}


> 结果如下:

![这里写图片描述](http://img.blog.csdn.net/20160311223315672)

> 和预期有点不一样啊。。虚表里面没有sun类的虚函数指针。是前面的知识有问题呢，还是编译器显示不完全呢？做实验。


- **测试二  sun虚表中有没有sun类的虚函数指针**

{% highlight cpp %}

    int main()
    {
    	father f;
    	sun s;	
    	typedef void(*FUNC)(void);
    	/*
    		虚函数表地址:(intptr_t*)(&s);
    		虚函数表 — 第一个函数地址:(intptr_t*)*(intptr_t*)(&s)
    	*/
    	intptr_t* vptr_adress = (intptr_t*)(&s);
    	intptr_t* vptr_func1_adress = (intptr_t*)*vptr_adress;
    	/*
    		验证 sun的虚表是不是 father1|father2|sun1|sun2
    	*/
    	FUNC f1 = (FUNC)*vptr_func1_adress;//father1
    	FUNC f2 = (FUNC)*(vptr_func1_adress + 1);//father2
    	FUNC f3 = (FUNC)*(vptr_func1_adress + 2);//析构
    	FUNC f4 = (FUNC)*(vptr_func1_adress + 3);//sun1
    	FUNC f5 = (FUNC)*(vptr_func1_adress + 4);//sun2
    	f1();
    	f2();
    	//f3();
    	f4();
    	f5();
    	system("pause");
    	return 0;
    }

{% endhighlight %}


> 结果如下:

![这里写图片描述](http://img.blog.csdn.net/20160311232007223)

> 结论：和我们原来说的一样：虚表会以继承的顺序（父类_子类）来构建虚表，只是VS虚表显示不完全。还有一个值得注意的细节是father的_vfptr[2]的函数指针是father类的析构函数，sun类的_vfptr[2]的函数指针是sun类的析构函数，这是因为我们把father类的析构函数设置成了虚析构。如果不定义成虚析构会怎样呢?来做个实验。


- **测试三 去掉基类的虚析构**

{% highlight cpp %}

    class father
    {
    public:
    	father(){};
    	virtual void  father1(){cout << "father::father1" << endl;}
    	virtual void  father2(){cout << "father::father2" << endl;}
    ~father(){};
    };

{% endhighlight %}

> 如预期一样，我们的虚函数没有虚函数里，不动态的指定析构函数了。


![这里写图片描述](http://img.blog.csdn.net/20160311233847230)

> 我们来思考一样，我们为什么要将基类的析构定义成虚函数：为了防止内存泄露。因为c++明确指出，当derived classs 对象经由一个base class 指针被删除，而base class 带着一个non-virtual 析构函数，其结果未有定义--实际执行时通常发生的是对象的derived成分没被销毁。

- **测试四 单继承 重写基类虚函数**


{% highlight cpp %}

    #include <iostream>
    using namespace std;
    
    
    class father
    {
    public:
    	father(){};
    	virtual void  father1(){ cout << "father::father1" << endl; }
    	virtual void  father2(){ cout << "father::father2" << endl; }
    	virtual ~father(){};
    };
    class sun :public father
    {
    public:
    	sun(){};
    	virtual void sun1(){ cout << "sun::sun1" << endl; }
    	virtual void sun2(){ cout << "sun::sun2" << endl; }
    	void father1(){ cout << "father::father1" << endl; }//重写父类虚函数
    	virtual ~sun(){};
    };
    int main()
    {
    	father f;
    	sun s;
    	system("pause");
    	return 0;
    }

{% endhighlight %}

>  按我们先前的理论，此时的子类虚表中的父类的father1()应该指向的是sun：：father1。结果是不是这个样子呢？结果见下图:

![这里写图片描述](http://img.blog.csdn.net/20160313105552361)

> 确实是这样子的，这就是为什么父对象可以根据当前赋值给他的子对象的特性以不同的方式运作。单继承我们了解了，那多继承又是什么样子呢？测试一下

- **测试五 多继承 不重写父类虚函数**

{% highlight cpp %}
    
    class father
    {
    public:
    	father(){};
    	virtual void  father1(){ cout << "father::father1" << endl; }
    	virtual void  father2(){ cout << "father::father2" << endl; }
    	virtual ~father(){};
    };
    class mather{
    public:
    	mather(){};
    	virtual void mather1(){ cout << "mather::mather1" << endl; }
    	virtual void mather2(){ cout << "mather::mather2" << endl; }
    	virtual ~mather(){};
    };
    class sun :public father,public mather
    {
    public:
    	sun(){};
    	virtual void sun1(){ cout << "sun::sun1" << endl; }
    	virtual void sun2(){ cout << "sun::sun2" << endl; }
    	virtual ~sun(){};
    };
    int main()
    {
    	father f;
    	mather m;
    	sun s;
    	system("pause");
    	return 0;
    }

{% endhighlight %}

> 这个不好猜啊  直接看结果

![这里写图片描述](http://img.blog.csdn.net/20160313111222432)

>  哇哦，当有多个父类的时候，每个父类都有一个虚表哦。这样比较好啊，比较独立。那我现在的理解是：每个子类有几个‘根’（父类），就有几个‘树’（虚表）。那现在的每个虚表是不是和我们上面讨论的单继承是一个样子呢？做下实验

- **测试六 多继承 重写父类虚函数**


{% highlight cpp %}

    class sun :public father,public mather
    {
    public:
    	sun(){};
    	virtual void sun1(){ cout << "sun::sun1" << endl; }
    	virtual void sun2(){ cout << "sun::sun2" << endl; }
    	void father1(){ cout << "sun::father1" << endl; };//重写 father基类虚函数
    	void mather1(){ cout << "sun::mather1" << endl; };//重写 mather基类虚函数
    	virtual ~sun(){};
    };

{% endhighlight %}

> 结果如下

![这里写图片描述](http://img.blog.csdn.net/20160313113937836)

> 正如你所想，在每个父类的相应的虚表中的函数调用变成了子类的函数调用。而突然间脑子有冒出一个新的问题：当有多个虚表的时候，怎么访问每个虚表呢？。。。做实验

- **测试七  获取多继承中的每个虚表地址及函数地址**

{% highlight cpp %}

int main()
{
	father f;
	mather m;
	sun s;

	typedef void(*FUNC)(void);
	//存虚表的地址
	intptr_t * virtual_address = (intptr_t *)(&s);
	//father虚表的地址
	intptr_t * vptr_f = (intptr_t *)*(virtual_address);
	//mather虚表的地址
	intptr_t * vptr_m = (intptr_t *)*(virtual_address+1);
	//father的虚函数地址
	FUNC f1 = (FUNC)*(vptr_f);
	FUNC f2 = (FUNC)*(vptr_f+1);
	//mather的虚函数地址
	FUNC m1 = (FUNC)*(vptr_m);
	FUNC m2 = (FUNC)*(vptr_m+1);
	//测试
	f1();
	f2();
	m1();
	m2();
	system("pause");
	return 0;
}

{% endhighlight %}

> 其实获取的过程我是在猜测。应该有一个表在存储多个虚表，这个表应该有一个地址，应该是根据声明顺序顺序存储的。每个虚表内的函数应该是根据声明也是顺序存储的。果不其然，结果如下：

![这里写图片描述](http://img.blog.csdn.net/20160313121259846)


- **最后说一个由于一段代码引发的思考**

> 代码

{% highlight cpp %}

	#include <iostream>
	using namespace std;
	
	
	class Base
	{
	public:
	    Base(){};
	    virtual void  Base1(){cout << "Base::Base1" << endl;}
	    virtual void  Base2(){cout << "Base::Base2" << endl;}
	    virtual ~Base(){};
	};
	
	int main()
	{
	    Base*  f = new Base;
	
	    intptr_t* vptr_adress = (intptr_t*)f;
	    intptr_t* vptr_func1_adress = (intptr_t*)*vptr_adress;
	
	    typedef void(*FUNC)(void);
	    FUNC f1 = (FUNC)*vptr_func1_adress;
	    FUNC f2 = (FUNC)*(vptr_func1_adress+1);
	
	    f1();
	    f2();
	
	    delete f;
	
	    f1();
	    f2();
	
	    system("pause");
	    return 0;
	}

{% endhighlight %}

> 代码结果

![这里写图片描述](https://sfault-image.b0.upaiyun.com/104/068/1040681728-56e2f2a93601a)

> **思考**：为什么一个基类被delete后其虚成员函数还可以被访问<br>

> **原因**：如果编译为每个实体分配一个函数实体，当代码量大的时候，得占多少内存啊。这显然不可取。每个类实例的是函数调用的指针。函数是与类一一对应的，而不是与类的实例一一对应的。所以说那个函数是一直存在且唯一存在的。<br>

> **从上得出的另一个结论**：c++其实不安全啊。如果私有话呢？没忍住，又做了一个私有化继承的例子

- **测试八  私有继承 会是什么样子的**

{% highlight cpp %}

    class sun :public father,private mather
    {
    public:
    	sun(){};
    	virtual void sun1(){ cout << "sun::sun1" << endl; }
    	virtual void sun2(){ cout << "sun::sun2" << endl; }
    	virtual ~sun(){};
    };

{% endhighlight %}

>  结果如下：

![这里写图片描述](http://img.blog.csdn.net/20160313125435556)
![这里写图片描述](http://img.blog.csdn.net/20160313125008223)

> 结论：照样可访问，c++不安全啊。。。

- **测试九 对子类实例化过程的一点思考**

{% highlight cpp %}

    #include <iostream>
    using namespace std;
    
    class father
    {
    public:
    	father(){
    		this->father1();//父类调father1
    	};
    	virtual void  father1(){ cout << "father::father1" << endl; }
    	virtual void  father2(){ cout << "father::father2" << endl; }
    	virtual ~father(){
    		this->father1();//父类调father1
    	};
    };
    
    class sun :public father
    {
    public:
    	sun(){};
    	virtual void sun1(){ cout << "sun::sun1" << endl; }
    	virtual void sun2(){ cout << "sun::sun2" << endl; }
    	void father1(){ cout << "sun::father1" << endl; }//重写了父类虚函数father1
    	virtual ~sun(){};
    };
    int main()
    {
    	father * f = new sun;
    	f->father1(); //父类调father1
    	delete f;
    	system("pause");
    	return 0;
    }

{% endhighlight %}

>  解释一下，我做了三次父类调用father1 一次 在父类 构造 一次 在mian（）中给父类赋予子类对象的值，一次在父类析构。结果如下:

![这里写图片描述](http://img.blog.csdn.net/20160313131620929)

为什么结果不同呢？原因是什么呢？debug下

![这里写图片描述](http://img.blog.csdn.net/20160313132324717)
![这里写图片描述](http://img.blog.csdn.net/20160313132349031)
![这里写图片描述](http://img.blog.csdn.net/20160313132416438)


> **分析**： <br>
> **第一次父类调用father1**：父类的虚函数表建好了，而子类的虚函数表没有构造好，构造的调用过程是先基类，后派生类。<br>
> **第二次父类调用father1**：因为虚函数的调用是在运行时决定的，此时父类将虚表指针指向子类虚表，所以调用的是子类中的father <br>
> **第三次父类调用father1**：当调用父类构造时子类的虚表已经没有了，说明析构的调用是先派生类，后基类。虚表的建立与消除大概就是这个顺序。
 
  



