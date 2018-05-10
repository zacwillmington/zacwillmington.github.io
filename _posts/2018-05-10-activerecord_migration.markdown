---
layout: post
title:      "ActiveRecord Migration"
date:       2018-05-10 15:34:04 +0000
permalink:  activerecord_migration
---



![](http://res.cloudinary.com/zacwillmington/image/upload/v1525028357/federico-consales-567604-unsplash_mj4sr4.jpg)
## Introduction

ActiveRecord makes it effortlessly easy to map object relationships a convention called ORM(object relational mapping). With object relational mapping we can create and save objects to the database while having the option to assign foreign keys to related objects with ease. ActiveRecord puts another level of abstraction to we can easily use its methods to execute SQL queries. In this blog post I am going to demonstrate how to add or edit databases with ActiveRecords migration features.

Inheritance
In order to use ActiveRecord you have to inherit from the ActiveRecord::Base to all your classes, and require gem 'activerecord' in your gem file below on line 6.

```
source "https://rubygems.org"


gem 'sinatra'
gem 'activerecord', :require => 'active_record'
gem 'sinatra-activerecord', :require => 'sinatra/activerecord'
gem 'rake'
gem 'require_all'
gem 'sqlite3'
gem 'thin'
gem 'shotgun'
gem 'pry'

group :test do
  gem 'rspec'
  gem 'capybara'
  gem 'rack-test'
  gem 'database_cleaner', git: 'https://github.com/bmabey/database_cleaner.git'
end
```

```
class Movie < ActiveRecord::Base
  
  
end
```

## Schema

A Schema is a structure of a database. Directly below shows the current tables and their schemas, this updates when any new migrations are added changing the structure of the schemas.

Under the schema file I have a screen shot of the database in a sql viewing tool called DB Browser for SQLite. Note that there are many different database languages, but I'm using SQLite for its easy use, and we aren't doing too much with it.

```
ActiveRecord::Schema.define(version: 20180426132809) do

  create_table "movies", force: :cascade do |t|
    t.string  "title"
    t.integer "release_date"
    t.string  "director"
    t.string  "lead"
    t.boolean "in_theaters"
  end

end
```

![](http://res.cloudinary.com/zacwillmington/image/upload/v1525018357/Screen_Shot_2018-04-29_at_9.11.29_AM_mj2pqz.png)

## Migration

At this point I have everything setup and ready to find some movies but, I decided I want to add a productions table to the application so I can find out which movie belongs to which production company.

In order to do this I need migrate a new class. With ActiveRecord you need to create a new class to make any changes to the database. I'll create a new file called 002_production_table.rb and that's where the migration happens. I named it 002 so active record knows that it was the second migration.

Usually the file name is automatically named based on a timestamp that ActiveRecord applies, you can run this command in the terminal $ bin/rails generate migration ProductionTable to auto generate a file/class. Much like the one shown below.

The purpose of time-stamping migrations is so that ActiveRecord knows what order the migrations happened. It looks like 20180426132809_create_movies.rb  2018-04-26-13-28-09 is in YYYY-MM-DD-HH-MM-SS format.

```
class ProductionTable < ActiveRecord::Migration

    def change
        create_table :productions do |t|
            t.string :name
        end
    end
end
```

Now that I've created a migration file and created a table I need to make the class where my methods with be run. Just like the Movie class I have to make another file for all my logic for the application. It too inherits from ActiveRecord::Base giving the newly created class a bunch of methods to help us map our database.

```
class Production < ActiveRecord::Base

end
```

One last step and then we have our new table. I must run rake db:migrate in the terminal to for the changes to the table to update, or you'll see a error message like this.

```
/home/zacwillmington/activerecord-crud-v-000/spec/spec_helper.rb:8:in `<top (required)>': Migrations are pending. Run `rake db:migrate SINATRA_ENV=test` to resolve the issue.
(RuntimeError)
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/lib/rspec/core/configuration.rb:1295:in `require'
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/lib/rspec/core/configuration.rb:1295:in `block in requires='
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/lib/rspec/core/configuration.rb:1295:in `each'
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/lib/rspec/core/configuration.rb:1295:in `requires='
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/lib/rspec/core/configuration_options.rb:109:in `block in process_options_into'
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/lib/rspec/core/configuration_options.rb:108:in `each'
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/lib/rspec/core/configuration_options.rb:108:in `process_options_into'
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/lib/rspec/core/configuration_options.rb:21:in `configure'
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/lib/rspec/core/runner.rb:105:in `setup'
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/lib/rspec/core/runner.rb:92:in `run'
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/lib/rspec/core/runner.rb:78:in `run'
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/lib/rspec/core/runner.rb:45:in `invoke'
        from /usr/local/rvm/gems/ruby-2.3.1/gems/rspec-core-3.4.4/exe/rspec:4:in `<top (required)>'
        from /usr/local/rvm/gems/ruby-2.3.1/bin/rspec:23:in `load'
				```
				
Easy, and let's have a look a the schema to see how how its changed.

```
ActiveRecord::Schema.define(version: 20180429110601) do

  create_table "movies", force: :cascade do |t|
    t.string  "title"
    t.integer "release_date"
    t.string  "director"
    t.string  "lead"
    t.boolean "in_theaters"
    t.integer "production_id"
  end

  create_table "productions", force: :cascade do |t|
    t.string "name"
  end

end
```

And there we go, an new foreign key at the bottom of the movies table and a new productions table. Next post I'll show you how to work with these two tables and how to create new productions and associate the objects with each other, check it out here >> [](https://www.zacwillmington.com/activerecord-associations/).
