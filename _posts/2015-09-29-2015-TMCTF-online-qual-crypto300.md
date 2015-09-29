---
layout: post
title: TMCTF Qualification Round 2015 - Crypto 300
---

Description for this challenge was:

```
What is the most famous default password of "keystore"?
```

along with the provided password protected file *level1.zip*.

Googling for *famous default password keystore* gave us the following entry at the first page of results:

```
Setting a Keystore Password - JXplorer (jxplorer.org/help/Setting_a_Keystore_Password.htm)

The cacerts keystore file has a default password of changeit; the clientcerts keystore file has a default password of passphrase. This option allows you to set a ...
```


Trying to open the password protected file *level1.zip* with the password *changeit* was a success. Inside of it there were three files:

```
level2.file.encrypted
level2.key.encrypted
Readme.txt
```

Content of file *Readme.txt* was as follows:

```
Welcome to the published key quest!
Your Quest is to find the published key!
 
You have a encrypted file and a encrypted key.
 - levelX.file.encrypted
 - levelX.key.encrypted
 
If you can find the key for decrypting "levelX.key.encrypted", then you can decrypt the file.
 
We encrypted the file using the following command.
>openssl enc -aes-256-cbc -salt -base64 -in ./levelX+1.zip -out levelX.file.encrypted -pass file:./levelX.key
 
We also encrypted the key using the following command.
>openssl rsautl -encrypt -inkey ThePublishedPublicKey.pem -pubin -in levelX.key | openssl base64 >levelX.key.encrypted
 
If you can decrypt "levelX.file.encrypted", you can get a next level zip file.
All zip files are encrypted by the level 1 quest's answer.
 
========================================================================
Level 2 quest's hint :
The key of the most famous "Snakeoil" CA.
========================================================================
 
Have fun!
```

Googling for *Snakeoil CA ext:key* resulted with the following entry at the first page:

```
libapache-mod-ssl/snakeoil-ca-rsa.key at master ... - GitHub (https://github.com/...mod-ssl/.../snakeoil-ca-rsa.key)

Contribute to libapache-mod-ssl development by creating an account on GitHub.
```

Using that same file to decrypt the Level2 files produced the following results:

```
$ wget https://raw.githubusercontent.com/ByteInternet/libapache-mod-ssl/master/pkg.sslcfg/snakeoil-ca-rsa.key
--2015-09-29 14:09:00--  https://raw.githubusercontent.com/ByteInternet/libapache-mod-ssl/master/pkg.sslcfg/snakeoil-ca-rsa.key
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 23.235.43.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|23.235.43.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 887 [text/plain]
Saving to: ‘snakeoil-ca-rsa.key’

100%[===================================================================================================================================================================================================>] 887         --.-K/s   in 0s      

2015-09-29 14:09:00 (369 MB/s) - ‘snakeoil-ca-rsa.key’ saved [887/887]

$ PASS=$(cat level2.key.encrypted | base64 -d | openssl rsautl -decrypt -inkey snakeoil-ca-rsa.key)
$ openssl enc -d -aes-256-cbc -salt -base64 -in level2.file.encrypted -out level2.file -pass pass:$PASS
$ file level2.file
level2.file: Zip archive data, at least v2.0 to extract
$ mv level2.file level2.zip
```

Unzipping the *level2.zip* with the (reused) password *changeit* resulted with the following three files:

```
level3.file.encrypted
level3.key.encrypted
Readme.txt
```

Content of `Readme.txt` was as follows:

```
========================================================================
Level 3 quest's hint :
 -This RSA key is intended for testing the most popular packet analyzer.
 -Prime1 < Prime2
========================================================================

Have fun!
```

By visiting the (*most popular packet analyzer*) Wireshark's Github repository (https://github.com/wireshark/wireshark), we found the following files inside the directory */test/keys*:

```
dhe1_keylog.dat     add a test for SSL/TLS decryption using the master secret
key.p12     SSL: Add support for private key password when decrypting
rsa-p-lt-q.key  ssl-utils: fix failing decryption for some RSA keys
rsasnakeoil2.key    Remove svn:executable attribute.
snakeoil-rsa.key    Add a test for DTLS decryption.
```

As the hint says *Prime1 < Prime2* by using the *rsa-p-lt-q.key* we got the following results:

```
$ wget https://raw.githubusercontent.com/wireshark/wireshark/master/test/keys/rsa-p-lt-q.key
--2015-09-29 14:09:41--  https://raw.githubusercontent.com/wireshark/wireshark/master/test/keys/rsa-p-lt-q.key
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.31.19.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.31.19.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 887 [text/plain]
Saving to: ‘rsa-p-lt-q.key’

100%[===================================================================================================================================================================================================>] 887         --.-K/s   in 0s      

2015-09-29 14:09:41 (474 MB/s) - ‘rsa-p-lt-q.key’ saved [887/887]

$ PASS=$(cat level3.key.encrypted | base64 -d | openssl rsautl -decrypt -inkey rsa-p-lt-q.key)
$ openssl enc -d -aes-256-cbc -salt -base64 -in level3.file.encrypted -out level3.file -pass pass:$PASS
$ file level3.file
level3.file: Zip archive data, at least v2.0 to extract
$ mv level3.file level3.zip
```

Unzipping the `level3.zip` with the (reused) password `changeit` resulted with the following three files:

```
level4.file.encrypted
level4.key.encrypted
Readme.txt
```

Content of `Readme.txt` was as follows:

```
========================================================================
Level 4 quest's hint :
 -This RSA key is intended for testing the most famous implementation of Python.
 -This published key is password-encrypted, and you may also have to find the password.
 -SHA1 hash for the encrypted key is 81754334cd36efd8ac0664ef78f92be4b1f9f4ac.
========================================================================

Have fun!
```

By visiting the (*most famous implementation of Python*) CPython's Github repository (https://github.com/python/cpython), we found the file *ssl_key.pem* inside the directory */Lib/test*. Using it we got the following results:

```
$ wget https://raw.githubusercontent.com/python/cpython/master/Lib/test/ssl_key.pem
--2015-09-29 14:19:36--  https://raw.githubusercontent.com/python/cpython/master/Lib/test/ssl_key.pem
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 23.235.43.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|23.235.43.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 916 [text/plain]
Saving to: ‘ssl_key.pem’

100%[===================================================================================================================================================================================================>] 916         --.-K/s   in 0s      

2015-09-29 14:19:36 (378 MB/s) - ‘ssl_key.pem’ saved [916/916]

$ PASS=$(cat level4.key.encrypted | base64 -d | openssl rsautl -decrypt -inkey ssl_key.pem)
$ openssl enc -d -aes-256-cbc -salt -base64 -in level4.file.encrypted -out level4.file -pass pass:$PASS
$ file level4.file
level4.file: Zip archive data, at least v2.0 to extract
$ mv level4.file level4.zip
```

Unzipping the *level4.zip* with the (reused) password *changeit* resulted with the following three files:

```
level5.file.encrypted
level5.key.encrypted
Readme.txt
```

Content of *Readme.txt* was as follows:

```
========================================================================
Level 5 quest's hint :
The key is one of the keys related to a famous vulnerable random generator.
========================================================================

Have fun!
```

By Googling for *famous vulnerable random generator* we've seen that there are references to the *Debian OpenSSL Bug*. Hence, by Googling for *Debian OpenSSL keys* we were being brought to the *Debian OpenSSL Predictable PRNG* Github repository (https://github.com/g0tmi1k/debian-ssh). By using its collection of *predictable* SSH/RSA private keys in a brute-force manner we got the following results:

```
$ wget https://github.com/g0tmi1k/debian-ssh/raw/master/common_keys/debian_ssh_rsa_2048_x86.tar.bz2
--2015-09-29 14:30:37--  https://github.com/g0tmi1k/debian-ssh/raw/master/common_keys/debian_ssh_rsa_2048_x86.tar.bz2
Resolving github.com (github.com)... 192.30.252.131
Connecting to github.com (github.com)|192.30.252.131|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://raw.githubusercontent.com/g0tmi1k/debian-ssh/master/common_keys/debian_ssh_rsa_2048_x86.tar.bz2 [following]
--2015-09-29 14:30:38--  https://raw.githubusercontent.com/g0tmi1k/debian-ssh/master/common_keys/debian_ssh_rsa_2048_x86.tar.bz2
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 23.235.43.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|23.235.43.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 50226987 (48M) [application/octet-stream]
Saving to: ‘debian_ssh_rsa_2048_x86.tar.bz2’

100%[===================================================================================================================================================================================================>] 50.226.987  2,26MB/s   in 21s    

2015-09-29 14:31:01 (2,26 MB/s) - ‘debian_ssh_rsa_2048_x86.tar.bz2’ saved [50226987/50226987]

$ tar xjf debian_ssh_rsa_2048_x86.tar.bz2
$ cat level5.key.encrypted | base64 -d > level5.key.raw
$ for i in $(find ./rsa/2048/ -iname "*"); do openssl rsautl -decrypt -inkey $i -in level5.key.raw >> brute_results.txt; echo >> brute_results.txt; done 2>/dev/null
$ sort -u brute_results.txt  # to trim the blank lines
WZIj2mhQmOsp8A26hqVlRC6aLoB94+9yNWATCTnWzwo=
$ PASS="WZIj2mhQmOsp8A26hqVlRC6aLoB94+9yNWATCTnWzwo="
$ openssl enc -d -aes-256-cbc -salt -base64 -in level5.file.encrypted -out level5.file -pass pass:$PASS
$ file level5.file
level5.file: Zip archive data, at least v2.0 to extract
$ mv level5.file level5.zip
```

Unzipping the *level5.zip* with the (reused) password *changeit* resulted with the following file:

```
flag.txt
```

Content of *flag.txt* was as follows:

```
Congratulations!
This is the final quest.

TMCTF{Anyway, don't use (disclosed|default) keys:)}
```

Hence, the final flag was:

**TMCTF{Anyway, don't use (disclosed|default) keys:)}**

-stamparm
