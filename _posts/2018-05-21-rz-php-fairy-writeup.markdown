---
layout: post
title:  "ringzer0team.com - PHP Fairy Writeup"
date:   2018-05-21 10:35:02 -0400
categories: ringzer0team.com writeup
---
Testing..

{% highlight php %}
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

