---
layout: post
title: Expiring Rails3 login sessions the cheap way
date: '2011-08-21T06:50:00.000-07:00'
author: Morgan Freshour
tags:
- ruby rails3
modified_time: '2012-01-23T13:27:32.739-08:00'
blogger_id: tag:blogger.com,1999:blog-3127719415764963943.post-3712589054278835291
blogger_orig_url: http://hillbilly-nerd.blogspot.com/2011/08/expiring-rails3-login-sessions-cheap.html
---

Okay, so for my game [Hexwar](https://github.com/mgfreshour/hexwar) I needed some way to expire the login stored in the session.  So, my first stop was google and the top answers [Ruby On Rails Security Guide](http://guides.rubyonrails.org/security.html#session-expiry), [Sessions and cookies in Ruby on Rails](http://www.quarkruby.com/2007/10/21/sessions-and-cookies-in-ruby-on-rails#sexpire), and [Rails Session Timeout](http://www.ruby-forum.com/topic/87136) all seemed a bit more involved than I wanted this early in the morning.  So, I'll just roll my own!


First, to understand what I need, one must understand what I have.  I'm using the classic pattern of a before_filter in the application controller that routes a person to the sessions controller to force a login.

{% highlight ruby %}
class ApplicationController < ActionController::Base
  before_filter :check_authentication, :except => [:check_authentication]

  private   
  def check_authentication
    @current_player ||= user.find(session[:player_id]) if session[:player_id] 

    redirect_to '/session/login' unless @current_player
  end
end

class SessionsController < ApplicationController
  def create
    # Check credentials

    session[:player_id] = player.id

    redirect_to root_url
  end
end
{% endhighlight %}


Now, I wanted a login to last one day.  I may change this in the future to expire if there's no activity for a period, but for now the hard one day limit works for me.  So, I just add an expiration time to the session and unset it when past.

{% highlight ruby %}
class ApplicationController < ActionController::Base
  before_filter :check_authentication, :except => [:check_authentication]

  private   
  def check_authentication
    if session[:expires].nil? || session[:expires] < Time.now
      session[:player_id] = nil
    end

    @current_player ||= user.find(session[:player_id]) if session[:player_id] 

    redirect_to '/session/login' unless @current_player
  end
end

class SessionsController < ApplicationController
  def create
    # Check credentials

    session[:player_id] = player.id
    session[:expires] = Time.now + 1.day

    redirect_to root_url
  end
end
{% endhighlight %}


I realize this code is in no way impressive and there's probably a more Ruby-centric way to do this, but I haven't written for my blog in a while and this is what I just finished.

