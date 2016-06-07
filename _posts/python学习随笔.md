* 包，模块，命名空间

python命名空间有这几层：包，模块，函数/变量/类，package.module.func/var/class，module1.foo与module2.foo是隔离的，互不冲突。

一个目录带上__init__.py文件就成为包

一个xxx.py就是一个模块。package1.module和package2.module互不冲突。模块中通常写上如下的模板，可以用`python xxx.py`的形式直接执行；其它模块`import xxx`时不会执行模板语句，这有利于模块测试。

	def main():
		pass

	if __name__=='__main__'：
		main()

* 运算符过载

\* and + overloaded operators:different things for different data type

* 动态解释语言，运行时检查

    如下示例的main函数中if语句有个明显错误，但运行时不会报错，因为运行时不会执行到错误语句，解释器不会去检查。

		#!/usr/bin/env python
		# -*- coding: utf-8 -*-

		def repeat(s, exclaim):
    		#result = s + s + s
    		result = s*10
    		if exclaim:
        		result = result + '!!!'
    		return result

		def main():
    		r = repeat(10, '')
    		print r
    		r = repeat('-', '')
    		print r
    		if r=='bill':
        		repeatttt(2, True)
    		else:
        		print 'wow'

		if __name__ == '__main__':
    		main()
