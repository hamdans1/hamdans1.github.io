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