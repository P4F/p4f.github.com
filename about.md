---
layout: page
title: About
permalink: /about/
---

{% for uhh in site.members %}
{% assign member = uhh[1] %}
<div id="{{ member.display_name }}" class="member">
<p></p>
<a href="{{ member.web }}">
<img class="before" src="http://www.gravatar.com/avatar/{{ member.gravatar }}?s=150"alt="{{ member.display_name }}" width="150" height="150"></a>
<h2>{{ member.display_name }}, {{ member.title }}</h2>
<ul>
<li>Twitter: <a href="https://twitter.com/{{ member.twitter }}">@{{ member.twitter }}</a></li>
{% if member.web %}<li>Web: <a href="{{ member.web }}">{{ member.web }}</a></li>{% endif %}
{% if member.email %}<li>Email: <a href="mailto:{{ member.email }}">{{ member.email }}</a></li>{% endif %}
{% if member.github %}<li>Github: <a href="https://github.com/{{ member.github }}">{{ member.github }}</a></li>{% endif %}
</ul>
{% if member.bio == blank or member.bio == nil %}{% else %}<p>{{ member.bio }}</p>{% endif %}
</div>
{% endfor %}