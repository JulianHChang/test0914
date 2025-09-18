# Chapter 11. Advanced Searching

[]()In a very real sense, this entire book so far has been about searching. You’ve seen all sorts of queries that use joins and WHERE clauses and grouping techniques to search out and return the results you need. Some types of searching operations, though, stand apart from others in that they represent a different way of thinking about searching. Perhaps you’re displaying a result set one page at a time. Half of that problem is to identify (search for) the entire set of records that you want to display. The other half of that problem is to repeatedly search for the next page to display as a user cycles through the records on a display. Your first thought may not be to think of pagination as a searching problem, but it *can* be thought of that way, and it can be solved that way; that is the type of searching solution this chapter is all about.

# 11.1 Paginating Through a Result Set

## Problem

[]()You want to paginate or “scroll through” a result set. For example, you want to return the first five salaries from table EMP, then the next five, and so forth. Your goal is to allow a user to view five records at a time, scrolling forward with each click of a *Next* button.

## Solution

Because there is no concept of first, last, or next in SQL, you must impose order on the rows you are working with. Only by imposing order can you accurately return ranges of records.

[]()Use the window function ROW\_NUMBER OVER to impose order, and specify the window of records that you want returned in your WHERE clause. For example, use this to return rows 1 through 5:

```
select sal 
   from ( 
 select row_number() over (order by sal) as rn, 
        sal 
   from emp 
        ) x 
  where rn between 1 and 5 


 SAL
----
 800
 950
1100
1250
1250
```

Then use this to return rows 6 through 10:

```
select sal 
   from ( 
 select row_number() over (order by sal) as rn, 
        sal 
   from emp 
        ) x 
  where rn between 6 and 10 


  SAL
-----
 1300
 1500
 1600
 2450
 2850
```

You can return any range of rows that you want simply by changing the WHERE clause of your query.

## Discussion

[]()The window function ROW\_NUMBER OVER in inline view X will assign a unique number to each salary (in increasing order starting from 1). Listed here is the result set for inline view X:

```
select row_number() over (order by sal) as rn, 
        sal 
   from emp 

RN         SAL
--  ----------
 1         800
 2         950
 3        1100
 4        1250
 5        1250
 6        1300
 7        1500
 8        1600
 9        2450
10        2850
11        2975
12        3000
13        3000
14        5000
```

Once a number has been assigned to a salary, simply pick the range you want to return by specifying values for RN.

[]()For Oracle users, an alternative: you can use ROWNUM instead of ROW NUMBER OVER to generate sequence numbers for the rows:

```
select sal 
   from ( 
 select sal, rownum rn 
   from ( 
 select sal 
   from emp 
  order by sal 
        ) 
        ) 
  where rn between 6 and 10 

  SAL
-----
 1300
 1500
 1600
 2450
 2850
```

Using ROWNUM forces you into writing an extra level of subquery. The innermost subquery sorts rows by salary. The next outermost subquery applies row numbers to those rows, and, finally, the very outermost SELECT returns the data you are after.[]()

# 11.2 Skipping n Rows from a Table

## Problem

[]()You want a query to return every other employee in table EMP; you want the first employee, third employee, and so forth. For example, from the following result set:

```
ENAME
--------
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

you want to return the following:

```
ENAME
----------
ADAMS
BLAKE
FORD
JONES
MARTIN
SCOTT
TURNER
```

## Solution

To skip the second or fourth or *n*th row from a result set, you must impose order on the result set; otherwise, there is no concept of first or next, second, or fourth.

Use the window function ROW\_NUMBER OVER to assign a number to each row, which you can then use in conjunction with the modulo function to skip unwanted rows. The modulo function is MOD for DB2, MySQL, PostgreSQL, and Oracle. In SQL Server, use the percent (%) operator. The following example uses MOD to skip even-numbered rows:

```
1  select ename
2    from (
3  select row_number() over (order by ename) rn,
4         ename
5    from emp
6         ) x
7   where mod(rn,2) = 1
```

## Discussion

The call to the window function ROW\_NUMBER OVER in inline view X will assign a rank to each row (no ties, even with duplicate names). The results are shown here:

```
select row_number() over (order by ename) rn, ename 
   from emp 

RN ENAME
-- --------
 1 ADAMS
 2 ALLEN
 3 BLAKE
 4 CLARK
 5 FORD
 6 JAMES
 7 JONES
 8 KING
 9 MARTIN
10 MILLER
11 SCOTT
12 SMITH
13 TURNER
14 WARD
```

The last step is to simply use modulus to skip every other row.[]()

# 11.3 Incorporating OR Logic When Using Outer Joins

## Problem

[]()You want to return the name and department information for all employees in departments 10 and 20 along with department information for departments 30 and 40 (but no employee information). Your first attempt looks like this:

```
select e.ename, d.deptno, d.dname, d.loc 
   from dept d, emp e 
  where d.deptno = e.deptno 
    and (e.deptno = 10 or e.deptno = 20) 
  order by 2 

ENAME       DEPTNO DNAME          LOC
------- ---------- -------------- -----------
CLARK           10 ACCOUNTING     NEW YORK
KING            10 ACCOUNTING     NEW YORK
MILLER          10 ACCOUNTING     NEW YORK
SMITH           20 RESEARCH       DALLAS
ADAMS           20 RESEARCH       DALLAS
FORD            20 RESEARCH       DALLAS
SCOTT           20 RESEARCH       DALLAS
JONES           20 RESEARCH       DALLAS
```

[]()[]()Because the join in this query is an inner join, the result set does not include department information for DEPTNOs 30 and 40.

You attempt to outer join EMP to DEPT with the following query, but you still do not get the correct results:

```
select e.ename, d.deptno, d.dname, d.loc 
   from dept d left join emp e 
     on (d.deptno = e.deptno) 
  where e.deptno = 10 
     or e.deptno = 20 
  order by 2 

ENAME       DEPTNO DNAME        LOC
------- ---------- ------------ -----------
CLARK           10 ACCOUNTING   NEW YORK
KING            10 ACCOUNTING   NEW YORK
MILLER          10 ACCOUNTING   NEW YORK
SMITH           20 RESEARCH     DALLAS
ADAMS           20 RESEARCH     DALLAS
FORD            20 RESEARCH     DALLAS
SCOTT           20 RESEARCH     DALLAS
JONES           20 RESEARCH     DALLAS
```

Ultimately, you would like the result set to be the following:

```
ENAME       DEPTNO DNAME        LOC
------- ---------- ------------ ---------
CLARK           10 ACCOUNTING    NEW YORK
KING            10 ACCOUNTING    NEW YORK
MILLER          10 ACCOUNTING    NEW YORK
SMITH           20 RESEARCH      DALLAS
JONES           20 RESEARCH      DALLAS
SCOTT           20 RESEARCH      DALLAS
ADAMS           20 RESEARCH      DALLAS
FORD            20 RESEARCH      DALLAS
                30 SALES         CHICAGO
                40 OPERATIONS    BOSTON
```

## Solution

Move the OR condition into the JOIN clause:

```
1  select e.ename, d.deptno, d.dname, d.loc
2    from dept d left join emp e
3      on (d.deptno = e.deptno
4         and (e.deptno=10 or e.deptno=20))
5   order by 2
```

[]()Alternatively, you can filter on EMP.DEPTNO first in an inline view and then outer join:

```
1  select e.ename, d.deptno, d.dname, d.loc
2    from dept d
3    left join
4         (select ename, deptno
5            from emp
6           where deptno in ( 10, 20 )
7         ) e on ( e.deptno = d.deptno )
8  order by 2
```

## Discussion

### DB2, MySQL, PostgreSQL, and SQL Server

Two solutions are given for these products. The first moves the OR condition into the JOIN clause, making it part of the join condition. By doing that, you can filter the rows returned from EMP without losing DEPTNOs 30 and 40 from DEPT.

The second solution moves the filtering into an inline view. Inline view E filters on EMP.DEPTNO and returns EMP rows of interest. These are then outer joined to DEPT. Because DEPT is the anchor table in the outer join, all departments, including 30 and 40, are returned.[]()

# 11.4 Determining Which Rows Are Reciprocals

## Problem

[]()[]()You have a table containing the results of two tests, and you want to determine which pair of scores are reciprocals. Consider the following result set from view V:

```
select *
  from V

TEST1      TEST2
----- ----------
   20         20
   50         25
   20         20
   60         30
   70         90
   80        130
   90         70
  100         50
  110         55
  120         60
  130         80
  140         70
```

Examining these results, you see that a test score for TEST1 of 70 and TEST2 of 90 is a reciprocal (there exists a score of 90 for TEST1 and a score of 70 for TEST2). Likewise, the scores of 80 for TEST1 and 130 for TEST2 are reciprocals of 130 for TEST1 and 80 for TEST2. Additionally, the scores of 20 for TEST1 and 20 for TEST2 are reciprocals of 20 for TEST2 and 20 for TEST1. You want to identify only one set of reciprocals. You want your result set to be this:

```
TEST1      TEST2
-----  ---------
   20         20
   70         90
   80        130
```

not this:

```
TEST1      TEST2
-----  ---------
   20         20
   20         20
   70         90
   80        130
   90         70
  130         80
```

## Solution

[]()[]()Use a self-join to identify rows where TEST1 equals TEST2, and vice versa:

```
select distinct v1.*
  from V v1, V v2
 where v1.test1 = v2.test2
   and v1.test2 = v2.test1
   and v1.test1 <= v1.test2
```

## Discussion

The self-join results in a Cartesian product in which every TEST1 score can be compared against every TEST2 score, and vice versa. The following query will identify the reciprocals:

```
select v1.*
  from V v1, V v2
 where v1.test1 = v2.test2
   and v1.test2 = v2.test1

TEST1      TEST2
----- ----------
   20         20
   20         20
   20         20
   20         20
   90         70
  130         80
   70         90
   80        130
```

[]()The use of DISTINCT ensures that duplicate rows are removed from the final result set. The final filter in the WHERE clause (and V1.TEST1 &lt;= V1.TEST2) will ensure that only one pair of reciprocals (where TEST1 is the smaller or equal value) is returned.[]()[]()

# 11.5 Selecting the Top n Records

## Problem

[]()You want to limit a result set to a specific number of records based on a ranking of some sort. For example, you want to return the names and salaries of the employees with the top five salaries.

## Solution

The solution to this problem depends on the use of a window function. Which window function you will use depends on how you want to deal with ties. []()The following solution uses DENSE\_RANK so that each tie in salary will count as only one against the total:

```
1  select ename,sal
2    from (
3  select ename, sal,
4         dense_rank() over (order by sal desc) dr
5    from emp
6         ) x
7   where dr <= 5
```

The total number of rows returned may exceed five, but there will be only five distinct salaries. Use ROW\_NUMBER OVER if you want to return five rows regardless of ties (as no ties are allowed with this function).

## Discussion

[]()The window function DENSE\_RANK OVER in inline view X does all the work. The following example shows the entire table after applying that function:

```
select ename, sal, 
        dense_rank() over (order by sal desc) dr 
   from emp 

ENAME      SAL         DR
------- ------ ----------
KING      5000          1
SCOTT     3000          2
FORD      3000          2
JONES     2975          3
BLAKE     2850          4
CLARK     2450          5
ALLEN     1600          6
TURNER    1500          7
MILLER    1300          8
WARD      1250          9
MARTIN    1250          9
ADAMS     1100         10
JAMES      950         11
SMITH      800         12
```

Now it’s just a matter of returning rows where DR is less than or equal to five.

# 11.6 Finding Records with the Highest and Lowest Values

## Problem

[]()[]()You want to find “extreme” values in your table. For example, you want to find the employees with the highest and lowest salaries in table EMP.

## Solution

### DB2, Oracle, and SQL Server

[]()[]()Use the window functions MIN OVER and MAX OVER to find the lowest and highest salaries, respectively:

```
1  select ename
2    from (
3  select ename, sal,
4         min(sal)over() min_sal,
5         max(sal)over() max_sal
6    from emp
7         ) x
8   where sal in (min_sal,max_sal)
```

## Discussion

### DB2, Oracle, and SQL Server

The window functions MIN OVER and MAX OVER allow each row to have access to the lowest and highest salaries. The result set from inline view X is as follows:

```
select ename, sal, 
        min(sal)over() min_sal, 
        max(sal)over() max_sal 
   from emp 

ENAME      SAL    MIN_SAL    MAX_SAL
------- ------ ---------- ----------
SMITH      800        800       5000
ALLEN     1600        800       5000
WARD      1250        800       5000
JONES     2975        800       5000
MARTIN    1250        800       5000
BLAKE     2850        800       5000
CLARK     2450        800       5000
SCOTT     3000        800       5000
KING      5000        800       5000
TURNER    1500        800       5000
ADAMS     1100        800       5000
JAMES      950        800       5000
FORD      3000        800       5000
MILLER    1300        800       5000
```

Given this result set, all that’s left is to return rows where SAL equals MIN\_SAL or MAX\_SAL.

# 11.7 Investigating Future Rows

## Problem

[]()You want to find any employees who earn less than the employee hired immediately after them. Based on the following result set:

```
ENAME             SAL HIREDATE
---------- ---------- ---------
SMITH             800 17-DEC-80
ALLEN            1600 20-FEB-81
WARD             1250 22-FEB-81
JONES            2975 02-APR-81
BLAKE            2850 01-MAY-81
CLARK            2450 09-JUN-81
TURNER           1500 08-SEP-81
MARTIN           1250 28-SEP-81
KING             5000 17-NOV-81
JAMES             950 03-DEC-81
FORD             3000 03-DEC-81
MILLER           1300 23-JAN-82
SCOTT            3000 09-DEC-82
ADAMS            1100 12-JAN-83
```

SMITH, WARD, MARTIN, JAMES, and MILLER earn less than the person hired immediately after they were hired, so those are the employees you want to find with a query.

## Solution

The first step is to define what “future” means. You must impose order on your result set to be able to define a row as having a value that is “later” than another.

You can use the LEAD OVER window function to access the salary of the next employee that was hired. It’s then a simple matter to check whether that salary is larger:

```
1  select ename, sal, hiredate
2    from (
3  select ename, sal, hiredate,
4         lead(sal)over(order by hiredate) next_sal
5    from emp
6         ) alias
7   where sal < next_sal
```

## Discussion

[]()The window function LEAD OVER is perfect for a problem such as this one. It not only makes for a more readable query than the solution for the other products, LEAD OVER also leads to a more flexible solution because an argument can be passed to it that will determine how many rows ahead it should look (by default one). Being able to leap ahead more than one row is important in the case of duplicates in the column you are ordering by.

The following example shows how easy it is to use LEAD OVER to look at the salary of the “next” employee hired:

```
select ename, sal, hiredate, 
        lead(sal)over(order by hiredate) next_sal 
   from emp 

ENAME      SAL HIREDATE    NEXT_SAL
------- ------ --------- ----------
SMITH      800 17-DEC-80       1600
ALLEN     1600 20-FEB-81       1250
WARD      1250 22-FEB-81       2975
JONES     2975 02-APR-81       2850
BLAKE     2850 01-MAY-81       2450
CLARK     2450 09-JUN-81       1500
TURNER    1500 08-SEP-81       1250
MARTIN    1250 28-SEP-81       5000
KING      5000 17-NOV-81        950
JAMES      950 03-DEC-81       3000
FORD      3000 03-DEC-81       1300
MILLER    1300 23-JAN-82       3000
SCOTT     3000 09-DEC-82       1100
ADAMS     1100 12-JAN-83
```

The final step is to return only rows where SAL is less than NEXT\_SAL. Because of []()LEAD OVER’s default range of one row, if there had been duplicates in table EMP—in particular, multiple employees hired on the same date—their SAL would be compared. This may or may not have been what you intended. If your goal is to compare the SAL of each employee with SAL of the next employee hired, excluding other employees hired on the same day, you can use the following solution as an alternative:

```
select ename, sal, hiredate
  from (
select ename, sal, hiredate,
       lead(sal,cnt-rn+1)over(order by hiredate) next_sal
  from (
select ename,sal,hiredate,
       count(*)over(partition by hiredate) cnt,
       row_number()over(partition by hiredate order by empno) rn
  from emp
       )
       )
 where sal < next_sal
```

The idea behind this solution is to find the distance from the current row to the row it should be compared with. For example, if there are five duplicates, the first of the five needs to leap five rows to get to its correct LEAD OVER row. The value for CNT represents, for each employee with a duplicate HIREDATE, how many duplicates there are in total for their HIREDATE. The value for RN represents a ranking for the employees in DEPTNO 10. The rank is partitioned by HIREDATE so only employees with a HIREDATE that another employee has will have a value greater than one. The ranking is sorted by EMPNO (this is arbitrary). Now that you know how many total duplicates there are and you have a ranking of each duplicate, the distance to the next HIREDATE is simply the total number of duplicates minus the current rank plus one (CNT-RN+1).

## See Also

For additional examples of using LEAD OVER in the presence of duplicates (and a more thorough discussion of this technique), see [Recipe 8.7](https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/ch08.html#sqlckbk-CHP-8-SECT-7) and [Recipe 10.2](https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/ch10.html#sqlckbk-CHP-10-SECT-2).[]()

# 11.8 Shifting Row Values

## Problem

[]()You want to return each employee’s name and salary along with the next highest and lowest salaries. If there are no higher or lower salaries, you want the results to wrap (first SAL shows last SAL and vice versa). You want to return the following result set:

```
ENAME             SAL    FORWARD     REWIND
---------- ---------- ---------- ----------
SMITH             800        950       5000
JAMES             950       1100        800
ADAMS            1100       1250        950
WARD             1250       1250       1100
MARTIN           1250       1300       1250
MILLER           1300       1500       1250
TURNER           1500       1600       1300
ALLEN            1600       2450       1500
CLARK            2450       2850       1600
BLAKE            2850       2975       2450
JONES            2975       3000       2850
SCOTT            3000       3000       2975
FORD             3000       5000       3000
KING             5000        800       3000
```

## Solution

[]()[]()The window functions LEAD OVER and LAG OVER make this problem easy to solve and the resulting queries very readable. Use the window functions LAG OVER and LEAD OVER to access prior and next rows relative to the current row:

```
1  select ename,sal,
2         coalesce(lead(sal)over(order by sal),min(sal)over()) forward,
3         coalesce(lag(sal)over(order by sal),max(sal)over()) rewind
4    from emp
```

## Discussion

The window functions LAG OVER and LEAD OVER will (by default and unless otherwise specified) return values from the row before and after the current row, respectively. You define what “before” or “after” means in the ORDER BY portion of the OVER clause. If you examine the solution, the first step is to return the next and prior rows relative to the current row, ordered by SAL:

```
select ename,sal, 
        lead(sal)over(order by sal) forward, 
        lag(sal)over(order by sal) rewind 
   from emp 


ENAME             SAL    FORWARD     REWIND
---------- ---------- ---------- ----------
SMITH             800        950
JAMES             950       1100        800
ADAMS            1100       1250        950
WARD             1250       1250       1100
MARTIN           1250       1300       1250
MILLER           1300       1500       1250
TURNER           1500       1600       1300
ALLEN            1600       2450       1500
CLARK            2450       2850       1600
BLAKE            2850       2975       2450
JONES            2975       3000       2850
SCOTT            3000       3000       2975
FORD             3000       5000       3000
KING             5000                  3000
```

Notice that REWIND is NULL for employee SMITH, and FORWARD is NULL for employee KING; that is because those two employees have the lowest and highest salaries, respectively. The requirement in the “Problem” section should NULL values exist in FORWARD or REWIND is to “wrap” the results, meaning that for the highest SAL, FORWARD should be the value of the lowest SAL in the table, and for the lowest SAL, REWIND should be the value of the highest SAL in the table. []()[]()The window functions MIN OVER and MAX OVER with no partition or window specified (i.e., an empty parentheses after the OVER clause) will return the lowest and highest salaries in the table, respectively. The results are shown here:

```
select ename,sal, 
        coalesce(lead(sal)over(order by sal),min(sal)over()) forward, 
        coalesce(lag(sal)over(order by sal),max(sal)over()) rewind 
   from emp 


ENAME             SAL    FORWARD     REWIND
---------- ---------- ---------- ----------
SMITH             800        950       5000
JAMES             950       1100        800
ADAMS            1100       1250        950
WARD             1250       1250       1100
MARTIN           1250       1300       1250
MILLER           1300       1500       1250
TURNER           1500       1600       1300
ALLEN            1600       2450       1500
CLARK            2450       2850       1600
BLAKE            2850       2975       2450
JONES            2975       3000       2850
SCOTT            3000       3000       2975
FORD             3000       5000       3000
KING             5000        800       3000
```

[]()Another useful feature of LAG OVER and LEAD OVER is the ability to define how far forward or back you would like to go. In the example for this recipe, you go only one row forward or back. If want to move three rows forward and five rows back, doing so is simple. Just specify the values 3 and 5, as shown here[]()[]():

```
select ename,sal, 
        lead(sal,3)over(order by sal) forward, 
        lag(sal,5)over(order by sal) rewind 
   from emp 

ENAME             SAL    FORWARD     REWIND
---------- ---------- ---------- ----------
SMITH             800      1250
JAMES             950      1250
ADAMS            1100      1300
WARD             1250      1500
MARTIN           1250      1600
MILLER           1300      2450         800
TURNER           1500      2850         950
ALLEN            1600      2975        1100
CLARK            2450      3000        1250
BLAKE            2850      3000        1250
JONES            2975      5000        1300
SCOTT            3000                  1500
FORD             3000                  1600
KING             5000                  2450
```

# 11.9 Ranking Results

## Problem

[]()You want to rank the salaries in table EMP while allowing for ties. You want to return the following result set:

```
RNK     SAL
--- -------
  1     800
  2     950
  3    1100
  4    1250
  4    1250
  5    1300
  6    1500
  7    1600
  8    2450
  9    2850
 10    2975
 11    3000
 11    3000
 12    5000
```

## Solution

Window functions make ranking queries extremely simple. []()Three window functions are particularly useful for ranking: DENSE\_RANK OVER, ROW\_NUMBER OVER, and RANK OVER.

[]()Because you want to allow for ties, use the window function DENSE\_RANK OVER:

```
1 select dense_rank() over(order by sal) rnk, sal
2   from emp
```

## Discussion

The window function DENSE\_RANK OVER does all the legwork here. In parentheses following the OVER keyword you place an ORDER BY clause to specify the order in which rows are ranked. The solution uses ORDER BY SAL, so rows from EMP are ranked in ascending order of salary.

# 11.10 Suppressing Duplicates

## Problem

[]()[]()You want to find the different job types in table EMP but do not want to see duplicates. The result set should be as follows:

```
JOB
---------
ANALYST
CLERK
MANAGER
PRESIDENT
SALESMAN
```

## Solution

All of the RDBMSs support the keyword DISTINCT, and it arguably is the easiest mechanism for suppressing duplicates from the result set. However, this recipe will also cover two additional methods for suppressing duplicates.

The traditional method of using DISTINCT and sometimes GROUP BY certainly works. []()The following solution is an alternative that makes use of the window function ROW\_NUMBER OVER:

```
1  select job
2    from (
3  select job,
4         row_number()over(partition by job order by job) rn
5    from emp
6         ) x
7   where rn = 1
```

### Traditional alternatives

[]()Use the DISTINCT keyword to suppress duplicates from the result set:

```
select distinct job
  from emp
```

Additionally, it is also possible to use GROUP BY to suppress duplicates:

```
select job
  from emp
 group by job
```

## Discussion

### DB2, Oracle, and SQL Server

This solution depends on some outside-the-box thinking about partitioned window functions. By using PARTITION BY in the OVER clause of ROW\_NUMBER, you can reset the value returned by ROW\_NUMBER to 1 whenever a new job is encountered. The following results are from inline view X:

```
select job, 
        row_number()over(partition by job order by job) rn 
   from emp 


JOB               RN
--------- ----------
ANALYST            1
ANALYST            2
CLERK              1
CLERK              2
CLERK              3
CLERK              4
MANAGER            1
MANAGER            2
MANAGER            3
PRESIDENT          1
SALESMAN           1
SALESMAN           2
SALESMAN           3
SALESMAN           4
```

Each row is given an increasing, sequential number, and that number is reset to one whenever the job changes. To filter out the duplicates, all you must do is keep the rows where RN is 1.

[]()An ORDER BY clause is mandatory when using ROW\_NUMBER OVER (except in DB2) but doesn’t affect the result. Which job is returned is irrelevant so long as you return one of each job.

### Traditional alternatives

The first solution shows how to use the keyword DISTINCT to suppress duplicates from a result set. []()[]()Keep in mind that DISTINCT is applied to the whole SELECT list; additional columns can and will change the result set. Consider the difference between these two queries:

```
select distinct job           select distinct job, deptno
  from emp                      from emp

JOB                           JOB           DEPTNO
---------                     --------- ----------
ANALYST                       ANALYST           20
CLERK                         CLERK             10
MANAGER                       CLERK             20
PRESIDENT                     CLERK             30
SALESMAN                      MANAGER           10
                              MANAGER           20
                              MANAGER           30
                              PRESIDENT         10
                              SALESMAN          30
```

By adding DEPTNO to the SELECT list, what you return is each DISTINCT pair of JOB/DEPTNO values from table EMP.

[]()The second solution uses GROUP BY to suppress duplicates. While using GROUP BY in this way is not uncommon, keep in mind that GROUP BY and DISTINCT are two very different clauses that are not interchangeable. I’ve included GROUP BY in this solution for completeness, as you will no doubt come across it at some point.[]()[]()

# 11.11 Finding Knight Values

## Problem

[]()[]()You want return a result set that contains each employee’s name, the department they work in, their salary, the date they were hired, and the salary of the last employee hired, in each department. You want to return the following result set:

```
DEPTNO ENAME             SAL HIREDATE    LATEST_SAL
------ ---------- ---------- ----------- ----------
    10 MILLER            1300 23-JAN-2007      1300
    10 KING              5000 17-NOV-2006      1300
    10 CLARK             2450 09-JUN-2006      1300
    20 ADAMS             1100 12-JAN-2007      1100
    20 SCOTT             3000 09-DEC-2007      1100
    20 FORD              3000 03-DEC-2006      1100
    20 JONES             2975 02-APR-2006      1100
    20 SMITH              800 17-DEC-2005      1100
    30 JAMES              950 03-DEC-2006       950
    30 MARTIN            1250 28-SEP-2006       950
    30 TURNER            1500 08-SEP-2006       950
    30 BLAKE             2850 01-MAY-2006       950
    30 WARD              1250 22-FEB-2006       950
    30 ALLEN             1600 20-FEB-2006       950
```

The values in LATEST\_SAL are the “knight values” because the path to find them is analogous to a knight’s path in the game of chess. You determine the result the way a knight determines a new location: by jumping to a row and then turning and jumping to a different column (see [Figure 11-1](#sqlckbk-CHP-11-FIG-1)). To find the correct values for LATEST\_SAL, you must first locate (jump to) the row with the latest HIREDATE in each DEPTNO, and then you select (jump to) the SAL column of that row.

![sql2 1101](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABVQAAAJsCAIAAACd6L89AAAACXBIWXMAAC4jAAAuIwF4pT92AAAYZWlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4gPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNi4wLWMwMDIgNzkuMTY0NDYwLCAyMDIwLzA1LzEyLTE2OjA0OjE3ICAgICAgICAiPiA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPiA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIiB4bWxuczpkYz0iaHR0cDovL3B1cmwub3JnL2RjL2VsZW1lbnRzLzEuMS8iIHhtbG5zOnhtcD0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLyIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczpzdEV2dD0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlRXZlbnQjIiB4bWxuczppbGx1c3RyYXRvcj0iaHR0cDovL25zLmFkb2JlLmNvbS9pbGx1c3RyYXRvci8xLjAvIiB4bWxuczp4bXBUUGc9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC90L3BnLyIgeG1sbnM6c3REaW09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9EaW1lbnNpb25zIyIgeG1sbnM6c3RGbnQ9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9Gb250IyIgeG1sbnM6eG1wRz0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL2cvIiB4bWxuczpwZGY9Imh0dHA6Ly9ucy5hZG9iZS5jb20vcGRmLzEuMy8iIHhtbG5zOnBob3Rvc2hvcD0iaHR0cDovL25zLmFkb2JlLmNvbS9waG90b3Nob3AvMS4wLyIgZGM6Zm9ybWF0PSJpbWFnZS9wbmciIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgSWxsdXN0cmF0b3IgMjQuMyAoTWFjaW50b3NoKSIgeG1wOkNyZWF0ZURhdGU9IjIwMjAtMTAtMjZUMTY6Mjk6MzEtMDQ6MDAiIHhtcDpNb2RpZnlEYXRlPSIyMDIwLTEwLTI2VDE2OjMwOjA3LTA0OjAwIiB4bXA6TWV0YWRhdGFEYXRlPSIyMDIwLTEwLTI2VDE2OjMwOjA3LTA0OjAwIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjIyNWM3OGI1LTdlNjMtNGNhNS1iNjZjLWI0MTg1NTI2NTJmZSIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDpjMDZiN2I4Ni1jZGFlLTQ4OTYtOGJmMS01MjY5MTllNGRjMWQiIHhtcE1NOlJlbmRpdGlvbkNsYXNzPSJwcm9vZjpwZGYiIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0idXVpZDo1NEJGNUVEMjQ2M0FERTExOTU3M0I0QUFEQjU0NTk4RCIgaWxsdXN0cmF0b3I6U3RhcnR1cFByb2ZpbGU9IlByaW50IiBpbGx1c3RyYXRvcjpUeXBlPSJEb2N1bWVudCIgaWxsdXN0cmF0b3I6Q3JlYXRvclN1YlRvb2w9IkFJUm9iaW4iIHhtcFRQZzpOUGFnZXM9IjEiIHhtcFRQZzpIYXNWaXNpYmxlVHJhbnNwYXJlbmN5PSJGYWxzZSIgeG1wVFBnOkhhc1Zpc2libGVPdmVycHJpbnQ9IlRydWUiIHBkZjpQcm9kdWNlcj0iQWRvYmUgUERGIGxpYnJhcnkgMTUuMDAiIHBob3Rvc2hvcDpDb2xvck1vZGU9IjMiIHBob3Rvc2hvcDpJQ0NQcm9maWxlPSJzUkdCIElFQzYxOTY2LTIuMSI+IDxkYzp0aXRsZT4gPHJkZjpBbHQ+IDxyZGY6bGkgeG1sOmxhbmc9IngtZGVmYXVsdCI+c3FsYzJfMTIwMTwvcmRmOmxpPiA8L3JkZjpBbHQ+IDwvZGM6dGl0bGU+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ1dWlkOjUyNjgzYmU4LTE0MWMtNGE0Ni1hYjE1LTEwZDE2ZjQ3Zjk3ZSIgc3RSZWY6ZG9jdW1lbnRJRD0ieG1wLmRpZDoyMjVjNzhiNS03ZTYzLTRjYTUtYjY2Yy1iNDE4NTUyNjUyZmUiIHN0UmVmOm9yaWdpbmFsRG9jdW1lbnRJRD0idXVpZDo1NEJGNUVEMjQ2M0FERTExOTU3M0I0QUFEQjU0NTk4RCIgc3RSZWY6cmVuZGl0aW9uQ2xhc3M9InByb29mOnBkZiIvPiA8eG1wTU06SGlzdG9yeT4gPHJkZjpTZXE+IDxyZGY6bGkgc3RFdnQ6YWN0aW9uPSJzYXZlZCIgc3RFdnQ6aW5zdGFuY2VJRD0ieG1wLmlpZDpGNzdGMTE3NDA3MjA2ODExOTk0Q0Q2NDI4RThFQjY4NCIgc3RFdnQ6d2hlbj0iMjAxMS0wNS0wNFQxNDoyMTozNS0wNDowMCIgc3RFdnQ6c29mdHdhcmVBZ2VudD0iQWRvYmUgSWxsdXN0cmF0b3IgQ1M0IiBzdEV2dDpjaGFuZ2VkPSIvIi8+IDxyZGY6bGkgc3RFdnQ6YWN0aW9uPSJzYXZlZCIgc3RFdnQ6aW5zdGFuY2VJRD0ieG1wLmlpZDoyMjVjNzhiNS03ZTYzLTRjYTUtYjY2Yy1iNDE4NTUyNjUyZmUiIHN0RXZ0OndoZW49IjIwMjAtMDktMThUMDg6NTc6MTItMDQ6MDAiIHN0RXZ0OnNvZnR3YXJlQWdlbnQ9IkFkb2JlIElsbHVzdHJhdG9yIDI0LjMgKE1hY2ludG9zaCkiIHN0RXZ0OmNoYW5nZWQ9Ii8iLz4gPHJkZjpsaSBzdEV2dDphY3Rpb249ImNvbnZlcnRlZCIgc3RFdnQ6cGFyYW1ldGVycz0iZnJvbSBhcHBsaWNhdGlvbi9wZGYgdG8gYXBwbGljYXRpb24vdm5kLmFkb2JlLnBob3Rvc2hvcCIvPiA8cmRmOmxpIHN0RXZ0OmFjdGlvbj0iZGVyaXZlZCIgc3RFdnQ6cGFyYW1ldGVycz0iY29udmVydGVkIGZyb20gYXBwbGljYXRpb24vdm5kLmFkb2JlLnBob3Rvc2hvcCB0byBpbWFnZS9wbmciLz4gPHJkZjpsaSBzdEV2dDphY3Rpb249InNhdmVkIiBzdEV2dDppbnN0YW5jZUlEPSJ4bXAuaWlkOmMwNmI3Yjg2LWNkYWUtNDg5Ni04YmYxLTUyNjkxOWU0ZGMxZCIgc3RFdnQ6d2hlbj0iMjAyMC0xMC0yNlQxNjozMDowNy0wNDowMCIgc3RFdnQ6c29mdHdhcmVBZ2VudD0iQWRvYmUgUGhvdG9zaG9wIDIxLjIgKE1hY2ludG9zaCkiIHN0RXZ0OmNoYW5nZWQ9Ii8iLz4gPC9yZGY6U2VxPiA8L3htcE1NOkhpc3Rvcnk+IDx4bXBUUGc6TWF4UGFnZVNpemUgc3REaW06dz0iNC41NDY1NzYiIHN0RGltOmg9IjIuMDY3MDUzIiBzdERpbTp1bml0PSJJbmNoZXMiLz4gPHhtcFRQZzpGb250cz4gPHJkZjpCYWc+IDxyZGY6bGkgc3RGbnQ6Zm9udE5hbWU9Ikd1YXJkaWFuU2Fuc0NvbmQtTWVkaXVtIiBzdEZudDpmb250RmFtaWx5PSJHdWFyZGlhbiBTYW5zIENvbmQiIHN0Rm50OmZvbnRGYWNlPSJNZWRpdW0iIHN0Rm50OmZvbnRUeXBlPSJPcGVuIFR5cGUiIHN0Rm50OnZlcnNpb25TdHJpbmc9IlZlcnNpb24gMS4wMDE7UFMgMDAxLjAwMTtob3Rjb252IDEuMC41NzttYWtlb3RmLmxpYjIuMC4yMTg5NSIgc3RGbnQ6Y29tcG9zaXRlPSJGYWxzZSIgc3RGbnQ6Zm9udEZpbGVOYW1lPSJHdWFyZGlhbiBTYW5zIENvbmQtTWVkaXVtLm90ZiIvPiA8cmRmOmxpIHN0Rm50OmZvbnROYW1lPSJHdWFyZGlhblNhbnNDb25kLUJvbGQiIHN0Rm50OmZvbnRGYW1pbHk9Ikd1YXJkaWFuIFNhbnMgQ29uZCIgc3RGbnQ6Zm9udEZhY2U9IkJvbGQiIHN0Rm50OmZvbnRUeXBlPSJPcGVuIFR5cGUiIHN0Rm50OnZlcnNpb25TdHJpbmc9IlZlcnNpb24gMS4wMDE7UFMgMDAxLjAwMTtob3Rjb252IDEuMC41NzttYWtlb3RmLmxpYjIuMC4yMTg5NSIgc3RGbnQ6Y29tcG9zaXRlPSJGYWxzZSIgc3RGbnQ6Zm9udEZpbGVOYW1lPSJHdWFyZGlhbiBTYW5zIENvbmQtQm9sZC5vdGYiLz4gPC9yZGY6QmFnPiA8L3htcFRQZzpGb250cz4gPHhtcFRQZzpQbGF0ZU5hbWVzPiA8cmRmOlNlcT4gPHJkZjpsaT5DeWFuPC9yZGY6bGk+IDxyZGY6bGk+TWFnZW50YTwvcmRmOmxpPiA8cmRmOmxpPlllbGxvdzwvcmRmOmxpPiA8cmRmOmxpPkJsYWNrPC9yZGY6bGk+IDwvcmRmOlNlcT4gPC94bXBUUGc6UGxhdGVOYW1lcz4gPHhtcFRQZzpTd2F0Y2hHcm91cHM+IDxyZGY6U2VxPiA8cmRmOmxpPiA8cmRmOkRlc2NyaXB0aW9uIHhtcEc6Z3JvdXBOYW1lPSJEZWZhdWx0IFN3YXRjaCBHcm91cCIgeG1wRzpncm91cFR5cGU9IjAiPiA8eG1wRzpDb2xvcmFudHM+IDxyZGY6U2VxPiA8cmRmOmxpIHhtcEc6c3dhdGNoTmFtZT0iUj0wIEc9MTQ2IEI9NjkiIHhtcEc6dHlwZT0iUFJPQ0VTUyIgeG1wRzp0aW50PSIxMDAuMDAwMDAwIiB4bXBHOm1vZGU9IlJHQiIgeG1wRzpyZWQ9IjAiIHhtcEc6Z3JlZW49IjE0NSIgeG1wRzpibHVlPSI2OCIvPiA8cmRmOmxpIHhtcEc6c3dhdGNoTmFtZT0iUj0xOTMgRz0zOSBCPTQ1IiB4bXBHOnR5cGU9IlBST0NFU1MiIHhtcEc6dGludD0iMTAwLjAwMDAwMCIgeG1wRzptb2RlPSJSR0IiIHhtcEc6cmVkPSIxOTMiIHhtcEc6Z3JlZW49IjM4IiB4bXBHOmJsdWU9IjQ1Ii8+IDxyZGY6bGkgeG1wRzpzd2F0Y2hOYW1lPSJSPTI0NyBHPTE0NyBCPTMwIiB4bXBHOnR5cGU9IlBST0NFU1MiIHhtcEc6dGludD0iMTAwLjAwMDAwMCIgeG1wRzptb2RlPSJSR0IiIHhtcEc6cmVkPSIyNDYiIHhtcEc6Z3JlZW49IjE0NyIgeG1wRzpibHVlPSIyOSIvPiA8cmRmOmxpIHhtcEc6c3dhdGNoTmFtZT0iUj0wIEc9MTEzIEI9MTg4IiB4bXBHOnR5cGU9IlBST0NFU1MiIHhtcEc6dGludD0iMTAwLjAwMDAwMCIgeG1wRzptb2RlPSJSR0IiIHhtcEc6cmVkPSIwIiB4bXBHOmdyZWVuPSIxMTIiIHhtcEc6Ymx1ZT0iMTg4Ii8+IDxyZGY6bGkgeG1wRzpzd2F0Y2hOYW1lPSJDPTIzIE09NTIgWT02NiBLPTE3IiB4bXBHOnR5cGU9IlBST0NFU1MiIHhtcEc6dGludD0iMTAwLjAwMDAwMCIgeG1wRzptb2RlPSJSR0IiIHhtcEc6cmVkPSIxNzAiIHhtcEc6Z3JlZW49IjExNiIgeG1wRzpibHVlPSI4NCIvPiA8cmRmOmxpIHhtcEc6c3dhdGNoTmFtZT0iUj0yNTUgRz0yMjIgQj0wIiB4bXBHOnR5cGU9IlNQT1QiIHhtcEc6dGludD0iMTAwLjAwMDAwMCIgeG1wRzptb2RlPSJSR0IiIHhtcEc6cmVkPSIyNTUiIHhtcEc6Z3JlZW49IjIyMiIgeG1wRzpibHVlPSIwIi8+IDxyZGY6bGkgeG1wRzpzd2F0Y2hOYW1lPSJSPTE1MyBHPTAgQj0yMDQiIHhtcEc6dHlwZT0iU1BPVCIgeG1wRzp0aW50PSIxMDAuMDAwMDAwIiB4bXBHOm1vZGU9IlJHQiIgeG1wRzpyZWQ9IjE1MyIgeG1wRzpncmVlbj0iMCIgeG1wRzpibHVlPSIyMDQiLz4gPC9yZGY6U2VxPiA8L3htcEc6Q29sb3JhbnRzPiA8L3JkZjpEZXNjcmlwdGlvbj4gPC9yZGY6bGk+IDxyZGY6bGk+IDxyZGY6RGVzY3JpcHRpb24geG1wRzpncm91cE5hbWU9IkdyYXlzY2FsZSIgeG1wRzpncm91cFR5cGU9IjEiPiA8eG1wRzpDb2xvcmFudHM+IDxyZGY6U2VxPiA8cmRmOmxpIHhtcEc6c3dhdGNoTmFtZT0iSz0xMDAiIHhtcEc6bW9kZT0iR1JBWSIgeG1wRzp0eXBlPSJQUk9DRVNTIiB4bXBHOmdyYXk9IjI1NSIvPiA8cmRmOmxpIHhtcEc6c3dhdGNoTmFtZT0iSz05MCIgeG1wRzptb2RlPSJHUkFZIiB4bXBHOnR5cGU9IlBST0NFU1MiIHhtcEc6Z3JheT0iMjI5Ii8+IDxyZGY6bGkgeG1wRzpzd2F0Y2hOYW1lPSJLPTgwIiB4bXBHOm1vZGU9IkdSQVkiIHhtcEc6dHlwZT0iUFJPQ0VTUyIgeG1wRzpncmF5PSIyMDMiLz4gPHJkZjpsaSB4bXBHOnN3YXRjaE5hbWU9Iks9NzAiIHhtcEc6bW9kZT0iR1JBWSIgeG1wRzp0eXBlPSJQUk9DRVNTIiB4bXBHOmdyYXk9IjE3OCIvPiA8cmRmOmxpIHhtcEc6c3dhdGNoTmFtZT0iSz02MCIgeG1wRzptb2RlPSJHUkFZIiB4bXBHOnR5cGU9IlBST0NFU1MiIHhtcEc6Z3JheT0iMTUyIi8+IDxyZGY6bGkgeG1wRzpzd2F0Y2hOYW1lPSJLPTUwIiB4bXBHOm1vZGU9IkdSQVkiIHhtcEc6dHlwZT0iUFJPQ0VTUyIgeG1wRzpncmF5PSIxMjciLz4gPHJkZjpsaSB4bXBHOnN3YXRjaE5hbWU9Iks9NDAiIHhtcEc6bW9kZT0iR1JBWSIgeG1wRzp0eXBlPSJQUk9DRVNTIiB4bXBHOmdyYXk9IjEwMSIvPiA8cmRmOmxpIHhtcEc6c3dhdGNoTmFtZT0iSz0zMCIgeG1wRzptb2RlPSJHUkFZIiB4bXBHOnR5cGU9IlBST0NFU1MiIHhtcEc6Z3JheT0iNzYiLz4gPHJkZjpsaSB4bXBHOnN3YXRjaE5hbWU9Iks9MjAiIHhtcEc6bW9kZT0iR1JBWSIgeG1wRzp0eXBlPSJQUk9DRVNTIiB4bXBHOmdyYXk9IjUwIi8+IDxyZGY6bGkgeG1wRzpzd2F0Y2hOYW1lPSJLPTEwIiB4bXBHOm1vZGU9IkdSQVkiIHhtcEc6dHlwZT0iUFJPQ0VTUyIgeG1wRzpncmF5PSIyNSIvPiA8cmRmOmxpIHhtcEc6c3dhdGNoTmFtZT0iSz01IiB4bXBHOm1vZGU9IkdSQVkiIHhtcEc6dHlwZT0iUFJPQ0VTUyIgeG1wRzpncmF5PSIxMiIvPiA8cmRmOmxpIHhtcEc6c3dhdGNoTmFtZT0iSz0wIDEiIHhtcEc6bW9kZT0iR1JBWSIgeG1wRzp0eXBlPSJQUk9DRVNTIiB4bXBHOmdyYXk9IjAiLz4gPC9yZGY6U2VxPiA8L3htcEc6Q29sb3JhbnRzPiA8L3JkZjpEZXNjcmlwdGlvbj4gPC9yZGY6bGk+IDwvcmRmOlNlcT4gPC94bXBUUGc6U3dhdGNoR3JvdXBzPiA8L3JkZjpEZXNjcmlwdGlvbj4gPC9yZGY6UkRGPiA8L3g6eG1wbWV0YT4gPD94cGFja2V0IGVu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)

###### Figure 11-1. A knight value comes from “up and over”

###### Tip

[]()The term *knight value* was coined by a clever coworker of Anthony’s, Kay Young. After having him review the recipes for correctness, Anthony admitted to Kay that he was stumped and could not come up with a good title. Because you need to initially evaluate one row and then “jump” and take a value from another, Kay came up with the term *knight value*.

## Solution

### DB2 and SQL Server

Use a CASE expression in a subquery to return the SAL of the last employee hired in each DEPTNO; for all other salaries, return 0. Use the window function MAX OVER in the outer query to return the nonzero SAL for each employee’s department:

```
 1  select deptno,
 2         ename,
 3         sal,
 4         hiredate,
 5         max(latest_sal)over(partition by deptno) latest_sal
 6    from (
 7  select deptno,
 8         ename,
 9         sal,
10         hiredate,
11         case
12           when hiredate = max(hiredate)over(partition by deptno)
13           then sal else 0
14         end latest_sal
15    from emp
16         ) x
17   order by 1, 4 desc
```

### Oracle

Use the window function MAX OVER to return the highest SAL for each DEPTNO. []()[]()Use the functions DENSE\_RANK and LAST, while ordering by HIREDATE in the KEEP clause to return the highest SAL for the latest HIREDATE in a given DEPTNO:

```
1  select deptno,
2          ename,
3          sal,
4          hiredate,
5           max(sal)
6             keep(dense_rank last order by hiredate)
7             over(partition by deptno) latest_sal
8    from emp
9  order by 1, 4 desc
```

## Discussion

### DB2 and SQL Server

The first step is to use the window function MAX OVER in a CASE expression to find the employee hired last, or most recently, in each DEPTNO. If an employee’s HIREDATE matches the value returned by MAX OVER, then use a CASE expression to return that employee’s SAL; otherwise, return zero. The results of this are shown here:

```
select deptno, 
        ename, 
        sal, 
        hiredate, 
        case 
            when hiredate = max(hiredate)over(partition by deptno) 
            then sal else 0 
        end latest_sal 
   from emp 


DEPTNO ENAME             SAL HIREDATE    LATEST_SAL
------ --------- ----------- ----------- ----------
    10 CLARK            2450 09-JUN-2006          0
    10 KING             5000 17-NOV-2006          0
    10 MILLER           1300 23-JAN-2007       1300
    20 SMITH             800 17-DEC-2005          0
    20 ADAMS            1100 12-JAN-2007       1100
    20 FORD             3000 03-DEC-2006          0
    20 SCOTT            3000 09-DEC-2007          0
    20 JONES            2975 02-APR-2006          0
    30 ALLEN            1600 20-FEB-2006          0
    30 BLAKE            2850 01-MAY-2006          0
    30 MARTIN           1250 28-SEP-2006          0
    30 JAMES             950 03-DEC-2006        950
    30 TURNER           1500 08-SEP-2006          0
    30 WARD             1250 22-FEB-2006          0
```

Because the value for LATEST\_SAL will be either zero or the SAL of the employee(s) hired most recently, you can wrap the previous query in an inline view and use MAX OVER again, but this time to return the greatest nonzero LATEST\_SAL for each DEPTNO:

```
select deptno, 
        ename, 
        sal, 
        hiredate, 
        max(latest_sal)over(partition by deptno) latest_sal 
   from ( 
 select deptno, 
        ename, 
        sal, 
        hiredate, 
        case 
            when hiredate = max(hiredate)over(partition by deptno) 
            then sal else 0 
        end latest_sal 
   from emp 
        ) x 
  order by 1, 4 desc 


DEPTNO  ENAME            SAL HIREDATE    LATEST_SAL
------- --------- ---------- ----------- ----------
    10  MILLER          1300 23-JAN-2007       1300
    10  KING            5000 17-NOV-2006       1300
    10  CLARK           2450 09-JUN-2006       1300
    20  ADAMS           1100 12-JAN-2007       1100
    20  SCOTT           3000 09-DEC-2007       1100
    20  FORD            3000 03-DEC-2006       1100
    20  JONES           2975 02-APR-2006       1100
    20  SMITH            800 17-DEC-2005       1100
    30  JAMES            950 03-DEC-2006        950
    30  MARTIN          1250 28-SEP-2006        950
    30  TURNER          1500 08-SEP-2006        950
    30  BLAKE           2850 01-MAY-2006        950
    30  WARD            1250 22-FEB-2006        950
    30  ALLEN           1600 20-FEB-2006        950
```

### Oracle

The key to the Oracle solution is to take advantage of the KEEP clause. The KEEP clause allows you to rank the rows returned by a group/partition and work with the first or last row in the group. Consider what the solution looks like without KEEP:

```
select deptno, 
        ename, 
        sal, 
        hiredate, 
        max(sal) over(partition by deptno) latest_sal 
   from emp 
  order by 1, 4 desc 


DEPTNO ENAME             SAL HIREDATE    LATEST_SAL
------ ---------- ---------- ----------- ----------
    10 MILLER           1300 23-JAN-2007       5000
    10 KING             5000 17-NOV-2006       5000
    10 CLARK            2450 09-JUN-2006       5000
    20 ADAMS            1100 12-JAN-2007       3000
    20 SCOTT            3000 09-DEC-2007       3000
    20 FORD             3000 03-DEC-2006       3000
    20 JONES            2975 02-APR-2006       3000
    20 SMITH             800 17-DEC-2005       3000
    30 JAMES             950 03-DEC-2006       2850
    30 MARTIN           1250 28-SEP-2006       2850
    30 TURNER           1500 08-SEP-2006       2850
    30 BLAKE            2850 01-MAY-2006       2850
    30 WARD             1250 22-FEB-2006       2850
    30 ALLEN            1600 20-FEB-2006       2850
```

Rather than returning the SAL of the latest employee hired, MAX OVER without KEEP simply returns the highest salary in each DEPTNO. KEEP, in this recipe, allows you to order the salaries by HIREDATE in each DEPTNO by specifying ORDER BY HIREDATE. []()Then, the function DENSE\_RANK assigns a rank to each HIREDATE in ascending order. []()Finally, the function LAST determines which row to apply the aggregate function to: the “last” row based on the ranking of DENSE\_RANK. In this case, the aggregate function MAX is applied to the SAL column for the row with the “last” HIREDATE. In essence, keep the SAL of the HIREDATE ranked last in each DEPTNO.

You are ranking the rows in each DEPTNO based on one column (HIREDATE), but then applying the aggregation (MAX) on another column (SAL). This ability to rank in one dimension and aggregate over another is convenient as it allows you to avoid extra joins and inline views as are used in the other solutions. []()Finally, by adding the OVER clause after the KEEP clause, you can return the SAL “kept” by KEEP for each row in the partition.

Alternatively, you can order by HIREDATE in descending order and “keep” the first SAL. Compare the following two queries, which return the same result set:[]()[]()

```
select deptno, 
        ename, 
        sal, 
        hiredate, 
        max(sal) 
          keep(dense_rank last order by hiredate) 
          over(partition by deptno) latest_sal 
   from emp 
  order by 1, 4 desc 


DEPTNO ENAME             SAL HIREDATE    LATEST_SAL
------ ---------- ---------- ----------- ----------
    10 MILLER           1300 23-JAN-2007       1300
    10 KING             5000 17-NOV-2006       1300
    10 CLARK            2450 09-JUN-2006       1300
    20 ADAMS            1100 12-JAN-2007       1100
    20 SCOTT            3000 09-DEC-2007       1100
    20 FORD             3000 03-DEC-2006       1100
    20 JONES            2975 02-APR-2006       1100
    20 SMITH             800 17-DEC-2005       1100
    30 JAMES             950 03-DEC-2006        950
    30 MARTIN           1250 28-SEP-2006        950
    30 TURNER           1500 08-SEP-2006        950
    30 BLAKE            2850 01-MAY-2006        950
    30 WARD             1250 22-FEB-2006        950
    30 ALLEN            1600 20-FEB-2006        950


 select deptno, 
        ename, 
        sal, 
        hiredate, 
        max(sal) 
          keep(dense_rank first order by hiredate desc) 
          over(partition by deptno) latest_sal 
   from emp 
  order by 1, 4 desc 


DEPTNO ENAME             SAL HIREDATE    LATEST_SAL
------ ---------- ---------- ----------- ----------
    10 MILLER           1300 23-JAN-2007       1300
    10 KING             5000 17-NOV-2006       1300
    10 CLARK            2450 09-JUN-2006       1300
    20 ADAMS            1100 12-JAN-2007       1100
    20 SCOTT            3000 09-DEC-2007       1100
    20 FORD             3000 03-DEC-2006       1100
    20 JONES            2975 02-APR-2006       1100
    20 SMITH             800 17-DEC-2005       1100
    30 JAMES             950 03-DEC-2006        950
    30 MARTIN           1250 28-SEP-2006        950
    30 TURNER           1500 08-SEP-2006        950
    30 BLAKE            2850 01-MAY-2006        950
    30 WARD             1250 22-FEB-2006        950
    30 ALLEN            1600 20-FEB-2006        950
```

# 11.12 Generating Simple Forecasts

## Problem

[]()[]()Based on current data, you want to return additional rows and columns representing future actions. For example, consider the following result set:

```
ID ORDER_DATE  PROCESS_DATE
-- ----------- ------------
 1 25-SEP-2005  27-SEP-2005
 2 26-SEP-2005  28-SEP-2005
 3 27-SEP-2005  29-SEP-2005
```

You want to return three rows per row returned in your result set (each row plus two additional rows for each order). Along with the extra rows, you would like to return two additional columns providing dates for expected order processing.

From the previous result set, you can see that an order takes two days to process. For the purposes of this example, let’s say the next step after processing is verification, and the last step is shipment. Verification occurs one day after processing, and shipment occurs one day after verification. You want to return a result set expressing the whole procedure. Ultimately you want to transform the previous result set to the following result set:

```
ID ORDER_DATE  PROCESS_DATE  VERIFIED     SHIPPED
-- ----------- ------------  -----------  -----------
 1 25-SEP-2005  27-SEP-2005
 1 25-SEP-2005  27-SEP-2005  28-SEP-2005
 1 25-SEP-2005  27-SEP-2005  28-SEP-2005  29-SEP-2005
 2 26-SEP-2005  28-SEP-2005
 2 26-SEP-2005  28-SEP-2005  29-SEP-2005
 2 26-SEP-2005  28-SEP-2005  29-SEP-2005  30-SEP-2005
 3 27-SEP-2005  29-SEP-2005
 3 27-SEP-2005  29-SEP-2005  30-SEP-2005
 3 27-SEP-2005  29-SEP-2005  30-SEP-2005  01-OCT-2005
```

## Solution

The key is to use a Cartesian product to generate two additional rows for each order and then simply use CASE expressions to create the required column values.

### DB2, MySQL, and SQL Server

Use the recursive WITH clause to generate rows needed for your Cartesian product. The DB2 and SQL Server solutions are identical except for the function used to retrieve the current date. DB2 uses CURRENT\_DATE and SQL Server uses GET-DATE. MySQL uses the CURDATE and requires the insertion of the keyword RECURSIVE after WITH to indicate that this is a recursive CTE. The SQL Server solution is shown here:

```
 1  with nrows(n) as (
 2  select 1 from t1 union all
 3  select n+1 from nrows where n+1 <= 3
 4  )
 5  select id,
 6         order_date,
 7         process_date,
 8         case when nrows.n >= 2
 9              then process_date+1
10              else null
11         end as verified,
12         case when nrows.n = 3
13              then process_date+2
14              else null
15         end as shipped
16    from (
17  select nrows.n id,
18         getdate()+nrows.n   as order_date,
19         getdate()+nrows.n+2 as process_date
20    from nrows
21         ) orders, nrows
22   order by 1
```

### Oracle

[]()Use the hierarchical CONNECT BY clause to generate the three rows needed for the Cartesian product. Use the WITH clause to allow you to reuse the results returned by CONNECT BY without having to call it again:

```
 1  with nrows as (
 2  select level n
 3    from dual
 4  connect by level <= 3
 5  )
 6  select id,
 7         order_date,
 8         process_date,
 9         case when nrows.n >= 2
10              then process_date+1
11              else null
12         end as verified,
13         case when nrows.n = 3
14              then process_date+2
15              else null
16         end as shipped
17  from (
18 select nrows.n id,
19        sysdate+nrows.n as order_date,
20        sysdate+nrows.n+2 as process_date
21   from nrows
22        ) orders, nrows
```

### PostgreSQL

You can create a Cartesian product many different ways; this solution uses the PostgreSQL function GENERATE\_SERIES:

```
 1 select id,
 2        order_date,
 3        process_date,
 4        case when gs.n >= 2
 5             then process_date+1
 6             else null
 7        end as verified,
 8        case when gs.n = 3
 9             then process_date+2
10             else null
11        end as shipped
12  from (
13 select gs.id,
14        current_date+gs.id as order_date,
15        current_date+gs.id+2 as process_date
16   from generate_series(1,3) gs (id)
17        ) orders,
18          generate_series(1,3)gs(n)
```

### MySQL

MySQL does not support a function for automatic row generation.

## Discussion

### DB2, MySQL, and SQL Server

The result set presented in the “Problem” section is returned via inline view ORDERS, and is shown here:

```
with nrows(n) as (
select 1 from t1 union all
select n+1 from nrows where n+1 <= 3
)
select nrows.n id,getdate()+nrows.n   as order_date,
       getdate()+nrows.n+2 as process_date
  from nrows

ID ORDER_DATE  PROCESS_DATE
-- ----------- ------------
 1 25-SEP-2005  27-SEP-2005
 2 26-SEP-2005  28-SEP-2005
 3 27-SEP-2005  29-SEP-2005
```

This query simply uses the WITH clause to make up three rows representing the orders you must process. []()[]()NROWS returns the values 1, 2, and 3, and those numbers are added to GETDATE (CURRENT\_DATE for DB2, CURDATE() for MySQL) to represent the dates of the orders. Because the “Problem” section states that processing time takes two days, the query also adds two days to the ORDER\_DATE (adds the value returned by NROWS to GETDATE and then adds two more days).

Now that you have your base result set, the next step is to create a Cartesian product because the requirement is to return three rows for each order. []()Use NROWS to create a Cartesian product to return three rows for each order:

```
with nrows(n) as (
select 1 from t1 union all
select n+1 from nrows where n+1 <= 3
)
select nrows.n,
       orders.*
  from (
select nrows.n id,
       getdate()+nrows.n    as order_date,
        getdate()+nrows.n+2 as process_date
  from nrows
       ) orders, nrows
 order by 2,1

  N  ID  ORDER_DATE  PROCESS_DATE
--- ---  ----------- ------------
  1   1  25-SEP-2005  27-SEP-2005
  2   1  25-SEP-2005  27-SEP-2005
  3   1  25-SEP-2005  27-SEP-2005
  1   2  26-SEP-2005  28-SEP-2005
  2   2  26-SEP-2005  28-SEP-2005
  3   2  26-SEP-2005  28-SEP-2005
  1   3  27-SEP-2005  29-SEP-2005
  2   3  27-SEP-2005  29-SEP-2005
  3   3  27-SEP-2005  29-SEP-2005
```

Now that you have three rows for each order, simply use a CASE expression to create the addition column values to represent the status of verification and shipment.

The first row for each order should have a NULL value for VERIFIED and SHIPPED. The second row for each order should have a NULL value for SHIPPED. The third row for each order should have non-NULL values for each column. The final result set is shown here:

```
with nrows(n) as (
select 1 from t1 union all
select n+1 from nrows where n+1 <= 3
)
select id,
       order_date,
       process_date,
       case when nrows.n >= 2
            then process_date+1
            else null

       end as verified,
       case when nrows.n = 3
           then process_date+2
           else null
       end as shipped
  from (
select nrows.n id,
       getdate()+nrows.n   as order_date,
       getdate()+nrows.n+2 as process_date
  from nrows
       ) orders, nrows
 order by 1

ID ORDER_DATE  PROCESS_DATE  VERIFIED     SHIPPED
-- ----------- ------------  -----------  -----------
 1 25-SEP-2005  27-SEP-2005
 1 25-SEP-2005  27-SEP-2005  28-SEP-2005
 1 25-SEP-2005  27-SEP-2005  28-SEP-2005  29-SEP-2005
 2 26-SEP-2005  28-SEP-2005
 2 26-SEP-2005  28-SEP-2005  29-SEP-2005
 2 26-SEP-2005  28-SEP-2005  29-SEP-2005  30-SEP-2005
 3 27-SEP-2005  29-SEP-2005
 3 27-SEP-2005  29-SEP-2005  30-SEP-2005
 3 27-SEP-2005  29-SEP-2005  30-SEP-2005  01-OCT-2005
```

The final result set expresses the complete order process, from the day the order was received to the day it should be shipped.

### Oracle

The result set presented in the problem section is returned via inline view ORDERS and is shown here:

```
with nrows as (
select level n
  from dual
connect by level <= 3
)
select nrows.n id,
       sysdate+nrows.n order_date,
       sysdate+nrows.n+2 process_date
  from nrows

ID ORDER_DATE   PROCESS_DATE
-- -----------  ------------
 1 25-SEP-2005   27-SEP-2005
 2 26-SEP-2005   28-SEP-2005
 3 27-SEP-2005   29-SEP-2005
```

This query simply uses CONNECT BY to make up three rows representing the orders you must process. Use the WITH clause to refer to the rows returned by CONNECT BY as NROWS.N. CONNECT BY returns the values 1, 2, and 3, and those numbers are added to SYSDATE to represent the dates of the orders. Since the “Problem” section states that processing time takes two days, the query also adds two days to the ORDER\_DATE (adds the value returned by GENERATE_ SERIES to SYSDATE and then adds two more days).

Now that you have your base result set, the next step is to create a Cartesian product because the requirement is to return three rows for each order. Use NROWS to create a Cartesian product to return three rows for each order:

```
with nrows as (
select level n
  from dual
connect by level <= 3
)
select nrows.n,
       orders.*
  from (
select nrows.n id,
       sysdate+nrows.n order_date,
       sysdate+nrows.n+2 process_date
  from nrows
  ) orders, nrows

  N  ID ORDER_DATE  PROCESS_DATE
--- --- ----------- ------------
  1   1 25-SEP-2005  27-SEP-2005
  2   1 25-SEP-2005  27-SEP-2005
  3   1 25-SEP-2005  27-SEP-2005
  1   2 26-SEP-2005  28-SEP-2005
  2   2 26-SEP-2005  28-SEP-2005
  3   2 26-SEP-2005  28-SEP-2005
  1   3 27-SEP-2005  29-SEP-2005
  2   3 27-SEP-2005  29-SEP-2005
  3   3 27-SEP-2005  29-SEP-2005
```

Now that you have three rows for each order, simply use a CASE expression to create the addition column values to represent the status of verification and shipment.

The first row for each order should have a NULL value for VERIFIED and SHIPPED. The second row for each order should have a NULL value for SHIPPED. The third row for each order should have non-NULL values for each column. The final result set is shown here:

```
with nrows as (
select level n
  from dual
connect by level <= 3
)
select id,
       order_date,
       process_date,
       case when nrows.n >= 2
            then process_date+1
            else null
       end as verified,
       case when nrows.n = 3
            then process_date+2
            else null
       end as shipped
  from (
select nrows.n id,
       sysdate+nrows.n order_date,
       sysdate+nrows.n+2 process_date
  from nrows
       ) orders, nrows

 ID ORDER_DATE  PROCESS_DATE  VERIFIED     SHIPPED
 -- ----------- ------------  -----------  -----------
  1 25-SEP-2005  27-SEP-2005
  1 25-SEP-2005  27-SEP-2005  28-SEP-2005
  1 25-SEP-2005  27-SEP-2005  28-SEP-2005  29-SEP-2005
  2 26-SEP-2005  28-SEP-2005
  2 26-SEP-2005  28-SEP-2005  29-SEP-2005
  2 26-SEP-2005  28-SEP-2005  29-SEP-2005  30-SEP-2005
  3 27-SEP-2005  29-SEP-2005
  3 27-SEP-2005  29-SEP-2005  30-SEP-2005
  3 27-SEP-2005  29-SEP-2005  30-SEP-2005  01-OCT-2005
```

The final result set expresses the complete order process from the day the order was received to the day it should be shipped.

### PostgreSQL

The result set presented in the problem section is returned via inline view ORDERS and is shown here:

```
select gs.id,
       current_date+gs.id as order_date,
       current_date+gs.id+2 as process_date
 from generate_series(1,3) gs (id)

ID ORDER_DATE  PROCESS_DATE
-- ----------- ------------
 1 25-SEP-2005  27-SEP-2005
 2 26-SEP-2005  28-SEP-2005
 3 27-SEP-2005  29-SEP-2005
```

This query simply uses the GENERATE\_SERIES function to make up three rows representing the orders you must process. GENERATE\_SERIES returns the values 1, 2, and 3, and those numbers are added to CURRENT\_DATE to represent the dates of the orders. Since the “Problem” section states that processing time takes two days, the query also adds two days to the ORDER\_DATE (adds the value returned by GENERATE\_SERIES to CURRENT\_DATE and then adds two more days). Now that you have your base result set, the next step is to create a Cartesian product because the requirement is to return three rows for each order. Use the GENERATE_ SERIES function to create a Cartesian product to return three rows for each order:

```
select gs.n,
       orders.*
  from (
select gs.id,
       current_date+gs.id as order_date,
       current_date+gs.id+2 as process_date
  from generate_series(1,3) gs (id)
       ) orders,
         generate_series(1,3)gs(n)

  N ID  ORDER_DATE  PROCESS_DATE
--- --- ----------- ------------
  1   1 25-SEP-2005  27-SEP-2005
  2   1 25-SEP-2005  27-SEP-2005
  3   1 25-SEP-2005  27-SEP-2005
  1   2 26-SEP-2005  28-SEP-2005
  2   2 26-SEP-2005  28-SEP-2005
  3   2 26-SEP-2005  28-SEP-2005
  1   3 27-SEP-2005  29-SEP-2005
  2   3 27-SEP-2005  29-SEP-2005
  3   3 27-SEP-2005  29-SEP-2005
```

Now that you have three rows for each order, simply use a CASE expression to create the addition column values to represent the status of verification and shipment.

The first row for each order should have a NULL value for VERIFIED and SHIPPED. The second row for each order should have a NULL value for SHIPPED. The third row for each order should have non-NULL values for each column. The final result set is shown here:

```
select id,
       order_date,
       process_date,
       case when gs.n >= 2
            then process_date+1
            else null
       end as verified,
       case when gs.n = 3
            then process_date+2
            else null
       end as shipped
  from (
select gs.id,
       current_date+gs.id as order_date,
       current_date+gs.id+2 as process_date
  from generate_series(1,3) gs(id)
       ) orders,
         generate_series(1,3)gs(n)

ID ORDER_DATE  PROCESS_DATE  VERIFIED     SHIPPED
-- ----------- ------------  -----------  -----------
 1 25-SEP-2005 27-SEP-2005
 1 25-SEP-2005 27-SEP-2005   28-SEP-2005
 1 25-SEP-2005 27-SEP-2005   28-SEP-2005  29-SEP-2005
 2 26-SEP-2005 28-SEP-2005
 2 26-SEP-2005 28-SEP-2005   29-SEP-2005
 2 26-SEP-2005 28-SEP-2005   29-SEP-2005  30-SEP-2005
 3 27-SEP-2005 29-SEP-2005
 3 27-SEP-2005 29-SEP-2005   30-SEP-2005
 3 27-SEP-2005 29-SEP-2005   30-SEP-2005  01-OCT-2005
```

The final result set expresses the complete order process from the day the order was received to the day it should be shipped.[]()[]()

# 11.13 Summing Up

The recipes from this chapter represent practical problems that can’t be solved with a single function. They are some the kinds of problems that business users will frequently look to you to solve for them.[]()