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
class CreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      t.string :username
      t.string :password_digest
    end
  end
end
```
```
class CreateGames < ActiveRecord::Migration[6.0]
  def change
    create_table :games do |t|
      t.string :title
      t.string :developer
      t.integer :year
      t.integer :user_id
      t.integer :review_id
    end
  end
end
```
```
class CreateReviews < ActiveRecord::Migration[6.0]
  def change
    create_table :reviews do |t|
      t.string :summary
      t.text :opinion
      t.integer :rating
    end
  end
end
```
```
class CreateGenres < ActiveRecord::Migration[6.0]
  def change
    create_table :genres do |t|
      t.string :name
      t.integer :game_id
    end
  end
end
```
These are the migration files where you create the tables with columns. 

### VG_Judge/config/routes.rb

Nested resources with the appropriate RESTful URLs are used.
```
Rails.application.routes.draw do
  # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html
  root "application#index"

  resources :users
  resources :games
  resources :reviews
  resources :sessions, only: [:new]

  resources :users do
    resources :games
  end
 
  get '/sessions', to: 'sessions#new'
  post '/sessions', to: 'sessions#create'
  get '/logout/:id', to: 'sessions#destroy'
  delete '/logout/:id', to: 'sessions#destroy', as: 'logout'
  get '/auth/:provider/callback', to: 'sessions#omniauth'
  get '/most_user', to: "users#most_users", as: 'most_users'
end
```

### VG_Judge/app/controllers

These are the generated controllers which include the application, users, sessions and games controllers where the CRUD is available. 

```
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
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
```
class GamesController < ApplicationController
  before_action :current_user
  
  def index
    if params[:user_id]
      @games = User.find(params[:user_id]).games
    elsif params[:title]
      @games = Game.search_by_title(params[:title])
    else
      @games = Game.all
    end
  end 

  def new
    @game = Game.new
    @game.build_review
  end

  def create
    @genres = Genre.all
    @game = current_user.games.build(game_params) 
    if @game.save
      redirect_to game_path(@game)
    else
      render :new
    end
  end

  def show
    @game = Game.find(params[:id])
  end

  def edit
    @game = Game.find(params[:id])
  end
  
  def update
    @game = Game.find(params[:id])
    if @game.update(game_params)
      redirect_to game_path(@game)
    else
      render :edit
    end
  end

  def destroy
    @game = Game.find(params[:id])
    @game.destroy
    redirect_to user_path(current_user)
  end

  private

  def game_params
    params.require(:game).permit(:title, :developer, :year, :genre_ids, review_attributes: [:summary, :opinion, :rating])
  end
  
end
```
```
class SessionsController < ApplicationController

  def new
    if !current_user
      @user = User.new
    else
      redirect_to user_path(current_user)
    end
  end

  def create
    @user = User.find_by(username: params[:username])
    if @user && @user.authenticate(params[:password])
      session[:user_id] = @user.id
      redirect_to user_path(@user)
    else
      flash[:alert] = "Your login credentials were incorrect. Please try again."
      redirect_to sessions_path(@user)
    end
  end

  def destroy 
    session.delete :user_id
    redirect_to root_path
  end

  def omniauth
    @user = User.from_omniauth(auth)
    @user.save
    session[:user_id] = @user.id
    redirect_to user_path(@user)
  end

  private

  def auth
    request.env['omniauth.auth']
  end

end
```
```
class UsersController <  ApplicationController
  before_action :current_user, only: [:show]
  before_action :require_login, only: [:show]

  def new
    if !current_user
      @user = User.new
    else
      redirect_to user_path(current_user)
    end
  end

  def create
    @user = User.new(user_params)
    if @user.save
      session[:user_id] = @user.id
      redirect_to user_path(@user)
    else
      render :new
    end
  end

  def show
    @game = Game.all
  end

  def most_users
    @users = User.joins(:games).group(:id).order('COUNT(games.id) DESC').limit(1)
  end

  private

  def user_params
    params.require(:user).permit(:username, :password)
  end

end
```
### VG_Judge/app/models

This set of code is part of the models. These set of models include at least one has_many, at least one belongs_to, and at least two has_many :through relationships. It also includes a many-to-many relationship implemented with has_many :through associations. 
```
class Game < ActiveRecord::Base
  belongs_to :user
  belongs_to :review
  has_many :genres
  accepts_nested_attributes_for :review
  validates :title, :developer, :year, :genre_ids, presence: true
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
class Session < ActiveRecord:Base
  validates :username, presence: true
  validates :password, presence: true
end
```
```
class Review < ApplicationRecord
  has_many :games
  has_many :users, through: :games
  validates :rating, :summary, :opinion, presence: true
  validates_numericality_of :rating, :greater_than_or_equal_to => 1, :less_than_or_equal_to =>10
end
```
```
class Genre < ApplicationRecord
	belongs_to :game, optional: true
end
```

### VG_Judge/app/views

The User new form has built in validation errors.Uses partials to display errors. 

```
<div class = "register">
<h1>CREATE AN ACCOUNT</h1>
</div>
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
     </li>
    </ul>
      <%= f.submit "Sign Up", :class => 'submit_signup' %>
  </div>
  <% end %>
</div>
```

### Going foward

I was able to grab my previous experience with the Sinatra project and use it here. There are many other features that I wanted to include but I made sure that I met the requirements and not go overboard. I very much enjoyed learning new methods, html and css. I can't wait to expand my knowledge on future projects. That's it for the walkthrough and I hope you got to learn something new. Thank you