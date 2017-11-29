---
layout: page
title: Tags
---

<div class="tags">
    <hr />

{% for tag in site.tags %}
{% capture tag_name %}{{ tag | first }}{% endcapture %}
<h2><i class="fa fa-tag"></i> <a href="{{site.url}}/tag/{{tag_name}}">{{tag_name}}</a></h2>
    <table>
        <tbody>
{% for post in site.tags[tag_name] %}
<tr>
    <td>
  <a href="{{ post.url }}">{{ post.title }}</a>
    </td>
    <td>
  <i class="fa fa-calendar"></i> {{ post.date | date_to_string }}
    </td>
</tr>
{% endfor %}
        </tbody>
    </table>
{% endfor %}

</div>
