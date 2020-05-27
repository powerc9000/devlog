# Cool Devlog

## Posts

{% for post in site.posts -%}
* [{{post.title}}]({{site.baseurl}}{{post.url}}) - <small>{{post.date}}</small>
{% endfor %}
