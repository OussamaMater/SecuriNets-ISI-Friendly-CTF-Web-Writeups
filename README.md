# SecuriNets ISI Friendly CTF 2023-2024

This guide will walk you through solving all the web challenges for this mini CTF.

## Action Potential

>Securinets{Em&rasse_M@I_FoU$A}

For this challenge, you are presented with a web page. Upon inspection, you will find the following code

```diff
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="style.css">
    <title>Formulaire</title>
</head>
<body>
    <form>
        <label>Enter Secret Code : </label>
        <input type="text" id="code" placeholder="Enter Code !!">
        <input type="submit" value="Submit" onclick="check()">
+        <script src="script.js"></script>
    </form>
</body>
</html>
```

Clicking on the script will open the function that checks the code. You can skip everything and simply click on the highlighted link where you claim your flag

```diff
function check(){
    var pass=document.getElementById("code").value;
    if (pass!="Choufli_Hal"){
        alert("Incorrect Code");
        return false;
    }
    else{
+        alert("https://aymen-benyounes008.github.io/Web-Exploitation-Task/2nd%20Task/flag.html");
    }
}
```

## Mokhek Ye9ef Badnek Y3oum

>Securinets{B@3_W_R$wa7}

This challenge is similar to the first one. By inspecting the code, you'll find a link where you can obtain your flag

```diff
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MYBY</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <img src="aaslema.png">
        <h2>display is a CSS property that changes the default display mode of an HTML element</h2>
        <p>Is this an image of a Floating Human In The Sea ?? </p>
        <p> If not can you show it ? I am sure its here hiding somewhere .</p><br>
        <img id="img1" src="img1.jpg">
+        <a href="https://aymen-benyounes008.github.io/Web-Exploitation-Task/Task/Task.html"><img id="img2" src="img2.jpg"> </a>
    </div>
</body>
</html>
```

### IDOR

>Securinets{1d0r5_4r3_Fr3quEn7!}

In this challenge, you are presented with a web page containing a dropdown menu. By selecting an order, you can retrieve its details. If you inspect the network tab, you will notice that each time you choose an order, a request is made to `/orders.php?id=<id>`. You can continuously change the id until you obtain the flag. The correct ID was 1.

### SQL Injection

>flag{y0u_F0uNd_mY_p@ssWoRd}

In this challenge, you encounter a web page that allows you to search for users by ID. As the name suggests, this is a SQL injection task. You can either use a tool like [sqlmap](https://github.com/sqlmapproject/sqlmap), which would instantly dump everything, or you can perform the injection manually.

If you prefer a manual approach, begin by checking the database type. Submitting a single quote (') yields a warning message `SQLite3::query()`, confirming the use of SQLite as the database.

> [!NOTE]
>Always identify  the database type, as payloads and enumeration techniques may differ.

Now that we know we're dealing with SQLite, the next step is to determine the columns present in the `users` table. Well, I took a wild guess since the filename is `users.php` and we are looking for _users_.

There are multiple ways to determine the columns; one of them is using the [UNION operator](https://www.w3schools.com/sql/sql_union.asp), so we can pass the following payload:

```sql
1 UNION SELECT name FROM PRAGMA_TABLE_INFO('users')
```

You can imagine that behind the scenes, the full query would look like this:

```sql
SELECT colum FROM users WHERE id = 1 UNION SELECT name FROM PRAGMA_TABLE_INFO('users')
```

> [!NOTE]
> A UNION query must have the exact number of columns on both queries. Since we only see one column being returned, we selected only one column as well.

Running the query reveals three columns: `id`, `password`, and `username`. Let's take a closer look at the password column:

```sql
1 UNION SELECT password FROM users
```

And indeed, the flag is stored as one of the passwords.

### Directory Indexing

>Securinets{directory_listening_for_the_win_guys}

This challenge revolves around [directory listing](https://sapt.medium.com/directory-listing-vulnerability-cyber-sapiens-internship-task-16-fca5c9a4bb8a). Open any profile page and then remove the `id` parameter from the URL, making it look like the following: `http://46.101.195.237:8000/profile/`. This action will display a list of all files, including the flag.

### ROB

>Securinets{Yaay_u_f0und_r0bots!!}

This challenge hints at eliminating the [robots](https://www.seobility.net/en/wiki/Robots.txt), To solve the challenge, navigate to `https://friendly.securinets-isi.com/robots.txt`.

### Remote Code Execution

> Securinets{;_@nd_H4cK_tH3_Pl4n3T}

The name suggests that this challenge is vulnerable to [RCE](https://www.bugcrowd.com/glossary/remote-code-execution-rce/), and we are presented with an application that pings a given host.

By passing `127.0.0.1` as the host, we can view the logs of the ping binary on the system. This indicates the execution of the command on the server itself, rather than using an external service. Therefore, we can infer that it is performing a task similar to:

```php
shell_exec('ping ' . $host);
```

So to exploit this, we can append our command alongside the host, for example, `127.0.0.1; ls -l`. In Linux, the semicolon (;) separates commands, signifying the execution of command 1 followed by command 2. Consequently, the final command would appear as:

```php
shell_exec('ping 127.0.1; ls -l');
```

Executing this command indeed returns the directory listing. We can proceed by passing `127.0.0.1; cat flag.php`, this will return the content of the file,  and you can read the flag by inspecting the page.
