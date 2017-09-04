---
layout: post
title: Kele
thumbnail-path: "/img/Kele/Bloc API.png"
short-description: Kele is a Ruby Gem API client built to access the Bloc API.
---

{:.center}
![]({{ site.baseurl}}/img/Kele/Bloc API.png)

## What is Kele?

Kele is a basic Ruby gem built to provide easy use access to the student level endpoints of the Bloc API. The goal here was to become more comfortable working with JSON Web Tokens, gain a better understanding of API clients, and learn how to build a Ruby gem. The name Kele, actually comes from the lead singer of the British indie rock band Bloc Party.

## Technologies Used

* Ruby
* Git
* JSON
* [Bloc API](http://docs.blocapi.apiary.io/#reference)

## User Stories

As I mentioned, I wanted to build a simple Ruby gem that would access a student's endpoint information in the Bloc API. I could have accessed everything using cURL, but I wanted an API client to allow for easy requests and response handling. 

1. I want users to initialize and authorize the Kele client using a Bloc username and password.
2. I want users to be able to retrieve the current user as a JSON blob.
3. I want users to be able to pull a list of their mentor's availability.
4. I want users to be able to view their roadmap and checkpoint. 
5. I want the ability to retrieve a list of messages, respond to an existing messasge, and create a new message thread for users. 
6. I want the user to be able to submit a checkpoint assignment.

### User Story 1: Initialization

I chose to build this project as a Ruby gem so that I could integrate it with other software that I might build in the future. My first step was building out the .gemspec file and defining the project's metadata. 

```ruby

Gem::Specification.new do |s|
    s.name          =   'kele'
    s.version       =   '0.0.1'
    s.date          =   '2017-07-22'
    s.summary       =   'Kele API Client'
    s.description   =   'A client for the Bloc API'
    s.authors       =   ['Sami Hamdan']
    s.email         =   'samihamdan00@gmail.com'
    s.files         =   ['lib/kele.rb']
    s.require_paths =   ["lib"]
    s.homepage      =
        'http://rubygems.org/gems/kele'
    s.license       =   'MIT'
    s.add_runtime_dependency 'httparty', '~>0.13'
end
```

I also installed httparty to provide a Ruby interface to make HTTP requests. For now, I'm only including lib/kele.rb in my files array, and even that is wishful coding as we haven't yet created that file. But I should note here that you will want to include future files in this array down the road. Later on we will create a module to go with our gem, and that should be listed in the file array. 

Finally, I want to create my lib/kele.rb file which will be the base file for my gem and will load when I call `require './lib/kele`.  Within this file I want to add an initialize method that creates a new Kele client when authorized with a valid username and password. In my intialization method I populated two instance variables:

1. The Bloc base API URL
2. The user's authentication token, retrievable from the Bloc API sessions endpoint. 

```ruby
class Kele
    include HTTParty

    def initialize(email, password)
        @options = {query: {email: email, password: password}}
        response = self.class.post("https://www.bloc.io/api/v1/sessions", @options)
        @auth_token = response["auth_token"]
    end
end
```
After I was done with my initialization method, it was time to test in IRB. I wanted to be sure that I had retrieved and stored my authentication token when I passed in a valid username and password. I also wanted to be sure an error was being raised when I passed invalid credentials.

```
$ irb
>> require './lib/kele'
=> true
>> Kele.new("useremail@bloc.com", "password")
```
This should return an JSON response with your email and password and an encrypted auth token when passed a valid username and password. It will return an empty auth token if you don't pass valid credentials. 

### User Story 2: JSON retrieval

Next, I want to retrieve the current user as a JSON blob and convert that to a Ruby hash. I created a new #getMe method and added the JSON gem as a runtime dependency. I used the #parse method to convert the JSON response to Ruby hash. 

I passed the auth token to the request using the `headers` option and returned the response object using the HTTParty #body method. 

```ruby
    def get_me
        response = self.class.get('https://www.bloc.io/api/v1/users/me', headers: {"authorization" => @auth_token})
        JSON.parse(response.body)
    end
```
After I added my method, I opened IRB to make sure that I could retrieve my user data and that data was being converted to a Ruby hash. 

![getMe response](/img/Kele/IRB-getMe.png)

### User Story 3: Pull Mentor's Availability

Retrieving a list of mentor availabilty would require accessing current user data, and using that information to pull the availability. I created a #getMentorAvailability method and added a mentorId argument. That info could be pulled from currentUser. 

```ruby
    def get_mentor_availability(mentor_id)
        response = self.class.get("https://www.bloc.io/api/v1/mentors/#{mentor_id}/student_availability", headers: {"authorization" => @auth_token})
        JSON.parse(response.body)
    end
```

Once again I converted my JSON response to a Ruby array after authenticating against the Bloc API. I opened IRB again and tested to make sure that I could retrieve a list of my mentor's available time slots and I was converting the response to a Ruby array.

### User Story 4: User Roadmap and Checkpoints

I next wanted users to be able to retrieve their roadmaps and checkpoints. I defined two methods, #getRoadmap and #getCheckpoint, and put them into a new file `lib/roadmap.rb`. I then made sure to require and include the new module in my `lib/kele.rb` file, and added the file to my gemspec file array that I mentioned earlier. 

```ruby
module Roadmap

    def get_roadmap(roadmap_id)
        response = self.class.get("https://www.bloc.io/api/v1/roadmaps/#{roadmap_id}", headers: {"authorization" => @auth_token})
        JSON.parse(response.body)
    end

    def get_checkpoint(checkpoint_id)
        response = self.class.get("https://www.bloc.io/api/v1/checkpoints/#{checkpoint_id}", headers: {"authorization" => @auth_token})
        JSON.parse(response.body)
    end
end
```

```ruby

require 'rubygems'
require 'httparty'
require 'json'
require_relative './roadmap'

class Kele
    include HTTParty
    include Roadmap
```

Once again I tested in IRB to make sure that I was able to retrieve roadmaps and associated sections and checkpoints. 

### User Story 5: Messaging

Implementing the messaging functionality for the API meant allowing users to be able to retrieve messages as well as responding to existing messages and creating new messages on new threads.

I created two new methods, #getMessages and # createMessage, to handle all messaging needs. Retrieving existing messages was simple enough, but creating a new one would require me to post a response using the #body method provided by HTTParty and the headers option. 

The #createMessage method had to take arguments for email sender, recipient, subject, and body. I also wanted to be sure to return a response indicating success if the response posted. 

```ruby
    def create_message(sender, recipient_id, subject, stripped_text)
        response = self.class.post("https://www.bloc.io/api/v1/messages", 
            body: {
                "sender": sender,
                "recipient_id": recipient_id,
                "subject": subject,
                "stripped-text": stripped_text
            },
            headers: {"authorization" => @auth_token})
        if response.success? 
            puts "message sent!"
        end
    end
```

### User Story 6: Submitting Checkpoints

Finally, I wanted to add the ability for users to submit completed checkpoints. I added a #createSubmission method to my `roadmap` module. The method would take arguments for checkpoint Id, assignment branch, git commit link, and any additional comments. 

```ruby
    def create_submission(checkpoint_id, assignment_branch, assignment_commit_link, comment, enrollment_id)
        response = self.class.post("https://www.bloc.io/api/v1/checkpoint_submissions",
            body: {
                "assignment_branch": assignment_branch,
                "assignment_commit_link": assignment_commit_link,
                "checkpoint_id": checkpoint_id,
                "comment": comment,
                "enrollment_id": enrollment_id,
            },
            headers: {"authorization" => @auth_token})
        JSON.parse(response.body)
    end
```

## Results

I was pleased with what I came up with for my first Ruby gem. There is some cleanup and refactoring I would like to do, like storing the stub URL in a reusable variable, or returning the necessary argument information within some of the methods that called for it. But I am happy with the size and speed of the gem, and plan to include it in some future projects, like a To-Do list. 

If you want to review any of the code written for the project you can take a look at the [GitHub Repo](https://github.com/hamdans1/Kele/tree/user-story-7).

## Conclusion

There were challenges here because I haven't worked with an API client before, but after getting through the first user story the rest of the project flowed pretty easily. The only other hiccup I faced was implementing the messaging features, as I hadn't yet posted a JSON package back, and getting the syntax correct took me a minute. 

Thank you for reading up on my first Ruby gem. I look forward to challenging myself down the road with more complex API clients and integrating those gems into more complex projects. 