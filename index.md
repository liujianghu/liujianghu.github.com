---
layout: page
title: Lake's blog
tagline: by Lake
---
{% include JB/setup %}

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


