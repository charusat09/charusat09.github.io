---
layout: post
title:  Update UI using Sidekiq Worker response
date:   2016-11-13 16:00:00
description: Transfer time taking servise to worker and use it's response on UI
---
#### Background:
<br/>
I was working on project where I had to update user account on third party server. This process was taking long time(i.e. 3-5 secs). Hence we decided to pass this service on Sidekiq worker and proceed ahead in request flow. Meantime UI had loader that informed user to wait for some time. As soon as we got response from third party server, we have to inform UI. I tried server solutions and then I came to use of Redis and Synchronous Sidekiq Worker to rescue me.

#### Solution:
<br/>
Sidekiq Workers and UI are not related. Workers are asynchronous. They are running somewhere and UI does not know where. So, basically we need to store the response somewhere and pull out that response back to UI. I used Redis to store sidekiq worker response and pulling out that for UI. There may be other work around.

#### Redis and Synchronous Sidekiq Worker:
<br/>

{% highlight javascript %}

$(".btn").on("click",function(){
  $(".btn").text("counting...");
  $.ajax({
    url:"<%= counter_index_path %>",
    type:"POST"
  });
  myResponse();
});

{% endhighlight %}

This button does ajax call to create method of counter controller. The job for create method is to call HardWorker several time synchronously. The real problem is, we want to inform UI about the response of our hard working guy.

{% highlight ruby %}

def create
  @count = HardWorker.new.perform
end

class HardWorker
  include Sidekiq::Worker
  def perform
    5.times do |count|
      sleep(1)
      Redis.new.set("counter", count)
    end
  end
end

{% endhighlight %}


Here I have hard worker which does really hard work!! It does some task, in our case it sleeps for a second. After that it store counter value to Redis store and that we need to pull out.

{% highlight javascript %}

myResponse = function(){
  var count = 0;
  var counting = setInterval(function(){
    $.ajax({
      url:"<%= counter_path(1)%>",
      type:"GET"
      })
      .done(function(data){
        count++;
        if (count > 10){
          $(".btn").text("I am done...")
          clearInterval(counting);
        }
        $('span').html(data);
    });
  },1000)
};

{% endhighlight %}

I use ajax call to my counter controller show method with a second span. Here the job for show method is pulling out data from Redis and inform UI about it.

{% highlight ruby %}

def show
  counter = Redis.new.get("counter")
  render json: counter
end

{% endhighlight %}

The data coming from show method response is populating over the page by jQuery. This is very generic solution. There are several methods for storing and pulling out data.You can store values to some file or mongo db and read it back. You can this code modify as per your requirements and plug in to your application.

#### Caveat:
<br/>

This solution has one limitation.
<ul>
  <li>
    UI has to know the `key` name that is used to store sidekiq response in Redis
  </li>
</ul>

You can find code here <a href="https://github.com/charusat09/sync_sidekiq">on GitHub</a>
