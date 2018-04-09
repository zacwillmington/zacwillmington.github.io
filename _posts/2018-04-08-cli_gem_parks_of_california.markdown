---
layout: post
title:      "CLI Gem Parks of California"
date:       2018-04-08 22:27:08 -0400
permalink:  cli_gem_parks_of_california
---



After learning a great deal about Ruby over the last month I’m at the point of creating my first project. I decided to create a Ruby CLI gem that uses Nokogiri to scrape a national park service website for information on all the national parks and monuments throughout California.
Here’s my process of how I went about it.


## Planning

I started off planing the structure of my CLI, I didn't want to go all in and write a bunch of code that would later be useless. I started by requiring Bundler gem and using that to create the boiler plate file structure for my gem. Bundler does a great job of creating the exact file structure and the code that enables the files needed to be able to collaborate. The small change I had to make was adding the `#!/usr/bin/env ruby` to bin/Parks-cal file. This lets it know that it is a ruby program.

After some thinking I then went with three classes for my project: CLI, Place and Scrape.
 

## CLI Class

I began by making the `Parks::CLI` class. This will be my controller for the gem. A controller’s job is taking the users input and then output the information. I knew I wanted to start with the interface because I heard that in this situation a good rule of thumb is to get the interface running with fake data, that way once I’ve got that working I can scrape the data I need and voilà… hoping it all works. 

In the first method of my app is #call. Its job is to run all the methods in the correct order to from start to finish.



```
  def call
        puts "Loading data..."
        parks = ParksCal::Scrape.new
        parks.scrape_names
        parks.scrape_data
        opening_greeting
        parks_selector
        leave_msg
    end
```



I’ll come back to talk you through the rest of the CLI class methods after I’ve explained the Scrape and Place classes, it will help give some clarity to the controller’s logic.



## Scrape Class

Scrape class was the trickiest park of the gem. Initially I’m only scraping one website to gather all of the park names and urls, that’s why I went with only one ParksCal::Scrape instance object for my gem, for the reason that the @parks_site_url never changes.   



```

    def initialize
        @parks_site_url = "https://www.nps.gov/state/ca/list.htm?program=parks"
    end

    def scrape_names
        html = open(@parks_site_url)
        doc = Nokogiri::HTML(html)
        park_names = doc.search("li.clearfix")
        index = 1
        parks = []
        park_names.each do |name|
            park_name = name.css("h3 a").text
            info_url = name.at_css('li a:contains(" Basic Information")')

            if park_name != "" && info_url
                parks << "#{index},#{park_name},#{info_url["href"]}"
                index += 1
            end
        end
        parks
    end
```


As soon as this instance is created in the ParksCal::CLI call method, the scraping process begins.  Scrape_names gathers each park name and the corresponding url that leads to more information on each park. The output looks like something along these lines.

 parks => [ “1, park_name_1, www.more_info_url_1.com”,
        “2, park_name_2, www.more_info_url_2.com”
       ]
	
Here we have a nice little array to work with in the scrape_data method. In the scrape_data method
I iterate through the scrape_names array and at each park I parse the www.more_info_url_1.com and scrape it for address, directions and opening hours. 

Before I create the new Place instance with this information I verify the values of the newly scraped data to make sure they are not empty strings. As you can see below I achieved this with the use of if statements.


```
def scrape_data
        self.scrape_names.each do |p|
            park_array = p.split(",")
            park_info_url = Nokogiri::HTML(open(park_array[2]))
            address = park_info_url.at("//div[@itemprop = 'address']").children.text.squeeze
            directions = park_info_url.search("div.directions span").text
            opening_hrs = park_info_url.search("div.HoursSection ul").text

            if directions == ""
             directions = "Directions not available."
            end
            if opening_hrs == ""
                opening_hrs = "Opening hours not available."
            end
            directions = directions.gsub("Directions Details", "")
            ParksCal::Place.new(park_array[0], park_array[1], address, directions, opening_hrs)
        end
    end
```


## Place Class

At this point in my program I’ve scraped the two urls and have the data required to make each park. Each Place instance is an object with properties: `index, :name, :address, :directions, :opening_hrs` This makes it incredibly easy to use and display each object in my controller.


```
class ParksCal::Place

    attr_accessor :index, :name, :address, :directions, :opening_hrs

    @@all = []

    def initialize(index, name, address, directions, opening_hrs)
        @index = index
        @name = name
        @address = address
        @directions = directions
        @opening_hrs = opening_hrs
        @@all << self
    end

     def self.all
         @@all
     end
end
```


All in all, it’s a very simple class, the glue that holds it all together. After the Scrape class does its thing I’ll have 34 instances in the .all method each containing index, name, address, directions and opening_hrs attributes, all ready to be worked with in the CLI class.


Let’s return to the CLI class. All has been scraped and is waiting for us to dig in to the data. The opening_greeting method runs letting us know what’s about to go down. 


```

    def opening_greeting
        puts "Welcome!!!"
        sleep 1
        puts "There are many beautiful places to visit around California."
        sleep 1
        puts "Here is a list of popular national parks and monuments of California."
        sleep 3
    end
```


Then parks_selector next inline is initiated and within the while loop running executes list_parks:


```
def parks_selector
        input = nil
        while input != 'exit'
            list_parks
            input = gets.strip
            int = input.to_i
            if int.between?(1,34)
               more_info(@places[int - 1])
           elsif !int.between?(1,34) && input != "exit"
               puts "Please select a number between 1 and 34 or exit."
            end
       end
    end


    def list_parks
        puts "Select a number for more info or exit.\n\n Where would you like to go?"
        @places = ParksCal::Place.all
        @places.each do |place|
            puts "#{place.index}. #{place.name}"
        end

    end

```


I made the decision to create @places because I knew I would need all the parks a some point in another method. The output looks something like this:



```
1. Alcatraz Island
2. Cabrillo
3. California
4. Castle Mountains
5. César E. Chávez
6. Channel Islands
7. Death Valley
8. Devils Postpile
9. Eugene O'Neill
10. Fort Point
11. Golden Gate

```


```

    def parks_selector
        input = nil
        while input != 'exit'
            list_parks
            input = gets.strip
            int = input.to_i

            if int.between?(1,34)
               more_info(@places[int - 1])
           elsif !int.between?(1,34) && input != "exit"
               puts "Please select a number between 1 and 34        		or exit."
            end
       end
    end

```


Once the input is true, a number between 1 and 34 the more_info method is called using the int number as my selector for the element in the array.


```
def more_info(place)
        puts "#{place.name} sounds great!"
        puts "Would you like more info? enter the appropriate 	 number or exit."

        puts "1. Address"
        puts "2. Opening hours"
        puts "3. Directions"
        input = nil

        while input != 'exit'
            input = gets.strip
            int = input.to_i
            if int.between?(1,3)
              if int == 1
                    puts "#{place.address}"
                elsif int == 2
                    puts "#{place.opening_hrs}"
                elsif int == 3
                    puts "#{place.directions}"
                end
            else
                puts "Enter the appropriate number or exit."
            end
        end
    end
```


At this point we have drilled down to the lowest part of the CLI gem giving the user the final three options for the chosen park. It’s pretty simple, it uses the same logic as the method above, giving us the object’s instance variable value. 


I really enjoyed creating this gem, it’s my first object oriented program I’ve build in the short time spent programming in Ruby. I’m super excited to build many more in the future. You can check out the repo on GitHub here. 



 

