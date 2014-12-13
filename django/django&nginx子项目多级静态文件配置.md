##django&nginx子项目多级静态文件配置
---
###1.django多静态文件目录配置

	STATIC_URL = '/static/'
	# Additional locations of static files

	STATICFILES_DIRS = (
	    # Put strings here, like "/home/html/static" or "C:/www/django/static".
	    # Always use forward slashes, even on Windows.
	    # Don't forget to use absolute paths, not relative paths.
	    'app1/static',
	    'app2/static',
	)

需要说明的一点，django是依次查找静态文件目录，找到后停止，找不到则抛出404。

###2.nginx多静态文件目录配置

    location ^~/static/ {
        #root /www/project/app/;
        root /www/project;
        try_files /app1$uri /app2$uri =404;
    }
