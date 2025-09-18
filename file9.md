# Chapter 9. Date Manipulation

[]()This chapter introduces recipes for searching and modifying dates. Queries involving dates are very common. Thus, you need to know how to think when working with dates, and you need to have a good understanding of the functions that your RDBMS platform provides for manipulating them. The recipes in this chapter form an important foundation for future work as you move on to more complex queries involving not only dates, but times, too.

Before getting into the recipes, we want to reinforce the concept (mentioned in the preface) of using these solutions as guidelines to solving your specific problems. Try to think “big picture.” For example, if a recipe solves a problem for the current month, keep in mind that you may be able to use the recipe for any month (with minor modifications), not just the month used in the recipe. Again, these recipes are guidelines, the absolute final option. There’s no possible way a book can contain an answer for all your problems, but if you understand what is presented here, modifying these solutions to fit your needs is trivial. Also consider alternative versions of these solutions. For instance, if the solution uses one particular function provided by your RDBMS, it is worth the time and effort to find out if there is an alternative—maybe one that is more or less efficient than what is presented here. Knowing your options will make you a better SQL programmer.

###### Tip

The recipes presented in this chapter use simple date data types. If you are using more complex date data types, you will need to adjust the solutions accordingly.

# 9.1 Determining Whether a Year Is a Leap Year

## Problem

[]()[]()You want to determine whether the current year is a leap year.

## Solution

If you’ve worked on SQL for some time, there’s no doubt that you’ve come across several techniques for solving this problem. Just about all the solutions we’ve encountered work well, but the one presented in this recipe is probably the simplest. This solution simply checks the last day of February; if it is the 29th, then the current year is a leap year.

### DB2

Use the recursive WITH clause to return each day in February. Use the aggregate function MAX to determine the last day in February:

```
 1   with x (dy,mth)
 2     as (
 3 select dy, month(dy)
 4   from (
 5 select (current_date -
 6          dayofyear(current_date) days +1 days)
 7           +1 months as dy
 8   from t1
 9        ) tmp1
10  union all
11 select dy+1 days, mth
12   from x
13  where month(dy+1 day) = mth
14 )
15 select max(day(dy))
16   from x
```

### Oracle

[]()Use the function LAST\_DAY to find the last day in February:

```
1 select to_char(
2          last_day(add_months(trunc(sysdate,'y'),1)),
3         'DD')
4   from t1
```

### PostgreSQL

[]()Use the function GENERATE\_SERIES to return each day in February, and then use the aggregate function MAX to find the last day in February:

```
 1 select max(to_char(tmp2.dy+x.id,'DD')) as dy
 2   from (
 3 select dy, to_char(dy,'MM') as mth
 4   from (
 5 select cast(cast(
 6             date_trunc('year',current_date) as date)
 7                        + interval '1 month' as date) as dy
 8   from t1
 9        ) tmp1
10        ) tmp2, generate_series (0,29) x(id)
11  where to_char(tmp2.dy+x.id,'MM') = tmp2.mth
```

### MySQL

[]()Use the function LAST\_DAY to find the last day in February:

```
1 select day(
2        last_day(
3        date_add(
4        date_add(
5        date_add(current_date,
6                 interval -dayofyear(current_date) day),
7                 interval 1 day),
8                 interval 1 month))) dy
9   from t1
```

### SQL Server

This solution uses SQL Server’s DAY function to attempt the date the 29th of February in the current year. If the 29th of February is not valid, the DAY function will return a NULL. The COALESCE function will return the value from the DAY function if it is valid, or the integer 28 if it is not:

```
 select coalesce
	        (day
			(cast(concat
			(year(getdate()),'-02-29')
			 as date))
			 ,28);
```

## Discussion

### DB2

The inline view TMP1 in the recursive view X returns the first day in February by:

1. Starting with the current date
2. Using DAYOFYEAR to determine the number of days into the current year that the current date represents
3. Subtracting that number of days from the current date to get December 31 of the prior year and then adding one to get to January 1 of the current year
4. Adding one month to get to February 1

The result of all this math is shown here:

```
 select (current_date 
            dayofyear(current_date) days +1 days) +1 months as dy 
    from t1 

DY
-----------
01-FEB-2005
```

[]()The next step is to return the month of the date returned by inline view TMP1 by using the MONTH function:

```
select dy, month(dy) as mth 
   from ( 
 select (current_date 
           dayofyear(current_date) days +1 days) +1 months as dy 
   from t1 
        ) tmp1 

DY          MTH
----------- ---
01-FEB-2005   2
```

The results presented thus far provide the start point for the recursive operation that generates each day in February. To return each day in February, repeatedly add one day to DY until you are no longer in the month of February. A portion of the results of the WITH operation is shown here:

```
  with x (dy,mth) 
     as ( 
 select dy, month(dy) 
   from ( 
 select (current_date - 
          dayofyear(current_date) days +1 days) +1 months as dy 
   from t1 
        ) tmp1 
  union all 
  select dy+1 days, mth 
    from x 
   where month(dy+1 day) = mth 
  ) 
  select dy,mth 
    from x 

DY          MTH
----------- ---
01-FEB-2005   2
…
10-FEB-2005   2
…
28-FEB-2005   2
```

The final step is to use the MAX function on the DY column to return the last day in February; if it is the 29th, you are in a leap year.

### Oracle

The first step is to find the beginning of the year using the TRUNC function:

```
select trunc(sysdate,'y') 
   from t1 

DY
-----------
01-JAN-2005
```

Because the first day of the year is January 1st, the next step is to add one month to get to February 1st:

```
select add_months(trunc(sysdate,'y'),1) dy 
   from t1 

DY
-----------
01-FEB-2005
```

The next step is to use the LAST\_DAY function to find the last day in February:

```
select last_day(add_months(trunc(sysdate,'y'),1)) dy 
   from t1 

DY
-----------
28-FEB-2005
```

The final step (which is optional) is to use TO\_CHAR to return either 28 or 29.

### PostgreSQL

The first step is to examine the results returned by inline view TMP1. Use the []()DATE\_TRUNC function to find the beginning of the current year and cast that result as a DATE:

```
select cast(date_trunc('year',current_date) as date) as dy 
   from t1 


DY
-----------
01-JAN-2005
```

The next step is to add one month to the first day of the current year to get the first day in February, casting the result as a date:

```
select cast(cast( 
             date_trunc('year',current_date) as date) 
                        + interval '1 month' as date) as dy 
   from t1 

DY
-----------
01-FEB-2005
```

[]()Next, return DY from inline view TMP1 along with the numeric month of DY. Return the numeric month by using the TO\_CHAR function:

```
select dy, to_char(dy,'MM') as mth 
    from ( 
  select cast(cast( 
              date_trunc('year',current_date) as date) 
                         + interval '1 month' as date) as dy 
    from t1 
         ) tmp1 

DY          MTH
----------- ---
01-FEB-2005   2
```

The results shown thus far comprise the result set of inline view TMP2. []()Your next step is to use the extremely useful function GENERATE\_SERIES to return 29 rows (values 1 through 29). Every row returned by GENERATE\_SERIES (aliased X) is added to DY from inline view TMP2. Partial results are shown here:

```
select tmp2.dy+x.id as dy, tmp2.mth 
   from ( 
 select dy, to_char(dy,'MM') as mth 
   from ( 
 select cast(cast( 
             date_trunc('year',current_date) as date) 
                        + interval '1 month' as date) as dy 
   from t1 
        ) tmp1 
        ) tmp2, generate_series (0,29) x(id) 
  where to_char(tmp2.dy+x.id,'MM') = tmp2.mth 

DY          MTH
----------- ---
01-FEB-2005  02
…
10-FEB-2005  02
…
28-FEB-2005  02
```

The final step is to use the MAX function to return the last day in February. []()The function TO\_CHAR is applied to that value and will return either 28 or 29.

### MySQL

The first step is to find the first day of the current year by subtracting from the current date the number of days it is into the year and then adding one day. Do all of this with the DATE\_ADD function:

```
select date_add( 
        date_add(current_date, 
                 interval -dayofyear(current_date) day), 
                 interval 1 day) dy 
   from t1 

DY
-----------
01-JAN-2005
```

Then add one month again using the DATE\_ADD function:

```
select date_add( 
        date_add( 
        date_add(current_date, 
                 interval -dayofyear(current_date) day), 
                 interval 1 day), 
                 interval 1 month) dy 
   from t1 

DY
-----------
01-FEB-2005
```

Now that you’ve made it to February, use the LAST\_DAY function to find the last day of the month:

```
select last_day( 
        date_add( 
        date_add( 
        date_add(current_date, 
                 interval -dayofyear(current_date) day), 
                 interval 1 day), 
                 interval 1 month)) dy 
   from t1 

DY
-----------
28-FEB-2005
```

[]()The final step (which is optional) is to use the DAY function to return either a 28 or 29.

### SQL Server

We can create a new date in most RDMSs by creating a string in a recognized date format and using CAST to change format. We can therefore use the current year by retrieving the year from the current date. In SQL Server, this is done by applying YEAR to GET\_DATE:

```
select YEAR(GETDATE());
```

This will return the year as an integer. We can then create 29th of February by using CONCAT and CAST:

```
select cast(concat
				   (year(getdate()),'-02-29');
```

However, this won’t be a *real* date if the current year isn’t a leap year. For example, there is no date 2019-02-29. []()Hence, if we try to use an operator like DAY to find any of its parts, it will return NULL. []()[]()Therefore, use COALESCE and DAY to determine whether there is a 29th day in the month.[]()[]()

# 9.2 Determining the Number of Days in a Year

## Problem

[]()You want to count the number of days in the current year.

## Solution

The number of days in the current year is the difference between the first day of the next year and the first day of the current year (in days). For each solution the steps are:

1. Find the first day of the current year.
2. Add one year to that date (to get the first day of the next year).
3. Subtract the current year from the result of Step 2.

The solutions differ only in the built-in functions that you use to perform these steps.

### DB2

[]()[]()Use the function DAYOFYEAR to help find the first day of the current year, and use DAYS to find the number of days in the current year:

```
1 select days((curr_year + 1 year)) - days(curr_year)
2   from (
3 select (current_date -
4         dayofyear(current_date) day +
5          1 day) curr_year
6   from t1
7        ) x
```

### Oracle

Use the function TRUNC to find the beginning of the current year, and use ADD_ MONTHS to then find the beginning of next year:

```
1 select add_months(trunc(sysdate,'y'),12) - trunc(sysdate,'y')
2   from dual
```

### PostgreSQL

[]()Use the function DATE\_TRUNC to find the beginning of the current year. Then use interval arithmetic to determine the beginning of next year:

```
1 select cast((curr_year + interval '1 year') as date) - curr_year
2   from (
3 select cast(date_trunc('year',current_date) as date) as curr_year
4   from t1
5        ) x
```

### MySQL

[]()Use ADDDATE to help find the beginning of the current year. []()Use DATEDIFF and interval arithmetic to determine the number of days in the year:

```
1 select datediff((curr_year + interval 1 year),curr_year)
2   from (
3 select adddate(current_date,-dayofyear(current_date)+1) curr_year
4   from t1
5        ) x
```

### SQL Server

[]()Use the function DATEADD to find the first day of the current year. Use DATEDIFF to return the number of days in the current year:

```
1 select datediff(d,curr_year,dateadd(yy,1,curr_year))
2   from (
3 select dateadd(d,-datepart(dy,getdate())+1,getdate()) curr_year
4   from t1
5        ) x
```

## Discussion

### DB2

The first step is to find the first day of the current year. Use DAYOFYEAR to determine how many days you are into the current year. Subtract that value from the current date to get the last day of last year, and then add 1:

```
select (current_date 
         dayofyear(current_date) day + 
          1 day) curr_year 
   from t1 

CURR_YEAR
-----------
01-JAN-2005
```

Now that you have the first day of the current year, just add one year to it; this gives you the first day of next year. Then subtract the beginning of the current year from the beginning of the next year.[]()

### Oracle

[]()The first step is to find the first day of the current year, which you can easily do by invoking the built-in TRUNC function and passing Y as the second argument (thereby truncating the date to the beginning of the year):

```
select select trunc(sysdate,'y') curr_year 
   from dual 

CURR_YEAR
-----------
01-JAN-2005
```

Then add one year to arrive at the first day of the next year. Finally, subtract the two dates to find the number of days in the current year.

### PostgreSQL

Begin by finding the first day of the current year. To do that, invoke the DATE_ TRUNC function as follows:

```
select cast(date_trunc('year',current_date) as date) as curr_year 
   from t1 

CURR_YEAR
-----------
01-JAN-2005
```

You can then easily add a year to compute the first day of next year. Then all you need to do is to subtract the two dates. Be sure to subtract the earlier date from the later date. The result will be the number of days in the current year.

### MySQL

Your first step is to find the first day of the current year. Use DAYOFYEAR to find how many days you are into the current year. Subtract that value from the current date, and add one:

```
select adddate(current_date,-dayofyear(current_date)+1) curr_year 
   from t1 

CURR_YEAR
-----------
01-JAN-2005
```

Now that you have the first day of the current year, your next step is to add one year to it to get the first day of next year. Then subtract the beginning of the current year from the beginning of the next year. The result is the number of days in the current year.

### SQL Server

[]()Your first step is to find the first day of the current year. Use DATEADD and DATEPART to subtract from the current date the number of days into the year the current date is, and add 1:

```
select dateadd(d,-datepart(dy,getdate())+1,getdate()) curr_year 
   from t1 

CURR_YEAR
-----------
01-JAN-2005
```

Now that you have the first day of the current year, your next step is to add one year to it to get the first day of the next year. Then subtract the beginning of the current year from the beginning of the next year. The result is the number of days in the current year.[]()

# 9.3 Extracting Units of Time from a Date

## Problem

[]()You want to break the current date down into six parts: day, month, year, second, minute, and hour. You want the results to be returned as numbers.

## Solution

Use of the current date is arbitrary. Feel free to use this recipe with other dates. Most vendors have now adopted the ANSI standard function for extracting parts of dates, EXTRACT, although SQL Server is an exception. They also retain their own legacy methods.

### DB2

DB2 implements a set of built-in functions that make it easy for you to extract portions of a date. []()[]()[]()[]()[]()[]()The function names HOUR, MINUTE, SECOND, DAY, MONTH, and YEAR conveniently correspond to the units of time you can return: if you want the day, use DAY; hour, use HOUR; etc. For example:

```
	1 select    hour( current_timestamp ) hr, 
 	2         minute( current_timestamp ) min, 
 	3         second( current_timestamp ) sec, 
 	4            day( current_timestamp ) dy, 
 	5          month( current_timestamp ) mth, 
 	6            year( current_timestamp ) yr 
 	7   from t1 

select
        extract(hour from current_timestamp)
      , extract(minute from current_timestamp
      , extract(second from current_timestamp)
      , extract(day from current_timestamp)
      , extract(month from current_timestamp)
      , extract(year from current_timestamp)

	  HR   MIN   SEC    DY   MTH    YR
	---- ----- ----- ----- ----- -----
	  20    28    36    15     6  2005
```

### Oracle

[]()[]()Use functions TO\_CHAR and TO\_NUMBER to return specific units of time from a date:

```
1  select to_number(to_char(sysdate,'hh24')) hour, 
 2         to_number(to_char(sysdate,'mi')) min, 
 3         to_number(to_char(sysdate,'ss')) sec, 
 4         to_number(to_char(sysdate,'dd')) day, 
 5         to_number(to_char(sysdate,'mm')) mth, 
 6         to_number(to_char(sysdate,'yyyy')) year 
 7   from dual 

  HOUR   MIN   SEC   DAY   MTH  YEAR
  ---- ----- ----- ----- ----- -----
    20    28    36    15     6  2005
```

### PostgreSQL

Use functions TO\_CHAR and TO\_NUMBER to return specific units of time from a date:

```
1 select to_number(to_char(current_timestamp,'hh24'),'99') as hr, 
 2        to_number(to_char(current_timestamp,'mi'),'99') as min, 
 3        to_number(to_char(current_timestamp,'ss'),'99') as sec, 
 4        to_number(to_char(current_timestamp,'dd'),'99') as day, 
 5        to_number(to_char(current_timestamp,'mm'),'99') as mth, 
 6        to_number(to_char(current_timestamp,'yyyy'),'9999') as yr 
 7   from t1 

   HR   MIN   SEC   DAY   MTH    YR
 ---- ----- ----- ----- ----- -----
 20      28    36    15     6  2005
```

### MySQL

[]()Use the DATE\_FORMAT function to return specific units of time from a date:

```
1 select date_format(current_timestamp,'%k') hr, 
 2        date_format(current_timestamp,'%i') min, 
 3        date_format(current_timestamp,'%s') sec, 
 4        date_format(current_timestamp,'%d') dy, 
 5        date_format(current_timestamp,'%m') mon, 
 6        date_format(current_timestamp,'%Y') yr 
 7   from t1 

  HR   MIN   SEC   DAY   MTH    YR
---- ----- ----- ----- ----- -----
  20    28    36    15     6  2005
```

### SQL Server

[]()Use the function DATEPART to return specific units of time from a date:

```
1 select datepart( hour, getdate()) hr, 
 2        datepart( minute,getdate()) min, 
 3        datepart( second,getdate()) sec, 
 4        datepart( day, getdate()) dy, 
 5        datepart( month, getdate()) mon, 
 6        datepart( year, getdate()) yr 
 7   from t1 

  HR   MIN   SEC   DAY   MTH    YR
---- ----- ----- ----- ----- -----
  20    28    36    15     6  2005
```

## Discussion

There’s nothing fancy in these solutions; just take advantage of what you’re already paying for. Take the time to learn the date functions available to you. This recipe only scratches the surface of the functions presented in each solution. You’ll find that each of the functions takes many more arguments and can return more information than what this recipe provides you.[]()

# 9.4 Determining the First and Last Days of a Month

## Problem

[]()You want to determine the first and last days for the current month.

## Solution

The solutions presented here are for finding first and last days for the current month. Using the current month is arbitrary. With a bit of adjustment, you can make the solutions work for any month.

### DB2

[]()Use the DAY function to return the number of days into the current month the current date represents. Subtract this value from the current date, and then add one to get the first of the month. To get the last day of the month, add one month to the current date, and then subtract from it the value returned by the DAY function as applied to the current date:

```
1 select (date(current_date) - day(date(current_date)) day + 1 day) firstday,
2        (date(current_date)+1 month
3         - day(date(current_date)+1 month) day) lastday
4   from t1
```

### Oracle

[]()[]()Use the function TRUNC to find the first of the month, and use the function LAST\_DAY to find the last day of the month:

```
1 select trunc(sysdate,'mm') firstday,
2        last_day(sysdate) lastday
3   from dual
```

###### Tip

Using TRUNC as described here will result in the loss of any time-of-day component, whereas LAST\_DAY will preserve the time of day.

### PostgreSQL

[]()Use the DATE\_TRUNC function to truncate the current date to the first of the current month. Once you have the first day of the month, add one month and subtract one day to find the end of the current month:

```
	1 select firstday,
	2        cast(firstday + interval '1 month'
	3                      - interval '1 day' as date) as lastday
	4   from (
	5 select cast(date_trunc('month',current_date) as date) as firstday
	6   from t1
	7        ) x
```

### MySQL

[]()Use the DATE\_ADD and DAY functions to find the number of days into the month the current date is. Then subtract that value from the current date and add one to find the first of the month. []()To find the last day of the current month, use the LAST\_DAY function:

```
1 select date_add(current_date,
2                 interval -day(current_date)+1 day) firstday,
3        last_day(current_date) lastday
4   from t1
```

### SQL Server

[]()[]()Use the DATEADD and DAY functions to find the number of days into the month represented by the current date. Then subtract that value from the current date and add one to find the first of the month. To get the last day of the month, add one month to the current date, and then subtract from that result the value returned by the DAY function applied to the current date, again using the functions DAY and DATEADD:

```
1 select dateadd(day,-day(getdate())+1,getdate()) firstday,
2        dateadd(day,
3                -day(dateadd(month,1,getdate())),
4                dateadd(month,1,getdate())) lastday
5   from t1
```

## Discussion

### DB2

To find the first day of the month, simply find the numeric value of the current day of the month, and then subtract this from the current date. For example, if the date is March 14th, the numeric day value is 14. Subtracting 14 days from March 14th gives you the last day of the month in February. From there, simply add one day to get to the first of the current month. The technique to get the last day of the month is similar to that of the first: subtract the numeric day of the month from the current date to get the last day of the prior month. Since we want the last day of the current month (not the last day of the prior month), we need to add one month to the current date.

### Oracle

To find the first day of the current month, use the TRUNC function with “mm” as the second argument to “truncate” the current date down to the first of the month. To find the last day of the current month, simply use the LAST\_DAY function.

### PostgreSQL

To find the first day of the current month, use the DATE\_TRUNC function with “month” as the second argument to “truncate” the current date down to the first of the month. To find the last day of the current month, add one month to the first day of the month, and then subtract one day.

### MySQL

To find the first day of the month, use the DAY function. The DAY function returns the day of the month for the date passed. If you subtract the value returned by DAY(CURRENT\_DATE) from the current date, you get the last day of the prior month; add one day to get the first day of the current month. To find the last day of the current month, simply use the LAST\_DAY function.

### SQL Server

To find the first day of the month, use the DAY function. The DAY function conveniently returns the day of the month for the date passed. If you subtract the value returned by DAY(GETDATE()) from the current date, you get the last day of the prior month; add one day to get the first day of the current month. To find the last day of the current month, use the DATEADD function. Add one month to the current date, then subtract from it the value returned by DAY(GETDATE()) to get the first day of the current month. Add one month to the current date, and then subtract from it the value returned by DAY(DATEADD(MONTH,1,GETDATE())) to get the last day of the current month.[]()

# 9.5 Determining All Dates for a Particular Weekday Throughout a Year

## Problem

[]()You want to find all the dates in a year that correspond to a given day of the week. For example, you may want to generate a list of Fridays for the current year.

## Solution

Regardless of vendor, the key to the solution is to return each day for the current year and keep only those dates corresponding to the day of the week that you care about. The solution examples retain all the Fridays.

### DB2

Use the recursive WITH clause to return each day in the current year. Then use the function DAYNAME to keep only Fridays:

```
 1   with x (dy,yr)
 2     as (
 3 select dy, year(dy) yr
 4   from (
 5 select (current_date -
 6          dayofyear(current_date) days +1 days) as dy
 7   from t1
 8        ) tmp1
 9  union all
 10 select dy+1 days, yr
 11   from x
 12  where year(dy +1 day) = yr
 13 )
 14 select dy
 15   from x
 16  where dayname(dy) = 'Friday'
```

### Oracle

Use the recursive CONNECT BY clause to return each day in the current year. []()Then use the function TO\_CHAR to keep only Fridays:

```
1   with x
2     as (
3 select trunc(sysdate,'y')+level-1 dy
4   from t1
5   connect by level <=
6      add_months(trunc(sysdate,'y'),12)-trunc(sysdate,'y')
7 )
8 select *
9   from x
10  where to_char( dy, 'dy') = 'fri'
```

### PostgreSQL

Use a recursive CTE to generate every day of the year, and filter out days that aren’t Fridays. This version makes use of the ANSI standard EXTRACT, so it will run on a wide variety of RDBMs:

```
1   with recursive cal (dy)
2   as (
3   select current_date
4    -(cast
5     (extract(doy from current_date) as integer)
6    -1)
7    union all
8    select dy+1
9    from cal
10   where extract(year from dy)=extract(year from (dy+1))
11     )
12
13   select dy,extract(dow from dy) from cal
14   where cast(extract(dow from dy) as integer) = 5
```

### MySQL

Use a recursive CTE to find all the days in the year. Then filter all days but Fridays:

```
1	  with recursive cal (dy,yr)
2   as
3     (
4     select dy, extract(year from dy) as yr
5   from
6     (select adddate
7             (adddate(current_date, interval - dayofyear(current_date)
8   day), interval 1 day) as dy) as tmp1
9   union all
10     select date_add(dy, interval 1 day), yr
11  from cal
12  where extract(year from date_add(dy, interval 1 day)) = yr
13  )
14     select dy from cal
15     where dayofweek(dy) = 6
```

### SQL Server

Use the recursive WITH clause to return each day in the current year. Then use the function DAYNAME to keep only Fridays:

```
 1   with x (dy,yr)
 2     as (
 3 select dy, year(dy) yr
 4   from (
 5 select getdate()-datepart(dy,getdate())+1 dy
 6   from t1
 7        ) tmp1
 8  union all
 9 select dateadd(dd,1,dy), yr
10   from x
11  where year(dateadd(dd,1,dy)) = yr
12 )
13 select x.dy
14   from x
15  where datename(dw,x.dy) = 'Friday'
16 option (maxrecursion 400)
```

## Discussion

### DB2

To find all the Fridays in the current year, you must be able to return every day in the current year. The first step is to find the first day of the year by using the DAYOFYEAR function. Subtract the value returned by DAYOFYEAR(CURRENT\_DATE) from the current date to get December 31 of the prior year, and then add one to get the first day of the current year:

```
select (current_date 
          dayofyear(current_date) days +1 days) as dy 
   from t1 

DY
-----------
01-JAN-2005
```

Now that you have the first day of the year, use the WITH clause to repeatedly add one day to the first day of the year until you are no longer in the current year. The result set will be every day in the current year (a portion of the rows returned by the recursive view X is shown here):

```
 with x (dy,yr) 
    as ( 
 select dy, year(dy) yr 
   from ( 
 select (current_date 
          dayofyear(current_date) days +1 days) as dy 
   from t1 
         ) tmp1 
 union all 
 select dy+1 days, yr 
   from x 
  where year(dy +1 day) = yr 
 ) 
 select dy 
   from x 

DY
-----------
01-JAN-2020
…
15-FEB-2020
…
22-NOV-2020
…
31-DEC-2020
```

The final step is to use the DAYNAME function to keep only rows that are Fridays.

### Oracle

To find all the Fridays in the current year, you must be able to return every day in the current year. Begin by using the TRUNC function to find the first day of the year:

```
select trunc(sysdate,'y') dy 
 	  from t1 

	DY
	-----------
	01-JAN-2020
```

Next, use the CONNECT BY clause to return every day in the current year (to understand how to use CONNECT BY to generate rows, see [Recipe 10.5](https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/ch10.html#sqlckbk-CHP-10-SECT-5)).

###### Tip

As an aside, this recipe uses the WITH clause, but you can also use an inline view.

A portion of the result set returned by view X is shown here:

```
 with x 
    as ( 
 select trunc(sysdate,'y')+level-1 dy 
 from t1 
  connect by level <= 
     add_months(trunc(sysdate,'y'),12)-trunc(sysdate,'y') 
 ) 
 select * 
 from x 

DY
-----------
01-JAN-2020
…
15-FEB-2020
…
22-NOV-2020
…
31-DEC-2020
```

[]()The final step is to use the TO\_CHAR function to keep only Fridays.

### PostgreSQL

To find the Fridays, first find all the days. You need to find the first day of the year, and then use the recursive CTE to fill in the rest of the days. Remember PostgreSQL is one of the packages that requires the use of the RECURSIVE keyword to identify a recursive CTE.

### MySQL

To find all the Fridays in the current year, you must be able to return every day in the current year. The first step is to find the first day of the year. Subtract the value returned by DAYOFYEAR(CURRENT\_DATE) from the current date, and then add one to get the first day of the current year:

```
select adddate( 
        adddate(current_date, 
                interval -dayofyear(current_date) day), 
                interval 1 day ) dy 
   from t1 

DY
-----------
01-JAN-2020
```

Once you’ve got the first day of the year, it’s simple to use a recursive CTE to add every day of the year:

```
with cal (dy) as
(select current

union all
select dy+1


	DY
	-----------
	01-JAN-2020
	…
	15-FEB-2020
	…
	22-NOV-2020
	…
	31-DEC-2020
```

The final step is to use the DAYNAME function to keep only Fridays.

### SQL Server

To find all the Fridays in the current year, you must be able to return every day in the current year. The first step is to find the first day of the year by using the DATEPART []()function. Subtract the value returned by DATEPART(DY,GETDATE()) from the current date, and then add one to get the first day of the current year:

```
select getdate()-datepart(dy,getdate())+1 dy 
   from t1 

DY
-----------
01-JAN-2005
```

Now that you have the first day of the year, use the WITH clause and the DATEADD function to repeatedly add one day to the first day of the year until you are no longer in the current year. The result set will be every day in the current year (a portion of the rows returned by the recursive view X is shown here):

```
with x (dy,yr) 
   as ( 
 select dy, year(dy) yr 
   from ( 
 select getdate()-datepart(dy,getdate())+1 dy 
   from t1 
        ) tmp1 
  union all 
 select dateadd(dd,1,dy), yr 
   from x 
  where year(dateadd(dd,1,dy)) = yr 
 ) 
 select x.dy 
   from x 
 option (maxrecursion 400) 

DY
-----------
01-JAN-2020
…
15-FEB-2020
…
22-NOV-2020
…
31-DEC-2020
```

Finally, use the DATENAME function to keep only rows that are Fridays. For this solution to work, you must set MAXRECURSION to at least 366 (the filter on the year portion of the current year, in recursive view X, guarantees you will never generate more than 366 rows).[]()

# 9.6 Determining the Date of the First and Last Occurrences of a Specific Weekday in a Month

## Problem

[]()You want to find, for example, the first and last Mondays of the current month.

## Solution

The choice to use Monday and the current month is arbitrary; you can use the solutions presented in this recipe for any weekday and any month. Because each weekday is 7 days apart from itself, once you have the first instance of a weekday, you can add 7 days to get the second and 14 days to get the third. Likewise, if you have the last instance of a weekday in a month, you can subtract 7 days to get the third and subtract 14 days to get the second.

### DB2

Use the recursive WITH clause to generate each day in the current month and use a CASE expression to flag all Mondays. The first and last Mondays will be the earliest and latest of the flagged dates:

```
 1   with x (dy,mth,is_monday)
 2     as (
 3 select dy,month(dy),
 4        case when dayname(dy)='Monday'
 5              then 1 else 0
 6        end
 7   from (
 8 select (current_date-day(current_date) day +1 day) dy
 9   from t1
10        ) tmp1
11  union all
12 select (dy +1 day), mth,
13        case when dayname(dy +1 day)='Monday'
14             then 1 else 0
15        end
16   from x
17  where month(dy +1 day) = mth
18 )
19 select min(dy) first_monday, max(dy) last_monday
20   from x
21  where is_monday = 1
```

### Oracle

Use the functions NEXT\_DAY and LAST\_DAY, together with a bit of clever date arithmetic, to find the first and last Mondays of the current month:

```
select next_day(trunc(sysdate,'mm')-1,'MONDAY') first_monday,
       next_day(last_day(trunc(sysdate,'mm'))-7,'MONDAY') last_monday
  from dual
```

### PostgreSQL

Use the function DATE\_TRUNC to find the first day of the month. Once you have the first day of the month, you can use simple arithmetic involving the numeric values of weekdays (Sun–Sat is 1–7) to find the first and last Mondays of the current month:

```
 1 select first_monday,
 2        case to_char(first_monday+28,'mm')
 3             when mth then first_monday+28
 4                      else first_monday+21
 5        end as last_monday
 6   from (
 7 select case sign(cast(to_char(dy,'d') as integer)-2)
 8             when 0
 9             then dy
10             when -1
11             then dy+abs(cast(to_char(dy,'d') as integer)-2)
12             when 1
13             then (7-(cast(to_char(dy,'d') as integer)-2))+dy
14        end as first_monday,
15        mth
16   from (
17 select cast(date_trunc('month',current_date) as date) as dy,
18        to_char(current_date,'mm') as mth
19   from t1
20        ) x
21        ) y
```

### MySQL

[]()Use the ADDDATE function to find the first day of the month. Once you have the first day of the month, you can use simple arithmetic on the numeric values of weekdays (Sun–Sat is 1–7) to find the first and last Mondays of the current month:

```
 1 select first_monday,
 2        case month(adddate(first_monday,28))
 3             when mth then adddate(first_monday,28)
 4                      else adddate(first_monday,21)
 5        end last_monday
 6  from (
 7 select case sign(dayofweek(dy)-2)
 8             when 0 then dy
 9             when -1 then adddate(dy,abs(dayofweek(dy)-2))
10             when 1 then adddate(dy,(7-(dayofweek(dy)-2)))
11        end first_monday,
12        mth
13   from (
14 select adddate(adddate(current_date,-day(current_date)),1) dy,
15        month(current_date) mth
16   from t1
17        ) x
18        ) y
```

### SQL Server

Use the recursive WITH clause to generate each day in the current month, and then use a CASE expression to flag all Mondays. The first and last Mondays will be the earliest and latest of the flagged dates:

```
 1   with x (dy,mth,is_monday)
 2     as (
 3 select dy,mth,
 4        case when datepart(dw,dy) = 2
 5             then 1 else 0
 6        end
 7   from (
 8 select dateadd(day,1,dateadd(day,-day(getdate()),getdate())) dy,
 9        month(getdate()) mth
10   from t1
11        ) tmp1
12  union all
13 select dateadd(day,1,dy),
14        mth,
15        case when datepart(dw,dateadd(day,1,dy)) = 2
16             then 1 else 0
17        end
18   from x
19  where month(dateadd(day,1,dy)) = mth
20 )
21 select min(dy) first_monday,
22        max(dy) last_monday
23   from x
24  where is_monday = 1
```

## Discussion

### DB2 and SQL Server

DB2 and SQL Server use different functions to solve this problem, but the technique is exactly the same. If you eyeball both solutions, you’ll see the only difference between the two is the way dates are added. This discussion will cover both solutions, using the DB2 solution’s code to show the results of intermediate steps.

###### Tip

If you do not have access to the recursive WITH clause in the version of SQL Server or DB2 that you are running, you can use the PostgreSQL technique instead.

The first step in finding the first and last Mondays of the current month is to return the first day of the month. Inline view TMP1 in recursive view X finds the first day of the current month by first finding the current date, specifically, the day of the month for the current date. The day of the month for the current date represents how many days into the month you are (e.g., April 10th is the 10th day of the April). If you subtract this day of the month value from the current date, you end up at the last day of the previous month (e.g., subtracting 10 from April 10th puts you at the last day of March). After this subtraction, simply add one day to arrive at the first day of the current month:

```
select (current_date-day(current_date) day +1 day) dy 
   from t1 

DY
-----------
01-JUN-2005
```

Next, find the month for the current date using the MONTH function and a simple CASE expression to determine whether the first day of the month is a Monday:

```
select dy, month(dy) mth, 
         case when dayname(dy)='Monday' 
              then 1 else 0 
         end is_monday 
   from  ( 
 select  (current_date-day(current_date) day +1 day) dy 
   from  t1 
         ) tmp1 

DY          MTH  IS_MONDAY
----------- --- ----------
01-JUN-2005   6          0
```

Then use the recursive capabilities of the WITH clause to repeatedly add one day to the first day of the month until you’re no longer in the current month. Along the way, you will use a CASE expression to determine which days in the month are Mondays (Mondays will be flagged with 1). A portion of the output from recursive view X is shown here:

```
with x (dy,mth,is_monday) 
      as ( 
  select dy,month(dy) mth, 
         case when dayname(dy)='Monday' 
              then 1 else 0 
         end is_monday 
    from ( 
  select (current_date-day(current_date) day +1 day) dy 
    from t1 
         ) tmp1 
   union all 
  select (dy +1 day), mth, 
         case when dayname(dy +1 day)='Monday' 
              then 1 else 0 
         end 
    from x 
   where month(dy +1 day) = mth 
  ) 
  select * 
    from x 

DY          MTH  IS_MONDAY
----------- --- ----------
01-JUN-2005   6          0
02-JUN-2005   6          0
03-JUN-2005   6          0
04-JUN-2005   6          0
05-JUN-2005   6          0
06-JUN-2005   6          1
07-JUN-2005   6          0
08-JUN-2005   6          0
…
```

Only Mondays will have a value of 1 for IS\_MONDAY, so the final step is to use the aggregate functions MIN and MAX on rows where IS\_MONDAY is 1 to find the first and last Mondays of the month.

### Oracle

[]()The function NEXT\_DAY makes this problem easy to solve. To find the first Monday of the current month, first return the last day of the prior month via some date arithmetic involving the TRUNC function:

```
select trunc(sysdate,'mm')-1 dy 
   from dual 

DY
-----------
31-MAY-2005
```

Then use the NEXT\_DAY function to find the first Monday that comes after the last day of the previous month (i.e., the first Monday of the current month):

```
select next_day(trunc(sysdate,'mm')-1,'MONDAY') first_monday 
   from dual 


FIRST_MONDAY
------------
06-JUN-2005
```

To find the last Monday of the current month, start by returning the first day of the current month by using the TRUNC function:

```
select trunc(sysdate,'mm') dy 
   from dual 

DY
-----------
01-JUN-2005
```

The next step is to find the last week (the last seven days) of the month. Use the []()LAST\_DAY function to find the last day of the month, and then subtract seven days:

```
select last_day(trunc(sysdate,'mm'))-7 dy 
   from dual 

DY
-----------
23-JUN-2005
```

If it isn’t immediately obvious, you go back seven days from the last day of the month to ensure that you will have at least one of any weekday left in the month. The last step is to use the function NEXT\_DAY to find the next (and last) []()Monday of the month:

```
select next_day(last_day(trunc(sysdate,'mm'))-7,'MONDAY') last_monday 
   from dual 

LAST_MONDAY
-----------
27-JUN-2005
```

### PostgreSQL and MySQL

PostgreSQL and MySQL also share the same solution approach. The difference is in the functions that you invoke. Despite their lengths, the respective queries are extremely simple; little overhead is involved in finding the first and last Mondays of the current month.

The first step is to find the first day of the current month. The next step is to find the first Monday of the month. Since there is no function to find the next date for a given weekday, you need to use a little arithmetic. The CASE expression beginning on line 7 (of either solution) evaluates the difference between the numeric value for the weekday of the first day of the month and the numeric value corresponding to Monday. Given that the function TO\_CHAR (PostgreSQL), when called with the *D* or *d* format, and the function DAYOFWEEK (MySQL) will return a numeric value from 1 to 7 representing days Sunday to Saturday, Monday is always represented by 2. []()The first test evaluated by CASE is the SIGN of the numeric value of the first day of the month (whatever it may be) minus the numeric value of Monday (2). If the result is zero, then the first day of the month falls on a Monday, and that is the first Monday of the month. If the result is –1, then the first day of the month falls on a Sunday, and to find the first Monday of the month, simply add the difference in days between 2 and 1 (numeric values of Monday and Sunday, respectively) to the first day of the month.

###### Tip

If you are having trouble understanding how this works, forget the weekday names and just do the math. For example, say you happen to be starting on a Tuesday and you are looking for the next Friday. []()When using TO\_CHAR with the *d* format, or DAYOFWEEK, Friday is 6 and Tuesday is 3. To get to 6 from 3, simply take the difference (6–3 = 3) and add it to the smaller value ((6–3) + 3 = 6). So, regardless of the actual dates, if the numeric value of the day you are starting from is less than the numeric value of the day you are searching for, adding the difference between the two dates to the date you are starting from will get you to the date you are searching for.

If the result from SIGN is 1, then the first day of the month falls between Tuesday and Saturday (inclusive). When the first day of the month has a numeric value greater than 2 (Monday), subtract from 7 the difference between the numeric value of the first day of the month and the numeric value of Monday (2), and then add that value to the first day of the month. You will have arrived at the day of the week that you are after, in this case Monday.

###### Tip

Again, if you are having trouble understanding how this works, forget the weekday names and just do the math. For example, suppose you want to find the next Tuesday and you are starting from Friday. Tuesday (3) is less than Friday (6). To get to 3 from 6, subtract the difference between the two values from 7 (7–( |3–6| ) = 4) and add the result (4) to the start day Friday. (The vertical bars in |3–6| generate the absolute value of that difference.) Here, you’re not adding 4 to 6 (which will give you 10); you are adding four days to Friday, which will give you the next Tuesday.

The idea behind the CASE expression is to create a sort of a “next day” function for PostgreSQL and MySQL. []()If you do not start with the first day of the month, the value for DY will be the value returned by CURRENT\_DATE, and the result of the CASE expression will return the date of the next Monday starting from the current date (unless CURRENT\_DATE is a Monday, then that date will be returned).

Now that you have the first Monday of the month, add either 21 or 28 days to find the last Monday of the month. The CASE expression in lines 2–5 determines whether to add 21 or 28 days by checking to see whether 28 days takes you into the next month. The CASE expression does this through the following process:

1. It adds 28 to the value of FIRST\_MONDAY.
2. []()Using either TO\_CHAR (PostgreSQL) or MONTH, the CASE expression extracts the name of the current month from the result of FIRST\_MONDAY + 28.
3. The result from step two is compared to the value MTH from the inline view. The value MTH is the name of the current month as derived from CURRENT_ DATE. If the 2 month values match, then the month is large enough for you to need to add 28 days, and the CASE expression returns FIRST\_MONDAY + 28. If the two month values do not match, then you do not have room to add 28 days, and the CASE expression returns FIRST\_MONDAY + 21 days instead. It is convenient that our months are such that 28 and 21 are the only two possible values you need worry about adding.

###### Tip

You can extend the solution by adding 7 and 14 days to find the second and third Mondays of the month, respectively.[]()

# 9.7 Creating a Calendar

## Problem

[]()[]()You want to create a calendar for the current month. The calendar should be formatted like a calendar you might have on your desk: seven columns across and (usually) five rows down.

## Solution

Each solution will look a bit different, but they all solve the problem the same way: return each day for the current month, and then pivot on the day of the week for each week in the month to create a calendar.

There are different formats available for calendars. For example, the Unix CAL command formats the days from Sunday to Saturday. The examples in this recipe are based on ISO weeks, so the Monday through Friday format is the most convenient to generate. Once you become comfortable with the solutions, you’ll see that reformatting however you like is simply a matter of modifying the values assigned by the ISO week before pivoting.

###### Tip

As you begin to use different types of formatting with SQL to create readable output, you will notice your queries becoming longer. Don’t let those long queries intimidate you; the queries presented for this recipe are extremely simple once broken down and run piece by piece.

### DB2

Use the recursive WITH clause to return every day in the current month. Then pivot on the day of the week using CASE and MAX:

```
 1   with x(dy,dm,mth,dw,wk)
 2   as (
 3 select (current_date -day(current_date) day +1 day) dy,
 4         day((current_date -day(current_date) day +1 day)) dm,
 5         month(current_date) mth,
 6         dayofweek(current_date -day(current_date) day +1 day) dw,
 7         week_iso(current_date -day(current_date) day +1 day) wk
 8   from t1
 9  union all
10 select dy+1 day, day(dy+1 day), mth,
11         dayofweek(dy+1 day), week_iso(dy+1 day)
12   from x
13  where month(dy+1 day) = mth
14  )
15  select max(case dw when 2 then dm end) as Mo,
16         max(case dw when 3 then dm end) as Tu,
17         max(case dw when 4 then dm end) as We,
18         max(case dw when 5 then dm end) as Th,
19         max(case dw when 6 then dm end) as Fr,
20         max(case dw when 7 then dm end) as Sa,
21         max(case dw when 1 then dm end) as Su
22   from x
23  group by wk
24  order by wk
```

### Oracle

[]()Use the recursive CONNECT BY clause to return each day in the current month. Then pivot on the day of the week using CASE and MAX:

```
 1  with x
 2    as (
 3 select *
 4   from (
 5 select to_char(trunc(sysdate,'mm')+level-1,'iw') wk,
 6        to_char(trunc(sysdate,'mm')+level-1,'dd') dm,
 7        to_number(to_char(trunc(sysdate,'mm')+level-1,'d')) dw,
 8        to_char(trunc(sysdate,'mm')+level-1,'mm') curr_mth,
 9        to_char(sysdate,'mm') mth
10   from dual
11  connect by level <= 31
12        )
13  where curr_mth = mth
14 )
15 select max(case dw when 2 then dm end) Mo,
16        max(case dw when 3 then dm end) Tu,
17        max(case dw when 4 then dm end) We,
18        max(case dw when 5 then dm end) Th,
19        max(case dw when 6 then dm end) Fr,
20        max(case dw when 7 then dm end) Sa,
21        max(case dw when 1 then dm end) Su
22   from x
23  group by wk
24  order by wk
```

### PostgreSQL

[]()Use the function GENERATE\_SERIES to return every day in the current month. Then pivot on the day of the week using MAX and CASE:

```
 1 select max(case dw when 2 then dm end) as Mo,
 2        max(case dw when 3 then dm end) as Tu,
 3        max(case dw when 4 then dm end) as We,
 4        max(case dw when 5 then dm end) as Th,
 5        max(case dw when 6 then dm end) as Fr,
 6        max(case dw when 7 then dm end) as Sa,
 7        max(case dw when 1 then dm end) as Su
 8   from (
 9 select *
10   from (
11 select cast(date_trunc('month',current_date) as date)+x.id,
12        to_char(
13           cast(
14     date_trunc('month',current_date)
15                as date)+x.id,'iw') as wk,
16        to_char(
17           cast(
18     date_trunc('month',current_date)
19                as date)+x.id,'dd') as dm,
20        cast(
21     to_char(
22        cast(
23   date_trunc('month',current_date)
24                 as date)+x.id,'d') as integer) as dw,
25         to_char(
26            cast(
27     date_trunc('month',current_date)
28                 as date)+x.id,'mm') as curr_mth,
29         to_char(current_date,'mm') as mth
30   from generate_series (0,31) x(id)
31        ) x
32  where mth = curr_mth
33        ) y
34  group by wk
35  order by wk
```

### MySQL

Use a recursive CTE to return each day in the current month. Then pivot on the day of the week using MAX and CASE:

```
with recursive  x(dy,dm,mth,dw,wk)
      as (
  select dy,
         day(dy) dm,
         datepart(m,dy) mth,
         datepart(dw,dy) dw,
         case when datepart(dw,dy) = 1
              then datepart(ww,dy)-1
              else datepart(ww,dy)
        end wk
   from (
 select date_add(day,-day(getdate())+1,getdate()) dy
   from t1
        ) x
  union all
  select dateadd(d,1,dy), day(date_add(d,1,dy)), mth,
         datepart(dw,dateadd(d,1,dy)),
         case when datepart(dw,date_add(d,1,dy)) = 1
              then datepart(wk,date_add(d,1,dy))-1
              else datepart(wk,date_add(d,1,dy))
         end
    from x
   where datepart(m,date_add(d,1,dy)) = mth
 )
 select max(case dw when 2 then dm end) as Mo,
        max(case dw when 3 then dm end) as Tu,
        max(case dw when 4 then dm end) as We,
        max(case dw when 5 then dm end) as Th,
        max(case dw when 6 then dm end) as Fr,
        max(case dw when 7 then dm end) as Sa,
        max(case dw when 1 then dm end) as Su
   from x
  group by wk
  order by wk;
```

### SQL Server

[]()Use the recursive WITH clause to return every day in the current month. Then pivot on the day of the week using CASE and MAX:

```
 1   with x(dy,dm,mth,dw,wk)
 2     as (
 3 select dy,
 4        day(dy) dm,
 5        datepart(m,dy) mth,
 6        datepart(dw,dy) dw,
 7        case when datepart(dw,dy) = 1
 8             then datepart(ww,dy)-1
 9             else datepart(ww,dy)
10        end wk
11   from (
12 select dateadd(day,-day(getdate())+1,getdate()) dy
13   from t1
14        ) x
15  union all
16  select dateadd(d,1,dy), day(dateadd(d,1,dy)), mth,
17         datepart(dw,dateadd(d,1,dy)),
18         case when datepart(dw,dateadd(d,1,dy)) = 1
19              then datepart(wk,dateadd(d,1,dy))  -1
20              else datepart(wk,dateadd(d,1,dy))
21         end
22    from x
23   where datepart(m,dateadd(d,1,dy)) = mth
24 )
25 select max(case dw when 2 then dm end) as Mo,
26        max(case dw when 3 then dm end) as Tu,
27        max(case dw when 4 then dm end) as We,
28        max(case dw when 5 then dm end) as Th,
29        max(case dw when 6 then dm end) as Fr,
30        max(case dw when 7 then dm end) as Sa,
31        max(case dw when 1 then dm end) as Su
32   from x
33  group by wk
34  order by wk
```

## Discussion

### DB2

The first step is to return each day in the month for which you want to create a calendar. Do that using the recursive WITH clause. Along with each day of the month (DM), you will need to return different parts of each date: the day of the week (DW), the current month you are working with (MTH), and the ISO week for each day of the month (WK). The results of the recursive view X prior to recursion taking place (the upper portion of the UNION ALL) are shown here:

```
select (current_date -day(current_date) day +1 day) dy, 
        day((current_date -day(current_date) day +1 day)) dm, 
        month(current_date) mth, 
        dayofweek(current_date -day(current_date) day +1 day) dw, 
        week_iso(current_date -day(current_date) day +1 day) wk 
   from t1 

DY          DM MTH         DW WK
----------- -- --- ---------- --
01-JUN-2005 01  06          4 22
```

The next step is to repeatedly increase the value for DM (move through the days of the month) until you are no longer in the current month. As you move through each day in the month, you will also return the day of the week that each day is, and which ISO week the current day of the month falls into. Partial results are shown here:

```
with x(dy,dm,mth,dw,wk) 
   as ( 
 select (current_date -day(current_date) day +1 day) dy, 
        day((current_date -day(current_date) day +1 day)) dm, 
        month(current_date) mth, 
        dayofweek(current_date -day(current_date) day +1 day) dw, 
        week_iso(current_date -day(current_date) day +1 day) wk 
   from t1 
  union all 
  select dy+1 day, day(dy+1 day), mth, 
         dayofweek(dy+1 day), week_iso(dy+1 day) 
    from x 
   where month(dy+1 day) = mth 
 ) 
 select * 
   from x 

DY          DM MTH         DW WK
----------- -- --- ---------- --
01-JUN-2020 01 06           4 22
02-JUN-2020 02 06           5 22
…
21-JUN-2020 21 06           3 25
22-JUN-2020 22 06           4 25
…
30-JUN-2020 30 06           5 26
```

What you are returning at this point is: each day for the current month, the two-digit numeric day of the month, the two-digit numeric month, the one-digit day of the week (1–7 for Sun–Sat), and the two-digit ISO week each day falls into. With all this information available, you can use a CASE expression to determine which day of the week each value of DM (each day of the month) falls into. A portion of the results is shown here:

```
with x(dy,dm,mth,dw,wk) 
   as ( 
 select (current_date -day(current_date) day +1 day) dy, 
        day((current_date -day(current_date) day +1 day)) dm, 
        month(current_date) mth, 
        dayofweek(current_date -day(current_date) day +1 day) dw, 
        week_iso(current_date -day(current_date) day +1 day) wk 
   from t1 
  union all 
  select dy+1 day, day(dy+1 day), mth, 
         dayofweek(dy+1 day), week_iso(dy+1 day) 
    from x 
   where month(dy+1 day) = mth 
  ) 
  select wk, 
         case dw when 2 then dm end as Mo, 
         case dw when 3 then dm end as Tu, 
         case dw when 4 then dm end as We, 
         case dw when 5 then dm end as Th, 
         case dw when 6 then dm end as Fr, 
         case dw when 7 then dm end as Sa, 
         case dw when 1 then dm end as Su 
    from x 

WK MO TU WE TH FR SA SU
-- -- -- -- -- -- -- --
22       01
22          02
22             03
22                04
22                   05
23 06
23    07
23       08
23          09
23             10
23                11
23                   12
```

As you can see from the partial output, every day in each week is returned as a row. What you want to do now is to group the days by week, and then collapse all the days for each week into a single row. Use the aggregate function MAX, and group by WK (the ISO week) to return all the days for a week as one row. To properly format the calendar and ensure that the days are in the right order, order the results by WK. The final output is shown here:

```
with x(dy,dm,mth,dw,wk) 
   as ( 
 select (current_date -day(current_date) day +1 day) dy, 
        day((current_date -day(current_date) day +1 day)) dm, 
        month(current_date) mth, 
        dayofweek(current_date -day(current_date) day +1 day) dw, 
        week_iso(current_date -day(current_date) day +1 day) wk 
   from t1 
  union all 
  select dy+1 day, day(dy+1 day), mth, 
         dayofweek(dy+1 day), week_iso(dy+1 day) 
    from x 
   where month(dy+1 day) = mth 
 ) 
 select max(case dw when 2 then dm end) as Mo, 
        max(case dw when 3 then dm end) as Tu, 
        max(case dw when 4 then dm end) as We, 
        max(case dw when 5 then dm end) as Th, 
        max(case dw when 6 then dm end) as Fr, 
        max(case dw when 7 then dm end) as Sa, 
        max(case dw when 1 then dm end) as Su 
   from x 
  group by wk 
  order by wk 


MO TU WE TH FR SA SU
-- -- -- -- -- -- --
      01 02 03 04 05
06 07 08 09 10 11 12
13 14 15 16 17 18 19
20 21 22 23 24 25 26
27 28 29 30
```

### Oracle

Begin by using the recursive CONNECT BY clause to generate a row for each day in the month for which you want to generate a calendar. If you aren’t running at least Oracle9*i* Database, you can’t use CONNECT BY this way. Instead, you can use a recursive CTE, as used in the solution for the other platforms.

Along with each day of the month, you will need to return different bits of information for each day: the day of the month (DM), the day of the week (DW), the current month you are working with (MTH), and the ISO week for each day of the month (WK). The results of the WITH view X for the first day of the current month are shown here:

```
select trunc(sysdate,'mm') dy, 
        to_char(trunc(sysdate,'mm'),'dd') dm, 
        to_char(sysdate,'mm') mth, 
        to_number(to_char(trunc(sysdate,'mm'),'d')) dw, 
        to_char(trunc(sysdate,'mm'),'iw') wk 
   from dual 

DY          DM MT         DW WK
----------- -- -- ---------- --
01-JUN-2020 01 06          4 22
```

The next step is to repeatedly increase the value for DM (move through the days of the month) until you are no longer in the current month. As you move through each day in the month, you will also return the day of the week for each day and the ISO week into which the current day falls. Partial results are shown here (the full date for each day is added for readability):

```
with x 
   as ( 
 select * 
   from ( 
 select trunc(sysdate,'mm')+level-1 dy, 
        to_char(trunc(sysdate,'mm')+level-1,'iw') wk, 
        to_char(trunc(sysdate,'mm')+level-1,'dd') dm, 
        to_number(to_char(trunc(sysdate,'mm')+level-1,'d')) dw, 
        to_char(trunc(sysdate,'mm')+level-1,'mm') curr_mth, 
        to_char(sysdate,'mm') mth 
   from dual 
  connect by level <= 31 
        ) 
  where curr_mth = mth 
 ) 
 select * 
   from x 

DY          WK DM         DW CU MT
----------- -- -- ---------- -- --
01-JUN-2020 22 01          4 06 06
02-JUN-2020 22 02          5 06 06
…
21-JUN-2020 25 21          3 06 06
22-JUN-2020 25 22          4 06 06
…
30-JUN-2020 26 30          5 06 06
```

What you are returning at this point is one row for each day of the current month. In that row you have: the two-digit numeric day of the month, the two-digit numeric month, the one-digit day of the week (1–7 for Sun–Sat), and the two-digit ISO week number. With all this information available, you can use a CASE expression to determine which day of the week each value of DM (each day of the month) falls into. A portion of the results is shown here:

```
with x 
   as ( 
 select * 
   from ( 
 select trunc(sysdate,'mm')+level-1 dy, 
        to_char(trunc(sysdate,'mm')+level-1,'iw') wk, 
        to_char(trunc(sysdate,'mm')+level-1,'dd') dm, 
        to_number(to_char(trunc(sysdate,'mm')+level-1,'d')) dw, 
        to_char(trunc(sysdate,'mm')+level-1,'mm') curr_mth, 
        to_char(sysdate,'mm') mth 
   from dual 
  connect by level <= 31 
        ) 
  where curr_mth = mth 
 ) 
 select wk, 
        case dw when 2 then dm end as Mo, 
        case dw when 3 then dm end as Tu, 
        case dw when 4 then dm end as We, 
        case dw when 5 then dm end as Th, 
        case dw when 6 then dm end as Fr, 
        case dw when 7 then dm end as Sa, 
        case dw when 1 then dm end as Su 
   from x 

WK MO TU WE TH FR SA SU
-- -- -- -- -- -- -- --
22       01
22          02
22             03
22                04
22                  05
23 06
23    07
23       08
23          09
23             10
23               11
23                 12
```

As you can see from the partial output, every day in each week is returned as a row, but the day number is in one of seven columns corresponding to the day of the week. Your task now is to consolidate the days into one row for each week. Use the aggregate function MAX and group by WK (the ISO week) to return all the days for a week as one row. To ensure the days are in the right order, order the results by WK. The final output is shown here:

```
with x 
   as ( 
 select * 
   from ( 
 select to_char(trunc(sysdate,'mm')+level-1,'iw') wk, 
        to_char(trunc(sysdate,'mm')+level-1,'dd') dm, 
        to_number(to_char(trunc(sysdate,'mm')+level-1,'d')) dw, 
        to_char(trunc(sysdate,'mm')+level-1,'mm') curr_mth, 
        to_char(sysdate,'mm') mth 
   from dual 
  connect by level <= 31 
        ) 
  where curr_mth = mth 
 ) 
 select max(case dw when 2 then dm end) Mo, 
        max(case dw when 3 then dm end) Tu, 
        max(case dw when 4 then dm end) We, 
        max(case dw when 5 then dm end) Th, 
        max(case dw when 6 then dm end) Fr, 
        max(case dw when 7 then dm end) Sa, 
        max(case dw when 1 then dm end) Su 
   from x 
  group by wk 
  order by wk 

MO TU WE TH FR SA SU
-- -- -- -- -- -- --
      01 02 03 04 05
06 07 08 09 10 11 12
13 14 15 16 17 18 19
20 21 22 23 24 25 26
27 28 29 30
```

### MySQL, PostgreSQL, and SQL Server

These solutions are the same except for differences in the specific functions used to call dates. We arbitrarily use the SQL Serve solution for the explanation. Begin by returning one row for each day of the month. You can do that using the recursive WITH clause. For each row that you return, you will need the following items: the day of the month (DM), the day of the week (DW), the current month you are working with (MTH), and the ISO week for each day of the month (WK). The results of the recursive view X prior to recursion taking place (the upper portion of the UNION ALL) are shown here:

```
select dy, 
        day(dy) dm, 
        datepart(m,dy) mth, 
        datepart(dw,dy) dw, 
        case when datepart(dw,dy) = 1 
             then datepart(ww,dy)-1 
             else datepart(ww,dy) 
        end wk 
   from ( 
 select dateadd(day,-day(getdate())+1,getdate()) dy 
   from t1 
        ) x 

DY          DM MTH         DW WK
----------- -- --- ---------- --
01-JUN-2005  1   6          4 23
```

Your next step is to repeatedly increase the value for DM (move through the days of the month) until you are no longer in the current month. As you move through each day in the month, you will also return the day of the week and the ISO week number. Partial results are shown here:

```
  with x(dy,dm,mth,dw,wk) 
     as ( 
 select dy, 
        day(dy) dm, 
        datepart(m,dy) mth, 
        datepart(dw,dy) dw, 
        case when datepart(dw,dy) = 1 
             then datepart(ww,dy)-1 
             else datepart(ww,dy) 
        end wk 
   from ( 
 select dateadd(day,-day(getdate())+1,getdate()) dy 
   from t1 
        ) x 
  union all 
  select dateadd(d,1,dy), day(dateadd(d,1,dy)), mth, 
         datepart(dw,dateadd(d,1,dy)), 
         case when datepart(dw,dateadd(d,1,dy)) = 1 
              then datepart(wk,dateadd(d,1,dy))-1 
              else datepart(wk,dateadd(d,1,dy)) 
         end 
   from x 
  where datepart(m,dateadd(d,1,dy)) = mth 
 ) 
 select * 
   from x 

DY          DM MTH         DW WK
----------- -- --- ---------- --
01-JUN-2005 01 06           4 23
02-JUN-2005 02 06           5 23
…
21-JUN-2005 21 06           3 26
22-JUN-2005 22 06           4 26
…
30-JUN-2005 30 06           5 27
```

For each day in the current month, you now have: the two-digit numeric day of the month, the two-digit numeric month, the one-digit day of the week (1–7 for Sun– Sat), and the two-digit ISO week number.

Now, use a CASE expression to determine which day of the week each value of DM (each day of the month) falls into. A portion of the results is shown here:

```
  with x(dy,dm,mth,dw,wk) 
     as ( 
 select dy, 
        day(dy) dm, 
        datepart(m,dy) mth, 
        datepart(dw,dy) dw, 
        case when datepart(dw,dy) = 1 
             then datepart(ww,dy)-1 
             else datepart(ww,dy) 
        end wk 
   from ( 
 select dateadd(day,-day(getdate())+1,getdate()) dy 
   from t1 
        ) x 
  union all 
  select dateadd(d,1,dy), day(dateadd(d,1,dy)), mth, 
         datepart(dw,dateadd(d,1,dy)), 
         case when datepart(dw,dateadd(d,1,dy)) = 1 
              then datepart(wk,dateadd(d,1,dy))-1 
              else datepart(wk,dateadd(d,1,dy)) 
         end 
    from x 
   where datepart(m,dateadd(d,1,dy)) = mth 
 ) 
 select case dw when 2 then dm end as Mo, 
        case dw when 3 then dm end as Tu, 
        case dw when 4 then dm end as We, 
        case dw when 5 then dm end as Th, 
        case dw when 6 then dm end as Fr, 
        case dw when 7 then dm end as Sa, 
        case dw when 1 then dm end as Su 
   from x 

WK MO TU WE TH FR SA SU
-- -- -- -- -- -- -- --
22       01
22          02
22             03
22                04
22                   05
23 06
23    07
23       08
23          09
23             10
23                11
23                   12
```

Every day in each week is returned as a separate row. In each row, the column containing the day number corresponds to the day of the week. You now need to consolidate the days for each week into one row. Do that by grouping the rows by WK (the ISO week) and applying the MAX function to the different columns. The results will be in calendar format as shown here:[]()[]()

```
with x(dy,dm,mth,dw,wk) 
     as ( 
 select dy, 
        day(dy) dm, 
        datepart(m,dy) mth, 
        datepart(dw,dy) dw, 
        case when datepart(dw,dy) = 1 
             then datepart(ww,dy)-1 
             else datepart(ww,dy) 
        end wk 
   from ( 
 select dateadd(day,-day(getdate())+1,getdate()) dy 
   from t1 
        ) x 
  union all 
  select dateadd(d,1,dy), day(dateadd(d,1,dy)), mth, 
         datepart(dw,dateadd(d,1,dy)), 
         case when datepart(dw,dateadd(d,1,dy)) = 1 
              then datepart(wk,dateadd(d,1,dy))-1 
              else datepart(wk,dateadd(d,1,dy)) 
         end 
    from x 
   where datepart(m,dateadd(d,1,dy)) = mth 
 ) 
 select max(case dw when 2 then dm end) as Mo, 
        max(case dw when 3 then dm end) as Tu, 
        max(case dw when 4 then dm end) as We, 
        max(case dw when 5 then dm end) as Th, 
        max(case dw when 6 then dm end) as Fr, 
        max(case dw when 7 then dm end) as Sa, 
        max(case dw when 1 then dm end) as Su 
   from x 
  group by wk 
  order by wk 

MO TU WE TH FR SA SU
-- -- -- -- -- -- --
      01 02 03 04 05
06 07 08 09 10 11 12
13 14 15 16 17 18 19
20 21 22 23 24 25 26
27 28 29 30
```

# 9.8 Listing Quarter Start and End Dates for the Year

## Problem

[]()You want to return the start and end dates for each of the four quarters of a given year.

## Solution

There are four quarters to a year, so you know you will need to generate four rows. After generating the desired number of rows, simply use the date functions supplied by your RDBMS to return to the quarter the start and end dates fall into. Your goal is to produce the following result set (one again, the choice to use the current year is arbitrary):

```
QTR Q_START     Q_END
--- ----------- -----------
  1 01-JAN-2020 31-MAR-2020
  2 01-APR-2020 30-JUN-2020
  3 01-JUL-2020 30-SEP-2020
  4 01-OCT-2020 31-DEC-2020
```

### DB2

Use table EMP and the window function ROW\_NUMBER OVER to generate four rows. Alternatively, you can use the WITH clause to generate rows (as many of the recipes do), or you can query against any table with at least four rows. The following solution uses the ROW\_NUMBER OVER approach:

```
 1 select quarter(dy-1 day) QTR,
 2        dy-3 month Q_start,
 3        dy-1 day Q_end
 4   from (
 5 select (current_date -
 6          (dayofyear(current_date)-1) day
 7            + (rn*3) month) dy
 8   from (
 9 select row_number()over() rn
10   from emp
11  fetch first 4 rows only
12         ) x
13         ) y
```

### Oracle

[]()Use the function ADD\_MONTHS to find the start and end dates for each quarter. Use ROWNUM to represent the quarter the start and end dates belong to. The following solution uses table EMP to generate four rows:

```
1 select rownum qtr,
2        add_months(trunc(sysdate,'y'),(rownum-1)*3) q_start,
3        add_months(trunc(sysdate,'y'),rownum*3)-1 q_end
4   from emp
5  where rownum <= 4
```

### PostgreSQL

Find the first day of the year based on the current date, and use a recursive CTE to fill in the first date of the remaining three quarters before finding the last day of each quarter:

```
  with recursive x (dy,cnt)
     as (
  select
        current_date -cast(extract(day from current_date)as integer) +1 dy
        , id
    from t1
   union all
  select cast(dy  + interval '3 months' as date) , cnt+1
    from x
   where cnt+1 <= 4
 )
 select  cast(dy - interval '3 months' as date) as Q_start
        , dy-1 as Q_end
		from x
```

### MySQL

Find the first day of the year from the current day, and use a CTE to create four rows, one for each quarter. Use ADDDATE to find the last day of each quarter (three months after the previous last day, or the first day of the quarter minus one):

```
1	      with recursive x (dy,cnt)
2     as (
3	         select
4         adddate(current_date,(-dayofyear(current_date))+1) dy
5           ,id
6	     from t1
7	   union all
8	         select adddate(dy, interval 3 month ), cnt+1
9	         from x
10         where cnt+1 <= 4
11        )
12
13       select quarter(adddate(dy,-1)) QTR
14    ,  date_add(dy, interval -3 month) Q_start
15    ,  adddate(dy,-1)  Q_end
16     from x
17     order by 1;
```

### SQL Server

Use the recursive WITH clause to generate four rows. Use the function DATEADD to find the start and end dates. []()Use the function DATEPART to determine which quarter the start and end dates belong to:

```
 1  with x (dy,cnt)
 2    as (
 3 select dateadd(d,-(datepart(dy,getdate())-1),getdate()),
 4        1
 5   from t1
 6  union all
 7 select dateadd(m,3,dy), cnt+1
 8   from x
 9  where cnt+1 <= 4
10 )
11 select datepart(q,dateadd(d,-1,dy)) QTR,
12         dateadd(m,-3,dy) Q_start,
13        dateadd(d,-1,dy) Q_end
14   from x
15 order by 1
```

## Discussion

### DB2

The first step is to generate four rows (with values one through four) for each quarter in the year. []()Inline view X uses the window function ROW\_NUMBER OVER and the FETCH FIRST clause to return only four rows from EMP. The results are shown here:

```
select row_number()over() rn 
   from emp 
  fetch first 4 rows only 

RN
--
 1
 2
 3
 4
```

The next step is to find the first day of the year, then add *n* months to it, where *n* is three times RN (you are adding 3, 6, 9, and 12 months to the first day of the year). The results are shown here:

```
select (current_date 
         (dayofyear(current_date)-1) day 
           + (rn*3) month) dy  
   from ( 
 select row_number()over() rn 
   from emp 
  fetch first 4 rows only 
        ) x 

DY
-----------
01-APR-2005
01-JUL-2005
01-OCT-2005
01-JAN-2005
```

At this point, the values for DY are one day after the end date for each quarter. The next step is to get the start and end dates for each quarter. Subtract one day from DY to get the end of each quarter, and subtract three months from DY to get the start of each quarter. []()Use the QUARTER function on DY-1 (the end date for each quarter) to determine which quarter the start and end dates belong to.

### Oracle

[]()[]()[]()The combination of ROWNUM, TRUNC, and ADD\_MONTHS makes this solution easy. To find the start of each quarter, simply add *n* months to the first day of the year, where *n* is (ROWNUM-1)\*3 (giving you 0, 3, 6, 9). To find the end of each quarter, add *n* months to the first day of the year, where *n* is ROWNUM\*3, and subtract one day. As an aside, when working with quarters, you may also find it useful to use TO\_CHAR and/or TRUNC with the Q formatting option.

### PostgreSQL, MySQL, and SQL Server

Like some of the previous recipes, this recipe uses the same structure across three RDMS implementations, but different syntax for the date operations. The first step is to find the first day of the year and then recursively add *n* months, where *n* is three times the current iteration (there are four iterations, therefore, you are adding 3\*1 months, 3\*2 months, etc.), using the DATEADD function or its equivalent. The results are shown here:

```
with x (dy,cnt) 
    as ( 
 select dateadd(d,-(datepart(dy,getdate())-1),getdate()), 
        1 
   from t1 
  union all 
 select dateadd(m,3,dy), cnt+1 
   from x 
  where cnt+1 <= 4 
 ) 
 select dy 
   from x 

DY
-----------
01-APR-2020
01-JUL-2020
01-OCT-2020
01-JAN-2020
```

The values for DY are one day after the end of each quarter. To get the end of each quarter, simply subtract one day from DY by using the DATEADD function. To find the start of each quarter, use the DATEADD function to subtract three months from DY. Use the []()DATEPART function on the end date for each quarter to determine which quarter the start and end dates belong to or its equivalent. If you are using PostgreSQL, note that you need CAST to ensure data types align after performing adding the three months to the start date, or the data types will different, and the UNION ALL in the recursive CTE will fail.[]()

# 9.9 Determining Quarter Start and End Dates for a Given Quarter

## Problem

[]()When given a year and quarter in the format of YYYYQ (four-digit year, one-digit quarter), you want to return the quarter’s start and end dates.

## Solution

The key to this solution is to find the quarter by using the modulus function on the YYYYQ value. (As an alternative to modulo, since the year format is four digits, you can simply substring out the last digit to get the quarter.) Once you have the quarter, simply multiply by three to get the ending month for the quarter. In the solutions that follow, inline view X will return all four year and quarter combinations. The result set for inline view X is as follows:

```
select 20051 as yrq from t1 union all 
 select 20052 as yrq from t1 union all 
 select 20053 as yrq from t1 union all 
 select 20054 as yrq from t1 
    YRQ
-------
  20051
  20052
  20053
  20054
```

### DB2

Use the function SUBSTR to return the year from inline view X. Use the MOD function to determine which quarter you are looking for:

```
 1 select (q_end-2 month) q_start,
 2        (q_end+1 month)-1 day q_end
 3   from (
 4 select date(substr(cast(yrq as char(4)),1,4) ||'-'||
 5        rtrim(cast(mod(yrq,10)*3 as char(2))) ||'-1') q_end
 6   from (
 7 select 20051 yrq from t1 union all
 8 select 20052 yrq from t1 union all
 9 select 20053 yrq from t1 union all
10 select 20054 yrq from t1
11        ) x
12        ) y
```

### Oracle

Use the function SUBSTR to return the year from inline view X. Use the MOD function to determine which quarter you are looking for:

```
 1 select add_months(q_end,-2) q_start,
 2        last_day(q_end) q_end
 3   from (
 4 select to_date(substr(yrq,1,4)||mod(yrq,10)*3,'yyyymm') q_end
 5   from (
 6 select 20051 yrq from dual union all
 7 select 20052 yrq from dual union all
 8 select 20053 yrq from dual union all
 9 select 20054 yrq from dual
10        ) x
11        ) y
```

### PostgreSQL

Use the function SUBSTR to return the year from the inline view X. Use the MOD function to determine which quarter you are looking for:

```
 1 select date(q_end-(2*interval '1 month')) as q_start,
 2        date(q_end+interval '1 month'-interval '1 day') as q_end
 3   from (
 4 select to_date(substr(yrq,1,4)||mod(yrq,10)*3,'yyyymm') as q_end
 5   from (
 6 select 20051 as yrq from t1 union all
 7 select 20052 as yrq from t1 union all
 8 select 20053 as yrq from t1 union all
 9 select 20054 as yrq from t1
10        ) x
11        ) y
```

### MySQL

Use the function SUBSTR to return the year from the inline view X. Use the MOD function to determine which quarter you are looking for:

```
 1 select date_add(
 2         adddate(q_end,-day(q_end)+1),
 3                interval -2 month) q_start,
 4        q_end
 5   from (
 6 select last_day(
 7     str_to_date(
 8          concat(
 9          substr(yrq,1,4),mod(yrq,10)*3),'%Y%m')) q_end
10   from (
11 select 20051 as yrq from t1 union all
12 select 20052 as yrq from t1 union all
13 select 20053 as yrq from t1 union all
14 select 20054 as yrq from t1
15        ) x
16        ) y
```

### SQL Server

[]()[]()[]()Use the function SUBSTRING to return the year from the inline view X. Use the modulus function (%) to determine which quarter you are looking for:

```
 1 select dateadd(m,-2,q_end) q_start,
 2        dateadd(d,-1,dateadd(m,1,q_end)) q_end
 3   from (
 4 select cast(substring(cast(yrq as varchar),1,4)+'-'+
 5        cast(yrq%10*3 as varchar)+'-1' as datetime) q_end
 6   from (
 7 select 20051 as yrq from t1 union all
 8 select 20052 as yrq from t1 union all
 9 select 20052 as yrq from t1 union all
10 select 20054 as yrq from t1
11        ) x
12        ) y
```

## Discussion

### DB2

The first step is to find the year and quarter you are working with. Substring out the year from inline view X (X.YRQ) using the SUBSTR function. To get the quarter, use modulus 10 on YRQ. Once you have the quarter, multiply by three to get the end month for the quarter. The results are shown here:

```
select substr(cast(yrq as char(4)),1,4) yr, 
        mod(yrq,10)*3 mth 
   from ( 
 select 20051 yrq from t1 union all 
 select 20052 yrq from t1 union all 
 select 20053 yrq from t1 union all 
 select 20054 yrq from t1 
        ) x 

YR      MTH
---- ------
2005      3
2005      6
2005      9
2005     12
```

At this point you have the year and end month for each quarter. Use those values to construct a date, specifically, the first day of the last month for each quarter. Use the concatenation operator || to glue together the year and month, and then use the []()DATE function to convert to a date:

```
select date(substr(cast(yrq as char(4)),1,4) ||'-'|| 
        rtrim(cast(mod(yrq,10)*3 as char(2))) ||'-1') q_end 
   from ( 
 select 20051 yrq from t1 union all 
 select 20052 yrq from t1 union all 
 select 20053 yrq from t1 union all 
 select 20054 yrq from t1 
        ) x 

Q_END
-----------
01-MAR-2005
01-JUN-2005
01-SEP-2005
01-DEC-2005
```

The values for Q\_END are the first day of the last month of each quarter. To get to the last day of the month, add one month to Q\_END and then subtract one day. To find the start date for each quarter, subtract two months from Q\_END.

### Oracle

The first step is to find the year and quarter you are working with. Substring out the year from inline view X (X.YRQ) using the SUBSTR function. To get the quarter, use modulus 10 on YRQ. Once you have the quarter, multiply by three to get the end month for the quarter. The results are shown here:

```
select substr(yrq,1,4) yr, mod(yrq,10)*3 mth 
   from ( 
 select 20051 yrq from t1 union all 
 select 20052 yrq from t1 union all 
 select 20053 yrq from t1 union all 
 select 20054 yrq from t1 
        ) x 

YR      MTH
---- ------
2005      3
2005      6
2005      9
2005     12
```

At this point you have the year and end month for each quarter. Use those values to construct a date, specifically, the first day of the last month for each quarter. Use the concatenation operator || to glue together the year and month, and then use the TO\_DATE function to convert to a date:

```
select to_date(substr(yrq,1,4)||mod(yrq,10)*3,'yyyymm') q_end 
   from ( 
 select 20051 yrq from t1 union all 
 select 20052 yrq from t1 union all 
 select 20053 yrq from t1 union all 
 select 20054 yrq from t1 
        ) x 

Q_END
-----------
01-MAR-2005
01-JUN-2005
01-SEP-2005
01-DEC-2005
```

The values for Q\_END are the first day of the last month of each quarter. To get to the last day of the month, use the LAST\_DAY function on Q\_END. To find the start date for each quarter, subtract two months from Q\_END using the ADD\_MONTHS function.

### PostgreSQL

The first step is to find the year and quarter you are working with. Substring out the year from inline view X (X.YRQ) using the SUBSTR function. To get the quarter, use modulus 10 on YRQ. Once you have the quarter, multiply by 3 to get the end month for the quarter. The results are shown here:

```
select substr(cast(yrq as varchar),1,4) yr, mod(yrq,10)*3 mth 
   from ( 
 select 20051 yrq from t1 union all 
 select 20052 yrq from t1 union all 
 select 20053 yrq from t1 union all 
 select 20054 yrq from t1 
        ) x 

YR        MTH
----  -------
2005        3
2005        6
2005        9
2005       12
```

At this point, you have the year and end month for each quarter. Use those values to construct a date, specifically, the first day of the last month for each quarter. Use the concatenation operator || to glue together the year and month, and then use the TO_ DATE function to convert to a date:

```
select to_date(substr(cast(yrq as varchar),1,4)||mod(yrq,10)*3,'yyyymm') q_end 
   from ( 
 select 20051 yrq from t1 union all 
 select 20052 yrq from t1 union all 
 select 20053 yrq from t1 union all 
 select 20054 yrq from t1 
        ) x 

Q_END
-----------
01-MAR-2005
01-JUN-2005
01-SEP-2005
01-DEC-2005
```

The values for Q\_END are the first day of the last month of each quarter. To get to the last day of the month, add one month to Q\_END and subtract one day. To find the start date for each quarter, subtract two months from Q\_END. Cast the final result as dates.

### MySQL

The first step is to find the year and quarter you are working with. Substring out the year from inline view X (X.YRQ) using the SUBSTR function. To get the quarter, use modulus 10 on YRQ. Once you have the quarter, multiply by three to get the end month for the quarter. The results are shown here:

```
select substr(cast(yrq as varchar),1,4) yr, mod(yrq,10)*3 mth 
   from ( 
 select 20051 yrq from t1 union all 
 select 20052 yrq from t1 union all 
 select 20053 yrq from t1 union all 
 select 20054 yrq from t1 
        ) x 

YR      MTH
---- ------
2005      3
2005      6
2005      9
2005     12
```

At this point, you have the year and end month for each quarter. Use those values to construct a date, specifically, the last day of each quarter. Use the CONCAT function to glue together the year and month, and then use the []()STR\_TO\_DATE function to convert to a date. Use the LAST\_DAY function to find the last day for each quarter:

```
select last_day( 
     str_to_date( 
          concat( 
          substr(yrq,1,4),mod(yrq,10)*3),'%Y%m')) q_end 
   from ( 
 select 20051 as yrq from t1 union all 
 select 20052 as yrq from t1 union all 
 select 20053 as yrq from t1 union all 
 select 20054 as yrq from t1 
        ) x 

Q_END
-----------
31-MAR-2005
30-JUN-2005
30-SEP-2005
31-DEC-2005
```

Because you already have the end of each quarter, all that’s left is to find the start date for each quarter. Use the DAY function to return the day of the month the end of each quarter falls on, and subtract that from Q\_END using the []()ADDDATE function to give you the end of the prior month; add one day to bring you to the first day of the last month of each quarter. The last step is to use the DATE\_ADD function to subtract two months from the first day of the last month of each quarter to get you to the start date for each quarter.

### SQL Server

The first step is to find the year and quarter you are working with. Substring out the year from inline view X (X.YRQ) using the []()SUBSTRING function. To get the quarter, use modulus 10 on YRQ. Once you have the quarter, multiply by three to get the end month for the quarter. The results are shown here:

```
select substring(yrq,1,4) yr, yrq%10*3 mth 
   from ( 
 select 20051 yrq from t1 union all 
 select 20052 yrq from t1 union all 
 select 20053 yrq from t1 union all 
 select 20054 yrq from t1 
        ) x 

YR        MTH
----   ------
2005        3
2005        6
2005        9
2005       12
```

At this point, you have the year and end month for each quarter. Use those values to construct a date, specifically, the first day of the last month for each quarter. Use the concatenation operator + to glue together the year and month, and then use the []()CAST function to convert to a date:

```
select cast(substring(cast(yrq as varchar),1,4)+'-'+ 
        cast(yrq%10*3 as varchar)+'-1' as datetime) q_end 
   from ( 
 select 20051 yrq from t1 union all 
 select 20052 yrq from t1 union all 
 select 20053 yrq from t1 union all 
 select 20054 yrq from t1 
        ) x 

Q_END
-----------
01-MAR-2005
01-JUN-2005
01-SEP-2005
01-DEC-2005
```

The values for Q\_END are the first day of the last month of each quarter. To get to the last day of the month, add one month to Q\_END and subtract one day using the DATEADD function. To find the start date for each quarter, subtract two months from Q\_END using the DATEADD function.[]()

# 9.10 Filling in Missing Dates

## Problem

[]()You need to generate a row for every date (or every month, week, or year) within a given range. Such rowsets are often used to generate summary reports. For example, you want to count the number of employees hired every month of every year in which any employee has been hired. Examining the dates of all the employees hired, there have been hirings from 2000 to 2003:

```
select distinct 
        extract(year from hiredate) as year 
   from emp 

YEAR
-----
 2000
 2001
 2002
 2003
```

You want to determine the number of employees hired each month from 2000 to 2003. A portion of the desired result set is shown here:

```
MTH          NUM_HIRED
----------- ----------
01-JAN-2001          0
01-FEB-2001          2
01-MAR-2001          0
01-APR-2001          1
01-MAY-2001          1
01-JUN-2001          1
01-JUL-2001          0
01-AUG-2001          0
01-SEP-2001          2
01-OCT-2001          0
01-NOV-2001          1
01-DEC-2001          2
```

## Solution

The trick here is that you want to return a row for each month even if no employee was hired (i.e., the count would be zero). Because there isn’t an employee hired every month between 2000 and 2003, you must generate those months yourself and then outer join to table EMP on HIREDATE (truncating the actual HIREDATE to its month so it can match the generated months when possible).

### DB2

Use the recursive WITH clause to generate every month (the first day of each month from January 1, 2000, to December 1, 2003). Once you have all the months for the required range of dates, outer join to table EMP and use the aggregate function COUNT to count the number of hires for each month:

```
 1   with x (start_date,end_date)
 2     as (
 3 select (min(hiredate)
 4          dayofyear(min(hiredate)) day +1 day) start_date,
 5        (max(hiredate)
 6          dayofyear(max(hiredate)) day +1 day) +1 year end_date
 7   from emp
 8  union all
 9 select start_date +1 month, end_date
10   from x
11  where (start_date +1 month) < end_date
12 )
13 select x.start_date mth, count(e.hiredate) num_hired
14   from x left join emp e
15     on (x.start_date = (e.hiredate-(day(hiredate)-1) day))
16  group by x.start_date
17  order by 1
```

### Oracle

[]()Use the CONNECT BY clause to generate each month between 2000 and 2003. []()Then outer join to table EMP and use the aggregate function COUNT to count the number of employees hired in each month:

```
 1   with x
 2     as (
 3 select add_months(start_date,level-1) start_date
 4   from (
 5 select min(trunc(hiredate,'y')) start_date,
 6        add_months(max(trunc(hiredate,'y')),12) end_date
 7   from emp
 8        )
 9  connect by level <= months_between(end_date,start_date)
10 )
11 select x.start_date MTH, count(e.hiredate) num_hired
12   from x left join emp e
13     on (x.start_date = trunc(e.hiredate,'mm'))
14  group by x.start_date
15  order by 1
```

### PostgreSQL

Use CTE to fill in the months since the earliest hire and then LEFT OUTER JOIN on the EMP table using the month and year of each generated month to enable the COUNT of the number of hiredates in each period:

```
	with recursive x (start_date, end_date)
as
(
    select
    cast(min(hiredate) - (cast(extract(day from min(hiredate))
    as integer) - 1) as date)
    , max(hiredate)
    from emp
  union all
    select cast(start_date + interval '1 month' as date)
    , end_date
    from x
    where start_date < end_date
 )

 select x.start_date,count(hiredate)
 from x left join emp
  on (extract(month from start_date) =
	            extract(month from emp.hiredate)
	  and extract(year from start_date)
	  = extract(year from emp.hiredate))
	 group by x.start_date
	 order by 1
```

### MySQL

Use a recursive CTE to generate each month between the start and end dates, and then check for hires by using an outer join to table EMP:

```
 with recursive x (start_date,end_date)
	     as
	    (
      select
          adddate(min(hiredate),
          -dayofyear(min(hiredate))+1)  start_date
          ,adddate(max(hiredate),
          -dayofyear(max(hiredate))+1)  end_date
          from emp
       union all
          select date_add(start_date,interval 1 month)
          , end_date
          from x
          where date_add(start_date, interval 1 month) < end_date
      )

       select x.start_date mth, count(e.hiredate) num_hired
	      from x left join emp e
	      on (extract(year_month from start_date)
	          =
	          extract(year_month from e.hiredate))
	      group by x.start_date
	      order by 1;
```

### SQL Server

Use the recursive WITH clause to generate every month (the first day of each month from January 1, 2000, to December 1, 2003). Once you have all the months for the required range of dates, outer join to table EMP and use the aggregate function COUNT to count the number of hires for each month:

```
1   with x (start_date,end_date)
2     as (
3 select (min(hiredate) -
4          datepart(dy,min(hiredate))+1) start_date,
5        dateadd(yy,1,
6         (max(hiredate) -
7          datepart(dy,max(hiredate))+1)) end_date
8   from emp
9  union all
10 select dateadd(mm,1,start_date), end_date
11   from x
12  where dateadd(mm,1,start_date) < end_date
13 )
14 select x.start_date mth, count(e.hiredate) num_hired
15   from x left join emp e
16     on (x.start_date =
17            dateadd(dd,-day(e.hiredate)+1,e.hiredate))
18 group by x.start_date
19 order by 1
```

## Discussion

### DB2

The first step is to generate every month (actually the first day of each month) from 2000 to 2003. []()Start using the DAYOFYEAR function on the MIN and MAX HIREDATEs to find the boundary months:

```
select (min(hiredate) 
          dayofyear(min(hiredate)) day +1 day) start_date, 
        (max(hiredate) 
          dayofyear(max(hiredate)) day +1 day) +1 year end_date 
   from emp 

START_DATE  END_DATE
----------- -----------
01-JAN-2000 01-JAN-2004
```

Your next step is to repeatedly add months to START\_DATE to return all the months necessary for the final result set. The value for END\_DATE is one day more than it should be. This is OK. As you recursively add months to START\_DATE, you can stop before you hit END\_DATE. A portion of the months created is shown here:

```
with x (start_date,end_date) 
   as ( 
 select (min(hiredate) 
          dayofyear(min(hiredate)) day +1 day) start_date, 
        (max(hiredate) 
          dayofyear(max(hiredate)) day +1 day) +1 year end_date 
   from emp 
  union all 
 select start_date +1 month, end_date 
   from x 
  where (start_date +1 month) < end_date 
 ) 
 select * 
   from x 

START_DATE  END_DATE
----------- -----------
01-JAN-2000 01-JAN-2004
01-FEB-2000 01-JAN-2004
01-MAR-2000 01-JAN-2004
…
01-OCT-2003 01-JAN-2004
01-NOV-2003 01-JAN-2004
01-DEC-2003 01-JAN-2004
```

At this point, you have all the months you need, and you can simply outer join to EMP.HIREDATE. Because the day for each START\_DATE is the first of the month, truncate EMP.HIREDATE to the first day of its month. Finally, use the aggregate function COUNT on EMP.HIREDATE.

### Oracle

The first step is to generate the first day of every for every month from 2000 to 2003. []()Start by using TRUNC and ADD\_MONTHS together with the MIN and MAX HIREDATE values to find the boundary months:

```
select min(trunc(hiredate,'y')) start_date, 
        add_months(max(trunc(hiredate,'y')),12) end_date 
   from emp 

START_DATE  END_DATE
----------- -----------
01-JAN-2000 01-JAN-2004
```

Then repeatedly add months to START\_DATE to return all the months necessary for the final result set. The value for END\_DATE is one day more than it should be, which is OK. As you recursively add months to START\_DATE, you can stop before you hit END\_DATE. A portion of the months created is shown here:

```
with x as ( 
 select add_months(start_date,level-1) start_date 
   from ( 
 select min(trunc(hiredate,'y')) start_date, 
        add_months(max(trunc(hiredate,'y')),12) end_date 
   from emp 
        ) 
  connect by level <= months_between(end_date,start_date) 
 ) 
 select * 
   from x 

START_DATE
-----------
01-JAN-2000
01-FEB-2000
01-MAR-2000
…
01-OCT-2003
01-NOV-2003
01-DEC-2003
```

At this point, you have all the months you need, and you can simply outer join to EMP.HIREDATE. Because the day for each START\_DATE is the first of the month, truncate EMP.HIREDATE to the first day of the month it is in. The final step is to use the aggregate function COUNT on EMP.HIREDATE.

### PostgreSQL

This solution uses a CTE to generate the months you need and is similar to the subsequent solutions for MySQL and SQL Server. The first step is to create the boundary dates using aggregate functions. You could simply find earliest and latest hire dates using the MIN() and MAX() functions, but the output makes more sense if you find the first day of the month containing the earliest hire date.

### MySQL

[]()[]()First, find the boundary dates by using the aggregate functions MIN and MAX along with the DAYOFYEAR and ADDDATE functions. The result set shown here is from inline view X:

```
with recursive x (start_date,end_date)
	     as (
          select
           adddate(min(hiredate),
          -dayofyear(min(hiredate))+1)  start_date
          ,adddate(max(hiredate),
          -dayofyear(max(hiredate))+1)  end_date
          from emp
       union all
       select date_add(start_date,interval 1 month)
       , end_date
       from x
       where date_add(start_date, interval 1 month) < end_date
         )
 select * from x
```

```
	select adddate(min(hiredate),-dayofyear(min(hiredate))+1) min_hd, 
 	       adddate(max(hiredate),-dayofyear(max(hiredate))+1) max_hd 
 	  from emp 

	MIN_HD      MAX_HD
	----------- -----------
	01-JAN-2000 01-JAN-2003
```

Next, increment MAX\_HD to the last month of the year by the CTE:

```
MTH
-----------
01-JAN-2000
01-FEB-2000
01-MAR-2000
…
01-OCT-2003
01-NOV-2003
01-DEC-2003
```

Now that you have all the months you need for the final result set, outer join to EMP.HIREDATE (be sure to truncate EMP.HIREDATE to the first day of the month) and use the aggregate function COUNT on EMP.HIREDATE to count the number of hires in each month.

### SQL Server

Begin by generating every month (actually, the first day of each month) from 2000 to 2003. Then find the boundary months by applying the DAYOFYEAR function to the MIN and MAX HIREDATEs:

```
select (min(hiredate) - 
          datepart(dy,min(hiredate))+1) start_date, 
        dateadd(yy,1, 
         (max(hiredate) - 
          datepart(dy,max(hiredate))+1)) end_date 
   from emp 

START_DATE  END_DATE
----------- -----------
01-JAN-2000 01-JAN-2004
```

Your next step is to repeatedly add months to START\_DATE to return all the months necessary for the final result set. The value for END\_DATE is one day more than it should be, which is OK, as you can stop recursively adding months to START\_DATE before you hit END\_DATE. A portion of the months created is shown here:

```
with x (start_date,end_date) 
   as ( 
 select (min(hiredate) - 
          datepart(dy,min(hiredate))+1) start_date, 
        dateadd(yy,1, 
         (max(hiredate) - 
          datepart(dy,max(hiredate))+1)) end_date 
   from emp 
  union all 
 select dateadd(mm,1,start_date), end_date 
   from x 
  where dateadd(mm,1,start_date) < end_date 
 ) 
 select * 
   from x 

START_DATE  END_DATE
----------- -----------
01-JAN-2000 01-JAN-2004
01-FEB-2000 01-JAN-2004
01-MAR-2000 01-JAN-2004
…
01-OCT-2003 01-JAN-2004
01-NOV-2003 01-JAN-2004
01-DEC-2003 01-JAN-2004
```

At this point, you have all the months you need. Simply outer join to EMP.HIREDATE. Because the day for each START\_DATE is the first of the month, truncate EMP.HIREDATE to the first day of the month. The final step is to use the aggregate function COUNT on EMP.HIREDATE.[]()[]()

# 9.11 Searching on Specific Units of Time

## Problem

[]()You want to search for dates that match a given month, day of the week, or some other unit of time. For example, you want to find all employees hired in February or December, as well as employees hired on a Tuesday.

## Solution

Use the functions supplied by your RDBMS to find month and weekday names for dates. This particular recipe can be useful in various places. Consider, if you wanted to search HIREDATEs but wanted to ignore the year by extracting the month (or any other part of the HIREDATE you are interested in), you can do so. The example solutions to this problem search by month and weekday name. By studying the date formatting functions provided by your RDBMS, you can easily modify these solutions to search by year, quarter, combination of year and quarter, month and year combination, etc.

### DB2 and MySQL

[]()[]()Use the functions MONTHNAME and DAYNAME to find the name of the month and weekday an employee was hired, respectively:

```
1 select ename
2   from emp
3 where monthname(hiredate) in ('February','December')
4    or dayname(hiredate) = 'Tuesday'
```

### Oracle and PostgreSQL

Use the function TO\_CHAR to find the names of the month and weekday an employee was hired. []()Use the function RTRIM to remove trailing whitespaces:

```
1 select ename
2   from emp
3 where rtrim(to_char(hiredate,'month')) in ('february','december')
4    or rtrim(to_char(hiredate,'day')) = 'tuesday'
```

### SQL Server

[]()Use the function DATENAME to find the names of the month and weekday an employee was hired:

```
1 select ename
2   from emp
3 where datename(m,hiredate) in ('February','December')
4    or datename(dw,hiredate) = 'Tuesday'
```

## Discussion

The key to each solution is simply knowing which functions to use and how to use them. To verify what the return values are, put the functions in the SELECT clause and examine the output. Listed here is the result set for employees in DEPTNO 10 (using SQL Server syntax):

```
select ename,datename(m,hiredate) mth,datename(dw,hiredate) dw 
   from emp 
  where deptno = 10 

ENAME   MTH        DW
------  ---------  -----------
CLARK   June       Tuesday
KING    November   Tuesday
MILLER  January    Saturday
```

Once you know what the function(s) return, finding rows using the functions shown in each of the solutions is easy.[]()

# 9.12 Comparing Records Using Specific Parts of a Date

## Problem

[]()You want to find which employees have been hired on the same month and weekday. For example, if an employee was hired on Monday, March 10, 2008, and another employee was hired on Monday, March 2, 2001, you want those two to come up as a match since the day of week and month match. In table EMP, only three employees meet this requirement. You want to return the following result set:

```
MSG
------------------------------------------------------
JAMES was hired on the same month and weekday as FORD
SCOTT was hired on the same month and weekday as JAMES
SCOTT was hired on the same month and weekday as FORD
```

## Solution

Because you want to compare one employee’s HIREDATE with the HIREDATE of the other employees, you will need to self-join table EMP. That makes each possible combination of HIREDATEs available for you to compare. Then, simply extract the weekday and month from each HIREDATE and compare.

### DB2

[]()After self-joining table EMP, use the function DAYOFWEEK to return the numeric day of the week. []()Use the function MONTHNAME to return the name of the month:

```
1 select a.ename ||
2        ' was hired on the same month and weekday as '||
3        b.ename msg
4   from emp a, emp b
5 where (dayofweek(a.hiredate),monthname(a.hiredate)) =
6       (dayofweek(b.hiredate),monthname(b.hiredate))
7   and a.empno < b.empno
8 order by a.ename
```

### Oracle and PostgreSQL

After self-joining table EMP, use the TO\_CHAR function to format the HIREDATE into weekday and month for comparison:

```
1 select a.ename ||
2        ' was hired on the same month and weekday as '||
3        b.ename as msg
4   from emp a, emp b
5 where to_char(a.hiredate,'DMON') =
6       to_char(b.hiredate,'DMON')
7   and a.empno < b.empno
8 order by a.ename
```

### MySQL

[]()After self-joining table EMP, use the DATE\_FORMAT function to format the HIREDATE into weekday and month for comparison:

```
1 select concat(a.ename,
2        ' was hired on the same month and weekday as ',
3        b.ename) msg
4   from emp a, emp b
5  where date_format(a.hiredate,'%w%M') =
6        date_format(b.hiredate,'%w%M')
7    and a.empno < b.empno
8 order by a.ename
```

### SQL Server

[]()After self-joining table EMP, use the DATENAME function to format the HIREDATE into weekday and month for comparison:

```
1 select a.ename +
2        ' was hired on the same month and weekday as '+
3        b.ename msg
4  from emp a, emp b
5 where datename(dw,a.hiredate) = datename(dw,b.hiredate)
6   and datename(m,a.hiredate) = datename(m,b.hiredate)
7   and a.empno < b.empno
8 order by a.ename
```

## Discussion

The only difference between the solutions is the date function used to format the HIREDATE. We’ll use the Oracle/PostgreSQL solution in this discussion (because it’s the shortest to type out), but the explanation holds true for the other solutions as well.

The first step is to self-join EMP so that each employee has access to the other employees’ HIREDATEs. Consider the results of the query shown here (filtered for SCOTT):

```
select a.ename as scott, a.hiredate as scott_hd, 
        b.ename as other_emps, b.hiredate as other_hds 
   from emp a, emp b 
  where a.ename = 'SCOTT' 
   and a.empno != b.empno 

SCOTT      SCOTT_HD    OTHER_EMPS OTHER_HDS
---------- ----------- ---------- -----------
SCOTT      09-DEC-2002 SMITH      17-DEC-2000
SCOTT      09-DEC-2002 ALLEN      20-FEB-2001
SCOTT      09-DEC-2002 WARD       22-FEB-2001
SCOTT      09-DEC-2002 JONES      02-APR-2001
SCOTT      09-DEC-2002 MARTIN     28-SEP-2001
SCOTT      09-DEC-2002 BLAKE      01-MAY-2001
SCOTT      09-DEC-2002 CLARK      09-JUN-2001
SCOTT      09-DEC-2002 KING       17-NOV-2001
SCOTT      09-DEC-2002 TURNER     08-SEP-2001
SCOTT      09-DEC-2002 ADAMS      12-JAN-2003
SCOTT      09-DEC-2002 JAMES      03-DEC-2001
SCOTT      09-DEC-2002 FORD       03-DEC-2001
SCOTT      09-DEC-2002 MILLER     23-JAN-2002
```

By self-joining table EMP, you can compare SCOTT’s HIREDATE to the HIREDATE of all the other employees. The filter on EMPNO is so that SCOTT’s HIREDATE is not returned as one of the OTHER\_HDS. The next step is to use your RDBMS’s supplied date formatting function(s) to compare the weekday and month of the HIREDATEs and keep only those that match:

```
select a.ename as emp1, a.hiredate as emp1_hd, 
        b.ename as emp2, b.hiredate as emp2_hd 
   from emp a, emp b 
  where to_char(a.hiredate,'DMON') = 
        to_char(b.hiredate,'DMON') 
    and a.empno != b.empno 
  order by 1 

EMP1       EMP1_HD     EMP2       EMP2_HD
---------- ----------- ---------- -----------
FORD       03-DEC-2001 SCOTT      09-DEC-2002
FORD       03-DEC-2001 JAMES      03-DEC-2001
JAMES      03-DEC-2001 SCOTT      09-DEC-2002
JAMES      03-DEC-2001 FORD       03-DEC-2001

SCOTT      09-DEC-2002 JAMES      03-DEC-2001
SCOTT      09-DEC-2002 FORD       03-DEC-2001
```

At this point, the HIREDATEs are correctly matched, but there are six rows in the result set rather than the three in the “Problem” section of this recipe. The reason for the extra rows is the filter on EMPNO. By using “not equals,” you do not filter out the reciprocals. For example, the first row matches FORD and SCOTT, and the last row matches SCOTT and FORD. The six rows in the result set are technically accurate but redundant. To remove the redundancy, use “less than” (the HIREDATEs are removed to bring the intermediate queries closer to the final result set):

```
select a.ename as emp1, b.ename as emp2 
   from emp a, emp b 
  where to_char(a.hiredate,'DMON') = 
        to_char(b.hiredate,'DMON') 
    and a.empno < b.empno 
  order by 1 

EMP1       EMP2
---------- ----------
JAMES      FORD
SCOTT      JAMES
SCOTT      FORD
```

The final step is to simply concatenate the result set to form the message.[]()

# 9.13 Identifying Overlapping Date Ranges

## Problem

[]()You want to find all instances of an employee starting a new project before ending an existing project. Consider table EMP\_PROJECT:

```
select * 
   from emp_project 

EMPNO ENAME      PROJ_ID PROJ_START  PROJ_END
----- ---------- ------- ----------- -----------
7782  CLARK            1 16-JUN-2005 18-JUN-2005
7782  CLARK            4 19-JUN-2005 24-JUN-2005
7782  CLARK            7 22-JUN-2005 25-JUN-2005
7782  CLARK           10 25-JUN-2005 28-JUN-2005
7782  CLARK           13 28-JUN-2005 02-JUL-2005
7839  KING             2 17-JUN-2005 21-JUN-2005
7839  KING             8 23-JUN-2005 25-JUN-2005
7839  KING            14 29-JUN-2005 30-JUN-2005
7839  KING            11 26-JUN-2005 27-JUN-2005
7839  KING             5 20-JUN-2005 24-JUN-2005
7934  MILLER           3 18-JUN-2005 22-JUN-2005
7934  MILLER          12 27-JUN-2005 28-JUN-2005
7934  MILLER          15 30-JUN-2005 03-JUL-2005
7934  MILLER           9 24-JUN-2005 27-JUN-2005
7934  MILLER           6 21-JUN-2005 23-JUN-2005
```

Looking at the results for employee KING, you see that KING began PROJ\_ID 8 before finishing PROJ\_ID 5 and began PROJ\_ID 5 before finishing PROJ\_ID 2. You want to return the following result set:

```
EMPNO ENAME      MSG
----- ---------- --------------------------------
7782  CLARK      project 7 overlaps project 4
7782  CLARK      project 10 overlaps project 7
7782  CLARK      project 13 overlaps project 10
7839  KING       project 8 overlaps project 5
7839  KING       project 5 overlaps project 2
7934  MILLER     project 12 overlaps project 9
7934  MILLER     project 6 overlaps project 3
```

## Solution

The key here is to find rows where PROJ\_START (the date the new project starts) occurs on or after another project’s PROJ\_START date and on or before that other project’s PROJ\_END date. To begin, you need to be able to compare each project with each other project (for the same employee). By self-joining EMP\_PROJECT on employee, you generate every possible combination of two projects for each employee. To find the overlaps, simply find the rows where PROJ\_START for any PROJ\_ID falls between PROJ\_START and PROJ\_END for another PROJ\_ID by the same employee.

### DB2, PostgreSQL, and Oracle

Self-join EMP\_PROJECT. []()[]()Then use the concatenation operator || to construct the message that explains which projects overlap:

```
1 select a.empno,a.ename,
2        'project '||b.proj_id||
3        ' overlaps project '||a.proj_id as msg
4   from emp_project a,
5        emp_project b
6  where a.empno = b.empno
7    and b.proj_start >= a.proj_start
8    and b.proj_start <= a.proj_end
9    and a.proj_id != b.proj_id
```

### MySQL

Self-join EMP\_PROJECT. []()Then use the CONCAT function to construct the message that explains which projects overlap:

```
1 select a.empno,a.ename,
2        concat('project ',b.proj_id,
3         ' overlaps project ',a.proj_id) as msg
4   from emp_project a,
5        emp_project b
6  where a.empno = b.empno
7    and b.proj_start >= a.proj_start
8    and b.proj_start <= a.proj_end
9    and a.proj_id != b.proj_id
```

### SQL Server

Self-join EMP\_PROJECT. []()[]()Then use the concatenation operator + to construct the message that explains which projects overlap:

```
1 select a.empno,a.ename,
2        'project '+b.proj_id+
3        ' overlaps project '+a.proj_id as msg
4   from emp_project a,
5        emp_project b
6  where a.empno = b.empno
7    and b.proj_start >= a.proj_start
8    and b.proj_start <= a.proj_end
9    and a.proj_id != b.proj_id
```

## Discussion

The only difference between the solutions lies in the string concatenation, so one discussion using the DB2 syntax will cover all three solutions. []()[]()The first step is a self-join of EMP\_PROJECT so that the PROJ\_START dates can be compared among the different projects. The output of the self-join for employee KING is shown here. You can observe how each project can “see” the other projects:

```
select a.ename, 
        a.proj_id as a_id, 
        a.proj_start as a_start, 
        a.proj_end as a_end, 
        b.proj_id as b_id, 
        b.proj_start as b_start 
   from emp_project a, 
        emp_project b 
  where a.ename = 'KING' 
    and a.empno = b.empno 
    and a.proj_id != b.proj_id 
 order by 2 

ENAME  A_ID  A_START     A_END       B_ID  B_START
------ ----- ----------- ----------- ----- -----------
KING       2 17-JUN-2005 21-JUN-2005     8 23-JUN-2005
KING       2 17-JUN-2005 21-JUN-2005    14 29-JUN-2005
KING       2 17-JUN-2005 21-JUN-2005    11 26-JUN-2005
KING       2 17-JUN-2005 21-JUN-2005     5 20-JUN-2005
KING       5 20-JUN-2005 24-JUN-2005     2 17-JUN-2005
KING       5 20-JUN-2005 24-JUN-2005     8 23-JUN-2005
KING       5 20-JUN-2005 24-JUN-2005    11 26-JUN-2005
KING       5 20-JUN-2005 24-JUN-2005    14 29-JUN-2005
KING       8 23-JUN-2005 25-JUN-2005     2 17-JUN-2005
KING       8 23-JUN-2005 25-JUN-2005    14 29-JUN-2005
KING       8 23-JUN-2005 25-JUN-2005     5 20-JUN-2005
KING       8 23-JUN-2005 25-JUN-2005    11 26-JUN-2005
KING      11 26-JUN-2005 27-JUN-2005     2 17-JUN-2005
KING      11 26-JUN-2005 27-JUN-2005     8 23-JUN-2005
KING      11 26-JUN-2005 27-JUN-2005    14 29-JUN-2005
KING      11 26-JUN-2005 27-JUN-2005     5 20-JUN-2005
KING      14 29-JUN-2005 30-JUN-2005     2 17-JUN-2005
KING      14 29-JUN-2005 30-JUN-2005     8 23-JUN-2005
KING      14 29-JUN-2005 30-JUN-2005     5 20-JUN-2005
KING      14 29-JUN-2005 30-JUN-2005    11 26-JUN-2005
```

As you can see from the result set, the self-join makes finding overlapping dates easy: simply return each row where B\_START occurs between A\_START and A\_END. If you look at the WHERE clause on lines 7 and 8 of the solution:

```
and b.proj_start >= a.proj_start
and b.proj_start <= a.proj_end
```

it is doing just that. Once you have the required rows, constructing the messages is just a matter of concatenating the return values.

[]()[]()Oracle users can use the window function LEAD OVER to avoid the self-join, if the maximum number of projects per employee is fixed. This can come in handy if the self-join is expensive for your particular results (if the self-join requires more resources than the sorts needed for LEAD OVER). For example, consider the alternative for employee KING using LEAD OVER:

```
select empno, 
        ename, 
        proj_id, 
        proj_start, 
        proj_end, 
        case 
        when lead(proj_start,1)over(order by proj_start) 
             between proj_start and proj_end 
        then lead(proj_id)over(order by proj_start) 
        when lead(proj_start,2)over(order by proj_start) 
             between proj_start and proj_end 
        then lead(proj_id)over(order by proj_start) 
        when lead(proj_start,3)over(order by proj_start) 
             between proj_start and proj_end 
        then lead(proj_id)over(order by proj_start) 
        when lead(proj_start,4)over(order by proj_start) 
             between proj_start and proj_end 
        then lead(proj_id)over(order by proj_start) 
        end is_overlap 
   from emp_project 
  where ename = 'KING' 

EMPNO ENAME  PROJ_ID PROJ_START  PROJ_END    IS_OVERLAP
----- ------ ------- ----------- ----------- ----------
7839  KING         2 17-JUN-2005 21-JUN-2005          5
7839  KING         5 20-JUN-2005 24-JUN-2005          8
7839  KING         8 23-JUN-2005 25-JUN-2005
7839  KING        11 26-JUN-2005 27-JUN-2005
7839  KING        14 29-JUN-2005 30-JUN-2005
```

Because the number of projects is fixed at five for employee KING, you can use LEAD OVER to examine the dates of all the projects without a self-join. From here, producing the final result set is easy. Simply keep the rows where IS\_OVERLAP is not NULL:

```
select empno,ename, 
        'project '||is_overlap|| 
        ' overlaps project '||proj_id msg 
   from ( 
 select empno, 
        ename, 
        proj_id, 
        proj_start, 
        proj_end, 
        case 
        when lead(proj_start,1)over(order by proj_start) 
             between proj_start and proj_end 
        then lead(proj_id)over(order by proj_start) 
        when lead(proj_start,2)over(order by proj_start) 
             between proj_start and proj_end 
        then lead(proj_id)over(order by proj_start) 
        when lead(proj_start,3)over(order by proj_start) 
             between proj_start and proj_end 
        then lead(proj_id)over(order by proj_start) 
        when lead(proj_start,4)over(order by proj_start) 
             between proj_start and proj_end 
        then lead(proj_id)over(order by proj_start) 
        end is_overlap 
   from emp_project 
  where ename = 'KING' 
        ) 
  where is_overlap is not null 

EMPNO ENAME  MSG
----- ------ --------------------------------
7839  KING   project 5 overlaps project 2
7839  KING   project 8 overlaps project 5
```

To allow the solution to work for all employees (not just KING), partition by ENAME in the LEAD OVER function[]():[]()

```
select empno,ename, 
        'project '||is_overlap|| 
        ' overlaps project '||proj_id msg 
   from ( 
 select empno, 
        ename, 
        proj_id, 
        proj_start, 
        proj_end, 
        case 
        when lead(proj_start,1)over(partition by ename 
                                        order by proj_start) 
             between proj_start and proj_end 
        then lead(proj_id)over(partition by ename 
                                   order by proj_start) 
        when lead(proj_start,2)over(partition by ename 
                                        order by proj_start) 
             between proj_start and proj_end 
        then lead(proj_id)over(partition by ename 
                                   order by proj_start) 
        when lead(proj_start,3)over(partition by ename 
                                        order by proj_start) 
             between proj_start and proj_end 
        then lead(proj_id)over(partition by ename 
                                   order by proj_start) 
        when lead(proj_start,4)over(partition by ename 
                                        order by proj_start) 
             between proj_start and proj_end 
        then lead(proj_id)over(partition by ename 
                                   order by proj_start) 
        end is_overlap 
  from emp_project 
       ) 
 where is_overlap is not null 

EMPNO ENAME  MSG
----- ------ -------------------------------
7782  CLARK  project 7 overlaps project 4
7782  CLARK  project 10 overlaps project 7
7782  CLARK  project 13 overlaps project 10
7839  KING   project 5 overlaps project 2
7839  KING   project 8 overlaps project 5
7934  MILLER project 6 overlaps project 3
7934  MILLER project 12 overlaps project 9
```

# 9.14 Summing Up

Date manipulations are a common problem for anyone querying a database—a series of events stored with their dates inspires business users to ask creative date-based questions. At the same time, dates are one of the less standardized areas of SQLs between vendors. We hope that you take away from this chapter an idea of how even when the syntax is different, there is still a common logic that can be applied to queries that use dates.[]()