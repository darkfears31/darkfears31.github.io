---
title: "It Is My Birthday from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# It Is My Birthday from PicoCTF
[page](https://play.picoctf.org/practice/challenge/109?page=14)
>Description
I sent out 2 invitations to all of my friends for my birthday! I'll know if they get stolen because the two invites look similar, and they even have the same md5 hash, but they are slightly different! You wouldn't believe how long it took me to find a collision. Anyway, see if you're invited by submitting 2 PDFs to my website. [http://mercury.picoctf.net:11590/](http://mercury.picoctf.net:11590/)

based on this description we have to search some file that has two identical MD5 hash. After bit of a search i found this site about [MD5 collision](https://www.mathstat.dal.ca/~selinger/md5collision/) and it has two files.
- **Windows version:**
    - [hello.exe](https://www.mathstat.dal.ca/~selinger/md5collision/hello.exe). MD5 Sum: cdc47d670159eef60916ca03a9d4a007
    - [erase.exe](https://www.mathstat.dal.ca/~selinger/md5collision/erase.exe). MD5 Sum: cdc47d670159eef60916ca03a9d4a007
- **Linux version (i386):**
    - [hello](https://www.mathstat.dal.ca/~selinger/md5collision/hello). MD5 Sum: da5c61e1edc0f18337e46418e48c1290
    - [erase](https://www.mathstat.dal.ca/~selinger/md5collision/erase). MD5 Sum: da5c61e1edc0f18337e46418e48c1290

i downloaded it and changed the name of it to 
>hello.pdf 
>erase.pdf

because site only takes files that is ".pdf" 
I uploaded it and got this: 
```
<?php  
  
if (isset($_POST["submit"])) {    $type1 = $_FILES["file1"]["type"];    $type2 = $_FILES["file2"]["type"];    $size1 = $_FILES["file1"]["size"];    $size2 = $_FILES["file2"]["size"];    $SIZE_LIMIT = 18 * 1024;  
  
    if (($size1 < $SIZE_LIMIT) && ($size2 < $SIZE_LIMIT)) {  
        if (($type1 == "application/pdf") && ($type2 == "application/pdf")) {            $contents1 = file_get_contents($_FILES["file1"]["tmp_name"]);            $contents2 = file_get_contents($_FILES["file2"]["tmp_name"]);  
  
            if ($contents1 != $contents2) {  
                if (md5_file($_FILES["file1"]["tmp_name"]) == md5_file($_FILES["file2"]["tmp_name"])) {                    highlight_file("index.php");  
                    die();  
                } else {  
                    echo "MD5 hashes do not match!";  
                    die();  
                }  
            } else {  
                echo "Files are not different!";  
                die();  
            }  
        } else {  
            echo "Not a PDF!";  
            die();  
        }  
    } else {  
        echo "File too large!";  
        die();  
    }  
}  
  
// FLAG: picoCTF{c0ngr4ts_u_r_1nv1t3d_3d3e4c57}  
  
?>  
<!DOCTYPE html>  
<html lang="en">  
  
<head>  
    <title>It is my Birthday</title>  
  
  
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css" rel="stylesheet">  
  
    <link href="https://getbootstrap.com/docs/3.3/examples/jumbotron-narrow/jumbotron-narrow.css" rel="stylesheet">  
  
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>  
  
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>  
  
  
</head>  
  
<body>  
  
    <div class="container">  
        <div class="header">  
            <h3 class="text-muted">It is my Birthday</h3>  
        </div>  
        <div class="jumbotron">  
            <p class="lead"></p>  
            <div class="row">  
                <div class="col-xs-12 col-sm-12 col-md-12">  
                    <h3>See if you are invited to my party!</h3>  
                </div>  
            </div>  
            <br/>  
            <div class="upload-form">  
                <form role="form" action="/index.php" method="post" enctype="multipart/form-data">  
                <div class="row">  
                    <div class="form-group">  
                        <input type="file" name="file1" id="file1" class="form-control input-lg">  
                        <input type="file" name="file2" id="file2" class="form-control input-lg">  
                    </div>  
                </div>  
                <div class="row">  
                    <div class="col-xs-12 col-sm-12 col-md-12">  
                        <input type="submit" class="btn btn-lg btn-success btn-block" name="submit" value="Upload">  
                    </div>  
                </div>  
                </form>  
            </div>  
        </div>  
    </div>  
    <footer class="footer">  
        <p>&copy; PicoCTF</p>  
    </footer>  
  
</div>  
  
<script>  
$(document).ready(function(){  
    $(".close").click(function(){  
        $("myAlert").alert("close");  
    });  
});  
</script>  
</body>  
  
</html>
```
There is our flag: 
**picoCTF{c0ngr4ts_u_r_1nv1t3d_3d3e4c57}** 
