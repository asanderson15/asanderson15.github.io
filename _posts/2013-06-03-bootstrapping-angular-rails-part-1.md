---
layout: post
title: Bootstrapping an AngularJS app in Rails 4.0 - Part 1
category: posts
summary: I was in the process of building a blog for myself as an exercise to try out Rails 4.0.  At the same time, I happened to be reading Rework, by Jason Fried and David Heinemeier Hansson.  One of my favorite pieces of advice they gave was to focus not only on the main task of what you are doing, but also to recognize the byproducts.  While I have a fair amount of experience with Rails and AngularJS, I am by no means an expert on either.  So no matter how simple I keep this project, I will inevitably be learning as I go.  It seemed like the perfect opportunity to develop not only my blog, but also some content. 
socialsummary: Part one of this tutorial covers creating a Rails 4.0 app, adding AngularJS, setting up structure, and implementing basic Rails and AngularJS controllers.
socialimage: http://asanderson15.github.io/images/AngularJS-Shield-medium.png
tags: AngularJS Rails Tutorial
year: 2013
month: 6
day: 3
published: true
comments: true
---

<br>

***The full GitHub repository for this tutorial is available [here](https://github.com/asanderson15/rails-angular-tutorial)***


### Introduction

I was in the process of building a blog for myself as an exercise to try out Rails 4.0.  At the same time, I happened to be reading [Rework](http://www.amazon.com/Rework-Jason-Fried/dp/0307463745), by Jason Fried and David Heinemeier Hansson.  One of my favorite pieces of advice they gave was to focus not only on the main task of what you are doing, but also to recognize the byproducts.  While I have a fair amount of experience with Rails and AngularJS, I am by no means an expert on either.  So no matter how simple I keep this project, I will inevitably be learning as I go.  It seemed like the perfect opportunity to develop not only my blog, but also some content. 

This series assumes a certain basic understanding of both Rails and AngularJS.  If you need an introduction to Rails, I recommend checking out the excellent [Ruby on Rails Tutorial](http://ruby.railstutorial.org/) by Michael Hartl.  For an intro to AngularJS, I recommend checking out the [homepage tutorials](http://angularjs.org/) and, to go a bit deeper, the excellent [egghead.io tutorial videos](http://egghead.io/) by John Lindquist.

I plan to focus these posts on: 

* Building a specific example from scratch, showing all my work along the way, and explaining what is going on at each step.<br><br>
* Demonstrating the ins and outs of using Rails on the back end together with AngularJS on the front end (which I believe to be an excellent pairing for many types of apps)

The scope of this exploration may expand as I go, but this is my current plan.  

So here we are.  Let's see where this takes us.<br><br>

### Creating your Rails 4.0 app

Let's start by creating a Rails 4.0 application.  (If you need help installing Ruby 2.0 or Rails 4.0 RC1, check out the ["Up and Running"](http://ruby.railstutorial.org/ruby-on-rails-tutorial-book?version=4.0#sec-up_and_running) section of Michael Hartl's Ruby on Rails Tutorial.)  

To create the app, go to the command line terminal and create a new application called `blog`.  Use the below flags to skip the default test framework and set the database to be PostgreSQL.

**Create new Rails blog application**

```bash
$ rails new blog --skip-test-unit --database=postgresql
      create
      create  README.rdoc
      create  Rakefile
      create  config.ru
      create  .gitignore
      create  Gemfile
      create  app
      create  app/assets/images/rails.png
      create  app/assets/javascripts/application.js
      create  app/assets/stylesheets/application.css
      create  app/controllers/application_controller.rb
      create  app/helpers/application_helper.rb
      create  app/views/layouts/application.html.erb
      create  app/mailers/.keep
      create  app/models/.keep
      create  app/controllers/concerns/.keep
      create  app/models/concerns/.keep
      create  bin
      create  bin/bundle
      create  bin/rails
      create  bin/rake
      create  config
      create  config/routes.rb
      create  config/application.rb
      create  config/environment.rb
      create  config/environments
      create  config/environments/development.rb
      create  config/environments/production.rb
      create  config/environments/test.rb
      create  config/initializers
      create  config/initializers/backtrace_silencers.rb
      create  config/initializers/filter_parameter_logging.rb
      create  config/initializers/inflections.rb
      create  config/initializers/mime_types.rb
      create  config/initializers/secret_token.rb
      create  config/initializers/session_store.rb
      create  config/initializers/wrap_parameters.rb
      create  config/locales
      create  config/locales/en.yml
      create  config/boot.rb
      create  config/database.yml
      create  db
      create  db/seeds.rb
      create  lib
      create  lib/tasks
      create  lib/tasks/.keep
      create  lib/assets
      create  lib/assets/.keep
      create  log
      create  log/.keep
      create  public
      create  public/404.html
      create  public/422.html
      create  public/500.html
      create  public/favicon.ico
      create  public/robots.txt
      create  tmp/cache
      create  tmp/cache/assets
      create  vendor/assets/javascripts
      create  vendor/assets/javascripts/.keep
      create  vendor/assets/stylesheets
      create  vendor/assets/stylesheets/.keep
         run  bundle install
Fetching gem metadata from https://rubygems.org/...........
Fetching gem metadata from https://rubygems.org/..
Resolving dependencies...
Using rake (10.0.4)
Using i18n (0.6.4)
Using minitest (4.7.3)
Using multi_json (1.7.2)
Using atomic (1.1.8)
Using thread_safe (0.1.0)
Using tzinfo (0.3.37)
Using activesupport (4.0.0.beta1)
Using builder (3.1.4)
Using erubis (2.7.0)
Using rack (1.5.2)
Using rack-test (0.6.2)
Using actionpack (4.0.0.beta1)
Using mime-types (1.23)
Using polyglot (0.3.3)
Using treetop (1.4.12)
Using mail (2.5.3)
Using actionmailer (4.0.0.beta1)
Using activemodel (4.0.0.beta1)
Using activerecord-deprecated_finders (0.0.3)
Using arel (4.0.0)
Using activerecord (4.0.0.beta1)
Using bundler (1.3.5)
Using coffee-script-source (1.6.2)
Using execjs (1.4.0)
Using coffee-script (2.2.0)
Using json (1.7.7)
Using rdoc (3.12.2)
Using thor (0.18.1)
Using railties (4.0.0.beta1)
Using coffee-rails (4.0.0)
Using hike (1.2.2)
Using jbuilder (1.0.2)
Using jquery-rails (2.2.1)
Installing pg (0.15.1)
Using tilt (1.3.7)
Using sprockets (2.9.3)
Using sprockets-rails (2.0.0.rc4)
Using rails (4.0.0.beta1)
Using sass (3.2.8)
Using sass-rails (4.0.0.rc1)
Using turbolinks (1.1.1)
Using uglifier (2.0.1)
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
```
<br>

Next, configure PostgreSQL in your database.yml file.  If you need help setting up Postgres itself, check out [this](http://railscasts.com/episodes/342-migrating-to-postgresql) RailsCasts tutorial.  Set the database username and password to the correct username and password for your specific Postgres set up:

**/config/database.yml**

```yaml
# ...

development:
  adapter: postgresql
  encoding: unicode
  database: blog_development
  pool: 5
  username: pguser
  password:

# ...

test:
  adapter: postgresql
  encoding: unicode
  database: blog_test
  pool: 5
  username: pguser
  password:

```
<br>

Next, create the initial database with the `rake db:create` command in the console:

```bash
$ bundle exec rake db:create
```

<br>

Now launch Rails and test the site out in your browser.  You should see something like the below screenshot.

**Launch Rails server**

```bash
blog $ rails s
=> Booting WEBrick
=> Rails 4.0.0.beta1 application starting in development on http://0.0.0.0:3000
=> Call with -d to detach
=> Ctrl-C to shutdown server
[2013-04-27 23:36:01] INFO  WEBrick 1.3.1
[2013-04-27 23:36:01] INFO  ruby 2.0.0 (2013-02-24) [x86_64-darwin12.3.0]
[2013-04-27 23:36:01] INFO  WEBrick::HTTPServer#start: pid=73722 port=3000
```

<br>

**http://localhost:3000/**

<img src="{{relative}}/images/bootstrapss4.jpg" class="img-polaroid">

<br>

Once you have confirmed things are working, hit Ctrl-C to shutdown your server and return to the command line.

<br>

**Shut down Rails server**

```bash
^C
[2013-04-28 09:32:03] INFO  going to shutdown ...
[2013-04-28 09:32:03] INFO  WEBrick::HTTPServer#start done.
Exiting
blog $
```

<br>

### Adding AngularJS to the app

Now that we have our Rails app set up, it is time to add the AngularJS framework.  Before adding it to our project, though, it is worth disabling Turbolinks since much of the functionality is duplicated within AngularJS.  Although it is possible to run Turbolinks in parallel with Angular (see [here](http://stackoverflow.com/questions/14797935/using-angularjs-with-turbolinks)), in my experience it is more work than it is worth.

<br>

##### Disable Turbolinks

First, go into your app/assets/javascripts/application.js file and remove the `//= require turbolinks` line from the file.  This will stop including turbolinks.js in each of your webpages.

Next, go to **app/views/layouts/application.html.erb** and remove `"data-turbolinks-track" => true` from both the `stylesheet_link_tag` and `javascript_include_tag` lines.  After removing these parameters, the beginning of application.html.erb should look something like this:

<br>

```erb
 <html>
 <head>
   <title>Application Title</title>
   <%= stylesheet_link_tag    "application", media: "all" %>
   <%= javascript_include_tag "application" %>
   <%= csrf_meta_tags %>
 </head>
```

<br>

##### Download AngularJS and copy into the project

Download the the latest stable version of AngularJS (at the time of writing 1.0.6) from the [AngularJS site](http://angularjs.org/), selecting the stable branch and the zip build (which includes both uncompressed and minified assets).  Unzip the files and add all of the JS files to your `vendor/assets/javascripts` directory.

<br>

##### Add AngularJS as a dependency in application.js

Now add a Sprockets require statement in your **app/assets/javascripts/application.js** file:

```javascript

// ...

//= require jquery
//= require jquery_ujs
//= require underscore
//= require bootstrap
//= require angular

// ...

```

<br>

Note that we have also removed the `require_tree` statement, which would have had the effect of loading every javascript file in the project with each page load.  This is obviously not an efficient approach with multiple controllers, so we have removed this parameter.

<br>

### Creating your first Rails controller

Let's generate our first Rails controller.  This will be the main homepage controller and we will call it `Main`.

**Generating the Main controller**

```bash
$ rails generate controller Main index
      create  app/controllers/main_controller.rb
       route  get "main/index"
      invoke  erb
      create    app/views/main
      create    app/views/main/index.html.erb
      invoke  helper
      create    app/helpers/main_helper.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/main.js.coffee
      invoke    scss
      create      app/assets/stylesheets/main.css.scss
```

<br>

Note that, as part of generating this controller, the `MainController` route was also added to `/config/routes.rb`.

**/config/routes.rb**

```ruby
Blog::Application.routes.draw do
  get "main/index"

  # ...

end
```

<br>

To test if this is working, return to the console and type `rails s` to relaunch the server, and then try typing this route into the browser:

**http://localhost:3000/main/index**

<img src="{{relative}}/images/bootstrapss5.png" class="img-polaroid">

<br>

### Angular asset structure and Sprockets setup

While you could work out of a single file, as your project grows beyond a single controller, it quickly becomes more and more difficult to maintain and track.  There are lots of opinions on the best way to track modules in an Angular project, but this is what works for me.  Create the following folders in the **app/assets/javascripts** directory:

```bash
./controllers
./controllers/main
./controllers/global
./directives
./directives/main
./directives/global
./filters
./filters/main
./filters/global
./services
./services/main
./services/global
```

<br>

The basic structure here is that we are separating out controllers, directives, filters, and services modules.  These are the ones I find myself using most commonly, but you can add any other types of modules you use as well.  Then, within each module type, I separate files by Rails controller they are associated with, and so you would have a separate folder for each controller.  In addition, I also usually include a global folder for any resources used across all controllers, but you could also break this down by type of functionality (e.g., a shared set of login files used by a subset of controllers).

Once these folders are set up, we need to require the right subset of controllers in our Sprockets set up, starting with **app/assets/javascripts/application.js**:

```javascript
//= require jquery
//= require jquery_ujs
//= require angular
```

<br>

Note that I have removed the `require_tree .` statement.  This statement recursively requires the entire javascripts folder, rendering our folder structure irrelevant for anything more than organization.  Because we want to selectively require these files, we need to specify all non-global requirements in our **app/assets/javascripts/[main].js.coffee** file.

```coffeescript
#= require_self
#= require_tree ./services/global
#= require_tree ./services/main
#= require_tree ./filters/global
#= require_tree ./filters/main
#= require_tree ./controllers/global
#= require_tree ./controllers/main
#= require_tree ./directives/global
#= require_tree ./directives/main
```

<br>

The one thing this structure leaves out is templates. I generally create a single `templates` directory at `app/assets/templates`.  You could create subdirectories here as well, but I usually wait until a project gets more complicated since this does not implicate any Sprockets requirement issues the way the javascript directories do.

<br>

### Setting up an AngularJS module, router, and initial controller

Now let's build our first AngularJS page to test whether all of this structure we just created is working properly.  Start by creating an extremely simple Angular controller called `IndexCtrl`.

**app/assets/javascripts/controllers/main/mainIndexCtrl.js.coffee**

```coffeescript
@IndexCtrl = ($scope) ->
  $scope.title = "My blog"
```

<br>

Next, create a simple `div` element with an `ng-controller="IndexCtrl"` attribute.  We will then create an `h1` element that references the `title` attribute of our controller's `$scope`.
  
**app/views/main/index.html.erb**

```html
<div ng-controller="IndexCtrl">
  <h1>{{ "{{ title"}} }}</h1>
</div>
```
<br>

If you run the server now and open the page, you will see that something is not quite right.

**http://localhost:3000/main/index**

<img src="{{relative}}/images/bootstrapss1.jpg" class="img-polaroid">

<br>

There are two things stopping this from working, both of which have to do with our main application layout file.  The first is that Angular is not even loading in the first place.  This is because you need to add an `ng-app` tag to the main `<html>` tag.  However, if you made this change (shown below) and ran the page, you would still not see the title loading properly.  Looking at the page source, you will notice the second issue, which is that main.js is not included.  Since main.js is what pulls in /controllers/main/mainIndexCtrl.js, our controller is not being included.  There is an easy fix for this as well.  Open your application layout file and add `controller_name` to the `javascript_include_tag`.  This will automatically load the `main` javascript file associated with each controller you create.  The `main.js` file in turn will require each of the controller specific javascript files.

<br>

**app/views/layouts/application.html.erb**

```erb
<!DOCTYPE html>
<html ng-app>
<head>
  <title>Blog</title>
  <%= stylesheet_link_tag    "application", media: "all" %>
  <%= javascript_include_tag "application", controller_name %>
  <%= csrf_meta_tags %>
</head>
<body>

<%= yield %>

</body>
</html>
```

<br>

After making these two changes, if you reload the page, you should see what we expected to see in the first place.  As you can see, Angular has correctly loaded the title for the page from the controller.

**http://localhost:3000/main/index**

<img src="{{relative}}/images/bootstrapss2.jpg" class="img-polaroid">

<br>

We now have a basic AngularJS controller up and running.  But it's not really showcasing much of the power AngularJS provides.  So let's add some data for our blog.  I have removed the blog title from `$scope` based on the assumption that the title will be static.  It is now hard-coded into the html for this page.  

**app/views/layouts/application.html.erb**

```erb

<head>
  <title>Blog</title>
  <%= stylesheet_link_tag    "application", media: "all" %>
  <%= javascript_include_tag "application", controller_name %>
  <%= csrf_meta_tags %>
</head>

```

<br>

In its place, I have created a `$scope.data` hash containing a single key, `posts`.  The `posts` key will contain an array of database post hashes.  Each individual post will have a `title` and `contents` strings.  For now, I have generated some placeholder data at [lipsum.com](http://www.lipsum.com/).

**app/assets/javascripts/controllers/main/mainIndexCtrl.js.coffee**

```coffeescript
@IndexCtrl = ($scope) ->
  $scope.data = 
    posts: [{title: 'My first post', contents: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec laoreet lobortis vulputate. Ut tempus, orci eu tempor sagittis, mauris orci ultrices arcu, in volutpat elit elit semper turpis. Maecenas id lorem quis magna lacinia tincidunt. In libero magna, pharetra in hendrerit vitae, luctus ac sem. Nulla velit augue, vestibulum a egestas et, imperdiet a lacus. Nam mi est, vulputate eu sollicitudin sed, convallis vel turpis. Cras interdum egestas turpis, ut vestibulum est placerat a. Proin quam tellus, cursus et aliquet ut, adipiscing id lacus. Aenean iaculis nulla justo.'}, {title: 'A walk down memory lane', contents: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin leo sem, imperdiet in faucibus et, feugiat ultricies tellus. Vivamus pellentesque iaculis dolor, sed pellentesque est dignissim vitae. Donec euismod purus non metus condimentum porttitor suscipit nibh tempor. Etiam malesuada elit in lectus pharetra facilisis. Fusce at nisl augue. Donec at est felis. Sed a gravida diam. Nunc nunc mi, egestas non dignissim et, porta aliquam ante.'}]
```

<br>

Having set up some placeholder data, AngularJS makes it easy to implement this in our html template.  I have created a new `<div>` element with an `ng-repeat` attribute.  By invoking this attribute, Angular will watch the contents of data.posts and add a separate `<div>` element for each post in that array.  We will use curly braces again to yield the title and contents of each post onto the page.

**app/views/main.html.erb**

```erb
<div class="container" ng-controller="IndexCtrl">
  <h1 class="text-center">My blog</h1>
  <div class="row" ng-repeat="post in data.posts">
      <h2>{{ "{{ post.title"}} }}</h2>
      <p>{{ "{{ post.contents"}} }}</p>
  </div>
</div>
```

<br>

If you reload the page, you should see two blog posts with the title and contents from our above placeholder data.

Notice that in the HTML above I added some [Twitter Bootstrap](http://twitter.github.io/bootstrap/) CSS tags to make this look a bit prettier.  This code will work fine without those class tags, but this makes it look a little more well-organized.  But they will not do anything until we add Bootstrap to the project.  To add Bootstrap, first add the "bootstrap-sass" gem to your gemfile.

**gemfile**

```ruby

# ...

  gem 'bootstrap-sass', '~> 2.3.1.0'

# ...

```

<br>

At the command line, type `bundle install` to install the gem.  Then, require Bootstrap in your **main.css.sass** and **application.js** files:

**app/assets/stylesheets/main.css.sass**

```sass

// Place all the styles related to the Main controller here.
// They will automatically be included in application.css.
// You can use Sass (SCSS) here: http://sass-lang.com/

@import "bootstrap";

```

<br>

**app/assets/javascripts/application.js**

```javascript
//= require jquery
//= require jquery_ujs
//= require bootstrap
//= require angular
```

<br>

After that, if you reload the page, you should see something that looks like this:

**http://localhost:3000/main/index**

<img src="{{relative}}/images/bootstrapss3.jpg" class="img-polaroid">

<br>

###Conclusion

That's it for Part 1.  We now have a functioning Rails app with some basic AngularJS functionality.  But this is only the basic groundwork.  In the next part, we will dive into building a more complete Angular module structure for the app with support for routing, templates, multiple controllers, and data sharing between controllers.

***I am currently working on part 2, which will begin to cover modularization, routing, templates, and multiple AngularJS controllers.  [Stay tuned!](http://twitter.com/asandersn/)***

<br><br>

<div id="disqus_thread"></div>
<script type="text/javascript">
  /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
  var disqus_shortname = 'adamandersonblog';
  var disqus_identifier = '2013-06-03-bootstrapping-angular-rails-part-1';
  var disqus_title = 'Bootstrapping an AngularJS app in Rails 4.0 - Part 1';
  var disqus_url = 'http://asanderson15.github.io/posts/2013/06/03/bootstrapping-angular-rails-part-1.html';

  /* * * DON'T EDIT BELOW THIS LINE * * */
  (function() {
      var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
      dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
      (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
  })();
</script>
<noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
<a href="http://disqus.com" class="dsq-brlink">comments powered by <span class="logo-disqus">Disqus</span></a>

<script type="text/javascript">
  /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
  var disqus_shortname = 'adamandersonblog'; // required: replace example with your forum shortname
  var disqus_identifier = '2013-06-03-bootstrapping-angular-rails-part-1';
  var disqus_title = 'Bootstrapping an AngularJS app in Rails 4.0 - Part 1';
  var disqus_url = 'http://asanderson15.github.io/posts/2013/06/03/bootstrapping-angular-rails-part-1.html';

  /* * * DON'T EDIT BELOW THIS LINE * * */
  (function () {
      var s = document.createElement('script'); s.async = true;
      s.type = 'text/javascript';
      s.src = '//' + disqus_shortname + '.disqus.com/count.js';
      (document.getElementsByTagName('HEAD')[0] || document.getElementsByTagName('BODY')[0]).appendChild(s);
  }());
</script>
    