---
layout: page
title: Lake's blog
tagline: by Lake
---
{% include JB/setup %}

  {% for post in site.posts %}
    <div class="post">
        <div class="post-title">
            <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>
        </div>
        
        <div class="n-comments">
            165</div>
        <!-- shadow -->
        <div class="thumb-shadow">
            <!-- post-thumb -->
            <div class="post-thumbnail">
               Are you going to Scarborough Fair
                    Parsley, sage, rosemary and thyme
                    Remember me to one who lives there
                    He was once a true love of mine

                    Tell him to make me a cambric shirt
                    Parsley, sage, rosemary and thyme
                    Without no seams nor needle work
                    Then he'll be a true love of mine
            </div>
            <!-- ENDS post-thumb -->
            <div class="the-excerpt">
                <div class="left">
                    <strong>Posted on</strong> {{ post.date | date_to_string }} 
                </div>
                <div class="right">
                    <a href="{{ BASE_PATH }}{{ post.url }}" class="read-more link-button"><span>Read more</span></a>
                </div>
            </div>
            
        </div>
        <!-- ENDS shadow -->
    </div>
  {% endfor %}


