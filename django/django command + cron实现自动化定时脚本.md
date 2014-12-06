#django command + cron实现自动化定时脚本


	1.首先把需要自动执行的django method写成django command
	2.将自己定义的django command添加到cron中使用cron服务实现定期执行


###第一步：添加自定义的django command
---

####1.在django项目app目录下创建management目录及command目录
我们自己建立的application叫做app在这个app目录下，需要新建management目录，这个目录里应该包括： _ _ init _ _ .py（内容为空，用于打包）和commands目录，然后在commands目录下包括： _ _ init _ _.py和mycommand.py ，其中 mycommand.py就是用于我们自定义command的方法，目录的树状结构如下：


	  .
	  ├── app
	  │   ├── __init__.py
	  │   ├── management
	  │   │   ├── commands
	  │   │   │   ├── __init__.py
	  │   │   │   └── mycommand.py
	  │   │   └── __init__.py
	  │   ├── models.py
	  │   ├── urls.py
	  │   └── views.py
	  ├── __init__.py
	  └── manage.py


####2.定义django command命令
在mycommand.py的command进行内容书写，简单示例如下：

	from django.core.management.base import BaseCommand           
	 
	class Command(BaseCommand):
	    def handle(self, *args, **options):         
	        print "everybody boom"

通过以上设置，就可以直接使用 	
	
	python manage.py mycommand

直接运行 Command 命令，打印 *everybody boom*.

详细django命令说明：[https://docs.djangoproject.com/en/dev/howto/custom-management-commands/](https://docs.djangoproject.com/en/dev/howto/custom-management-commands/ "django command")


###第二步：执行linux cron自动化脚本
---

####首先确定自己的cron服务是开启的：

	sudo service crond start
	#sudo service crond stop
	#sudo service crond restart

####一个简单的shell脚本：
	#!bin/bash
	p=$PWD
	touch $p/djangocron
	touch $p/djangocron.log
	echo "0 6,12,18 * * 1-5 python $p/manage.py mycommand > $p/djangocron.log 2>&1" > djangocron
	crontab djangocron 
	crontab -l

说明：

	这个脚本放在与manage.py同一目录下，以便p=$PWD获取能用的当前路径。 
  touch先建立一个djangocron文件，用于存放crontab的命令，建立的djangocron.log用来存输出信息。

这一句写入djangocron文件，就是cron的配置了 0 6,12,18 * * 1-5这几个参数未改动的格式是 * * * * * 五个参数使用空格隔开分别表示 分钟 小时 天 月 周，我这里的配置就是每个周一到周五的6点0分 12点0分 18点0分执行后面的命令; 

后面的命令 python $p/manage.py mycommand，$p是路径的引用，其实就是 python manage.py mycommad.   

小箭头 > $p/djangocron.log 2>&1的意思就是 将命令的输出结果转存到djangocron.log文件，2>&1的意思是同时将错误信息也存入djangocron.log文件。 

crontab djangocron的作用是将我们写的djangocron文件在crontab中装载，装载后可以： 

	crontab -l #查看
	crontab -e #修改

只要调整cron前面设置的时间参数，就可以测试自己的命令有没有成功执行，查看djangocron.log来查看输出和error。

####使用corn运行多个django command定时脚本：
command.sh:

	#!bin/bash

	p=$PWD
	touch $p/djangocron
	touch $p/djangocron.log
	echo "30 * * * * python $p/manage.py hour_command > $p/djangocron.log 2>&1" > djangocron
	echo "59 23 * * * python $p/manage.py day_command > $p/djangocron.log 2>&1" >> djangocron
	crontab djangocron
	crontab -l

这个脚本设置了两条定时命令：

	1.每个小时的第30分钟运行一次 hour_command 命令。
	2.每天的23点59分运行一次 day_command 命令。

使用 **sh command.sh** 启动 corntab 定时脚本。
