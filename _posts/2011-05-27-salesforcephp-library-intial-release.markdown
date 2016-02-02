---
layout: post
title: Salesforce.php library intial release
date: '2011-05-27T07:59:00.000-07:00'
author: Morgan Freshour
tags:
- activerecord
- php
- salesforce
modified_time: '2011-05-27T19:35:40.304-07:00'
blogger_id: tag:blogger.com,1999:blog-3127719415764963943.post-4532259550327871331
blogger_orig_url: http://hillbilly-nerd.blogspot.com/2011/05/salesforcephp-library-intial-release.html
---

Okay, so "initial release" sounds a little more important than is true.  I'm just open sourcing the small object mapping library for the [Salesforce.com partner API](http://wiki.developerforce.com/index.php/Force.com_Toolkit_for_PHP). It's on github [Salesforce Lib](https://github.com/mgfreshour/salesforce.php).


A little background may be in order.  One day my boss tells me, "We want to integrate our site with Salesforce.com."  My initial reaction was "cool, a few more weeks of job security."  After a decent amount of work learning about the Salesforce API and creating an prototype library for this project, the project was dropped.  So now, I've posted that initial library to github incase other would find it useful.


The idea behind it was to provide an [ActiveRecord](http://martinfowler.com/eaaCatalog/activeRecord.html)'ish interface to the salesforce partner API.  The salesforce_Table class is meant to be a base class for the table mapping classes.

{% highlight php %}
<?php
class mySalesforceAccount extends salesforce_Table {
  public function __construct() {
    parent::__construct('Account');
  }

  public function DoBusinessStuffToAccount($param) {
    // ...
  }
}
{% endhighlight %}


Then one could use the child object to manipulate data in the Salesforce Accounts table.

{% highlight php %}
<?php
// ...
$account = new mySalesforceAccount();
$account->getById($account_id);

if ($account->name == 'Acme Co.') {
  echo 'Don\'t buy rockets!!  They are defective!!'
}
{% endhighlight %}

There is [some] support for loading the layouts from Salesforce's meta data.  Currently salesforce_TableLayout handles the display and only does very basic HTML.  The future plan was to make this an abstract base class so different displays for layouts would be created by extended the classes.



{% highlight php %}
<?php
$account = new mySalesforceAccount();
$account->getById($account_id);
$layout = new salesforce_TableLayout($account);

// This should show and HTML form similar to the one on salesforce
//  with the fields containing values from $account already
echo $layout->getLayoutDisplay('editLayoutSections');
{% endhighlight %}

Finally, save works as well, err.. a good part of the time  :P



{% highlight php %}
<?php
$account = new mySalesforceAccount();
$account->getById($account_id);

// pretending we just received a post from an edit layout form
$account->fieldsFromArray($_POST);

try {
  $account->save();
  echo 'Yay!!  We Saved it!!'
} catch (Exception $e) {
  echo 'ERROR : '.$e->getMessage();
}
{% endhighlight %}

Oh well, it's out on github [https://github.com/mgfreshour/salesforce.php](https://github.com/mgfreshour/salesforce.php) now if you're interested.  I'd like to work some more on it and perhaps even get it production ready, but that'd require doing programming at home as well as at work! *yuck*