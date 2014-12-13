##使用gevent进行异步、并发开发

	from gevent.monkey import patch_all  
	import gevent
	import time
	import random
	
	def go(x):
		time.sleep(random.random())
		print x
	
	def run(n):
		patch_all(
		        socket=True,
		        dns=True,
		        time=True,
		        select=True,
		        thread=False,
		        os=True,
		        ssl=True,
		        httplib=False,
		        aggressive=True
		    )
	
		threads = []
		for i in range(n):
			#go(i)
			threads.append(gevent.spawn(go,i))
		gevent.joinall(threads)
	
	run(20)
