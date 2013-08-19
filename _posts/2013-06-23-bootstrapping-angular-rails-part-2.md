---
layout: post
title: Bootstrapping an AngularJS app in Rails 4.0 - Part 2
category: posts
summary: Part two of this tutorial focuses on AngularJS, covering modularization, routing, templates, and multiple AngularJS controllers.
socialsummary: Part two of this tutorial focuses on AngularJS, covering modularization, routing, templates, and multiple AngularJS controllers.
socialimage: http://asanderson.org/images/AngularJS-Shield-large.png
tags: AngularJS Rails Tutorial
year: 2013
month: 6
day: 23
published: true
comments: true
---
***The full GitHub repository for this tutorial is available [here](https://github.com/asanderson15/rails-angular-tutorial)***


## Introduction

This is part two of my tutorial covering building an Angular and Rails blog application from scratch.  Jumping off right from where the [last part](http://asanderson.org/posts/2013/06/03/bootstrapping-angular-rails-part-1.html) left off, in this post, I will cover:

* Modularizing the AngularJS app
* Setting up AngularJS routing
* Using Angular's routing to manage multiple controllers and templates

As before, this series assumes a certain basic understanding of both Rails and AngularJS.  If you need an introduction to Rails, I recommend checking out the excellent [Ruby on Rails Tutorial](http://ruby.railstutorial.org/) by Michael Hartl.  For an intro to AngularJS, I recommend checking out the [homepage tutorials](http://angularjs.org/) and, to go a bit deeper, the excellent [egghead.io tutorial videos](http://egghead.io/) by John Lindquist.

<br>


## Angular modules

The first thing we will do is create a module for our application.  This module will allow us to leverage the `ng-app` directive to specify the main application module.  Since this will be the master Angular module for the `Main` Rails controller, we will declare it up front in our main.js.coffee file:

**app/assets/javascripts/main.js.coffee**

```coffeescript
# Place all the behaviors and hooks related to the matching controller here.
# All this logic will automatically be available in application.js.
# You can use CoffeeScript in this file: http://coffeescript.org/

#= require_self
#= require_tree ./Controllers/main
#= require_tree ./Directives/main
#= require_tree ./Filters/main
#= require_tree ./Services/main

# Creates new Angular module called 'Blog'
Blog = angular.module('Blog', [])

```
<br>

The next step is to specify the `Blog` module we just created as our main application module in the `ng-app` directive, which was declared in our opening `<html>` tag.

**app/views/layouts/application.html.erb**

```erb
<!DOCTYPE html>
<html ng-app="Blog">
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

Even with this addition, not much has changed.  The application is still declaring `IndexCtrl` to be the `ng-controller` and our `Blog` module does not have any functionality of its own.  Let's change that by adding routing.

<!--- more -->

<br>

## Implementing routing

### Rails route

Before getting into setting up Angular routes, let's first change our main controller to be the root page when users visit our site.  To do this, change the routes config file in Rails like so:

**config/routes.rb**

```ruby
Blog::Application.routes.draw do
  root to: 'main#index'

  # ...
end
```

Now, if you visit [http://localhost:3000/](http://localhost:3000/), you should see the index action on the Main controller.  With this change, we are now ready to set up Angular routing.


### Angular routes

Setting up routing in Angular requires configuring your new `Blog` module's routeProvider.  This is done by calling the `when` and `otherwise` functions on $routeProvider as part of the module's configuration.  

The `when` function allows you to configure a set of URLs to be handled without a page reload.  The first argument takes a string defining the URL for the route.  The second argument is a hash containing a templateUrl and a controller name.  We will come back to templates next.  The controller name is the name of an Angular controller like the `IndexCtrl` controller we set up earlier.  

The `otherwise` function allows you to set a default to handle the case where the given route matches none of the configured options.  Let's try out the below configuration for our blog app:

**app/assets/javascripts/main.js.coffee**

```coffeescript
# ...

# Creates new Angular module called 'Blog'
Blog = angular.module('Blog', [])

# Sets up routing
Blog.config(['$routeProvider', ($routeProvider) ->
  # Route for '/post'
  $routeProvider.when('/post', { templateUrl: '../assets/mainPost.html', controller: 'PostCtrl' } )

  # Default
  $routeProvider.otherwise({ templateUrl: '../assets/mainIndex.html', controller: 'IndexCtrl' } )

])

```
<br>

Once we complete our set up below, the default route when you visit [http://localhost:3000/](http://localhost:3000/) will be the `IndexCtrl` controller and the **mainIndex.html** view template.  If you were to visit [http://localhost:3000/#/post](http://localhost:3000/#/post), `MainPostCtrl` will be the Angular controller and **mainPost.html** will be the view template.  So let's talk about what this actually means.

<br>

### View Controllers

When first loading a page, the `Blog` Angular module we just created will look to its routeProvider to determine the view template and controller based on the URL.  Once it has that information, the chosen controller becomes the main `ng-controller` for the page.  In other words, we no longer need to specify `ng-controller` in a `<div>` element like we do now.  Rather, the `Blog` module specified in the `ng-app` directive acts to specify the controller itself.  

But what part of the page is controlled by this module-specified `ng-controller`?  That brings us to a new tag: introducing the `ng-view` directive.  The `ng-view` directive itself does two important things.  First, it defines the scope of the Angular module-specified `ng-controller` within the page.  Second, it tells the Angular module where to insert the view template specified in the routing.

<br>

### View Templates

So what is the view template?  A view template is just a partial view consisting of ordinary HTML.  It is similar to an HTML template in Rails, just without the ERB syntax.  In that same vein, the `ng-view` attribute is somewhat analogous to a `<%= yield %>` call.

Rather than getting into more detail in the abstract, let's try an example.  First, let's create a new folder to store our Angular templates: `app/assets/templates`.  Then let's pull the HTML for our view that we previously defined in *index.html.erb* into a separate view template:

**app/assets/templates/mainIndex.html**

```html
<h1 class="text-center">My blog</h1>
<div class="row" ng-repeat="post in data.posts">
    <h2>{{ "{{ post.title"}} }}</h2>
    <p>{{ "{{ post.contents"}} }}</p>
</div>
```
<br>

Then, remove that same code from the Index template and replace the `ng-controller` attribute with an `ng-view` attribute instead:

**app/views/main/index.html.erb**

```erb
<div class="container" ng-view>
</div>
```
<br>


Now, try loading the page again.  As you can see, everything should look exactly like it did before:

**http://localhost:3000/**
<img src="{{relative}}/images/bootstrapss2-1.jpg" class="img-polaroid">

<br>

Looking at the web inspector, you can see that Angular injected the template inside the div with the `ng-view` attribute:

<img src="{{relative}}/images/bootstrapss2-2.jpg" class="img-polaroid">

<br>

So let's take a moment to recap the process that is happening:

1. The page loads, along with Angular, and creates the `Blog` module, along with its route configuration.
2. The `ng-app` attribute tells Angular to bootstrap using `Blog` as the application module.
3. The `Blog` module's `routeProvider` is invoked and, based on the URL, it sets the `ng-controller` to be IndexCtrl and the view template to be mainIndex.html.
4. Angular initializes the IndexCtrl controller.
5. Angular inserts the mainIndex.html partial in the div with the `ng-view` attribute.

This may seem a bit complicated at first, but it gets significantly easier after doing it a few times.  More importantly, though, this is a huge amount of power with only a very minimal amount of set up and boilerplate.

<br>

## Setting up multiple AngularJS controllers and templates

Now that we have modularized our AngularJS application, let's implement our second route, **/post**.  Eventually, we will link the title of each post to an individual post page.  But for now, there will only be a single static post page.  We will get into making it dynamic with a shared model in the next few posts.

First, let's review the route we set up earlier:


**app/assets/javascripts/main.js.coffee**

```coffeescript
# ...

# Sets up routing
Blog.config(['$routeProvider', ($routeProvider) ->
  # Route for '/post'
  $routeProvider.when('/post', { templateUrl: '../assets/mainPost.html', controller: 'PostCtrl' } )

# ...

```
<br>

As you can see, the template for the view is mainPost.html, while the controller is MainPostCtrl.  Let's set up a simple template and controller for this new view.

**app/assets/javascripts/Controllers/main/**

```coffeescript
@PostCtrl = ($scope) ->

  $scope.data = 
    post: {title: 'My first post', contents: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec laoreet lobortis vulputate. Ut tempus, orci eu tempor sagittis, mauris orci ultrices arcu, in volutpat elit elit semper turpis. Maecenas id lorem quis magna lacinia tincidunt. In libero magna, pharetra in hendrerit vitae, luctus ac sem. Nulla velit augue, vestibulum a egestas et, imperdiet a lacus. Nam mi est, vulputate eu sollicitudin sed, convallis vel turpis. Cras interdum egestas turpis, ut vestibulum est placerat a. Proin quam tellus, cursus et aliquet ut, adipiscing id lacus. Aenean iaculis nulla justo.'}

```
<br>

**app/assets/templates/mainPost.html**

```html
<h1 class="text-center">{{ "{{ data.post.title"}} }}</h1>
<div class="row">
    <p>{{ "{{ data.post.contents"}} }}</p>
</div>
```
<br>

We will also add a link to the title of each blog post.  Again, this link will only take us to the static page, not text of the specific blog post clicked on.

**app/assets/templates/mainIndex.html**

```html
<h1 class="text-center">My blog</h1>
<div class="row" ng-repeat="post in data.posts">
    <h2><a href="./post">{{ "{{ post.title"}} }}</a></h2>
    <p>{{ "{{ post.contents"}} }}</p>
</div>
```
<br>

Now, reload the page and try clicking on the links in the title.  You should see an error message like this:

**http://localhost:3000/post**

<img src="{{relative}}/images/bootstrapss2-3.jpg" class="img-polaroid">
<br>

What happened?  Well, the link we just created was an absolute route.  As Angular is configured by default, it does not implement HTML5 pushState features.  This means that absolute routes will still cause the browser to request the new page directly from the server.  We will talk about workarounds for this later, but for now, let's try implementing the link via the Angular controller.  First, we will add an ng-click directive to our `<a>` tag.  This is similar to `onClick`, but works with Angular's own functionality.

**app/assets/templates/mainIndex.html**

```html
<h1 class="text-center">My blog</h1>
<div class="row" ng-repeat="post in data.posts">
    <h2><a ng-click="viewPost()">{{ "{{ post.title"}} }}</a></h2>
    <p>{{ "{{ post.contents"}} }}</p>
</div>
```
<br>

Then, we will add the function to our IndexCtrl controller:

**app/assets/templates/Controllers/main/mainIndexCtrl.js.coffee**

```coffeescript
@IndexCtrl = ($scope, $location) ->

  $scope.data = 
    posts: [{title: 'My first post', contents: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec laoreet lobortis vulputate. Ut tempus, orci eu tempor sagittis, mauris orci ultrices arcu, in volutpat elit elit semper turpis. Maecenas id lorem quis magna lacinia tincidunt. In libero magna, pharetra in hendrerit vitae, luctus ac sem. Nulla velit augue, vestibulum a egestas et, imperdiet a lacus. Nam mi est, vulputate eu sollicitudin sed, convallis vel turpis. Cras interdum egestas turpis, ut vestibulum est placerat a. Proin quam tellus, cursus et aliquet ut, adipiscing id lacus. Aenean iaculis nulla justo.'}, {title: 'A walk down memory lane', contents: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin leo sem, imperdiet in faucibus et, feugiat ultricies tellus. Vivamus pellentesque iaculis dolor, sed pellentesque est dignissim vitae. Donec euismod purus non metus condimentum porttitor suscipit nibh tempor. Etiam malesuada elit in lectus pharetra facilisis. Fusce at nisl augue. Donec at est felis. Sed a gravida diam. Nunc nunc mi, egestas non dignissim et, porta aliquam ante.'}]

  $scope.viewPost = ->
    $location.url('/post')

```
<br>

You'll notice that I injected Angular's $location module into IndexCtrl to facilitate this functionality.  Thanks to Angular's on-demand dependency injection, adding this functionality is as easy as inserting it as the name of one of the arguments to the controller.  The `$location.url` function is a simple redirection function similar to others found in Javascript.  It starts from the path of the current page and then uses the '#' prefix to avoid the ordinary browser routing functionality and avoid a page reload.

If you try reloading the page and clicking on one of the post title links again, you should now see something more like this:

**http://localhost:3000/#/post**

<img src="{{relative}}/images/bootstrapss2-4.jpg" class="img-polaroid">

<br>

That's more like it.  

<br>

## Route parameters

Let's touch on one more key feature of Angular routing before wrapping up: route parameters.  This allows you to handle common RESTful URLs like `/users/123` and `/users/123/photos/5`.  The syntax for adding in these parameters is simple.  For the preceding examples, they would have been `/users/:userId` and `/users/:userId/photos/:photoId`.  Let's create a post id parameter and display the post id number in the title on the post page.

First, let's update our routing configuration to add an `:id` parameter:

**app/assets/javascripts/main.js.coffee**

```coffeescript

# ...

# Sets up routing
Blog.config(['$routeProvider', ($routeProvider) ->
  # Route for '/post'
  $routeProvider.when('/post/:postId', { templateUrl: '../assets/mainPost.html', controller: 'PostCtrl' } )

# ...
```
<br>

Next, let's update the PostCtrl controller to handle this parameter.  To do this, we simply add the $routeParams module to the controller.  Parameters in the route are automatically passed into the $routeParams hash based on the name given in the configuration:

**app/assets/javascripts/controllers/main/mainPostCtrl.js.coffee**

```coffeescript
@PostCtrl = ($scope, $routeParams) ->

  # ...

  $scope.data.postId = $routeParams.postId
```
<br>

Now, let's give a small example of how this can be useful for passing parameters.  Let's update our IndexCtrl controller to accept a postId.

**app/assets/javascripts/controllers/main/mainIndexCtrl.js.coffee**

```coffeescript
@IndexCtrl = ($scope, $location) ->

  # ...

  $scope.viewPost = (postId) ->
    $location.url('/post/'+postId)
```
<br>

We will then implement this in the view templates for these two pages:

**app/assets/templates/mainIndex.html**

```html
<h1 class="text-center">My blog</h1>
<div class="row" ng-repeat="post in data.posts">
    <h2><a ng-click="viewPost($index)">{{ "{{ post.title"}} }}</a></h2>
    <p>{{ "{{ post.contents"}} }}</p>
</div>
```

**app/assets/templates/mainPost.html**

```html
<h1 class="text-center">{{ "{{ data.postId "}} }} - {{ "{{ data.post.title"}} }}</h1>
<div class="row">
    <p>{{ "{{ data.post.contents "}} }}</p>
</div>
```
<br>

The $index parameter gives you the index of the current item in the `ng-repeat` iterator.  When you pass this parameter in, it is added to the URL by the call to $location.url.  That parameter is read by the PostCtrl controller through the $routeParams module and stored in the $scope.data hash.  From there, it is accessed by the view template and inserted into the page as text.

In a real world context, we would replace the $index parameter with the post's name, id, or other unique identifier.  This allows us to put in place a permalink structure, at least once we enable HTML5 pushState.

## Conclusion

So that's Part 2.  We now have a Rails app with a single Rails controller combined with an Angular module, routing, and 2 controllers and views.  In the next part, we will dive into building a simple Rails API to serve blog post data to our main controller.  From there, we will set up our own custom Angular shared service module to access and cache this data client-side.

***Find this helpful? Check out [Part 3](http://asanderson.org/posts/2013/08/19/bootstrapping-angular-rails-part-3.html). And [follow me](http://twitter.com/asandersn/) on Twitter for more updates.***

<br><br>

<div id="disqus_thread"></div>
<script type="text/javascript">
  /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
  var disqus_shortname = 'adamandersonblog';
  var disqus_identifier = '2013-06-23-bootstrapping-angular-rails-part-2';
  var disqus_title = 'Bootstrapping an AngularJS app in Rails 4.0 - Part 2';
  var disqus_url = 'http://asanderson.org/posts/2013/06/23/bootstrapping-angular-rails-part-2.html';

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
  var disqus_identifier = '2013-06-23-bootstrapping-angular-rails-part-2';
  var disqus_title = 'Bootstrapping an AngularJS app in Rails 4.0 - Part 2';
  var disqus_url = 'http://asanderson.org/posts/2013/06/23/bootstrapping-angular-rails-part-2.html';

  /* * * DON'T EDIT BELOW THIS LINE * * */
  (function () {
      var s = document.createElement('script'); s.async = true;
      s.type = 'text/javascript';
      s.src = '//' + disqus_shortname + '.disqus.com/count.js';
      (document.getElementsByTagName('HEAD')[0] || document.getElementsByTagName('BODY')[0]).appendChild(s);
  }());
</script>
    