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
```ruby
> puts JSON.pretty_generate(UserSerializer.new(User.first).as_json)
```

And I got this in return:
![User Serializer Response](/img/get-it-done/User Serializer Test.png)

### User Story 2: Authentication

I next wanted to authenticate users from the command line, so I needed a Users Controller. I created a base API controller from which my other controllers could inherit. To my API controller I added a private #authenticated? method:

```ruby
class ApiController < ApplicationController
    skip_before_action :verify_authencity_token

    private

    def authenticated?
        authenticate_or_request_with_http_basic{|email, password_digest| User.where(email: email, password_digest: password_digest).present?}
    end
end
```
I edited my routes to make sure that I had an API route that my users would run through, and that my API route had a separate namespace and would support JSON requests.

```ruby
  namespace :api, defaults: { format: :json } do
    resources :users do
    ...
```
Lastly, I got back to the business of my Users Controller, making sure it inherited from my API Controller and had an index method that would return a JSON representation of all users using the User Serializer.

```ruby
    def index
        users = User.all
        render json: users, each_serializer: UserSerializer
    end
```

From there I ran a curl request from my command line, while running the application from the c9 server.
`$ curl -u samihamdan00@gmail.com:password https://get-it-done-hamdans1.c9users.io/api/users`

![User Index Authentication](/img/get-it-done/Authentication User Index.png)


### User Story 3: Create from Command Line

Now that I had my API and Users controller set up, it was time to implement #create functionality with Users, Lists, and Items. My first step was to provide API routes for List and Item.

```ruby
  namespace :api, defaults: { format: :json } do
    resources :users do
      resources :lists
    end

    resources :lists, only: [] do
      resources :items, only: [:create, :update]
    end

    resources :items, only: [:destroy]
  end
```

Next I created controllers for both Lists and Items to match the routes I laid out. I implemented #create menthods for all three of my controllers. I also added a private #params method to be sure I was requiring the right information.

With my #create methods I wanted to be sure to render a JSON rep of the user, or return a proper error message.

```ruby
    def create
        user = User.new(user_params)
        if user.save
            render json: user
        else
            render json: {errors: user.errors.full_messages}, status: :unprocessable_entity
        end
    end
        private

    def user_params
        params.require(:user).permit(:email, :password_digest)
    end

end
```
Now I was ready to test from my command line. Again, I ran the application from the c9 server and headed over to my terminal to test.
`$ curl -u samihamdan00@gmail.com:password -d "user[email]=member@test.com" -d "user[password_digest]=password" http://get-it-done-hamdans1.c9users.io/api/users/`

![Create from CL](/img/get-it-done/Create from Command Line.png)

### User Story 4: Destroy

Of course if I want users to be able to create, I should also allow them to destroy. For now, I would only concern myself with destroying users and lists. Items would really be a part of updating lists, and we'll get to that in the last user story.

Implementing destroy was as simple as adding a #destroy method to my Users and Lists controllers.

```ruby
    def destroy
        begin
            user = User.find(params[:id])
            user.destroy
            render json: {}, status: :no_content
        rescue ActiveRecord::RecordNotFound
            render :json => {}, :status => :not_found
        end
    end
```
As you can see I returned HTTP 204 to let the user know the request successfuly went through and the content was destroyed. I also made sure to add a rescue statement in case of an error. I added a similar method to my Lists controller and I was good to go.

```ruby
    def destroy
        begin
            list = List.find(params[:id])
            list.destroy
            render json: {}, status: :no_content
        rescue ActiveRecord::RecordNotFound
            render :json => {}, :status => :not_found
        end
    end
```
Again I tested deletion from the command line to be sure my code was working, and I got a 204 on my console.

`$ curl -u samihamdan00@gmail.com:password -X DELETE https://get-it-done-hamdans1.c9users.io/api/users/3/`

### User Story 5: Updating Lists and Items

The last piece of the puzzle would be allowing users to update their lists and the items on those lists from the command line. I added an #update method to both controllers that would render the updated list or item when successful.

```ruby
    def update
        list = List.find(params[:id])
        if list.update(list_params)
            render json: list
        else
            render json: {errors: list.errors.full_messages}, status: :unprocessable_entity
        end
    end
```
```ruby
    def update
        item = Item.find(params[:id])
        if item.update(item_params)
            render json: item
        else
            render json: {errors: item.errors.full_messages}, status: :unprocessable_entity
        end
    end
```
I tested the update implementation from the command line to make sure everything was working well, and then I was done.

`curl -X PUT -u samihamdan00@gmail.com:password -d "list[private]=true" https://get-it-done-hamdans1.c9users.io/api/users/lists/1`

## Results

This project was a great success. It was my first foray into developing my own API and it was a lot easier than I anticipated. It encouraged me to start playing with other API, namely Facebook and Twitter, and working on integrating those into my own applications. I'm also working on a Rails application that is a self-deleting To Do list, and hope to be able to integrate this into that.

## Conclusion

I have to say, this was no great challenge. I think I nailed this down in the space of two days and was really satisfied with what I came up with. Thanks for taking the time to read through, and make sure to check back to see how I work this into my next Rails app, "What's Next".  You can find my work on my Github page here:
<br>
[github link](https://github.com/hamdans1/get-it-done)
