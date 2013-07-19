---
layout: post
title: Adding Bootstrap 3.0.0 WIP to your Rails app
category: posts
summary: By adding the Rails-Bootstrap-Three gem to your Rails project, you can quickly include Bootstrap 3.0.0 WIP to your web application.
socialsummary: By adding the Rails-Bootstrap-Three gem to your Rails project, you can quickly include Bootstrap 3.0.0 WIP to your web application.
socialimage: http://asanderson.org/images/twitter-bird-blue-on-white.png
tags: Rails CSS
year: 2013
month: 7
day: 18
published: true
comments: true
---

As I have begun to include Bootstrap 3.0.0 (and especially it's new responsive mobile framework) into my web application development, it has become clear that someone needs to DRY this process up.  So I quickly built a simple gem to do this: [Bootstrap-Rails-Three](http://github.com/asanderson15/bootstrap-rails-three/).  

Version 3.0.0 of Bootstrap contains some really powerful new capabilities, as well as some good-looking updates to the CSS itself.  That said, it should be noted that **v.3.0.0 is a work in progress and is subject to bugs and/or breaking changes.  It should be used in production apps at your own peril.**

## Installation

Installation is simple.  Just add this line to your application's Gemfile:

<!--- more -->

```ruby
    gem 'bootstrap-rails-three', :github => 'asanderson15/bootstrap-rails-three', :branch => 'master'
```

And then execute:

```bash
    $ bundle install
```

Or install it yourself as:

```bash
    $ gem install bootstrap-rails-three
```

## Usage

Once the gem is installed, add the following code to your *application.css* and *application.js* files:

**app/assets/javascripts/application.js**

```javascript
//= require twitter/bootstrap
```

**app/assets/stylesheets/application.css**

```css
*= require twitter/bootstrap
```

## Is this useful for you?

I hope this is useful for other developers trying to experiment with the new Bootstrap 3.0.0.  If you have any comments or questions, don't hesitate to contact me on [Twitter](http://twitter.com/asandersn/). If people show a lot of interest, I will try to be more diligent about keeping it up to date with the latest updates to Bootstrap as it continues towards a release version.

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