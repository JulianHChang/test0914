# Chapter 10. Working with Ranges

[]()This chapter is about “everyday” queries that involve ranges. Ranges are common in everyday life. For example, projects that we work on range over consecutive periods of time. In SQL, it’s often necessary to search for ranges, or to generate ranges, or to otherwise manipulate range-based data. The queries you’ll read about here are slightly more involved than the queries found in the preceding chapters, but they are just as common, and they’ll begin to give you a sense of what SQL can really do for you when you learn to take full advantage of it.

# 10.1 Locating a Range of Consecutive Values

## Problem

[]()You want to determine which rows represent a range of consecutive projects. Consider the following result set from view V, which contains data about a project and its start and end dates:

```
select *
  from V

PROJ_ID PROJ_START  PROJ_END
------- ----------- -----------
      1 01-JAN-2020 02-JAN-2020
      2 02-JAN-2020 03-JAN-2020
      3 03-JAN-2020 04-JAN-2020
      4 04-JAN-2020 05-JAN-2020
      5 06-JAN-2020 07-JAN-2020
      6 16-JAN-2020 17-JAN-2020
      7 17-JAN-2020 18-JAN-2020
      8 18-JAN-2020 19-JAN-2020
      9 19-JAN-2020 20-JAN-2020
     10 21-JAN-2020 22-JAN-2020
     11 26-JAN-2020 27-JAN-2020
     12 27-JAN-2020 28-JAN-2020
     13 28-JAN-2020 29-JAN-2020
     14 29-JAN-2020 30-JAN-2020
```

Excluding the first row, each row’s PROJ\_START should equal the PROJ\_END of the row before it (“before” is defined as PROJ\_ID–1 for the current row). Examining the first five rows from view V, PROJ\_IDs 1 through 3 are part of the same “group” as each PROJ\_END equals the PROJ\_START of the row after it. Because you want to find the range of dates for consecutive projects, you would like to return all rows where the current PROJ\_END equals the next row’s PROJ\_START. If the first five rows comprised the entire result set, you would like to return only the first three rows. The final result set (using all 14 rows from view V) should be:

```
PROJ_ID PROJ_START  PROJ_END
------- ----------- -----------
     1  01-JAN-2020 02-JAN-2020
     2  02-JAN-2020 03-JAN-2020
     3  03-JAN-2020 04-JAN-2020
     6  16-JAN-2020 17-JAN-2020
     7  17-JAN-2020 18-JAN-2020
     8  18-JAN-2020 19-JAN-2020
    11  26-JAN-2020 27-JAN-2020
    12  27-JAN-2020 28-JAN-2020
    13  28-JAN-2020 29-JAN-2020
```

The rows with PROJ\_IDs 4, 5, 9, 10, and 14 are excluded from this result set because the PROJ\_END of each of these rows does not match the PROJ\_START of the row following it.

## Solution

[]()[]()This solution takes best advantage of the window function LEAD OVER to look at the “next” row’s BEGIN\_DATE, thus avoiding the need to self-join, which was necessary before window functions were widely introduced:

```
1 select proj_id, proj_start, proj_end
2   from (
3 select proj_id, proj_start, proj_end,
4        lead(proj_start)over(order by proj_id) next_proj_start
5   from V
6        ) alias
7 where next_proj_start = proj_end
```

## Discussion

### DB2, MySQL, PostgreSQL, SQL Server, and Oracle

Although it is possible to develop a solution using a self-join, the window function LEAD OVER is perfect for this type of problem, and more intuitive. The function LEAD OVER allows you to examine other rows without performing a self-join (though the function must impose order on the result set to do so). Consider the results of the inline view (lines 3–5) for IDs 1 and 4:

```
select * 
   from ( 
 select proj_id, proj_start, proj_end, 
        lead(proj_start)over(order by proj_id) next_proj_start 
   from v 
        ) 
  where proj_id in ( 1, 4 ) 

PROJ_ID PROJ_START  PROJ_END    NEXT_PROJ_START
------- ----------- ----------- ---------------
      1 01-JAN-2020 02-JAN-2020 02-JAN-2020
      4 04-JAN-2020 05-JAN-2020 06-JAN-2020
```

Examining this snippet of code and its result set, it is particularly easy to see why PROJ\_ID 4 is excluded from the final result set of the complete solution. It’s excluded because its PROJ\_END date of 05-JAN-2020 does not match the “next” project’s start date of 06-JAN-2020.

[]()The function LEAD OVER is extremely handy when it comes to problems such as this one, particularly when examining partial results. []()When working with window functions, keep in mind that they are evaluated after the FROM and WHERE clauses, so the LEAD OVER function in the preceding query must be embedded within an inline view. Otherwise, the LEAD OVER function is applied to the result set after the WHERE clause has filtered out all rows except for PROJ\_ID’s 1 and 4.

Now, depending on how you view the data, you may very well want to include PROJ\_ID 4 in the final result set. Consider the first five rows from view V:

```
select * 
   from V 
  where proj_id <= 5 

PROJ_ID PROJ_START  PROJ_END
------- ----------- -----------
      1 01-JAN-2020 02-JAN-2020
      2 02-JAN-2020 03-JAN-2020
      3 03-JAN-2020 04-JAN-2020
      4 04-JAN-2020 05-JAN-2020
      5 06-JAN-2020 07-JAN-2020
```

If your requirement is such that PROJ\_ID 4 is in fact contiguous (because PROJ_ START for PROJ\_ID 4 matches PROJ\_END for PROJ\_ID 3), and that only PROJ_ ID 5 should be discarded, the proposed solution for this recipe is incorrect (!) or, at the very least, incomplete:

```
select proj_id, proj_start, proj_end 
   from ( 
 select proj_id, proj_start, proj_end,  
        lead(proj_start)over(order by proj_id) next_start 
   from V 
 where proj_id <= 5 
       ) 
 where proj_end = next_start 

PROJ_ID PROJ_START  PROJ_END
------- ----------- -----------
      1 01-JAN-2020 02-JAN-2020
      2 02-JAN-2020 03-JAN-2020
      3 03-JAN-2020 04-JAN-2020
```

[]()If you believe PROJ\_ID 4 should be included, simply add LAG OVER to the query and use an additional filter in the WHERE clause:

```
select proj_id, proj_start, proj_end 
   from ( 
 select proj_id, proj_start, proj_end,  
        lead(proj_start)over(order by proj_id) next_start, 
        lag(proj_end)over(order by proj_id) last_end 
   from V 
 where proj_id <= 5 
       ) 
 where proj_end = next_start 
    or proj_start = last_end 

PROJ_ID PROJ_START  PROJ_END
------- ----------- -----------
      1 01-JAN-2020 02-JAN-2020
      2 02-JAN-2020 03-JAN-2020
      3 03-JAN-2020 04-JAN-2020
      4 04-JAN-2020 05-JAN-2020
```

Now PROJ\_ID 4 is included in the final result set, and only the evil PROJ\_ID 5 is excluded. Please consider your exact requirements when applying these recipes to your code.[]()

# 10.2 Finding Differences Between Rows in the Same Group or Partition

## Problem

[]()You want to return the DEPTNO, ENAME, and SAL of each employee along with the difference in SAL between employees in the same department (i.e., having the same value for DEPTNO). The difference should be between each current employee and the employee hired immediately afterward (you want to see if there is a correlation between seniority and salary on a “per department” basis). For each employee hired last in his department, return “N/A” for the difference. The result set should look like this:

```
DEPTNO ENAME             SAL HIREDATE    DIFF
------ ---------- ---------- ----------- ----------
    10 CLARK            2450 09-JUN-2006      -2550
    10 KING             5000 17-NOV-2006       3700
    10 MILLER           1300 23-JAN-2007        N/A
    20 SMITH             800 17-DEC-2005      -2175
    20 JONES            2975 02-APR-2006        -25
    20 FORD             3000 03-DEC-2006          0
    20 SCOTT            3000 09-DEC-2007       1900
    20 ADAMS            1100 12-JAN-2008        N/A
    30 ALLEN            1600 20-FEB-2006        350
    30 WARD             1250 22-FEB-2006      -1600
    30 BLAKE            2850 01-MAY-2006       1350
    30 TURNER           1500 08-SEP-2006        250
    30 MARTIN           1250 28-SEP-2006        300
    30 JAMES             950 03-DEC-2006        N/A
```

## Solution

[]()The is another example of where the window functions LEAD OVER and LAG OVER come in handy. You can easily access next and prior rows without additional joins. Alternative methods such as subqueries or self-joins are possible but awkward:

```
1  with next_sal_tab (deptno,ename,sal,hiredate,next_sal)
2  as
3  (select deptno, ename, sal, hiredate,
4        lead(sal)over(partition by deptno
5                          order by hiredate) as next_sal
6   from emp )
7
8     select deptno, ename, sal, hiredate
9  ,    coalesce(cast(sal-next_sal as char), 'N/A') as diff
10    from next_sal_tab
```

In this case, for the sake of variety, we have used a CTE rather than a subquery—both will work across most RDBMSs these days, with the preference usually relating to readability.

### Discussion

The first step is to use the LEAD OVER window function to find the “next” salary for each employee within their department. The employees hired last in each department will have a NULL value for NEXT\_SAL:

```
select deptno,ename,sal,hiredate, 
        lead(sal)over(partition by deptno order by hiredate) as next_sal 
   from emp 

DEPTNO ENAME             SAL HIREDATE      NEXT_SAL
------ ---------- ---------- ----------- ----------
    10 CLARK            2450 09-JUN-2006       5000
    10 KING             5000 17-NOV-2006       1300
    10 MILLER           1300 23-JAN-2007
    20 SMITH             800 17-DEC-2005       2975
    20 JONES            2975 02-APR-2006       3000
    20 FORD             3000 03-DEC-2006       3000
    20 SCOTT            3000 09-DEC-2007       1100
    20 ADAMS            1100 12-JAN-2008
    30 ALLEN            1600 20-FEB-2006       1250
    30 WARD             1250 22-FEB-2006       2850
    30 BLAKE            2850 01-MAY-2006       1500
    30 TURNER           1500 08-SEP-2006       1250
    30 MARTIN           1250 28-SEP-2006        950
    30 JAMES             950 03-DEC-2006
```

The next step is to take the difference between each employee’s salary and the salary of the employee hired immediately after them in the same department:

```
select deptno,ename,sal,hiredate, sal-next_sal diff 
   from ( 
 select deptno,ename,sal,hiredate, 
        lead(sal)over(partition by deptno order by hiredate) next_sal 
   from emp 
        ) 

DEPTNO ENAME             SAL HIREDATE          DIFF
------ ---------- ---------- ----------- ----------
    10 CLARK            2450 09-JUN-2006      -2550
    10 KING             5000 17-NOV-2006       3700
    10 MILLER           1300 23-JAN-2007
    20 SMITH             800 17-DEC-2005      -2175
    20 JONES            2975 02-APR-2006        -25
    20 FORD             3000 03-DEC-2006          0
    20 SCOTT            3000 09-DEC-2007       1900
    20 ADAMS            1100 12-JAN-2008
    30 ALLEN            1600 20-FEB-2006        350
    30 WARD             1250 22-FEB-2006      -1600
    30 BLAKE            2850 01-MAY-2006       1350
    30 TURNER           1500 08-SEP-2006        250
    30 MARTIN           1250 28-SEP-2006        300
    30 JAMES             950 03-DEC-2006
```

[]()The next step is to use the COALESCE function to insert “N/A” when there is no next salary. To be able to return “N/A” you must cast the value of DIFF to a string:

```
select deptno,ename,sal,hiredate, 
        nvl(to_char(sal-next_sal),'N/A') diff 
   from ( 
 select deptno,ename,sal,hiredate, 
        lead(sal)over(partition by deptno order by hiredate) next_sal 
   from emp 
        ) 

DEPTNO ENAME             SAL HIREDATE    DIFF
------ ---------- ---------- ----------- ---------------
    10 CLARK            2450 09-JUN-2006 -2550
    10 KING             5000 17-NOV-2006 3700
    10 MILLER           1300 23-JAN-2007 N/A
    20 SMITH             800 17-DEC-2005 -2175
    20 JONES            2975 02-APR-2006 -25
    20 FORD             3000 03-DEC-2006 0
    20 SCOTT            3000 09-DEC-2007 1900
    20 ADAMS            1100 12-JAN-2008 N/A
    30 ALLEN            1600 20-FEB-2006 350
    30 WARD             1250 22-FEB-2006 -1600
    30 BLAKE            2850 01-MAY-2006 1350
    30 TURNER           1500 08-SEP-2006 250
    30 MARTIN           1250 28-SEP-2006 300
    30 JAMES             950 03-DEC-2006 N/A
```

[]()While the majority of the solutions provided in this book do not deal with “what if” scenarios (for the sake of readability and the author’s sanity), the scenario involving duplicates when using the LEAD OVER function in this manner must be discussed. In the simple sample data in table EMP, no employees have duplicate HIREDATEs, yet this is an unlikely situation. Normally, we would not discuss a “what if” situation such as duplicates (since there aren’t any in table EMP), but the workaround involving LEAD may not be immediately obvious. Consider the following query, which returns the difference in SAL between the employees in DEPTNO 10 (the difference is performed in the order in which they were hired):

```
select deptno,ename,sal,hiredate, 
        lpad(nvl(to_char(sal-next_sal),'N/A'),10) diff 
   from ( 
 select deptno,ename,sal,hiredate, 
        lead(sal)over(partition by deptno 
                          order by hiredate) next_sal 
   from emp 
  where deptno=10 and empno > 10 
        ) 

DEPTNO ENAME    SAL HIREDATE    DIFF
------ ------ ----- ----------- ----------
    10 CLARK   2450 09-JUN-2006      -2550
    10 KING    5000 17-NOV-2006       3700
    10 MILLER  1300 23-JAN-2007        N/A
```

This solution is correct considering the data in table EMP, but if there were duplicate rows, the solution would fail. Consider the following example, which shows four more employees hired on the same day as KING:

```
insert into emp (empno,ename,deptno,sal,hiredate) 
 values (1,'ant',10,1000,to_date('17-NOV-2006')) 

 insert into emp (empno,ename,deptno,sal,hiredate) 
 values (2,'joe',10,1500,to_date('17-NOV-2006')) 

 insert into emp (empno,ename,deptno,sal,hiredate) 
 values (3,'jim',10,1600,to_date('17-NOV-2006')) 

 insert into emp (empno,ename,deptno,sal,hiredate) 
 values (4,'jon',10,1700,to_date('17-NOV-2006')) 

 select deptno,ename,sal,hiredate, 
        lpad(nvl(to_char(sal-next_sal),'N/A'),10) diff 
   from ( 
 select deptno,ename,sal,hiredate, 
        lead(sal)over(partition by deptno 
                          order by hiredate) next_sal 
   from emp 
  where deptno=10 
        ) 

DEPTNO ENAME    SAL HIREDATE    DIFF
------ ------ ----- ----------- ----------
    10 CLARK   2450 09-JUN-2006       1450
    10 ant     1000 17-NOV-2006       -500
    10 joe     1500 17-NOV-2006      -3500
    10 KING    5000 17-NOV-2006       3400
    10 jim     1600 17-NOV-2006       -100
    10 jon     1700 17-NOV-2006        400
    10 MILLER  1300 23-JAN-2007        N/A
```

You’ll notice that with the exception of employee JON, all employees hired on the same date (November 17) evaluate their salary against another employee hired on the same date! This is incorrect. All employees hired on November 17 should have the difference of salary computed against MILLER’s salary, not another employee hired on November 17. Take, for example, employee ANT. The value for DIFF for ANT is –500 because ANT’s SAL is compared with JOE’s SAL and is 500 less than JOE’s SAL, hence the value of –500. The correct value for DIFF for employee ANT should be –300 because ANT makes 300 less than MILLER, who is the next employee hired by HIREDATE. The reason the solution seems to not work is due to the default behavior of Oracle’s []()[]()LEAD OVER function. By default, LEAD OVER looks ahead only one row. So, for employee ANT, the next SAL based on HIREDATE is JOE’s SAL, because LEAD OVER simply looks one row ahead and doesn’t skip duplicates. Fortunately, Oracle planned for such a situation and allows you to pass an additional parameter to LEAD OVER to determine how far ahead it should look. In the previous example, the solution is simply a matter of counting: find the distance from each employee hired on November 17 to January 23 (MILLER’s HIREDATE). The following shows how to accomplish this:

```
select deptno,ename,sal,hiredate, 
        lpad(nvl(to_char(sal-next_sal),'N/A'),10) diff 
   from ( 
 select deptno,ename,sal,hiredate, 
        lead(sal,cnt-rn+1)over(partition by deptno 
                          order by hiredate) next_sal 
   from ( 
 select deptno,ename,sal,hiredate, 
        count(*)over(partition by deptno,hiredate) cnt, 
        row_number()over(partition by deptno,hiredate order by sal) rn 
   from emp 
  where deptno=10 
        ) 
        ) 

DEPTNO ENAME     SAL HIREDATE    DIFF
------ ------  ----- ----------- ----------
    10 CLARK    2450 09-JUN-2006       1450
    10 ant      1000 17-NOV-2006       -300
    10 joe      1500 17-NOV-2006        200
    10 jim      1600 17-NOV-2006        300
    10 jon      1700 17-NOV-2006        400
    10 KING     5000 17-NOV-2006       3700
    10 MILLER   1300 23-JAN-2007        N/A
```

Now the solution is correct. As you can see, all the employees hired on November 17 now have their salaries compared with MILLER’s salary. Inspecting the results, employee ANT now has a value of –300 for DIFF, which is what we were hoping for. If it isn’t immediately obvious, the expression passed to LEAD OVER; CNT-RN+1 is simply the distance from each employee hired on November 17 to MILLER. Consider the following inline view, which shows the values for CNT and RN:

```
select deptno,ename,sal,hiredate, 
        count(*)over(partition by deptno,hiredate) cnt, 
        row_number()over(partition by deptno,hiredate order by sal) rn 
   from emp 
  where deptno=10 

DEPTNO ENAME    SAL HIREDATE           CNT         RN
------ ------ ----- ----------- ---------- ----------
    10 CLARK   2450 09-JUN-2006          1          1
    10 ant     1000 17-NOV-2006          5          1
    10 joe     1500 17-NOV-2006          5          2
    10 jim     1600 17-NOV-2006          5          3
    10 jon     1700 17-NOV-2006          5          4
    10 KING    5000 17-NOV-2006          5          5
    10 MILLER  1300 23-JAN-2007          1          1
```

The value for CNT represents, for each employee with a duplicate HIREDATE, how many duplicates there are in total for their HIREDATE. The value for RN represents a ranking for the employees in DEPTNO 10. The rank is partitioned by DEPTNO and HIREDATE so only employees with a HIREDATE that another employee has will have a value greater than one. The ranking is sorted by SAL (this is arbitrary; SAL is convenient, but we could have just as easily chosen EMPNO). Now that you know how many total duplicates there are and you have a ranking of each duplicate, the distance to MILLER is simply the total number of duplicates minus the current rank plus one (CNT-RN+1). The results of the distance calculation and its effect on LEAD OVER are shown here:

```
select deptno,ename,sal,hiredate, 
        lead(sal)over(partition by deptno 
                          order by hiredate) incorrect, 
        cnt-rn+1 distance, 
        lead(sal,cnt-rn+1)over(partition by deptno 
                          order by hiredate) correct 
   from ( 
 select deptno,ename,sal,hiredate, 
        count(*)over(partition by deptno,hiredate) cnt, 
        row_number()over(partition by deptno,hiredate 
                             order by sal) rn 
   from emp 
  where deptno=10 
        ) 

DEPTNO ENAME    SAL HIREDATE    INCORRECT    DISTANCE    CORRECT
------ ------ ----- ----------- ---------- ---------- ----------
    10 CLARK   2450 09-JUN-2006       1000          1       1000
    10 ant     1000 17-NOV-2006       1500          5       1300
    10 joe     1500 17-NOV-2006       1600          4       1300
    10 jim     1600 17-NOV-2006       1700          3       1300
    10 jon     1700 17-NOV-2006       5000          2       1300
    10 KING    5000 17-NOV-2006       1300          1       1300
    10 MILLER  1300 23-JAN-2007                     1
```

Now you can clearly see the effect that you have when you pass the correct distance to LEAD OVER. The rows for INCORRECT represent the values returned by LEAD OVER using a default distance of one. The rows for CORRECT represent the values returned by LEAD OVER using the proper distance for each employee with a duplicate HIREDATE to MILLER. At this point, all that is left is to find the difference between CORRECT and SAL for each row, which has already been shown.[]()

# 10.3 Locating the Beginning and End of a Range of Consecutive Values

## Problem

[]()This recipe is an extension of the prior recipe, and it uses the same view V from the prior recipe. Now that you’ve located the ranges of consecutive values, you want to find just their start and end points. Unlike the prior recipe, if a row is not part of a set of consecutive values, you still want to return it. Why? Because such a row represents both the beginning and end of its range. Using the data from view V:

```
select * 
   from V 


PROJ_ID PROJ_START  PROJ_END
------- ----------- -----------
      1 01-JAN-2020 02-JAN-2020
      2 02-JAN-2020 03-JAN-2020
      3 03-JAN-2020 04-JAN-2020
      4 04-JAN-2020 05-JAN-2020
      5 06-JAN-2020 07-JAN-2020
      6 16-JAN-2020 17-JAN-2020
      7 17-JAN-2020 18-JAN-2020
      8 18-JAN-2020 19-JAN-2020
      9 19-JAN-2020 20-JAN-2020
     10 21-JAN-2020 22-JAN-2020
     11 26-JAN-2020 27-JAN-2020
     12 27-JAN-2020 28-JAN-2020
     13 28-JAN-2020 29-JAN-2020
     14 29-JAN-2020 30-JAN-2020
```

you want the final result set to be as follows:

```
PROJ_GRP PROJ_START  PROJ_END
-------- ----------- -----------
       1 01-JAN-2020 05-JAN-2020
       2 06-JAN-2020 07-JAN-2020
       3 16-JAN-2020 20-JAN-2020
       4 21-JAN-2020 22-JAN-2020
       5 26-JAN-2020 30-JAN-2020
```

## Solution

This problem is a bit more involved than its predecessor. First, you must identify what the ranges are. A range of rows is defined by the values for PROJ\_START and PROJ\_END. For a row to be considered “consecutive” or part of a group, its PROJ\_START value must equal the PROJ\_END value of the row before it. In the case where a row’s PROJ\_START value does not equal the prior row’s PROJ\_END value and its PROJ\_END value does not equal the next row’s PROJ\_START value, this is an instance of a single row group. Once you have identify the ranges, you need to be able to group the rows in these ranges together (into groups) and return only their start and end points.

Examine the first row of the desired result set. The PROJ\_START is the PROJ_ START for PROJ\_ID 1 from view V, and the PROJ\_END is the PROJ\_END for PROJ\_ID 4 from view V. Despite the fact that PROJ\_ID 4 does not have a consecutive value following it, it is the last of a range of consecutive values, and thus it is included in the first group.

The most straightforward approach for this problem is to use the LAG OVER window function. Use LAG OVER to determine whether each prior row’s PROJ\_END equals the current row’s PROJ\_START to help place the rows into groups. []()[]()Once they are grouped, use the aggregate functions MIN and MAX to find their start and end points:

```
 1 select proj_grp, min(proj_start), max(proj_end)
 2   from (
 3 select proj_id,proj_start,proj_end,
 4        sum(flag)over(order by proj_id) proj_grp
 5   from (
 6 select proj_id,proj_start,proj_end,
 7        case when
 8             lag(proj_end)over(order by proj_id) = proj_start
 9             then 0 else 1
10        end flag
11   from V
12        ) alias1
13        ) alias2
14  group by proj_grp
```

## Discussion

The window function LAG OVER is extremely useful in this situation. []()You can examine each prior row’s PROJ\_END value without a self-join, without a scalar subquery, and without a view. The results of the LAG OVER function without the CASE expression are as follows:

```
select proj_id,proj_start,proj_end, 
       lag(proj_end)over(order by proj_id) prior_proj_end 
   from V 


PROJ_ID PROJ_START  PROJ_END    PRIOR_PROJ_END
------- ----------- ----------- --------------
      1 01-JAN-2020 02-JAN-2020
      2 02-JAN-2020 03-JAN-2020 02-JAN-2020
      3 03-JAN-2020 04-JAN-2020 03-JAN-2020
      4 04-JAN-2020 05-JAN-2020 04-JAN-2020
      5 06-JAN-2020 07-JAN-2020 05-JAN-2020
      6 16-JAN-2020 17-JAN-2020 07-JAN-2020
      7 17-JAN-2020 18-JAN-2020 17-JAN-2020
      8 18-JAN-2020 19-JAN-2020 18-JAN-2020
      9 19-JAN-2020 20-JAN-2020 19-JAN-2020
     10 21-JAN-2020 22-JAN-2020 20-JAN-2020
     11 26-JAN-2020 27-JAN-2020 22-JAN-2020
     12 27-JAN-2020 28-JAN-2020 27-JAN-2020
     13 28-JAN-2020 29-JAN-2020 28-JAN-2020
     14 29-JAN-2020 30-JAN-2020 29-JAN-2020
```

The CASE expression in the complete solution simply compares the value returned by LAG OVER to the current row’s PROJ\_START value; if they are the same, return 0, else return 1. The next step is to create a running total on the zeros and ones returned by the CASE expression to put each row into a group. []()The results of the running total are shown here:

```
select proj_id,proj_start,proj_end, 
        sum(flag)over(order by proj_id) proj_grp 
   from ( 
 select proj_id,proj_start,proj_end, 
        case when 
             lag(proj_end)over(order by proj_id) = proj_start 
             then 0 else 1 
        end flag 
   from V 
        ) 


PROJ_ID PROJ_START  PROJ_END      PROJ_GRP
------- ----------- ----------- ----------
      1 01-JAN-2020 02-JAN-2020          1
      2 02-JAN-2020 03-JAN-2020          1
      3 03-JAN-2020 04-JAN-2020          1
      4 04-JAN-2020 05-JAN-2020          1
      5 06-JAN-2020 07-JAN-2020          2
      6 16-JAN-2020 17-JAN-2020          3
      7 17-JAN-2020 18-JAN-2020          3
      8 18-JAN-2020 19-JAN-2020          3
      9 19-JAN-2020 20-JAN-2020          3
     10 21-JAN-2020 22-JAN-2020          4
     11 26-JAN-2020 27-JAN-2020          5
     12 27-JAN-2020 28-JAN-2020          5
     13 28-JAN-2020 29-JAN-2020          5
     14 29-JAN-2020 30-JAN-2020          5
```

Now that each row has been placed into a group, simply use the aggregate functions MIN and MAX on PROJ\_START and PROJ\_END, respectively, and group by the values created in the PROJ\_GRP running total column.[]()

# 10.4 Filling in Missing Values in a Range of Values

## Problem

[]()You want to return the number of employees hired each year for the entire decade of the 2005s, but there are some years in which no employees were hired. You would like to return the following result set:

```
YR          CNT
---- ----------
2005          1
2006         10
2007          2
2008          1
2009          0
2010          0
2011          0
2012          0
2013          0
2014          0
```

## Solution

The trick to this solution is returning zeros for years that saw no employees hired. If no employee was hired in a given year, then no rows for that year will exist in table EMP. If the year does not exist in the table, how can you return a count, any count, even zero? The solution requires you to outer join. You must supply a result set that returns all the years you want to see, and then perform a count against table EMP to see if there were any employees hired in each of those years.

### DB2

Use table EMP as a pivot table (because it has 14 rows) and the built-in function []()YEAR to generate one row for each year in the decade of 2005. Outer join to table EMP and count how many employees were hired each year:

```
 1 select x.yr, coalesce(y.cnt,0) cnt
 2   from (
 3 select year(min(hiredate)over()) -
 4        mod(year(min(hiredate)over()),10) +
 5        row_number()over()-1 yr
 6   from emp fetch first 10 rows only
 7        ) x
 8   left join
 9        (
10 select year(hiredate) yr1, count(*) cnt
11   from emp
12  group by year(hiredate)
13        ) y
14     on ( x.yr = y.yr1 )
```

### Oracle

The Oracle solution follows the same structure as the DB2 solution, with only the differences in the syntax Oracle handles causing a distinct solution to be required:

```
 1 select x.yr, coalesce(cnt,0) cnt
 2   from (
 3 select extract(year from min(hiredate)over()) -
 4        mod(extract(year from min(hiredate)over()),10) +
 5        rownum-1 yr
 6   from emp
 7  where rownum <= 10
 8        ) x
 9   left join
10        (
11 select to_number(to_char(hiredate,'YYYY')) yr, count(*) cnt
12   from emp
13  group by to_number(to_char(hiredate,'YYYY'))
14        ) y
15     on ( x.yr = y.yr )
```

### PostgreSQL and MySQL

[]()Use table T10 as a pivot table (because it has 10 rows) and the built-in function EXTRACT to generate one row for each year in the decade of 2005. Outer join to table EMP and count how many employees were hired each year:

```
 1 select y.yr, coalesce(x.cnt,0) as cnt
 2   from (
 3 selectmin_year-mod(cast(min_year as int),10)+rn as yr
 4   from (
 5 select (select min(extract(year from hiredate))
 6           from emp) as min_year,
 7        id-1 as rn
 8   from t10
 9        ) a
10        ) y
11   left join
12        (
13 select extract(year from hiredate) as yr, count(*) as cnt
14   from emp
15  group by extract(year from hiredate)
16        ) x
17     on ( y.yr = x.yr )
```

### SQL Server

Use table EMP as a pivot table (because it has 14 rows) and the built-in function YEAR to generate one row for each year in the decade of 2005. Outer join to table EMP and count how many employees were hired each year:

```
 1 select x.yr, coalesce(y.cnt,0) cnt
 2   from (
 3 select top (10)
 4        (year(min(hiredate)over()) -
 5         year(min(hiredate)over())%10)+
 6         row_number()over(order by hiredate)-1 yr
 7   from emp
 8        ) x
 9   left join
10        (
11 select year(hiredate) yr, count(*) cnt
12   from emp
13  group by year(hiredate)
14        ) y
15     on ( x.yr = y.yr )
```

## Discussion

Despite the difference in syntax, the approach is the same for all solutions. Inline view X returns each year in the decade of the ’80s by first finding the year of the earliest HIREDATE. The next step is to add RN–1 to the difference between the earliest year and the earliest year modulus ten. To see how this works, simply execute inline view X and return each of the values involved separately. []()Listed here is the result set for inline view X using the window function MIN OVER (DB2, Oracle, SQL Server) and a scalar subquery (MySQL, PostgreSQL):

```
select year(min(hiredate)over()) - 
        mod(year(min(hiredate)over()),10) + 
        row_number()over()-1 yr, 
        year(min(hiredate)over()) min_year, 
        mod(year(min(hiredate)over()),10) mod_yr, 
        row_number()over()-1 rn 
   from emp fetch first 10 rows only 

  YR   MIN_YEAR     MOD_YR         RN
---- ---------- ---------- ----------
2005       2005          0          0
2006       2005          0          1
2007       2005          0          2
2008       2005          0          3
1984       2005          0          4
2010       2005          0          5
2011       2005          0          6
2012       2005          0          7
2013       2005          0          8
2014       2005          0          9


 select min_year-mod(min_year,10)+rn as yr, 
        min_year, 
        mod(min_year,10) as mod_yr 
        rn 
   from ( 
 select (select min(extract(year from hiredate)) 
           from emp) as min_year, 
         id-1 as rn 
   from t10 
        ) x 

  YR   MIN_YEAR     MOD_YR         RN
---- ---------- ---------- ----------
2005       2005          0          0
2006       2005          0          1
2007       2005          0          2
2008       2005          0          3
2009       2005          0          4
2010       2005          0          5
2011       2005          0          6
2012       2005          0          7
2013       2005          0          8
2014       2005          0          9
```

Inline view Y returns the year for each HIREDATE and the number of employees hired during that year:

```
select year(hiredate) yr, count(*) cnt 
   from emp 
  group by year(hiredate) 

   YR        CNT
----- ----------
 2005          1
 2006         10
 2007          2
 2008          1
```

Finally, outer join inline view Y to inline view X so that every year is returned even if there are no employees hired.[]()

# 10.5 Generating Consecutive Numeric Values

## Problem

[]()You would like to have a “row source generator” available to you in your queries. Row source generators are useful for queries that require pivoting. For example, you want to return a result set such as the following, up to any number of rows that you specify:

```
ID
---
  1
  2
  3
  4
  5
  6
  7
  8
  9
 10
…
```

[]()If your RDBMS provides built-in functions for returning rows dynamically, you do not need to create a pivot table in advance with a fixed number of rows. That’s why a dynamic row generator can be so handy. Otherwise, you must use a traditional pivot table with a fixed number of rows (that may not always be enough) to generate rows when needed.

## Solution

This solution shows how to return 10 rows of increasing numbers starting from 1. You can easily adapt the solution to return any number of rows.

The ability to return increasing values from one opens the door to many other solutions. For example, you can generate numbers to add to dates in order to generate sequences of days. You can also use such numbers to parse through strings.

### DB2 and SQL Server

Use the recursive WITH clause to generate a sequence of rows with incrementing values. Using a recursive CTE will in fact work with the majority of RDBMSs today:

```
 1 with x (id)
 2 as (
 3 select 1
 4  union all
 5 select id+1
 6   from x
 7  where id+1 <= 10
 8 )
 9 select * from x
```

### Oracle

[]()In Oracle Database you can generate rows using the MODEL clause:

```
1 select array id
2   from dual
3  model
4    dimension by (0 idx)
5    measures(1 array)
6    rules iterate (10) (
7      array[iteration_number] = iteration_number+1
8    )
```

### PostgreSQL

[]()Use the handy function GENERATE\_SERIES, which is designed for the express purpose of generating rows:

```
1 select id
2   from generate_series (1, 10) x(id)
```

## Discussion

### DB2 and SQL Server

The recursive WITH clause increments ID (which starts at one) until the WHERE clause is satisfied. To kick things off, you must generate one row having the value 1. You can do this by selecting 1 from a one-row table or, in the case of DB2, by using the VALUES clause to create a one-row result set.

### Oracle

[]()In the MODEL clause solution, there is an explicit ITERATE command that allows you to generate multiple rows. Without the ITERATE clause, only one row will be returned, since DUAL has only one row. For example:

```
select array id 
   from dual 
 model 
   dimension by (0 idx) 
   measures(1 array) 
   rules () 

 ID
 --
  1
```

The MODEL clause not only allows you array access to rows, it allows you to easily “create” or return rows that are not in the table you are selecting against. In this solution, IDX is the array index (location of a specific value in the array) and ARRAY (aliased ID) is the “array” of rows. The first row defaults to 1 and can be referenced with ARRAY\[0]. []()Oracle provides the function ITERATION\_NUMBER so you can track the number of times you’ve iterated. The solution iterates 10 times, causing ITERATION\_NUMBER to go from 0 to 9. Adding one to each of those values yields the results 1 through 10.

It may be easier to visualize what’s happening with the model clause if you execute the following query:

```
select 'array['||idx||'] = '||array as output 
   from dual 
  model 
    dimension by (0 idx) 
    measures(1 array) 
    rules iterate (10) ( 
      array[iteration_number] = iteration_number+1 
    ) 

OUTPUT
------------------
array[0] = 1
array[1] = 2
array[2] = 3
array[3] = 4
array[4] = 5
array[5] = 6
array[6] = 7
array[7] = 8
array[8] = 9
array[9] = 10
```

### PostgreSQL

[]()All the work is done by the function GENERATE\_SERIES. The function accepts three parameters, all numeric values. The first parameter is the start value, the second parameter is the ending value, and the third parameter is an optional “step” value (how much each value is incremented by). If you do not pass a third parameter, the increment defaults to one.

The GENERATE\_SERIES function is flexible enough so that you do not have to hardcode parameters. For example, if you wanted to return 5 rows starting from value 10 and ending with value 30, incrementing by 5 such that the result set is the following:

```
 ID
---
 10
 15
 20
 25
 30
```

you can be creative and do something like this:

```
select id
  from generate_series(
         (select min(deptno) from emp),
         (select max(deptno) from emp),
         5
       ) x(id)
```

Notice here that the actual values passed to GENERATE\_SERIES are not known when the query is written. Instead, they are generated by subqueries when the main query executes.[]()

# 10.6 Summing Up

Queries that take into account ranges are one of the most common requests from business users—they are a natural consquence of the way that businesses operate. At least some of the time, however, a degree of dexterity is needed to apply the range correctly, and the recipes in this chapter should demonstrate how to apply that dexterity.[]()