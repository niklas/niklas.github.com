---
layout: post
title: "Arranged marriage between Draper and RJS"
date: 2012-02-22 23:23
comments: true
categories: [Ruby]
---

How to solve the problem of Javascript-rich interfaces with a Rails backend.

<!--more-->

RJS is dead
-----------

Do you remember RJS? It was introduced somewhere around version 1 of rails (or
even earlier, please correct me) and provided a template language in pure ruby
to respond to AJAX actions.


{% codeblock app/views/posts/update.js.rjs lang:ruby %}
page.replace_html 'posts', :partial => 'list'
page.visual_effect :highlight, 'posts'
{% endcodeblock %}

This was still during the benign reign of [Prototype](http://www.prototypejs.org/)
and the template would have been called `update.rjs`. There also was a way of
rendering Javascript directly from your controller actions:

{% codeblock app/controllers/posts_controller.b lang:ruby %}
class PostController < ApplicationController # no inherited resources or similar
  def create
     # update the model ..
     if request.rjs?
       render :update do |page|
         page['posts_count'].replace Post.count # this would target element #post_count
       end
     end
  end
end
{% endcodeblock %}

The significant part here is the `page` object, which magically generated
Prototype-JS for us.  But over time, RJS became deprecated. I did not follow
the proceedings, so I do not know the reasons. Maybe it tempted to pinch the MVC
layer a little bit too much (Controller example above) or with the rise of
[jQuery](http://jquery.com/) (yeah!) it lost its primer target. Or there was to 
much magic involved.

I admit, if not properly cleaning up after every commit, RJS templates could
become a mess. Additionally, the interaction with helpers was strange. You
could call a helper on the `page` object, but from within this helper, the page
object was only accessible through hoops. So no real refactoring was possible.

Until now!

Long live RJS!
--------------

I think the current approach of generating JS from Rails is tedious.
`lala.js.coffee.erb` looks nice in theory, but in practice this is a mess. My
HAML-coddled eyes don't want to see no angle brackets no more! Manual escaping of HTML?
Letting Rails serve only JSON on the other hand makes a full-blown Rails stack kind of
obsolete.

With the recent occurrence of Presenters like [Draper](https://github.com/jcasimir/draper) or [your own quickly setup presenter](http://railscasts.com/episodes/287-presenters-from-scratch), it now becomes possible to write Rails applications controller their browsed views by Javascript. I will show my findings with the first, but it may easily achieved with every other class following decorator pattern (TM).


Add draper and versatile-rjs to your Gemfile to resurrect that magic zombie.

{% codeblock Gemfile lang:ruby %}
gem 'draper'
gem 'versatile_rjs', :git => 'git://github.com/niklas/versatile_rjs.git
{% endcodeblock %}

Open a bottle of champagne, dim the light, and initialize first contact.

{% gist 1888149 %}

Put this into an initializer or your favored monkey patch cage and require it
once during application startup.

And it may prevail
------------------

As demonstrated in the comment of above gist, you can still know extend the
`page` proxy, but use the presenter around it. And because it uses jQuery (of
course), the calls to it can be chained. In our code we use symbol-based
lookup for elements on the page. Here is some example code; some endpoints are
still missing, but you'll get the idea.
 


{% codeblock application_decorator.rb lang:ruby %}
class ApplicationDecorator < Draper::Base
  # You may implement #selector_for in your subclass (don't forget to call super in the else)
  def selector_for(name, resource=nil, *more)
    case name
    when :errors_for
      %Q~#{selector_for(:form_for, resource)} .errors~
    when :form_for
      if resource.to_key
        %Q~form##{h.dom_id resource, :edit}~
      else
        %Q~form##{h.dom_id resource}~
      end
    else
      raise ArgumentError, "cannot find selector for #{name}"
    end
  end

  # select a specific element on the page.
  def select(*a)
    page.select selector_for(*a)
  end

  def remove(*a)
    select(*a).remove()
  end

  # append validation errors for given `resource` to its form
  def insert_errors_for(resource)
    select(:errors_for, resource).remove()
    select(:form_for, resource).append errors_for(resource)
  end
end
{% endcodeblock %}

{% codeblock fnord_decorator.rb lang:ruby %}
class FnordDecorator < ApplicationDecorator
  def update_box_for(resource)
    select(:box, resource).html render_the_resource_with_partial_or_whatever
  end
end
{% endcodeblock %}

{% codeblock create.js.rjs lang:ruby %}
page.decorate resource do |fnord|
  if resource.errors.empty?
    fnord.update_box_for(resource)
    fnord.remove :form_for, resource
  else
    fnord.insert_errors_for(resource)
  end
end
{% endcodeblock %}

So...
-----

There we go, finally a JS generating interface that looks clean, removes
duplication and uses all the Rails' features.
