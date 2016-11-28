---
layout: post
title:  "where vs select impact in Rails on memory & performance"
date:   2016-11-28 11:34:33 +0530
categories: blog
---
I was writing a seed file where in it was needed to update details pertaining to service_id in a table. My current task needed to look for 10000+ services and update the records.

In a traditional setup, I would had stored all the services in a variable and worked on each record *(I know it is bad but wanted to see how bad it can get)*.

{% highlight ruby %}
services_all = Service.all
services_all.each do |record|
 #Do something on record
end
{% endhighlight %}

To understand the impact of this I wanted to see two kinds of impact on system & resources as:
1. Memory allocated to the variable
2. Time take by SQL to execute the query

There is an excellent module in Ruby to check the memory used by a variable called as Object Space.
To use it inside Rails Console, do this :

{% highlight ruby %}
require 'objspace'
{% endhighlight %}

This module includes a lot of awesome things to actually study the internals of an object but lets focus on our current considered issue. The **memsize_of** method helps to get the size of an object in bytes in following fashion.

{% highlight ruby %}
ObjectSpace.memsize_of(services_all)
#=> 120632
{% endhighlight %}

So I decided to improvise and just select the *id* of the services using *select* method in ActiveRecord

{% highlight ruby %}
services_all_id = Service.select(:id).all
{% endhighlight %}

So curiously I had anticipated that this should be a lot smaller than the earlier approach. To my disappointment it was not :(

{% highlight ruby %}
ObjectSpace.memsize_of(services_all_id)
#=> 120632
{% endhighlight %}

Suprisingly the size of this object was exactly the same. So ActiveRecord treats both of the queries as same instantitates all of the objects & attributes immediately.

Now, I thought that to optimise this issue lets try to use the *where* method in ActiveRecord.
I was aware of the number of records I needed to work with so tweaked the query accordingly.

{% highlight ruby %}
services_all_where = Service.where(:id => 1..11000)
{% endhighlight %}

And now went on to the explorative path to get astonished.
{% highlight ruby %}
ObjectSpace.memsize_of(services_all_where)
#=> 264
{% endhighlight %}

As per the behaviour of ActiveRecord, *where* returns an ActiveRecord Relation having characterstic lazy loading. In simple terms the actual query is not performed till user asks for the object and it is in an uninstantiated way in the beginning saving us the precious memory.

Now to study the SQL impact of these same ways to get the data, I went ahead to use the **Benchmark** module that ruby provides.
Benchmark module has the *measure* method which does all the time measuring stuff for us

{% highlight ruby %}
Benchmark.measure { "a"*1_000_000}
#=> 1.166667 0.050000 1.216667 ( 0571355 )
{% endhighlight %}

These numbers displayed as an output are actually time taken by code to execute in seconds in four different categories viz. user / system / total / real. The actual meaning of these times are as follows :
1. user : Total user CPU time
2. system : System CPU time
3. total : Sum of User & System CPU times
4. real : Elapsed real time required to process this benchmarking activity

{% highlight ruby %}
Benchmark.measure { "a"*1_000_000}
       user        system    total (user + system)      real
#=>   1.166667    0.050000          1.216667        ( 0.571355 )
{% endhighlight %}

Now on to the analysis which should be inline with our earlier observation realted to memory usage.

{% highlight ruby %}
Benchmark.measure { Service.all }
Service Load (206.2ms) SELECT `services`.* FROM `services`
#=> 0.490000  0.040000  0.530000  ( 0.552428 )

Benchmark.measure { Service.select(:id).all }
Service Load (5.7ms) SELECT id FROM `services`
#=> 0.060000  0.000000  0.060000  ( 0.063805 )

Benchmark.measure { Service.where(:id => 1..11000) }
#=> 0.000000  0.000000  0.000000  ( 0.000088 )
{% endhighlight %}

Yes it is, We can see the query load lowering dramatically and eventually during the *where* query ActiveRecord did not made any query which clears our perspective about lazy loading in a practical way.

So in the end `objspace` & `Benchmark` modules are really a bliss to study the impact on memory and system loads.