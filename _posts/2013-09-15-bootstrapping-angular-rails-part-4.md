---
layout: post
title: Bootstrapping an AngularJS app in Rails 4.0 - Part 4
category: posts
summary: Part four of this tutorial covers building a shared AngularJS service to access our Rails API.
socialsummary: Part four of this tutorial covers building a shared AngularJS service to access our Rails API.
socialimage: http://asanderson.org/images/AngularJS-Shield-large.png
tags: AngularJS Rails Tutorial
year: 2013
month: 9
day: 15
published: true
comments: true
---
***The full GitHub repository for this tutorial is available [here](https://github.com/asanderson15/rails-angular-tutorial)***


## Introduction

This is part four of my tutorial covering building an Angular and Rails blog application from scratch.  We will start where we left off in [part 3](http://asanderson.org/posts/2013/08/19/bootstrapping-angular-rails-part-3.html). In the last post, we set up a simple Rails API and used our Angular controller to access that API on the client side. In this post I will cover:

* Creating a shared AngularJS service to act as our model that:
    * Loads data from the Rails API on initial page load
    * Shares that data across all controllers
    * Does not require data to reload when switching pages

As before, this series assumes a certain basic understanding of both Rails and AngularJS.  If you need an introduction to Rails, I recommend checking out the excellent [Ruby on Rails Tutorial](http://ruby.railstutorial.org/) by Michael Hartl.  For an intro to AngularJS, I recommend checking out the [homepage tutorials](http://angularjs.org/) and, to go a bit deeper, the excellent [egghead.io tutorial videos](http://egghead.io/) by John Lindquist.

<br>

## Setting up a shared AngularJS service as our model

First off, let's talk for a second about why we are choosing to use a shared service as the place where our model lives.  An Angular service is a type of Angular object that is instantiated only once, no matter how many controllers or other objects inject it.  This means that any Angular controller that injects this service will access the same instance, allowing for data to be easily shared across multiple controllers.  This is not the only way for controllers to share or pass data in Angular.  Other options include broadcasting messages on `$rootScope` or using global variables.  But in my experience, shared services is easiest and the most scalable approach.

Setting up an Angular service is similar to setting up a controller or an application module.  In keeping with the application structure we set up in [part 1](http://asanderson.org/posts/2013/06/03/bootstrapping-angular-rails-part-1.html), create the new coffeescript file in the Services folder under main:

**app/assets/javascripts/Services/main/postData.js.coffeescript**

```coffeescript
angular.module('Blog').factory('postData', ['$http', ($http) ->

  postData =
    data:
      posts: [{title: 'My first post', contents: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec laoreet lobortis vulputate. Ut tempus, orci eu tempor sagittis, mauris orci ultrices arcu, in volutpat elit elit semper turpis. Maecenas id lorem quis magna lacinia tincidunt. In libero magna, pharetra in hendrerit vitae, luctus ac sem. Nulla velit augue, vestibulum a egestas et, imperdiet a lacus. Nam mi est, vulputate eu sollicitudin sed, convallis vel turpis. Cras interdum egestas turpis, ut vestibulum est placerat a. Proin quam tellus, cursus et aliquet ut, adipiscing id lacus. Aenean iaculis nulla justo.'}, {title: 'A walk down memory lane', contents: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin leo sem, imperdiet in faucibus et, feugiat ultricies tellus. Vivamus pellentesque iaculis dolor, sed pellentesque est dignissim vitae. Donec euismod purus non metus condimentum porttitor suscipit nibh tempor. Etiam malesuada elit in lectus pharetra facilisis. Fusce at nisl augue. Donec at est felis. Sed a gravida diam. Nunc nunc mi, egestas non dignissim et, porta aliquam ante.'}]

  console.log("Initialized postData.")

  return postData

])
```

<br>

<!--- more -->

This code leverages Angular's `factory` method to create the service.  The `factory` method allows us to create and initialize an object (our service) and then returns it.  This method is called and the actual initialization happens only once, the first time the service is injected into a controller.  The first argument, `postData`, is the name of our service.  The second allows us to inject one or more other Angular modules.  In this case, we will start by injecting only `$http`, which will allow us to implement data loading from our API below.

In this case, we initialized the `postData` object to contain a `posts` object, containing an array of post objects.  We populate this with the sample data we used before in part 2.  After initialization, we log a notification to the console and then return our `postData` object.

Now that we have our service, let's go ahead and inject it into our controllers and then try accessing the sample data we just set up.

**app/assets/javascripts/Controllers/main/mainIndexCtrl.js.coffeescript**

```coffeescript
@IndexCtrl = ($scope, $location, $http, postData) ->

  $scope.data = postData.data

  loadPosts = ->
    $http.get('./posts.json').success( (data) ->
      $scope.data.posts = data
      console.log('Successfully loaded posts.')
    ).error( ->
      console.error('Failed to load posts.')
    )

  # Temporarily disable loading posts from the API
  # loadPosts()

  $scope.viewPost = (postId) ->
    $location.url('/post/'+postId)

```

<br>

**app/assets/javascripts/Controllers/main/mainPostCtrl.js.coffeescript**

```coffeescript
@PostCtrl = ($scope, $routeParams, postData) ->

  $scope.data =
    post: postData.data.posts[0]

  $scope.data.postId = $routeParams.postId
  console.log($routeParams)

```

<br>

In the main `IndexCtrl`, we start by injecting our new `postData` service as a dependency.  We then set our `$scope.data` object to point to the `data` object from our `postData` service.  We temporarily disable our `loadPosts()` function from overwriting with data from the API.  With these small tweaks, both of our controllers are now leveraging the same set of data from the shared `postData` service.  You can prove this to yourself by checking the console logs.  As you will see, when you switch between the index page and the post page, the "Successfully loaded posts." message will appear only once.

<br>

## Loading API data in a service

The next step is to discard the sample data and to use the new `postData` service to pull blog post data from our Rails API.  We will do this in a very similar way to how we loaded data directly in our controller.  First, let's cut the `loadPosts` function from our `IndexCtrl` controller and move it to our `postData` service:

**app/assets/javascripts/Services/main/postData.js.coffeescript**

```coffeescript
angular.module('Blog').factory('postData', ['$http', ($http) ->

  postData =
    data:
      posts: [{title: 'My first post', contents: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec laoreet lobortis vulputate. Ut tempus, orci eu tempor sagittis, mauris orci ultrices arcu, in volutpat elit elit semper turpis. Maecenas id lorem quis magna lacinia tincidunt. In libero magna, pharetra in hendrerit vitae, luctus ac sem. Nulla velit augue, vestibulum a egestas et, imperdiet a lacus. Nam mi est, vulputate eu sollicitudin sed, convallis vel turpis. Cras interdum egestas turpis, ut vestibulum est placerat a. Proin quam tellus, cursus et aliquet ut, adipiscing id lacus. Aenean iaculis nulla justo.'}, {title: 'A walk down memory lane', contents: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin leo sem, imperdiet in faucibus et, feugiat ultricies tellus. Vivamus pellentesque iaculis dolor, sed pellentesque est dignissim vitae. Donec euismod purus non metus condimentum porttitor suscipit nibh tempor. Etiam malesuada elit in lectus pharetra facilisis. Fusce at nisl augue. Donec at est felis. Sed a gravida diam. Nunc nunc mi, egestas non dignissim et, porta aliquam ante.'}]

  postData.loadPosts = ->
    $http.get('./posts.json').success( (data) ->
      postData.data.posts = data
      console.log('Successfully loaded posts.')
    ).error( ->
      console.error('Failed to load posts.')
    )

  return postData

])
```

<br>

As you can see, I made a few modifications to the function in order to make it work in the service context.  First, `loadPosts` is defined as a function of the `postData` object.  This is necessary because only the object is returned from the Angular factory.  Similarly, once the post data is retrieved from the server, the array we set is `postData.data.posts`.

Now all that is left is to call `loadPosts`.  Let's invoke it from our main `IndexCtrl` controller upon load.

**app/assets/javascripts/Controllers/main/mainIndexCtrl.js.coffeescript**

```coffeescript
@IndexCtrl = ($scope, $location, $http, postData) ->

  $scope.data = postData.data

  postData.loadPosts()

  $scope.viewPost = (postId) ->
    $location.url('/post/'+postId)

```

<br>

If you visit **localhost** in your browser, you should see that we are back to properly loading our sample posts from the database and displaying them.

**http://localhost:3000/**

<img src="{{relative}}/images/bootstrapss3-2.jpg" class="img-polaroid">

<br>

You should also notice a message in the console reading "Successfully loaded posts."

<br>

## Sharing data across controllers without reloading

Part of the value of a shared service is that it avoids reloading data even when switching views.  One problem you will notice with the current setup is that it fails to do that.  Try opening a post and then returning to the homepage.  You will see in the console that data is reloading each time the `IndexCtrl` controller is invoked.  The second problem you will notice is that, if you start on the post page (or hit refresh on it), the `loadPosts` call is never invoked, and no data will be loaded from the database (instead, the sample data will be loaded).  There is an easy fix to both of these issues.

First, let's have `postData` track whether post data has already been loaded and load posts only if they have not yet been loaded.  (And, while we are at it, let's replace our sample data with the "Loading" title we set up earlier.)

**app/assets/javascripts/Services/main/postData.js.coffeescript**

```coffeescript
angular.module('Blog').factory('postData', ['$http', ($http) ->

  postData =
    data:
      posts: [{title: 'Loading', contents: ''}]
    isLoaded: false

  postData.loadPosts = ->
    if !postData.isLoaded
      $http.get('./posts.json').success( (data) ->
        postData.data.posts = data
        postData.isLoaded = true
        console.log('Successfully loaded posts.')
      ).error( ->
        console.error('Failed to load posts.')
      )

  return postData

])
```

<br>

Next, let's add a call to `loadPosts` in the `PostCtrl` controller:

**app/assets/javascripts/Controllers/main/mainPostCtrl.js.coffeescript**

```coffeescript
@PostCtrl = ($scope, $routeParams, postData) ->

  $scope.data =
    post: postData.data.posts[0]

  postData.loadPosts()

  $scope.data.postId = $routeParams.postId

```

<br>

If you visit **http://localhost:3000** in your browser, you can confirm that this change addresses the first problem of the data reloading each time you switch back to the main `IndexCtrl` controller.  However, this has not resolved the second problem.  If you visit the posts page and hit refresh, you will see "0 - Loading" as the page title.  But if you check the console, you will see that the data has successfully loaded from the API.  So what is going on?

The explanation has to do with setting `$scope.data.post` as a specific post object before loading posts.  This is what happens when the code executes:

1. Because `postData` is a dependency of `PostCtrl`, Angular initiates `postData` before `PostCtrl`.  So, the `postData` service is created with `postData.data.posts[0]` being an object with a title of "Loading" and no contents.

2. When `PostCtrl` is created, it sets `$scope.data.post` to point to the specific `postData.data.posts[0]` object.

3. The `loadPosts` function is called on `postData`, pulling the data from the API.

4. When the API returns the posts from the database and the callback is invoked, rather than modifying the `postData.data.posts` array, the callback replaces the entire `posts` array with a new array.  However, the PostCtrl's `$scope.data.post` object is still pointing to the old placeholder "loading" object.

This type of error can come up in many contexts when using Angular, and it is important to watch out for it while coding.  It is also very often the culprit when something does not seem to be updating as expected.  The fix is fairly simple, and there are two options.  You can either have the controller point to the parent of the object that will be reassigned, or you can change the value(s) of the object rather than reassigning the object in its entirety.  We will cover this a bit more in the next section, when we dive deeper into adding additional functionality to the API.

In this case, let's change our controller to refer to the parent object rather than directly to the array.  We will then specify the correct post from the `posts` array in the view template itself using our route parameter.

**app/assets/javascripts/Controllers/main/mainPostCtrl.js.coffeescript**

```coffeescript
@PostCtrl = ($scope, $routeParams, postData) ->

  $scope.data =
    postData: postData.data

  postData.loadPosts()

  $scope.data.postId = $routeParams.postId

```

<br>

**app/assets/templates/mainPost.html**

```html
<h1 class="text-center">{{ "{{data.postId"}} }} - {{ "{{data.postData.posts[data.postId].title"}} }}</h1>
<div class="row">
    <p>{{ "{{data.postData.posts[data.postId].contents"}} }}</p>
</div>
```

<br>

You can now hit refresh on any page and things should work as expected.

<br>

## A quick note about dependency injection and deployment

Before closing, I should mention one more thing about deploying into production with AngularJS.  While everything we have done up to this point has worked great in a test/development environment, as currently structured, you will get errors in production due to the way Rails (and other frameworks) minify Javascript code.  When minified, the Controllers' initialization variables will no longer be in their verbose form (`$scope`, `$routeParams`, etc. are minified to `a`, `b`, etc.).  Without their verbose names, Angular can no longer tell what dependencies to inject.

Fortunately, the workaround for this is simple.  At the end of our controller files, we will add a line to manually inject the dependencies.

**app/assets/javascripts/Controllers/main/mainIndexCtrl.js.coffeescript**

```coffeescript
@IndexCtrl = ($scope, $location, $http, postData) ->

  $scope.data = postData.data

  postData.loadPosts()

  $scope.viewPost = (postId) ->
    $location.url('/post/'+postId)

@IndexCtrl.$inject = ['$scope', '$location', '$http', 'postData']

```

<br>

**app/assets/javascripts/Controllers/main/mainPostCtrl.js.coffeescript**

```coffeescript
@PostCtrl = ($scope, $routeParams, postData) ->

  $scope.data =
    postData: postData.data

  postData.loadPosts()

  $scope.data.postId = $routeParams.postId

@PostCtrl.$inject = ['$scope', '$routeParams', 'postData']

```

<br>

This additional line, which identifies the dependencies using strings that will not be minified, will ensure that the proper dependencies are injected.

<br>

## Conclusion

That's it for part 4.  We now have a shared AngularJS service that allows us to display multiple pages without reloading data from the server.  This works well if the client only needs to read data from the server.  But things get more complicated when the client also has access to create, update, and destroy data on the server.  In the next part, we will build on this shared AngularJS service to add additional CRUD functionality and keep the client and server in sync. 

***Find this helpful? Check out [Part 5](http://asanderson.org/posts/2013/11/20/bootstrap-angular-rails-part-5.html). And [follow me](http://twitter.com/asandersn/) on Twitter for more updates.***

<br><br>

<div id="disqus_thread"></div>
<script type="text/javascript">
  /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
  var disqus_shortname = 'adamandersonblog';
  var disqus_identifier = '2013-08-19-bootstrapping-angular-rails-part-4';
  var disqus_title = 'Bootstrapping an AngularJS app in Rails 4.0 - Part 4';
  var disqus_url = 'http://asanderson.org/posts/2013/09/15/bootstrapping-angular-rails-part-4.html';

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
  var disqus_identifier = '2013-09-15-bootstrapping-angular-rails-part-4';
  var disqus_title = 'Bootstrapping an AngularJS app in Rails 4.0 - Part 4';
  var disqus_url = 'http://asanderson.org/posts/2013/09/15/bootstrapping-angular-rails-part-4.html';

  /* * * DON'T EDIT BELOW THIS LINE * * */
  (function () {
      var s = document.createElement('script'); s.async = true;
      s.type = 'text/javascript';
      s.src = '//' + disqus_shortname + '.disqus.com/count.js';
      (document.getElementsByTagName('HEAD')[0] || document.getElementsByTagName('BODY')[0]).appendChild(s);
  }());
</script>
    