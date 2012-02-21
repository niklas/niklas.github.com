---
layout: post
title: "Hello World"
date: 2012-02-21 02:30
comments: true
categories: Gibberish
---

In this blog, I am going to publish my deepest feelings about code, usually
guarded by a cataract of words supporting my holy opinion.

<!--more-->

## Sandbox / Cheatsheet / Testbed

#Code

Why isn't that in the stdlib?

{% codeblock slashing Pathnames lang:ruby %}
class Pathname
  alias_method, :/, :join
  # vim highlighting is confused by unbalanced underscores_
end
{% endcodeblock %}

Because I did not supply a patch, that's why!

#Gist

Trying to define an eigenclass method in ruby keeping closure - cannot remember
why or if it's working.

{% gist 1203256 %}
