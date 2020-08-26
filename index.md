---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

<ul class="posts">
	{% for post in site.posts %}
	<li>
		<span>{{ post.date | date_to_string }}</span> » <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a><br>
		<span style="padding-left:2em">
			{{ post.excerpt | remove: '<p>' | remove: '</p>' }}
		</span>
	</li>
	<br>
	{% endfor %}
</ul>