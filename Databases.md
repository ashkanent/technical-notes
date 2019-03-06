# General Info
* Two popular types of data storage are relational (like SQL) and non-relational (like MongoDB)
* Some examples of DBMS:
  * MySQL
  * SQL Server
  * PostgresSQL
  * Oracle
  * MS SQL server
  * MongoDB
  * Oracle NoSQL DB

# Relational Databases

## MySQL
* You need to run MySQL server to be able to create databases and work with them  
  * You can install it in your computer or run it in docker
* You can use MySQL Workbench as a GUI to interact with MySQL server.
* SQL or *Structured Query Language* is the query language that can be used to retrieve
data from *Relational Databases*.
* in these type of databases, data is normalized, meaning they should all have valid values for
all the defined columns of the table and they all have the exact same fields. (we can't have a row that has more fields compared to other rows)
* data is usually in multiple tables and they are ***related*** to each other.

### Some Basic Commands
* `create database Ashkan;` --> creates this database named "Ashkan"
* `show databases;` --> to list all the databases
* `use Ashkan;` --> to start using a database
* `create table student (rollNo int, name varchar(20));` --> create the table
* `show tables;` --> to list all the tables
* `desc student;` --> describe this table (shows columns and their types)
* `insert into student values(1, 'Ashkan');`
* `select * from student;` --> select everything from table with no condition (no where clause)
* using autocommit and rollback:
  ```SQL
  set autocommit = 0; -- by default it is set to '1'
  insert into student values (2, 'Jose');
  rollback; /* it will undo all the commits above*/
  insert into student values (2, 'Jossie');
  commit; -- after this point we can't rollback anymore
  ```
* `delete from student where  rollNo > 2;` --> deleting values from table
* `delete from student;` == `truncate student;` --> deletes all the data
  * the difference is that with `truncate` there is no rollback!
* `drop table student;` --> deletes the table
* `drop database Ashkan;` --> deletes the entire database
* `select count(*) from student;` --> total number of rows in this table
* `select max(marks) from student;` --> returns the row with maximum value for 'marks'
  * you can use `min(marks)`, `avg(marks)` or `sum(marks)` as well.
* `select count(*) as 'Total count' from student;`
  * this is to set alias
  * we used single quotes for *Total count* because there is space in there
* `select * from student order by marks desc;` --> to order the return values


### Notes
* statements in SQL are of these types:
  1. **DDL** —Data Definition Language
    * like when you create or drop tables, databases, etc.
  2. **DML** —Data Manipulation Language
    * when you insert, update or delete values
  3. **DRL/DQL** —Data Retrieval/Query Language
    * When you query data from database
  4. **TCL** —Transaction Control Language
    * Setting permissions, setting autocommit and rollback, etc.
* Horizontal scaling is difficult/impossible (adding more db servers), vertical scaling is possible with some limits (adding more data).
* It has some limitations for lots of read/writes queries per second.


# NoSQL

## MongoDB
  * it is a *NoSQL* database.
  * each database is compromised of multiple _collections_ (equivalent of tables in MySQL)
  * each collection has documents (like rows of data in MySQL)
    * data is in JSON format
    * each document can have different fields/schema (unlike MySQL)
  * No (few) relations.
    * for example you can have _Orders_ collection that can duplicate some data from _Users_ and _Products_ collections. Down side is this duplication but at the same time queries is much more efficient as you don't need to connect multiple tables like you do in MySQL to read something
      * if you update something in _Users_ you have to update it in _Orders_ as well which is not ideal
   * this is ideal for an application that has more reads than writes.
 * great performance for lots of read & writes
