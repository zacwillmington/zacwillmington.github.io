---
layout: post
title:      "Adding Functionality to Your Website With JQuery"
date:       2018-05-10 15:42:18 +0000
permalink:  adding_functionality_to_your_website_with_jquery
---


## Introduction

Jquery is a great way to add functionality to your web app. Jquery makes it easy to select the elements you want to work with, which is why it is such a popular framework. There a plenty of companies that still use Jquery in their applications, so having a understanding will help you to get that first job, transition to another or level-up your skills. I am going to teach you how to “listen” for events(clicks or keystrokes) in your app with Jquery.

First Step
Don’t forget to add the <script> tag I have on line 13 to your body, or else none of the jQuery methods will work.

### HTML

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width">
    <title>repl.it</title>
    <link href="index.css" rel="stylesheet" type="text/css" />
  </head>
  <body>
  	<div class="content">Content
  	</div>
  	<button class="js-btn">Button</button>
  	<script src="//code.jquery.com/jquery.min.js"></script>
    <script src="index.js"></script>
  </body>
</html>
```

## What is an Event Listener?

An event listener “listens” for the something to happen to the id or the class name that is inside the parenthesis, it's usually a class name e.g. $(‘#id’) or $(‘.name_of_class’). Good thing about classes is that they can be used on multiple elements. In this case button “.js-btn” class is waiting to be clicked.

Event listeners must be followed by a JQuery method. The .on() event function is an event handler and the method we are using on the "js-btn" class. It is a frequently used method in JQuery. It's saying "when .js-btn is clicked run this function." 

 $("class/id").on( event/s="click" , handler = function);

Our function is now handling the event, which is on line 3. Notice a pattern? Another event lister and method $("div").hide(".content");. If you look the CSS code snippet below you'll see that .content sets the styling for the div. All that line on 3 does is hide the styling in the CSS file, rendering it invisible.

### JS

```
function foo(){
	$('.js-btn').on('click', function(event){
		$('div').hide('.content');
					//.remove();
					//.append();
					//.hide();
					//.show();
					//.hover();
					//.focus();
					// and many more..
	});
}

foo();
```

### CSS

```
.content{
	color: blue;
	border: 1px solid black;
	border-radius: 5px;
	width: 100px;
	height: 75px;
	text-align: center;

}
.js-btn{
	color: red;
	width: 100px;

}
```

This basic program above is just removing the content div, but what if we want to show it again? We can easily toggle between shown and hidden with the code below.

 
```
function foo(){
	$('.js-btn').on('click', function(event){
		$('.content').toggle();
					//.remove();
					//.append();
					//.hide();
					//.show();
					//.hover();
					//.focus();
					// and many more..
	});
}

foo();
```

The process goes, ".js-btn" is clicked function runs finding <div class ="content"> and setting the display to hidden. We can see this my inspecting with developer tools. Open a chrome browser, got to "https://VictoriousHonorableRoute--zacwillmington.repl.co" press option + command + j and click on the button. You should see something like this.

![](http://res.cloudinary.com/zacwillmington/image/upload/v1523821349/Screen_Shot_2018-04-15_at_12.38.58_PM_g5azys.png)


Here you'll see in the bottom box element.style {display: block;}. Now watch what happens when we click the button.

![](http://res.cloudinary.com/zacwillmington/image/upload/v1523821351/Screen_Shot_2018-04-15_at_12.39.26_PM_n8oawa.png)

The display is changed to none, yet the div is still there, we can see it in the box above. With JQuery we are manipulating the DOM. DOM stands for document object model, the basic idea is that the structure of the webpage represents the DOM. The DOM is an API(application programming interface) and it structures the HTML in to nodes so that we can easily manipulate it. In the picture above we are changing the state of the DOM by setting display to none. You'll hear DOM frequently and if you want to be a web developer you'll need to have a good understanding of how it works.

![](http://res.cloudinary.com/zacwillmington/image/upload/v1523974522/Screen_Shot_2018-04-17_at_7.11.22_AM_xmpznx.png)

Have a play around here you can have a go creating you own HTML CSS and JS. For more information on JQuery [](https://api.jquery.com/category/manipulation/).



https://repl.it/@zacwillmington/VictoriousHonorableRoute



