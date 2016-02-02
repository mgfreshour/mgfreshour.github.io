---
layout: post
title: Why JOIN ... USING clauses can be a bad habit
date: '2011-07-28T15:26:00.000-07:00'
author: Morgan Freshour
tags: 
modified_time: '2011-07-28T15:26:37.184-07:00'
blogger_id: tag:blogger.com,1999:blog-3127719415764963943.post-4889738059572549484
blogger_orig_url: http://hillbilly-nerd.blogspot.com/2011/07/why-join-using-clauses-can-be-bad-habit.html
---

So first, let me say that I do like the JOIN ... USING clauses.  They make straight forward joins look a lot better.  The problem is if you get into a habit of using them and doing further join filtering in the WHERE clause.  Most of the time, this is perfectly okay because the relational calculus handles things nicely.  But what if your SQL becomes more complicated based on certain options?  Let's look at an example. _(This story is based on a subtle defect I had to work on today.)_

Given two tables :

<table border="1">  <tr><th colspan="2">users</th></tr>
  <tr><td>user_id</td><td>S /ERIAL</td></tr>
  <tr><td>some_flag</td><td>BOOLEAN</td></tr>
</table>

<table border="1">  <tr><th colspan="2">user_attributes</th></tr>
  <tr><td>user_attribute_id</td><td>SERIAL</td></tr>
  <tr><td>user_id</td><td>FORIEGN KEY</td></tr>
  <tr><td>name</td><td>VARCHAR</td></tr>
  <tr><td>value</td><td>VARCHAR</td></tr>
</table>

Such that users have 0..n user_attributes (don't complain about this schema design, it's just for example).



You need a report that shows all users with a certain attribute, easy :

{% highlight javascript %}
sql = "SELECT user_id, value
       FROM users
         JOIN user_attributes USING(user_id)
       WHERE name='CLOWN_ATTRIBUTE'";
{% endhighlight %}



Bam! Done.  Oh wait, the product owner wants an option to display all users that don't have that attribute set.  Still easy :



{% highlight javascript %}
sql = "SELECT user_id, value
       FROM users
         LEFT JOIN user_attributes USING(user_id)
       WHERE";

if (display_without) {
  sql += " user_attribute_id IS NULL";
} else {
  sql += "  name='CLOWN_ATTRIBUTE'"
}
{% endhighlight %}



...and done again!  You're a master programmer.  Of course, product owners are never happy.  Now they want another option to display all users and that have that attribute or have some_flag set.  That's easy too :



{% highlight javascript %}
sql = "SELECT user_id, value
       FROM users
         LEFT JOIN user_attribute USING(user_id)
       WHERE";

if (show_all_with_flag) {
  sql += " (name='CLOWN_ATTRIBUTE' OR some_flag = TRUE)";
} else if (display_without) {
  sql += " user_attribute_id IS NULL";
} else {
  sql += "  name='CLOWN_ATTRIBUTE'";
}
{% endhighlight %}



And done!  I'm going home.

...

...

...

What do you mean the report isn't working?



So, because of "(name='CLOWN_ATTRIBUTE' OR some_flag = TRUE)" things aren't quite working right.  The OR condition allows it to connect to _any_ row in the user_attributes table that belongs to a user with some_flag set.  While one can probably use some fancy SQL logic to make this still work with the JOIN ... USING clause, I find it much easier to :



{% highlight javascript %}
sql = "SELECT u.user_id, ua.value
       FROM users u
         LEFT JOIN user_attribute ua 
           ON u.user_id = ua.user_id AND name='CLOWN_ATTRIBUTE'
       WHERE";

if (show_all_with_flag) {
  sql += " (ua.user_attribute_id IS NOT NULL OR u.some_flag = TRUE)";
} else if (display_without) {
  sql += " ua.user_attribute_id IS NULL";
}
{% endhighlight %}



Oh well, I know to look out for one more thing now.