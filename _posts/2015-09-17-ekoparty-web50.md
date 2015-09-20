---
layout: post
title: Ekoparty 2015 Pre-ctf - Web 50
---

# EkoParty

## Web50

As we navigate to `http://challs.ctf.site:10000/hackersmarket/` we see a 'hackersmarket' where you can buy and sell exploits! Well, let us see if we can break in and steal the exploits ourselves shall we? :) As we navigate to the separate pages you can see the `p` parameter changing of the `index.php` file changing. The unconventional .tpl extension is being used to reference page content. This looks ripe for a local file inclusion attack! Typically this involves `../../../../../passwd` which should be displayed, but inclusion of and `..\` appears to cause a redirect to the home page. Looking some more we see `1NULLo3KCSKeCZeDQc7ZxY8xcbYiDGrnbY` for a bitcoin address. That `NULL` looks out of place, maybe a clue? Simply doing `http://challs.ctf.site:10000/hackersmarket/index.php?p=index.php` displays the page overlapped, so this indicates the code is being executed twice. If we look at the page source there is some PHP code:

```
<?php
// NULL Code Obfuscator
// www.null-life.com
include 'encoder.php';

error_reporting(0);
$code =
'GmjTplJrnv4Hd9uqLEm22igpg6kuJ9quCATTrlMu1/4SaZauTi7X0TRLp9VUftTTSASOrhZigOtTdfmuUy7TqgNvlOtTM9OpA2+U6wAhm+Eea936A2LUtXlz+YQaaNOmAHqB/hx926oDb5TrXy7UoF0p2q5SM86uFW+f/RYn0/V5LtOuUyqD7xRr07NTKYPvFGuAoRthnutdeoPiVDX583kE1+gaYpauTi7RoFwqg+8Ua9G1eQSa6FMmlecfa6zrC2eA+gAm1+gaYpanWi6IhFMu064WbZvhU2ia4hZRlOsHUZDhHXqW4Ad926xdIdf+EmmWrFo1+fNTa5/9Fi6IhFMu064WbZvhU2ia4hZRlOsHUZDhHXqW4Ad926ldIYPvFGuAoRthnutdeoPiVCfIhA4=';

$base = "\x62\x61\x73\x65\x36\x34\x5f\x64\x65\x63\x6f\x64\x65";
eval(NULLphp\getcode(basename(__FILE__), $base($code)));
?>
```

It looks like the NULL obfuscator is at work here with the `eval` with reference to `getcode` function. If we look at the inclusions there is `encoder.php`. If we include this through the vulnerability we can see the source code of the function:

```
<?php

namespace NULLphp;

$seed = 13;
function rand() {
    global $seed;

    return ($seed = ($seed * 127 + 257) % 256);
}

function srand($init) {
    global $seed;

    $seed = $init;
}

function generateseed($string) {
    $output = 0;

    for ($i = 0; $i < strlen($string); $i++) {
        $output += ord($string[$i]);
    }

    return $output;
}

function getcode($filename, $code) {
    srand(generateseed($filename));

    $result = '';
    for ($i = 0; $i < strlen($code); $i++) {
        $result .= chr(ord($code[$i]) ^ rand());
    }

    return $result;
}
```

A seed is generated using the filename, which is passed to the function call using `basename(__FILE__)` each provided code byte is then XORed with the generated seed with the resulting integer becoming a char with `chr()`. The resulting code is returned and then executed using `eval()`. If we change this eval to an echo for the `login.php` page we see this decoded script:

```
$ php login.php 
if (!empty($_POST['email']) && !empty($_POST['password'])) {
    $email = $_POST['email'];
    $pass  = $_POST['password'];

    // I can not disclose the real key at this moment
    // $key = 'random_php_obfuscation';
    $key = '';
    if ($email === 'admin@hackermarket.onion' && $pass === 'admin') {
        echo '<div class="alert alert-success" role="alert"><strong>well done!</strong> EKO{' . $key . '}</div>';
    } else {
        echo '<div class="alert alert-danger" role="alert"><strong>Oh snap!</strong> Wrong credentials</div>';
    }
} else {
    header('Location: index.php');
```

The resulting flag is commented out, being: `EKO{random_php_obfuscation}`
