---
layout: post
title:      "Build a Flashcard App with Sinatra!"
date:       2018-07-03 00:16:07 +0000
permalink:  build_a_flashcard_app_with_sinatra
---


In this post I’m going to walk you through my simple CRUD (create, read, update, delete) application. I decided to make a flashcard app, seeing as it ticks all the boxes for a CRUD application and I have used many flashcard apps before. I learned a lot about object relations, sessions and controller actions while building this application. I used HTML, CSS and Sinatra to develop it. Here’s how it went.

## The Process

1.Setting out my file structure requiring gems and establishing a database connection.
2. Creating migrations and setting up tables; users, decks and cards. 
3. Connect controller actions to views.
4. Code up erb files to display forms allowing me to create, read, update and delete any information of the user’s decks or cards.
5. Once it was fully functional I added styling to the CSS file. 

## Database Structure(schema)
I created migrations for three tables; users, decks and cards. The relations between models are as follows; a user has many decks,  a deck belongs to a user and has many cards and a card belongs to a deck. Now I know the relations I created my models “user, deck and card.” Thanks to Activerecord, the object relations were a breeze. Here's the schema.

``` 
create_table "cards", force: :cascade do |t|
    t.string  "front_side"
    t.string  "back_side"
    t.integer "deck_id"
    t.string  "learnt"
    t.boolean "flip"
  end

  create_table "decks", force: :cascade do |t|
    t.string  "title"
    t.string  "category"
    t.integer "user_id"
    t.integer "number_of_cards"
  end

  create_table "users", force: :cascade do |t|
    t.string "username"
    t.string "email"
    t.string "password_digest"
  end

end
```

Activerecord recognizes relationships between classes by the user_id and deck_id. Once I had migrated my database I could use my Pry console to check if the objects were interacting properly by creating a temporary objects for a user, deck and card.

## Controllers

Before hooking up all of my links and controller actions I mount my controllers in the config.ru file so my application_controller file can access the controllers for each class. Next is to create my connections between each controller action and erb file.  Once I am able to navigate to each of my pages (show, new, edit and delete) now I can move on to the HTML forms.

## Views

Up until this point my app doesn't do very much. There is no data in the DB because I'm unable to submit information pertaining to each user and their decks and cards. That's where forms come in.
On each page I made a form, crafting it to the action I wanted the request to perform. For example.

```
<h1 class="login">Login</h1>
<div class="login-form">
    <form action="/login" method="post">
        <label for="username">Username</label>
        <input id="username" type="text" name="username" required>
        <br>
        <label for="email">Email</label>
        <input id="email" type="text" name="email" required>
        <br>
        <label for="password">Password</label>
        <input id="password" type="password" name="password" required>
        <br>
        <input type="submit" value="Login">
    </form>
</div>
```

On this post request the params hash will look something like this.

```
params = {:username => "example user",
           :email => "example@email.com",
           :password => "password-one"
           }
```


Next, the login form directs us to the controller action. Here I made sure to check if the user already has an account. If so, then I authenticate the encrypted password with params[:password], then set the session[:id] to the logged in user's ID.

```
post '/login' do
         @user = User.find_by(:email => params['email'])
         if @user != nil && @user.authenticate( params['password'])
            session[:id] = @user.id
            redirect to "/users/#{current_user.id}"
        else
            flash[:message] = {:error => ["Looks like you don't have an account. Try signing up."]}
            redirect to '/login'
        end
     end
```

Now that you know the rough overview of how I made this flashcard app, I'll run you through some of the technical challenges that I faced.

## Next Card Feature

The next card feature was a little tricky. I went with a separate controller action and a form to submit the request.

```
<form class="delete-card" action="/users/<%=current_user.id%>/decks/<%=current_deck.id%>/cards/<%=current_card.id%>/next" method="post">
        <input type="submit" value="Next card">
 </form>
```

When the request is posted it hits my controller action `/users/:id/decks/:deck_id/cards/:card_id/next`.

```
post '/users/:id/decks/:deck_id/cards/:card_id/next' do
        next_card_id(current_card
            .deck)
        if @next_id == nil
            redirect to "/users/#{current_user.id}/decks/#{current_deck.id}/cards/#{current_deck.cards.first.id}"
        else
            redirect to "/users/#{current_user.id}/decks/#{current_deck.id}/cards/#{@next_id}"
        end
    end
```

next_card_id() is defined in the helpers block in the application controller.  It makes it easy access all controllers from inside the method. The goal of this method is to find the nth + 1 value in an array of ids. Because the array of ids can be in ascending order with gaps (ids = [2, 3, 22, 31, 40, 42]) I figured an index assigned to each number can pinpoint in the nth value in the array, which in this case is the current id. Then, increment the ids[index + 1] to find the next value's integer. This this is an iterative approach to the problem.

At most, I believe one person won't create more than one thousand cards. In the worst case, with a runtime of O(n) if the last ID in the array is the @current_value set on line 2 which will demand the iteration to pass over every ID. 


```
def next_card_id(deck)
           @current_id = params[:card_id].to_i
           @card_ids = deck.cards.ids
           
           @card_ids.each_with_index do |id, index|
                if @current_id == id
                    @next_id = @card_ids[index + 1]
                end
                if @next_id == nil
                    @last_id = @current_id
                end
           end
       end
```

Based on how I've written the method I'll know that if @next_id is equal to nil then I've reached the final ID in the array.  At that point it will redirect back to the first card.

```

post '/users/:id/decks/:deck_id/cards/:card_id/next' do
        next_card_id(current_card
            .deck)
        if @next_id == nil
            redirect to "/users/#{current_user.id}/decks/#{current_deck.id}/cards/#{current_deck.cards.first.id}"
        else
            redirect to "/users/#{current_user.id}/decks/#{current_deck.id}/cards/#{@next_id}"
        end
    end
```


## Flip Card Feature

When I was thinking about how to make a card flip I was going to use JQuery to hide the backside of the card and maybe use a cool CSS transition to show the card flipping over and hiding the frontside simultaneously. It sounds good in theory, but perhaps if I had some time to work on the frontend I would've made this feature better.

I made the decision to make unintuitive flip and hide buttons with separate forms that post to the same controller action.

```
<form class="delete-card" action="/users/<%=current_user.id%>/decks/<%=current_deck.id%>/cards/<%=current_card.id%>/flip" method="post">
        <input type="submit" name="Flip" value="Flip">
    </form>
    <form class="delete-card" action="/users/<%=current_user.id%>/decks/<%=current_deck.id%>/cards/<%=current_card.id%>/flip" method="post">
        <input type="submit" name="Hide" value="Hide">
    </form>
```

How the controller action works depends on what key values the params hash contains from the form submission. If the flip button is clicked the params = {‘Flip’ => [‘Flip’]} is received. I update the flip key based on this.  I don't like the feeling of changing data of an object just to show or hide the front and back sides. Maybe I'll drop the :filp row from the database and pull in JQuery like I mentioned earlier to do this in a better way.

Here in the erb file the if statement checks @card.filp displaying the backside if it evaluates to true.

```
<h2><%=current_card.front_side%></h2>
<% if current_card.flip == true %>
  <h2><%=current_card.back_side%></h2>
<% end %>
```

## Flash error message

I've found a really cool feature that Activerecord has built in for error messages. In my model class I used the validates keyword followed by the object attributes and their constraints. I wanted to make sure every CRUD action has valid information coming from the form submisson.

```
class User < ActiveRecord::Base
    has_many :decks
    has_secure_password
    validates :username, presence: true, length: { minimum: 6 }, uniqueness: true
    validates :email, presence: true
    validates :password, presence: true, length: { in: 6..15, too_short: "%{count} characters is the minimum allowed" }
end
```
```
<h1 class="signup">Sign Up</h1>

<div class="signup-form">
    <form action="/signup" method="post">
        <label for="username">Username</label>
        <input id="username" type="text" name="username" >
        <br>
        <label for="email">Email</label>
        <input id="email" type="text" name="email" >
        <br>
        <label for="password">Password</label>
        <input id="password" type="password" name="password">
        <br>
        <input type="submit" value="Signup">
    </form>
</div>
view raw
```

When the form is submitted it hits the '/signup' post action, checks if the user is logged in and creates the new user object. This is where Activerecord comes in. handy. Those validates' lines in the user model automatically store the errors in the user object in the @user.erros.messages method. If I were to fill in a password that is 4 characters, then I'd get a message stored in messages like: messages = {:password => ["6 characters is the minimum allowed"]}.

```
post '/signup' do
          if is_logged_in?
              redirect to "/users/#{current_user.id}"
          else
            @user = User.new(:username => params['username'], :email => params['email'], :password => params['password'])
            if @user.valid? && @user.present?
                @user.save
                session[:id] = @user.id
                redirect to "/users/#{current_user.id}"
            else
                flash[:message] = @user.errors.messages
                redirect to '/signup'
            end
        end
     end
```

This makes it easy for me, custom message that write them selves.  I wanted to display the messages above the login form, but the problem was I would need to have a Rails form to display them. 

The issue I was running in to was that if the object was not valid then when I redirected to the signup page the object no longer existed because it was not saved to the database, and it was a new route back to the signup page. This gave me a NoMethodError for nil class. If I was running Rails then I could have just used the render method to render the signup.erb file and along with a Rails form I could display the errors.
To get around this issue I decide to use flash gem to hold all my errors that Activerecord created for me. These errors were held from the time I set them(on line 11 above) until after I redirected to the signup.erb file.

```
 <%if flash[:message]%>
            <div class="errors">
                <ul>
                <%flash[:message].each do |error, message|%>
                    <li><%=error.capitalize.to_s.gsub("_", " ")%>: <%=message[0]%></li>
                <%end%>
                </ul>
            </div>
        <%end%>
          <%= yield %>
```


To keep it fairly DRY I have an if block in the layout.erb file to display it on every page, so long as there is a error message present.

## Authentication/security

My security comes down to verifying the user's ID with current_user method. If they don't have an account then I redirect them to the signup page.

```
def current_user
    @current_user ||= User.find(session[:id]) if session[:id]
  end
```

In the login action I first check to see if there is a user in the database with the email address. I checked with the email because emails are unique. If there is an email in the database then I see if the value in the params is the same as the encrypted password. I do this by using bcrypt #authenticate(params[:password]) method, passing in params.

```
 post '/login' do
         @user = User.find_by(:email => params['email'])
         if @user != nil && @user.authenticate( params['password'])
            session[:id] = @user.id
            redirect to "/users/#{current_user.id}"
        else
            flash[:message] = {:error => ["Looks like you don't have an account. Try signing up."]}
            redirect to '/login'
        end
     end
```

If they passed those conditions then the session is all theirs allowing them to browse till their little heart’s content.

At this point security is a known unknown for me. I haven't gone deep into security yet, so I'm sure this is a weak way of keep my users secure. But keep posted for an article in the future where I will go into more depth with Rails security.

## Conclusion

I had so much fun making this app, it's definitely my favourite project to date. It's really interesting working across models, views and controllers to get them to working together. There is so much more I could do to this app to refine it to my liking, but that's for another time.

You can check out the github repo[ here](https://github.com/zacwillmington/lembra). ![](http://)
 

