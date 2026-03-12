# SQL injection

vulnerability in WHERE clause allowing retrieval of hidden data
> https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data

## Mission

> This lab contains a SQL injection vulnerability in the product category filter. When the user selects a category, the application carries out a SQL query like the following:
>> SELECT * FROM products WHERE category = 'Gifts' AND released = 1
> 
> To solve the lab, perform a SQL injection attack that causes the application to display one or more unreleased products.

Site queries database from url based variables, where filters key is category and it's value. Like below
> https://0a9f004f03a650418371e68f004a0033.web-security-academy.net/filter?category=Corporate+gifts

To get all the data without querys released parameter needing to be 1, or what ever the category is.</br>
We can query the url with added characters like below, which will change the sql query to database.
> https://0a9f004f03a650418371e68f004a0033.web-security-academy.net/filter?category=Corporate+gifts' OR '1' = '1' --

First we need to end the string with '(quote), so that sql query thinks the category value ended there.<br />
Then using OR and logical operation if 1 is 1, which is allways true.<br />
Last we need to end the query with SQL comments --(dashdash), which will comment out the 'AND released = 1' from the sql query.<br />

So the sql query goes something like get every data from products, if category=<value> OR if 1=1.
So the query is true no matter what and will print all the data from the products table.
> SELECT * FROM products WHERE category = 'notFound' OR '1'='1'. 

<img width="1547" height="1205" alt="sqlHidden" src="https://github.com/user-attachments/assets/dccd324e-a58c-477a-99e6-c33b52ea220e" />
