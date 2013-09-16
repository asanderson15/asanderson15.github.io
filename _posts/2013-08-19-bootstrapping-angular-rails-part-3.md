---
layout: post
title: Bootstrapping an AngularJS app in Rails 4.0 - Part 3
category: posts
summary: Part three of this tutorial focuses on building a simple Rails API to serve blog post data to our main Angular controller.
socialsummary: Part three of this tutorial focuses on building a simple Rails API to serve blog post data to our main Angular controller.
socialimage: http://asanderson.org/images/AngularJS-Shield-large.png
tags: AngularJS Rails Tutorial
year: 2013
month: 8
day: 19
published: true
comments: true
---
***The full GitHub repository for this tutorial is available [here](https://github.com/asanderson15/rails-angular-tutorial)***


## Introduction

This is part three of my tutorial covering building an Angular and Rails blog application from scratch.  Jumping in again from where the [last post](http://asanderson.org/posts/2013/06/23/bootstrapping-angular-rails-part-2.html) left off, in this post I will cover:

* Building a simple Rails API to serve blog post data
* Connecting to that API using our main controller.  

As before, this series assumes a certain basic understanding of both Rails and AngularJS.  If you need an introduction to Rails, I recommend checking out the excellent [Ruby on Rails Tutorial](http://ruby.railstutorial.org/) by Michael Hartl.  For an intro to AngularJS, I recommend checking out the [homepage tutorials](http://angularjs.org/) and, to go a bit deeper, the excellent [egghead.io tutorial videos](http://egghead.io/) by John Lindquist.

<br>

## Creating a simple Rails model

To start with, we need a database model to store our blog data.  As with our example data currently in our main AngularJS controller, we will keep this model simple.  Each blog entry will have a `title` and will have its body stored in a `contents` field.  We will also add fields for an `author` and a `date`.  Luckily, Rails makes setting this model up simple.  At the command line, use the Rails `generate` command to create the migration that sets up this model:

<br>

```bash
$ rails generate model Post title:string contents:text author:string post_date:datetime
      invoke  active_record
      create    db/migrate/20130803203410_create_posts.rb
      create    app/models/post.rb
```

<br>

As you can see, Rails generates our first database migration.  If you open that migration up to inspect, it should look something like this.

**db/migrate/[date-time-created]\_create\_posts.rb**

```ruby
class CreatePosts < ActiveRecord::Migration
  def change
    create_table :posts do |t|
      t.string :title
      t.text :contents
      t.string :author
      t.datetime :post_date

      t.timestamps
    end
  end
end
```

<br>

<!--- more -->

Rails also created a file containing the Post model itself, though as you can see, there is no specific functionality or validation defined as of yet.  We will come back to implement this later.

**app/models/post.rb**

```ruby
class Post < ActiveRecord::Base

end
```

<br>

The only thing left to create this model is to actually run the migration.  You can complete that by running `rake db:migrate` at the command line:

```bash
$ rake db:migrate
==  CreatePosts: migrating ====================================================
-- create_table(:posts)
   -> 0.0037s
==  CreatePosts: migrated (0.0038s) ===========================================

$
```

<br>

We now have a Posts model, and a table for that model set up in the database, but no data.  Let's set up two quick example posts using the Rails console so we will be able to test with real data.

```bash
$ rails c
Loading development environment (Rails 4.0.0)

2.0.0p0 :001 > ########  First, let's create a new post.  #########
2.0.0p0 :002 > ####################################################

2.0.0p0 :003 > first_post = Post.new
 => #<Post id: nil, title: nil, contents: nil, author: nil, post_date: nil, created_at: nil, updated_at: nil>

2.0.0p0 :004 > first_post.title = "The first post on the database"
 => "The first post on the database"

2.0.0p0 :005 > first_post.contents = "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec laoreet lobortis vulputate. Ut tempus, orci eu tempor sagittis, mauris orci ultrices arcu, in volutpat elit elit semper turpis. Maecenas id lorem quis magna lacinia tincidunt. In libero magna, pharetra in hendrerit vitae, luctus ac sem. Nulla velit augue, vestibulum a egestas et, imperdiet a lacus. Nam mi est, vulputate eu sollicitudin sed, convallis vel turpis. Cras interdum egestas turpis, ut vestibulum est placerat a. Proin quam tellus, cursus et aliquet ut, adipiscing id lacus. Aenean iaculis nulla justo."
 => "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec laoreet lobortis vulputate. Ut tempus, orci eu tempor sagittis, mauris orci ultrices arcu, in volutpat elit elit semper turpis. Maecenas id lorem quis magna lacinia tincidunt. In libero magna, pharetra in hendrerit vitae, luctus ac sem. Nulla velit augue, vestibulum a egestas et, imperdiet a lacus. Nam mi est, vulputate eu sollicitudin sed, convallis vel turpis. Cras interdum egestas turpis, ut vestibulum est placerat a. Proin quam tellus, cursus et aliquet ut, adipiscing id lacus. Aenean iaculis nulla justo."

2.0.0p0 :006 > first_post.author = "Foo"
 => "Foo"

2.0.0p0 :007 > first_post.post_date = DateTime.current()
 => Sat, 03 Aug 2013 20:50:26 +0000

2.0.0p0 :008 > first_post.save
   (0.2ms)  BEGIN
  SQL (5.8ms)  INSERT INTO "posts" ("author", "contents", "created_at", "post_date", "title", "updated_at") VALUES ($1, $2, $3, $4, $5, $6) RETURNING "id"  [["author", "Foo"], ["contents", "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec laoreet lobortis vulputate. Ut tempus, orci eu tempor sagittis, mauris orci ultrices arcu, in volutpat elit elit semper turpis. Maecenas id lorem quis magna lacinia tincidunt. In libero magna, pharetra in hendrerit vitae, luctus ac sem. Nulla velit augue, vestibulum a egestas et, imperdiet a lacus. Nam mi est, vulputate eu sollicitudin sed, convallis vel turpis. Cras interdum egestas turpis, ut vestibulum est placerat a. Proin quam tellus, cursus et aliquet ut, adipiscing id lacus. Aenean iaculis nulla justo."], ["created_at", Sat, 03 Aug 2013 20:50:38 UTC +00:00], ["post_date", Sat, 03 Aug 2013 20:50:26 UTC +00:00], ["title", "The first post on the database"], ["updated_at", Sat, 03 Aug 2013 20:50:38 UTC +00:00]]
   (0.5ms)  COMMIT
 => true

2.0.0p0 :009 > ##### Now let's set up a second post ########

2.0.0p0 :010 >   second = Post.new
 => #<Post id: nil, title: nil, contents: nil, author: nil, post_date: nil, created_at: nil, updated_at: nil>

2.0.0p0 :011 > second.title = "This is the second post"
 => "This is the second post"

2.0.0p0 :012 > second.contents = "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin leo sem, imperdiet in faucibus et, feugiat ultricies tellus. Vivamus pellentesque iaculis dolor, sed pellentesque est dignissim vitae. Donec euismod purus non metus condimentum porttitor suscipit nibh tempor. Etiam malesuada elit in lectus pharetra facilisis. Fusce at nisl augue. Donec at est felis. Sed a gravida diam. Nunc nunc mi, egestas non dignissim et, porta aliquam ante."
 => "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin leo sem, imperdiet in faucibus et, feugiat ultricies tellus. Vivamus pellentesque iaculis dolor, sed pellentesque est dignissim vitae. Donec euismod purus non metus condimentum porttitor suscipit nibh tempor. Etiam malesuada elit in lectus pharetra facilisis. Fusce at nisl augue. Donec at est felis. Sed a gravida diam. Nunc nunc mi, egestas non dignissim et, porta aliquam ante."

2.0.0p0 :013 > second.author = "Authorman"
 => "Authorman"

2.0.0p0 :014 > second.post_date = DateTime.current()-7
 => Sat, 27 Jul 2013 20:52:48 +0000

2.0.0p0 :015 > second.save
   (0.2ms)  BEGIN
  SQL (3.0ms)  INSERT INTO "posts" ("author", "contents", "created_at", "post_date", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["author", "Authorman"], ["contents", "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin leo sem, imperdiet in faucibus et, feugiat ultricies tellus. Vivamus pellentesque iaculis dolor, sed pellentesque est dignissim vitae. Donec euismod purus non metus condimentum porttitor suscipit nibh tempor. Etiam malesuada elit in lectus pharetra facilisis. Fusce at nisl augue. Donec at est felis. Sed a gravida diam. Nunc nunc mi, egestas non dignissim et, porta aliquam ante."], ["created_at", Sat, 03 Aug 2013 20:54:19 UTC +00:00], ["post_date", Sat, 27 Jul 2013 20:54:16 UTC +00:00], ["updated_at", Sat, 03 Aug 2013 20:54:19 UTC +00:00]]
   (2.2ms)  COMMIT
 => true

2.0.0p0 :016 > exit

$
```

<br>

As you can see, we manually created two sample blog entries and saved them to the database.  The next step is to build an API so our client app can access them.

<br>

## Creating a Rail API controller

Similar to how we created our model, we can use Rails to generate a barebones controller for our API with the `generate` command.

```bash
$ rails generate controller posts
      create  app/controllers/posts_controller.rb
      invoke  erb
      create    app/views/posts
      invoke  helper
      create    app/helpers/posts_helper.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/posts.js.coffee
      invoke    scss
      create      app/assets/stylesheets/posts.css.scss
```

<br>

As you can see, this generated a number of files and directories.  However, since we are not implementing any views as part of this API (we'll be responding with JSON instead), the only one we really need is `posts_controller.rb`.  To avoid confusion later, let's delete `app/views/posts`, `posts_helper.rb`, `posts.js.coffee`, and `posts.css.scss`.  Now let's take a look at our newly created PostsController.

**app/controllers/posts_controller.rb**

```ruby
class PostsController < ApplicationController

end
```

<br>

As you can see, not much there.  Before creating our API actions, let's add a routes for our API in our route config file.  This is as simple as adding the single line `resources :posts` to the file.

**config/routes.rb**

```ruby
Blog::Application.routes.draw do
  root to: 'main#index'

  resources :posts

end
```

<br>

Then, use the `rake routes` command at the command line to see the new routes we have created.

```bash
$ rake routes
   Prefix Verb   URI Pattern               Controller#Action
     root GET    /                         main#index
    posts GET    /posts(.:format)          posts#index
          POST   /posts(.:format)          posts#create
 new_post GET    /posts/new(.:format)      posts#new
edit_post GET    /posts/:id/edit(.:format) posts#edit
     post GET    /posts/:id(.:format)      posts#show
          PATCH  /posts/:id(.:format)      posts#update
          PUT    /posts/:id(.:format)      posts#update
          DELETE /posts/:id(.:format)      posts#destroy
```

<br>

Rails makes it simple to create all of these API actions with a single line.  Pretty awesome.

Let's start with implementing a single API action&mdash;the `index` action&mdash;which for now we will allow to return all post data.  This is obviously not a good long-term solution, but will be fine for now.  We will respond to all API requests with JSON, which can be easily parsed by our Angular controller.

**app/controllers/posts_controller.rb**

```ruby
class PostsController < ApplicationController
  respond_to :json

  def index

    # Gather all post data
    posts = Post.all

    # Respond to request with post data in json format
    respond_with(posts) do |format|
      format.json { render :json => posts.as_json }
    end

  end

end
```

<br>

This implementation is fairly straightforward.  First, we configure the controller to respond to JSON requests for all actions.  Next, we define our `index` action to pull all post data from our model, reformat that data as JSON, and then respond to the request with it.  Now let's test whether this is working properly.  Open up two command line windows.  In the first, use the `rails s` command to launch the Rails server.  Then, in the second terminal window, run the following `curl` command on the API:

```bash
$ curl http://localhost:3000/posts.json

[{"id":1,"title":"The first post on the database","contents":"Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec laoreet lobortis vulputate. Ut tempus, orci eu tempor sagittis, mauris orci ultrices arcu, in volutpat elit elit semper turpis. Maecenas id lorem quis magna lacinia tincidunt. In libero magna, pharetra in hendrerit vitae, luctus ac sem. Nulla velit augue, vestibulum a egestas et, imperdiet a lacus. Nam mi est, vulputate eu sollicitudin sed, convallis vel turpis. Cras interdum egestas turpis, ut vestibulum est placerat a. Proin quam tellus, cursus et aliquet ut, adipiscing id lacus. Aenean iaculis nulla justo.","author":"Foo","post_date":"2013-08-03T20:50:26.000Z","created_at":"2013-08-03T20:50:38.811Z","updated_at":"2013-08-03T20:50:38.811Z"},{"id":2,"title":"This is the second post","contents":"Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin leo sem, imperdiet in faucibus et, feugiat ultricies tellus. Vivamus pellentesque iaculis dolor, sed pellentesque est dignissim vitae. Donec euismod purus non metus condimentum porttitor suscipit nibh tempor. Etiam malesuada elit in lectus pharetra facilisis. Fusce at nisl augue. Donec at est felis. Sed a gravida diam. Nunc nunc mi, egestas non dignissim et, porta aliquam ante.","author":"Authorman","post_date":"2013-07-27T20:54:16.000Z","created_at":"2013-08-03T20:54:19.243Z","updated_at":"2013-08-03T20:54:19.243Z"}]%

```

<br>

You should see a response like this, with the sample data we created in the console being returned in JSON format.  While we will eventually build this Rails API out further to handle the full CRUD implementation, let's stop here for now and turn to our client to ingest this data dump.

<br>

## Accessing the blog API from the AngularJS main controller

Now let's turn back to our client main `IndexCtrl` controller.  Loading main page content asynchronously, in this case the posts themselves, poses a small presentation problem.  While that data is loading, the page will show up as blank.  This is not ideal, even if we are able to pull in the post data within a few hundred milliseconds.  To avoid that, let's start by replacing our sample content with a placeholder post entitled "Loading...".

**app/assets/javascripts/Controllers/main/mainIndexCtrl.js.coffee**

```coffeescript
@IndexCtrl = ($scope, $location) ->

  $scope.data = 
    posts: [{title: 'Loading posts...', contents: ''}]

  $scope.viewPost = (postId) ->
    $location.url('/post/'+postId)

```

<br>

If you load the page, you should see something like this.

**http://localhost:3000**

<img src="{{relative}}/images/bootstrapss3-1.jpg" class="img-polaroid">

<br>

This will let the user know to hold on for just a sec while the content loads rather than displaying him or her a blank page initially.  Angular provides some more elegant ways to handle this problem that I will touch on in another post. 

Now let's set up a function to load the blog posts when the controller first loads.  Here is the code to implement this GET request to the API.  I will walk through what is going on below.

**app/assets/javascripts/Controllers/main/mainIndexCtrl.js.coffee**

```coffeescript
@IndexCtrl = ($scope, $location, $http) ->

  $scope.data = 
    posts: [{title: 'Loading posts...', contents: ''}]

  loadPosts = ->
    $http.get('./posts.json').success( (data) ->
      $scope.data.posts = data
      console.log('Successfully loaded posts.')
    ).error( ->
      console.error('Failed to load posts.')
    )

  loadPosts()

  $scope.viewPost = (postId) ->
    $location.url('/post/'+postId)

```

<br>

Stepping through this, you see that we injected an additional Angular module into our controller, specifically `$http`.  This module neatly encapsulates all of the asynchronous HTTP functionality that we will need to interact with our API.

Next, we have created a `loadPosts` function.  The first line uses `$http` to make a GET request to [http://localhost:3000/posts.json](http://localhost:3000/posts.json) to retrieve the JSON data we created earlier containing our initial sample posts.  This `$http.get()` call returns a request object.  Using JS dot notation, we then immediately create two callbacks: one for success and one for error.  In the case of success, our callback will take a `data` argument that represents the JSON data object returned from the API.  We set our `$scope.data.posts` object (which is bound to our HTML view) to contain that returned post data.  Then we call `console.log` to let ourselves know that our API call returned successfully and that our post data is loaded (this is obviously optional).  In the case of an error, we simply log an error to the console to let ourselves know that as well.

Finally, we call `loadPosts()` from the main Index Controller.  This function will run when the controller is first created.

So now when we load our page, the following will happen, starting from the time Angular loads on the client browser:

- Angular routes the request to the `IndexCtrl` controller and renders the appropriate template in the `ng-view` directive in index.html.erb

- The `IndexCtrl` controller initializes with the "Loading..." data. Because this data is bound to the view, this data immediately renders in the browser

- At the same time, `IndexCtrl` calls the `loadPosts()` function, asynchronously requesting the post data stored in our Rails database from our API

- As soon as Rails returns our posts data, the `success` callback is called. This callback replaces the temporary "Loading..." data with the real data from the server.

- Since this data is bound to the view, it will immediately re-render in the browser with the real data

If you reload the page now, it should look something like this.  You may or may not see the "Loading..." temporary data come up, depending on how fast your browser is and how quickly your Rails instance responds to the API call.

**http://localhost:3000/**

<img src="{{relative}}/images/bootstrapss3-2.jpg" class="img-polaroid">

<br>

And there it is.  The sample post data that we entered into our database from the Rails console is now being accessed by Angular via our Rails API and displayed on the client side in the browser!

## Conclusion

That's it for part 3.  We now have a Rails app that exposes an API for accessing and updating blog data.  That Rails backend connects to the client through the main AngularJS controller, which automagically displays the content.  While this is a good high-level demonstration, the structure iof pulling down data in a main controller is not a great or sustainable solution as your app grows.  So in the next part, we will cover a more sophisticated way to use AngularJS to access the API, creating a shared service that can be accessed by any controller. 

***Find this helpful? Check out [Part 4](http://asanderson.org/posts/2013/09/15/bootstrapping-angular-rails-part-4.html). And [follow me](http://twitter.com/asandersn/) on Twitter for more updates.***

<br><br>

<div id="disqus_thread"></div>
<script type="text/javascript">
  /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
  var disqus_shortname = 'adamandersonblog';
  var disqus_identifier = '2013-08-19-bootstrapping-angular-rails-part-3';
  var disqus_title = 'Bootstrapping an AngularJS app in Rails 4.0 - Part 3';
  var disqus_url = 'http://asanderson.org/posts/2013/08/19/bootstrapping-angular-rails-part-3.html';

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
  var disqus_identifier = '2013-08-19-bootstrapping-angular-rails-part-3';
  var disqus_title = 'Bootstrapping an AngularJS app in Rails 4.0 - Part 3';
  var disqus_url = 'http://asanderson.org/posts/2013/08/19/bootstrapping-angular-rails-part-3.html';

  /* * * DON'T EDIT BELOW THIS LINE * * */
  (function () {
      var s = document.createElement('script'); s.async = true;
      s.type = 'text/javascript';
      s.src = '//' + disqus_shortname + '.disqus.com/count.js';
      (document.getElementsByTagName('HEAD')[0] || document.getElementsByTagName('BODY')[0]).appendChild(s);
  }());
</script>
    