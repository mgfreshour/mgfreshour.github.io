---
layout: post
title: Using Zend framework MVC in a subdirectory of an existing project
date: '2011-05-27T13:14:00.000-07:00'
author: Morgan Freshour
tags:
- zend framework
- php
modified_time: '2011-05-27T19:35:54.501-07:00'
thumbnail: http://1.bp.blogspot.com/-EbMAC8nQaNU/TeACyxx82ZI/AAAAAAAAAJE/9dWlE99jSho/s72-c/zf_success.png
blogger_id: tag:blogger.com,1999:blog-3127719415764963943.post-1271370289140156880
blogger_orig_url: http://hillbilly-nerd.blogspot.com/2011/05/using-zend-framework-mvc-in.html
---

I like MVC frameworks.  [Zend framework](http://framework.zend.com) isn't my [favorite](http://en.wikipedia.org/wiki/Comparison_of_web_application_frameworks), but my company is using *an older* Zend framework for it's mobile site.  I might as well stick with it.  Unfortunately, the desktop web site is still a jumbled mess of plain old PHP scripts with logic, database calls, and HTML in a single file.  So what does one do when they want to develop new features in an MVC framework?  Create a new virtual host just for those features? (This is how our mobile site works, it's in _/var/sites/example.com/mobile_) Create your own framework or use one that was designed for standalone use?  No, you figure out how to use the newest version of the Zend framework from within a subdirectory of the current site!!



### Research
First, a quick trip to google..  this looks promising [Zend Framework Run in a Subdirectory](http://www.malcolmhardie.com/weblogs/angus/2008/11/19/zend-framework-run-in-a-subdirectory/)  ... it's a dead link.. crap..  Well, I guess I should just try it and see what happens.



### Experiment
I used the zf.sh command line tool to create a ZF project in a subdirectory _/var/sites/example.com/zf_test_. Changed my .htaccess file to include "RewriteBase /zf_test/public/" and then fired up Chrome and hit the URL.  



Of course I got a blank white screen and a PHP error message.  The odd part was the PHP error had a call stack in the mobile site's bootstrap file.. hmm...  So the problem was that APPLICATION_PATH and APPLICATION_ENV where already being defined in the mobile site's code and for some reason (I'm still not sure why) the mobile site's bootstrap was being called.  So I changed them to DESKTOP_APPLICATION_PATH and DESKTOP_APPLICATION_ENV in _zf_test/public/index.php_ and _zf_test/config/application.ini_.



Save, upload, restart apache for good measure, and hit refresh.  



*PHP Fatal error:  Undefined class constant 'EXCEPTION_NO_ROUTE' in ...* Hmm...  a quick trip to google helped me find this was because the version of zend framework on the server on the one on my development box where a bit off.  So, can I include the full ZF library in the project locally like I can with Rails?  Yep, and almost as easily.  I simply copied the 

_library/Zend_ folder from my local install to _zf_test/library_, uploaded, refreshed :



*PHP Fatal error:  Cannot redeclare class zend_loader in ...* Okay, so something is initializing the old zend loader before my new zend framework can (dang "php_value auto_prepend_file ...").  Well, there can't be that many differences between the versions of zend loader can there?  Let's just get rid of the new one and let the old one handle loading.  

{% highlight shell %}rm -rf /var/sites/example.com/zf_test/library/Zend/Loader*
{% endhighlight %}

Refresh and ... *An error occurred Page not found*


I'm starting to get annoyed. Okay, it looks like it's actually loading something in the App.  After a bit of playing around, I set _resources.frontController.params.displayExceptions = 1_ and find that it's searching for a controller called zf_test.  That's what I was afraid would stop the subdirectory thing from working.  I google and find [Base URL and subdirectories](http://framework.zend.com/wiki/display/PLAY/Base+URL+and+subdirectories) that sounds promising.  So, added to index.php :


{% highlight php %}
<?php
$ctrl  = Zend_Controller_Front::getInstance();
$router = $ctrl->getRouter();
$router->setRewriteBase('/zf_test');
{% endhighlight %}

*PHP Fatal error:  Call to undefined method Zend_Controller_Router_Rewrite::setRewriteBase() in ...* &^$%$#&@$#!^&*!!  I dig into the ZF front controller code and find it has a function setBaseUrl() that is passed to the router.  Well, that's easy. 



{% highlight php %}
<?php
$ctrl  = Zend_Controller_Front::getInstance();
$ctrl->setBaseUrl('/zf_test/');
{% endhighlight %}

Save, upload, refresh...



### Success!!
![Zend Framework Default Screen](/images/zf_success.png)

### Find something else to work on
Okay, now I'm tired of playing with Zend Framework in a subdirectory of a project.  I'll implement something in it later.