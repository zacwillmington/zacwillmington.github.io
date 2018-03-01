---
layout: post
title:      "Intro To OOP: Classes in Ruby"
date:       2018-03-01 14:34:44 +0000
permalink:  intro_to_oop_classes_in_ruby
---


OOP is the abbreviation for object oriented programming. OOP is a topic I have been recently exposed to. Since diving into Ruby I have been intrigued by its class structure, workability and its ease to produce D.R.Y programming principles. 

My aim for this blog is to share my thoughts on Ruby classes and perhaps help those newbies out there by giving an explanation from a beginner’s point of view. I am still early in the learning process with Ruby, so any constructive criticism is welcome. 

Let’s start this off by defining what a class is. I like to think of a class as a structure or functionality for an object. Objects are to be defined using a class. You can initialize an object by using the class_name.new. When doing so, the object that has been created will have the characteristics of the class it is created by. If that doesn't make sense I'll clear it up with some examples. 

```
class Person 
	

end 
```


This is the bare-bones structure of a class. The class name is “Person.” The class name must always be capitalized, don’t ask me why, it just does.
Now that we have our class defined, let’s make it do something.

```
 class Person
	def talk
		puts ”?ay era woH”
	end
 End
 ```
In the above example I defined a method within our class called #talk (the “#” in front of a word means it is a method.) The method outputs the string I created when we run #talk on our object.
 
This is how we defined an object.

```zac = Person.new```


This new object is tied to the “Person” class there for it can use whatever method the “Person” class contains. Lets see if Zac can talk.

```
class Person
	def talk
		puts ”?ay era woH”
	end
 end

zac = Person.new   #Initializes a new instance object “zac”.
zac.talk                  #Calls or runs the method talk on the “zac” object.

output => “?ay era woH”
```


Oh no! The person class gives the new person created a rare condition that makes them talk backwards. Let’s see if we can fix it.


```
class Person
	def talk
		"?ay era woH"
	end
 end


zac.talk.reverse       #Calls the talk method which returns the backward                   
                                      #then reverse method is called switching the string.
														
Output => “How are ya?”
```

There we go. The cool thing about classes is that you can create objects that inherit the functionality of the class, talking on its methods. This applies to all objects that are created when using a class. Let’s create another person to see.

```
lauren = Person.new

lauren.talk

Output =>  "?ay era woH”
```


See, “lauren” has the same condition as “zac.” Luckily we know how to fix it using the #reverse method which is a part of the Ruby language. #reverse is a handy method that you will be able to use when programming in Ruby.
Have a go at recreating this same class in a new repl.it and see if you can make your person shout. Google #upcase Ruby for some info. 

This is the basics of a class in ruby. There is a lot more you can do with classes in Ruby that I will go over in another post.

Until then, keep coding!


