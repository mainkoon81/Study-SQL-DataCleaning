# Study-SQL-DataCleaning

#### 1) Clean and re-structure messy data.
#### 2) Convert columns to different data types.
#### 3) Tricks for manipulating NULLs.
 - *Left( )……beginning???
 - *Right( )………ending???
 - *Length( ) 
 - *POSITION( )
 - *STRPOS( )
 - *CONCAT( )
 - *CAST( )
 - *COALESCE( ) 

<img src="https://user-images.githubusercontent.com/31917400/33804790-61f11248-dda4-11e7-80b3-48be3592c300.png" width="600" height="300" />

### left( ) / right( ) / length( )
phone_num: 000-000-0000
 - *LEFT( ) pulls a specified number of characters for each row in a specified column starting at the beginning (or from the left). For example, pull digits of a phone number using: 
 - *LENGTH( ) provides the number of characters for each row of a specified column. To get the length of each phone number, using: 
```
SELECT LEFT(phone_num, 3) area_code, RIGHT(phone_num, 8) only_num, RIGHT(phone_num, LENGTH(phone_num) – 4) alt_num

#It's a func() within a func()…The innermost func is evaluated first, then the func that encapsulates it comes later.
```
 - > Q1. In the accounts table, there is a column holding the website for each company. The last three digits specify what type of web address they are using. A list of extensions (and pricing) is provided here. Pull these extensions and provide how many of each website type exist in the accounts table.
```
SELECT right(website, 3) "type", count(*)
FROM accounts
GROUP BY "type" 
```
 - > Q2. There is much debate about how much the name (or even the first letter of a company name) matters. Use the accounts table to pull the first letter of each company name to see the distribution of company names that begin with each letter (or number). 
```
SELECT left(upper(name), 1) "letter", count(*)
FROM accounts
GROUP BY "letter" 
ORDER BY 2 DESC 
```
 - > Q3. Use the accounts table and a CASE statement to create two groups: one group of company names that start with a number and a second group of those company names that start with a letter. What proportion of company names start with a letter?
```
SELECT SUM("num") "nums", SUM("letter") "letters"
FROM 
(SELECT name, CASE WHEN LEFT(UPPER(name), 1) IN ('0','1','2','3','4','5','6','7','8','9') THEN 1 ELSE 0 END AS "num", CASE WHEN LEFT(UPPER(name), 1) IN ('0','1','2','3','4','5','6','7','8','9') THEN 0 ELSE 1 END AS "letter" 
FROM accounts) table_1
```
 - > Q4. Consider vowels as a, e, i, o, and u. What proportion of company names start with a vowel, and what percent start with anything else?
```
SELECT SUM("vow") "vowels", SUM("other") "others"
FROM 
(SELECT name, CASE WHEN LEFT(UPPER(name), 1) IN ('A','E','I','O','U') THEN 1 ELSE 0 END AS "vow", CASE WHEN LEFT(UPPER(name), 1) IN ('A','E','I','O','U') THEN 0 ELSE 1 END AS "other" 
FROM accounts) table_2
```
### POSITION( ) , STRPOS( )
 - What if…separate city and state? Figuring out where the city and state split is important since it will be different for each row.  
 - *position( ) or strpos( ) returns a numerical value equal to how far away from the left that particular character appears. It takes a character and a column, and provides the index where that character is for each row.
```
SELECT position(',' IN city_state) "comma_pos", strpos(city_state, '-') "hyphen_pos"
```
 - > Q5. Use the accounts table to create a first and last name column that hold the first and last names for the primary_poc.
```
SELECT left(primary_poc, strpos(primary_poc, ' ') - 1) "first", right(primary_poc, length(primary_poc) - strpos(primary_poc, ' ')) "last"
FROM accounts
```
 - > Q6. Now see if you can do the same thing for all of the name of each rep in the sales_rep table. Again provide a first and last name column.
```
SELECT left(name, strpos(name, ' ') - 1) "first", right(name, length(name) - strpos(name, ' ')) "last"
FROM sales_reps
```
### CONCAT( ) or piping..||
 - It allow you to combine columns together across rows.
```
SELECT first_c, last_c, CONCAT(first_c, '  ', last_c) AS "full_c"
SELECT first_c, last_c, first_c || '  ' || last_c AS "full_c"
```
 - > Q7. Each company in the accounts table wants to create an email address for each primary_poc. The email address should be the first name of the primary_poc . last name of the primary_poc @ company name . com.
```
WITH t1 AS 
(SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') -1 ) "first",  RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) "last", name
FROM accounts)
SELECT "first", "last", CONCAT("first", '.', "last", '@', name, '.com')
FROM t1
```
 - > Q8. You may have noticed that in the previous solution some of the company names have spaces, which will certainly not work as an email address. See if you can create an email that will work by removing all of the spaces in the account name, but otherwise, your solution should be just as in question 1. ( https://www.postgresql.org/docs/8.1/static/functions-string.html ) 
```
WITH t1 AS 
(SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') -1 ) "first",  RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) "last", name
FROM accounts)
SELECT "first", "last", CONCAT("first", '.', "last", '@', REPLACE(name, ' ', ''), '.com')
FROM t1
```
 - > Q9. We would also like to create an initial password. The first password will be the first letter of the primary_poc's first name (lowercase), then the last letter of their first name (lowercase), the first letter of their last name (lowercase), the last letter of their last name (lowercase), the number of letters in their first name, the number of letters in their last name, and then the name of the company they are working with all capitalized with no spaces.
```
WITH t1 AS 
(SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') -1 ) "first",  RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) "last", name
FROM accounts)
SELECT "first", "last", CONCAT("first", '.', "last", '@', REPLACE(name, ' ', ''), '.com') "email", CONCAT(left(lower("first"), 1), right(lower("first"), 1), left(lower("last"), 1), right(lower("last"), 1), length("first"), length("last"), replace(upper(name), ' ', '')) "pw"
FROM t1
```
### DATE_PART( ) , TO_DATE( ), CAST( ) 
 - For getting values from dates, 
 - *DATE_PART(format, str): to pull a specific portion of a date (pulling ‘month’ or ‘dow’(dayofweek)). For grouping for certain components..... SELECT DATE_PART('month', column) AS "new_column"
 - *TO_DATE(str, format): collecting date values then creating a correct format for “dates” 
``` 
SELECT DATE_PART('month', TO_date(column, 'month')) AS "clean_month"

#changed a month name into the number associated with that particular month (January=1)
```



























