# MySQL CLI

* [See mysqltutorial](https://www.mysqltutorial.org/mysql-cheat-sheet.aspx)

```bash
# Login
mysql
mysql -u <username> -p;
mysql -u <username> -p <database>;
mysql -h <host> -u <user> -p <database>;

# Logout
exit;
```

```sql
prompt -- return to prompt

-- USER 
select user, host from mysql.user;
select current_user(); -- show current user
select user();
create user 'username'@'localhost' identified by 'password';
grant all on database.* to 'user'@'localhost'; -- grant all access to user for * tables

-- DATABASE
select current_database(); -- show current database
select database();
show databases; -- show available databases
use <name>; -- use a database
create database <name>;
drop database <name>;

-- TABLES
show tables;
describe <table>;
show index from <table>;

select now(); -- see datetime format (timestamp)

show variables; -- see system variables
```


