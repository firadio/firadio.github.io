---
layout: page
title: About
description: 打码改变世界
keywords: Xiang Xisheng, 项希盛
comments: true
menu: 关于
permalink: /about/
---

大家好，我是一名热爱编程的85后，出生于1988年，毕业于2009年。我在编程的道路上不断探索、学习和进步，希望能成为一名优秀的程序员。

2017年，我创立了温州飞儿云，提供云主机服务。在这个过程中，我学习了很多关于云计算和网络技术的知识，也结识了很多志同道合的朋友。飞儿云的理念是为用户提供高品质的云计算服务，让更多的人享受到云计算的便利。

我相信，编程是一项有趣而且充满挑战的工作。通过编程，我们可以创造出各种有用的工具和应用程序，让人们的生活更加便捷和高效。

作为一名程序员，我一直保持着学习和探索的心态，不断提高自己的编程能力和专业水平。我相信只有持续不断地学习和创新，才能跟上时代的步伐，迎接未来的挑战。感谢您花时间阅读我的个人介绍，期待能有机会与大家共同探讨、学习和成长。

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'mazhuang.org' %}
<li>
微信公众号：<br />
<img style="height:192px;width:192px;border:1px solid lightgrey;" src="{{ site.url }}/assets/images/qrcode.jpg" alt="闷骚的程序员" />
</li>
{% endif %}
</ul>


## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
