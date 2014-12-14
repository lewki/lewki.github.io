---
layout: post
title: "Django Apache with WSGI"
categories:
tags:
---

## <p>一、 安装环境检查 </p>

此环境是Centos6.x Mini 安装

iptables 防火墙开放端口80, 22 </p>
{% highlight bash %}
#/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
#/sbin/iptables -I INPUT -p tcp --dport 22 -j ACCEPT
#/etc/init.d/iptables save
{% endhighlight %}
关闭SELINUX
{% highlight bash %}
# setenforce 0
{% endhighlight %}

## <p>二、 依赖包安装</p>
{% highlight bash %}
# yum install wget -y
# wget http://archive.apache.org/dist/apr/apr-1.5.1.tar.bz2
# wget http://archive.apache.org/dist/apr/apr-util-1.5.1.tar.bz2
# yum install unzip python-devel pcre-devel ncurses-devel  sqlite-devel -y              
# yum install libgcc gcc-c++ openssl-devel -y
{% endhighlight %}

安装 Python-mysql
{% highlight bash %}
# wget -c https://pypi.python.org/packages/source/M/MySQL-python/MySQL-python-1.2.4.zip
# unzip MySQL-python-1.2.4.zip
# cd MySQL-python-1.2.4
# python setup.py install 
{% endhighlight %}
更新Python到2.7(Django平台需求 >=2.7)
{% highlight bash %}
# wget -c https://www.python.org/ftp/python/2.7.1/Python-2.7.1.tgz
# tar xf Python-2.7.1.tgz
# cd Python-2.7.1
# ./configure --enable-shared --prefix=/usr/local/python27 
# make && make install

# mv /usr/bin/python{,.bak}
# ln -sv /usr/local/python27/bin/python /usr/bin/python

# cat /etc/ld.so.conf
/usr/local/python27/lib/
include ld.so.conf.d/*.conf

#ldconfig
{% endhighlight %}
检查Python版本号
{% highlight bash %}
# python -V
Python 2.7.1
{% endhighlight %}

修复YUM源
{% highlight bash %}
# vi /usr/bin/yum
#!/usr/bin/python  改为 #!/usr/bin/python2.6 
{% endhighlight %}

##<p>三、 安装Apache2.4</p>
{% highlight bash %}
# http://mirrors.hust.edu.cn/apache//httpd/httpd-2.4.10.tar.bz2
# tar xf httpd-2.4.10.tar.bz2
# tar xf apr-1.5.1.tar.bz2
# tar xf apr-util-1.5.1.tar.bz2

# cp -r apr-1.5.1/ httpd-2.4.10/srclib/apr
# cp -r apr-util-1.5.1/ httpd-2.4.10/srclib/apr-util
# cd httpd-2.4.10/
# ./configure --prefix=/usr/local/httpd --with-included-apr --enable-so --enable-mods-shared=all --enable-cgi --enable-rewrite --with-zlib 
# make && make install

# ln -sv /usr/local/httpd/bin/apachectl /etc/init.d/httpd
# service httpd start
{% endhighlight %}

##<p>四、 安装关联模块mod_wsgi</p>
{% highlight bash %}
# wget https://pypi.python.org/packages/source/m/mod_wsgi/mod_wsgi-4.3.2.tar.gz
# tar xf mod_wsgi-4.3.2.tar.gz
# cd mod_wsgi-4.3.2
# ./configure --with-apxs=/usr/local/httpd/bin/apxs 
# make && make install
{% endhighlight %}

配置 Apache2.4
{% highlight bash %}
# vi /usr/local/httpd/conf/httpd.conf 
添加
LoadModule wsgi_module modules/mod_wsgi.so
{% endhighlight %}
验证mod_wsgi模块
{% highlight bash %}
# /usr/local/httpd/bin/apachectl -t -D DUMP_MODULES
Loaded Modules:
core_module (static)
so_module (static)
http_module (static)
...
wsgi_module (shared)
{% endhighlight %}    


##<p>五、 安装setuptools</p>
{% highlight bash %}
# wget https://pypi.python.org/packages/source/s/setuptools/setuptools-7.0.tar.gz
# tar xf setuptools-7.0.tar.gz
# cd setuptools-7.0
# python setup.py install
{% endhighlight %}

##<p>六、 安装Django</p>
{% highlight bash %}
# wget -c https://www.djangoproject.com/m/releases/1.7/Django-1.7.1.tar.gz
# tar xf Django-1.7.1.tar.gz
# cd Django-1.7.1
# python setup.py install
 添加环境变量：
# PATH=$PATH:/usr/local/python27/bin/
# vi /etc/profile  添加：
  PATH=$PATH:/usr/local/python27/bin/
# source /etc/profile
{% endhighlight %}

##<p>七、 新建Django项目</p>
1.新建工作目录
{% highlight bash %}
# mkdir /var/www
{% endhighlight %}
2.进入工作目录
{% highlight bash %}
# cd /var/www
# django-admin.py startproject luSite [注意项目名大小写]

[root@localhost www]# pwd
/var/www
[root@localhost www]# tree luSite/
luSite/
├── luSitSite
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── views.py
│   └── wsgi.py
└── manage.py
{% endhighlight %}

3. 修改文件views.py   路径：/var/www/luSite/views.py

- views.py:  
{% highlight bash %}
from django.http import HttpResponse, Http404
import datetime

def hello(request):
    return HttpResponse("Hello, nice day")

def current_datetime(request):
    now = datetime.datetime.now()
    html = "<html><body> What is time? now: %s.</body></html>" % now
    return HttpResponse(html)

def hours_ahead(request, offset):
    try:
        offset = int(offset)
    except ValueError:
        raise Http404()
    dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
    html = "<html><body>In %s hour(s), it will be %s.</body></html>" % (offset, dt)
        return HttpResponse(html)
{% endhighlight %}

4. 修改文件urls.py  路径: /var/www/luSite/urls.py
- urls.py
{% highlight bash %}
from django.conf.urls import include, url
from django.contrib import admin
from luSite.views import hello, current_datetime, hours_ahead

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^hello/$', hello),
    url(r'^time/$', current_time),
    url(r'time/plus/(\d{1,2})/$', hours_ahead),
]
{% endhighlight %}


##<p>八、 迁移Django到Apache</p>
新建文件 wsgi.py  路径：/usr/local/httpd/conf/wsgi.py

- wsgi.py
{% highlight bash %}
#!/usr/bin/env python
import os
import sys
path='/var/www/luSite'

if path not in sys.path:
    sys.path.append(path)
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "luSite.settings")
   
from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
{% endhighlight %}

编辑Apache配置文件 httpd.conf </p> 
修改三处地方: vi /usr/local/httpd/conf/httpd.conf </p>
①  添加：</p>
{% highlight bash %}
WSGIScriptAlias  / /usr/local/httpd/conf/wsgi.py
{% endhighlight %}
Tip： 接受来自URL / 下所有请求， 交由文件wsgi.py 处理。</p>
②修改Apache家目录（DocumentRoot）： </p>
{% highlight bash %}
DocumentRoot "/usr/local/httpd/htdocs"
<Directory "/usr/local/httpd/htdocs">
改为：
DocumentRoot "/var/www"
<Directory "/var/www">
{% endhighlight %}
③ 修改根目录访问权限
{% highlight bash %}
<Directory />
    AllowOverride none
    Require all denied
</Directory>
改为：
<Directory />
    AllowOverride none
    Allow from all
</Directory>
{% endhighlight %}

重启Apache：
{% highlight bash %}
# service httpd restart
{% endhighlight %}

</p>
![验证结果](/assets/django.png)

