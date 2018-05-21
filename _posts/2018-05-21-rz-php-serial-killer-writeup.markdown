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

Cleaning this up, we have the following PHP code. This shows the same 'RandomClass' class as exists in the discovered serialized string. What if we could inject our own object with arbitrary values? Without going into too much detail (recommended reading links to follow at the end of this post), we can take advantage of the __construct() ('magic method')[http://php.net/manual/en/language.oop5.magic.php]

{% highlight php linenos %}
<?php

class RandomClass {

    private static $instance;
    private $uStruct;
    
    public function __construct() {
        $this->uStruct = (object) array();
    }
    
    public static function GetInstance() {
        if (!isset(self::$instance)) {
            self::$instance = new self();
        }
        return self::$instance;
    }

    public function __set($key, $value) {
        $this->uStruct->$key = $value;
    }

    public function __get($key) {
        return $this->uStruct->$key;
    }
    
    public function DoAction() {
        $action = $this->uStruct->action;
        $this->$action();
    }

    public function GetCurrentDate() {
        GetCurrentDate($this->uStruct);
    }

    public function ShowFlag() {
        if ($this->uStruct->time !== null && $this->uStruct->flag == 'Please?') {
            ShowFlag($this->uStruct);
        }
    }

    public function GetOutput() {
        return $this->uStruct->output;
    }
}
?>
{% endhighlight %}

