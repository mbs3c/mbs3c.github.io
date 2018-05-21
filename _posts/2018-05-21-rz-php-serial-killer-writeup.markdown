---
layout: post
title:  "Writeup - ringzer0team.com challenges (web 41) - 'Serial Killer!'"
date:   2018-05-21 11:14:41 -0400
categories: ringzer0team.com writeup
---
![Challenge Intro]({{ site.baseurl }}/images/rz-serial-init.png)
This challenge initially displays a timestamp on the page. Digging into the request being sent, we find a base64 encoded string in our GET request:

![Challenge Intro]({{ site.baseurl }}/images/rz-serial-base64.png)

When decoded, this is immediately recognizable as a serialized PHP object:

{% highlight php %}
O:11:"RandomClass":1:{s:20:"RandomClassuStruct";O:8:"stdClass":1:{s:6:"action";s:14:"GetCurrentDate";}}
{% endhighlight %}

What happens if we add a single character anywhere within the "?o=" value? A wild stack trace appears:

![Challenge Intro]({{ site.baseurl }}/images/rz-serial-stack-trace.png)
{% highlight php linenos %}
{% endhighlight %}

