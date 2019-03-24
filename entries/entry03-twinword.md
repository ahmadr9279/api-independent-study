# Entry 3: Twinword API

Now for the fun part.

## Wrapping my head around APIs

Before jumping specifically into the Twinword API, I decided to do a little research on APIs in general.  I didn't want to copy/paste code blindly, so I decided to just skim over a lot of different resources to get myself familiar with the vocabulary, syntax, flow, etc.

After googling "ruby APIs", the first result is Codecademy. The syntax in their example looks a little something like this:

```ruby
require 'open-uri'

kittens = open('http://placekitten.com/')
response_status = kittens.status
response_body = kittens.read[559, 441]

puts response_status
puts response_body
```

So it looks like they're using something called `open-uri`.  After googling *that*, [ruby-doc.org](https://ruby-doc.org/stdlib-2.1.0/libdoc/open-uri/rdoc/OpenURI.html) says that:
>OpenURI is an easy-to-use wrapper for Net::HTTP, Net::HTTPS and Net::FTP.  
>It is possible to open an http, https or ftp URL as though it were a file:

```ruby
open("http://www.ruby-lang.org/") {|f|
  f.each_line {|line| p line}
}
```

I wasn't exactly sure how (or if) this would work with the Twinword API, but at least Codecademy is getting me familiar with the language.  Their handful of lessons covered things such as 
- Authentication and API Keys (which I noticed Twinword requires)
- Making a request (which I know I'd need to do)
- Parsing JSON (which I think the Twinword API gives back, but I'm not sure)
- etc

## Mashape

Going back to the Twinword API [demo](https://www.twinword.com/api/word-associations.php), I saw the link for the documentation.  It's all hosted via something called **Mashape**, which is described as an online API marketplace.  My understanding is that Mashape is a hub for lots of different APIs. It is the source of both documentation and how you sign up to use the API.  

After signing up, I was given an `X-Mashape-Key`.  I'm assuming this key will go in my code, but I don't want my key to be visible on github, so I need to find a way to hide it.

As mentioned before, the first 10,000 queries are free.  I need to keep that in mind for the future so that I don't get charged any money.

## Documentation is your friend

Codecademy is good. Google is great. But all that searching only did me so good. The way that I eventually figured out how to use the specific Twinword API was to read the specific [Twinword API documentation](https://market.mashape.com/twinword/word-associations#word-associations-get).

I'm still not sure how `post`ing the API is different than `get`ting the API, but I'm pretty sure I want to `get`.  Luckily, they have code snippets for each language.

<img src="../images/03-documentation-languages" />

I decided to make a new model file `associate.rb` to test out this code individually before implementing it in the entire app.

```ruby
# These code snippets use an open-source library. http://unirest.io/ruby
response = Unirest.get "https://twinword-word-associations-v1.p.mashape.com/associations/?entry=sound",
  headers:{
    "X-Mashape-Key" => "<required>"
  }
```

Even after inserting my API key, the API didn't work. The file didn't know what `Unirest` was. But I noticed the comment about it and learned from [their site](http://unirest.io/ruby) that I need to install the `unirest` gem and require it at the top of my file.

That website also included an example of what the response would look like:

```ruby
response = Unirest.post "http://httpbin.org/post", 
                        headers:{ "Accept" => "application/json" }, 
                        parameters:{ :age => 23, :foo => "bar" }

response.code # Status code
response.headers # Response headers
response.body # Parsed body
response.raw_body # Unparsed body
```

I recognized the words **response headers** and **response body** from the [Twinword documentation on Mashape](https://market.mashape.com/twinword/word-associations#word-associations-get).  So I decided to

```ruby
puts response.headers
puts response.body
```

## It almost feels like magic

When I used the word "car", I got back this:

```ruby
{:cf_ray=>"342a4a121d9d56d5-IAD",
 :content_encoding=>"gzip",
 :content_type=>"application/json",
 :date=>"Mon, 20 Mar 2017 17:11:13 GMT",
 :server=>"Mashape/5.0.6",
 :set_cookie=>
  ["__cfduid=deee48d762c01cad2dd3a86dcff7c695d1490029872; expires=Tue, 20-Mar-18 17:11:12 GMT; path=/; domain=.twinword.com; HttpOnly"],
 :vary=>"Accept-Encoding",
 :x_content_type_options=>"nosniff",
 :x_ratelimit_queries_limit=>"10000",
 :x_ratelimit_queries_remaining=>"9990",
 :x_ua_compatible=>"IE=edge",
 :content_length=>"653",
 :connection=>"keep-alive"}
 
{"entry"=>"car",
 "request"=>"car",
 "response"=>{"car"=>1},
 "associations"=>
  "parking, bike, wagon, driver, ride, garage, vehicle, pedal, scooter, bus, automobile, cyclist, route, used, motorcycle, chauffeur, truck, traffic, drive, motorist, transporter, taxi, driveway, trucker, road, limousine, bicycle, surrey, motorcade, jalopy",
 "associations_array"=>
  ["parking",
   "bike",
   "wagon",
   "driver",
   "ride",
   "garage",
   "vehicle",
   "pedal",
   "scooter",
   "bus",
   "automobile",
   "cyclist",
   "route",
   "used",
   "motorcycle",
   "chauffeur",
   "truck",
   ...],
 "associations_scored"=>
  {"parking"=>1.5115365,
   "bike"=>1.4396062,
   "wagon"=>1.4217881,
   "driver"=>1.253938,
   "ride"=>1.2474502,
   "garage"=>1.2165134,
   "vehicle"=>1.0968634,
   "pedal"=>1.0428078,
   "scooter"=>1.0365075,
   "bus"=>1.027294,
   "automobile"=>0.97784275,
   "cyclist"=>0.95932865,
   "route"=>0.94497013,
   "used"=>0.9097221,
   "motorcycle"=>0.90569836,
   "chauffeur"=>0.894917,
   "truck"=>0.8807954,
   ...},
 "version"=>"4.0.0",
 "author"=>"twinword inc.",
 "email"=>"feedback@twinword.com",
 "result_code"=>"200",
 "result_msg"=>"Success"}
```

This is perfect! I can dive into `response.headers[:x_ratelimit_queries_remaining]` to grab the number of queries remaining.  And I can grab the `associations_array` and use the rest of the model to find an example/non-example.

I wanted to make sure this would work for any word, though.

## Making the method

Perhaps it might be good to eventually use object orientation, but I'll only do that if I feel like I need it.  For now, it seems simple enough to just build a method that takes in a word and gives back an array of associated words.  

```ruby
require 'unirest'
require 'pp'

def get_associations(word)
  response = Unirest.get "https://twinword-word-associations-v1.p.mashape.com/associations/?entry=#{word}",
  headers:{
    "X-Mashape-Key" => "<my key here>"
  }
  result = response.body
  header = response.headers
  remaining = header[:x_ratelimit_queries_remaining]
  puts "Remaining: #{remaining} queries."
  associations = result["associations_array"]
end

puts get_associations("car") #=> ["parking", "bike", "wagon", "driver", "ride", "garage", "vehicle", "pedal", "scooter", etc...
```

## The rest of the logic

Phew!  The hard part (the new stuff) was over.  All that was left to do was add the other logic in the main part of the model that found an example if given a non-example, and vice versa.  Just plain ruby stuff in the [model](https://github.com/brianmueller/green-glass-door/blob/3e49748d6b058fbaa19bb8a034fc4daca754158a/models/model.rb), [controller](https://github.com/brianmueller/green-glass-door/blob/3e49748d6b058fbaa19bb8a034fc4daca754158a/application_controller.rb), and [view](https://github.com/brianmueller/green-glass-door/blob/3e49748d6b058fbaa19bb8a034fc4daca754158a/views/stage1.erb).

## Takeaways

1. To learn about something brand new, I found it helpful to go down rabbit holes of Googling.  When I didn't understand something, I'd google it.  In my search results, if I found something in the explanation confusing, I'd google that as well.  [Lather, rinse, repeat](https://en.wikipedia.org/wiki/Lather,_rinse,_repeat).
2. Read the documentation. Then read it again. If you found yourself confused by it, read it one (or twelve) more time(s). The Twinword API documentation was my best source of troubleshooting.
3. If I want to add functionality (i.e. **m**odel) to my app, it's best to make sure that functionality works by itself on a single ruby file first. If I tried adding it to the app right away, and the app didn't work, I wouldn't know whether the code was wrong or the code was right but I implemented it wrong. Testing out the API code by itself makes me confident that the code works on its own before implementing it into the entire app.


## To-Do
1. Find a way to hide the API key.
2. Make sure I don't go beyond the 10,000 query/month limit.

[Previous](entry02-mvp.md)  |  [Next](entry04-gitignore.md)

[Table of Contents](../README.md)
