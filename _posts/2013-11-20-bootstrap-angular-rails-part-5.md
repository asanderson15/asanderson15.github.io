---
layout: post
title: Bootstrapping an AngularJS app in Rails 4.0 - Part 5
category: posts
summary: Part five of this tutorial covers creating posts from the client side and using promises to handle asynchronous data loading.
socialsummary: Part five of this tutorial covers creating posts from the client side and using promises to handle asynchronous data loading.
socialimage: http://asanderson.org/images/AngularJS-Shield-large.png
tags: AngularJS Rails Tutorial
year: 2013
month: 11
day: 20
published: true
comments: true
---
**Apologies for the delay getting part 5 out! I have been super busy with finishing up work on my last job, starting a new one, and family obligations. I am hoping to bring the cadence of these posts back up to about one per month, life permitting.**

<br>

***The full GitHub repository for this tutorial is available [here](https://github.com/asanderson15/rails-angular-tutorial)***

## Introduction

This is part five of my tutorial covering building an Angular and Rails blog application from scratch.  We will start where we left off in [part 4](http://asanderson.org/posts/2013/09/15/bootstrapping-angular-rails-part-4.html). In the last post, we set up a shared AngularJS service that allowed a single point of access to API data for all client-side Angular controllers. In this post, we will continue to build out the shared AngularJS service and Rails API, implementing the functionality to create posts.  We will also touch on AngularJS promises for the first time and leverage them to help deal with asynchronous data loading.

<br>

## Creating new posts from the client

Before getting too deeply into implementing the new post functionality, we need to do a bit of refactoring to accommodate the functionality.  Let's start by adding a navigation bar to the top of our homepage.  To do this, we must first move the `container` class out of the `ng-view` div in our main index view:

**app/views/main/index.html.erb**

```erb
<div ng-view>
</div>
```

<br>

This will allow the navigation bar to be the full width of the browser window.  Now, let's implement the navigation bar itself.  (For more information about how to set up or customize a Bootstrap top navbar, check out the documentation [here](http://getbootstrap.com/2.3.2/components.html#navbar).)  Make sure to remember to add the `margin-top: 40px` style to the container `div`&mdash;this ensures that the navbar and page content do not overlap.

**app/assets/templates/mainIndex.html**

```html
<div class="navbar navbar-fixed-top">
  <div class="navbar-inner">
    <a href="#" class="brand">My blog</a>
    <ul class="nav pull-right">
      <li><a ng-click="navNewPost()">New Post</a></li>
    </ul>
  </div>
</div>

<div class="container" style="margin-top: 40px">
  <h1 class="text-center">My blog</h1>
  <div class="row" ng-repeat="post in data.posts">
      <h2><a ng-click="viewPost($index)">{{ "{{ post.title"}} }}</a></h2>
      <p>{{ "{{ post.contents"}} }}</p>
  </div>
</div>
```

<br>

Notice that we have added an `ng-click` directive to call `navNewPost()` upon clicking the "New Post" navigation item.  We will need to implement that in our AngularJS controller next:

**app/assets/javascripts/Controllers/main/mainIndexCtrl.js.coffee**

```coffeescript
@IndexCtrl = ($scope, $location, $http, postData) ->

  # ...

  $scope.viewPost = (postId) ->
    $location.url('/post/'+postId)

  $scope.navNewPost = ->
    $location.url('/post/new')

@PostCtrl.$inject = ['$scope', '$location', '$http', 'postData']
```

<br>

Just like with the `viewPost` method that we previously implemented, to implement this we simply leverage Angular's `$location` service to redirect us to the appropriate route. Now we have our blog homepage ready to accommodate the new post functionality.  It should look something like this:

<!--- more -->

**http://localhost:3000/**

<img src="{{relative}}/images/bootstrapss5-1.jpg" class="img-polaroid">

<br>

### New post controller and template

Next we need to create our new page and controller, and configure our AngularJS router to respond to this new route.  Before we refactor our router, let's stop and take a moment to create the Angular template and controller for our new post page:

**app/assets/templates/mainCreatePost.html**

```html
<div class="navbar navbar-fixed-top">
  <div class="navbar-inner">
    <a ng-click="navHome()" class="brand">My blog</a>
    <ul class="nav pull-right">
      <li><a ng-click="navNewPost()">New Post</a></li>
    </ul>
  </div>
</div>

<div class="container" style="margin-top: 40px">
  <h1 class="text-center">Create new post</h1>
  <form>
    <fieldset>
      <label><strong>Title</strong></label>
      <input class="span12" ng-model="formData.newPostTitle" placeholder="New Post" type="text" maxlength="250">
      <label><strong>Body</strong></label>
      <textarea class="span12" style="height: 250px" ng-model="formData.newPostContents" placeholder="Type your post here..."></textarea>
      <button class="btn btn-primary" ng-click="createPost()">Create</button>
      <button class="btn btn-danger pull-right" ng-click="clearPost()">Clear post</button>
    </fieldset>
  </form>
</div>
```

<br>

As you can see, we have added a top navbar very similar to the one on our homepage.  This is obviously not very [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself), but we will come back and DRY this up in a future post.  

The main body of the page is a very simple form that will allow us to fill in a title and the contents for our new post.  It includes both a submit button, to create a new post, and a clear button that will clear the form.  Both are tied to `ng-click` directives.  For the form itself, we have added `ng-model` directives to both our `input` and `textarea`.  This is an AngularJS binding similar to the curly brace notation we used earlier to display text directly on the page.  As you will see in a moment (and as you may have seen in some boilerplate AngularJS examples), this binding allows us to access and manipulate the values in the form directly from our AngularJS controller (and vice-versa).

Now, let's create the new AngularJS controller for our new post page:

**app/assets/javascripts/Controllers/main/mainCreatePostCtrl.js.coffee**

```coffeescript
@CreatePostCtrl = ($scope, $location, postData) ->

  $scope.data = postData.data
  postData.loadPosts()

  $scope.formData =
    newPostTitle: ''
    newPostContents: ''

  $scope.navNewPost = ->
    $location.url('/post/new')

  $scope.navHome = ->
    $location.url('/')

@CreatePostCtrl.$inject = ['$scope', '$location', 'postData']
```

<br>

Let's walk through what we have done here.  First, as with our other controllers, we inject the shared `postData` service, add it to `$scope`, and call `loadPosts()` to make sure all of our posts are loaded.  Next, we initiate the `formData` object on our `$scope` to contain our form values, which we initially set to be blank strings.  Note that we are not required to explicitly initiate this object for our form to work.  If we left this out of our controller, AngularJS would create this object automatically to accommodate the binding we have created in our HTML template.  Finally, we have added navigation functionality to handle our top navbar.  Note here that the URL to return to home using `$location` is simply `'/'`.

Now we are ready to take the final step of configuring our router to handle this new post page.  To do so, we simply chain an additional `when` call for our **/post/new** route, pointing the `templateURL` to our newly created template and `controller` to our new controller.

**app/assets/javascripts/main.js.coffee**

```coffeescript

# ...

  # Routes for '/post/'
  $routeProvider
    .when('/post/new', { templateUrl: '../assets/mainCreatePost.html', controller: 'CreatePostCtrl' } )
    .when('/post/:postId', { templateUrl: '../assets/mainPost.html', controller: 'PostCtrl' } )

  # Default
  $routeProvider.otherwise({ templateUrl: '../assets/mainIndex.html', controller: 'IndexCtrl' } )

# ...

```

<br>

The order is important here because AngularJS matches routes in the order that you specify.  So if we were to put our **/post/new** route *after* our **/post/:postId** route, Angular would interpret a **/post/new** URL as actually being a **/post/:postId** with `postId == 'new'`. 

Having implemented this new route, if you click on the "New Post" link on the homepage of the blog, the form should look something like this:

**http://localhost:3000/#/post/new**

<img src="{{relative}}/images/bootstrapss5-2.jpg" class="img-polaroid">

<br>

To confirm that we are correctly receiving data from our form to our `CreatePostCtrl` controller, let's implement the `createPost()` method and have it log the form values to the console:

**app/assets/javascripts/Controllers/main/mainCreatePostCtrl.js.coffee**

```coffeescript
@CreatePostCtrl = ($scope, $location, postData) ->

  # ...

  $scope.createPost = ->
    console.log($scope.formData)


```

<br>

Now reload the page, type some text into the form, and click create:

**http://localhost:3000/#/post/new**

<img src="{{relative}}/images/bootstrapss5-3.jpg" class="img-polaroid">

<br>

As you can see, the values input into the form are readily available through the `formData` object.  This is the magic of AngularJS bindings in action!

<br>

### Rails API implementation

Before wiring this form up to our shared `postData` service on the client side, let's first implement this on the server side:

**app/controllers/post_controller.rb**

```ruby
class PostsController < ApplicationController
  respond_to :json

  # ...

  def create

    # Create and save new post from data received from the client
    new_post = Post.new
    new_post.title = params[:new_post][:title][0...250] # Get only first 250 characters
    new_post.contents = params[:new_post][:contents]

    # Confirm post is valid and save or return HTTP error
    if new_post.valid?
      new_post.save!
    else
      render "public/422", :status => 422
      return
    end

    # Respond with newly created post in json format
    respond_with(new_post) do |format|
      format.json { render :json => new_post.as_json }
    end
    
  end

end
```

<br>

What we have done here is fairly self-explanatory.  We will receive the title and contents for our new blog post from our shared AngularJS service on the client side.  With that data, we confirm validity before creating and saving the new post.  Finally, we will respond by returning the fully baked new post object that is now included in our database.  This allows our client-side shared service to handle the post immediately without requiring a reload of all posts.  This will happen as part of our `success` callback, which we will cover in a minute.

Notice that we are checking whether our `new_post` object is valid before saving, but we do not yet have any validation logic in our `Post` model. Let's quickly add validation to our model to ensure we do not end up with any blank blog posts:

**app/models/post.rb**

```ruby
class Post < ActiveRecord::Base

  validates :title, presence: true, length: { minimum: 1, maximum: 250 }
  validates :contents, presence: true

end
```

<br>

This ensures that we have both a title and contents, and that our title is no longer than 250 characters.  This validation is run automatically by Rails each time we try to save a Post object.

<br>

### AngularJS shared service

Next, we will implement the new post creation functionality in our Angular shared service.  This is also relatively straightforward and similar to what we did with our load posts functionality.

**app/assets/javascripts/Services/main/postData.js.coffee**

```coffeescript

# ...

  postData.createPost = (newPost) ->
    # Client-side data validation
    if newPost.newPostTitle == '' or newPost.newPostContents == ''
      alert('Neither the Title nor the Body are allowed to be left blank.')
      return false

    # Create data object to POST
    data =
      new_post:
        title: newPost.newPostTitle
        contents: newPost.newPostContents

    # Do POST request to /posts.json
    $http.post('./posts.json', data).success( (data) ->
      
      # Add new post to array of posts
      postData.data.posts.push(data)
      console.log('Successfully created post.')

    ).error( ->
      console.error('Failed to create new post.')
    )

    return true

# ...

```

Walking through this functionality, we first do a modicum of data validation before sending our POST request off to the server. While not strictly necessary, it is generally a good practice to add both client- and server-side validation logic.  Having checked for blank data, we next assemble our data object that we will send via the POST request.  Finally, we leverage Angular's `$http` service to do a POST request to the server.  As with our asynchronous `loadPosts` method, we create both a `success` and an `error` callback.  In the case that we are successful creating our new post, we will add it to the array of posts.  If our request fails, we will log an error message to the console.

<br>

### Bringing it all together

There is only one small additional change to tie everything together.  Let's implement the `createPost` function in the `CreatePostCtrl` controller for real to call the `createPost` method of our `postData` service, sending our form data as the argument.  We will also quickly implement the functionality for the Clear button while we are at it.

**app/assets/javascripts/Controllers/main/createPostCtrl.js.coffee**

```coffeescript

# ...

  $scope.createPost = ->
    postData.createPost($scope.formData)

  $scope.clearPost = ->
    $scope.formData.newPostTitle = ''
    $scope.formData.newPostContents = ''

# ...

```

<br>

Now that we have implemented the functionality to create a new post on both the client and server side, let's try creating a post to see if everything is working as expected.  Sadly, what you will see is the following error message in the console:

**http://localhost:3000/#/post/new**

<img src="{{relative}}/images/bootstrapss5-4.jpg" class="img-polaroid">

<br>

Looking at Rails in the console, you will see an error message similar to the following:

**Rails in the console**

```bash
Started POST "/posts.json" for 127.0.0.1 at 2013-11-20 11:52:11 -0800
Processing by PostsController#create as JSON
  Parameters: {"new_post"=>{"newPostTitle"=>"The first post created from the client side", "newPostContents"=>"This is a newly created post."}, "post"=>{}}
Can't verify CSRF token authenticity
Completed 422 Unprocessable Entity in 1ms
```

<br>

As the Rails error message states, although the data typed into the form is being sent correctly to the server, we are getting an error because we are failing to send the CSRF token along with our HTTP POST request.  Fortunately, there is an easy solve for this with Angular.  Going back to where we define our master AngularJS module, `Blog`, let's add the following line of code after our router configuration:

**app/assets/javascripts/main.js.coffee**

```coffeescript

# ...

Blog.config(["$httpProvider", (provider) ->
  provider.defaults.headers.common['X-CSRF-Token'] = $('meta[name=csrf-token]').attr('content')
])

# ...

```

<br>

This code adds a meta tag with our CSRF token to the HTTP request header by default everytime we make a request.  This allows our request to go through, while maintaining basic protection against CSRF attacks.

If we now try to create a post again, we should see the following:

**http://localhost:3000/#/post/new**

<img src="{{relative}}/images/bootstrapss5-5.jpg" class="img-polaroid">

<br>

If we navigate back to our homepage now, we will notice that the new post is now there as well.  

**http://localhost:3000/#/**

<img src="{{relative}}/images/bootstrapss5-6.jpg" class="img-polaroid">

<br>

Note that this did not require a separate reload of post data by our shared `postData` service, only a route change to our homepage.

<br>

## Promises with Angular's $q module

The next thing we will touch on is promises.  I had initially planned to jump right into update and delete functionality in this post, but before long I realized that we need to make some significant updates to the basic home and individual post page in preparation for that.  These changes are a worthwhile topic in themselves because, for the first time, we will need to touch on promises, which are critical for so many things.  Rather than get too deep and abstract on this topic, though, let's start with why they are useful right now.

In preparation for implementing edit functionality on our individual post pages, let's refactor our homepage to link to blog posts based on their actual Rails database IDs rather than on their index in the client-side array we have loaded them into.  We cannot count on Rails returning our results in the same order each time, nor can we safely assume that there are not additional posts now available that also affect the index of each post in the array.  So if we hit reload on our post page and the URL is based on an array index value, we may or may not be reloading the same page data.  Instead, we will rely on the database ID, which will ensure that the same, correct post loads each time.

**app/assets/templates/mainIndex.html**

```html

<!-- ... -->

<div class="container" style="margin-top: 40px">
  <h1 class="text-center">My blog</h1>
  <div class="row" ng-repeat="post in data.posts">
      <h2><a ng-click="viewPost(post.id)">{{ "{{ post.title"}} }}</a></h2>
      <p>{{ "{{ post.contents"}} }}</p>
  </div>
</div>

```

<br>

We are now passing the post's database ID to the `viewPosts` method rather than its `$index` value.  Next, let's update the `mainPost` template and `PostCtrl` controller:

**app/assets/templates/mainPost.html**

```html
<div class="navbar navbar-fixed-top">
  <div class="navbar-inner">
    <a href="#" class="brand">My blog</a>
    <ul class="nav pull-right">
      <li><a ng-click="navNewPost()">New Post</a></li>
    </ul>
  </div>
</div>

<div class="container" style="margin-top: 40px">
  <h1 class="text-center">{{ "{{ data.currentPost.title"}} }}</h1>
  <div class="row">
    <p>{{ "{{ data.currentPost.contents"}} }}</p>
  </div>
  <div class="row">
    <button class="btn btn-default" ng-click="editPost()">Edit</button>
  </div>
</div>

```

<br>

As you can see, we are now binding our title and contents to a new `currentPost` object rather than directly to our shared service.  We also added an edit button at the bottom of the page.  We will come back and implement that in the next part of this tutorial.

Next, let's update our `PostCtrl` controller:

**app/assets/javascripts/Controllers/main/mainPostCtrl.js.coffee**

```coffeescript
@PostCtrl = ($scope, $routeParams, postData) ->

  $scope.data =
    postData: postData.data
    currentPost:
      title: 'Loading...'
      contents: ''

  $scope.data.postId = $routeParams.postId

  postData.loadPosts()

  $scope.prepPostData = ->
    post = _.findWhere(postData.data.posts, { id: parseInt($scope.data.postId) })
    $scope.data.currentPost.title = post.title
    $scope.data.currentPost.contents = post.contents

  $scope.navNewPost = ->
    $location.url('/post/new')

  $scope.navHome = ->
    $location.url('/')

  $scope.prepPostData()

@PostCtrl.$inject = ['$scope', '$routeParams', 'postData']
```

<br>

Here, we are setting up our `currentPost` object with a call to our `prepPostData` function.  This function leverages the [underscore.js](http://underscorejs.com) library to locate the correct post based on the `id` key in the array of hashes.

Looking at the above code, you will see that there is a small, but significant problem with it.  We are relying on a function to set up this data when the page loads rather than when the data loads, and while the data loading begins when the page loads, the data load is being done asynchronously.  We need to make sure that our data is loaded from server at the time `prepPostData` runs.  To ensure this order of operations, we will leverage Angular's promise functionality.

Angular implements a simple Promise API through its `$q` module.  This allows you to chain promises using the `then` method, which are called in order only at the point when the `resolve` method is called.

The way we will approach this problem is as follows:

1. Upon page load, we will set up a deferred promise to call `prepPostData` upon resolution.  This deferred promise will be passed to the `loadPosts` method on our `postData` service.

2. We will modify our `loadPosts` method to accept a deferred promise to be resolved at the time data successfully loads.  The method will check if a deferred promise exists and, if so, call resolve once the data is loaded (or immediately if the data is already loaded).

3. Upon resolution, our `prepPostData` method will be called, overwriting the placeholder data in the the `currentPost` object with the loaded post data.  Angular's bindings to that object will then automatically update the view.

The first step in implementing this is to change loadPosts() on our postData service to accept a deferred promise and, if one is passed, to resolve it at the point when data becomes available:

**app/assets/javascripts/Services/main/postData.js.coffee**

```coffeescript 

# ...

  postData.loadPosts = (deferred) ->
    if !postData.isLoaded
      $http.get('./posts.json').success( (data) ->
        postData.data.posts = data
        postData.isLoaded = true
        console.log('Successfully loaded posts.')
        if deferred
          deferred.resolve()
      ).error( ->
        console.error('Failed to load posts.')
        if deferred
          deferred.reject('Failed to load posts.')
      )
    else
      if deferred
        deferred.resolve()

# ...

```

<br>

As you can see, the `loadPosts` method now check for a deferred promise in our callbacks and, if it exists, resolves it or rejects it.  (We won't cover rejecting deferred promises right now, but it is an important topic for error recovery and messaging.)  

Next, we will quickly update our other post controllers to send `null` in place of a deferred promise since they don't rely on pre-loading to work:

**app/assets/javascripts/Controllers/main/mainIndexCtrl.js.coffee**

```coffeescript

# ...

  postData.loadPosts(null)

# ...
```

<br>

**app/assets/javascripts/Controllers/main/mainCreatePostCtrl.js.coffee**

```coffeescript

# ...

  postData.loadPosts(null)

# ...
```

<br>

Now we will update the `PostCtrl` controller to create a promise upon instantiation of the controller.  This will be resolved by the `loadPosts` function once data is loading, causing the controller to run its own `prepPostData` function.

**app/assets/javascripts/Controllers/main/mainPostCtrl.js.coffee**

```coffeescript
@PostCtrl = ($scope, $routeParams, $q, postData) ->

  $scope.data =
    postData: postData.data
    currentPost:
      title: 'Loading...'
      contents: ''

  $scope.data.postId = $routeParams.postId

  $scope.navNewPost = ->
    $location.url('/post/new')

  $scope.navHome = ->
    $location.url('/')

  # This will be run once the loadPosts successfully completes (or immediately
  # if data is already loaded)
  $scope.prepPostData = ->
    post = _.findWhere(postData.data.posts, { id: parseInt($scope.data.postId) })
    $scope.data.currentPost.title = post.title
    $scope.data.currentPost.contents = post.contents

  # Create promise to be resolved after posts load
  @deferred = $q.defer()
  @deferred.promise.then($scope.prepPostData)

  # Provide deferred promise chain to the loadPosts function
  postData.loadPosts(@deferred)


@PostCtrl.$inject = ['$scope', '$routeParams', '$q', 'postData']
```

<br>

First, we define our `prepPostData` function, which contains the logic that will set up the model for our view.  This is the same as before.  Next, we create a promise called `deferred`, and add a `then` statement to call our `prepPostData` method when the promise is resolved.  We then pass this deferred promise to our `loadPosts` method, which calls `resolve` upon successfully loading the data.  In this way, we are able to ensure that we are setting up the model only *after* we have loaded the necessary data from our server.  Angular's bindings then kick in, updating the view.

With that, we have a significantly more functional and resilient post view page, and we are now ready to implement update and delete functionality in our next post.

<br>

## Conclusion

That's it for part 5.  We can now create blog posts directly from the client side rather than only through the console.  And we have an update post page that will support eventually support permalinking, as well as populate itself asynchronously as data loads from the server.  In the next post, we will finish adding CRUD functionality and start to talk about polling the server for changes to stay in sync.

***I am currently working on part 6, which will finish adding CRUD functionality and maintain synchronization between the AngularJS client and Rails server.  [Stay tuned!](http://twitter.com/asandersn/)***

<br><br>

<div id="disqus_thread"></div>
<script type="text/javascript">
  /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
  var disqus_shortname = 'adamandersonblog';
  var disqus_identifier = '2013-11-20-bootstrapping-angular-rails-part-5';
  var disqus_title = 'Bootstrapping an AngularJS app in Rails 4.0 - Part 6';
  var disqus_url = 'http://asanderson.org/posts/2013/11/20/bootstrapping-angular-rails-part-5.html';

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
  var disqus_identifier = '2013-11-20-bootstrapping-angular-rails-part-5';
  var disqus_title = 'Bootstrapping an AngularJS app in Rails 4.0 - Part 5';
  var disqus_url = 'http://asanderson.org/posts/2013/11/20/bootstrapping-angular-rails-part-5.html';

  /* * * DON'T EDIT BELOW THIS LINE * * */
  (function () {
      var s = document.createElement('script'); s.async = true;
      s.type = 'text/javascript';
      s.src = '//' + disqus_shortname + '.disqus.com/count.js';
      (document.getElementsByTagName('HEAD')[0] || document.getElementsByTagName('BODY')[0]).appendChild(s);
  }());
</script>
    