<h1>使用vps部署django:uwsgi+nginx</h1>

<h2>vps环境：</h2>
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

<h2>环境配置：</h2>
<pre><code>安装一些必要的依赖包：
升级包：yum update
安装一些必要包：yum install python python-devel libxml2 libxml2-devel python-setuptools 
zlib-devel wget openssl-devel pcre pcre-devel sudo gcc make autoconf automake
</code></pre>

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
		cd ../Python-2.7.5
		make # 这时可以发现之前make时缺了不少模块
		make install
</code></pre>

<h4>安装pip:</h4>
<pre><code>直接安装:easy_install -i http://pypi.douban.com/simple pip
如果出现错误：
	# issue: ImportError: cannot import name HTTPSHandler
则安装：yum install openssl openssl-devel -y
重新编译安装python:
进入之前的python 2.7.5文件夹：
	make && make install
</code></pre>

<h3>安装django:</h3>
<code>pip install Django==1.5.5</code>

<h2>配置uwsgi:</h2>
<h3>安装uwsgi与测试：</h3>
<p>直接通过pip安装：
	pip install uwsgi</p>
<h5>第一个wsgi程序：</h5>
<pre><code>#test.py
def application(env,start_response):
	start_response('200 OK',[('Content-Type','text/html')])
	return ["hello world"]
</code></pre>
<code>测试运行uwsgi:uwsgi --http :9090 --wsgi-file test.py</code>
<p>通过浏览器 ： [vps的ip]:9090 看到常见的 hello world 界面则配置成功</p>
<h3>uwsgi配置django：</h3>
<p>建立一个django项目mysite,在mysite下建立文件django_wsgi.py 文件，和manage.py文件同级目录
配置如下：</p>
<pre><code>#!/usr/bin/env python
# coding: utf-8

import os
import sys

# 将系统的编码设置为UTF8
reload(sys)
sys.setdefaultencoding('utf8')

#注意："mysite.settings" 和项目文件夹对应。
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")

from django.core.handlers.wsgi import WSGIHandler
application = WSGIHandler()
</code></pre>
<pre><code>配置好后，直接运行：
uwsgi --http :8000 --chdir /home/google/kola/mytest/mysite --module django_wsgi
进入浏览器： [vps的ip]：8000 就能看到django的欢迎界面
</code></pre>
<h4>uwsgi进程关闭</h4>
<pre><code>使用awk选出所有的进程id:
# ps -ef|grep uwsgi|grep -v grep|awk '{print $2}' 
使用xargs kill所有进程:
# ps -ef|grep uwsgi|grep -v grep|awk '{print $2}'|xargs kill -9
</code></pre>

<h2>nginx安装配置:</h2>
<h3>编译安装nginx:</h3>
<pre><code>下载：
	wget http://nginx.org/download/nginx-1.6.0.tar.gz
解压: 
	tar -zxvf nginx-1.6.0.tar.gz
编译：
	./configure --prefix=/usr/local/nginx # 编译选项配置，这里从简
编译安装：
	make && make install # 编译安装
将生成的nginx可执行文件在/usr/sbin里建立软链接 ：
	ln -s /usr/local/nginx/sbin/nginx /usr/sbin/nginx</code></pre>
<p>安装成功后，直接运行命令 “nginx”，进入浏览器输入vpsIP，看到welcome to nginx则配置成功</p>
<h3>django+nginx+uwsgi配置：</h3>
<p>在django项目文件夹新建django_socket.xml(跟manage.py同一目录)：</p>
<pre><code><uwsgi>
    <socket>:8077</socket>
    <chdir>/root/mysite</chdir>
    <module>django_wsgi</module>
    <processes>4</processes> <!-- 进程数 --> 
    <daemonize>uwsgi.log</daemonize>
</uwsgi>
</code></pre>
<h3>修改nginx配置文件/usr/local/nginx/conf/nginx.conf：</h3>
<pre><code><b>注意logs日志文件夹、static、media文件夹都必须存在</b>
server {
        listen 80;
        server_name localhost;
		
        access_log /var/log/test/access.log;
        error_log /var/log/test/error.log;
		
        #charset koi8-r;
        #access_log logs/host.access.log main;
		
        location / {
         include uwsgi_params;
         uwsgi_pass 127.0.0.1:8077;
        }
		
        #error_page 404 /404.html;
        # redirect server error pages to the static page /50x.html
        #
		
        error_page 500 502 503 504 /50x.html;
		
        location = /50x.html {
            root html;
        }
		
        location /static/ {
            alias /root/mysite/static;
            index index.html index.htm;
        }
        location /media/ {
            alias /root/mysite/media/;
        }
    }</code></pre>
<p>配置成功后，
重启nginx：
nginx -s reload </p>

<h4>uwsgi + nginx + django 测试运行：</h4>
<pre><code>进入django项目文件夹：
运行uwsgi命令：
uwsgi -x django_socket.xml
访问uwsgi.log日志看是否有异常发生，如无异常，打开浏览器，访问[vps的ip地址] ;
单独使用Django启动的程序一模一样时，就说明成功！
</code></pre>
<h4>停止nginx:</h4>
<pre><code>强行停止nginx所有进程：
	pkill -9 nginx  
从容停止：
	kill -QUIT 【Nginx主进程号】  
快速停止：
	kill -TERM 【Nginx主进程号】  或  kill -INT 【Nginx主进程号】
PS：（ps -ef|grep命令获得master进程的PID）</code></pre>
<h4>附录django测试项目的树目录：</h4>
<pre><code>.
├── django_socket.xml
├── django_wsgi.py
├── manage.py
├── media
├── mysite
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── views.py
│   └── wsgi.py
├── static
└── uwsgi.log
</code></pre>
