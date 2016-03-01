---
layout: post
title: rails + angular + bower
date:   2016-02-08 15:29:23 -0500
published: true
categories:
---
#Rails Angular Talk

## Objectives
- Why Rails?
- How to implement Rails and Bower

##Why Rails?
A couple of key features of Rails. 

- Rails API is a great lightweight, powerful server-side solution. You can add your front-end framework on top of it.

- It’s very structured and has a set of standards that the Rails community has agreed upon as best practice. So if another Rails dev is checking out your app, it’s easier to get up and running.
- allows you to scaffold apps very quickly so it's great for Startups, not too painful to roll out a prototype
  - AirBnb, Hulu, Twitter, Blue Apron, Plated, Daily Burn, GA website, Robinhood FAQ
- Auto-magical- can be good or bad
  - GOOD - rapid prototyping
  - BAD - Don’t always know what’s going on under the hood. As long as you play in their sandbox you’re ok   

#### Rails API
You can use rails for it's rails API only. It will scaffold out a project with no views. You can host Rails in one place and your front end in another.


## Rails App Instructions
Credit goes to this the [Angular-Rails](http://angular-rails.com/) tutorial. It was my reference to get Bower up and running.

- `rails new books -T --database=postgresql`

> This command will create a new rails project + directory called books. It will also skip the test unit and use the `gem pg` for our database. By default Rails uses SQLite

- `cd books && bin/rake db:create`

> We'll `cd` into the project directory and create our database

- At this point, we can start our Rails server with `rails server` and checkout the welcome page at [localhost:3000](cd books && bin/rake db:create)

## Add and configure Angular + Bootstrap using Bower
Bower was created by Twitter specifically to manage front-end assets. We'll use it to set up our front-end packages and dependencies.

- Add `gem 'bower-rails'` to the `Gemfile`
- Next, `touch Bowerfile` to create a `Bowerfile` in the root of the project and add:

~~~ruby
# This will add Angular and Bootstrap-Sass
asset 'angular'
asset 'bootstrap-sass-official'
~~~

- Run `rake bower:install`
  
> Bower installs dependencies in `vendor/assets/bower_components`. The Rails [asset pipeline](https://launchschool.com/blog/rails-asset-pipeline-best-practices) dictates that 3rd party code lives in the `vendor` directory.

- Since we're adding stuff outside of the normal flow of the asset pipeine we'll need to add it to `config/application.rb`.

~~~ruby
# Paste this code inside of
# class Application < Rails::Application
config.assets.paths << Rails.root.join("vendor","assets","bower_components")
config.assets.paths << Rails.root.join("vendor","assets","bower_components","bootstrap-sass-official","assets","fonts")
config.assets.precompile << %r(.*.(?:eot|svg|ttf|woff|woff2)$)
~~~


- Let's add the Angular require paths to `app/assets/javascripts/application.js`. We'll also remove turbolinks since it can hinder performance for front-end frameworks. Your requires should look like so:

~~~ruby
//= require jquery
//= require jquery_ujs
//= require angular/angular
//= require_tree .
~~~
 - Let's import our bootstrap stylesheets in `app/assets/stylesheets/application.css.scss` beneath all of the comments. Also, be sure to add the `.scss` extension to the file name:

~~~scss
@import "bootstrap-sass-official/assets/stylesheets/bootstrap-sprockets";
@import "bootstrap-sass-official/assets/stylesheets/bootstrap";
~~~
## MVC to serve our Angular app

- Add `root 'home#index'` to `routes.rb`. This will be the main route that will serve our Angular app.
- Let's create a controller for that route. Create `app/assets/controllers/home_controller.rb` and add:

~~~ruby
class HomeController < ApplicationController
  def index
  end
end
~~~

> We should be able to run `rails server` and see a Rails welcome page rendered.

### Instantiate our Angular App

- For this demo we're gonna use a Book Model so we'll call this our bookApp. Let's create `app/assets/javascripts/app.js` and add code to instantiate our Angular app:

~~~javascript
angular.module('bookApp',[
])
 .controller('BooksController', BooksController);

  function BooksController() {
    var vm = this;
    vm.names = ["Marc", "Maren", "Diesel"]
  }
~~~
> We're using `var vm` here for view model. I've also included some dummy data to make sure our view is wired up correctly.

### Create a view

- Create `app/assets/views/home/index.html.erb` and add:

~~~html
<div class="container-fluid" ng-app="bookApp">
  <div class="panel panel-success">
    <div class="panel-heading">
      <h1 ng-if="name">Hello, {{name}}</h1>
    </div>
    <div class="panel-body">
      <form class="form-inline">
        <div class="form-group">
          <input class="form-control" type="text" placeholder="Enter your name" autofocus ng-model="name">
        </div>
      </form>
    </div>
  </div>

  <div class="panel panel-success" ng-controller="BooksController as ctrl">
    <div class="panel-heading">
      <ul ng-repeat="book in ctrl.names">
        Hello, {{book}}
      </ul>
    </div>
  </div>
</div>
~~~
> You could also add `ng-app` to the `body` tag in `app/assets/views/layouts/application.html.erb`. This would make our entire app belong to `booksApp`.

We can start up our `rails server` and go to `localhost:3000` to make sure that Angular is working.

####Create the Book model

- `rails g model Book title author`
> We'll create a model for Book with title and author fields (both Strings,  by default)

- `rake db:migrate`

####Let's add some seed data for our Book model

- Let's add some seed data to `db/seed.rb` to populate our database:

~~~ruby
Book.destroy_all
Book.create([
  {title: "50 Shades", author: "Schmitty"},
  {title: "Lawyer Books", author: "Grisham"}
  ])
~~~
- `rake db:migrate db:seed`

##Set up our REST-ful api endpoint to serve the JSON

- We're gonna use [jbuilder](https://github.com/rails/jbuilder) to serve our books API data. Let's create `app/assets/views/books/index.json.jbuilder` and add:

~~~ruby
json.array!(@books) do |book|
  json.extract! book, :id, :title, :author
end
~~~

> We're creating a view folder for books, though we aren't going to create an actual view. We're serving JSON only.

- Eventually, we're gonna use `$http.get` in our Angular controller to grab our data. Create a file in `app/assets/controllers/books_controller.rb`. Then, let's make all the books available via JSON in our `BooksController` index action:

~~~ruby
class BooksController < ApplicationController  
  def index
    @books = Book.all
  end
end
~~~
- Add a route to our `app/config/routes.rb` file under our `root` route.

~~~ruby
 get '/books' => 'books#index'
~~~
> A route consists of the HTTP verb, the URL and the controller/action.

- Now we can start our server `rails server` and go to `http://localhost:3000/books.json` to see that Rails is serving our data as JSON.

## Grab the data using Angular

- Let's add some code to grab all the books from our endpoint in `app/assets/javascripts/app.js`

~~~javascript
angular.module('booksApp',[
])
 .controller('BooksController', BooksController);

  function BooksController($http) {
    var vm = this;
    vm.names = ["Marc", "Maren", "Diesel"]
    vm.books = getBooks().success(function(data){
      vm.books = data;
    });

    function getBooks(){
      return $http.get('http://localhost:3000/books.json');
    }
  }
~~~

> We'll add a getBooks() function to grab all the books from our REST-ful endpoint. We'll assign the results of that function to the variable `vm.books`. Also, don't forget to inject `$http` into `BooksController`.

## Update our view

- Let's go to our `app/assets/views/home/home.html.erb` page and make some small tweaks. First, we'll adjust our `ng-repeat` so it repeats our `vm.book` variable. We'll also add some code to render the `title` and `author` of each book. Here's the code for our `ul`:

~~~html
<ul ng-repeat="book in ctrl.books">
  Title: {{book.title}} </br> 
  Author: {{book.author}}
</ul>
~~~

##Conclusion
I really love Rails. It's great to pair the rapid prototyping ability of Rails with an Angular front-end!




