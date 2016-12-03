---
layout: post
title:  "Find where the method lies : source_location"
date:   2016-12-03 20:10:26 +0530
categories: blog
comments: true
---

While checking out some Rails magic happennig or even while rollercoasting into a very large codebase, it would be really super duper helpful if there could be something that would help to find where the method was defined. *(Ofcourse, there are these cool text editors that help to find this but they generally dont't reach inside gems)*

`source_location` is the method that comes to our rescue!

**Usage:**<br/>
We have a take the object which we are curious about and then chain it to `method` method with the method we are searching for and chain this to `source_location`.<br/> Woww this looks so confusing but be rest assured it is not. Let's go. <br/> 

{% highlight ruby %}
#We are trying to find location of lights_on method in home model
#Let's create a new Home object.
my_home = Home.new

#Now the not so tough thing to look for location in source
my_home.method(:lights_on).source_location
#=> ["/Users/ashish/Development/rails_projects/home_automation/app/models/home.rb", 1409]

my.home.method(:create).source_location
#=> ["/Users/ashish/.rbenv/versions/1.9.3-p551/lib/ruby/gems/1.9.1/gems/activerecord-3.1.12/lib/active_record/callbacks.rb", 267]
{% endhighlight %}

So this returns an Array of results :<br/>
The first element shows the path of the file where the method is stored <br/>
The second element shows the line number where the method is define <br/>

This can be nifty tool to to debug a lot of stuff, learn new things about implementation of code and really really helpful to have a in the Metaprogrammed methods.<br/>

To learn more about more nifty such methods you can visit Ruby official methods here [here](https://ruby-doc.org/core-1.9.3/Method.html).<br/>

Thanks to [Prathamesh Sonpatki](https://twitter.com/_cha1tanya) for bringing this my knowledge.


