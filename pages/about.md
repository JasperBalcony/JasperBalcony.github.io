---
layout: page
title: About
description: Personal blog using jekyll,which include most note from DevelopNote repoistor
keywords: jaseprxgwang
comments: true
menu: 关于
permalink: /about/
---



## 联系

* GitHub：[@JasperXgwang](https://github.com/JasperXgwang)
* 博客：[{{ site.title }}]({{ site.url }})
* CSDN：[Jasper](https://blog.csdn.net/weixin_37195606)

## Skill Keywords

#### Software Engineer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_software_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

#### Mobile Developer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_mobile_app_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

#### Windows Developer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_windows_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>
