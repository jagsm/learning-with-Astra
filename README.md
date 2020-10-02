# Welcome to Learning with Astra #

I have created the Datastax Astra account
https://astra.datastax.com/org/e8c4fb8c-b8a7-49d4-9d8e-6cb763ae4a10/database/66adda38-56d9-403d-8ce2-39b491afa556

## Explain your use case ##

Create an data store for ecommerce-store and it shouild have the below capabilities
    - Order page should show the history of orders for the logged in user
    - Create a new admin page "Search", given an order number show full order+user details
    - Create a new management dashboard page to show the revenue by year/month by city
   
Include diagrams, screenshots etc to make it more interesting and better convey your ideas.

## Create your own tables on Astra ##

    - create products table to store the all the products info.
    - create users table to store the user details
    - create orders table to store the order details
    - create a table to pull the user record
    - create a table to pull the revenue for given month


Example tables that we used in the workshop:

```
create table killrvideo.products (
id uuid,
type text,
short_desc text,
long_desc text,
price double,
image blob,
PRIMARY KEY (id)
);

CREATE TABLE killrvideo.users (
    user_id text,
    password text,
first_name text,
last_name text,
address_1 text,
address_2 text,
city text,
phone text,
created_on timestamp,
modified_on timestamp,
    PRIMARY KEY ((user_id))
);

--Orders for any given user partitioned by year; including "date" in PK to support any non uuid based order num
--where there may be duplicates (just safety measure)
CREATE TABLE killrvideo.orders (
user_id text,
year int,
month int,
order_num timeuuid,
date int,
status int,
amount double,
tax double,
details text,
created_on timestamp,
PRIMARY KEY ((user_id, year), order_num, date)
) WITH CLUSTERING ORDER BY (order_num DESC, date DESC);


--Pull a user record given an order number
CREATE TABLE killrvideo.orders_by_user (
order_num timeuuid,
user_id text,
year int,
date int,
PRIMARY KEY ((order_num))
);


--what is my revenue from a city for a given month?
CREATE TABLE killrvideo.revenue_per_month (
city text,
year int,
month int,
user_id text,
order_num timeuuid,
amount double,
PRIMARY KEY ((city, year, month), user_id, order_num)
);
```

Show us your own tables - for the data model of your choice.


## Insert some data ##
 - Inserting data to users table
```
Insert into  users (user_id, password, first_name, last_name, address_1, address_2, city, phone, created_on, modified_on) values ('user1','password1','Datastax1','Astra1','London Bridge','West Minister','London','+449876543210',toTimeStamp(now()),toTimeStamp(now()));

Insert into  users (user_id, password, first_name, last_name, address_1, address_2, city, phone, created_on, modified_on) values ('user2','password2','Datastax2','Astra2','London Bridge','West Minister','London','+449876543210',toTimeStamp(now()),toTimeStamp(now()));
```
- Inserting data to orders table
```
Insert into  orders (user_id, year, month, order_num, date, status, amount, tax, details, created_on) values ('user1',2020,10,Now(),02,1,500.00,5.00,'Headphones',toTimeStamp(now()));

Insert into  orders (user_id, year, month, order_num, date, status, amount, tax, details, created_on) values ('user2',2020,10,Now(),02,1,100.00,5.00,'Headphones',toTimeStamp(now()));

```
- Inserting data to orders_by_user table
```
Insert into  orders_by_user (order_num,user_id,year,date) values (Now(),'user1',2020,02);
Insert into  orders_by_user (order_num,user_id,year,date) values (Now(),'user2',2020,02);

```
- Inserting data to revenue_per_month table
```
Insert  into revenue_per_month (city,year,month,user_id,order_num,amount) values ('London',2020,02,'user1',Now(),10.00);
Insert  into revenue_per_month (city,year,month,user_id,order_num,amount) values ('London',2020,02,'user2',Now(),5.00);
```

## Read the data from tables ##

```
SELECT * FROM users;

 user_id | address_1     | address_2     | city   | created_on                      | first_name | last_name | modified_on                     | password  | phone
---------+---------------+---------------+--------+---------------------------------+------------+-----------+---------------------------------+-----------+---------------
   user2 | London Bridge | West Minister | London | 2020-10-02 15:46:35.964000+0000 |  Datastax2 |    Astra2 | 2020-10-02 15:46:35.964000+0000 | password2 | +449876543210
   user1 | London Bridge | West Minister | London | 2020-10-02 15:43:13.253000+0000 |  Datastax1 |    Astra1 | 2020-10-02 15:43:13.253000+0000 | password1 | +449876543210

```
```
select * from orders;

 user_id | year | order_num                            | date | amount | created_on                      | details    | month | status | tax
---------+------+--------------------------------------+------+--------+---------------------------------+------------+-------+--------+-----
   user1 | 2020 | 4a186d90-04c7-11eb-bd49-e5ed22f5e99b |    2 |    100 | 2020-10-02 15:52:35.305000+0000 | Headphones |    10 |      1 |   5   
   user1 | 2020 | 2d6970e0-04c7-11eb-bd49-e5ed22f5e99b |    2 |    100 | 2020-10-02 15:51:47.182000+0000 | Headphones |    10 |      1 |   5
   user1 | 2020 | 10e4b880-04c7-11eb-bd49-e5ed22f5e99b |    2 |    500 | 2020-10-02 15:50:59.336000+0000 | Headphones |    10 |      1 |   5
   
```
```
select * from orders_by_user;
order_num                            | date | user_id | year
--------------------------------------+------+---------+------
 f8ea74d0-04c7-11eb-bd49-e5ed22f5e99b |    2 |   user2 | 2020
 e37b3e40-04c7-11eb-bd49-e5ed22f5e99b |    2 |   user1 | 2020
 
 ```
 ```
 select * from revenue_per_month;
 city   | year | month | user_id | order_num                            | amount
--------+------+-------+---------+--------------------------------------+--------
 London | 2020 |     2 |   user1 | a1ca1d80-04c8-11eb-bd49-e5ed22f5e99b |     10
 London | 2020 |     2 |   user2 | b4e13340-04c8-11eb-bd49-e5ed22f5e99b |      5
 
 ```


## Experiment with CRUD and show the outputs: ##

- Updating the User info in the user table 

```
UPDATE users 
SET phone = '+441233445678' 
WHERE user_id = 'user1' ;

SELECT * FROM users;

 user_id | address_1     | address_2     | city   | created_on                      | first_name | last_name | modified_on                     | password  | phone
---------+---------------+---------------+--------+---------------------------------+------------+-----------+---------------------------------+-----------+---------------
   user2 | London Bridge | West Minister | London | 2020-10-02 15:46:35.964000+0000 |  Datastax2 |    Astra2 | 2020-10-02 15:46:35.964000+0000 | password2 | +449876543210
   user1 | London Bridge | West Minister | London | 2020-10-02 15:43:13.253000+0000 |  Datastax1 |    Astra1 | 2020-10-02 15:43:13.253000+0000 | password1 | +441233445678
```
- Delete the user from user table
```
Delete from users where user_id= 'user2';

SELECT * FROM users;
 user_id | address_1     | address_2     | city   | created_on                      | first_name | last_name | modified_on                     | password  | phone
---------+---------------+---------------+--------+---------------------------------+------------+-----------+---------------------------------+-----------+---------------
   user1 | London Bridge | West Minister | London | 2020-10-02 15:43:13.253000+0000 |  Datastax1 |    Astra1 | 2020-10-02 15:43:13.253000+0000 | password1 | +441233445678
```

## Screenshots from Datastax Astra ##

![image](https://github.com/jagsm/learning-with-Astra/blob/master/Screenshot%202020-10-02%20at%2017.07.56.png)

![image](https://github.com/jagsm/learning-with-Astra/blob/master/Screenshot%202020-10-02%20at%2017.30.54.png)
