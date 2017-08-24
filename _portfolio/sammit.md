---
layout: post
title: Sammit
thumbnail-path: 
short-description: Sammit is a Reddit replica I built to learn the fundamentals of back end programming and the Ruby on Rails framework
---

{:.center}
![]({{ site.basurl }}/img/sammit/sammit-frontpage.png)

## What is Sammit?

Sammit is a replica of the popular internet message board site, Reddit. This was my first Rails project in the Bloc program. The primary goal here was to demonstrate all of the lessons I had learned so far using Ruby, primarily regarding data relationships and the MVC framework.

## Technologies Used

* Ruby
* HTML/CSS/Javascript
* Ruby on Rails
* Git
* SQL
* Object Relational Mapping
* Heroku
* RSpec
* Cloud9
* ActiveMailer
* SendGrid
* MVC framework

## User Stories

Before I could get started with building out the application, I had to come up with a list of User and Developer stories. This project demanded more out of me in both planning and development then any other project I've worked on. Partly because I was new to the technologies and partly becaues I wanted to include so much in the application so I could gain some experience across different techs. Most importantly, I wanted to use Test-Driven Development in coming up with all of these solutions so I could start to develop good habits.

1. I want users to be able to create posts.
2. I want users to be able to write comments on posts, and for comments to belong to posts.
3. I want to seed data on my development database to make sure my views are rendering correctly.
4. I want users to be able to perform CRUD operations on our models.
5. I want users to be able to sign up, sign in, and sign out using authenticated usernames and passwords.
6. I want to implement a session controller to determine when and what user is signed in. 
7. I want to create a data association between users, posts, and comments.
8. I want to create authorized roles within the application, and allow for admins to create topics.
9. I want to implement a Vote model to allow users to upvote and downvote different posts, topics, and comments. 
10. I want to implement a Favorite model and functionality to allow users to favorite certain posts or comments.
11. I want users to receieve an email when there is a comment or change on a favorited post. 
12. I want users to have a viewable public profile. 

### User Story 1: Post Model

My first step was to create a Post model. Easy enough, I used the rails generator and ran:
`$ rails generate model Post title:string body:text`
This gave me a post.rb, post_spec.rb, and a db file for creating posts. Running `rake db:migrate` gave me a schema file and we are off to the races. 

### User Story 2: Comment Model and Post relationship

Setting up my Comment model was similar to setting up a Post model, but I also wanted to add a reference to the Post model within the Comment table. 
`$ rails generate model Comment body:text post:references`

Within comment.rb I added a line to associate comments to posts:
`belongs_to :post`
And within post.rb I added a line to do the same:
`has_many :comments`

Again I ran `$ rake db:migrate` here to add comments to my database.

### User Story 3: Seeding Data

So now I wanted to see if my models are working properly and prepare my application for developing CRUD controllers. I know that I need to seed data so I can view the database inside the Rails console. 

First, I created a file `lib/random_data.rb`. Within this file I created a module that held three methods for creating fake data.

```ruby
module RandomData
    def self.random_name
        first_name = random_word.capitalize
        last_name = random_word.capitalize
        "#{first_name} #{last_name}"
    end
    
    def self.random_email
        "#{random_word}@#{random_word}.#{random_word}"
    end
    
    def self.random_paragraph
        sentences = []
        rand(4..6).times do
            sentences << random_sentence
        end
        
        sentences.join(" ")
    end
    
    def self.random_sentence
        strings = []
        rand(3..8).times do
            strings << random_word
        end
        
        sentence = strings.join(" ")
        sentence.capitalize << "."
    end
    
    def self.random_word
        letters = ('a'..'z').to_a
        letters.shuffle!
        letters[0,rand(3..8)].join
    end
end
```
Then I went to `db/seeds.rb` and required that module. From there I just ran a few simple loops to create Posts and Comments, and a message that would confirm in the console when the seeding was completed.
```ruby
50.times do
    post = Post.create!(
        title: RandomData.random_sentence,
        body: RandomData.random_paragraph
        )
100.times do
    Comment.create!(
        post: posts.sample,
        body: RandomData.random_paragraph
    )
end
puts "Seed finished"
puts "#{Post.count} posts created"
puts "#{Comment.count} comments created"
```
After that I made sure to make my random_data file accessible to the rest of my application by editing my config/application.rb file and ran `$ rake db:reset`.

### User Story 4: CRUD Ops

To implement CRUD (Create, Read, Update, Destroy) operations, I would need to generate resources for Post and Comment. First I generated a Post controller using: 
`$ rails generate controller posts index show new edit`
Then I went into my config/routes.rb file and refactored my post routes to:
`resources :posts`
After that I added methods to my PostsController for #index, #create, #new, and #edit. With those methods completed, I edited the views that corresponded to the controller actions. 

```ruby
 def show
    @post = Post.find(params[:id])
  end

  def new
    @post = Post.new
  end

  def edit
    @post = Post.find(params[:id])
  end
  
  def create
    @post = Post.new
    @post.user = current_user
    
    if @post.save
      flash[:notice] = "Post was saved"
      redirect_to [@topic, @post]
    else
      flash.now[:alert] = "There was an error saving the post. Please try again."
      render :new
    end
  end
  ```
  After completing ALL of my controller actions and views, I decided it was time to start deploying my application to Heroku so I could see it in action outside of the dev environment on cloud9.

### User Story 5: Users

I wanted to develop a custom authentication system rather then use Devise or OmniAuth or any of the other authentication gems available. I chose the custom path because it would give me a better understanding of the relationships and better control over the User resources.

First I generated a user model with name, email, and password_digest attributes and ran db:migrate. I bundle installed 'BCrypt' to handle my password encryption needs, and added a #hasSecurePassword method to my User model. 

Once that was set up I created a UsersController with CRUD ops, and created resources on my routes.rb. The last piece for my users was setting up views so that users could sign up, sign in, and sign out. I added those link options my application layout, as I wanted them to be ever present. 

```ruby
<div class="container">
  <ul class="nav nav-tabs">
    <li> <%= link_to "Sammit", root_path %></li>
    <li> <%= link_to "Topics", topics_path %></li>
    <li> <%= link_to "About", about_path %></li>
    <div class="pull-right user-info">
      <% if current_user %>
        <%= image_tag current_user.avatar_url(48), class: "gravatar" %>
        <div class="pull-right">
          <%= link_to current_user.name, user_path(current_user) %> <br/> <%= link_to "Sign Out", session_path(current_user), method: :delete %>
        </div>
      <% else %>
        <%= link_to "Sign In", new_session_path %> or
        <%= link_to "Sign Up", new_user_path %>
      <% end %>
    </div>
  </ul>
  ```
![Sammit Frontpage](/img/Sammit/Sammit-Frontpage.png)

### User Story 6: Sessions Controller

  I want to require my user data to persist while they are 'signed in'. To do that I would need a sessions controller that would store data in a user client's cookies. I also wanted to create helper methods like #current_user that would be useful for things like authorization down the road. 

  I generated a sessions controller with create and destroy methods, and I created a sessions_helper with several other methods that would be necessary. 

  ```ruby
  class SessionsController < ApplicationController
    
    def create
        user = User.find_by(email: params[:session][:email].downcase)
        
        if user && user.authenticate(params[:session][:password])
            create_session(user)
            flash[:notice] = "Welcome, #{user.name}"
            redirect_to root_path
        else
            flash.now[:alert] = "Invalid email/password combination"
            render :new
        end
    end
    
    def destroy
        destroy_session(current_user)
        flash[:notice] = "You've been signed out, come back soon!"
        redirect_to root_path
    end
    
end
```
```ruby
module SessionsHelper
    
    def create_session(user)
        session[:user_id] = user.id
    end
    
    def destroy_session(user)
        session[:user_id] = nil
    end
    
    def current_user
        User.find_by(id: session[:user_id])
    end
    
end
```

### User Story 7: Data Association

I already have an association between Comments and Posts, but now I need to integrate users into the mix.  To do this I need to run a new migration that will add a Users column to my Posts table, and I need to declare those associations within the models. 

Running the migration was as simple as:
`$ rails g migration AddUserToPosts user_id:integer:index`
and then running `$ rake db:migrate`.

After that I updated the models to include the same associations I used for Posts and Comments. 

### User Story 8: Authorization and Topics

I wanted to sort my posts into topic groups, and I only wanted certain users to be able to create new topics. First I generated a Topic resource with model, controller, and views. I also wanted to create an association between Topics and Posts, as topics have many posts and posts belong to topics.

I again ran a migration to AddTopicToPosts after creating my initial Topic model with name, public?, and description attributes.

(I should add here, that when I initially created my Topic controller I didn't know I could develop table associations when I initially generated the model, I thought I had to go back with a separate migration.)

Creating topics would also require me to start nesting my routes, because I didn't want users to have access to posts outside of a topic. Here I updated routes.rb to include:

```ruby
  resources :topics do
    resources :posts, except: [:index]
  end
```

After creating my topics and editing my routes, I wanted to be sure to add an authorization mechanic to my users. This was as simple as adding `enum role: [:member, :admin]` to my User model.  This associated a role with a number to allow for easier reference. Then I ran a rails migration for AddRoleToUsers and I was set. 

![Topics index view](/img/Sammit/Sammit-Topics.png)

My last step here was to update my seeds.rb file to include my new associations, and to allow for the creation of topics.

### User Story 9: Upvotes and Downvotes

