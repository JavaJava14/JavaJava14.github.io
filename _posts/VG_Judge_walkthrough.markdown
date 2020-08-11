---
layout: post
title: "VG_Judge"
date: 07-21-2020 00:10:01 -0500
permalink: VG_Judge_walkthrough
---

Though this project had some similarities from the sinatra project, there was more to build on the application using rails. Once the server is started, the user is provided with a VG_Judge page where they can create an account to access their home page. 

The reason I had made VG_Judge which is an application that allows the user to review games is because I enjoy games and I like to know the reviews on a game before purchasing it. 

Ill go ahead and guide you through the steps I went through developing my project. First I setup my database. 

### VG_Judge/db/migrate
```
    create_table :users do |t|
      t.string :username
      t.string :password_digest
    end
```
```
    create_table :games do |t|
      t.string :title
      t.string :developer
      t.integer :year
      t.integer :user_id
      t.integer :review_id
    end
```
For this project I went ahead and created the users, games, reviews and genres table in order to complete the requirements shown in the next section in my models. 
### VG_Judge/app/models

These set of models include at least one has_many, at least one belongs_to, and at least two has_many :through relationships. It also includes a many-to-many relationship implemented with has_many :through associations. The join tabe includes a user-submittable attribute. These models also include reasonable validations such as presence, uniqueness and numericality. Scope method is also used to retrieve and query objects. It returns all games found in the database where game title matches text field input that the user creates and adds to params for title.
```
class Game < ActiveRecord::Base
  belongs_to :user
  belongs_to :review
  ...
  scope :search_by_title, -> (search_title){where("title = ?", search_title)}
end
```
```
class User < ActiveRecord::Base
  has_many :games
  has_many :reviews, through: :games
  validates :username, :password, presence: true
  validates_uniqueness_of :username
  has_secure_password
end
```
```
class Review < ApplicationRecord
  has_many :users, through: :games
  validates_numericality_of :rating, :greater_than_or_equal_to => 1, :less_than_or_equal_to =>10
end
```
### VG_Judge/config/routes.rb
 
Nested resources with the appropriate RESTful URLs are used. It makes it easier to generate paths and URLs which avoids hardcoded strings in views. 
```
  resources :users
  resources :games
  resources :reviews
  resources :sessions

  resources :users do
    resources :games
  end

end
```
### VG_Judge/app/views/layouts/application.html.erb
This set of code represents a `new` nested route
```
<li class = "navbar_newgr"><%= link_to("New Game Review", new_user_game_path(current_user), :class => 'a-bar') %></li>
```
### VG_Judge/app/views/games/index.erb
This set of code represents a `index` nested route
```
<% @games.each do |game| %>
<td><%= link_to game.user.username, user_games_path(game.user) %></td>
```
### VG_Judge/app/controllers

These are the generated controllers which include the application, users, sessions and games controllers where the CRUD is available. 
### Controllers/application_controller.rb
In my ApplicationController I wrote my helper methods to ensure that the user is logged in to their account. 
```
  helper_method :current_user
  helper_method :logged_in?

  def current_user
    @user = User.find_by(id: session[:user_id])
  end

  def logged_in?
   !!current_user
  end

  def require_login
    if !logged_in?
      flash[:alert] = "Enter credentials to login"
      redirect_to root_path
    end
  end
end
```
### controllers/games_controller.rb
I made use of a before filter in both users and games controller which requires the user to be logged in before the action can run. In order to DRY my code I also added another before filter to find the id of game. I also achieved DRY code by using methods from has_many association such as the build metho which returns one or more new objects of the collection type that have been instantiated with attributes and liked to this object through a foreign key, but have not yet been saved. 
### https://apidock.com/rails/ActiveRecord/Associations/ClassMethods/has_many
```
  before_action :find_game, only: [:show, :edit, :destroy]
  before_action :require_login
  
  def create
    @genres = Genre.all
    @game = current_user.games.build(game_params) 
    if @game.save
      redirect_to game_path(@game)
    else
      render :new
    end
  end
end
```
### The use of Bcrypt and authentication method
Using bcrypt in my application allowed it be less vunerable. Bcrypt handles validating password and password_confirmation and converts password into the password_digest which is in the user table. Using methods such as authenticate from has_secure_password in the user model made it possible to return true if the password is correct from the password string. It makes sure the user is who they say they are.
### controllers/sessions_controller.rb
In our sessions controller the user is able to login locally or via github. The user is able to logout successfully which is done by deleting the session. 
```
  def create
    if auth
      @user = User.find_or_create_by(uid: auth['uid']) do |u|
        u.username = auth['info']['name']
        u.password = SecureRandom.hex
      ...
      @user = User.find_by(username: params[:username])
      if @user && @user.authenticate(params[:password])
      ...

  def destroy 
    session.delete :user_id
    redirect_to root_path
  end

  private

  def auth
    request.env['omniauth.auth']
  end

end
``` 
### controllers/users_controller.rb
Users controller is where new user object is instantiated and sets key :user_id in the session hash and stores the user's id in it. I use the permit and require method here and also in my other controllers which helped me set the parameter as permitted and limit which attributes should be allowed for mass updating. The require method helps makes sure that a specific parameter is present. 
### https://api.rubyonrails.org/classes/ActionController/Parameters.html
```
  def create
    @user = User.new(user_params)
    if @user.save
      session[:user_id] = @user.id
      redirect_to user_path(@user)
    else
      render :new
    end
  end

  ...

  private

  def user_params
    params.require(:user).permit(:username, :password)
  end

end
```
### views/users/new.erb
The form_for here allows us to create a form which the user creates the attributes for the user model object. This form also displays validation errors in use of a partial.  
```
  <%= form_for @user, controller: 'users', action: 'create' do |f| %>
  <div class = "fields_with_errors"><%= render 'login_error'%></div>
  <div>
    <ul>
     <li> 
      <%= f.label :username, :class => "label_signup"%>
      <%= f.text_field :username, :class => "field_signup" %>
     </li>
     <li>
      <%= f.label :password, :class => "label_signup2" %>
      <%= f.text_field :password, :class => "field_signup2" %>
    ...
```
### views/users/_login_error.erb
```
  <% if @user.errors.any? %>
    <% @user.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
  <% end %>
```
### Going foward

I was able to grab my previous experience with the Sinatra project and use it here. There are many other features that I wanted to include but I made sure that I met the requirements and not go overboard. I did have trouble working with has_many :through association but I eventually went through trial and error to get results. I very much enjoyed learning new methods, html and css. I can't wait to expand my knowledge on future projects. That's it for the walkthrough and I hope you got to learn something new. Thank you
