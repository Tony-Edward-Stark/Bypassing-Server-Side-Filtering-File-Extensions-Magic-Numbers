# 1. Bypassing-Server-Side-Filtering-File-Extensions
Client-side filters are easy to bypass -- we can see the code for them, even if it's been obfuscated and needs processed before we can read it; but what happens when we can't see or manipulate the code? Well, that's a server-side filter. In short, we have to perform a lot of testing to build up an idea of what is or is not allowed through the filter, then gradually put together a payload which conforms to the restrictions. 
```php
<?php
    //Get the extension
    $extension = pathinfo($_FILES["fileToUpload"]["name"])["extension"];
    //Check the extension against the blacklist -- .php and .phtml
    switch($extension){
        case "php":
        case "phtml":
        case NULL:
            $uploadFail = True;
            break;
        default:
            $uploadFail = False;
    }
?>

```
In this instance, the code is looking for the last period (.) in the file name and uses that to confirm the extension, so that is what we'll be trying to bypass here. Other ways the code could be working include: searching for the first period in the file name, or splitting the file name at each period and checking to see if any blacklisted extensions show up. 

We can see that the code is filtering out the .php and .phtml extensions, so if we want to upload a PHP script we're going to have to find another extension. The wikipedia page for [PHP](https://en.wikipedia.org/wiki/PHP) gives us a few common extensions that we can try; however, there are actually a variety of other more rarely used extensions available that webservers may nonetheless still recognise. These include: .php3, .php4, .php5, .php7, .phps, .php-s, .pht and .phar. Many of these bypass the filter (which only blocks.php and .phtml).

Changing the **shell file's extension** to required file we can upload the file in restricted file in the server.

#  Bypassing-Server-Side-Filtering-Magic-Numbers

magic numbers are used as a more accurate identifier of files. The magic number of a file is a string of hex digits, and is always the very first thing in a file. Knowing this, it's possible to use magic numbers to validate file uploads, simply by reading those first few bytes and comparing them against either a whitelist or a blacklist. Bear in mind that this technique can be very effective against a PHP based webserver; however, **it can sometimes fail against other types of webserver**
**Let's assume, to a targeted site** if we upload our standard shell.php file, we get an error; however, if we upload a JPEG, the website is fine with it. so let's try adding the JPEG magic number to the top of our **shell.php** file. A quick look at the list of [file signatures on Wikipedia](https://en.wikipedia.org/wiki/List_of_file_signatures) shows us that there are several possible magic numbers of JPEG files. It shouldn't matter which we use here, so let's just pick one **(FF D8 FF DB)**. We could add the ASCII representation of these digits (ÿØÿÛ) directly to the top of the file but it's often easier to work directly with the hexadecimal representation, so let's cover that method.

Before we get started, let's use the Linux file command to check the file type of our shell:
![image](https://github.com/Tony-Edward-Stark/Bypassing-Server-Side-Filtering-File-Extensions-Magic-Numbers/assets/112267976/9ca49c6c-5d2f-4990-ae78-fa002f9fd8ad)
As expected, the command tells us that the filetype is PHP. Keep this in mind as we proceed with the explanation. We can see that the magic number we've chosen is **four bytes long**, so let's open up the reverse shell script and add four random characters on the first line. These characters do not matter, so for this example we'll just use four "A"s:

![oe434wu](https://github.com/Tony-Edward-Stark/Bypassing-Server-Side-Filtering-File-Extensions-Magic-Numbers/assets/112267976/31ae56a8-1b06-4426-abb0-66cb2e06301f)

Save the file and exit. Next we're going to reopen the file in **hexeditor** (which comes by default on Kali), In hexeditor the file looks like this:

![otIyN96](https://github.com/Tony-Edward-Stark/Bypassing-Server-Side-Filtering-File-Extensions-Magic-Numbers/assets/112267976/96bcfa10-00df-4883-9293-d6208977424f)

Note the four bytes in the red box: they are all **41**, which is the **hex code for a capital "A"** -- exactly what we added at the top of the file previously. Change this to the magic number we found earlier for JPEG files: **FF D8 FF DB**

![2OlGKdQ](https://github.com/Tony-Edward-Stark/Bypassing-Server-Side-Filtering-File-Extensions-Magic-Numbers/assets/112267976/16c39fad-f8d8-4c40-b283-530b22c6dc83)

Now if we save and exit the file (Ctrl + x), we can use **file** once again, and see that we have successfully spoofed the filetype of our shell

![ldyt88v](https://github.com/Tony-Edward-Stark/Bypassing-Server-Side-Filtering-File-Extensions-Magic-Numbers/assets/112267976/eceb2796-9daf-4512-bb23-7a1682ecd129)

**Perfect. Now let's try uploading the modified shell and see if it bypasses the filter!**



