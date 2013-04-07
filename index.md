---
layout: page
title: Liujianghu-刘江湖
---
{% include JB/setup %}

### Blog Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
<ul class='pager'>
        {% if paginator.total_pages > 1 %}
            <li class='first-page'><a href='/'>Frist</a></li>
            {% if paginator.previous_page %}
                {% if paginator.previous_page == 1 %}
                    <li><a href='/'>Pre</a></li>
                {% else %}
                    <li><a href='/page{{paginator.previous_page}}'>Pre</a></li>
                {% endif %}
            {% endif %}
            {% if 1 == paginator.page %}
                <li class='active'><a href='/'>1</a></li>
            {% else %}
                <li><a href='/'>1</a></li>
            {% endif %}
        {% endif %}
        {% for count in (2..paginator.total_pages) %}
            {% if count == paginator.page %}
                <li class='active'><a href='/page{{count}}'>{{count}}</a></li>
            {% else %}
                <li><a href='/page{{count}}'>{{count}}</a></li>
            {% endif %}
        {% endfor %}
        {% if paginator.next_page %}
            <li><a href='/page{{paginator.next_page}}'>Next</a></li>
        {% endif %}
        {% if paginator.total_pages > 1 %}
            <li class='last-page'><a href='/page{{paginator.total_pages}}'>Last</a></li>
        {% endif %}
    </ul>