---
layout: post
title:  "Writeup - ringzer0team.com challenges (web 41) - 'Serial Killer!'"
date:   2018-05-21 11:14:41 -0400
categories: ringzer0team.com writeup
---
![Challenge Intro]({{ site.baseurl }}/images/rz-serial-init.png)
This challenge initially displays a timestamp on the page. Digging into the request being sent, we find a base64 encoded PHP object:

![Challenge Intro]({{ site.baseurl }}/images/rz-serial-base64.png)

![Challenge Intro]({{ site.baseurl }}/images/rz-serial-stack-trace.png)
{% highlight php linenos %}
{% endhighlight %}

