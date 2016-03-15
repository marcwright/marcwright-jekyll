---
layout: post
title: rails + angular + bower
date:   2016-03-15 15:29:23 -0500
published: true
categories:
---

#### tl;dr

Rails provides an excellent backend for your Javascript front-end framework.

### Why Rails?
Rails API is a great lightweight, powerful server-side solution. You can add your front-end framework on top of it.

The Rails motto is "convention over configuration". Rails provides a set of standards that the community has agreed upon as best practice. So if another Rails dev is checking out your app, it’s easier to get up and running.


Rails allows you to scaffold apps very quickly so it's great for startups. It's relatively painless to roll out a prototype.

- e.g.- AirBnb, Hulu, Twitter, Blue Apron, Plated, Daily Burn, GA website, Robinhood FAQ

Rails is "auto-magical", which can be good or bad.

- GOOD - rapid prototyping
- BAD - Sometimes you don’t always know what’s going on under the hood, but as long as you play in their sandbox you’re ok.

### Rails API
You can also roll out a Rails app using the server + API only. It will scaffold out a project with no views. This would allow you to host Rails on one server and your front end on another. This makes for an extremely lightweight Rails app since it takes out any middlewares and extra cruft that the views would normally use.

For a clunky analogy... the Rails API would be like a hard-top convertible in that we can completely remove the views. A soft-top convertible is like scaffolding a  standard Rails app but ignoring the views (even though the functionality is always available).

## Rails App Code Along
Completed code for this article can be found [here](https://github.com/marcwright/books-angular-rails-demo). Credit goes to this excellent [Angular-Rails](http://angular-rails.com/) tutorial. It was my reference for the initial Bower implementation.

1) `rails new books -T --database=postgresql`

  *This command will create a new rails project + directory called books. `-T` tells Rails not to scaffold a test suite. `--database=postgresql` will automatically bundle the `gem pg` so we may use a PostgreSQL database. By default Rails uses SQLite.*

2) `cd books && bin/rake db:create`

  *This will `cd` into the project directory and create our database*

3) At this point, we can start our Rails server with `rails server` and checkout the welcome page at [http://127.0.0.1:3000](http://127.0.0.1:3000)

## Add and configure Angular + Bootstrap using Bower
Bower was created by Twitter specifically to manage front-end assets. We'll use it to set up our front-end packages and dependencies.

4) Add `gem 'bower-rails'` to the `Gemfile`

5) Next, on the command line, run `touch Bowerfile`. This will create a `Bowerfile` in the root of the project. Then add the following to that file:

~~~ruby
# This will add Angular and Bootstrap-Sass
asset 'angular'
asset 'bootstrap-sass-official'
~~~

6) Run `rake bower:install`

  *Bower installs dependencies in `vendor/assets/bower_components`. The Rails [asset pipeline](https://launchschool.com/blog/rails-asset-pipeline-best-practices) dictates that 3rd party code lives in the `vendor` directory.*

7) Since we're adding stuff outside of the normal flow of the asset pipeine we'll need to add it to `config/application.rb`.

~~~ruby
# Paste this code inside of
# class Application < Rails::Application
config.assets.paths << Rails.root.join("vendor","assets","bower_components","bootstrap-sass-official","assets","fonts")
config.assets.precompile << %r(.*.(?:eot|svg|ttf|woff|woff2)$)
~~~


8) Let's add the Angular require paths to `app/assets/javascripts/application.js`. [Turbolinks](http://guides.rubyonrails.org/working_with_javascript_in_rails.html) is a gem that uses Ajax to speed up page rendering in most applications. We'll remove turbolinks since it can hinder performance for front-end frameworks. Your requires should look like so:

~~~ruby
//= require jquery
//= require jquery_ujs
//= require angular/angular
//= require_tree .
~~~


9) Let's import our bootstrap stylesheets in `app/assets/stylesheets/application.css.scss` beneath all of the comments. Also, be sure to add the `.scss` extension to the file name:

~~~scss
@import "bootstrap-sass-official/assets/stylesheets/bootstrap-sprockets";
@import "bootstrap-sass-official/assets/stylesheets/bootstrap";
~~~

## Add a doorway route for our Angular app

10) Add `root 'home#index'` to `routes.rb`. This will be the main route that will serve as the doorway to our Angular app.

11) Let's create a controller for that route. Create `app/assets/controllers/home_controller.rb` and add:

~~~ruby
class HomeController < ApplicationController
  def index
  end
end
~~~



## Instantiate our Angular App

12) For this demo we're gonna use a Book Model so we'll call this our bookApp. Let's create `app/assets/javascripts/app.js` and add code to instantiate our Angular app:

~~~javascript
angular.module('bookApp',[])
 .controller('BooksController', BooksController);

  function BooksController() {
    var vm = this;
    vm.names = ["Marc", "Maren", "Diesel"]
  }
~~~

*We're using `var vm` here for view model. I've also included some dummy data to make sure our view is wired up correctly.*

## Create a view

13) Create `app/assets/views/home/index.html.erb` and add:

~~~html
<div class="container-fluid" ng-app="bookApp">
  <div class="panel panel-success">
    <div class="panel-heading">
      {% raw %}
      <h1 ng-if="name">Hello, {{name}}</h1>
      {% endraw %}
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
        {% raw %}
        Hello, {{book}}
        {% endraw %}
      </ul>
    </div>
  </div>
</div>
~~~

  *You could also add `ng-app` to the `body` tag in `app/assets/views/layouts/application.html.erb`. Then our entire app would belong to `booksApp`.*

14) We can start up our `rails server` and go to `localhost:3000` to make sure that Angular is working.

## Create the Book model + add some seed data

15) `rails g model Book title author`

  *We'll create a model for Book with title and author fields (both Strings,  by default)*

16) Let's add some seed data to `db/seed.rb` to populate our database:

~~~ruby
Book.destroy_all
Book.create([
  {title: "50 Shades", author: "Schmitty"},
  {title: "Lawyer Books", author: "Grisham"}
])
~~~

17) From the command line run `rake db:migrate db:seed`

  *This will create our schema file and seed our database*

## Set up our REST-ful api endpoint to serve the JSON

18) Add a route to our `app/config/routes.rb` file under our `root` route.

~~~ruby
 get '/books' => 'books#index'
~~~

  *A route consists of the HTTP verb, the URL and the controller/action.*

19) Eventually, we're gonna use `$http.get` in our Angular controller to grab our data. Create a file in `app/assets/controllers/books_controller.rb`. Then, let's make all the books available via JSON in our `BooksController` index action:

~~~ruby
class BooksController < ApplicationController
  def index
    @books = Book.all
  end
end
~~~

20) We're gonna use [jbuilder](https://github.com/rails/jbuilder) to serve our books API data. Let's create `app/assets/views/books/index.json.jbuilder` and add:

~~~ruby
json.array!(@books) do |book|
  json.extract! book, :id, :title, :author
end
~~~

21) Now we can start our server `rails server` and go to `http://localhost:3000/books.json` to see that Rails is serving our data as JSON.

## Grab the data using Angular

23) Let's add some code to grab all the books from our endpoint in `app/assets/javascripts/app.js`

~~~javascript
angular.module('booksApp',[])
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

  *We'll add a getBooks() function to grab all the books from our REST-ful endpoint. We'll assign the results of that function to the variable `vm.books`. Also, don't forget to inject `$http` into `BooksController`.*

## Update our view

24) Let's go to our `app/assets/views/home/home.html.erb` page and make some small tweaks. First, we'll adjust our `ng-repeat` so it repeats our `vm.book` variable. We'll also add some code to render the `title` and `author` of each book. Here's the code for our `ul`:

~~~html
<ul ng-repeat="book in ctrl.books">
  {% raw %}
  Title: {{book.title}} </br> 
  Author: {{book.author}}
  {% endraw %}
</ul>
~~~

## Conclusion
I really love Rails. It's great to pair the rapid prototyping ability of Rails with an Angular front-end!




