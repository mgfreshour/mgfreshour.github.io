---
layout: post
title: Bit Field in PHP
date: '2011-11-08T08:13:00.000-08:00'
author: Morgan Freshour
tags: 
modified_time: '2011-11-08T08:13:36.512-08:00'
blogger_id: tag:blogger.com,1999:blog-3127719415764963943.post-2247495884822944242
blogger_orig_url: http://hillbilly-nerd.blogspot.com/2011/11/bit-field-in-php.html
---

So, a coworker ran across a piece of legacy code and wasn't sure what it was doing.



{% highlight php %}
<?php
foreach($external_reports as $report){
  if ($show_reports & parseInt($report['type'])){
 display_report($report);
  }
}
{% endhighlight %}


His first guess was a typo in a logical AND (&&). I completely understand his confusion, binary operators are not commonly used in PHP programs. Being an old C programmer, when I saw this I knew immediately it was a bit field. Checking the database table that _$external_reports_ was pull from confirmed this :

<pre>+----------+------+
| Name     | Type |
+----------+------+
| Report A |    1 |  
| Report B |    2 |
        ...
| Repord H |  256 |
| Report I |  512 |
+----------+------+
</pre>

So, what is a bit field? Wikipedia says [Bit Field](http://en.wikipedia.org/wiki/Bit_field) is a "common idiom used in computer programming to compactly store multiple logical values as a short series of bits where each of the single bits can be addressed separately." These are extremely important when the memory usage is vital (and probably other scenarios that I can't think of while I write this).  Infact, C/C++ has the idea of bit fields built into the [struct type](http://publications.gbdirect.co.uk/c_book/chapter6/bitfields.html).



Okay, so the important part, how they work.  First let's look at that table again with an extra piece of information, the binary representation of the numbers :

<pre>+----------+------+
| Name     | Type |
+----------+------+------------+
| Report A |    1 | 0000000001 |
| Report B |    2 | 0000000010 |
        ...
| Repord H |  256 | 0100000000 |
| Report I |  512 | 1000000000 |
+----------+------+------------+
</pre>

So, if _$show_reports_ = 512 (binary of 1000000011).  So the code loops through each of the report types AND'ing it to the value.  So the results are :

<pre>Report A                     Report B
  0000000001                   0000000010
& 1000000011                 & 1000000011
============                 ============
  0000000001 (TRUE)            0000000010 (TRUE)



Repord H                     Report I
  0100000000                   1000000000
& 1000000011                 & 1000000011
============                 ============
  0000000000 (FALSE)           1000000000 (TRUE)
</pre>  

So, _display_report()_ will be displayed for A,B, & I.



So, how does the value change?  Well to add a new report one just OR's it:

{% highlight php %}
<?php
$value = $value | $new_type;
{% endhighlight %}



To remove a value, it's AND'ed on the (ones' complement)[http://en.wikipedia.org/wiki/One's_complement] of the value:

{% highlight php %}
<?php
$value = $value & ~$new_value;
{% endhighlight %}


Okay, so reading back over this, it doesn't seem a good blog post.  Oh well :P
