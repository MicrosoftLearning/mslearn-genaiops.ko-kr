---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
---

# Microsoft Learn - 실습 연습

다음 연습은 [Microsoft Learn](https://docs.microsoft.com/training/) 학습을 지원하기 위해 고안되었습니다.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| |
| --- | --- | 
{% for activity in labs  %}| [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
