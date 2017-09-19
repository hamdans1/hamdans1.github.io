---
layout: post
title: Get-it-Done
thumbnail-path: "img/get-it-done/SerializerReturn.png"
short-description: An externally usable API that lets users modify to-do list from the command line. 
---

{:.center}
![]({{site.baseurl}}/img/get-it-done/SerializerReturn.png)

## What is Get-it-Done?

When I set out to work on Get-it-Done I knew that I was going to work on a to-do list web application. I also knew I needed more API practice. I had already worked integrating the Bloc API into a Ruby gem, but I wanted to build my own externally usable API.  Something that could be useful in the future to me would be an API that would allow users to modify their to-do list from the command line.

## Technologies Used

* Rails
* Git
* ActiveModel
* JSON
* Ruby

## User Stories

I needed to build an API that would be able to return data, as well as CRUD functionality and authentication. 

1. I wanted to create Rails Serializers that would return JSON of users, lists, and items. 
2. I wanted users to be able to authenticate themselves from the command line.
3. I wanted users to be able to create new users, lists, and items from the command line. 
4. I wanted users to be able to delete themselves and ther lists from the command line.
5. I wanted users to be able to update lists and items form the command line.

### User Story 1: Serialization

My first step was making sure my application would be able to return JSON representations of users, lists, and items after I created them. So first I had to generate models for each, and create the necessary data relationship between them.

After that, I would need the necessary serializers to return the formatted responses that could be read, generated, and parsed. I bundle installed the Active Model Serializer gem, and used the Rails generator to create a serializer. 

Once I generated the serializer I added whatever attribues I wanted serialized, in this case I didn't have a need to create any faux-attributes or variations and I just wanted to return the base attributes associated with each model. 

```ruby
class UserSerializer < ActiveModel::Serializer
  attributes :id, :email, :created_at

end
```

I should note here that I created the serializers knowing that I would want to build out an API controller to allow authentication and access from the command line. This is probably a seriazliers best use. 

Now to make sure the serializer was working I created myself as a User and ran the following from my Rails console: 
`puts JSON.pretty_generate(UserSerializer.new(User.first).as_json)`

And I got this in return: 
![User Serializer Response](/img/get-it-done/User Serializer Test.png)

### User Story 2: Authentication

