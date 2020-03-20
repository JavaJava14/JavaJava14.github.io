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
Then I want to be able to see if I can execute my program through my run file by calling a method from another class. I do so by creating a call method then adding a message for the user by adding greet method to call. 
### NEWEGG_CPU/lib/cli.rb
```
NeweggCpu::CLI.new.call
```
```
def call
    greet
    menu
  end

  def greet
    puts "\nNewegg featured items from Desktop Processors"
    puts "\nWelcome"
    puts ""
  end
```
