<h3>使用vps部署django:uwsgi+nginx</h3>

<h2>配置：</h2>

<ul>
<li>centos 6.5 32位</li>
<li>python 2.7.5</li>
<li>django 1.5.5</li>
<li>nginx 1.6.0</li>
<li>uwsgi 2.0.3</li>
</ul>

<p>第一次使用vps,测试了下django+nginx+uwsgi的配置,把其中遇到的一些问题和相关过程做了个记录,
vps的系统是centos6.5 ,所以自带的python版本是2.6.6,首先升级python到自己熟练的版本,我选择的是
python2.7.5。
</p>

<h3>升级python:</h3>
<pre><code>1.下载python安装包：
	wget --no-check-certificate http://www.python.org/ftp/python/2.7.5/Python-2.7.5.tgz
2.解压：
	tar -xzvf Python-2.7.5.tgz
3.编译安装： 
	cd Python-2.7.5      （yum -y install gcc）        
	./configure   （yum -y install gcc automake autoconf libtool make ）
	make && make install
	（Python 默认安装目录在/usr/local/lib/python2.7
4、更改系统默认版本：
	mv /usr/bin/python /usr/bin/python.bak
	ln -s /usr/local/bin/python2.7 /usr/bin/python
	输入 python -V 查看默认版本是否修改成功。
5、修复不能正常工作的yum：
	在完成了上面4步之后，如果有使用yum的话会发现出错，这是因为yum 依赖2.6.6而现在默认的 Python 版本是 2.7.5 。
	vim /usr/bin/yum
	将首行显示的 !#/usr/bin/python 修改为 !#/usr/bin/python2.7 （centos5是python2.4，centos6是python2.6）
<b>注意:</b>
第三步括号中内容,是依赖包,需要提前使用yum命令安装,
如果不安装gcc,编译./configure时会出现如下错误：
	configure: error: in `/root/kola/Python-2.7.5':
	configure: error: no acceptable C compiler found in $PATH
	See `config.log' for more details	
同样make之前需要安装： yum -y install gcc automake autoconf libtool make
</code></pre>

<h3>安装setuptools、pip:</h3>
<h4>安装setuptools:</h4>
<pre><code>1.下载：
	wget http://pypi.douban.com/packages/source/s/setuptools/setuptools-3.3.tar.gz
2.解压安装：  
	tar zxvf setuptools-3.3.tar.gz
	cd setuptools-3.3
	python setup.py build
	sudo python setup.py install
注意如果安装失败：
	缺少zlib，安装setuptools时出错。
		issue: RuntimeError: Compression requires the (missing) zlib module
		<b>yum install zlib zlib-devel -y</b>
	重新make Python2.7再安装
		cd ../Python-2.7.6
		make # 这时可以发现之前make时缺了不少模块
		make install
</code></pre>

<h4>安装pip:</h4>
<pre><code>
直接安装:easy_install -i http://pypi.douban.com/simple pip
如果出现错误：
	# issue: ImportError: cannot import name HTTPSHandler
则安装：yum install openssl openssl-devel -y
重新编译安装python:
进入之前的python 2.7.5文件夹：
	make && make install
</code></pre>

<h3>安装django:</h3>
<code>pip install Django==1.5.5</code>

<h2>安装配置uwsgi:</h2>
<p>直接通过pip安装：
	pip install uwsgi</p>
<h5>第一个wsgi程序：</h5>
<pre><code>
#test.py
def application(env,start_response):
	start_response('200 OK',[('Content-Type','text/html')])
	return ["hello world"]
</code></pre>
<code>测试运行uwsgi:uwsgi --http :9090 --wsgi-file test.py</code>
通过浏览器 ： [vpsip]:9090 看到常见的 hello world 界面则配置成功
