---
layout: post
title:  "Rake Task : Arguments for advanced usage"
date:   2016-12-11 12:24:48 +0530
categories: blog
comments: true
---

Rake tasks are really helpful. They come in for our help when need to perform some automated stuff and sprinkle the Ruby magic to create that big impact with least of the efforts.

I encountered a similar issue where a rake task was required to remove some records from specific tables and update status of some fields based on identification e.g. company_id, region_name. It was easy enough to write a rake task with certain specifics, target certain tables. Run it and forget it.

Now with Ruby's commending DRY - *Don't repeat yourself* mantra, I thought that maybe this rake task can be used for again for some other company or store in future. The train of thought started chugging.<br/>
What if ? :
- we could send in values as arguments to a rake task
- we could use those parameters to perform the magic in our task
<br/>

Rake tasks allow us to do exactly that. Let's see how we can do it: <br/>
{% highlight ruby %}
#We will be sending company_id and region_name as parameters to our rake task
#Let's create the interactive Rake task

namespace  :convert do
  desc 'This rake task removes products in a company & stores in area which are defunct getting arguments as [company_id, region_name]'
  task :defunct, [:company_id, :region_name] => [:environment] do |t, args|
    company_id = args[:company_id]
    region_name = args[:region_name]

    begin
      ActiveRecord::Base.transaction do
        #Deleting defunct products for a company
        Product.where(:company_id => company_id).delete_all

        #Deleting defunct stores from inactive regions
        Store.where(:company_id => company_id, :region => region_name).delete_all
        puts "The task was successfully completed."
      end
    rescue Exception => e
      puts "There was some error while deleting the products or store. Here are the details : #{e}"
    end
  end
end
{% endhighlight %}

Let's see this in action:<br/>
On command line use the following command to execute our super rake task:<br/>
 `rake convert:defunct[2987, "Baner"]` <br/>
Some terminal emulators can give trouble to execute this rake task so if your encounter an issue issue the command as <br/> `rake 'convert:defunct[2987, "Baner"]'`

Now we will see how this actually works : <br/>
Two important parts of the whole code are highlighted and explained below: <br/><br/>
{% highlight ruby %}
task :defunct, [:company_id, :region_name] => [:environment] do |t, args|
{% endhighlight %}
Here we are asking rake to store the our parameters in an internal hash with keys as **:company_id** & **:region_name** and values are the ones that we input. <br/><br/>
{% highlight ruby %}
company_id = args[:company_id]
region_name = args[:region_name]
{% endhighlight %}

Here we access the values from the rake hash and assign them to variables which can be used further.

Now let your imagination fly and use this advanced rake method to achieve a greater magic!
