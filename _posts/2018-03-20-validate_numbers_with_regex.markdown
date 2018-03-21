---
layout: post
title:      "Validate Numbers With Regex"
date:       2018-03-20 20:52:21 -0400
permalink:  validate_numbers_with_regex
---



I came across a challenge recently where I had to check if a given cell phone number was valid. I thought that this is a pretty common problem in the web development world, especially when it comes to requiring information from users, so I figured I’d give you a quick run down of Regular Expressions using Ruby to validate a number.

First I’ll show you the solution, then a run down of the different symbols and what they mean.


 ```
 def valid_number?(number)
       	number.match(/([0-9] *?){10}|(\([0-9]{3}\)(([0-9]{3}-[0-9]{4})|[0-9]{7})\b)/) ? true : false

     end
```

First off I decided to utilize the #match method to match the string to the regex in the brackets and return either true or false. Inside the first parenthesis in blue.


`/([0-9].*?){10}|(\([0-9]{3}\)(([0-9]{3}-[0-9]{4})|[0-9]{7})\b)/`

When you see brackets used in this way the Regex is reading any collection of numbers that are non-negative. `“.*?”` is a search/match condition known as non-greedy, Regex is by default greedy. Greedy basically means that the Regex is going to search through the number that is input and match numbers based on a condition, which is any collection of non-negative numbers from the beginning of the input to “{10}” numbers back. So the greedy part is the search process the greedy search will find every block of 10 numbers whereas the non-greedy will search the whole block of numbers and then work back until there is only 10 numbers.

 Here’s an example.


 ![](http://res.cloudinary.com/zacwillmington/image/upload/v1521594307/Screen_Shot_2018-03-20_at_6.01.40_PM_pqd9dq.png)

Those greedy buggers. So, now what’s going on with all those paren and numbers?


`/([0-9].*?){10}|(\([0-9]{3}\)(([0-9]{3}-[0-9]{4})|[0-9]{7})\b)/`


If that last regex didn't pass, we have or/pipe operator run the next block which checks for numbers in this format “678 667-8898” or  “678 7784547” and the last symbol is a word-boundary, stopping the search at a total of 10 numbers. Finishing up this implementation I decided to refine my solution with a true or false ternary operator.
