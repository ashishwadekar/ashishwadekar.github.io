---
layout: post
title:  "Freedom to seed any custom seed file!"
date:   2016-12-02 19:45:32 +0530
categories: blog
comments: true
---

So while developing for web using Rails framework. We many a times come across the common task of populating the database with some predefined value or making changes to a lot of records in an existing database using a seed file.

Rails has an already tried and tested path to do this stuff. We have to write the seeding instructions in a file and save it in the `db` folder named as `seeds.rb`. Now just issue the command **rake db:seed** and Voila! your data is seeded in the database.

Well this seems to be a dream come true, but this quickly turns into a nightmare if your workflow demands generation of multiple seed files and storing them with a naming convention for task purposes. You have to go through the dreaded path of renaming the existing `seeds.rb` to something like `seeds.1.rb`, then save your current seed file as `seeds.rb` and run the seed command and again repeat this whole thing. Phew!

I faced this similar issue yesterday at work and decided to solve this problem once and for all. I wrote a custom rake task just to seed any file. **Yes it is possible.** 

Let's have a look at it :<br/>
Filename: **seeder_power.rake**<br/>
Location: **lib/tasks/**

{% highlight ruby %}
#lib/tasks/seeder_power.rake
namespace :db do
	namespace :seed do
		Dir[File.joinf(Rails.root, 'db', '*.rb')].each do |filename|
			task_name = File.basename(filename, '.rb').intern
			task task_name => :environment do
				load(filename) if File.exist?(filename)
			end
		end
	end
end
{% endhighlight %}

To use this superpower, we now have to use the command **rake db:seed:filename**<br/>
For e.g. if your new seed file name is *user_likes_chocolate.rb* then now to seed this file use the command **rake db:seed:user_likes_chocolate**

Basically the above rake task looks for files inside the `db` directory having an extension `.rb`. Then it [symlinks](https://en.wikipedia.org/wiki/Symbolic_link)  to them. The file name to be searched for is added to the task after the namespace. The file is loaded if it exists.

*Thats it. You can now seed any file without compromising on filename nor the structure!*
