---
layout: post
title: "Create your first API wrapper with TDD"
date: 2015-02-04 22:47:14 -0500
comments: true
categories: 
---

Whether at you job or personal projects you would need at some point have to build you own API wrapper in ruby. Today, i decided to build my own Wrapper that i will turn into a gem in a later blog post. I want to do it methodically according to Test Driven Development principles.

It is always a hassle to test API calls but there is a way on how it should be done and we will talk about it later on.

I picked the [WMATA API](https://developer.wmata.com/) because it is well written and well documented. As you can see in their developer website each call has [a json response example](https://developer.wmata.com/docs/services/547636a6f9182302184cda78/operations/547636a6f918230da855363f). We will use these to test our code against it later.
 
Let's talk about our goal. I want to be able to set [my api key](https://developer.wmata.com/demokey) and then make an api call in this manner:

```ruby
WMATA.api_key = "kfgpmgvfgacx98de9q3xazww"
WMATA.next_trains("A01") # A01 stands for a station code
```

Now, that i know what i want, let's set up our TDD environment. The initial repository tree would look like this:

```ruby
lib
|___wamta.rb
spec
|___spec_helper.rb
|___trains_helper.rb
Gemfile
```
Let's take a look at the `Gemfile` and see what are the tools we are going to use:

```ruby
source 'https://rubygems.org'
gem 'httparty' #i use httparty to make http queries 
gem 'pry' #i use binding.pry to open a IRB console during runtime
gem 'webmock' , group: :test 
gem 'rspec' , group: :test # this is my favorite testing framework
```
Now that we have all the necessary gems do a `bundle install `. Let's take a look at the `spec/spec_helper.rb`:

```ruby
$:.unshift 'lib'
require 'wmata'
require 'webmock/rspec'
WebMock.disable_net_connect!(allow_localhost: true)
RSpec.configure do |config|
    config.before(:each) do
      stub_request(:get, %r|https://api.wmata.com/StationPrediction.svc/json/GetPrediction/|).
        to_return(status: 200, body: File.open("./spec/fixtures/next_trains.json"){|f| f.read}, headers: {})
    end
end    
```

This is where it get's tricky. If you rely on the API, your tests are not really tests, because they depend on the status of the API. If you have no internet, those tests will not work. Your tests should only test your own code, not the response of the API. We are using webmock to return to each call made to `https://api.wmata.com/StationPrediction.svc/json/GetPrediction/` the json response  in `next_trains.json`. This response is the expected result provided by the documentation [here](https://developer.wmata.com/docs/services/547636a6f9182302184cda78/operations/547636a6f918230da855363f)
 
 So let's add the file `next_trains.json` in `spec/fixtures/next_trains.json`

```json
{
"Trains":[
{
"Car":"6",
"Destination":"SilvrSpg",
"DestinationCode":"B08",
"DestinationName":"Silver Spring",
"Group":"1",
"Line":"RD",
"LocationCode":"A01",
"LocationName":"Metro Center",
"Min":"3"
},
{
"Car":"6",
"Destination":"Grsvnor",
"DestinationCode":"A11",
"DestinationName":"Grosvenor-Strathmore",
"Group":"2",
"Line":"RD",
"LocationCode":"A01",
"LocationName":"Metro Center",
"Min":"4"
}]}
```

We have everything setup now let's write our first test in `spec/train_spec.rb`.

```ruby
require 'spec_helper'
describe WMATA do
  before(:all) { WMATA.api = 'xxxxx'  }
     it 'should predict next trains' do
       expect(WMATA).to respond_to :next_trains
       next_trains = WMATA.next_trains
       expect(next_trains).not_to be_empty
       ["Car","Min","DestinationName"].each{ |field|
        expect(next_trains.first[field]).not_to be_nil
      }
   end
end
```
Notice that the api key does not have to be real because the call to the live api will not be made. It will be mocked and will return the expected json response. 

Let's create the module that will handle the API calls in `lib/wamta.rb`
```ruby
$:.unshift(File.dirname(__FILE__))
require 'client'
module WMATA
  def self.client
    @client ||= Client.new(api)
  end

  def self.api
    @api || raise("please set the api key")
  end

  def self.api=(api_key)
    @client = Client.new(api_key)
    @api = api_key
  end

  def self.next_trains(station = "A06")
    client.next_trains(station)
  end
end
```

Good, now we have a module that will create a client object when an api_key is passed and if we want to call `next_trains` without setting up the api key it will throw an exception "please set the api key". Now, let's create the Client class in `lib/client.rb` that will actually handle the API calls.

```ruby
require 'httparty'
module WMATA

  class Client
    include HTTParty
    base_uri "https://api.wmata.com"
    def initialize(api_key)
        @options = {query: {api_key: api_key} }
    end

    def next_trains(stations='A06')
      response = self.class.get("/StationPrediction.svc/json/GetPrediction/#{stations}", @options)
      JSON.parse(response.body)["Trains"]
    end
  end

end
```

When you run the tests now, everything should be green. This part makes sure that our code handles correctly the expected response, now let's make a live API call :

```ruby
WMATA.api = "kfgpmgvfgacx98de9q3xazww"
WMATA.next_trains("A01") # A01 stands for a station code
```

It should return an array of all the next trains for the specified station. Now that it is working we can do the same for all the other api calls

![alt](http://i.imgur.com/tEiuIfo.png)
