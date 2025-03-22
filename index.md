---
title: Esercizi di Knowledge Mining di Azure
permalink: index.html
layout: home
---

# Esercizi di Knowledge Mining di Azure

Gli esercizi seguenti sono progettati a supporto dei moduli su Microsoft Learn.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %} {% if activity.url contains 'ai-foundry' %} {% continue %} {% endif %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
