# Cody's First Blog

## Flag 0 – PHP

The front page has a comment field. You can leave a comment but it will not be displayed on the page until an admin verifies it manually.
Writting a basic php command like `<?php echo "test" ?>` will immediately give you the flag.

### Why php?
The blog entry talks about php. Why not start there ?


## Flag 1 – Forced browsing ? Authentication issue ?

Looking at the actual html of the front page you will find a link to a admin login panel.
The link is embedded in a comment. The login panel itself proves pretty tough however we can bypass the login alltogether browsing to `admin.inc` instead of `admin.auth.inc` .

In the admin area you can approve all of the comments. And find the second flag.

## Flag 2 – PHP

Now this flag is a little strange.
From the admin panel found during flag 1 you can approve the comment from flag 0. But this doesnt actually do anything. Revisiting the front page shows that the code will be embedded in a comment similar to the link to the admin login panel.
The trick here is to play with the page parameter. The comment on the front page mentions the php include function and that's exactly what happens on the server side. The php include function is famous for lfi and rfi.

### RFI
The comment on the front page mentions that the server cannot talk to the outside world. And indeed rfi doesn't work.

### LFI
Basic LFI is difficult because we don't know what files can be found on the server and directory traversel does not work.
However navigating to `.../?page=http://127.0.0.1/index` works and the site can be used to execute php and js code via the comments.
The workflow here is writting a comment on the regular index.php page, then approving it and after that navigating to `.../?page=http://127.0.0.1/index`. The code will get executed.

Great but where to find the flag ?
The flag is hidden in the index.php file as a comment. In order to display it you can read the file content via php and write it to the comment section. This can be done using `<?php readfile(index.php); ?>. The flag can be found in the source code of the displayed page.  


