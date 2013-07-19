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

```rails
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

Once the gem is installed, add the following code to your *application.css* and *application.js* files:

**app/assets/javascripts/application.js**

```javascript
//= require twitter/bootstrap
```

**app/assets/stylesheets/application.css**

```css
*= require twitter/bootstrap
```

## Conclusion

I hope this is useful for other developers trying to experiment with the new Bootstrap 3.0.0.  If you have any comments or questions, don't hesitate to contact me on [Twitter](http://twitter.com/asandersn/).

