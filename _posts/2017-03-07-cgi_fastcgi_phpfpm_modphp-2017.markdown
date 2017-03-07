---
layout: post
title : cgi、fastcgi、php-fpm、mod_php的理解
subtitle: ""
date: 2017-03-07
author: "chile.zhang"
header-img: "img/post-bg-2016.jpg"
tags:
    - php
---

##cgi、fastcgi、php-fpm、mod_php的理解

### 1、CGI （Common Gateway Interface）
CGI是一种协议，规定了服务器软件和对应解释器程序的数据传输格式，例如一些环境变量（SERVER_NAME，PATH_INFO，CONTENT_TYPE等等）、http的header信息、post数据或者query字符串等等，
这些会传给到CGI处理程序，如php的就是php-cgi程序，处理完后在给回服务器软件。但是这里会有个性能问题，PHP-CGI进程每次启动，都需要读取php.ini文件，初始化执行环境，
完成请求后又会销毁掉进程，这样会导致效率非常低下，所以处理每个进程会很慢。后来就有了FastCGI。

### 2、FastCGI（Fast Common Gateway Interface）
FastCGI是提升了CGI的性能，因为优化了处理流程。首先，FastCGI会先启动一个master进程，解析配置文件，初始化执行环境，然后启动多个worker，
当请求过来时，web服务器通过一个socket长链接来给FastCGI进程，master进程接收到后会传递给一个worker，然后可以立即接受下一个请求。这种模式（称作prefork），
就避免了重复的加载php.ini文件，初始化执行环境，提高效率。

### 3、PHP-FPM （PHP FastCGI Process Manager）
php-fpm是FastCGI的一种实现，其执行流程与FastCGI基本一致，默认情况下监听127.0.0.1:9000，更多的FastCGI进程的配置和管理，如最大进程数（max_children）生成进程的模式（动态、静态）。

### 4、MOD_PHP
mod_php是Apache的一个php模块，这样在Apache进程产生的时候，php的解释器和执行环境就已经加载好，其内部通过SAPI（ Server Application Programming Interface）在Apache和php间传递数据，
这样的模式，就是和Apache同生死，但每个进程都会占用更多的内存，无论是否用到这个模块，如当请求静态文件的时候，php的模块仍然是会加载，所以效率也不会太高，修改php.ini会需要重启Apache。
