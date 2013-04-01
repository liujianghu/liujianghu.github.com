---
layout: page
title: Hello World!
tagline: Supporting tagline
---
{% include JB/setup %}

Read [Jekyll Quick Start](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)

Complete usage and documentation available at: [Jekyll Bootstrap](http://jekyllbootstrap.com)

## Update Author Attributes

In `_config.yml` remember to specify your own data:
    
    title : My Blog =)
    
    author :
      name : Name Lastname
      email : blah@email.test
      github : username
      twitter : username

The theme should reference these variables whenever needed.
    
## Sample Posts

This blog contains sample posts which help stage pages and blog data.
When you don't need the samples anymore just delete the `_posts/core-samples` folder.

    $ rm -rf _posts/core-samples

Here's a sample "posts list".

<div id="posts">
  {% for post in site.posts %}
    <div class="post">
        <h1>
            <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h1>
        <div class="n-comments">
            165</div>
        <!-- shadow -->
        <div class="thumb-shadow">
            <!-- post-thumb -->
            <div class="post-thumbnail">
                <!-- meta -->
                <ul class="meta">
                    <li><strong>Posted on</strong> {{ post.date | date_to_string }} </li>
                    <li><strong>By</strong> <a href="#">Ansimuz</a></li>
                    <li><strong>Posted in</strong>
                        <div class="meta-tags">
                            <a href="#">Webdesign</a> <a href="#">Code</a> <a href="#">Photo</a>
                        </div>
                    </li>
                </ul>
                <!-- ENDS meta -->
                <a href="single.html" class="cover">
                    <img src="img/dummies/596x270.gif" alt="Feature image" /></a>
            </div>
            <!-- ENDS post-thumb -->
            <div class="the-excerpt">
                Pellentesque habitant 
            </div>
            <a href="{{ BASE_PATH }}{{ post.url }}" class="read-more link-button"><span>Read more</span></a>
        </div>
        <!-- ENDS shadow -->
    </div>
  {% endfor %}
</div>


## To-Do

This theme is still unfinished. If you'd like to be added as a contributor, [please fork](http://github.com/plusjade/jekyll-bootstrap)!
We need to clean up the themes, make theme usage guides with theme-specific markup examples.


