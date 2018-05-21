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

Cleaning this up, we have the following PHP code. This shows the same 'RandomClass' class as exists in the discovered serialized string. What if we could inject our own object with arbitrary values? In order to do that, we must first understand how the available PHP ['magic methods'](http://php.net/manual/en/language.oop5.magic.php) within the stack trace might assist us.

Walking through the PHP code, we first notice two private members (?). Immediately following that is the __construct() method. This magic method is called on each newly-created object, generally utilized for initialization of some sort. Inside the constructor, we see an empty array cast to an object, which is assigned to $this->uStruct.

From the PHP documentation, ["If a value of any other type is converted to an object, a new instance of the stdClass built-in class is created. If the value was NULL, the new instance will be empty. An array converts to an object with properties named by keys and corresponding values."](http://php.net/manual/en/language.types.object.php#language.types.object.casting)


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
