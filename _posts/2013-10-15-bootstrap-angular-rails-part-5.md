---
layout: post
title: Bootstrapping an AngularJS app in Rails 4.0 - Part 5
category: posts
summary: Part five of this tutorial covers building a shared AngularJS service to access our Rails API.
socialsummary: Part five of this tutorial covers building a shared AngularJS service to access our Rails API.
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

This is part four of my tutorial covering building an Angular and Rails blog application from scratch.  We will start where we left off in [part 4](http://asanderson.org/posts/2013/09/15/bootstrapping-angular-rails-part-4.html). In the last post, we set up a shared AngularJS service that allowed a single point of access to API data for all client-side Angular controllers. In this post, we will continue to build out this shared service, implementing:

* Functionality to create a new post
* The ability to update existing posts
* The ability to delete existing posts
* Partial reloading and polling to stay in sync

As before, this series assumes a certain basic understanding of both Rails and AngularJS.  If you need an introduction to Rails, I recommend checking out the excellent [Ruby on Rails Tutorial](http://ruby.railstutorial.org/) by Michael Hartl.  For an intro to AngularJS, I recommend checking out the [homepage tutorials](http://angularjs.org/) and, to go a bit deeper, the excellent [egghead.io tutorial videos](http://egghead.io/) by John Lindquist.

<br>

## Creating new posts from the client



### New post controller and template



### Rails API implementation



### AngularJS shared service



### Bringing it all together



## Updating and deleting existing posts from the client



### Edit post controller and template



### Rails API implementations



### AngularJS shared service



### Wiring everything together



## Partial reloads and polling for changes


<br>

## Conclusion

That's it for part 4.  We now have a shared AngularJS service that allows us to display multiple pages without reloading data from the server.  This works well if the client only needs to read data from the server.  But things get more complicated when the client also has access to create, update, and destroy data on the server.  In the next part, we will build on this shared AngularJS server to add additional CRUD functionality and keep the client and server in sync. 

***I am currently working on part 5, which will add additional CRUD functionality and maintain synchronization between the AngularJS client and Rails server.  [Stay tuned!](http://twitter.com/asandersn/)***

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
    