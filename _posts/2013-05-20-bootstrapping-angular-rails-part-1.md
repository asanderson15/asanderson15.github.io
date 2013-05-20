---
layout: post
title: Bootstrapping an AngularJS app in Rails 4.0 - Part 1
category: posts
tags: AngularJS
year: 2013
month: 5
day: 20
published: false
---

<br>

### Introduction



### Creating your Rails 4.0 app

<br>

### Adding AngularJS to the app

Now that we have our Rails app set up, it is time to add the AngularJS framework.  Before adding it to our project, though, it is worth disabling Turbolinks since much of the functionality is duplicated within AngularJS.  Although it is possible to run Turbolinks in parallel with Angular (see [here](http://stackoverflow.com/questions/14797935/using-angularjs-with-turbolinks), in my experience it is more work than it is worth.

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

Note that we have also removed the `require_tree` statement, which would have had the effect of loading every javascript file in the project with each page load.  This is obviously not an efficient approach, so we have removed this parameter.

<br>

### Creating your first Rails controller

FILL IN

<br>

### Angular asset structure and Sprockets setup

While you could work out of a single file, as your project grows beyond a single controller, it quickly becomes more and more difficult to maintain and track.  There are lots of opinions on the best way to track modules in an Angular project, but this is what works for me.  Create the following folders in the **app/assets/javascripts** directory:

```bash
./controllers
./controllers/[rails ctrl name]
./controllers/global
./directives
./directives/[rails ctrl name]
./directives/global
./filters
./filters/[rails ctrl name]
./filters/global
./services
./services/[rails ctrl name]
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

<br>

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

The one thing this structure leaves out is templates. I generally create a `templates` directory at `app/assets/templates`.  You could create subdirectories here as well, but I usually wait until a project gets more complicated since this does not implicate any Sprockets requirement issues the way the javascript directories do.

### Setting up an AngularJS module, router, and initial controller

**app/assets/javascripts/controllers/main/mainIndexCtrl.js.coffee**

```coffeescript
@IndexCtrl = ($scope) ->
  $scope.title = "My blog"
```

<br>
  
**app/views/main/index.html.erb**

```html
<div ng-controller="IndexCtrl">
  <h1>{{ "{{ title"}} }}</h1>
</div>
```
<br>

If you run the server now and open the page, you will see that something is wrong.

<br>

**http://localhost:3000/main/index**

<img src="images/bootstrapss1.jpg" class="img-polaroid">

<br>

There are two things stopping this from working, both of which have to do with our main application layout file.  The first is that Angular is not even loading in the first place.  This is because you need to add an `ng-app` tag to the main `<html>` tag.  However, if you make this change (shown below) and ran the page, you would still not see the title loading properly.  Looking at the page source, you will notice the second issue, which is that main.js is not included.  Since main.js is what pulls in /controllers/main/mainIndexCtrl.js, our controller is not being included.  There is an easy fix for this.  Open your application layout file and add `controller_name` to the `javascript_include_tag`.  This will automatically load the main javascript file associated with each controller you create.

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

<br>

**http://localhost:3000/main/index**

<img src="images/bootstrapss2.jpg" class="img-polaroid">

<br>

So we now have a basic AngularJS controller up and running.  But it's not really showcasing much of the power AngularJS provides.  So let's add some data for our blog.  I have removed the blog title from `$scope` based on the assumption that it will be static.  It is now hard-coded into the html for this page.  In its place, I have created a `$scope.data` hash containing a single key, `posts`.  The `posts` key will contain an array of database post hashes.  Each individual post will have a `title` and `contents` strings.

<br>

**app/assets/javascripts/controllers/main/mainIndexCtrl.js.coffee**

```coffeescript
@IndexCtrl = ($scope) ->
  $scope.data = 
    posts: [{title: 'My first post', contents: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec laoreet lobortis vulputate. Ut tempus, orci eu tempor sagittis, mauris orci ultrices arcu, in volutpat elit elit semper turpis. Maecenas id lorem quis magna lacinia tincidunt. In libero magna, pharetra in hendrerit vitae, luctus ac sem. Nulla velit augue, vestibulum a egestas et, imperdiet a lacus. Nam mi est, vulputate eu sollicitudin sed, convallis vel turpis. Cras interdum egestas turpis, ut vestibulum est placerat a. Proin quam tellus, cursus et aliquet ut, adipiscing id lacus. Aenean iaculis nulla justo.'}, {title: 'A walk down memory lane', contents: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin leo sem, imperdiet in faucibus et, feugiat ultricies tellus. Vivamus pellentesque iaculis dolor, sed pellentesque est dignissim vitae. Donec euismod purus non metus condimentum porttitor suscipit nibh tempor. Etiam malesuada elit in lectus pharetra facilisis. Fusce at nisl augue. Donec at est felis. Sed a gravida diam. Nunc nunc mi, egestas non dignissim et, porta aliquam ante.'}]
```

<br>

Having set up some placeholder data, AngularJS makes it easy to implement this in our html template.  I have created a new `<div>` with an `ng-repeat` tag.  By invoking this tag, Angular will watch the contents of data.posts and add a separate `<div>` for each post in that array.  We will then use curly braces again to yield the title and contents of each post onto the page.

<br>

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

Note that I have also added some basic [Twitter Bootstrap](http://twitter.github.io/bootstrap/) CSS tags to make this look a bit prettier.  This code will work fine without those class tags, but this makes it look a tiny bit more well-organized.  To learn more about adding Bootstrap to your project  If you reload the page, you should see something that looks like this.

<br>

**http://localhost:3000/main/index**

<img src="images/bootstrapss3.jpg" class="img-polaroid">

<br>

***I am currently working on part 2, which will cover modularization, routing, templates, and multiple AngularJS controllers.***

