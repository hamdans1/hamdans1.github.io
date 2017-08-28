---
layout: post
title: Sammit
thumbnail-path: "/img/Sammit/Sammit-Frontpage.png"
short-description: Sammit is a Reddit replica I built to learn the fundamentals of back end programming and the Ruby on Rails framework
---

{:.center}
![]({{ site.basurl }}/img/Sammit/Sammit-Frontpage.png)

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
10. I want to implement a Favorite model to allow users to tag posts as and get an email after a comment is made on a favorite post.
11. I want users to have a viewable public profile. 

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

Like Reddit, I wanted to implement some sort of voting structure so that I could sort posts according to their popularity ranking. I had to create a Vote model to handle vote data and associate that with User and Post models. I wanted to add UI elements that allow users to vote, but limit those votes to only once per post. And I wanted to sort posts in the order of vote totals, but develop a basic time-decay algorithm to keep fresh posts on top.

First I generated the model using ` $ rails g model Vote value:integer user:references:index post:references:index`.

Then I updated my three relevant models to include hasMany and belongsTo associations between the three. Once I had done that I could implement #upVotes, #downVotes, and #points methods in my Post model. These were relatively simple methods using `where(value:).count` to assign votes, and `.sum(:value)` to call points. 

```ruby
    def up_votes
        votes.where(value:1).count
    end
    
    def down_votes
        votes.where(value:-1).count
    end
    
    def points
        votes.sum(:value)
    end
```

My next step was to generate a VotesController and create a partial view that I could render on my posts show view. The partial linked to POST routes for upVote and downVote and I used a basic bootstrap glyphicon for graphics. I had to adapt my resource routing to this include these vote routes without adding unnecessary additional routes:

```ruby
Rails.application.routes.draw do
  resources :topics do
    resources :posts, except: [:index]
  end
  
  
  resources :posts, only: [] do
    resources :comments, only: [:create, :destroy]
    
    post '/up-vote' => 'votes#up_vote', as: :up_vote
    post '/down-vote' => 'votes#down_vote', as: :down_vote
  end
  
  resources :users, only: [:new, :create, :show]
  
  resources :sessions, only: [:new, :create, :destroy]

  get 'about' => 'welcome#about'

  root 'welcome#index'
  
end
```

I included two methods for upVote and downVote in my votes controller that both called on a private method for updateVote. 

```ruby
class VotesController < ApplicationController
    before_action :require_sign_in
    
    def up_vote
        update_vote(1)
        redirect_to :back
    end
    
    def down_vote
        update_vote(-1)
        redirect_to :back
    end
    
    private
    
    def update_vote(new_value)
        @post = Post.find(params[:post_id])
        @vote = @post.votes.where(user_id: current_user.id).first
        
        if @vote
            @vote.update_attribute(:value, new_value)
        else
            @vote = current_user.votes.create(value: new_value, post: @post)
        end
    end
    
end
```
The last piece to the puzzle was implementing the ranking system. My first step was to add a rank attribute to my Post table and migrate. I added an afterSave callback method to my Vote model and a private updatePost method. 

```ruby
class Vote < ActiveRecord::Base
  belongs_to :user
  belongs_to :post
  after_save :update_post
  
  validates :value, inclusion: { in: [-1,1], message: "%{value} is not a valid vote."}, presence: true
  
  private
  
  def update_post
    post.update_rank
  end
  
end
```

My call on post.updateRank was wishful coding so I went back to my Post model and implemented updateRank and added a default_scope to order by rank. Once that was done I only needed to update my seeds.rb file to include voting and I was done implementing voting.

```ruby
    default_scope {order('rank DESC')}
    scope :visible_to, -> (user) {user ? all : joins(:topic).where('topics.public' => true) }
    ...
    def update_rank
        age_in_days = (created_at - Time.new(1970,1,1)) / 1.day.seconds
        new_rank = points + age_in_days
        update_attribute(:rank, new_rank)
    end
    ...
end
```
![Sammit Topic Posts view](/img/Sammit/Sammit-TopicsPosts.png)

### User Story 10: Favoriting and ActiveMailer

In addition to the vote system, I wanted users to be able to tag a post as a favorite and be notified when a post receives a new comment. I need to generate a model that would track which posts a user has favorited. I need to add a 'favorite' button on posts that would allow users to flag a favorite. Finally I wanted to add a notification feature that sent an email to users when one of their favorited posts got a new comment. 

Generating the model was simple enough. My model wouldn't need any unique attributes as it would only need to references users and posts. The model would also require a 'belongs_to' association with both users and posts. 

I then added a #favoriteFor method on my User model that would determine if a post had been favorited by a user.

```ruby
    def favorite_for(post)
        favorites.where(post_id: post.id).first
    end
```

My next step was to generate a FavoritesController and edit my route resources. I nested my favorites routes in my posts, similarly to comments. Once that was done, I created a partial I could add to my post show view and I moved on to develop the methods in my controller.

```ruby
<% if favorite = current_user.favorite_for(post) %>
    <%= link_to [post,favorite], class: 'btn btn-danger', method: :delete do %>
        <i class= "glyphicon glyphicon-star-empty"> </i>&nbsp; Unfavorite
    <% end %>
<% else %>
    <%= link_to [post, Favorite.new], class: 'btn btn-primary', method: :post do %>
    <i class="glyphicon glyphicon-star"> </i>&nbsp; Favorite
    <% end %>
<% end %>
```

I wanted Sammit to send an email when a favorited post received a new comment, to do that I installed SendGrid via heroku.  After attaching my Heroku login to SendGrid I edited my enivornment configs and added a setup_mail initializer. To protect sensitive data in my config files from being accessible on GitHub, I decided to use the Figaro gem. 

The final step would be implementing a Favorite mailer. I used `$ rails generate mailer Favorite Mailer` to create my mailer. I set my personal email as the default email address and added two methods: newComment would send when a favorited post received a new comment, and newPost would send when a favorited user posted a new post. I also made sure when creating my corresponding favorite mailer views to use both html and plain text to support all email clients. 

```ruby
class FavoriteMailer < ApplicationMailer
    default from: "samihamdan00@gmail.com"
    
    def new_comment(user, post, comment)
        
        headers["Message-ID"] = "<comments/#{comment.id}@pure-cliffs-33222.herokuapp.com>"
        headers["In-Reply-To"] = "<post/#{post.id}@pure-cliffs-33222.herokuapp.com>"
        headers["References"] = "<post/#{post.id}@pure-cliffs-33222.herokuapp.com>"
        
        @user = user
        @post = post
        @comment = comment
        
        mail(to: user.email, subject: "New comment on #{post.title}")
    end
    
    def new_post(post)
        
        headers["Message-ID"] = "<posts/#{post.id}@pure-cliffs-33222.herokuapp.com"
        headers["In-Reply-To"] = "<post/#{post.id}@pure-cliffs-33222.herokuapp.com>"
        headers["References"] = "<post/#{post.id}@pure-cliffs-33222.herokuapp.com>"
        
        @post = post
        
        mail(to: post.user.email, subject: "You're following #{post.title}")
    end
    
end
```
Finally I added a private callback to my Comment model to handle repeat functions and I added a favoritePost function to my Post model to handle favorite creation. 

```ruby
    def favorite_for(post)
        favorites.where(post_id: post.id).first
    end
```
```ruby
    def favorite_post
        Favorite.create(post: self, user: self.user)
        FavoriteMailer.new_post(self).deliver_now
    end
```
![Sammit Post view with Favorites](/img/Sammit/Sammit-My Post.png)

### User Story 11: User Profiles
The last piece I wanted to implement was a user profile. I wanted users to be able to publicly share their Sammit contributions by displaying basic user info and a list of posts and comments. I needed to develop a #show method on my UsersController and build out the corresponding view. I also wanted to implement gravatar functionality so that users would have a picture id on their profile and around their login info. 

I used my #show method to assign instance variables for @user and @posts that I could call on my show view. 
```ruby
    def show
        @user = User.find(params[:id])
        @posts = @user.posts.visible_to(current_user)
    end
```
I implemented gravatar by adding a #avatar_url method on my User model. Following the Gravatar how-to, I created the md5 hash and assigned it to a new variable, and then I compiled the URL so that it coule be called in the URL with a size parameter.
```ruby
    def avatar_url(size)
        gravatar_id = Digest::MD5::hexdigest(self.email).downcase
        "http://gravatar.com/avatar/#{gravatar_id}.png?s=#{size}"
    end
```
Finally, I updated my show view so that it would display the users gravatar, their post history, and their comment history. 
```ruby
<div class="row">
    <div class="col-md-8">
        <div class="media">
            <br />
            <% avatar_url = @user.avatar_url(128) %>
            <% if avatar_url %>
                <div class="media-left">
                    <%= image_tag avatar_url, class: 'media-object' %>
                </div>
            <% end %>
            <div class="media-body">
                <h2 class="media-heading"><%= @user.name %></h2>
                <small>
                    <%= pluralize(@user.posts.count, 'post') %>,
                    <%= pluralize(@user.comments.count, 'comment') %>
                </small>
            </div>
        </div>
    </div>
</div>

<h2>Posts</h2>
<%= render @user.posts %>

<h2>Comments</h2>
<%= render @user.comments %>
```
![Sammit Profile view](/img/Sammit/Sammit-Profile Page.png)

## Results

An important note here, throughout this process I was first developing each step using TDD and rspec. I chose not to include these steps here in my user stories because I didn't want the length to get too carried away. All of my spec files and the rest of Sammit can be found on the public Github repo. 
[Github Repo](https://github.com/hamdans1/bloccit)

I was very pleased with what I came up with for Sammit. I achieved all of my development goals from when I started out. In the future I'd like to return to update some of the wireframes and stylesheets for the application to see what I could make it look like. I might also return to add some different features like a TipJar(reddit gold) or a premium role to go with standard and admin that could have some alternative access. 

## Conclusion

Sammit challenged me in a wide range of ways. This was my first Rails application so it was my first time playing within the framework. Beyond that, I was pushing myself to use a large set of new gems and technologies that I had never worked with before. Unlike my experience with BlocChat where I was spending a decent bit of time focusing on the front-end styling, I almost completely disregarded that here in favor of focusing on the tech used. I have no doubts that my preference is to think about the functionality and the back end data interworkings instead of spending time on the HTML and scripting. 

If you don't want to fork the repo and take a look under the hood, you can poke around Sammit on via heroku using this link:

[Heroku Link](https://pure-cliffs-33222.herokuapp.com)

Thanks so much for reading and I hoped you learned something from my work here!