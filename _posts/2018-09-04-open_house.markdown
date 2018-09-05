---
layout: post
title:      "Open House"
date:       2018-09-05 01:33:09 +0000
permalink:  open_house
---



## About the app
Open_house is an application for scheduling and posting real estate open houses. When creating and account you have two options, admin account or standard user. Admins can create listings and set the time of appointment for each of their listings. They can also delete and update the apartment. The standard user can search all apartments and make an appointment for the apartment they wish to see and if they cannot make the appointment they are able to cancel.

## Database design
Any project I start, I always begin by designing the database schema thinking up the models and their relation to each other. I knew I wanted a users table, appointments table and apartments table. I decided to make the appointments table a join table for a user and an apartment. The associations between models goes as follows.


```

class User < ApplicationRecord
    has_many :appointments
    has_many :apartments, through: :appointments    
end

class Appointment < ApplicationRecord
    belongs_to :user
    belongs_to :apartment
end

class Apartment < ApplicationRecord
    has_many :appointments
    has_many :users, through: :appointments
end
```

## Schema

```
ActiveRecord::Schema.define(version: 2018_08_26_164139) do

  create_table "apartments", force: :cascade do |t|
    t.string "address"
    t.datetime "available_times"
    t.string "image"
    t.integer "bedrooms"
    t.integer "bathrooms"
    t.integer "parking"
    t.float "price"
  end

  create_table "appointments", force: :cascade do |t|
    t.datetime "time"
    t.integer "user_id"
    t.integer "apartment_id"
    t.string "name"
  end

  create_table "users", force: :cascade do |t|
    t.string "name"
    t.string "email"
    t.string "password_digest"
    t.boolean "admin", default: false
    t.string "uid"
    t.string "phone"
  end

end
```

Once I have my models created and migrations run, my next is testing the functionality making sure the associations are working correctly. I do this in the rails console by creating instances and associating them.

## Routing Restfully

```
root 'sessions#index'

  resources :users do
      resources :appointments
  end

  resources :users do
      resources :apartments
  end

  resources :apartments do
      resources :appointments
  end

  resources :apartments

  get '/past_appointments' => 'apartments#past_appointments'

  get '/apartments' => 'apartments#index'


  get '/signup' => 'users#new'
  post '/signup' => 'users#create'

  get '/signin' => 'sessions#new'
  post '/signin' => 'sessions#create'

  get '/auth/:provider/callback' => 'sessions#create'

  get 'logout' => 'sessions#destroy'

end
```

Since this is my first Rails app I learnt how easy it is to create nested urls e.g. users/:id/apartments/:apartment_id or apartments/:id/appointments/:appointment_id

Resources :users creates all the routes you need for a model to be created, read, updated and deleted. And if you want a nested url you can nest a resource in a resource’s block.

```
  resources :users do
     resources :apartments
  end
```

That will give all the nested routes you need.

The reason I nested resources is to have the use of url params that are passed to different controller actions for example users/:id/apartments/:apartment_id conveniently stores the user's id and the apartment's id  in the params for easy access in the receiving controller action.


## Two Factor authentication

A really cool feature I enjoyed adding to my app is two factor authentication using Omniauth. I decided to authenticate with GitHub, but next project I'll use google or twitter for the account authentication for users that don't have a GitHub account.

![](http://res.cloudinary.com/zacwillmington/image/upload/v1536110758/Screen_Shot_2018-08-27_at_7.35.15_PM_xi1nyh.png)


Signing in takes us to the sessions controller.

When we hit that Signin With GitHub button, it sends us to the GitHub authentication page asking you to authorise open_house to use your account information. The callback request is sent back with information that includes uid(user id), email and username among other pieces of information. The three mentioned is all we need  from GitHub to create or find an account in our own database. There is more to two factor authentication which I’ll cover in another post, but for now I though I’d give you the gist.

```
def create
        if auth_hash = request.env["omniauth.auth"]
            @user = User.find_or_create_by_omniauth(auth_hash)

             if @user.save || @user.id != nil
                 session[:user_id] = @user.id
                 redirect_to user_path(@user)
             else
                  flash[:notice] = "Change email to public on github.com"
                  redirect_to '/signin'
              end
        else
            @user = User.find_by(:email => params[:email])
            if @user && @user.authenticate(params[:password])
                session[:user_id] = @user.id
                redirect_to user_path(helpers.current_user)

            else
                flash[:notice] = User.error_msg_for_signin_attempt(@user)
                render :new
            end
        end
    end
```
		
In the create action if the auth_hash is available then we know that it was successfully authenticated. Next it searches or creates a user with the custom find_or_create_by_omniauth method using the the info received from GitHub.

```
   def self.find_or_create_by_omniauth(auth_hash)
        oauth_email = auth_hash[:info][:email]

        self.where(:email => oauth_email).first_or_create do |user|
            user.name = auth_hash[:info][:name]
            user.email = auth_hash[:info][:email]
            user.password =  SecureRandom.hex
        end
    end
``` 
		
		If there is no auth_hash then we know the sigin in is a request from the sigin form, therefore authenticating the email with the password entered.

## User Destroy

One of the hurdles I came across in the development was the users/destroy action because only an admin can create an apartment. If an admin deletes his/her account there will be a showing with no leasing agent(admin) attending. This is bad, no one likes a no-show. I had to somehow delete the user, the user's appointments, all the apartments that belong to the user and appointments that have been made on each apartment which belong to non-admin accounts. If I lost you there then hang on, it will all make sense in the custom destroy methods I created on lines 6 and 8 below.

```
def destroy
        @user = User.find_by(:id => params[:id])

        if helpers.current_user == @user
            if helpers.current_user.admin
                @user.destroy_all_associated_appointments_belonging_to_apartments  #deletes all appointments tied to each apartment that the user created, which includes the appointments of people attending the showing.

                @user.destroy_all_apartments_belonging_to_appointments #deletes each apartment that the user created
            end
                @user.appointments.destroy_all
                @user.destroy
                redirect_to root_path
        else
            redirect_to user_path(helpers.current_user)
        end

    end
```

In the code above I added an if statement to check if the current user is an admin account. If the user is admin, then the user instance is called on these two custom destroy methods.

destroy_all_apartments_belonging_to_appointments deletes each apartment that the user created.


```
def destroy_all_apartments_belonging_to_appointments
        self.appointments.each do |appointment|
            appointment.apartment.destroy 
        end
end
```
	
	destroy_all_associated_appointments_belonging_to_apartments deletes all appointments associated to each apartment, which includes the appointments of people attending the showing. 

```
	def destroy_all_associated_appointments_belonging_to_apartments
        self.appointments.each do |appointment|
            appointment.apartment.appointments.destroy_all
        end
  end
```
		
		Have a look at the code here. [](https://github.com/zacwillmington/open_house)
