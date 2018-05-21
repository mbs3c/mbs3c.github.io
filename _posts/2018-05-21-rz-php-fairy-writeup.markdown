---
layout: post
title:  "ringzer0team.com - PHP Fairy Writeup"
date:   2018-05-21 10:35:02 -0400
categories: ringzer0team.com writeup
---

This challenge initially presents a login screen which has a single password field as input, a login button, and another button which displays the underlying PHP source code (with the flag obfuscated). Taking a look at the code, password input is handled by $_GET['pass'], and there are several obstacles in the way which prevent access to the flag.

The conditions to retrieve the flag are as follows:
1. Line 3: Legal input consists of alphanumeric characters only
2. Line 8: Several comparisons (using both loose and strict operations) take place between input and a generated MD5 hash. This is supposed to be confusing, as the second checks after the or are the first ones in reverse. What it boils down to is that input cannot be identical to the MD5 hash, yet it must be equal (after type juggling occurs)
3. Line 10: Input value must equal the size of the MD5 hash - 32 chars (128 bits)

Reading through the PHP manual provides some valuable information:

["If you compare a number with a string or the comparison involves numerical strings, then each string is converted to a number and the comparison performed numerically."](http://php.net/manual/en/language.operators.comparison.php)

["When a string is evaluated in a numeric context, the resulting value and type are determined as follows. The value is given by the initial portion of the string. If the string starts with valid numeric data, this will be the value used. Otherwise, the value will be 0 (zero)."](http://php.net/manual/en/language.types.string.php#language.types.string.conversion)

With this knowledge in hand, we can observe that the MD5 hash begins with a zero: 'echo -n admin1674227342 | md5sum'
Therefore, we know that using a zero as input will pass both the strict and loose comparisons, leaving the final string length obstacle. This can be bypassed by inputting 32 zero characters, which doesn't change the actual value but does provide the necessary length to allow for bypass.


{% highlight php linenos %}
<?php
$output = "";
if (isset($_GET['code'])) {
  $content = file_get_contents(__FILE__);
  $content = preg_replace('/FLAG\-[0-9a-zA-Z_?!.,]+/i', 'FLAG-XXXXXXXXXXXXXXXXXXXXXXX', $content);
  echo '<div class="code-highlight">';
  highlight_string($content);
  echo '</div>';
}

if (isset($_GET['pass'])) {
  if(!preg_match('/^[^\W_]+$/', $_GET['pass'])) {
    $output = "Don't hack me please :(";
  } else {

    $pass = md5("admin1674227342");
    if ((((((((($_GET['pass'] == $pass)))) && (((($pass !== $_GET['pass']))))) || ((((($pass == $_GET['pass'])))) && ((($_GET['pass'] !== $pass)))))))) { // Trolling u lisp masta
      if (strlen($pass) == strlen($_GET['pass'])) {
        $output = "<div class='alert alert-success'>FLAG-XXXXXXXXXXXXXXXXXXXXXXX</div>";
      } else {
        $output = "<div class='alert alert-danger'>Wrong password</div>";
      }
    } else {
      $output = "<div class='alert alert-danger'>Wrong password</div>";
    }
  }
}
?>
{% endhighlight %}

