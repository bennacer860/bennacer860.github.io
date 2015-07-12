---
layout: post
title: "HTTParty debugging"
date: 2015-01-02 15:24:47 -0500
comments: true
categories: 
---

Whenever you start working on API call you should use HTTParty. In my opinion it is the best tool to make a API wrapper in a very modular way.I have been playing with it lately, and i got this weird error 

```
/Users/***/.rvm/rubies/ruby-2.1.5/lib/ruby/2.1.0/uri/generic.rb:214:in `initialize':
the scheme http does not accept registry part: :80 (or bad hostname?) (URI::InvalidURIError)`
```

Well, i know that HTTParty rely on URI module, but i would like to have more details in order to debug it. How about getting more verbose for the http requests i am making. The solution is easy, HTTParty provides a simple tool called debug_output. You can display the debug in stderr and it will show you only the errors or jsut be more verbose and use stdout, it woudl be something like:


```ruby
include HTTParty
debug_output $stdout
```
let see the output:

```
opening connection to api.themoviedb.org:80...    
opened                                         
<- "GET /3movie/12?api_key=87465574ffc9 HTTP/1.1\r\nAccept: application/json\r\nContent-Type: application/json\r\nConnection: close\r\nHost: api.themoviedb.org\r\n\r\n"                                                  
-> "HTTP/1.1 404 Not Found\r\n"                                                                                        
-> "Content-Type: text/html\r\n"                                                                                       
-> "Date: Fri, 02 Jan 2015 20:20:30 GMT\r\n"                                                                           
-> "Server: nginx\r\n"                                                                                                 
-> "Content-Length: 162\r\n"                                                                                           
-> "Connection: Close\r\n"

```

Mmmm...i was making a /3movie/ call instead of movie/3, i know what my problem is now.
