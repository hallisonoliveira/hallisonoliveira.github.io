---
layout: page
title: Lista de Posts
permalink: /posts
---
<div class="container">
    {% assign post_list_size = site.posts | size %}
    {%- if post_list_size > 0 -%}
    <ul>
            {% for category in site.categories %}
            <li>
                {{ category | first }}
                <ul>
                    {% for post in category.last reversed %}
                    <li><a href="{{ post.url }}">{{ post.date | date: "%d/%m/%Y" }} - {{ post.title }}</a></li>
                    {% endfor %}
                </ul>
            </li>
            {% endfor %}
    </ul>
    {% else %}
        <div style="text-align: center">
            Aguarde... Os posts estão sendo produzidos.
        </div>
    {% endif %}
</div>