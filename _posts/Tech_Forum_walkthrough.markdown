---
layout: post
title: "Tech_Forum"
date: 05-17-2020 00:10:01 -0500
permalink: Tech_Forum_walkthrough
---


I had fun working on this project. My favorite part was working with css and html. This project is a CRUD, MVC app using Sinatra. Once the server is started, the user is provided with a Tech_Forum page where they can create an account to access the forums. 

The reason I had made a Tech_Forum based on creating How-To's related to computers is because I myself love to keep track of every guide I need for future debugging. I was able to generate the sinatra application by using the corneal gem. 

Ill go ahead and guide you through the steps I went through developing my project. First I made sure to establish the connection to the database. 

### Tech_Forum/config/environment.rb

This file acts as my environment where I can require any dependencies that I need to make the application run. 

```
ENV['SINATRA_ENV'] ||= "development"

require 'bundler/setup'
Bundler.require(:default, ENV['SINATRA_ENV'])

ActiveRecord::Base.establish_connection(
  :adapter => "sqlite3",
  :database => "db/#{ENV['SINATRA_ENV']}.sqlite"
)

require './app/controllers/application_controller'
require_all 'app'
```

### Tech_Forum/config.ru

```
require './config/environment'

if ActiveRecord::Base.connection.migration_context.needs_migration?
  raise 'Migrations are pending. Run `rake db:migrate` to resolve the issue.'
end

use Rack::MethodOverride
use UsersController
use ForumsController
use SessionsController
run ApplicationController
```

The config file runs and uses the controller files and also checks if rake db:migrate is needed to be entered through the terminal.   

### Tech_Forum/db/migrate

```
class CreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      t.string :username
      t.string :password
    end
  end
end
```
```
class CreateForums < ActiveRecord::Migration[6.0]
  def change
    create_table :forums do |t|
      t.string :title
      t.text :comment
      t.integer :user_id
    end
  end
end
```
These are the migration files where you create the tables with columns. 

### Tech_Forum/app/controllers

These are the generated controllers which include the application, users, sessions and forums controllers where the routes are connected. 

```
require './config/environment'

class ApplicationController < Sinatra::Base

  configure do
    set :public_folder, 'public'
    set :views, 'app/views'
    enable :sessions 
    set :session_secret, "secret"
  end

  get '/' do
    erb :welcome
  end

  def current_user
    User.find_by(id: session[:user_id])
  end

  def logged_in?
    !!current_user
  end

end
```
```
class ForumsController < ApplicationController

get "/forums/computer/new" do 
    if logged_in?
      erb :"/forums/computer/new.html"
    else
      redirect '/login'
    end
  end

  post "/forums/computer" do
    @guide = current_user.forums.build(params)
    if !params[:title].empty? && !params[:comment].empty?
      @guide.save
      redirect "/forums/computer"
    else
      @error = "Please submit title and comment."
      erb :'forums/computer/new.html'
    end
  end
```
```
class SessionsController < ApplicationController

  get '/login' do
    if logged_in?
      redirect '/home'
    else
      erb :'/users/login.html'
    end
  end

  post '/login' do 
    if params[:username].empty? || params[:password].empty?
      @error = "Please fill in Username and password"
      erb :'/users/login.html'
    else
      if user = User.find_by(username: params[:username] ,password: params[:password]) 
      session[:user_id] = user.id
      redirect '/home'
      else
      @error = "Account not found"
      erb :'/users/login.html'
      end
    end
  end
```
```
class UsersController < ApplicationController

  get '/register' do
    if logged_in?
      redirect '/home'
    else
      erb :'/users/register.html'
    end
  end

  post '/register' do
    user = User.new(params)
    if user.save
      session[:user_id] = user.id
      redirect '/home'
    elsif User.find_by(username: user.username)
      @error = "Account with that username already exists"
      erb :'/users/register.html'
    else
      @error = "Please fill out every field below"
      erb :'/users/register.html'
    end
  end
end
```
### Tech_Forum/app/models

This set of code is part of the models. The user model includes validations on username and password and uniqueness.The user model has a one to many association. The Forum model has a 1:1 connection with the user model. 

```
class Forum < ActiveRecord::Base
  belongs_to :user
end
```
```
class User < ActiveRecord::Base
  has_many :forums
  validates :username, :password, presence: true
  validates_uniqueness_of :username  
end
```

### Tech_Forum/app/views

This is part of the styling made to one of the few erb files I made. 

```
<style> 
  #UND { 
		text-decoration: none; 
  }
	div.big {
		line-height: 2.5;
		font-size: 2.0rem;
	}
	.body {
		background-color: #ffffff;
	}

</style>
<body class = "body">
	<div>
		<p hidden><%= computer = "/forums/computer" %> </p>
	</div>
	<div class="big">
		<a id = "UND" href= <%=computer%>>Computer - Post your guides</a>
  </div>
</body>
```

### Going foward

This application only works because I was able to utilize every piece of knowledge and tools I have learned. Going through this project I had to scrap a few ideas out because the features were not necessary and take to much fun of creating a simple application. I was able to learn more about html and css language which I very much enjoy. I can't wait to expand my knowledge on future projects.  

That's it for the walkthrough and I hope you got to learn something new. Thank you