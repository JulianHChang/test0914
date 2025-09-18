# Chapter 6. Working with Strings

[]()This chapter focuses on string manipulation in SQL. Keep in mind that SQL is not designed to perform complex string manipulation, and you can (and will) find working with strings in SQL to be cumbersome and frustrating at times. Despite SQL’s limitations, there are some useful built-in functions provided by the different DBMSs, and we’ve tried to use them in creative ways. This chapter in particular is representative of the message we tried to convey in the introduction; SQL is the good, the bad, and the ugly. Hopefully you take away from this chapter a better appreciation for what can and can’t be done in SQL when working with strings. In many cases you’ll be surprised by how easy parsing and transforming strings can be, while at other times you’ll be aghast by the kind of SQL that is necessary to accomplish a particular task.

[]()[]()Many of the recipes that follow use the TRANSLATE and REPLACE functions that are now available in all the DBMSs covered in this book, with the exception of MySQL, which only has `replace`. In this last case, it is worth noting early on that you can replicate the effect of TRANSLATE by using nested REPLACE functions.

The first recipe in this chapter is critically important, as it is leveraged by several of the subsequent solutions. In many cases, you’d like to have the ability to traverse a string by moving through it a character at a time. Unfortunately, SQL does not make this easy. []()Because there is limited loop functionality in SQL, you need to mimic a loop to traverse a string. We call this operation “walking a string” or “walking through a string,” and the very first recipe explains the technique. This is a fundamental operation in string parsing when using SQL, and is referenced and used by almost all recipes in this chapter. We strongly suggest becoming comfortable with how the technique works.

# 6.1 Walking a String

## Problem

[]()[]()You want to traverse a string to return each character as a row, but SQL lacks a loop operation. For example, you want to display the ENAME “KING” from table EMP as four rows, where each row contains just characters from KING.

## Solution

[]()Use a Cartesian product to generate the number of rows needed to return each character of a string on its own line. Then use your DBMS’s built-in string parsing function to extract the characters you are interested in (SQL Server users will use SUBSTRING instead of SUBSTR and DATALENGTH instead of LENGTH):

```
1 select substr(e.ename,iter.pos,1) as C 
 2   from (select ename from emp where ename = 'KING') e, 
 3        (select id as pos from t10) iter 
 4  where iter.pos <= length(e.ename) 


C
-
K
I
N
G
```

## Discussion

The key to iterating through a string’s characters is to join against a table that has enough rows to produce the required number of iterations. This example uses table T10, which contains 10 rows (it has one column, ID, holding the values 1 through 10). The maximum number of rows that can be returned from this query is 10.

The following example shows the Cartesian product between E and ITER (i.e., between the specific name and the 10 rows from T10) without parsing ENAME:

```
select ename, iter.pos 
   from (select ename from emp where ename = 'KING') e, 
        (select id as pos from t10) iter 

ENAME             POS
---------- ----------
KING                1
KING                2
KING                3
KING                4
KING                5
KING                6
KING                7
KING                8
KING                9
KING               10
```

The cardinality of inline view E is 1, and the cardinality of inline view ITER is 10. The Cartesian product is then 10 rows. Generating such a product is the first step in mimicking a loop in SQL.

###### Tip

It is common practice to refer to table T10 as a “pivot” table.

The solution uses a WHERE clause to break out of the loop after four rows have been returned. To restrict the result set to the same number of rows as there are characters in the name, that WHERE clause specifies ITER.POS &lt;= LENGTH(E. ENAME) as the condition:

```
select ename, iter.pos 
   from (select ename from emp where ename = 'KING') e, 
        (select id as pos from t10) iter 
  where iter.pos <= length(e.ename) 

ENAME             POS
---------- ----------
KING                1
KING                2
KING                3
KING                4
```

Now that you have one row for each character in E.ENAME, you can use ITER.POS as a parameter to SUBSTR, allowing you to navigate through the characters in the string. ITER.POS increments with each row, and thus each row can be made to return a successive character from E.ENAME. This is how the solution example works.

Depending on what you are trying to accomplish, you may or may not need to generate a row for every single character in a string. The following query is an example of walking E.ENAME and exposing different portions (more than a single character) of the string:

```
select substr(e.ename,iter.pos) a,
       substr(e.ename,length(e.ename)-iter.pos+1) b
  from (select ename from emp where ename = 'KING') e,
       (select id pos from t10) iter
 where iter.pos <= length(e.ename)


A          B
---------- ----------
KING       G
ING        NG
NG         ING
G          KING
```

The most common scenarios for the recipes in this chapter involve walking the whole string to generate a row for each character in the string, or walking the string such that the number of rows generated reflects the number of particular characters or delimiters that are present in the string.[]()[]()

# 6.2 Embedding Quotes Within String Literals

## Problem

[]()[]()You want to embed quote marks within string literals. You would like to produce results such as the following with SQL:

```
QMARKS
--------------
g'day mate
beavers' teeth
'
```

## Solution

[]()The following three SELECTs highlight different ways you can create quotes: in the middle of a string and by themselves:

```
1 select 'g''day mate' qmarks from t1 union all
2 select 'beavers'' teeth'    from t1 union all
3 select ''''                 from t1
```

## Discussion

When working with quotes, it’s often useful to think of them like parentheses. When you have an opening parenthesis, you must always have a closing parenthesis. The same goes for quotes. Keep in mind that you should always have an even number of quotes across any given string. To embed a single quote within a string, you need to use two quotes:

```
select 'apples core', 'apple''s core', 
         case when '' is null then 0 else 1 end 
   from t1 

 'APPLESCORE 'APPLE''SCOR CASEWHEN''ISNULLTHEN0ELSE1END
 ----------- ------------ -----------------------------
 apples core apple's core                             0
```

The following is the solution stripped down to its bare elements. You have two outer quotes defining a string literal, and within that string literal, you have two quotes that together represent just one quote in the string that you actually get:

```
select '''' as quote from t1 

Q
-
'
```

When working with quotes, be sure to remember that a string literal comprising two quotes alone, with no intervening characters, is NULL.

# 6.3 Counting the Occurrences of a Character in a String

## Problem

[]()You want to count the number of times a character or substring occurs within a given string. Consider the following string:

```
10,CLARK,MANAGER
```

You want to determine how many commas are in the string.

## Solution

Subtract the length of the string without the commas from the original length of the string to determine the number of commas in the string. Each DBMS provides functions for obtaining the length of a string and removing characters from a string. []()In most cases, these functions are LENGTH and REPLACE, respectively (SQL Server users will use the built-in function LEN rather than LENGTH):

```
1 select (length('10,CLARK,MANAGER')-
2        length(replace('10,CLARK,MANAGER',',','')))/length(',')
3        as cnt
4   from t1
```

## Discussion

You arrive at the solution by using simple subtraction. The call to LENGTH on line 1 returns the original size of the string, and the first call to LENGTH on line 2 returns the size of the string without the commas, which are removed by REPLACE.

By subtracting the two lengths, you obtain the difference in terms of characters, which is the number of commas in the string. The last operation divides the difference by the length of your search string. This division is necessary if the string you are looking for has a length greater than 1. In the following example, counting the occurrence of “LL” in the string “HELLO HELLO” without dividing will return an incorrect result:

```
select 
        (length('HELLO HELLO')- 
        length(replace('HELLO HELLO','LL','')))/length('LL') 
        as correct_cnt, 
        (length('HELLO HELLO')- 
        length(replace('HELLO HELLO','LL',''))) as incorrect_cnt 
   from t1 

CORRECT_CNT INCORRECT_CNT
----------- -------------
          2             4
```

# 6.4 Removing Unwanted Characters from a String

## Problem

[]()You want to remove specific characters from your data. A scenario where this may occur is in dealing with badly formatted numeric data, especially currency data, where commas have been used to separate zeros, and currency markers are mixed in the column with the quantity. Another scenario is that you want to export data from your database as a CSV file, but there is a text field containing commas, which will be read as separators when the CSV file is accessed. Consider this result set:

```
ENAME             SAL
---------- ----------
SMITH             800
ALLEN            1600
WARD             1250
JONES            2975
MARTIN           1250
BLAKE            2850
CLARK            2450
SCOTT            3000
KING             5000
TURNER           1500
ADAMS            1100
JAMES             950
FORD             3000
MILLER           1300
```

You want to remove all zeros and vowels as shown by the following values in columns STRIPPED1 and STRIPPED2:

```
ENAME      STRIPPED1         SAL STRIPPED2
---------- ---------- ---------- ---------
SMITH      SMTH              800 8
ALLEN      LLN              1600 16
WARD       WRD              1250 125
JONES      JNS              2975 2975
MARTIN     MRTN             1250 125
BLAKE      BLK              2850 285
CLARK      CLRK             2450 245
SCOTT      SCTT             3000 3
KING       KNG              5000 5
TURNER     TRNR             1500 15
ADAMS      DMS              1100 11
JAMES      JMS               950 95
FORD       FRD              3000 3
MILLER     MLLR             1300 13
```

## Solution

Each DBMS provides functions for removing unwanted characters from a string. The functions REPLACE and TRANSLATE are most useful for this problem.

### DB2, Oracle, PostgreSQL, and SQL Server

Use the built-in functions TRANSLATE and REPLACE to remove unwanted characters and strings:

```
1 select ename,
2        'replace (translate(ename,'AEIOU', 'aaaaa'), 'a', '') as stripped1,
3        sal,
4        replace(cast(sal as char(4)),'0','') as stripped2
5   from emp
```

Note that for DB2, the AS keyword is optional for assigning a column alias and can be left out.

### MySQL

MySQL does not offer a TRANSLATE function, so several calls to REPLACE are needed:

```
 1 select ename,
 2        replace(
 3        replace(
 4        replace(
 5        replace(
 6        replace(ename,'A',''),'E',''),'I',''),'O',''),'U','')
 7        as stripped1,
 8        sal,
 9        replace(sal,0,'') stripped2
10   from emp
```

## Discussion

The built-in function REPLACE removes all occurrences of zeros. To remove the vowels, use TRANSLATE to convert all vowels into one specific character (we used “a”; you can use any character); then use REPLACE to remove all occurrences of that character.[]()

# 6.5 Separating Numeric and Character Data

## Problem

[]()You have numeric data stored with character data together in one column. This could easily happen if you inherit data where units of measurement or currency have been stored with their quantity (e.g., a column with *100 km*, *AUD$200*, or *40 pounds*, rather than either the column making the units clear or a separate column showing the units where necessary).

You want to separate the character data from the numeric data. Consider the following result set:

```
DATA
---------------
SMITH800
ALLEN1600
WARD1250
JONES2975
MARTIN1250
BLAKE2850
CLARK2450
SCOTT3000
KING5000
TURNER1500
ADAMS1100
JAMES950
FORD3000
MILLER1300
```

You would like the result to be:

```
ENAME             SAL
---------- ----------
SMITH             800
ALLEN            1600
WARD             1250
JONES            2975
MARTIN           1250
BLAKE            2850
CLARK            2450
SCOTT            3000
KING             5000
TURNER           1500
ADAMS            1100
JAMES             950
FORD             3000
MILLER           1300
```

## Solution

Use the built-in functions TRANSLATE and REPLACE to isolate the character from the numeric data. Like other recipes in this chapter, the trick is to use TRANSLATE to transform multiple characters into a single character you can reference. This way you are no longer searching for multiple numbers or characters; rather, you are searching for just one character to represent all numbers or one character to represent all characters.

### DB2

Use the functions TRANSLATE and REPLACE to isolate and separate the numeric from the character data:

```
 1 select replace(
 2      translate(data,'0000000000','0123456789'),'0','') ename,
 3        cast(
 4      replace(
 5    translate(lower(data),repeat('z',26),
 6           'abcdefghijklmnopqrstuvwxyz'),'z','') as integer) sal
 7    from (
 8  select ename||cast(sal as char(4)) data
 9    from emp
10         ) x
```

### Oracle

Use the functions TRANSLATE and REPLACE to isolate and separate the numeric from the character data:

```
 1 select replace(
 2      translate(data,'0123456789','0000000000'),'0') ename,
 3      to_number(
 4        replace(
 5        translate(lower(data),
 6                  'abcdefghijklmnopqrstuvwxyz',
 7                   rpad('z',26,'z')),'z')) sal
 8   from (
 9 select ename||sal data
10   from emp
11        )
```

### PostgreSQL

Use the functions TRANSLATE and REPLACE to isolate and separate the numeric from the character data:

```
 1 select replace(
 2      translate(data,'0123456789','0000000000'),'0','') as ename,
 3           cast(
 4        replace(
 5      translate(lower(data),
 6                'abcdefghijklmnopqrstuvwxyz',
 7                rpad('z',26,'z')),'z','') as integer) as sal
 8   from (
 9 select ename||sal as data
10   from emp
11        ) x
```

### SQL Server

Use the functions TRANSLATE and REPLACE to isolate and separate the numeric from the character data:

```
 1 select replace(
 2      translate(data,'0123456789','0000000000'),'0','') as ename,
 3           cast(
 4        replace(
 5      translate(lower(data),
 6                'abcdefghijklmnopqrstuvwxyz',
 7                replicate('z',26),'z','') as integer) as sal
 8   from (
 9 select concat(ename,sal) as data
10   from emp
11        ) x
```

## Discussion

The syntax is a bit different for each DBMS, but the technique is the same. The syntax is slightly different for each DBMS, but the technique is the same; we will use the Oracle solution for this discussion. The key to solving this problem is to isolate the numeric and character data. You can use TRANSLATE and REPLACE to do this. To extract the numeric data, first isolate all character data using TRANSLATE:

```
select data, 
        translate(lower(data), 
                 'abcdefghijklmnopqrstuvwxyz', 
                 rpad('z',26,'z')) sal 
   from (select ename||sal data from emp) 

DATA                 SAL
-------------------- -------------------
SMITH800             zzzzz800
ALLEN1600            zzzzz1600
WARD1250             zzzz1250
JONES2975            zzzzz2975
MARTIN1250           zzzzzz1250
BLAKE2850            zzzzz2850
CLARK2450            zzzzz2450
SCOTT3000            zzzzz3000
KING5000             zzzz5000
TURNER1500           zzzzzz1500
ADAMS1100            zzzzz1100
JAMES950             zzzzz950
FORD3000             zzzz3000
MILLER1300           zzzzzz1300
```

By using TRANSLATE you convert every nonnumeric character into a lowercase Z. The next step is to remove all instances of lowercase Z from each record using REPLACE, leaving only numerical characters that can then be cast to a number:

```
select data, 
        to_number( 
          replace( 
        translate(lower(data), 
                  'abcdefghijklmnopqrstuvwxyz', 
                  rpad('z',26,'z')),'z')) sal 
   from (select ename||sal data from emp) 

 DATA                        SAL
 -------------------- ----------
 SMITH800                    800
 ALLEN1600                  1600
 WARD1250                   1250
 JONES2975                  2975
 MARTIN1250                 1250
 BLAKE2850                  2850
 CLARK2450                  2450
 SCOTT3000                  3000
 KING5000                   5000
 TURNER1500                 1500
 ADAMS1100                  1100
 JAMES950                    950
 FORD3000                   3000
 MILLER1300                 1300
```

To extract the nonnumeric characters, isolate the numeric characters using TRANSLATE:

```
select data, 
        translate(data,'0123456789','0000000000') ename 
   from (select ename||sal data from emp) 

 DATA                 ENAME
 -------------------- ----------
 SMITH800             SMITH000
 ALLEN1600            ALLEN0000
 WARD1250             WARD0000
 JONES2975            JONES0000
 MARTIN1250           MARTIN0000
 BLAKE2850            BLAKE0000
 CLARK2450            CLARK0000
 SCOTT3000            SCOTT0000
 KING5000             KING0000
 TURNER1500           TURNER0000
 ADAMS1100            ADAMS0000
 JAMES950             JAMES000
 FORD3000             FORD0000
 MILLER1300           MILLER0000
```

By using TRANSLATE, you convert every numeric character into a zero. The next step is to remove all instances of zero from each record using REPLACE, leaving only nonnumeric characters:

```
select data, 
        replace(translate(data,'0123456789','0000000000'),'0') ename 
   from (select ename||sal data from emp) 

 DATA                 ENAME
 -------------------- -------
 SMITH800             SMITH
 ALLEN1600            ALLEN
 WARD1250             WARD
 JONES2975            JONES
 MARTIN1250           MARTIN
 BLAKE2850            BLAKE
 CLARK2450            CLARK
 SCOTT3000            SCOTT
 KING5000             KING
 TURNER1500           TURNER
 ADAMS1100            ADAMS
 JAMES950             JAMES
 FORD3000             FORD
 MILLER1300           MILLER
```

Put the two techniques together and you have your solution.[]()

# 6.6 Determining Whether a String Is Alphanumeric

## Problem

[]()[]()You want to return rows from a table only when a column of interest contains no characters other than numbers and letters. Consider the following view V (SQL Server users will use the operator + for concatenation instead of ||):

```
create view V as
select ename as data
  from emp
 where deptno=10
 union all
select ename||', $'|| cast(sal as char(4)) ||'.00' as data
  from emp
 where deptno=20
 union all
select ename|| cast(deptno as char(4)) as data
  from emp
 where deptno=30
```

The view V represents your table, and it returns the following:

```
DATA
--------------------
CLARK
KING
MILLER
SMITH, $800.00
JONES, $2975.00
SCOTT, $3000.00
ADAMS, $1100.00
FORD, $3000.00
ALLEN30
WARD30
MARTIN30
BLAKE30
TURNER30
JAMES30
```

However, from the view’s data you want to return only the following records:

```
DATA
-------------
CLARK
KING
MILLER
ALLEN30
WARD30
MARTIN30
BLAKE30
TURNER30
JAMES30
```

In short, you want to omit those rows containing data other than letters and digits.

## Solution

It may seem intuitive at first to solve the problem by searching for all the possible non-alphanumeric characters that can be found in a string, but, on the contrary, you will find it easier to do the exact opposite: find all the alphanumeric characters. By doing so, you can treat all the alphanumeric characters as one by converting them to one single character. The reason you want to do this is so the alphanumeric characters can be manipulated together, as a whole. Once you’ve generated a copy of the string in which all alphanumeric characters are represented by a single character of your choosing, it is easy to isolate the alphanumeric characters from any other characters.

### DB2

Use the function TRANSLATE to convert all alphanumeric characters to a single character; then identify any rows that have characters other than the converted alphanumeric character. For DB2 users, the CAST function calls in view V are necessary; otherwise, the view cannot be created due to type conversion errors. Take extra care when working with casts to CHAR as they are fixed length (padded):

```
1 select data
2   from V
3  where translate(lower(data),
4                  repeat('a',36),
5                  '0123456789abcdefghijklmnopqrstuvwxyz') =
6                  repeat('a',length(data))
```

### MySQL

The syntax for view V is slightly different in MySQL:

```
create view V as
select ename as data
  from emp
 where deptno=10
 union all
select concat(ename,', $',sal,'.00') as data
  from emp
 where deptno=20
 union all
select concat(ename,deptno) as data
  from emp
 where deptno=30
```

Use a regular expression to easily find rows that contain non-alphanumeric data:

```
1 select data
2   from V
3  where data regexp '[^0-9a-zA-Z]' = 0
```

### Oracle and PostgreSQL

Use the function TRANSLATE to convert all alphanumeric characters to a single character; then identify any rows that have characters other than the converted alphanumeric character. The CAST function calls in view V are not needed for Oracle and PostgreSQL. Take extra care when working with casts to CHAR as they are fixed length (padded).

If you decide to cast, cast to VARCHAR or VARCHAR2:

```
1 select data
2   from V
3  where translate(lower(data),
4                  '0123456789abcdefghijklmnopqrstuvwxyz',
5                  rpad('a',36,'a')) = rpad('a',length(data),'a')
```

### SQL Server

The technique is the same, with the exception of there being no RPAD in SQL Server:

```
1 select data
2   from V
3  where translate(lower(data),
4                  '0123456789abcdefghijklmnopqrstuvwxyz',
5                  replicate('a',36)) = replicate('a',len(data))
```

## Discussion

The key to these solutions is being able to reference multiple characters concurrently. By using the function TRANSLATE, you can easily manipulate all numbers or all characters without having to “iterate” and inspect each character one by one.

### DB2, Oracle, PostgreSQL, and SQL Server

Only 9 of the 14 rows from view V are alphanumeric. To find the rows that are alphanumeric only, simply use the function TRANSLATE. In this example, TRANSLATE converts characters 0–9 and a–z to “a”. Once the conversion is done, the converted row is then compared with a string of all “a” with the same length (as the row). If the length is the same, then you know all the characters are alphanumeric and nothing else.

By using the TRANSLATE function (using the Oracle syntax):

```
where translate(lower(data),
                '0123456789abcdefghijklmnopqrstuvwxyz',
                 rpad('a',36,'a'))
```

you convert all numbers and letters into a distinct character (we chose “a”). Once the data is converted, all strings that are indeed alphanumeric can be identified as a string comprising only a single character (in this case, “a”). This can be seen by running TRANSLATE by itself:

```
select data, translate(lower(data), 
                   '0123456789abcdefghijklmnopqrstuvwxyz', 
                    rpad('a',36,'a')) 
   from V 

DATA                 TRANSLATE(LOWER(DATA)
-------------------- ---------------------
CLARK                aaaaa
…
SMITH, $800.00       aaaaa, $aaa.aa
…
ALLEN30              aaaaaaa
…
```

The alphanumeric values are converted, but the string lengths have not been modified. Because the lengths are the same, the rows to keep are the ones for which the call to TRANSLATE returns all “a"s. You keep those rows, rejecting the others, by comparing each original string’s length with the length of its corresponding string of “a"s:

```
select data, translate(lower(data), 
                   '0123456789abcdefghijklmnopqrstuvwxyz', 
                    rpad('a',36,'a')) translated, 
         rpad('a',length(data),'a') fixed 
   from V 

DATA                 TRANSLATED           FIXED
-------------------- -------------------- ----------------
CLARK                aaaaa                aaaaa
…
SMITH, $800.00       aaaaa, $aaa.aa       aaaaaaaaaaaaaa
…
ALLEN30              aaaaaaa              aaaaaaa
…
```

The last step is to keep only the strings where TRANSLATED equals FIXED.

### MySQL

[]()The expression in the WHERE clause:

```
where data regexp '[^0-9a-zA-Z]' = 0
```

causes rows that have only numbers or characters to be returned. The value ranges in the brackets, “0-9a-zA-Z”, represent all possible numbers and letters. The character ^ is for negation, so the expression can be stated as “not numbers or letters.” A return value of 1 is true and 0 is false, so the whole expression can be stated as “return rows where anything other than numbers and letters is false.”[]()[]()

# 6.7 Extracting Initials from a Name

## Problem

[]()[]()You want convert a full name into initials. Consider the following name:

```
Stewie Griffin
```

You would like to return:

```
S.G.
```

## Solution

It’s important to keep in mind that SQL does not provide the flexibility of languages such as C or Python; therefore, creating a generic solution to deal with any name format is not something particularly easy to do in SQL. The solutions presented here expect the names to be either first and last name, or first, middle name/middle initial, and last name.

### DB2

[]()Use the built-in functions REPLACE, TRANSLATE, and REPEAT to extract the initials:

```
1 select replace(
2        replace(
3        translate(replace('Stewie Griffin', '.', ''),
4                  repeat('#',26),
5                  'abcdefghijklmnopqrstuvwxyz'),
6                   '#','' ), ' ','.' )
7                  ||'.'
8   from t1
```

### MySQL

[]()[]()[]()[]()Use the built-in functions CONCAT, CONCAT\_WS, SUBSTRING, and SUBSTRING_ INDEX to extract the initials:

```
 1 select case
 2          when cnt = 2 then
 3            trim(trailing '.' from
 4                 concat_ws('.',
 5                  substr(substring_index(name,' ',1),1,1),
 6                  substr(name,
 7                         length(substring_index(name,' ',1))+2,1),
 8                  substr(substring_index(name,' ',-1),1,1),
 9                  '.'))
10          else
11            trim(trailing '.' from
12                 concat_ws('.',
13                  substr(substring_index(name,' ',1),1,1),
14                  substr(substring_index(name,' ',-1),1,1)
15                  ))
16          end as initials
17   from (
18 select name,length(name)-length(replace(name,' ','')) as cnt
19   from (
20 select replace('Stewie Griffin','.','') as name from t1
21        )y
22        )x
```

### Oracle and PostgreSQL

[]()Use the built-in functions REPLACE, TRANSLATE, and RPAD to extract the initials:

```
1 select replace(
2        replace(
3        translate(replace('Stewie Griffin', '.', ''),
4                  'abcdefghijklmnopqrstuvwxyz',
5                  rpad('#',26,'#') ), '#','' ),' ','.' ) ||'.'
6   from t1
```

### SQL Server

```
1 select replace(
2        replace(
3        translate(replace('Stewie Griffin', '.', ''),
4                  'abcdefghijklmnopqrstuvwxyz',
5                  replicate('#',26) ), '#','' ),' ','.' ) + '.'
6   from t1
```

## Discussion

By isolating the capital letters, you can extract the initials from a name. The following sections describe each vendor-specific solution in detail.

### DB2

The REPLACE function will remove any periods in the name (to handle middle initials), and the TRANSLATE function will convert all non-uppercase letters to #.

```
select translate(replace('Stewie Griffin', '.', ''), 
                  repeat('#',26), 
                  'abcdefghijklmnopqrstuvwxyz') 
   from t1 

TRANSLATE('STE
--------------
S##### G######
```

At this point, the initials are the characters that are not #. The function REPLACE is then used to remove all the # characters:

```
select replace( 
        translate(replace('Stewie Griffin', '.', ''), 
                   repeat('#',26), 
                   'abcdefghijklmnopqrstuvwxyz'),'#','') 
   from t1 


REP
---
S G
```

The next step is to replace the white space with a period by using REPLACE again:

```
select replace( 
 	       replace( 
 	       translate(replace('Stewie Griffin', '.', ''), 
 	                 repeat('#',26), 
 	                'abcdefghijklmnopqrstuvwxyz'),'#',''),' ','.') || '.' 
 	  from t1 

	REPLA
	-----
	S.G
```

The final step is to append a decimal to the end of the initials.

### Oracle and PostgreSQL

The REPLACE function will remove any periods in the name (to handle middle initials), and the TRANSLATE function will convert all non-uppercase letters to *#*.

```
select translate(replace('Stewie Griffin','.',''), 
                  'abcdefghijklmnopqrstuvwxyz', 
                  rpad('#',26,'#')) 
   from t1 

TRANSLATE('STE
--------------
S##### G######
```

At this point, the initials are the characters that are not #. The function REPLACE is then used to remove all the # characters:

```
select replace( 
        translate(replace('Stewie Griffin','.',''), 
                  'abcdefghijklmnopqrstuvwxyz', 
                   rpad('#',26,'#')),'#','') 
   from t1 

REP
---
S G
```

The next step is to replace the white space with a period by using REPLACE again:

```
select replace( 
        replace( 
      translate(replace('Stewie Griffin','.',''), 
                'abcdefghijklmnopqrstuvwxyz', 
                rpad('#',26,'#') ),'#',''),' ','.') || '.' 
   from t1 

REPLA
-----
S.G
```

The final step is to append a decimal to the end of the initials.

### MySQL

The inline view Y is used to remove any period from the name. []()The inline view X finds the number of white spaces in the name so the SUBSTR function can be called the correct number of times to extract the initials. []()The three calls to SUBSTRING_ INDEX parse the string into individual names based on the location of the white space. Because there is only a first and last name, the code in the ELSE portion of the case statement is executed:

```
select substr(substring_index(name, ' ',1),1,1) as a, 
        substr(substring_index(name,' ',-1),1,1) as b 
   from (select 'Stewie Griffin' as name from t1) x 

A B
- -
S G
```

If the name in question has a middle name or initial, the initial would be returned by executing:

```
substr(name,length(substring_index(name, ' ',1))+2,1)
```

which finds the end of the first name and then moves two spaces to the beginning of the middle name or initial, that is, the start position for SUBSTR. Because only one character is kept, the middle name or initial is successfully returned. []()The initials are then passed to CONCAT\_WS, which separates the initials by a period:

```
select concat_ws('.', 
                  substr(substring_index(name, ' ',1),1,1), 
                  substr(substring_index(name,' ',-1),1,1), 
                  '.' ) a 
   from (select 'Stewie Griffin' as name from t1) x 

A
-----
S.G..
```

The last step is to trim the extraneous period from the initials.[]()[]()

# 6.8 Ordering by Parts of a String

## Problem

[]()You want to order your result set based on a substring. Consider the following records:

```
ENAME
----------
SMITH
ALLEN
WARD
JONES
MARTIN
BLAKE
CLARK
SCOTT
KING
TURNER
ADAMS
JAMES
FORD
MILLER
```

You want the records to be ordered based on the *last* two characters of each name:

```
ENAME
---------
ALLEN
TURNER
MILLER
JONES
JAMES
MARTIN
BLAKE
ADAMS
KING
WARD
FORD
CLARK
SMITH
SCOTT
```

## Solution

The key to this solution is to find and use your DBMS’s built-in function to extract the substring on which you want to sort. This is typically done with the SUBSTR function.

### DB2, Oracle, MySQL, and PostgreSQL

[]()[]()Use a combination of the built-in functions LENGTH and SUBSTR to order by a specific part of a string:

```
1 select ename
2   from emp
3  order by substr(ename,length(ename)-1,)
```

### SQL Server

[]()[]()Use functions SUBSTRING and LEN to order by a specific part of a string:

```
1 select ename
2   from emp
3  order by substring(ename,len(ename)-1,2)
```

## Discussion

By using a SUBSTR expression in your ORDER BY clause, you can pick any part of a string to use in ordering a result set. You’re not limited to SUBSTR either. You can order rows by the result of almost any expression.[]()

# 6.9 Ordering by a Number in a String

## Problem

[]()You want order your result set based on a number within a string. Consider the following view:

```
create view V as
select e.ename ||' '||
        cast(e.empno as char(4))||' '||
        d.dname as data
  from emp e, dept d
 where e.deptno=d.deptno
```

This view returns the following data:

```
DATA
 ----------------------------
 CLARK   7782 ACCOUNTING
 KING    7839 ACCOUNTING
 MILLER  7934 ACCOUNTING
 SMITH   7369 RESEARCH
 JONES   7566 RESEARCH
 SCOTT   7788 RESEARCH
 ADAMS   7876 RESEARCH
 FORD    7902 RESEARCH
 ALLEN   7499 SALES
 WARD    7521 SALES
 MARTIN  7654 SALES
 BLAKE   7698 SALES
 TURNER  7844 SALES
 JAMES   7900 SALES
```

You want to order the results based on the employee number, which falls between the employee name and respective department:

```
DATA
---------------------------
SMITH    7369 RESEARCH
ALLEN    7499 SALES
WARD     7521 SALES
JONES    7566 RESEARCH
MARTIN   7654 SALES
BLAKE    7698 SALES
CLARK    7782 ACCOUNTING
SCOTT    7788 RESEARCH
KING     7839 ACCOUNTING
TURNER   7844 SALES
ADAMS    7876 RESEARCH
JAMES    7900 SALES
FORD     7902 RESEARCH
MILLER   7934 ACCOUNTING
```

## Solution

Each solution uses functions and syntax specific to its DBMS, but the method (making use of the built-in functions REPLACE and TRANSLATE) is the same for each. The idea is to use REPLACE and TRANSLATE to remove nondigits from the strings, leaving only the numeric values upon which to sort.

### DB2

Use the built-in functions REPLACE and TRANSLATE to order by numeric characters in a string:

```
1 select data
2   from V
3  order by
4         cast(
5      replace(
6    translate(data,repeat('#',length(data)),
7      replace(
8    translate(data,'##########','0123456789'),
9             '#','')),'#','') as integer)
```

### Oracle

Use the built-in functions REPLACE and TRANSLATE to order by numeric characters in a string:

```
1 select data
2   from V
3  order by
4         to_number(
5           replace(
6         translate(data,
7           replace(
8         translate(data,'0123456789','##########'),
9                  '#'),rpad('#',20,'#')),'#'))
```

### PostgreSQL

Use the built-in functions REPLACE and TRANSLATE to order by numeric characters in a string:

```
1 select data
2   from V
3  order by
4         cast(
5      replace(
6    translate(data,
7      replace(
8    translate(data,'0123456789','##########'),
9             '#',''),rpad('#',20,'#')),'#','') as integer)
```

### MySQL

As of the time of this writing, MySQL does not provide the TRANSLATE function.

## Discussion

The purpose of view V is only to supply rows on which to demonstrate this recipe’s solution. The view simply concatenates several columns from the EMP table. The solution shows how to take such concatenated text as input and sort it by the employee number embedded within.

The ORDER BY clause in each solution may look intimidating, but it performs quite well and is straightforward once you examine it piece by piece. To order by the numbers in the string, it’s easiest to remove any characters that are not numbers. Once the nonnumeric characters are removed, all that is left to do is cast the string of numerals into a number and then sort as you see fit. Before examining each function call, it is important to understand the order in which each function is called. Starting with the innermost call, TRANSLATE (line 8 from each of the original solutions), you see that:

From the innermost call, the sequence of steps is TRANSLATE (line 8); REPLACE (line 7) ; TRANSLATE (line 6); REPLACE (line 5). The final step is to use CAST to return the result as a number.

The first step is to convert the numbers into characters that do not exist in the rest of the string. For this example, we chose # and used TRANSLATE to convert all nonnumeric characters into occurrences of #. For example, the following query shows the original data on the left and the results from the first translation:

```
select data, 
        translate(data,'0123456789','##########') as tmp 
   from V 

DATA                           TMP
 ------------------------------ -----------------------
 CLARK   7782 ACCOUNTING        CLARK   #### ACCOUNTING
 KING    7839 ACCOUNTING        KING    #### ACCOUNTING
 MILLER  7934 ACCOUNTING        MILLER  #### ACCOUNTING
 SMITH   7369 RESEARCH          SMITH   #### RESEARCH
 JONES   7566 RESEARCH          JONES   #### RESEARCH
 SCOTT   7788 RESEARCH          SCOTT   #### RESEARCH
 ADAMS   7876 RESEARCH          ADAMS   #### RESEARCH
 FORD    7902 RESEARCH          FORD    #### RESEARCH
 ALLEN   7499 SALES             ALLEN   #### SALES
 WARD    7521 SALES             WARD    #### SALES
 MARTIN  7654 SALES             MARTIN  #### SALES
 BLAKE   7698 SALES             BLAKE   #### SALES
 TURNER 7844 SALES              TURNER  #### SALES
 JAMES  7900 SALES              JAMES   #### SALES
```

TRANSLATE finds the numerals in each string and converts each one to the # character. The modified strings are then returned to REPLACE (line 11), which removes all occurrences of #:

```
select data, 
 replace( 
 translate(data,'0123456789','##########'),'#') as tmp 
   from V 

DATA                           TMP
 ------------------------------ -------------------
 CLARK   7782 ACCOUNTING        CLARK   ACCOUNTING
 KING    7839 ACCOUNTING        KING    ACCOUNTING
 MILLER  7934 ACCOUNTING        MILLER  ACCOUNTING
 SMITH   7369 RESEARCH          SMITH   RESEARCH
 JONES   7566 RESEARCH          JONES   RESEARCH
 SCOTT   7788 RESEARCH          SCOTT   RESEARCH
 ADAMS   7876 RESEARCH          ADAMS   RESEARCH
 FORD    7902 RESEARCH          FORD    RESEARCH
 ALLEN   7499 SALES             ALLEN   SALES
 WARD    7521 SALES             WARD    SALES
 MARTIN  7654 SALES             MARTIN  SALES
 BLAKE   7698 SALES             BLAKE   SALES
 TURNER  7844 SALES             TURNER  SALES
 JAMES   7900 SALES             JAMES   SALES
```

The strings are then returned to TRANSLATE once again, but this time it’s the second (outermost) TRANSLATE in the solution. TRANSLATE searches the original string for any characters that match the characters in TMP. If any are found, they too are converted to #s.

This conversion allows all nonnumeric characters to be treated as a single character (because they are all transformed to the same character):

```
select data, translate(data, 
              replace( 
              translate(data,'0123456789','##########'), 
              '#'), 
              rpad('#',length(data),'#')) as tmp 
   from V 

DATA                           TMP
------------------------------ ---------------------------
CLARK   7782 ACCOUNTING        ########7782###########
KING    7839 ACCOUNTING        ########7839###########
MILLER  7934 ACCOUNTING        ########7934###########
SMITH   7369 RESEARCH          ########7369#########
JONES   7566 RESEARCH          ########7566#########
SCOTT   7788 RESEARCH          ########7788#########
ADAMS   7876 RESEARCH          ########7876#########
FORD    7902 RESEARCH          ########7902#########
ALLEN   7499 SALES             ########7499######
WARD    7521 SALES             ########7521######
MARTIN  7654 SALES             ########7654######
BLAKE   7698 SALES             ########7698######
TURNER  7844 SALES             ########7844######
JAMES   7900 SALES             ########7900######
```

The next step is to remove all # characters through a call to REPLACE (line 8), leaving you with only numbers:

```
select data, replace( 
              translate(data, 
              replace( 
            translate(data,'0123456789','##########'), 
                      '#'), 
                      rpad('#',length(data),'#')),'#') as tmp 
   from V 

DATA                           TMP
------------------------------ -----------
CLARK   7782 ACCOUNTING        7782
KING    7839 ACCOUNTING        7839
MILLER  7934 ACCOUNTING        7934
SMITH   7369 RESEARCH          7369
JONES   7566 RESEARCH          7566
SCOTT   7788 RESEARCH          7788
ADAMS   7876 RESEARCH          7876
FORD    7902 RESEARCH          7902
ALLEN   7499 SALES             7499
WARD    7521 SALES             7521
MARTIN  7654 SALES             7654
BLAKE   7698 SALES             7698
TURNER  7844 SALES             7844
JAMES   7900 SALES             7900
```

Finally, cast TMP to a number (line 4) using the appropriate DBMS function (often CAST) to accomplish this:

```
select data, to_number( 
               replace( 
              translate(data, 
              replace( 
        translate(data,'0123456789','##########'), 
                  '#'), 
                  rpad('#',length(data),'#')),'#')) as tmp 
   from V 

DATA                                  TMP
------------------------------ ----------
CLARK   7782 ACCOUNTING              7782
KING    7839 ACCOUNTING              7839
MILLER  7934 ACCOUNTING              7934
SMITH   7369 RESEARCH                7369
JONES   7566 RESEARCH                7566
SCOTT   7788 RESEARCH                7788
ADAMS   7876 RESEARCH                7876
FORD    7902 RESEARCH                7902
ALLEN   7499 SALES                   7499
WARD    7521 SALES                   7521
MARTIN  7654 SALES                   7654
BLAKE   7698 SALES                   7698
TURNER  7844 SALES                   7844
JAMES   7900 SALES                   7900
```

When developing queries like this, it’s helpful to work with your expressions in the SELECT list. That way, you can easily view the intermediate results as you work toward a final solution. However, because the point of this recipe is to order the results, ultimately you should place all the function calls into the ORDER BY clause:

```
select data 
   from V 
  order by 
         to_number( 
           replace( 
         translate( data, 
           replace( 
         translate( data,'0123456789','##########'), 
                   '#'),rpad('#',length(data),'#')),'#')) 


DATA
---------------------------
SMITH   7369 RESEARCH
ALLEN   7499 SALES
WARD    7521 SALES
JONES   7566 RESEARCH
MARTIN  7654 SALES
BLAKE   7698 SALES
CLARK   7782 ACCOUNTING
SCOTT   7788 RESEARCH
KING    7839 ACCOUNTING
TURNER  7844 SALES
ADAMS   7876 RESEARCH
JAMES   7900 SALES
FORD    7902 RESEARCH
MILLER  7934 ACCOUNTING
```

As a final note, the data in the view is comprised of three fields, only one being numeric. Keep in mind that if there had been multiple numeric fields, they would have all been concatenated into one number before the rows were sorted.[]()

# 6.10 Creating a Delimited List from Table Rows

## Problem

[]()[]()You want to return table rows as values in a delimited list, perhaps delimited by commas, rather than in vertical columns as they normally appear. You want to convert a result set from this:

```
DEPTNO EMPS
------ ----------
    10 CLARK
    10 KING
    10 MILLER
    20 SMITH
    20 ADAMS
    20 FORD
    20 SCOTT
    20 JONES
    30 ALLEN
    30 BLAKE
    30 MARTIN
    30 JAMES
    30 TURNER
    30 WARD
```

to this:

```
 DEPTNO EMPS
------- ------------------------------------
     10 CLARK,KING,MILLER
     20 SMITH,JONES,SCOTT,ADAMS,FORD
     30 ALLEN,WARD,MARTIN,BLAKE,TURNER,JAMES
```

## Solution

Each DBMS requires a different approach to this problem. The key is to take advantage of the built-in functions provided by your DBMS. Understanding what is available to you will allow you to exploit your DBMS’s functionality and come up with creative solutions for a problem that is typically not solved in SQL.

Most DBMSs have now adopted a function specifically designed to concatenate strings, such as MySQL’s GROUP\_CONCAT function (one of the earliest) or STRING\_ADD (added to SQL Server as recently as SQL Server 2017). These functions have similar syntax, and make this task straightforward.

### DB2

[]()Use LIST\_AGG to build the delimited list:

```
1 select deptno,
2        list_agg(ename ',') within GROUP(Order by 0) as emps
3   from emp
4  group by deptno
```

### MySQL

[]()Use the built-in function GROUP\_CONCAT to build the delimited list:

```
1 select deptno,
2        group_concat(ename order by empno separator, ',') as emps
3   from emp
4  group by deptno
```

### Oracle

[]()Use the built-in function SYS\_CONNECT\_BY\_PATH to build the delimited list:

```
 1 select deptno,
 2        ltrim(sys_connect_by_path(ename,','),',') emps
 3   from (
 4 select deptno,
 5        ename,
 6        row_number() over
 7                 (partition by deptno order by empno) rn,
 8        count(*) over
 9                 (partition by deptno) cnt
10   from emp
11        )
12  where level = cnt
13  start with rn = 1
14 connect by prior deptno = deptno and prior rn = rn-1
```

### PostgreSQL and SQL Server

```
1 select deptno,
2        string_agg(ename order by empno separator, ',') as emps
3   from emp
4  group by deptno
```

## Discussion

Being able to create delimited lists in SQL is useful because it is a common requirement. The SQL:2016 standard added LIST\_AGG to perform this task, but only DB2 has implemented this function so far. Thankfully, other DBMS have similar functions, often with simpler syntax.

### MySQL

The function GROUP\_CONCAT in MySQL concatenates the values found in the column passed to it, in this case ENAME. It’s an aggregate function, thus the need for GROUP BY in the query.

### PostgreSQL and SQL Server

[]()The STRING\_AGG function syntax is similar enough to GROUP\_CONCAT that the same query can be used with the GROUP\_CONCAT simply changed to STRING\_AGG.

### Oracle

The first step to understanding the Oracle query is to break it down. Running the inline view by itself (lines 4–10), you generate a result set that includes the following for each employee: her department, her name, a rank within her respective department that is derived by an ascending sort on EMPNO, and a count of all employees in her department. For example:

```
select deptno, 
         ename, 
         row_number() over 
                   (partition by deptno order by empno) rn, 
         count(*) over (partition by deptno) cnt 
   from emp 

DEPTNO ENAME      RN CNT
------ ---------- -- ---
    10 CLARK       1   3
    10 KING        2   3
    10 MILLER      3   3
    20 SMITH       1   5
    20 JONES       2   5
    20 SCOTT       3   5
    20 ADAMS       4   5
    20 FORD        5   5
    30 ALLEN       1   6
    30 WARD        2   6
    30 MARTIN      3   6
    30 BLAKE       4   6
    30 TURNER      5   6
    30 JAMES       6   6
```

The purpose of the rank (aliased RN in the query) is to allow you to walk the tree. []()Since the function ROW\_NUMBER generates an enumeration starting from one with no duplicates or gaps, just subtract one (from the current value) to reference a prior (or parent) row. For example, the number prior to 3 is 3 minus 1, which equals 2. In this context, 2 is the parent of 3; you can observe this on line 12. Additionally, the lines:

```
start with rn = 1
connect by prior deptno = deptno
```

identify the root for each DEPTNO as having RN equal to 1 and create a new list whenever a new department is encountered (whenever a new occurrence of 1 is found for RN).

At this point, it’s important to stop and look at the ORDER BY portion of the ROW\_NUMBER function. Keep in mind the names are ranked by EMPNO, and the list will be created in that order. The number of employees per department is calculated (aliased CNT) and is used to ensure that the query returns only the list that has all the employee names for a department. This is done because SYS\_CONNECT_ BY\_PATH builds the list iteratively, and you do not want to end up with partial lists.

For hierarchical queries, the pseudocolumn LEVEL starts with 1 (for queries not using CONNECT BY, LEVEL is 0, unless you are on release 10g and later when LEVEL is available only when using CONNECT BY) and increments by one after each employee in a department has been evaluated (for each level of depth in the hierarchy). Because of this, you know that once LEVEL reaches CNT, you have reached the last EMPNO and will have a complete list.

###### Tip

[]()The SYS\_CONNECT\_BY\_PATH function prefixes the list with your chosen delimiter (in this case, a comma). You may or may not want that behavior. In this recipe’s solution, the call to the function LTRIM removes the leading comma from the list.[]()[]()

# 6.11 Converting Delimited Data into a Multivalued IN-List

## Problem

[]()[]()[]()You have delimited data that you want to pass to the IN-list iterator of a WHERE clause. Consider the following string:

```
7654,7698,7782,7788
```

You would like to use the string in a WHERE clause, but the following SQL fails because EMPNO is a numeric column:

```
select ename,sal,deptno
  from emp
 where empno in ( '7654,7698,7782,7788' )
```

This SQL fails because, while EMPNO is a numeric column, the IN list is composed of a single string value. You want that string to be treated as a comma-delimited list of numeric values.

## Solution

On the surface it may seem that SQL should do the work of treating a delimited string as a list of delimited values for you, but that is not the case. When a comma embedded within quotes is encountered, SQL can’t possibly know that signals a multivalued list. SQL must treat everything between the quotes as a single entity, as one string value. You must break the string up into individual EMPNOs. The key to this solution is to walk the string, but not into individual characters. You want to walk the string into valid EMPNO values.

### DB2

By walking the string passed to the IN-list, you can easily convert it to rows. The functions ROW\_NUMBER, LOCATE, and SUBSTR are particularly useful here:

```
 1 select empno,ename,sal,deptno
 2   from emp
 3  where empno in (
 4 select cast(substr(c,2,locate(',',c,2)-2) as integer) empno
 5   from (
 6 select substr(csv.emps,cast(iter.pos as integer)) as c
 7   from (select ','||'7654,7698,7782,7788'||',' emps
 8          from t1) csv,
 9        (select id as pos
10           from t100 ) iter
11  where iter.pos <= length(csv.emps)
12        ) x
13  where length(c) > 1
14    and substr(c,1,1) = ','
15        )
```

### MySQL

By walking the string passed to the IN-list, you can easily convert it to rows:

```
 1 select empno, ename, sal, deptno
 2   from emp
 3  where empno in
 4        (
 5 select substring_index(
 6        substring_index(list.vals,',',iter.pos),',',-1) empno
 7   from (select id pos from t10) as iter,
 8        (select '7654,7698,7782,7788' as vals
 9           from t1) list
10   where iter.pos <=
11        (length(list.vals)-length(replace(list.vals,',','')))+1
12        )
```

### Oracle

By walking the string passed to the IN-list, you can easily convert it to rows. The functions ROWNUM, SUBSTR, and INSTR are particularly useful here:

```
 1 select empno,ename,sal,deptno
 2   from emp
 3  where empno in (
 4        select to_number(
 5                   rtrim(
 6                 substr(emps,
 7                  instr(emps,',',1,iter.pos)+1,
 8                  instr(emps,',',1,iter.pos+1)
 9                  instr(emps,',',1,iter.pos)),',')) emps
10          from (select ','||'7654,7698,7782,7788'||',' emps from t1) csv,
11               (select rownum pos from emp) iter
12         where iter.pos <= ((length(csv.emps)-
13                   length(replace(csv.emps,',')))/length(','))-1
14 )
```

### PostgreSQL

By walking the string passed to the IN-list, you can easily convert it to rows. []()The function SPLIT\_PART makes it easy to parse the string into individual numbers:

```
 1 select ename,sal,deptno
 2   from emp
 3  where empno in (
 4 select cast(empno as integer) as empno
 5   from (
 6 select split_part(list.vals,',',iter.pos) as empno
 7   from (select id as pos from t10) iter,
 8        (select ','||'7654,7698,7782,7788'||',' as vals
 9           from t1) list
10  where iter.pos <=
11        length(list.vals)-length(replace(list.vals,',',''))
12        ) z
13  where length(empno) > 0
14        )
```

### SQL Server

By walking the string passed to the IN-list, you can easily convert it to rows. The functions ROW\_NUMBER, CHARINDEX, and SUBSTRING are particularly useful here:

```
 1 select empno,ename,sal,deptno
 2   from emp
 3  where empno in (select substring(c,2,charindex(',',c,2)-2) as empno
 4   from (
 5 select substring(csv.emps,iter.pos,len(csv.emps)) as c
 6   from (select ','+'7654,7698,7782,7788'+',' as emps
 7           from t1) csv,
 8        (select id as pos
 9          from t100) iter
10  where iter.pos <= len(csv.emps)
11       ) x
12  where len(c) > 1
13    and substring(c,1,1) = ','
14       )
```

## Discussion

The first and most important step in this solution is to walk the string. Once you’ve accomplished that, all that’s left is to parse the string into individual numeric values using your DBMS’s provided functions.

### DB2 and SQL Server

The inline view X (lines 6–11) walks the string. The idea in this solution is to “walk through” the string so that each row has one less character than the one before it:

```
,7654,7698,7782,7788,
7654,7698,7782,7788,
654,7698,7782,7788,
54,7698,7782,7788,
4,7698,7782,7788,
,7698,7782,7788,
7698,7782,7788,
698,7782,7788,
98,7782,7788,
8,7782,7788,
,7782,7788,
7782,7788,
782,7788,
82,7788,
2,7788,
,7788,
7788,
788,
88,
8,
,
```

Notice that by enclosing the string in commas (the delimiter), there’s no need to make special checks as to where the beginning or end of the string is.

The next step is to keep only the values you want to use in the IN-list. The values to keep are the ones with leading commas, with the exception of the last row with its lone comma. Use SUBSTR or SUBSTRING to identify which rows have a leading comma, then keep all characters found before the next comma in that row. Once that’s done, cast the string to a number so it can be properly evaluated against the numeric column EMPNO (lines 4–14):

```
 EMPNO
------
  7654
  7698
  7782
  7788
```

The final step is to use the results in a subquery to return the desired rows.

### MySQL

The inline view (lines 5–9) walks the string. The expression on line 10 determines how many values are in the string by finding the number of commas (the delimiter) and adding one. []()The function SUBSTRING\_INDEX (line 6) returns all characters in the string before (to the left of ) the *n*th occurrence of a comma (the delimiter):

```
+---------------------+
| empno               |
+---------------------+
| 7654                |
| 7654,7698           |
| 7654,7698,7782      |
| 7654,7698,7782,7788 |
+---------------------+
```

Those rows are then passed to another call to SUBSTRING\_INDEX (line 5); this time the *n*th occurrence of the delimited is –1, which causes all values to the right of the *n*th occurrence of the delimiter to be kept:

```
+-------+
| empno |
+-------+
| 7654  |
| 7698  |
| 7782  |
| 7788  |
+-------+
```

The final step is to plug the results into a subquery.

### Oracle

The first step is to walk the string:

```
select emps,pos 
   from (select ','||'7654,7698,7782,7788'||',' emps 
           from t1) csv, 
        (select rownum pos from emp) iter 
  where iter.pos <= 
  ((length(csv.emps)-length(replace(csv.emps,',')))/length(','))-1 


EMPS                         POS
--------------------- ----------
,7654,7698,7782,7788,          1
,7654,7698,7782,7788,          2
,7654,7698,7782,7788,          3
,7654,7698,7782,7788,          4
```

The number of rows returned represents the number of values in your list. The values for POS are crucial to the query as they are needed to parse the string into individual values. []()[]()The strings are parsed using SUBSTR and INSTR. POS is used to locate the *n*th occurrence of the delimiter in each string. By enclosing the strings in commas, no special checks are necessary to determine the beginning or end of a string. The values passed to SUBSTR and INSTR (lines 7–9) locate the *n*th and *n*th+1 occurrence of the delimiter. By subtracting the value returned for the current comma (the location in the string where the current comma is) from the value returned by the next comma (the location in the string where the next comma is) you can extract each value from the string:

```
select substr(emps, 
        instr(emps,',',1,iter.pos)+1, 
        instr(emps,',',1,iter.pos+1) 
        instr(emps,',',1,iter.pos)) emps 
   from (select ','||'7654,7698,7782,7788'||',' emps 
           from t1) csv, 
        (select rownum pos from emp) iter 
  where iter.pos <= 
   ((length(csv.emps)-length(replace(csv.emps,',')))/length(','))-1 

 EMPS
 -----------
 7654,
 7698,
 7782,
 7788,
```

The final step is to remove the trailing comma from each value, cast it to a number, and plug it into a subquery.

### PostgreSQL

The inline view Z (lines 6–9) walks the string. The number of rows returned is determined by how many values are in the string. To find the number of values in the string, subtract the size of the string without the delimiter from the size of the string with the delimiter (line 9). []()The function SPLIT\_PART does the work of parsing the string. It looks for the value that comes before the *n*th occurrence of the delimiter:

```
select list.vals, 
        split_part(list.vals,',',iter.pos) as empno, 
        iter.pos 
   from (select id as pos from t10) iter, 
        (select ','||'7654,7698,7782,7788'||',' as vals 
           from t1) list 
  where iter.pos <= 
        length(list.vals)-length(replace(list.vals,',','')) 

     vals             | empno | pos
----------------------+-------+-----
,7654,7698,7782,7788, |       |   1
,7654,7698,7782,7788, | 7654  |   2
,7654,7698,7782,7788, | 7698  |   3
,7654,7698,7782,7788, | 7782  |   4
,7654,7698,7782,7788, | 7788  |   5
```

The final step is to cast the values (EMPNO) to a number and plug it into a subquery.[]()[]()[]()

# 6.12 Alphabetizing a String

## Problem

[]()[]()You want alphabetize the individual characters within strings in your tables. Consider the following result set:

```
ENAME
----------
ADAMS
ALLEN
BLAKE
CLARK
FORD
JAMES
JONES
KING
MARTIN
MILLER
SCOTT
SMITH
TURNER
WARD
```

You would like the result to be:

```
OLD_NAME   NEW_NAME
---------- --------
ADAMS      AADMS
ALLEN      AELLN
BLAKE      ABEKL
CLARK      ACKLR
FORD       DFOR
JAMES      AEJMS
JONES      EJNOS
KING       GIKN
MARTIN     AIMNRT
MILLER     EILLMR
SCOTT      COSTT
SMITH      HIMST
TURNER     ENRRTU
WARD       ADRW
```

## Solution

This problem is a good example of the way increased standardization allows for more similar, and therefore portable solutions.

### DB2

To alphabetize rows of strings, it is necessary to walk each string and then order its characters:

```
 1 select ename,
 2        listagg(c,'')  WITHIN GROUP( ORDER BY c)
 3   from (
 4         select a.ename,
 5         substr(a.ename,iter.pos,1
 6         ) as c
 7  from emp a,
 8      (select id as pos from t10) iter
 9        where iter.pos <= length(a.ename)
 10        order by 1,2
 11      ) x
 12       Group By c
```

### MySQL

[]()The key here is the GROUP\_CONCAT function, which allows you to not only concatenate the characters that make up each name but also order them:

```
1 select ename, group_concat(c order by c separator '')
2   from (
3 select ename, substr(a.ename,iter.pos,1) c
4   from emp a,
5        ( select id pos from t10 ) iter
6  where iter.pos <= length(a.ename)
7        ) x
8  group by ename
```

### Oracle

[]()The function SYS\_CONNECT\_BY\_PATH allows you to iteratively build a list:

```
 1 select old_name, new_name
 2   from (
 3 select old_name, replace(sys_connect_by_path(c,' '),' ') new_name
 4   from (
 5 select e.ename old_name,
 6        row_number() over(partition by e.ename
 7                         order by substr(e.ename,iter.pos,1)) rn,
 8        substr(e.ename,iter.pos,1) c
 9   from emp e,
10        ( select rownum pos from emp ) iter
11  where iter.pos <= length(e.ename)
12  order by 1
13         ) x
14  start with rn = 1
15 connect by prior rn = rn-1 and prior old_name = old_name
16         )
17  where length(old_name) = length(new_name)
```

### PostgreSQL

[]()PostgreSQL has now added STRING\_AGG to order characters within a string.

```
	 select ename, string_agg(c , ''
				  ORDER BY c)
from (
	 select a.ename,
	        substr(a.ename,iter.pos,1) as c
	   from emp a,
	        (select id as pos from t10) iter
	  where iter.pos <= length(a.ename)
	  order by 1,2
	        ) x
	        Group By c
```

### SQL Server

If you are using SQL Server 2017 or beyond, the PostgreSQL solution with STRING\_AGG will work. Otherwise, to alphabetize rows of strings, it is necessary to walk each string and then order their characters:

```
 1 select ename,
 2           max(case when pos=1 then c else '' end)+
 3           max(case when pos=2 then c else '' end)+
 4           max(case when pos=3 then c else '' end)+
 5           max(case when pos=4 then c else '' end)+
 6           max(case when pos=5 then c else '' end)+
 7           max(case when pos=6 then c else '' end)
 8      from (
 9    select e.ename,
10         substring(e.ename,iter.pos,1) as c,
11         row_number() over (
12          partition by e.ename
13              order by substring(e.ename,iter.pos,1)) as pos
14    from emp e,
15         (select row_number()over(order by ename) as pos
16            from emp) iter
17   where iter.pos <= len(e.ename)
18          ) x
19   group by ename
```

## Discussion

### SQL Server

[]()[]()The inline view X returns each character in each name as a row. []()The function SUBSTR or SUBSTRING extracts each character from each name, and the function ROW\_NUMBER ranks each character alphabetically:

```
ENAME  C  POS
-----  -  ---
ADAMS  A  1
ADAMS  A  2
ADAMS  D  3
ADAMS  M  4
ADAMS  S  5
…
```

To return each letter of a string as a row, you must walk the string. This is accomplished with inline view ITER.

Now that the letters in each name have been alphabetized, the last step is to put those letters back together, into a string, in the order they are ranked. Each letter’s position is evaluated by the CASE statements (lines 2–7). If a character is found at a particular position, it is then concatenated to the result of the next evaluation (the following CASE statement). Because the aggregate function MAX is used as well, only one character per position POS is returned so that only one row per name is returned. The CASE evaluation goes up to the number six, which is the maximum number of characters in any name in table EMP.

### MySQL

The inline view X (lines 3–6) returns each character in each name as a row. The function SUBSTR extracts each character from each name:

```
ENAME  C
-----  -
ADAMS  A
ADAMS  A
ADAMS  D
ADAMS  M
ADAMS  S
…
```

Inline view ITER is used to walk the string. []()From there, the rest of the work is done by the GROUP\_CONCAT function. By specifying an order, the function not only concatenates each letter, it does so alphabetically.

### Oracle

The real work is done by inline view X (lines 5–11), where the characters in each name are extracted and put into alphabetical order. This is accomplished by walking the string and then imposing order on those characters. The rest of the query merely glues the names back together.

The tearing apart of names can be seen by executing only inline view X:

```
OLD_NAME          RN C
---------- --------- -
ADAMS              1 A
ADAMS              2 A
ADAMS              3 D
ADAMS              4 M
ADAMS              5 S
…
```

The next step is to take the alphabetized characters and rebuild each name. []()This is done with the function SYS\_CONNECT\_BY\_PATH by appending each character to the ones before it:

```
OLD_NAME   NEW_NAME
---------- ---------
ADAMS      A
ADAMS      AA
ADAMS      AAD
ADAMS      AADM
ADAMS      AADMS
…
```

The final step is to keep only the strings that have the same length as the names they were built from.

### PostgreSQL

Similar to the MySQL solution, the inline view—from line 4 onwards—is used to convert each name into a row of characters. The STRING\_AGG function then concatenates the characters in alphabetical order.[]()[]()

# 6.13 Identifying Strings That Can Be Treated as Numbers

## Problem

[]()You have a column that is defined to hold character data. Unfortunately, the rows contain mixed numeric and character data. Consider view V:

```
create view V as
select replace(mixed,' ','') as mixed
  from (
select substr(ename,1,2)||
       cast(deptno as char(4))||
       substr(ename,3,2) as mixed
  from emp
 where deptno = 10
 union all
select cast(empno as char(4)) as mixed
  from emp
 where deptno = 20
 union all
select ename as mixed
  from emp
 where deptno = 30
       ) x
select * from v

 MIXED
 --------------
 CL10AR
 KI10NG
 MI10LL
 7369
 7566
 7788
 7876
 7902
 ALLEN
 WARD
 MARTIN
 BLAKE
 TURNER
 JAMES
```

You want to return rows that are numbers only, or that contain at least one number. If the numbers are mixed with character data, you want to remove the characters and return only the numbers. For the sample data shown previously, you want the following result set:

```
   MIXED
--------
      10
      10
      10
    7369
    7566
    7788
    7876
    7902
```

## Solution

The functions REPLACE and TRANSLATE are extremely useful for manipulating strings and individual characters. The key is to convert all numbers to a single character, which then makes it easy to isolate and identify any number by referring to a single character.

### DB2

Use functions TRANSLATE, REPLACE, and POSSTR to isolate the numeric characters in each row. The calls to CAST are necessary in view V; otherwise, the view will fail to be created due to type conversion errors. You’ll need the function REPLACE to remove extraneous whitespace due to casting to the fixed-length CHAR:

```
 1 select mixed old,
 2        cast(
 3          case
 4          when
 5            replace(
 6           translate(mixed,'9999999999','0123456789'),'9','') = ''
 7          then
 8             mixed
 9          else replace(
10            translate(mixed,
11               repeat('#',length(mixed)),
12             replace(
13              translate(mixed,'9999999999','0123456789'),'9','')),
14                      '#','')
15          end as integer ) mixed
16   from V
17  where posstr(translate(mixed,'9999999999','0123456789'),'9') > 0
```

### MySQL

The syntax for MySQL is slightly different and will define view V as:

```
create view V as
select concat(
         substr(ename,1,2),
         replace(cast(deptno as char(4)),' ',''),
         substr(ename,3,2)
       ) as mixed
  from emp
 where deptno = 10
 union all
select replace(cast(empno as char(4)), ' ', '')
  from emp where deptno = 20
 union all
select ename from emp where deptno = 30
```

Because MySQL does not support the TRANSLATE function, you must walk each row and evaluate it on a character-by-character basis.

```
 1 select cast(group_concat(c order by pos separator '') as unsigned)
 2        as MIXED1
 3   from (
 4 select v.mixed, iter.pos, substr(v.mixed,iter.pos,1) as c
 5   from V,
 6        ( select id pos from t10 ) iter
 7  where iter.pos <= length(v.mixed)
 8    and ascii(substr(v.mixed,iter.pos,1)) between 48 and 57
 9        ) y
10  group by mixed
11  order by 1
```

### Oracle

Use functions TRANSLATE, REPLACE, and INSTR to isolate the numeric characters in each row. The calls to CAST are not necessary in view V. Use the function REPLACE to remove extraneous whitespace due to casting to the fixed-length CHAR. If you decide you would like to keep the explicit type conversion calls in the view definition, it is suggested you cast to VARCHAR2:

```
 1 select to_number (
 2         case
 3         when
 4            replace(translate(mixed,'0123456789','9999999999'),'9')
 5           is not null
 6         then
 7              replace(
 8            translate(mixed,
 9              replace(
10            translate(mixed,'0123456789','9999999999'),'9'),
11                     rpad('#',length(mixed),'#')),'#')
12         else
13              mixed
14         end
15         ) mixed
16   from V
17  where instr(translate(mixed,'0123456789','9999999999'),'9') > 0
```

### PostgreSQL

Use functions TRANSLATE, REPLACE, and STRPOS to isolate the numeric characters in each row. The calls to CAST are not necessary in view V. Use the function REPLACE to remove extraneous whitespace due to casting to the fixed-length CHAR. If you decide you would like to keep the explicit type conversion calls in the view definition, it is suggested you cast to VARCHAR:

```
 1 select cast(
 2        case
 3        when
 4         replace(translate(mixed,'0123456789','9999999999'),'9','')
 5         is not null
 6        then
 7           replace(
 8        translate(mixed,
 9           replace(
10        translate(mixed,'0123456789','9999999999'),'9',''),
11                 rpad('#',length(mixed),'#')),'#','')
12        else
13          mixed
14        end as integer ) as mixed
15    from V
16  where strpos(translate(mixed,'0123456789','9999999999'),'9') > 0
```

### SQL Server

[]()The built-in function ISNUMERIC along with a wildcard search allows you to easily identify strings that contain numbers, but getting numeric characters out of a string is not particularly efficient because the TRANSLATE function is not supported.

## Discussion

The TRANSLATE function is useful here as it allows you to easily isolate and identify numbers and characters. The trick is to convert all numbers to a single character; this way, rather than searching for different numbers, you search for only one character.

### DB2, Oracle, and PostgreSQL

The syntax differs slightly among these DBMSs, but the technique is the same. We’ll use the solution for PostgreSQL for the discussion.

The real work is done by functions TRANSLATE and REPLACE. Getting the final result set requires several function calls, each listed here in one query:

```
select mixed as orig, 
 translate(mixed,'0123456789','9999999999') as mixed1, 
 replace(translate(mixed,'0123456789','9999999999'),'9','') as mixed2, 
  translate(mixed, 
  replace( 
  translate(mixed,'0123456789','9999999999'),'9',''), 
           rpad('#',length(mixed),'#')) as mixed3,  
  replace( 
  translate(mixed, 
  replace(  
 translate(mixed,'0123456789','9999999999'),'9',''), 
           rpad('#',length(mixed),'#')),'#','') as mixed4 
   from V  
  where strpos(translate(mixed,'0123456789','9999999999'),'9') > 0 

  ORIG  | MIXED1 | MIXED2 | MIXED3 | MIXED4 | MIXED5
--------+--------+--------+--------+--------+--------
 CL10AR | CL99AR | CLAR   | ##10## | 10     |     10
 KI10NG | KI99NG | KING   | ##10## | 10     |     10
 MI10LL | MI99LL | MILL   | ##10## | 10     |     10
 7369   | 9999   |        | 7369   | 7369   |   7369
 7566   | 9999   |        | 7566   | 7566   |   7566
 7788   | 9999   |        | 7788   | 7788   |   7788
 7876   | 9999   |        | 7876   | 7876   |   7876
 7902   | 9999   |        | 7902   | 7902   |   7902
```

First, notice that any rows without at least one number are removed. How this is accomplished will become clear as you examine each of the columns in the previous result set. The rows that are kept are the values in the ORIG column and are the rows that will eventually make up the result set. The first step to extracting the numbers is to use the function TRANSLATE to convert any number to a 9 (you can use any digit; 9 is arbitrary); this is represented by the values in MIXED1. Now that all numbers are 9s, they can be treated as a single unit. The next step is to remove all of the numbers by using the function REPLACE. Because all digits are now 9, REPLACE simply looks for any 9s and removes them. This is represented by the values in MIXED2. The next step, MIXED3, uses values that are returned by MIXED2. These values are then compared to the values in ORIG. If any characters from MIXED2 are found in ORIG, they are converted to the # character by TRANSLATE. The result set from MIXED3 shows that the letters, not the numbers, have now been singled out and converted to a single character. Now that all nonnumeric characters are represented by #s, they can be treated as a single unit. The next step, MIXED4, uses REPLACE to find and remove any # characters in each row; what’s left are numbers only. The final step is to cast the numeric characters as numbers. Now that you’ve gone through the steps, you can see how the WHERE clause works. The results from MIXED1 are passed to STRPOS, and if a 9 is found (the position in the string where the first 9 is located), the result must be greater than 0. For rows that return a value greater than zero, it means there’s at least one number in that row and it should be kept.

### MySQL

The first step is to walk each string, evaluate each character, and determine whether it’s a number:

```
select v.mixed, iter.pos, substr(v.mixed,iter.pos,1) as c 
   from V, 
       ( select id pos from t10 ) iter 
  where iter.pos <= length(v.mixed)  
  order by 1,2 

+--------+------+------+
| mixed  | pos  | c    |
+--------+------+------+
| 7369   |    1 | 7    |
| 7369   |    2 | 3    |
| 7369   |    3 | 6    |
| 7369   |    4 | 9    |
…
| ALLEN  |    1 | A    |
| ALLEN  |    2 | L    |
| ALLEN  |    3 | L    |
| ALLEN  |    4 | E    |
| ALLEN  |    5 | N    |
…
| CL10AR |    1 | C    |
| CL10AR |    2 | L    |
| CL10AR |    3 | 1    |
| CL10AR |    4 | 0    |
| CL10AR |    5 | A    |
| CL10AR |    6 | R    |
+--------+------+------+
```

Now that each character in each string can be evaluated individually, the next step is to keep only the rows that have a number in the C column:

```
select v.mixed, iter.pos, substr(v.mixed,iter.pos,1) as c 
    from V, 
        ( select id pos from t10 ) iter 
  where iter.pos <= length(v.mixed) 
   and ascii(substr(v.mixed,iter.pos,1)) between 48 and 57 
  order by 1,2 

+--------+------+------+
| mixed  | pos  | c    |
+--------+------+------+
| 7369   |    1 | 7    |
| 7369   |    2 | 3    |
| 7369   |    3 | 6    |
| 7369   |    4 | 9    |
…
| CL10AR |    3 | 1    |
| CL10AR |    4 | 0    |
…
+--------+------+------+
```

At this point, all the rows in column C are numbers. The next step is to use GROUP\_CONCAT to concatenate the numbers to form their respective whole number in MIXED. The final result is then cast as a number:

```
select cast(group_concat(c order by pos separator '') as unsigned) 
          as MIXED1 
   from (  
 select v.mixed, iter.pos, substr(v.mixed,iter.pos,1) as c  
   from V,  
       ( select id pos from t10 ) iter  
  where iter.pos <= length(v.mixed)  
   and ascii(substr(x.mixed,iter.pos,1)) between 48 and 57 
        ) y 
   group by mixed 
   order by 1 

+--------+
| MIXED1 |
+--------+
|    10  |
|    10  |
|    10  |
|  7369  |
|  7566  |
|  7788  |
|  7876  |
|  7902  |
+--------+
```

As a final note, keep in mind that any digits in each string will be concatenated to form one numeric value. For example, an input value of, say, `99Gennick87` will result in the value 9987 being returned. This is something to keep in mind, particularly when working with serialized data.[]()

# 6.14 Extracting the nth Delimited Substring

## Problem

[]()[]()You want to extract a specified, delimited substring from a string. Consider the following view V, which generates source data for this problem:

```
create view V as
select 'mo,larry,curly' as name
  from t1
 union all
select 'tina,gina,jaunita,regina,leena' as name
  from t1
```

Output from the view is as follows:

```
select * from v 

NAME
-------------------
mo,larry,curly
tina,gina,jaunita,regina,leena
```

You would like to extract the second name in each row, so the final result set would be as follows:

```
  SUB
-----
larry
gina
```

## Solution

The key to solving this problem is to return each name as an individual row while preserving the order in which the name exists in the list. Exactly how you do these things depends on which DBMS you are using.

### DB2

After walking the NAMEs returned by view V, use the function ROW\_NUMBER to keep only the second name from each string:

```
 1 select substr(c,2,locate(',',c,2)-2)
 2   from (
 3 select pos, name, substr(name, pos) c,
 4        row_number() over( partition by name
 5                       order by length(substr(name,pos)) desc) rn
 6  from (
 7 select ',' ||csv.name|| ',' as name,
 8         cast(iter.pos as integer) as pos
 9   from V csv,
10        (select row_number() over() pos from t100 ) iter
11  where iter.pos <= length(csv.name)+2
12        ) x
13  where length(substr(name,pos)) > 1
14   and substr(substr(name,pos),1,1) = ','
15        ) y
16  where rn = 2
```

### MySQL

After walking the NAMEs returned by view V, use the position of the commas to return only the second name in each string:

```
 1 select name
 2   from (
 3 select iter.pos,
 4        substring_index(
 5        substring_index(src.name,',',iter.pos),',',-1) name
 6   from V src,
 7        (select id pos from t10) iter,
 8  where iter.pos <=
 9        length(src.name)-length(replace(src.name,',',''))
10        ) x
11  where pos = 2
```

### Oracle

[]()[]()After walking the NAMEs returned by view V, retrieve the second name in each list by using SUBSTR and INSTR:

```
	 1 select sub
	 2   from (
	 3 select iter.pos,
	 4        src.name,
	 5        substr( src.name,
	 6         instr( src.name,',',1,iter.pos )+1,
	 7         instr( src.name,',',1,iter.pos+1 ) -
	 8         instr( src.name,',',1,iter.pos )-1) sub
	 9   from (select ','||name||',' as name from V) src,
	10        (select rownum pos from emp) iter
	11  where iter.pos < length(src.name)-length(replace(src.name,','))
	12        )
	13  where pos = 2
```

### PostgreSQL

[]()Use the function SPLIT\_PART to help return each individual name as a row:

```
 1 select name
 2   from (
 3 select iter.pos, split_part(src.name,',',iter.pos) as name
 4   from (select id as pos from t10) iter,
 5        (select cast(name as text) as name from v) src
 7  where iter.pos <=
 8         length(src.name)-length(replace(src.name,',',''))+1
 9        ) x
10  where pos = 2
```

### SQL Server

[]()[]()The SQL Server STRING\_SPLIT function will do the whole job, but can only take a single cell. Hence, we use a STRING\_AGG within a CTE to present the data the way STRING\_SPLIT requires.

```
1 with agg_tab(name)
2     as
3     (select STRING_AGG(name,',') from V)
4 select value from
5     STRING_SPLIT(
6     (select name from agg_tab),',')
```

## Discussion

### DB2

For DB2, the strings are walked and the results are represented by inline view X:

```
select ','||csv.name|| ',' as name, 
         iter.pos 
   from v csv, 
         (select row_number() over() pos from t100 ) iter 
  where iter.pos <= length(csv.name)+2 

EMPS                             POS
------------------------------- ----
,tina,gina,jaunita,regina,leena,    1
,tina,gina,jaunita,regina,leena,    2
,tina,gina,jaunita,regina,leena,    3
…
```

The next step is to then step through each character in each string:

```
select pos, name, substr(name, pos) c, 
         row_number() over(partition by name 
                         order by length(substr(name, pos)) desc) rn 
   from ( 
 select ','||csv.name||',' as name, 
        cast(iter.pos as integer) as pos 
   from v csv, 
       (select row_number() over() pos from t100 ) iter 
  where iter.pos <= length(csv.name)+2 
        ) x 
  where length(substr(name,pos)) > 1 

POS EMPS            C                RN
--- --------------- ---------------- --
  1 ,mo,larry,curly, ,mo,larry,curly, 1
  2 ,mo,larry,curly, mo,larry,curly,  2
  3 ,mo,larry,curly, o,larry,curly,   3
  4 ,mo,larry,curly, ,larry,curly,    4
  …
```

Now that different portions of the string are available to you, simply identify which rows to keep. The rows you are interested in are the ones that begin with a comma; the rest can be discarded:

```
select pos, name, substr(name,pos) c, 
        row_number() over(partition by name 
                        order by length(substr(name, pos)) desc) rn 
   from ( 
 select ','||csv.name||',' as name, 
        cast(iter.pos as integer) as pos 
   from v csv, 
        (select row_number() over() pos from t100 ) iter  
  where iter.pos <= length(csv.name)+2 
        ) x  
  where length(substr(name,pos)) > 1  
     and substr(substr(name,pos),1,1) = ',' 


POS  EMPS                               C                                    RN
 ---  --------------                     ----------------                      --
   1  ,mo,larry,curly,                   ,mo,larry,curly,                       1
   4  ,mo,larry,curly,                   ,larry,curly,                          2
  10  ,mo,larry,curly,                   ,curly,                                3
   1  ,tina,gina,jaunita,regina,leena,   ,tina,gina,jaunita,regina,leena,       1
   6  ,tina,gina,jaunita,regina,leena,   ,gina,jaunita,regina,leena,            2
  11  ,tina,gina,jaunita,regina,leena,   ,jaunita,regina,leena,                 3
  19  ,tina,gina,jaunita,regina,leena,   ,regina,leena,                         4
  26  ,tina,gina,jaunita,regina,leena,   ,leena,                                5
```

This is an important step as it sets up how you will get the *n*th substring. Notice that many rows have been eliminated from this query because of the following condition in the WHERE clause:

```
substr(substr(name,pos),1,1) = ','
```

You’ll notice that `,mo,larry,curly,` was ranked 4, but now is ranked 2. Remember, the WHERE clause is evaluated before the SELECT, so the rows with leading commas are kept, *then* ROW\_NUMBER performs its ranking. At this point it’s easy to see that, to get the *n*th substring, you want rows where RN equals *n*. The last step is to keep only the rows you are interested in (in this case where RN equals two) and use SUBSTR to extract the name from that row. The name to keep is the first name in the row: `larry` from `,larry,curly,` and `gina` from `,gina,jaunita,regina,leena,`.

### MySQL

The inline view X walks each string. You can determine how many values are in each string by counting the delimiters in the string:

```
select iter.pos, src.name 
   from (select id pos from t10) iter, 
        V src 
  where iter.pos <= 
        length(src.name)-length(replace(src.name,',','')) 

+------+--------------------------------+
| pos  | name                           |
+------+--------------------------------+
|    1 | mo,larry,curly                 |
|    2 | mo,larry,curly                 |
|    1 | tina,gina,jaunita,regina,leena |
|    2 | tina,gina,jaunita,regina,leena |
|    3 | tina,gina,jaunita,regina,leena |
|    4 | tina,gina,jaunita,regina,leena |
+------+--------------------------------+
```

In this case, there is one fewer row than values in each string because that’s all that is needed. The function SUBSTRING\_INDEX takes care of parsing the needed values:

```
	select iter.pos,src.name name1, 
 	        substring_index(src.name,',',iter.pos) name2, 
 	        substring_index( 
 	         substring_index(src.name,',',iter.pos),',',-1) name3 
 	   from (select id pos from t10) iter, 
 	        V src 
 	  where  iter.pos <= 
 	        length(src.name)-length(replace(src.name,',','')) 

+------+--------------------------------+--------------------------+---------+
| pos  | name1                        | name2                      | name3   |
+------+--------------------------------+--------------------------+---------+
|    1 | mo,larry,curly                 | mo                       | mo      |
|    2 | mo,larry,curly                 | mo,larry                 | larry   |
|    1 | tina,gina,jaunita,regina,leena | tina                     | tina    |
|    2 | tina,gina,jaunita,regina,leena | tina,gina                | gina    |
|    3 | tina,gina,jaunita,regina,leena | tina,gina,jaunita        | jaunita |
|    4 | tina,gina,jaunita,regina,leena | tina,gina,jaunita,regina | regina  |
+------+--------------------------------+--------------------------+---------+
```

We’ve shown three name fields, so you can see how the nested SUBSTRING\_INDEX calls work. The inner call returns all characters to the left of the *n*th occurrence of a comma. The outer call returns everything to the right of the first comma it finds (starting from the end of the string). The final step is to keep the value for NAME3 where POS equals *n*, in this case 2.

### SQL Server

[]()STRING\_SPLIT is the workhorse here, but needs its data the right way. The CTE is merely to turn the two rows of the V.names column into a single value, as required by STRING\_SPLIT being a table-valued function.

### Oracle

The inline view walks each string. The number of times each string is returned is determined by how many values are in each string. The solution finds the number of values in each string by counting the number of delimiters in it. Because each string is enclosed in commas, the number of values in a string is the number of commas minus one. The strings are then UNIONed and joined to a table with a cardinality that is at least the number of values in the largest string. []()[]()The functions SUBSTR and INSTR use the value of POS to parse each string:

```
select iter.pos, src.name, 
        substr( src.name, 
         instr( src.name,',',1,iter.pos )+1, 
         instr( src.name,',',1,iter.pos+1 ) 
         instr( src.name,',',1,iter.pos )-1) sub 
   from (select ','||name||',' as name from v) src, 
        (select rownum pos from emp) iter 
  where iter.pos < length(src.name)-length(replace(src.name,',')) 


POS NAME                              SUB
--- --------------------------------- -------------
  1 ,mo,larry,curly,                  mo
  1 , tina,gina,jaunita,regina,leena, tina
  2 ,mo,larry,curly,                  larry
  2 , tina,gina,jaunita,regina,leena, gina
  3 ,mo,larry,curly,                  curly
  3 , tina,gina,jaunita,regina,leena, jaunita
  4 , tina,gina,jaunita,regina,leena, regina
  5 , tina,gina,jaunita,regina,leena, leena
```

The first call to INSTR within SUBSTR determines the start position of the substring to extract. The next call to INSTR within SUBSTR finds the position of the *n*th comma (same as the start position) as well the position of the *n*th + 1 comma. Subtracting the two values returns the length of the substring to extract. Because every value is parsed into its own row, simply specify WHERE POS = *n* to keep the *n*th substring (in this case, where POS = 2, so the second substring in the list).

### PostgreSQL

The inline view X walks each string. The number of rows returned is determined by how many values are in each string. To find the number of values in each string, find the number of delimiters in each string and add one. The function SPLIT\_PART uses the values in POS to find the *n*th occurrence of the delimiter and parse the string into values:

```
select iter.pos, src.name as name1, 
         split_part(src.name,',',iter.pos) as name2 
    from (select id as pos from t10) iter, 
         (select cast(name as text) as name from v) src 
   where iter.pos <= 
         length(src.name)-length(replace(src.name,',',''))+1 

 pos |             name1              |  name2
-----+--------------------------------+---------
   1 | mo,larry,curly                 | mo
   2 | mo,larry,curly                 | larry
   3 | mo,larry,curly                 | curly
   1 | tina,gina,jaunita,regina,leena | tina
   2 | tina,gina,jaunita,regina,leena | gina
   3 | tina,gina,jaunita,regina,leena | jaunita
   4 | tina,gina,jaunita,regina,leena | regina
   5 | tina,gina,jaunita,regina,leena | leena
```

We’ve shown NAME twice so you can see how SPLIT\_PART parses each string using POS. Once each string is parsed, the final step is to keep the rows where POS equals the *n*th substring you are interested in, in this case, 2.[]()[]()

# 6.15 Parsing an IP Address

## Problem

[]()You want to parse an IP address’s fields into columns. Consider the following IP address:

```
111.22.3.4
```

You would like the result of your query to be:

```
A     B    C      D
----- ----- ----- ---
111   22    3     4
```

## Solution

The solution depends on the built-in functions provided by your DBMS. Regardless of your DBMS, being able to locate periods and the numbers immediately surrounding them are the keys to the solution.

### DB2

Use the recursive WITH clause to simulate an iteration through the IP address while using SUBSTR to easily parse it. A leading period is added to the IP address so that every set of numbers has a period in front of it and can be treated the same way.

```
 1 with x (pos,ip) as (
 2   values (1,'.92.111.0.222')
 3   union all
 4  select pos+1,ip from x where pos+1 <= 20
 5 )
 6 select max(case when rn=1 then e end) a,
 7        max(case when rn=2 then e end) b,
 8        max(case when rn=3 then e end) c,
 9        max(case when rn=4 then e end) d
10   from (
11 select pos,c,d,
12        case when posstr(d,'.') > 0 then substr(d,1,posstr(d,'.')-1)
13             else d
14        end as e,
15        row_number() over( order by pos desc) rn
16   from (
17 select pos, ip,right(ip,pos) as c, substr(right(ip,pos),2) as d
18   from x
19  where pos <= length(ip)
20   and substr(right(ip,pos),1,1) = '.'
21      ) x
22      ) y
```

### MySQL

[]()The function SUBSTR\_INDEX makes parsing an IP address an easy operation:

```
1 select substring_index(substring_index(y.ip,'.',1),'.',-1) a,
2        substring_index(substring_index(y.ip,'.',2),'.',-1) b,
3        substring_index(substring_index(y.ip,'.',3),'.',-1) c,
4        substring_index(substring_index(y.ip,'.',4),'.',-1) d
5   from (select '92.111.0.2' as ip from t1) y
```

### Oracle

Use the built-in function SUBSTR and INSTR to parse and navigate through the IP address:

```
1 select ip,
2        substr(ip, 1, instr(ip,'.')-1 ) a,
3        substr(ip, instr(ip,'.')+1,
4                    instr(ip,'.',1,2)-instr(ip,'.')-1 ) b,
5        substr(ip, instr(ip,'.',1,2)+1,
6                    instr(ip,'.',1,3)-instr(ip,'.',1,2)-1 ) c,
7        substr(ip, instr(ip,'.',1,3)+1 ) d
8   from (select '92.111.0.2' as ip from t1)
```

### PostgreSQL

[]()Use the built-in function SPLIT\_PART to parse an IP address:

```
1 select split_part(y.ip,'.',1) as a,
2        split_part(y.ip,'.',2) as b,
3        split_part(y.ip,'.',3) as c,
4        split_part(y.ip,'.',4) as d
5   from (select cast('92.111.0.2' as text) as ip from t1) as y
```

### SQL Server

Use the recursive WITH clause to simulate an iteration through the IP address while using SUBSTR to easily parse it. A leading period is added to the IP address so that every set of numbers has a period in front of it and can be treated the same way:

```
 1  with x (pos,ip) as (
 2    select 1 as pos,'.92.111.0.222' as ip from t1
 3    union all
 4   select pos+1,ip from x where pos+1 <= 20
 5  )
 6  select max(case when rn=1 then e end) a,
 7         max(case when rn=2 then e end) b,
 8         max(case when rn=3 then e end) c,
 9         max(case when rn=4 then e end) d
10    from  (
11  select pos,c,d,
12         case when charindex('.',d) > 0
13            then substring(d,1,charindex('.',d)-1)
14            else d
15        end as e,
16        row_number() over(order by pos desc) rn
17   from (
18 select pos, ip,right(ip,pos) as c,
19        substring(right(ip,pos),2,len(ip)) as d
20   from x
21  where pos <= len(ip)
22    and substring(right(ip,pos),1,1) = '.'
23       ) x
24       ) y
```

## Discussion

By using the built-in functions for your database, you can easily walk through parts of a string. The key is being able to locate each of the periods in the address. Then you can parse the numbers between each.

In [Recipe 6.17](#sqlckbk-CHP-6-SECT-17) we will see how regular expressions can be used with most RDBMSs—parsing an IP address is also a good area to apply this idea.[]()

# 6.16 Comparing Strings by Sound

## Problem

[]()Between spelling mistakes and legitimate ways to spell words differently, such as British versus American spelling, there are many times that two words that you want to match are represented by different strings of characters. Fortunately, SQL provides a way to represent the way words sound, which allows you to find strings that sound the same even though the underlying characters aren’t identical.

For example, you have a list of authors’ names, including some from an earlier era when spelling wasn’t as fixed as it is now, combined with some extra misspellings and typos. The following column of names is an example:

```
 a_name
----
1 Johnson
2 Jonson
3 Jonsen
4 Jensen
5 Johnsen
6 Shakespeare
7 Shakspear
8 Shaekspir
9 Shakespar
```

Although this is likely part of a longer list, you’d like to identify which of these names are plausible phonetic matches for other names on the list. While this is an exercise where there is more than one possible solution, your solution will look something like this (the meaning of the last column will become clearer by the end of the recipe):

```
a_name1        a_name2       soundex_name
----           ----          ----
Jensen	       Johnson	     J525
Jensen	       Jonson	     J525
Jensen	       Jonsen	     J525
Jensen	       Johnsen	     J525
Johnsen	       Johnson	     J525
Johnsen	       Jonson	     J525
Johnsen	       Jonsen	     J525
Johnsen	       Jensen	     J525
...
Jonson	       Jensen	     J525
Jonson	       Johnsen	     J525
Shaekspir	   Shakspear	 S216
Shakespar	   Shakespeare	 S221
Shakespeare	   Shakespar	 S221
Shakspear	   Shaekspir	 S216
```

## Solution

[]()Use the SOUNDEX function to convert strings of characters into the way they sound when spoken in English. A simple self-join allows you to compare values from the same column.

```
1  select an1.a_name as name1, an2.a_name as name2,
2  SOUNDEX(an1.a_name) as Soundex_Name
3  from author_names an1
4  join author_names an2
5  on (SOUNDEX(an1.a_name)=SOUNDEX(an2.a_name)
6  and an1.a_name not like an2.a_name)
```

## Discussion

The thinking behind SOUNDEX predates both databases and computing, as it originated with the US Census as an attempt to resolve different spellings of proper names for both people and places. There are many algorithms that attempt the same task as SOUNDEX, and, of course, there are alternative versions for languages other than English. However, we cover SOUNDEX, as it comes with most RDBMSs.

Soundex keeps the first letter of the name and then replaces the remaining values with numbers that have the same value if they are phonetically similar. For example, *m* and *n* are both replaced with the number 5.

In the previous example, the actual Soundex output is shown in the *Soundex\_Name* column. This is just to show what is happening, and not necessary for the solution; some RDMSs even have a function that hides the Soundex result, such as SQL Server’s `Difference` function, which compares two strings using Soundex and returns a similarity scale from 0 to 4 (e.g., 4 is a perfect match between the Soundex outputs, representing 4/4 characters in the Soundex version if the two strings match).

Sometimes Soundex will be sufficient for your needs; other times it won’t be. However, a small amount of research, possibly using texts such as *Data Matching* (Christen, 2012), will help you find other algorithms that are frequently (but not always) simple to implement as a user-defined function, or in another programming language to suit your taste and needs.[]()

# 6.17 Finding Text Not Matching a Pattern

## Problem

[]()You have a text field that contains some structured text values (e.g., phone numbers), and you want to find occurrences where those values are structured incorrectly. For example, you have data like the following:

```
select emp_id, text 
   from employee_comment 

EMP_ID     TEXT
---------- ------------------------------------------------------------
7369       126 Varnum, Edmore MI 48829, 989 313-5351
7499       1105 McConnell Court
           Cedar Lake MI 48812
           Home: 989-387-4321
           Cell: (237) 438-3333
```

and you want to list rows having invalidly formatted phone numbers. For example, you want to list the following row because its phone number uses two different separator characters:

```
7369            126 Varnum, Edmore MI 48829, 989 313-5351
```

You want to consider valid only those phone numbers that use the same character for both delimiters.

## Solution

This problem has a multipart solution:

1. Find a way to describe the universe of apparent phone numbers that you want to consider.
2. Remove any validly formatted phone numbers from consideration.
3. See whether you still have any apparent phone numbers left. If you do, you know those are invalidly formatted.

```
select emp_id, text 
 from employee_comment 
 where regexp_like(text, '[0-9]{3}[-. ][0-9]{3}[-. ][0-9]{4}') 
   and regexp_like( 
          regexp_replace(text, 
             '[0-9]{3}([-. ])[0-9]{3}\1[0-9]{4}','***'), 
          '[0-9]{3}[-. ][0-9]{3}[-. ][0-9]{4}') 

    EMP_ID TEXT
---------- ------------------------------------------------------------
  7369     126 Varnum, Edmore MI 48829, 989 313-5351
  7844     989-387.5359
  9999     906-387-1698, 313-535.8886
```

Each of these rows contains at least one apparent phone number that is not correctly formatted.

## Discussion

The key to this solution lies in the detection of an “apparent phone number.” Given that the phone numbers are stored in a comment field, any text at all in the field could be construed to be an invalid phone number. You need a way to narrow the field to a more reasonable set of values to consider. You don’t, for example, want to see the following row in your output:

```
    EMP_ID TEXT
---------- ----------------------------------------------------------
      7900 Cares for 100-year-old aunt during the day. Schedule only
           for evening and night shifts.
```

Clearly there’s no phone number at all in this row, much less one that is invalid. We can all see that. The question is, how do you get the RDBMS to “see” it? We think you’ll enjoy the answer. Please read on.

###### Tip

This recipe comes (with permission) from an article by Jonathan Gennick called “Regular Expression Anti-Patterns.”

The solution uses Pattern A to define the set of “apparent” phone numbers to consider:

```
Pattern A: [0-9]{3}[-. ][0-9]{3}[-. ][0-9]{4}
```

Pattern A checks for two groups of three digits followed by one group of four digits. Any one of a dash (-), a period (.), or a space is accepted as a delimiter between groups. You could come up with a more complex pattern. For example, you could decide that you also want to consider seven-digit phone numbers. But don’t get side-tracked. The point now is that somehow you do need to define the universe of possible phone number strings to consider, and for this problem that universe is defined by Pattern A. You can define a different Pattern A, and the general solution still applies.

The solution uses Pattern A in the WHERE clause to ensure that only rows having potential phone numbers (as defined by the pattern!) are considered:

```
select emp_id, text
  from employee_comment
 where regexp_like(text, '[0-9]{3}[-. ][0-9]{3}[-. ][0-9]{4}')
```

Next, you need to define what a “good” phone number looks like. The solution does this using Pattern B:

```
Pattern B: [0-9]{3}([-. ])[0-9]{3}\1[0-9]{4}
```

This time, the pattern uses \\1 to reference the first subexpression. Whichever character is matched by (\[-. ]) must also be matched by \\1. Pattern B describes good phone numbers, which must be eliminated from consideration (as they are not bad). []()The solution eliminates the well-formatted phone numbers through a call to REGEXP_ REPLACE:

```
regexp_replace(text,
	   '[0-9]{3}([-. ])[0-9]{3}\1[0-9]{4}','***'),
```

This call to REGEXP\_REPLACE occurs in the WHERE clause. Any well-formatted phone numbers are replaced by a string of three asterisks. Again, Pattern B can be any pattern that you desire. The point is that Pattern B describes the acceptable pattern that you are after.

Having replaced well-formatted phone numbers with strings of three asterisks (**\***), any “apparent” phone numbers that remain must, by definition, be poorly formatted. The solution applies REGEXP\_LIKE to the output from REGEXP\_LIKE to see whether any poorly formatted phone numbers remain:

```
and regexp_like(
       regexp_replace(text,
          '[0-9]{3}([-. ])[0-9]{3}\1[0-9]{4}','***'),
       '[0-9]{3}[-. ][0-9]{3}[-. ][0-9]{4}')
```

###### Tip

Regular expressions are a big topic in their own right, requiring practice to master. Once you do master them, you will find they match a great variety of string patterns with ease. We recommend studying a book such as *Mastering Regular Expressions* by Jeffrey Friedl to get your regular expression skills to the required level.[]()

# 6.18 Summing Up

Matching on strings can be a painful task. SQL has added a range of tools to reduce the pain, and mastering them will keep you out of trouble. Although a lot can be done with the native SQL string functions, using the regular expression functions that are increasingly available takes it to another level altogether.[]()