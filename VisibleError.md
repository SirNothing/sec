# Visible error-based SQL injection
> https://portswigger.net/web-security/sql-injection/blind/lab-sql-injection-visible-error-based


This lab contains a SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie. The results of the SQL query are not returned.

The database contains a different table called users, with columns called username and password. To solve the lab, find a way to leak the password for the administrator user, then log in to their account.

### Finding the vulnerable input

As the lab says that there is tracking cookie which value is used in sql query.
Finding the cookie and trying to get errors with different sql syntax, like '(quote), logical 1=1, 1=2 or CAST string to int etc.

Adding '(quote) to the cookie will give visible sql error. So this input is prorably vulnerable.

<img width="1616" height="833" alt="sqlVisError1" src="https://github.com/user-attachments/assets/7e9fce82-6f7c-4d03-9856-afbb74a319a0" />

To get back to valid sql syntax we will need to add the sql comment(--) at the end to discard rest of the query as comment.

Next we can't try to add SELECT to the query and see if database runs our code. CAST is to make sure database take's it as int and 'SELECT 1' just return "database" columns as 1(won't add or take from database). If CAST is working we can also narrow down the database type to Postresql, mysql. Like with oracle it's TO_NUMBER and not CAST.
> 1okdvW73OMEG6Cuy' AND CAST((SELECT 1) AS int)--

<img width="1614" height="535" alt="sqlVisError2" src="https://github.com/user-attachments/assets/365ff88a-7662-42ed-af7d-36cb1b0676a7" />

And we get another error as WHERE clause must return a boolein value. Fixing the query to make the AND as boolein.

> 1okdvW73OMEG6Cuy' AND 1=CAST((SELECT 1) AS int)--

SQL syntax is again working and next we can try to extract real data from the database.
> fdL7JBSSBrhV7Hmn' AND 1=CAST((SELECT username FROM users) AS int)--

We get again 'Underminated string error' because the query is too long. Make it shorter by removing some of the cookie hash.
> f' AND 1=CAST(( SELECT username FROM users) AS int)--

database gives a new error: 'ERROR: more than one row returned by a subquery used as an expression'<br />
Database don't know which username to return as there are multiple rows of data.<br />
To just query one username from database, we need to add *LIMIT* to the query(Different syntax in other database).

> f' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--

And we got the first username from the database, which is the administrator.

<img width="1749" height="909" alt="sqlCookieErrorUser" src="https://github.com/user-attachments/assets/930d25f5-304d-4e80-8a49-8641b0584e7c" />

Next just change the username to password and it will give us the administrator's password also.

<img width="1171" height="615" alt="sqlCookieErrorPass" src="https://github.com/user-attachments/assets/707e07f9-b757-4e4c-975c-1db6f1a9c3a6" />

Then just login with the username/password.
