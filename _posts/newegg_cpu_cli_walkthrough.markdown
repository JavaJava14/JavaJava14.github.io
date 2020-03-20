---
layout: post
title: "Featuring Dekstop processors"
date: 2020-03-20 13:28:30 -0500
permalink: newegg_cpu_cli_walkthrough
---



It's amazing to put together a functional application from what I have been learning. This project is a command line interface which retrieves information on featured items from Newegg.com on their AMD desktop processors section. 

The reason I had selected Newegg is because its a very popular site that I have used multiple times. I was able to generate my gem and have a file structure layed out for me by using bundler. 

Ill go ahead and guide you through the steps I went through developing my cli. First I made sure to set up my environment dependencies 

### NEWEGG_CPU/lib/newegg_cpu.rb

This file acts as my environment where I can require any dependencies that I need to make the application run. 

```
require 'open-uri'
require 'nokogiri'
require 'httparty'
require 'pry'
require 'byebug'

require_relative "newegg_cpu/version"
require_relative './newegg_cpu/cli'
require_relative './newegg_cpu/cpu'
require_relative './newegg_cpu/scraper'
```

### NEWEGG_CPU/bin/run

```
NeweggCpu::CLI.new.call
```

Then I want to be able to see if I can execute my program through my run file by calling a method from another class. I do so by creating a call method then adding a message for the user by adding greet method to call. 

### NEWEGG_CPU/lib/cli.rb

```
  def call
    greet
  end

  def greet
    puts "\nNewegg featured items from Desktop Processors"
    puts "\nWelcome"
    puts ""
  end
```

### NEWEGG_CPU/lib/newegg_cpu/cpu.rb

This is where I add my attributes which will let the program know what info is linked to the dekstop proccesors.I provided the name, price and url. 

```
class NeweggCpu::Cpu

  attr_accessor :name, :price, :url

  @@all = []
  
  def initialize(name,price,url)
    @name = name
    @price = price
    @url = url
    @@all << self
  end

  def self.all 
    @@all
  end
end
```

### NEWEGG_CPU/lib/scraper.rb

This is where I use HTTParty to grab the raw html file and then parse it with Nokogiri to make it a readable format. I go and start iterating over the featured items on the page I provided. It grabs the name, price and url. Then I called the initialized method in cpu.rb. 

### NEWEGG_CPU/lib/cli.rb

After I'm done with the scraper class I add the ability for the user to interact with application. It starts off greeting the user and then giving them the option to search to generate a list of featured items or to exit out the program. It will ask for their input to select which item they like to know more about. If the input did not meet requirements then it will go ahead and ask for the input again. 

```
class NeweggCpu::CLI

  def call
    greet
    menu
  end

  def greet
    puts "\nNewegg featured items from Desktop Processors"
    puts "\nWelcome"
    puts ""
  end

  def generate_list 
    NeweggCpu::Scraper.get_cpu
  end

  def select_product
    puts ""
    puts "Enter any number from 1 through 36 to learn about the product"
    input = gets.chomp.to_i
    if input > 0 && input <= NeweggCpu::Cpu.all.length# 1-36
     NeweggCpu::Cpu.all[input - 1].tap do |item|
      puts ""
      puts "#{item.name}"
      puts "#{item.price}".gsub(/[\\nt–]/, "").strip
      puts "#{item.url}"
      select_product_continued
     end
    else 
      puts ""
      puts "You must enter a valid number"
      select_product
    end 
  end

  def select_product_continued
    puts ""
    puts "Enter list to view info on featured items or menu"
    input = gets.chomp
    case input.downcase
      when "menu" 
        puts ""
        puts "Enter list to view info on featured items."
        menu
        when "list"
          select_product
        else
        puts "Invalid response"
        select_product_continued
    end
  end

  def menu
    puts "Enter search to find featured items."
    puts "Enter exit to exit program."
    input = gets.chomp
    case input.downcase
      when "search"
        puts ""
        generate_list
        select_product
      when "list"
        select_product
      when "exit"
        puts ""
        puts "come back soon!"
        exit
      else
        puts ""
        puts "Sorry, that input is not recognized."
        puts ""
        menu
    end
  end
end
```

### Going foward

What was important for me to be able to develop a fucntional application was utilizing every piece of knowledge and tools I've learned. Also making sure I planned out what I wanted for my project. I did struggle with certain aspects of this project but I kept trying out new methods to get the program running. Everytime something didn't work it only lead to me new ideas and I was able to keep learning. This project was fun to play around with and I can't wait for future projects. 

That's it for the walkthrough and I hope you got to learn something new. Thank you
