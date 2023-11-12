# Advance_-SQL_-with_-Parch-_and_-Posey_-database-


/*In the accounts table, there is a column holding the website for each company. 
The last three digits specify what type of web address they are using.
Pull these extensions and provide how many of each website type exist in the accounts table.*/
SELECT RIGHT (website, 3) AS domain, COUNT(*) num_companies
FROM accounts
GROUP BY 1
ORDER BY 2 DESC;


/*There is much debate about how much the name (or even the first letter of a company name) matters. 
Use the accounts table to pull the first letter of each company name to
see the distribution of company names that begin with each letter (or number)*/
SELECT LEFT(UPPER(name), 1) AS first_letter, COUNT(*) num_companies
FROM accounts
GROUP BY 1
ORDER BY 2 DESC;

/*	Use the accounts table and a CASE statement to create two groups: 
one group of company names that start with a number and a second group of those company names that start with a letter. 
What proportion of company names start with a letter?*/

SELECT SUM(num) nums, SUM(letter) letters
FROM (SELECT name, CASE WHEN LEFT(UPPER(name), 1) IN ('0','1','2','3','4','5','6','7','8','9') 
                          THEN 1 ELSE 0 END AS num, 
            CASE WHEN LEFT(UPPER(name), 1) IN ('0','1','2','3','4','5','6','7','8','9') 
                          THEN 0 ELSE 1 END AS letter
         FROM accounts) t1;


/* Consider vowels as a, e, i, o, and u. What proportion of company names start with a vowel, 
and what percent start with anything else?*/

SELECT SUM(vowels) vowels, SUM(other) other
FROM (SELECT name, CASE WHEN LEFT(UPPER(name), 1) IN ('A','E','I','O','U') 
                           THEN 1 ELSE 0 END AS vowels, 
             CASE WHEN LEFT(UPPER(name), 1) IN ('A','E','I','O','U') 
                          THEN 0 ELSE 1 END AS other
            FROM accounts) t1;

/* use acounts table to create first & last name column that hold the first & last name for the Primary_POC*/
SELECT primary_poc, LEFT(primary_poc, STRPOS(accounts.primary_poc, ' ')-1),
RIGHT(primary_poc, LENGTH(primary_poc)- STRPOS(accounts.primary_poc, ' '))
FROM accounts

/* now see if u can do same thing for every rep name in the sales_reps table*/
SELECT name full_name, LEFT(name, STRPOS(s.name, ' ')-1) first_name,
			 RIGHT(name, LENGTH(name)- STRPOS(s.name, ' ')) last_name
FROM sales_reps s

/*Each company in the accounts table wants to create an email address for each primary_poc. 
the email address should be the first name of the primary_poc . primary_poc lastname @ company name.com*/
WITH t1 As
	(SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ')-1) first_name,
	RIGHT(primary_poc, LENGTH(primary_poc)- STRPOS(primary_poc, ' ')) last_name, name
	FROM accounts)
SELECT first_name, last_name, CONCAT(first_name, '.', last_name, '@', name, '.com')
FROM t1

/*remove space in the company name part of the email*/
WITH t1 As
	(SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ')-1) first_name,
	RIGHT(primary_poc, LENGTH(primary_poc)- STRPOS(primary_poc, ' ')) last_name, name
	FROM accounts)
SELECT first_name, last_name, CONCAT(first_name, '.', last_name, '@', REPLACE(name, ' ', ''),'.com')
FROM t1

/*We would also like to create an initial password, 
which they will change after their first log in. 
The first password will be the first letter of the primary_poc’s first name (lowercase), 
then the last letter of their first name (lowercase), the first letter of their last name (lowercase), 
the last letter of their last name (lowercase), the number of letters in their first name, 
the number of letters in their last name, and then the name of the company they are working with,
all capitalized with no spaces.*/
WITH t1 AS (
    SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') -1 ) first_name,  
	RIGHT(primary_poc, 
	LENGTH(primary_poc)-STRPOS(primary_poc, ' ')) last_name, 
	name
    FROM accounts)
SELECT first_name, last_name, CONCAT(first_name, '.', last_name, '@', name, '.com'),
LEFT(LOWER(first_name), 1) || RIGHT(LOWER(first_name), 1) || LEFT(LOWER(last_name), 1) || RIGHT(LOWER(last_name), 1)
|| LENGTH(first_name) || LENGTH(last_name) || REPLACE(UPPER(name), ' ', '')
FROM t1


/*Each company in the sales_rep table wants to create an email address for each sales_rep. 
the email address should be the first name of the sales_reps . sales_reps lastname @ region_name .com*/

SELECT first_name, last_name, CONCAT(first_name, '.', last_name, '@', region_name, '.com')			
FROM (
SELECT LEFT(sales_rep_name, STRPOS(sales_rep_name, ' ')-1) first_name,
			RIGHT(sales_rep_name, LENGTH(sales_rep_name) - STRPOS(sales_rep_name, ' ')) last_name, region_name
			FROM (SELECT s.name sales_rep_name, r.name region_name
FROM sales_reps s
JOIN region r
ON s.region_id = r.id))
			
/*We would also like to create an initial password, 
which they will change after their first log in. 
The first password will be the first letter of the first name (lowercase), 
then the last letter of their first name (lowercase), the first letter of their last name (lowercase), 
the last letter of their last name (lowercase), the number of letters in their first name, 
the number of letters in their last name, and then the name of the region they are working with,
all capitalized with no spaces.*/	

--CTE mtd
			
WITH t2 AS (
SELECT LEFT(sales_rep_name, STRPOS(sales_rep_name, ' ')-1) first_name,
			RIGHT(sales_rep_name, LENGTH(sales_rep_name) - STRPOS(sales_rep_name, ' ')) last_name, region_name
			FROM (SELECT s.name sales_rep_name, r.name region_name
FROM sales_reps s
JOIN region r
ON s.region_id = r.id))
SELECT first_name, last_name, CONCAT(first_name, '.', last_name, '@', region_name, '.com') email,
LEFT(LOWER(first_name),1) || RIGHT(LOWER(first_name),1) || LEFT(LOWER(last_name),1) || RIGHT(LOWER(last_name),1) ||
LENGTH(first_name) || LENGTH(last_name)	|| UPPER(region_name) AS password
FROM t2
									 
									 
									 

--subquery mtd
SELECT first_name, last_name, CONCAT(first_name, '.', last_name, '@', region_name, '.com') email,
LEFT(LOWER(first_name),1) || RIGHT(LOWER(first_name),1) || LEFT(LOWER(last_name),1) || RIGHT(LOWER(last_name),1) ||
LENGTH(first_name) || LENGTH(last_name)	|| UPPER(region_name) AS password
FROM (SELECT LEFT(sales_rep_name, STRPOS(sales_rep_name, ' ')-1) first_name,
			RIGHT(sales_rep_name, LENGTH(sales_rep_name) - STRPOS(sales_rep_name, ' ')) last_name, region_name
			FROM (SELECT s.name sales_rep_name, r.name region_name
FROM sales_reps s
JOIN region r
ON s.region_id = r.id))
						
