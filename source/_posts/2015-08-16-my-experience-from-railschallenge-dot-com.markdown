---
layout: post
title: "My experience from Railschallenge.com"
date: 2015-08-16 18:13:16 -0400
comments: true
categories: 
---

#Rails challenge

A couple of months ago I stumbled on a reddit post talking about a [Rails challenge](http://railschallenge.com/). I accepted the challenge and finished it with no error. In this blog i would try to share what i learned form that experience.

#Rubocop
It was my first time using Rubocop and it was life changing. For those who don't know Rubocop, It is a static code analyser that would use the Ruby community outline and give you the list of offenses you have in your code. An example would be use `&&` instead of `and` (more information [here](http://devblog.avdi.org/2010/08/02/using-and-and-or-in-ruby/)).

#Serializers
I started by parsing my own JSON data structure and then i realized that it is getting too complicated to i used [ActiveModel::Serializer](https://github.com/rails-api/active_model_serializers). Serializers describe which attributes and relationships should be serialized. Adapters describe how attributes and relationships should be serialized.

One thing i struggled with was raising an expection when unpermitted parameters were used. I wasted tremendous amount of implementing that fuctionality instead of using the `config.action_controller.action_on_unpermitted_parameters = :raise`

#The Dispatcher algorithm
This was supposed to be the most complicated part of the challenge. After refining my algorithm a couple of times i realized that i could pass the test using this basic logic. I would order the `capacacity` of the `responder` by increasing order i would just assign available responder until in don't have anymore or until i responded fully to the emergency need. It would look like this.

```ruby
def dispatch
    RESPONDER_TYPE.each do |key,value|
      # find a responder on duty and available that can handle the emergency on his own
      responder = Responder.by_type(value).on_duty.available.where("capacity = ?", self.send(key))
      # see if we ahve an exact match
      if responder.count == 1
        self.responders << responder
      else
        # if there is no exact match add resources until it is enough for the emergency severity
        add_resource(value,self.send(key))
      end
    end
    self.fully_responded = true if self.enough_resources?
  end

  def add_resource(type,severity)
    responders_by_type = Responder.by_type(type).on_duty.available.order(capacity: :desc)
    responder_severity_sum = 0
    responder_list = []
    responders_by_type.each do |responder|
      if (responder_severity_sum < severity)
        responder_severity_sum += responder.capacity
        self.responders << responder
      end
    end
  end
```

#Conclusion
I really enjoyed working on this challenge. My only regret was not doing it before the deadline because they were going through your solution and grade it. I heard they will have other challenges soon, so stay tuned!
