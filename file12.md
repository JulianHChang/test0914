# Chapter 12. Reporting and Reshaping

[]()This chapter introduces queries you may find helpful for creating reports. These typically involve reporting-specific formatting considerations along with different levels of aggregation. Another focus of this chapter is transposing or pivoting result sets: reshaping the data by turning rows into columns.

In general, these recipes have in common that they allow you to present data in formats or shapes different from the way they are stored. As your comfort level with pivoting increases, you’ll undoubtedly find uses for it outside of what are presented in this chapter.

# 12.1 Pivoting a Result Set into One Row

## Problem

[]()[]()You want to take values from groups of rows and turn those values into columns in a single row per group. For example, you have a result set displaying the number of employees in each department:

```
DEPTNO        CNT
------ ----------
    10          3
    20          5
    30          6
```

You would like to reformat the output so that the result set looks as follows:

```
DEPTNO_10   DEPTNO_20   DEPTNO_30
---------  ----------  ----------
        3           5           6
```

This is a classic example of data presented in a different shape than the way it is stored.

## Solution

Transpose the result set using CASE and the aggregate function SUM:

```
1 select sum(case when deptno=10 then 1 else 0 end) as deptno_10,
2        sum(case when deptno=20 then 1 else 0 end) as deptno_20,
3        sum(case when deptno=30 then 1 else 0 end) as deptno_30
4   from emp
```

## Discussion

[]()This example is an excellent introduction to pivoting. The concept is simple: for each row returned by the unpivoted query, use a CASE expression to separate the rows into columns. Then, because this particular problem is to count the number of employees per department, use the aggregate function SUM to count the occurrence of each DEPTNO. If you’re having trouble understanding how this works exactly, execute the query with the aggregate function SUM and include DEPTNO for readability:

```
select deptno, 
        case when deptno=10 then 1 else 0 end as deptno_10, 
        case when deptno=20 then 1 else 0 end as deptno_20, 
        case when deptno=30 then 1 else 0 end as deptno_30 
   from emp 
  order by 1 

 DEPTNO   DEPTNO_10   DEPTNO_20   DEPTNO_30
 ------  ----------  ----------  ----------
     10           1           0           0
     10           1           0           0
     10           1           0           0
     20           0           1           0
     20           0           1           0
     20           0           1           0
     20           0           1           0
     30           0           0           1
     30           0           0           1
     30           0           0           1
     30           0           0           1
     30           0           0           1
     30           0           0           1
```

You can think of each CASE expression as a flag to determine which DEPTNO a row belongs to. At this point, the “rows to columns” transformation is already done; the next step is to simply sum the values returned by DEPTNO\_10, DEPTNO\_20, and DEPTNO\_30, and then to group by DEPTNO. The following are the results:

```
select deptno, 
        sum(case when deptno=10 then 1 else 0 end) as deptno_10, 
        sum(case when deptno=20 then 1 else 0 end) as deptno_20, 
        sum(case when deptno=30 then 1 else 0 end) as deptno_30 
   from emp 
  group by deptno 

DEPTNO   DEPTNO_10   DEPTNO_20   DEPTNO_30
------  ----------  ----------  ----------
    10           3           0           0
    20           0           5           0
    30           0           0           6
```

If you inspect this result set, you see that logically the output makes sense; for example, DEPTNO 10 has three employees in DEPTNO\_10 and zero in the other departments. Since the goal is to return one row, the last step is to remove the DEPTNO and GROUP BY clause and simply sum the CASE expressions:

```
select sum(case when deptno=10 then 1 else 0 end) as deptno_10, 
        sum(case when deptno=20 then 1 else 0 end) as deptno_20, 
        sum(case when deptno=30 then 1 else 0 end) as deptno_30 
   from emp 

  DEPTNO_10   DEPTNO_20   DEPTNO_30
  ---------  ----------  ----------
          3           5           6
```

The following is another approach that you may sometimes see applied to this same sort of problem:

```
select max(case when deptno=10 then empcount else null end) as deptno_10
       max(case when deptno=20 then empcount else null end) as deptno_20,
       max(case when deptno=10 then empcount else null end) as deptno_30
  from (
select deptno, count(*) as empcount
  from emp
 group by deptno
       ) x
```

This approach uses an inline view to generate the employee counts per department. CASE expressions in the main query translate rows to columns, getting you to the following results:[]()[]()

```
DEPTNO_10   DEPTNO_20   DEPTNO_30
---------  ----------  ----------
        3        NULL        NULL
     NULL           5        NULL
     NULL        NULL           6
```

Then the MAX functions collapses the columns into one row:

```
DEPTNO_10   DEPTNO_20   DEPTNO_30
---------  ----------  ----------
        3           5           6
```

# 12.2 Pivoting a Result Set into Multiple Rows

## Problem

[]()[]()You want to turn rows into columns by creating a column corresponding to each of the values in a single given column. However, unlike in the previous recipe, you need multiple rows of output. Like the earlier recipe, pivoting into multiple rows is a fundamental method of reshaping data.

For example, you want to return each employee and their position (JOB), and you currently use a query that returns the following result set:

```
JOB        ENAME
---------  ----------
ANALYST    SCOTT
ANALYST    FORD
CLERK      SMITH
CLERK      ADAMS
CLERK      MILLER
CLERK      JAMES
MANAGER    JONES
MANAGER    CLARK
MANAGER    BLAKE
PRESIDENT  KING
SALESMAN   ALLEN
SALESMAN   MARTIN
SALESMAN   TURNER
SALESMAN   WARD
```

You would like to format the result set such that each job gets its own column:

```
CLERKS  ANALYSTS  MGRS   PREZ  SALES
------  --------  -----  ----  ------
MILLER  FORD      CLARK  KING  TURNER
JAMES   SCOTT     BLAKE        MARTIN
ADAMS             JONES        WARD
SMITH                          ALLEN
```

## Solution

Unlike the first recipe in this chapter, the result set for this recipe consists of more than one row. Using the previous recipe’s technique will not work for this recipe, as the MAX(ENAME) for each JOB would be returned, which would result in one ENAME for each JOB (i.e., one row will be returned as in the first recipe). To solve this problem, you must make each JOB/ENAME combination unique. []()[]()Then, when you apply an aggregate function to remove NULLs, you don’t lose any ENAMEs.

[]()Use the ranking function ROW\_NUMBER OVER to make each JOB/ENAME combination unique. Pivot the result set using a CASE expression and the aggregate function MAX while grouping on the value returned by the window function:

```
 1  select max(case when job='CLERK'
 2                  then ename else null end) as clerks,
 3         max(case when job='ANALYST'
 4                  then ename else null end) as analysts,
 5         max(case when job='MANAGER'
 6                  then ename else null end) as mgrs,
 7         max(case when job='PRESIDENT'
 8                  then ename else null end) as prez,
 9         max(case when job='SALESMAN'
10                  then ename else null end) as sales
11   from (
12 select job,
13        ename,
14        row_number()over(partition by job order by ename) rn
15   from emp
16        ) x
17  group by rn
```

## Discussion

The first step is to use the window function ROW\_NUMBER OVER to help make each JOB/ENAME combination unique:

```
select job, 
        ename, 
        row_number()over(partition by job order by ename) rn 
   from emp 

  JOB       ENAME              RN
  --------- ---------- ----------
  ANALYST   FORD                1
  ANALYST   SCOTT               2
  CLERK     ADAMS               1
  CLERK     JAMES               2
  CLERK     MILLER              3
  CLERK     SMITH               4
  MANAGER   BLAKE               1
  MANAGER   CLARK               2
  MANAGER   JONES               3
  PRESIDENT KING                1
  SALESMAN  ALLEN               1
  SALESMAN  MARTIN              2
  SALESMAN  TURNER              3
  SALESMAN  WARD                4
```

Giving each ENAME a unique “row number” within a given job prevents any problems that might otherwise result from two employees having the same name and job. The goal here is to be able to group on row number (on RN) without dropping any employees from the result set due to the use of MAX. This step is the most important step in solving the problem. Without this first step, the aggregation in the outer query will remove necessary rows. Consider what the result set would look like without using ROW\_NUMBER OVER, using the same technique as shown in the first recipe:

```
select max(case when job='CLERK' 
                 then ename else null end) as clerks, 
        max(case when job='ANALYST' 
                 then ename else null end) as analysts, 
        max(case when job='MANAGER' 
                 then ename else null end) as mgrs, 
        max(case when job='PRESIDENT' 
                 then ename else null end) as prez, 
        max(case when job='SALESMAN' 
                 then ename else null end) as sales 
   from emp 


CLERKS      ANALYSTS    MGRS        PREZ        SALES
----------  ----------  ----------  ----------  ----------
SMITH       SCOTT       JONES       KING        WARD
```

Unfortunately, only one row is returned for each JOB: the employee with the MAX ENAME. []()[]()When it comes time to pivot the result set, using MIN or MAX should serve as a means to remove NULLs from the result set, not restrict the ENAMEs returned. How this works will be come clearer as you continue through the explanation.

The next step uses a CASE expression to organize the ENAMEs into their proper column (JOB):

```
select rn, 
        case when job='CLERK' 
             then ename else null end as clerks, 
        case when job='ANALYST' 
             then ename else null end as analysts, 
        case when job='MANAGER' 
             then ename else null end as mgrs, 
        case when job='PRESIDENT' 
             then ename else null end as prez, 
        case when job='SALESMAN' 
             then ename else null end as sales 
   from ( 
 select job, 
        ename, 
        row_number()over(partition by job order by ename) rn 
   from emp 
        ) x 

RN  CLERKS      ANALYSTS    MGRS        PREZ        SALES
--  ----------  ----------  ----------  ----------  ----------
 1              FORD
 2              SCOTT
 1  ADAMS
 2  JAMES
 3  MILLER
 4  SMITH
 1                          BLAKE
 2                          CLARK
 3                          JONES
 1                                      KING
 1                                                  ALLEN
 2                                                  MARTIN
 3                                                  TURNER
 4                                                  WARD
```

At this point, the rows are transposed into columns, and the last step is to remove the NULLs to make the result set more readable. To remove the NULLs, use the aggregate function MAX and group by RN. (You can use the function MIN as well. The choice to use MAX is arbitrary, as you will only ever be aggregating one value per group.) There is only one value for each RN/JOB/ENAME combination. Grouping by RN in conjunction with the CASE expressions embedded within the calls to MAX ensures that each call to MAX results in picking only one name from a group of otherwise NULL values:

```
select max(case when job='CLERK' 
                 then ename else null end) as clerks, 
        max(case when job='ANALYST' 
                 then ename else null end) as analysts, 
        max(case when job='MANAGER' 
                 then ename else null end) as mgrs, 
        max(case when job='PRESIDENT' 
                 then ename else null end) as prez, 
        max(case when job='SALESMAN' 
                 then ename else null end) as sales 
   from ( 
 select job, 
        ename, 
        row_number()over(partition by job order by ename) rn 
   from emp 
        ) x 
 group by rn 

CLERKS  ANALYSTS  MGRS   PREZ  SALES
------  --------  -----  ----  ------
MILLER  FORD      CLARK  KING  TURNER
JAMES   SCOTT     BLAKE        MARTIN
ADAMS             JONES        WARD
SMITH                          ALLEN
```

The technique of using ROW\_NUMBER OVER to create unique combinations of rows is extremely useful for formatting query results. Consider the following query that creates a sparse report showing employees by DEPTNO and JOB:

```
select deptno dno, job, 
        max(case when deptno=10 
                 then ename else null end) as d10, 
        max(case when deptno=20 
                 then ename else null end) as d20, 
        max(case when deptno=30 
                 then ename else null end) as d30, 
        max(case when job='CLERK' 
                 then ename else null end) as clerks, 
        max(case when job='ANALYST' 
                 then ename else null end) as anals, 
        max(case when job='MANAGER' 
                 then ename else null end) as mgrs, 
        max(case when job='PRESIDENT' 
                 then ename else null end) as prez, 
        max(case when job='SALESMAN' 
                 then ename else null end) as sales 
   from ( 
 Select deptno, 
        job, 
        ename, 
        row_number()over(partition by job order by ename) rn_job, 
        row_number()over(partition by deptno order by ename) rn_deptno 
   from emp 
        ) x 
  group by deptno, job, rn_deptno, rn_job 
  order by 1 

DNO JOB       D10    D20   D30    CLERKS ANALS MGRS  PREZ SALES
--- --------- ------ ----- ------ ------ ----- ----- ---- ------
 10 CLERK     MILLER              MILLER
 10 MANAGER   CLARK                            CLARK
 10 PRESIDENT KING                                   KING
 20 ANALYST          FORD                FORD
 20 ANALYST          SCOTT               SCOTT
 20 CLERK            ADAMS        ADAMS
 20 CLERK            SMITH        SMITH
 20 MANAGER          JONES                     JONES
 30 CLERK                  JAMES  JAMES
 30 MANAGER                BLAKE               BLAKE
 30 SALESMAN               ALLEN                          ALLEN
 30 SALESMAN               MARTIN                         MARTIN
 30 SALESMAN               TURNER                         TURNER
 30 SALESMAN               WARD                           WARD
```

By simply modifying what you group by (hence the nonaggregate items in the previous SELECT list), you can produce reports with different formats. It is worth the time of changing things around to understand how these formats change based on what you include in your GROUP BY clause.[]()[]()

# 12.3 Reverse Pivoting a Result Set

## Problem

[]()[]()[]()You want to transform columns to rows. Consider the following result set:

```
DEPTNO_10  DEPTNO_20  DEPTNO_30
---------- ---------- ----------
         3          5          6
```

You would like to convert that to the following:

```
DEPTNO COUNTS_BY_DEPT
------ --------------
    10              3
    20              5
    30              6
```

Some readers may have noticed that the first listing is the output from the first recipe in this chapter. To make this output available for this recipe, we can store it in a view with the following query:

```
create view emp_cnts as
(
select sum(case when deptno=10 then 1 else 0 end) as deptno_10,
          sum(case when deptno=20 then 1 else 0 end) as deptno_20,
          sum(case when deptno=30 then 1 else 0 end) as deptno_30
     from emp
)
```

In the solution and discussion that follow, the queries will refer to the EMP\_CNTS view created by the preceding query.

## Solution

Examining the desired result set, it’s easy to see that you can execute a simple COUNT and GROUP BY on table EMP to produce the desired result. The object here, though, is to imagine that the data is not stored as rows; perhaps the data is denormalized and aggregated values are stored as multiple columns.

To convert columns to rows, use a Cartesian product. You’ll need to know in advance how many columns you want to convert to rows because the table expression you use to create the Cartesian product must have a cardinality of at least the number of columns you want to transpose.

Rather than create a denormalized table of data, the solution for this recipe will use the solution from the first recipe of this chapter to create a “wide” result set. The full solution is as follows:

```
 1 select dept.deptno,
 2        case dept.deptno
 3             when 10 then emp_cnts.deptno_10
 4             when 20 then emp_cnts.deptno_20
 5             when 30 then emp_cnts.deptno_30
 6        end as counts_by_dept
 7   from emp_cnts cross join
 8        (select deptno from dept where deptno <= 30) dept
```

## Discussion

The view EMP\_CNTS represents the denormalized view, or “wide” result set that you want to convert to rows, and is shown here:

```
DEPTNO_10   DEPTNO_20   DEPTNO_30
---------  ----------  ----------
        3           5           6
```

Because there are three columns, you will create three rows. Begin by creating a Cartesian product between inline view EMP\_CNTS and some table expression that has at least three rows. The following code uses table DEPT to create the Cartesian product; DEPT has four rows:

```
select dept.deptno,
         emp_cnts.deptno_10,
         emp_cnts.deptno_20,
         emp_cnts.deptno_30
    from (
  Select sum(case when deptno=10 then 1 else 0 end) as deptno_10,
         sum(case when deptno=20 then 1 else 0 end) as deptno_20,
         sum(case when deptno=30 then 1 else 0 end) as deptno_30
    from emp
         ) emp_cnts,
         (select deptno from dept where deptno <= 30) dept

  DEPTNO DEPTNO_10  DEPTNO_20  DEPTNO_30
  ------ ---------- ---------- ---------
      10          3          5         6
      20          3          5         6
      30          3          5         6
```

The Cartesian product enables you to return a row for each column in inline view EMP\_CNTS. Since the final result set should have only the DEPTNO and the number of employees in said DEPTNO, use a CASE expression to transform the three columns into one:[]()[]()[]()

```
 select dept.deptno, 
          case dept.deptno 
               when 10 then emp_cnts.deptno_10 
               when 20 then emp_cnts.deptno_20 
               when 30 then emp_cnts.deptno_30 
          end as counts_by_dept 
     from ( 
          emp_cnts 
 cross join (select deptno from dept where deptno <= 30) dept 

  DEPTNO COUNTS_BY_DEPT
  ------ --------------
      10              3
      20              5
      30              6
```

# 12.4 Reverse Pivoting a Result Set into One Column

## Problem

[]()You want to return all columns from a query as just one column. For example, you want to return the ENAME, JOB, and SAL of all employees in DEPTNO 10, and you want to return all three values in one column. You want to return three rows for each employee and one row of white space between employees. You want to return the following result set:

```
EMPS
----------
CLARK
MANAGER
2450

KING
PRESIDENT
5000

MILLER
CLERK
1300
```

## Solution

The key is to use a recursive CTE combined with Cartesian product to return four rows for each employee. [Chapter 10](https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/ch10.html#sqlckbk-CHP-10) covers the recursive CTE we need, and it’s explored further in [Appendix B](https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/app02.html#sqlckbk-APP-B). Using the Cartesian join lets you return one column value per row and have an extra row for spacing between employees.

Use the window function ROW\_NUMBER OVER to rank each row based on EMPNO (1–4). Then use a CASE expression to transform three columns into one (the keyword RECURSIVE is needed after the first WITH in PostgreSQL and MySQL):

```
1   with four_rows (id)
2     as
3   (
4    select 1
5      union all
6    select id+1
7      from four_rows
8      where id < 4
9    )
10   ,
11    x_tab (ename,job,sal,rn )
12     as
13   (
      select  e.ename,e.job,e.sal,
14      row_number()over(partition by e.empno
15      order by e.empno)
16      from emp e
17      join four_rows on 1=1
18    )
19
20   select
21     case rn
22     when 1 then ename
23     when 2 then job
24     when 3 then cast(sal as char(4))
25    end emps
26  from x_tab
```

## Discussion

The first step is to use the window function ROW\_NUMBER OVER to create a ranking for each employee in DEPTNO 10:

```
select e.ename,e.job,e.sal,
       row_number()over(partition by e.empno
                  order by e.empno) rn
from emp e
 where e.deptno=10

ENAME       JOB             SAL         RN
---------- --------- ---------- ----------
CLARK      MANAGER         2450          1
KING       PRESIDENT       5000          1
MILLER     CLERK           1300          1
```

At this point, the ranking doesn’t mean much. You are partitioning by EMPNO, so the rank is 1 for all three rows in DEPTNO 10. Once you add the Cartesian product, the rank will begin to take shape, as shown in the following results:

```
with four_rows (id)
  as
  (select 1
  from dual
  union all
  select id+1
  from four_rows
  where id < 4
  )
 select e.ename,e.job,e.sal,
 row_number()over(partition by e.empno
 order by e.empno)
  from emp e
  join four_rows on 1=1

  ENAME      JOB              SAL         RN
  ---------- --------- ---------- ----------
  CLARK      MANAGER         2450          1
  CLARK      MANAGER         2450          2
  CLARK      MANAGER         2450          3
  CLARK      MANAGER         2450          4
  KING       PRESIDENT       5000          1
  KING       PRESIDENT       5000          2
  KING       PRESIDENT       5000          3
  KING       PRESIDENT       5000          4
  MILLER     CLERK           1300          1
  MILLER     CLERK           1300          2
  MILLER     CLERK           1300          3
  MILLER     CLERK           1300          4
```

You should stop at this point and understand two key points:

- RN is no longer 1 for each employee; it is now a repeating sequence of values from 1 to 4, the reason being that window functions are applied after the FROM and WHERE clauses are evaluated. So, partitioning by EMPNO causes the RN to reset to 1 when a new employee is encountered.
- We’ve used a recursive CTE to ensure that for each employee there are four rows. We don’t need the RECURSIVE keyword in SQL Server or DB2, but we do for Oracle, MySQL, and PostgreSQL.

The hard work is now done, and all that is left is to use a CASE expression to put ENAME, JOB, and SAL into one column for each employee (you need to use CAST to convert SAL to a string to keep CASE happy):[]()

```
 with four_rows (id)
  as
  (select 1
  union all
  select id+1
  from four_rows
  where id < 4
  )
  ,
  x_tab (ename,job,sal,rn )
  as
  (select e.ename,e.job,e.sal,
 row_number()over(partition by e.empno
 order by e.empno)
  from emp e
  join four_rows on 1=1)

   select case rn
  when 1 then ename
  when 2 then job
  when 3 then cast(sal as char(4))
 end emps
  from x_tab

  EMPS
  ----------
  CLARK
  MANAGER
  2450

  KING
  PRESIDENT
  5000

  MILLER
  CLERK
  1300
```

# 12.5 Suppressing Repeating Values from a Result Set

## Problem

[]()You are generating a report, and when two rows have the same value in a column, you want to display that value only once. For example, you want to return DEPTNO and ENAME from table EMP, you want to group all rows for each DEPTNO, and you want to display each DEPTNO only one time. You want to return the following result set:

```
DEPTNO ENAME
------ ---------
    10 CLARK
       KING
       MILLER
    20 SMITH
       ADAMS
       FORD
       SCOTT
       JONES
    30 ALLEN
       BLAKE
       MARTIN
       JAMES
       TURNER
       WARD
```

## Solution

[]()This is a simple formatting problem that is easily solved by the window function LAG OVER:

```
1   select
2           case when
3              lag(deptno)over(order by deptno) = deptno then null
4              else deptno end DEPTNO
5       , ename
6    from emp
```

Oracle users can also use DECODE as an alternative to CASE:

```
1 select to_number(
2           decode(lag(deptno)over(order by deptno),
3                 deptno,null,deptno)
4        ) deptno, ename
5   from emp
```

## Discussion

The first step is to use the window function LAG OVER to return the prior DEPTNO for each row:

```
select lag(deptno)over(order by deptno) lag_deptno,
       deptno,
       ename
  from emp

LAG_DEPTNO     DEPTNO ENAME
---------- ---------- ----------
                   10 CLARK
        10         10 KING
        10         10 MILLER
        10         20 SMITH
        20         20 ADAMS
        20         20 FORD
        20         20 SCOTT
        20         20 JONES
        20         30 ALLEN
        30         30 BLAKE
        30         30 MARTIN
        30         30 JAMES
        30         30 TURNER
        30         30 WARD
```

If you inspect the previous result set, you can easily see where DEPTNO matches LAG_ DEPTNO. For those rows, you want to set DEPTNO to NULL. Do that by using DECODE (TO\_NUMBER is included to cast DEPTNO as a number):[]()

```
select to_number( 
            CASE WHEN (lag(deptno)over(order by deptno)
= deptno THEN null else deptno END deptno , 
                  deptno,null,deptno) 
         ) deptno, ename 
   from emp 

DEPTNO ENAME
------ ----------
    10 CLARK
       KING
       MILLER
    20 SMITH
       ADAMS
       FORD
       SCOTT
       JONES
    30 ALLEN
       BLAKE
       MARTIN
       JAMES
       TURNER
       WARD
```

# 12.6 Pivoting a Result Set to Facilitate Inter-Row Calculations

## Problem

[]()[]()You want to make calculations involving data from multiple rows. To make your job easier, you want to pivot those rows into columns such that all values you need are then in a single row.

In this book’s example data, DEPTNO 20 is the department with the highest combined salary, which you can confirm by executing the following query:

```
select deptno, sum(sal) as sal 
   from emp 
  group by deptno 

DEPTNO        SAL
------ ----------
    10       8750
    20      10875
    30       9400
```

You want to calculate the difference between the salaries of DEPTNO 20 and DEPTNO 10 and between DEPTNO 20 and DEPTNO 30.

The final result will look like this:

```
d20_10_diff    d20_30_diff
------------   ----------
2125            1475
```

## Solution

Transpose the totals using the aggregate function SUM and a CASE expression. Then code your expressions in the select list:

```
1 select d20_sal - d10_sal as d20_10_diff,
2        d20_sal - d30_sal as d20_30_diff
3   from (
4 select sum(case when deptno=10 then sal end) as d10_sal,
5        sum(case when deptno=20 then sal end) as d20_sal,
6        sum(case when deptno=30 then sal end) as d30_sal
7   from emp
8        ) totals_by_dept
```

It is also possible to write this query using a CTE, which some people may find more readable:

```
with totals_by_dept (d10_sal, d20_sal, d30_sal)
as
(select
          sum(case when deptno=10 then sal end) as d10_sal,
           sum(case when deptno=20 then sal end) as d20_sal,
           sum(case when deptno=30 then sal end) as d30_sal

from emp)

select   d20_sal - d10_sal as d20_10_diff,
          d20_sal - d30_sal as d20_30_diff
     from totals_by_dept
```

## Discussion

The first step is to pivot the salaries for each DEPTNO from rows to columns by using a CASE expression:

```
select case when deptno=10 then sal end as d10_sal, 
        case when deptno=20 then sal end as d20_sal, 
        case when deptno=30 then sal end as d30_sal 
   from emp 

D10_SAL    D20_SAL    D30_SAL
------- ---------- ----------
               800
                         1600
                         1250
              2975
                         1250
                         2850
   2450
              3000
   5000
                         1500
              1100
                          950
              3000
   1300
```

The next step is to sum all the salaries for each DEPTNO by applying the aggregate function SUM to each CASE expression:

```
select sum(case when deptno=10 then sal end) as d10_sal, 
        sum(case when deptno=20 then sal end) as d20_sal, 
        sum(case when deptno=30 then sal end) as d30_sal 
   from emp 

D10_SAL    D20_SAL    D30_SAL
------- ---------- ----------
   8750      10875       9400
```

The final step is to simply wrap the previous SQL in an inline view and perform the subtractions.[]()[]()

# 12.7 Creating Buckets of Data, of a Fixed Size

## Problem

[]()You want to organize data into evenly sized buckets, with a predetermined number of elements in each bucket. The total number of buckets may be unknown, but you want to ensure that each bucket has five elements. For example, you want to organize the employees in table EMP into groups of five based on the value of EMPNO, as shown in the following results:

```
GRP      EMPNO ENAME
--- ---------- -------
  1       7369 SMITH
  1       7499 ALLEN
  1       7521 WARD
  1       7566 JONES
  1       7654 MARTIN
  2       7698 BLAKE
  2       7782 CLARK
  2       7788 SCOTT
  2       7839 KING
  2       7844 TURNER
  3       7876 ADAMS
  3       7900 JAMES
  3       7902 FORD
  3       7934 MILLER
```

## Solution

The solution to this problem is greatly simplified by functions for ranking rows. Once the rows are ranked, creating buckets of five is simply a matter of dividing and then taking the mathematical ceiling of the quotient.

Use the window function ROW\_NUMBER OVER to rank each employee by EMPNO. Then divide by five to create the groups (SQL Server users will use CEILING, not CEIL):

```
1 select ceil(row_number()over(order by empno)/5.0) grp,
2        empno,
3        ename
4   from emp
```

## Discussion

The window function ROW\_NUMBER OVER assigns a rank or “row number” to each row sorted by EMPNO:

```
select row_number()over(order by empno) rn, 
        empno, 
        ename 
   from emp 

RN      EMPNO ENAME
-- ---------- ----------
 1       7369 SMITH
 2       7499 ALLEN
 3       7521 WARD
 4       7566 JONES
 5       7654 MARTIN
 6       7698 BLAKE
 7       7782 CLARK
 8       7788 SCOTT
 9       7839 KING
10       7844 TURNER
11       7876 ADAMS
12       7900 JAMES
13       7902 FORD
14       7934 MILLER
```

[]()[]()The next step is to apply the function CEIL (or CEILING) after dividing ROW_ NUMBER OVER by five. Dividing by five logically organizes the rows into groups of five (i.e., five values less than or equal to 1, five values greater than 1 but less than or equal to 2); the remaining group (composed of the last 4 rows since 14, the number of rows in table EMP, is not a multiple of 5) has a value greater than 2 but less than or equal to 3.

The CEIL function will return the smallest whole number greater than the value passed to it; this will create whole number groups. The results of the division and application of the CEIL are shown here. You can follow the order of operation from left to right, from RN to DIVISION to GRP:[]()

```
select row_number()over(order by empno) rn, 
        row_number()over(order by empno)/5.0 division, 
        ceil(row_number()over(order by empno)/5.0) grp, 
        empno, 
        ename 
   from emp 

RN   DIVISION GRP EMPNO ENAME
-- ---------- --- ----- ----------
 1         .2   1  7369 SMITH
 2         .4   1  7499 ALLEN
 3         .6   1  7521 WARD
 4         .8   1  7566 JONES
 5          1   1  7654 MARTIN
 6        1.2   2  7698 BLAKE
 7        1.4   2  7782 CLARK
 8        1.6   2  7788 SCOTT
 9        1.8   2  7839 KING
10          2   2  7844 TURNER
11        2.2   3  7876 ADAMS
12        2.4   3  7900 JAMES
13        2.6   3  7902 FORD
14        2.8   3  7934 MILLER
```

# 12.8 Creating a Predefined Number of Buckets

## Problem

[]()You want to organize your data into a fixed number of buckets. For example, you want to organize the employees in table EMP into four buckets. The result set should look similar to the following:

```
GRP EMPNO ENAME
--- ----- ---------
  1  7369 SMITH
  1  7499 ALLEN
  1  7521 WARD
  1  7566 JONES
  2  7654 MARTIN
  2  7698 BLAKE
  2  7782 CLARK
  2  7788 SCOTT
  3  7839 KING
  3  7844 TURNER
  3  7876 ADAMS
  4  7900 JAMES
  4  7902 FORD
  4  7934 MILLER
```

This is a common way to organize categorical data as dividing a set into a number of smaller equal sized sets is an important first step for many kinds of analysis. For example, taking the averages of these groups on salary or any other value may reveal a trend that is concealed by variability when looking at the cases individually.

This problem is the opposite of the previous recipe, where you had an unknown number of buckets but a predetermined number of elements in each bucket. In this recipe, the goal is such that you may not necessarily know how many elements are in each bucket, but you are defining a fixed (known) number of buckets to be created.

## Solution

[]()The solution to this problem is simple now that the NTILE function is widely available. NTILE organizes an ordered set into the number of buckets you specify, with any stragglers distributed into the available buckets starting from the first bucket. The desired result set for this recipe reflects this: buckets 1 and 2 have four rows, while buckets 3 and 4 have three rows.

Use the NTILE window function to create four buckets:

```
1 select ntile(4)over(order by empno) grp,
2        empno,
3        ename
4   from emp
```

## Discussion

All the work is done by the NTILE function. The ORDER BY clause puts the rows into the desired order, and the function itself then assigns a group number to each row, for example, so that the first quarter (in this case) are put into group one, the second into group two, etc.

# 12.9 Creating Horizontal Histograms

## Problem

[]()[]()You want to use SQL to generate histograms that extend horizontally. For example, you want to display the number of employees in each department as a horizontal histogram with each employee represented by an instance of \*. You want to return the following result set:

```
DEPTNO CNT
------ ----------
    10 ***
    20 *****
    30 ******
```

## Solution

The key to this solution is to use the aggregate function COUNT and use GROUP BY DEPTNO to determine the number of employees in each DEPTNO. The value returned by COUNT is then passed to a string function that generates a series of * characters.

### DB2

[]()Use the REPEAT function to generate the histogram:

```
1 select deptno,
2        repeat('*',count(*)) cnt
3   from emp
4  group by deptno
```

### Oracle, PostgreSQL, and MySQL

[]()Use the LPAD function to generate the needed strings of * characters:

```
1 select deptno,
2        lpad('*',count(*),'*') as cnt
3   from emp
4  group by deptno
```

### SQL Server

[]()Generate the histogram using the REPLICATE function:

```
1 select deptno,
2        replicate('*',count(*)) cnt
3   from emp
4  group by deptno
```

## Discussion

The technique is the same for all vendors. The only difference lies in the string function used to return a * for each employee. The Oracle solution will be used for this discussion, but the explanation is relevant for all the solutions.

The first step is to count the number of employees in each department:

```
select deptno, 
        count(*) 
   from emp 
  group by deptno 

DEPTNO   COUNT(*)
------ ----------
    10          3
    20          5
    30          6
```

The next step is to use the value returned by COUNT to control the number of * characters to return for each department. Simply pass COUNT( * ) as an argument to the string function LPAD to return the desired number of \*:

```
select deptno, 
        lpad('*',count(*),'*') as cnt 
   from emp 
  group by deptno 

DEPTNO CNT
------ ----------
    10 ***
    20 *****
    30 ******
```

For PostgreSQL users, you may need to use CAST to ensure that COUNT(\*) returns an integer as shown here:

```
select deptno, 
        lpad('*',count(*)::integer,'*') as cnt 
   from emp 
  group by deptno 

DEPTNO CNT
------ ----------
    10 ***
    20 *****
    30 ******
```

This CAST is necessary because PostgreSQL requires the numeric argument to LPAD to be an integer.[]()[]()

# 12.10 Creating Vertical Histograms

## Problem

[]()[]()You want to generate a histogram that grows from the bottom up. For example, you want to display the number of employees in each department as a vertical histogram with each employee represented by an instance of \*. You want to return the following result set:

```
D10 D20 D30
--- --- ---
        *
    *   *
    *   *
*   *   *
*   *   *
*   *   *
```

## Solution

The technique used to solve this problem is built on a technique used earlier in this chapter: use the ROW\_NUMBER OVER function to uniquely identify each instance of * for each DEPTNO. Use the aggregate function MAX to pivot the result set and group by the values returned by ROW\_NUMBER OVER (SQL Server users should not use DESC in the ORDER BY clause):

```
 1 select max(deptno_10) d10,
 2        max(deptno_20) d20,
 3        max(deptno_30) d30
 4   from (
 5 select row_number()over(partition by deptno order by empno) rn,
 6        case when deptno=10 then '*' else null end deptno_10,
 7        case when deptno=20 then '*' else null end deptno_20,
 8        case when deptno=30 then '*' else null end deptno_30
 9   from emp
10        ) x
11  group by rn
12  order by 1 desc, 2 desc, 3 desc
```

## Discussion

The first step is to use the window function ROW\_NUMBER to uniquely identify each instance of * in each department. Use a CASE expression to return a * for each employee in each department:

```
select row_number()over(partition by deptno order by empno) rn, 
        case when deptno=10 then '*' else null end deptno_10, 
        case when deptno=20 then '*' else null end deptno_20, 
        case when deptno=30 then '*' else null end deptno_30 
   from emp 

RN DEPTNO_10  DEPTNO_20  DEPTNO_30
-- ---------- ---------- ---------
 1 *
 2 *
 3 *
 1            *
 2            *
 3            *
 4            *
 5            *
 1                       *
 2                       *
 3                       *
 4                       *
 5                       *
 6                       *
```

The next and last step is to use the aggregate function MAX on each CASE expression, grouping by RN to remove the NULLs from the result set. Order the results ASC or DESC depending on how your RDBMS sorts NULLs:[]()[]()

```
select max(deptno_10) d10, 
        max(deptno_20) d20, 
        max(deptno_30) d30 
   from ( 
 select row_number()over(partition by deptno order by empno) rn, 
        case when deptno=10 then '*' else null end deptno_10, 
        case when deptno=20 then '*' else null end deptno_20, 
        case when deptno=30 then '*' else null end deptno_30 
   from emp 
        ) x 
  group by rn 
  order by 1 desc, 2 desc, 3 desc 

D10 D20 D30
--- --- ---
        *
    *   *
    *   *
*   *   *
*   *   *
*   *   *
```

# 12.11 Returning Non-GROUP BY Columns

## Problem

[]()[]()You are executing a GROUP BY query, and you want to return columns in your select list that are not also listed in your GROUP BY clause. This is not normally possible, as such ungrouped columns would not represent a single value per row.

Say that you want to find the employees who earn the highest and lowest salaries in each department, as well as the employees who earn the highest and lowest salaries in each job. You want to see each employee’s name, the department he works in, his job title, and his salary. You want to return the following result set:

```
DEPTNO ENAME  JOB         SAL DEPT_STATUS     JOB_STATUS
------ ------ --------- ----- --------------- --------------
    10 MILLER CLERK      1300 LOW SAL IN DEPT TOP SAL IN JOB
    10 CLARK  MANAGER    2450                 LOW SAL IN JOB
    10 KING   PRESIDENT  5000 TOP SAL IN DEPT TOP SAL IN JOB
    20 SCOTT  ANALYST    3000 TOP SAL IN DEPT TOP SAL IN JOB
    20 FORD   ANALYST    3000 TOP SAL IN DEPT TOP SAL IN JOB
    20 SMITH  CLERK       800 LOW SAL IN DEPT LOW SAL IN JOB
    20 JONES  MANAGER    2975                 TOP SAL IN JOB
    30 JAMES  CLERK       950 LOW SAL IN DEPT
    30 MARTIN SALESMAN   1250                 LOW SAL IN JOB
    30 WARD   SALESMAN   1250                 LOW SAL IN JOB
    30 ALLEN  SALESMAN   1600                 TOP SAL IN JOB
    30 BLAKE  MANAGER    2850 TOP SAL IN DEPT
```

Unfortunately, including all these columns in the SELECT clause will ruin the grouping. Consider the following example: employee KING earns the highest salary. You want to verify this with the following query:

```
select ename,max(sal)
  from empgroup by ename
```

Instead of seeing KING and KING’s salary, the previous query will return all 14 rows from table EMP. The reason is because of the grouping: the MAX(SAL) is applied to each ENAME. So, it would seem the previous query can be stated as “find the employee with the highest salary,” but in fact what it is doing is “find the highest salary for each ENAME in table EMP.” This recipe explains a technique for including ENAME without the need to GROUP BY that column.

## Solution

Use an inline view to find the high and low salaries by DEPTNO and JOB. Then keep only the employees who make those salaries.

Use the window functions MAX OVER and MIN OVER to find the highest and lowest salaries by DEPTNO and JOB. Then keep the rows where the salaries are those that are highest or lowest by DEPTNO or JOB:

```
 1 select deptno,ename,job,sal,
 2        case when sal = max_by_dept
 3             then 'TOP SAL IN DEPT'
 4             when sal = min_by_dept
 5             then 'LOW SAL IN DEPT'
 6        end dept_status,
 7        case when sal = max_by_job
 8             then 'TOP SAL IN JOB'
 9             when sal = min_by_job
10             then 'LOW SAL IN JOB'
11        end job_status
12   from (
13 select deptno,ename,job,sal,
14        max(sal)over(partition by deptno) max_by_dept,
15        max(sal)over(partition by job)   max_by_job,
16        min(sal)over(partition by deptno) min_by_dept,
17        min(sal)over(partition by job)   min_by_job
18   from emp
19        ) emp_sals
20  where sal in (max_by_dept,max_by_job,
21                min_by_dept,min_by_job)
```

## Discussion

The first step is to use the window functions MAX OVER and MIN OVER to find the highest and lowest salaries by DEPTNO and JOB:

```
select deptno,ename,job,sal, 
        max(sal)over(partition by deptno) maxDEPT, 
        max(sal)over(partition by job) maxJOB, 
        min(sal)over(partition by deptno) minDEPT, 
        min(sal)over(partition by job) minJOB 
   from emp 

DEPTNO ENAME  JOB         SAL MAXDEPT MAXJOB MINDEPT MINJOB
------ ------ --------- ----- ------- ------ ------- ------
    10 MILLER CLERK      1300    5000   1300    1300    800
    10 CLARK  MANAGER    2450    5000   2975    1300   2450
    10 KING   PRESIDENT  5000    5000   5000    1300   5000
    20 SCOTT  ANALYST    3000    3000   3000     800   3000
    20 FORD   ANALYST    3000    3000   3000     800   3000
    20 SMITH  CLERK       800    3000   1300     800    800
    20 JONES  MANAGER    2975    3000   2975     800   2450
    20 ADAMS  CLERK      1100    3000   1300     800    800
    30 JAMES  CLERK       950    2850   1300     950    800
    30 MARTIN SALESMAN   1250    2850   1600     950   1250
    30 TURNER SALESMAN   1500    2850   1600     950   1250
    30 WARD   SALESMAN   1250    2850   1600     950   1250
    30 ALLEN  SALESMAN   1600    2850   1600     950   1250
    30 BLAKE  MANAGER    2850    2850   2975     950   2450
```

At this point, every salary can be compared with the highest and lowest salaries by DEPTNO and JOB. Notice that the grouping (the inclusion of multiple columns in the SELECT clause) does not affect the values returned by MIN OVER and MAX OVER. []()This is the beauty of window functions: the aggregate is computed over a defined “group” or partition and returns multiple rows for each group. The last step is to simply wrap the window functions in an inline view and keep only those rows that match the values returned by the window functions. Use a simple CASE expression to display the “status” of each employee in the final result set:[]()[]()

```
select deptno,ename,job,sal, 
        case when sal = max_by_dept 
             then 'TOP SAL IN DEPT' 
             when sal = min_by_dept 
             then 'LOW SAL IN DEPT' 
        end dept_status, 
        case when sal = max_by_job 
             then 'TOP SAL IN JOB' 
             when sal = min_by_job 
             then 'LOW SAL IN JOB' 
        end job_status 
   from ( 
 select deptno,ename,job,sal, 
        max(sal)over(partition by deptno) max_by_dept, 
        max(sal)over(partition by job) max_by_job, 
        min(sal)over(partition by deptno) min_by_dept, 
        min(sal)over(partition by job) min_by_job 
   from emp 
        ) x 
  where sal in (max_by_dept,max_by_job, 
                min_by_dept,min_by_job) 

DEPTNO ENAME  JOB         SAL DEPT_STATUS     JOB_STATUS
------ ------ --------- ----- --------------- --------------
    10 MILLER CLERK      1300 LOW SAL IN DEPT TOP SAL IN JOB
    10 CLARK  MANAGER    2450                 LOW SAL IN JOB
    10 KING   PRESIDENT  5000 TOP SAL IN DEPT TOP SAL IN JOB
    20 SCOTT  ANALYST    3000 TOP SAL IN DEPT TOP SAL IN JOB
    20 FORD   ANALYST    3000 TOP SAL IN DEPT TOP SAL IN JOB
    20 SMITH  CLERK       800 LOW SAL IN DEPT LOW SAL IN JOB
    20 JONES  MANAGER    2975                 TOP SAL IN JOB
    30 JAMES  CLERK       950 LOW SAL IN DEPT
    30 MARTIN SALESMAN   1250                 LOW SAL IN JOB
    30 WARD   SALESMAN   1250                 LOW SAL IN JOB
    30 ALLEN  SALESMAN   1600                 TOP SAL IN JOB
    30 BLAKE  MANAGER    2850 TOP SAL IN DEPT
```

# 12.12 Calculating Simple Subtotals

## Problem

[]()[]()[]()For the purposes of this recipe, a *simple subtotal* is defined as a result set that contains values from the aggregation of one column along with a grand total value for the table. An example would be a result set that sums the salaries in table EMP by JOB and that also includes the sum of all salaries in table EMP. The summed salaries by JOB are the subtotals, and the sum of all salaries in table EMP is the grand total. Such a result set should look as follows:

```
JOB              SAL
--------- ----------
ANALYST         6000
CLERK           4150
MANAGER         8275
PRESIDENT       5000
SALESMAN        5600
TOTAL          29025
```

## Solution

[]()The ROLLUP extension to the GROUP BY clause solves this problem perfectly. If ROLLUP is not available for your RDBMS, you can solve the problem, albeit with more difficulty, using a scalar subquery or a UNION query.

### DB2 and Oracle

Use the aggregate function SUM to sum the salaries, and use the ROLLUP extension of GROUP BY to organize the results into subtotals (by JOB) and a grand total (for the whole table):

```
1 select case grouping(job)
2             when 0 then job
3             else 'TOTAL'
4        end job,
5        sum(sal) sal
6   from emp
7  group by rollup(job)
```

### SQL Server and MySQL

[]()Use the aggregate function SUM to sum the salaries, and use WITH ROLLUP to organize the results into subtotals (by JOB) and a grand total (for the whole table). Then use COALESCE to supply the label TOTAL for the grand total row (which will otherwise have a NULL in the JOB column):

```
1 select coalesce(job,'TOTAL') job,
2        sum(sal) sal
3   from emp
4  group by job with rollup
```

[]()With SQL Server, you also have the option to use the GROUPING function shown in the Oracle/DB2 recipe rather than COALESCE to determine the level of aggregation.

### PostgreSQL

Similar to the SQL Server and MySQL solutions, you use the ROLLUP extension to GROUP BY with slightly different syntax:

```
select coalesce(job,'TOTAL') job,
        sum(sal) sal
   from emp
  group by rollup(job)
```

## Discussion

### DB2 and Oracle

The first step is to use the aggregate function SUM, grouping by JOB in order to sum the salaries by JOB:

```
select job, sum(sal) sal 
   from emp 
  group by job 

JOB         SAL
--------- -----
ANALYST    6000
CLERK      4150
MANAGER    8275
PRESIDENT  5000
SALESMAN   5600
```

The next step is to use the ROLLUP extension to GROUP BY to produce a grand total for all salaries along with the subtotals for each JOB:

```
select job, sum(sal) sal 
   from emp 
  group by rollup(job) 

JOB           SAL
--------- -------
ANALYST      6000
CLERK        4150
MANAGER      8275
PRESIDENT    5000
SALESMAN     5600
            29025
```

The last step is to use the GROUPING function in the JOB column to display a label for the grand total. If the value of JOB is NULL, the GROUPING function will return 1, which signifies that the value for SAL is the grand total created by ROLLUP. If the value of JOB is not NULL, the GROUPING function will return 0, which signifies the value for SAL is the result of the GROUP BY, not the ROLLUP. Wrap the call to GROUPING(JOB) in a CASE expression that returns either the job name or the label TOTAL, as appropriate:

```
select case grouping(job) 
             when 0 then job 
             else 'TOTAL' 
        end job, 
        sum(sal) sal 
   from emp 
  group by rollup(job) 


JOB              SAL
--------- ----------
ANALYST         6000
CLERK           4150
MANAGER         8275
PRESIDENT       5000
SALESMAN        5600
TOTAL          29025
```

### SQL Server and MySQL

The first step is to use the aggregate function SUM, grouping the results by JOB to generate salary sums by JOB:

```
select job, sum(sal) sal 
   from emp 
  group by job 


JOB         SAL
--------- -----
ANALYST    6000
CLERK      4150
MANAGER    8275
PRESIDENT  5000
SALESMAN   5600
```

The next step is to use GROUP BY’s ROLLUP extension to produce a grand total for all salaries along with the subtotals for each JOB:

```
select job, sum(sal) sal 
   from emp 
  group by job with rollup 

JOB           SAL
--------- -------
ANALYST      6000
CLERK        4150
MANAGER      8275
PRESIDENT    5000
SALESMAN     5600
            29025
```

The last step is to use the COEALESCE function against the JOB column. If the value of JOB is NULL, the value for SAL is the grand total created by ROLLUP. If the value of JOB is not NULL, the value for SAL is the result of the “regular” GROUP BY, not the ROLLUP:

```
select coalesce(job,'TOTAL') job, 
        sum(sal) sal 
   from emp 
  group by job with rollup 

JOB              SAL
--------- ----------
ANALYST         6000
CLERK           4150
MANAGER         8275
PRESIDENT       5000
SALESMAN        5600
TOTAL          29025
```

### PostgreSQL

The solution is the same in its manner of operation as the preceeding solution for MySQL and SQL Server. The only difference is the syntax for the ROLLUP clause: write ROLLUP(JOB) after GROUP BY.[]()[]()[]()

# 12.13 Calculating Subtotals for All Possible Expression Combinations

## Problem

[]()[]()[]()You want to find the sum of all salaries by DEPTNO, and by JOB, for every JOB/ DEPTNO combination. You also want a grand total for all salaries in table EMP. You want to return the following result set:

```
DEPTNO JOB       CATEGORY                  SAL
------ --------- --------------------- -------
    10 CLERK     TOTAL BY DEPT AND JOB    1300
    10 MANAGER   TOTAL BY DEPT AND JOB    2450
    10 PRESIDENT TOTAL BY DEPT AND JOB    5000
    20 CLERK     TOTAL BY DEPT AND JOB    1900
    30 CLERK     TOTAL BY DEPT AND JOB     950
    30 SALESMAN  TOTAL BY DEPT AND JOB    5600
    30 MANAGER   TOTAL BY DEPT AND JOB    2850
    20 MANAGER   TOTAL BY DEPT AND JOB    2975
    20 ANALYST   TOTAL BY DEPT AND JOB    6000
       CLERK     TOTAL BY JOB             4150
       ANALYST   TOTAL BY JOB             6000
       MANAGER   TOTAL BY JOB             8275
       PRESIDENT TOTAL BY JOB             5000
       SALESMAN  TOTAL BY JOB             5600
    10           TOTAL BY DEPT            8750
    30           TOTAL BY DEPT            9400
    20           TOTAL BY DEPT           10875
                 GRAND TOTAL FOR TABLE   29025
```

## Solution

Extensions added to GROUP BY in recent years make this a fairly easy problem to solve. If your platform does not supply such extensions for computing various levels of subtotals, then you must compute them manually (via self-joins or scalar subqueries).

### DB2

For DB2, you will need to use CAST to return from GROUPING as the CHAR(1) data type:

```
 1 select deptno,
 2        job,
 3        case cast(grouping(deptno) as char(1))||
 4             cast(grouping(job) as char(1))
 5             when '00' then 'TOTAL BY DEPT AND JOB'
 6             when '10' then 'TOTAL BY JOB'
 7             when '01' then 'TOTAL BY DEPT'
 8             when '11' then 'TOTAL FOR TABLE'
 9        end category,
10        sum(sal)
11   from emp
12  group by cube(deptno,job)
13  order by grouping(job),grouping(deptno)
```

### Oracle

[]()Use the CUBE extension to the GROUP BY clause with the concatenation operator ||:

```
 1 select deptno,
 2        job,
 3        case grouping(deptno)||grouping(job)
 4             when '00' then 'TOTAL BY DEPT AND JOB'
 5             when '10' then 'TOTAL BY JOB'
 6             when '01' then 'TOTAL BY DEPT'
 7             when '11' then 'GRAND TOTALFOR TABLE'
 8        end category,
 9        sum(sal) sal
10   from emp
11  group by cube(deptno,job)
12  order by grouping(job),grouping(deptno)
```

### SQL Server

Use the CUBE extension to the GROUP BY clause. For SQL Server, you will need to CAST the results from GROUPING to CHAR(1), and you will need to use the + operator for concatenation (as opposed to Oracle’s || operator):

```
 1 select deptno,
 2        job,
 3        case cast(grouping(deptno)as char(1))+
 4             cast(grouping(job)as char(1))
 5             when '00' then 'TOTAL BY DEPT AND JOB'
 6             when '10' then 'TOTAL BY JOB'
 7             when '01' then 'TOTAL BY DEPT'
 8             when '11' then 'GRAND TOTAL FOR TABLE'
 9        end category,
10        sum(sal) sal
11   from emp
12  group by deptno,job with cube
13  order by grouping(job),grouping(deptno)
```

### PostgreSQL

PostgreSQL is similar to the preceding, but with slightly different syntax for the CUBE operator and the concatenation:

```
select deptno,job
,case concat(
cast (grouping(deptno) as char(1)),cast (grouping(job) as char(1))
  )
  when '00' then 'TOTAL BY DEPT AND JOB'
               when '10' then 'TOTAL BY JOB'
               when '01' then 'TOTAL BY DEPT'
               when '11' then 'GRAND TOTAL FOR TABLE'
          end category
  , sum(sal) as sal
    from emp
    group by cube(deptno,job)
```

### MySQL

Although part of the functionality is available, it is not complete, as MySQL has no CUBE function. Hence, use multiple UNION ALLs, creating different sums for each:

```
 1 select deptno, job,
 2        'TOTAL BY DEPT AND JOB' as category,
 3        sum(sal) as sal
 4   from emp
 5  group by deptno, job
 6  union all
 7 select null, job, 'TOTAL BY JOB', sum(sal)
 8   from emp
 9  group by job
10  union all
11 select deptno, null, 'TOTAL BY DEPT', sum(sal)
12   from emp
13  group by deptno
14  union all
15 select null,null,'GRAND TOTAL FOR TABLE', sum(sal)
16  from emp
```

## Discussion

### Oracle, DB2, and SQL Server

The solutions for all three are essentially the same. The first step is to use the aggregate function SUM and group by both DEPTNO and JOB to find the total salaries for each JOB and DEPTNO combination:

```
select deptno, job, sum(sal) sal 
   from emp 
  group by deptno, job 

DEPTNO JOB           SAL
------ --------- -------
    10 CLERK        1300
    10 MANAGER      2450
    10 PRESIDENT    5000
    20 CLERK        1900
    20 ANALYST      6000
    20 MANAGER      2975
    30 CLERK         950
    30 MANAGER      2850
    30 SALESMAN     5600
```

The next step is to create subtotals by JOB and DEPTNO along with the grand total for the whole table. Use the CUBE extension to the GROUP BY clause to perform aggregations on SAL by DEPTNO, JOB, and for the whole table:

```
select deptno, 
        job, 
        sum(sal) sal 
   from emp 
  group by cube(deptno,job) 


DEPTNO JOB           SAL
------ --------- -------
                   29025
       CLERK        4150
       ANALYST      6000
       MANAGER      8275
       SALESMAN     5600
       PRESIDENT    5000
    10              8750
    10 CLERK        1300
    10 MANAGER      2450
    10 PRESIDENT    5000
    20             10875
    20 CLERK        1900
    20 ANALYST      6000
    20 MANAGER      2975
    30              9400
    30 CLERK         950
    30 MANAGER      2850
    30 SALESMAN     5600
```

Next, use the GROUPING function in conjunction with CASE to format the results into more meaningful output. The value from GROUPING(JOB) will be 1 or 0 depending on whether the values for SAL are due to the GROUP BY or the CUBE. If the results are due to the CUBE, the value will be 1; otherwise, it will be 0. The same goes for GROUPING(DEPTNO). Looking at the first step of the solution, you should see that grouping is done by DEPTNO and JOB. Thus, the expected values from the calls to GROUPING when a row represents a combination of both DEPTNO and JOB is 0. The following query confirms this:

```
select deptno, 
        job, 
        grouping(deptno) is_deptno_subtotal, 
        grouping(job) is_job_subtotal, 
        sum(sal) sal 
   from emp 
  group by cube(deptno,job) 
  order by 3,4 

DEPTNO JOB       IS_DEPTNO_SUBTOTAL IS_JOB_SUBTOTAL     SAL
------ --------- ------------------ --------------- -------
    10 CLERK                      0               0    1300
    10 MANAGER                    0               0    2450
    10 PRESIDENT                  0               0    5000
    20 CLERK                      0               0    1900
    30 CLERK                      0               0     950
    30 SALESMAN                   0               0    5600
    30 MANAGER                    0               0    2850
    20 MANAGER                    0               0    2975
    20 ANALYST                    0               0    6000
    10                            0               1    8750
    20                            0               1   10875
    30                            0               1    9400
       CLERK                      1               0    4150
       ANALYST                    1               0    6000
       MANAGER                    1               0    8275
       PRESIDENT                  1               0    5000
       SALESMAN                   1               0    5600
                                  1               1   29025
```

The final step is to use a CASE expression to determine which category each row belongs to based on the values returned by GROUPING(JOB) and GROUPING(DEPTNO) concatenated:

```
select deptno, 
        job, 
        case grouping(deptno)||grouping(job) 
             when '00' then 'TOTAL BY DEPT AND JOB' 
             when '10' then 'TOTAL BY JOB' 
             when '01' then 'TOTAL BY DEPT' 
             when '11' then 'GRAND TOTAL FOR TABLE' 
        end category, 
        sum(sal) sal 
   from emp 
  group by cube(deptno,job) 
  order by grouping(job),grouping(deptno) 

DEPTNO JOB       CATEGORY                  SAL
------ --------- --------------------- -------
    10 CLERK     TOTAL BY DEPT AND JOB    1300
    10 MANAGER   TOTAL BY DEPT AND JOB    2450
    10 PRESIDENT TOTAL BY DEPT AND JOB    5000
    20 CLERK     TOTAL BY DEPT AND JOB    1900
    30 CLERK     TOTAL BY DEPT AND JOB     950
    30 SALESMAN  TOTAL BY DEPT AND JOB    5600
    30 MANAGER   TOTAL BY DEPT AND JOB    2850
    20 MANAGER   TOTAL BY DEPT AND JOB    2975
    20 ANALYST   TOTAL BY DEPT AND JOB    6000
       CLERK     TOTAL BY JOB             4150
       ANALYST   TOTAL BY JOB             6000
       MANAGER   TOTAL BY JOB             8275
       PRESIDENT TOTAL BY JOB             5000
       SALESMAN  TOTAL BY JOB             5600
    10           TOTAL BY DEPT            8750
    30           TOTAL BY DEPT            9400
    20           TOTAL BY DEPT           10875
                 GRAND TOTAL FOR TABLE   29025
```

This Oracle solution implicitly converts the results from the GROUPING functions to a character type in preparation for concatenating the two values. DB2 and SQL Server users will need to explicitly CAST the results of the GROUPING functions to CHAR(1), as shown in the solution. In addition, SQL Server users must use the + operator, and not the || operator, to concatenate the results from the two GROUPING calls into one string.

[]()For Oracle and DB2 users, there is an additional extension to GROUP BY called GROUPING SETS; this extension is extremely useful. For example, you can use GROUPING SETS to mimic the output created by []()CUBE as is shown here (DB2 and SQL []()Server users will need to use CAST to ensure the values returned by the GROUPING function are in the correct format in the same way as in the CUBE solution):

```
select deptno, 
        job, 
        case grouping(deptno)||grouping(job) 
             when '00' then 'TOTAL BY DEPT AND JOB' 
             when '10' then 'TOTAL BY JOB' 
             when '01' then 'TOTAL BY DEPT' 
             when '11' then 'GRAND TOTAL FOR TABLE' 
        end category, 
        sum(sal) sal 
   from emp 
  group by grouping sets ((deptno),(job),(deptno,job),()) 

DEPTNO JOB       CATEGORY                  SAL
------ --------- --------------------- -------
    10 CLERK     TOTAL BY DEPT AND JOB    1300
    20 CLERK     TOTAL BY DEPT AND JOB    1900
    30 CLERK     TOTAL BY DEPT AND JOB     950
    20 ANALYST   TOTAL BY DEPT AND JOB    6000
    10 MANAGER   TOTAL BY DEPT AND JOB    2450
    20 MANAGER   TOTAL BY DEPT AND JOB    2975
    30 MANAGER   TOTAL BY DEPT AND JOB    2850
    30 SALESMAN  TOTAL BY DEPT AND JOB    5600
    10 PRESIDENT TOTAL BY DEPT AND JOB    5000
       CLERK     TOTAL BY JOB             4150
       ANALYST   TOTAL BY JOB             6000
       MANAGER   TOTAL BY JOB             8275
       SALESMAN  TOTAL BY JOB             5600
       PRESIDENT TOTAL BY JOB             5000
    10           TOTAL BY DEPT            8750
    20           TOTAL BY DEPT           10875
    30           TOTAL BY DEPT            9400
                 GRAND TOTAL FOR TABLE   29025
```

What’s great about GROUPING SETS is that it allows you to define the groups. The GROUPING SETS clause in the preceding query causes groups to be created by DEPTNO, by JOB, and by the combination of DEPTNO and JOB, and finally the empty parentheses requests a grand total. GROUPING SETS gives you enormous flexibility for creating reports with different levels of aggregation; for example, if you wanted to modify the preceding example to exclude the GRAND TOTAL, simply modify the GROUPING SETS clause by excluding the empty parentheses:

```
/* no grand total */

 select deptno, 
        job, 
        case grouping(deptno)||grouping(job) 
             when '00' then 'TOTAL BY DEPT AND JOB' 
             when '10' then 'TOTAL BY JOB' 
             when '01' then 'TOTAL BY DEPT' 
             when '11' then 'GRAND TOTAL FOR TABLE' 
        end category, 
        sum(sal) sal 
   from emp 
  group by grouping sets ((deptno),(job),(deptno,job)) 


DEPTNO JOB       CATEGORY                    SAL
------ --------- --------------------- ----------
    10 CLERK     TOTAL BY DEPT AND JOB       1300
    20 CLERK     TOTAL BY DEPT AND JOB       1900
    30 CLERK     TOTAL BY DEPT AND JOB        950
    20 ANALYST   TOTAL BY DEPT AND JOB       6000
    10 MANAGER   TOTAL BY DEPT AND JOB       2450
    20 MANAGER   TOTAL BY DEPT AND JOB       2975
    30 MANAGER   TOTAL BY DEPT AND JOB       2850
    30 SALESMAN  TOTAL BY DEPT AND JOB       5600
    10 PRESIDENT TOTAL BY DEPT AND JOB       5000
       CLERK     TOTAL BY JOB                4150
       ANALYST   TOTAL BY JOB                6000
       MANAGER    TOTAL BY JOB                8275
       SALESMAN  TOTAL BY JOB                5600
       PRESIDENT TOTAL BY JOB                5000
    10           TOTAL BY DEPT               8750
    20           TOTAL BY DEPT              10875
    30           TOTAL BY DEPT               9400
```

You can also eliminate a subtotal, such as the one on DEPTNO, simply by omitting (DEPTNO) from the GROUPING SETS clause:

```
/* nosubtotals by DEPTNO */

 select deptno, 
        job, 
        case grouping(deptno)||grouping(job) 
             when '00' then 'TOTAL BY DEPT AND JOB' 
             when '10' then 'TOTAL BY JOB' 
             when '01' then 'TOTAL BY DEPT' 
             when '11' then 'GRAND TOTAL FOR TABLE' 
        end category, 
        sum(sal) sal 
   from emp 
  group by grouping sets ((job),(deptno,job),()) 
  order by 3 


DEPTNO JOB       CATEGORY                     SAL
------ --------- --------------------- ----------
                 GRAND TOTAL FOR TABLE      29025
    10 CLERK     TOTAL BY DEPT AND JOB       1300
    20 CLERK     TOTAL BY DEPT AND JOB       1900
    30 CLERK     TOTAL BY DEPT AND JOB        950
    20 ANALYST   TOTAL BY DEPT AND JOB       6000
    20 MANAGER   TOTAL BY DEPT AND JOB       2975
    30 MANAGER   TOTAL BY DEPT AND JOB       2850
    30 SALESMAN  TOTAL BY DEPT AND JOB       5600
    10 PRESIDENT TOTAL BY DEPT AND JOB       5000
    10 MANAGER   TOTAL BY DEPT AND JOB       2450
       CLERK     TOTAL BY JOB                4150
       SALESMAN  TOTAL BY JOB                5600
       PRESIDENT TOTAL BY JOB                5000
       MANAGER   TOTAL BY JOB                8275
       ANALYST   TOTAL BY JOB                6000
```

As you can see, GROUPING SETS makes it easy indeed to play around with totals and subtotals to look at your data from different angles.[]()

### MySQL

The first step is to use the aggregate function SUM and group by both DEPTNO and JOB:

```
select deptno, job, 
        'TOTAL BY DEPT AND JOB' as category, 
        sum(sal) as sal 
   from emp 
  group by deptno, job 

DEPTNO JOB       CATEGORY                  SAL
------ --------- --------------------- -------
    10 CLERK     TOTAL BY DEPT AND JOB    1300
    10 MANAGER   TOTAL BY DEPT AND JOB    2450
    10 PRESIDENT TOTAL BY DEPT AND JOB    5000
    20 CLERK     TOTAL BY DEPT AND JOB    1900
    20 ANALYST   TOTAL BY DEPT AND JOB    6000
    20 MANAGER   TOTAL BY DEPT AND JOB    2975
    30 CLERK     TOTAL BY DEPT AND JOB     950
    30 MANAGER   TOTAL BY DEPT AND JOB    2850
    30 SALESMAN  TOTAL BY DEPT AND JOB    5600
```

The next step is to use UNION ALL to append TOTAL BY JOB sums:

```
select deptno, job, 
        'TOTAL BY DEPT AND JOB' as category, 
        sum(sal) as sal 
   from emp 
  group by deptno, job 
  union all 
 select null, job, 'TOTAL BY JOB', sum(sal) 
   from emp 
  group by job 

DEPTNO JOB       CATEGORY                  SAL
------ --------- --------------------- -------
    10 CLERK     TOTAL BY DEPT AND JOB    1300
    10 MANAGER   TOTAL BY DEPT AND JOB    2450
    10 PRESIDENT TOTAL BY DEPT AND JOB    5000
    20 CLERK     TOTAL BY DEPT AND JOB    1900
    20 ANALYST   TOTAL BY DEPT AND JOB    6000
    20 MANAGER   TOTAL BY DEPT AND JOB    2975
    30 CLERK     TOTAL BY DEPT AND JOB     950
    30 MANAGER   TOTAL BY DEPT AND JOB    2850
    30 SALESMAN  TOTAL BY DEPT AND JOB    5600
       ANALYST   TOTAL BY JOB             6000
       CLERK     TOTAL BY JOB             4150
       MANAGER   TOTAL BY JOB             8275
       PRESIDENT TOTAL BY JOB             5000
       SALESMAN  TOTAL BY JOB             5600
```

The next step is to UNION ALL the sum of all the salaries by DEPTNO:

```
select deptno, job, 
        'TOTAL BY DEPT AND JOB' as category, 
        sum(sal) as sal 
   from emp 
  group by deptno, job 
  union all 
 select null, job, 'TOTAL BY JOB', sum(sal) 
   from emp 
  group by job 
  union all 
 select deptno, null, 'TOTAL BY DEPT', sum(sal) 
   from emp 
  group by deptno 

DEPTNO JOB       CATEGORY                  SAL
------ --------- --------------------- -------
    10 CLERK     TOTAL BY DEPT AND JOB    1300
    10 MANAGER   TOTAL BY DEPT AND JOB    2450
    10 PRESIDENT TOTAL BY DEPT AND JOB    5000
    20 CLERK     TOTAL BY DEPT AND JOB    1900
    20 ANALYST   TOTAL BY DEPT AND JOB    6000
    20 MANAGER   TOTAL BY DEPT AND JOB    2975
    30 CLERK     TOTAL BY DEPT AND JOB     950
    30 MANAGER   TOTAL BY DEPT AND JOB    2850
    30 SALESMAN  TOTAL BY DEPT AND JOB    5600
       ANALYST   TOTAL BY JOB             6000
       CLERK     TOTAL BY JOB             4150
       MANAGER   TOTAL BY JOB             8275
       PRESIDENT TOTAL BY JOB             5000
       SALESMAN  TOTAL BY JOB             5600
    10           TOTAL BY DEPT            8750
    20           TOTAL BY DEPT            10875
    30           TOTAL BY DEPT             9400
```

The final step is to use UNION ALL to append the sum of all salaries:[]()[]()[]()

```
select deptno, job, 
        'TOTAL BY DEPT AND JOB' as category, 
        sum(sal) as sal 
   from emp 
  group by deptno, job 
  union all 
 select null, job, 'TOTAL BY JOB', sum(sal) 
   from emp 
  group by job 
  union all 
 select deptno, null, 'TOTAL BY DEPT', sum(sal) 
   from emp 
  group by deptno 
  union all 
 select null,null, 'GRAND TOTAL FOR TABLE', sum(sal) 
   from emp 

DEPTNO JOB       CATEGORY                  SAL
------ --------- --------------------- -------
    10 CLERK     TOTAL BY DEPT AND JOB    1300
    10 MANAGER   TOTAL BY DEPT AND JOB    2450
    10 PRESIDENT TOTAL BY DEPT AND JOB    5000
    20 CLERK     TOTAL BY DEPT AND JOB    1900
    20 ANALYST   TOTAL BY DEPT AND JOB    6000
    20 MANAGER   TOTAL BY DEPT AND JOB    2975
    30 CLERK     TOTAL BY DEPT AND JOB     950
    30 MANAGER   TOTAL BY DEPT AND JOB    2850
    30 SALESMAN  TOTAL BY DEPT AND JOB    5600
       ANALYST   TOTAL BY JOB             6000
       CLERK     TOTAL BY JOB             4150
       MANAGER   TOTAL BY JOB             8275
       PRESIDENT TOTAL BY JOB             5000
       SALESMAN  TOTAL BY JOB             5600
    10           TOTAL BY DEPT            8750
    20           TOTAL BY DEPT           10875
    30           TOTAL BY DEPT            9400
                 GRAND TOTAL FOR TABLE   29025
```

# 12.14 Identifying Rows That Are Not Subtotals

## Problem

[]()You’ve used the CUBE extension of the GROUP BY clause to create a report, and you need a way to differentiate between rows that would be generated by a normal []()GROUP BY clause and those rows that have been generated as a result of using CUBE or ROLLUP.

The following is the result set from a query using the CUBE extension to GROUP BY to create a breakdown of the salaries in table EMP:

```
DEPTNO JOB           SAL
------ --------- -------
                   29025
       CLERK        4150
       ANALYST      6000
       MANAGER      8275
       SALESMAN     5600
       PRESIDENT    5000
    10              8750
    10 CLERK        1300
    10 MANAGER      2450
    10 PRESIDENT    5000
    20             10875
    20 CLERK        1900
    20 ANALYST      6000
    20 MANAGER      2975
    30              9400
    30 CLERK         950
    30 MANAGER      2850
    30 SALESMAN     5600
```

This report includes the sum of all salaries by DEPTNO and JOB (for each JOB per DEPTNO), the sum of all salaries by DEPTNO, the sum of all salaries by JOB, and finally a grand total (the sum of all salaries in table EMP). You want to clearly identify the different levels of aggregation. You want to be able to identify which category an aggregated value belongs to (i.e., does a given value in the SAL column represent a total by DEPTNO? By JOB? The grand total?). You would like to return the following result set:

```
DEPTNO JOB           SAL DEPTNO_SUBTOTALS JOB_SUBTOTALS
------ --------- ------- ---------------- -------------
                   29025                1             1
       CLERK        4150                1             0
       ANALYST      6000                1             0
       MANAGER      8275                1             0
       SALESMAN     5600                1             0
       PRESIDENT    5000                1             0
    10              8750                0             1
    10 CLERK        1300                0             0
    10 MANAGER      2450                0             0
    10 PRESIDENT    5000                0             0
    20             10875                0             1
    20 CLERK        1900                0             0
    20 ANALYST      6000                0             0
    20 MANAGER      2975                0             0
    30              9400                0             1
    30 CLERK         950                0             0
    30 MANAGER      2850                0             0
    30 SALESMAN     5600                0             0
```

## Solution

Use the GROUPING function to identify which values exist due to CUBE’s or ROLLUP’s creation of subtotals, or *superaggregate* values. The following is an example for PostgreSQL, DB2, and Oracle:

```
 1 select deptno, jo) sal,
 2        grouping(deptno) deptno_subtotals,
 3        grouping(job) job_subtotals
 4   from emp
 5  group by cube(deptno,job)
```

The only difference between the SQL Server solution and that for DB2 and Oracle lies in how the CUBE/ROLLUP clauses are written:

```
 1 select deptno, job, sum(sal) sal,
 2        grouping(deptno) deptno_subtotals,
 3        grouping(job) job_subtotals
 4   from emp
 5  group by deptno,job with cube
```

This recipe is meant to highlight the use of CUBE and GROUPING when working with subtotals. As of the time of this writing, MySQL doesn’t support either CUBE or GROUPING.

## Discussion

If DEPTNO\_SUBTOTALS is 0 and JOB\_SUBTOTALS is 1 (in which case JOB is NULL), the value of SAL represents a subtotal of salaries by DEPTNO created by CUBE. If JOB\_SUBTOTALS is 0 and DEPTNO\_SUBTOTALS is 1 (in which case DEPTNO is NULL), the value of SAL represents a subtotal of salaries by JOB created by CUBE. Rows with 0 for both DEPTNO\_SUBTOTALS and JOB\_SUBTOTALS represent rows created by regular aggregation (the sum of SAL for each DEPTNO/JOB combination).[]()

# 12.15 Using Case Expressions to Flag Rows

## Problem

[]()You want to map the values in a column, perhaps the EMP table’s JOB column, into a series of “Boolean” flags. For example, you want to return the following result set:

```
ENAME  IS_CLERK IS_SALES IS_MGR IS_ANALYST IS_PREZ
------ -------- -------- ------ ---------- -------
KING          0        0      0          0       1
SCOTT         0        0      0          1       0
FORD          0        0      0          1       0
JONES         0        0      1          0       0
BLAKE         0        0      1          0       0
CLARK         0        0      1          0       0
ALLEN         0        1      0          0       0
WARD          0        1      0          0       0
MARTIN        0        1      0          0       0
TURNER        0        1      0          0       0
SMITH         1        0      0          0       0
MILLER        1        0      0          0       0
ADAMS         1        0      0          0       0
JAMES         1        0      0          0       0
```

Such a result set can be useful for debugging and to provide yourself a view of the data different from what you’d see in a more typical result set.

## Solution

Use a CASE expression to evaluate each employee’s JOB, and return a 1 or 0 to signify their JOB. You’ll need to write one CASE expression, and thus create one column for each possible job:

```
 1 select ename,
 2        case when job = 'CLERK'
 3             then 1 else 0
 4        end as is_clerk,
 5        case when job = 'SALESMAN'
 6             then 1 else 0
 7        end as is_sales,
 8        case when job = 'MANAGER'
 9             then 1 else 0
10        end as is_mgr,
11        case when job = 'ANALYST'
12             then 1 else 0
13        end as is_analyst,
14        case when job = 'PRESIDENT'
15             then 1 else 0
16        end as is_prez
17   from emp
18  order by 2,3,4,5,6
```

## Discussion

The solution code is pretty much self-explanatory. If you are having trouble understanding it, simply add JOB to the SELECT clause:[]()

```
select ename, 
        job, 
        case when job = 'CLERK' 
             then 1 else 0 
        end as is_clerk, 
        case when job = 'SALESMAN' 
             then 1 else 0 
        end as is_sales, 
        case when job = 'MANAGER' 
             then 1 else 0 
        end as is_mgr, 
        case when job = 'ANALYST' 
            then 1 else 0 
        end as is_analyst, 
        case when job = 'PRESIDENT' 
            then 1 else 0 
        end as is_prez 
   from emp 
  order by 2 

ENAME  JOB       IS_CLERK IS_SALES IS_MGR IS_ANALYST IS_PREZ
------ --------- -------- -------- ------ ---------- -------
SCOTT  ANALYST          0        0      0          1       0
FORD   ANALYST          0        0      0          1       0
SMITH  CLERK            1        0      0          0       0
ADAMS  CLERK            1        0      0          0       0
MILLER CLERK            1        0      0          0       0
JAMES  CLERK            1        0      0          0       0
JONES  MANAGER          0        0      1          0       0
CLARK  MANAGER          0        0      1          0       0
BLAKE  MANAGER          0        0      1          0       0
KING   PRESIDENT        0        0      0          0       1
ALLEN  SALESMAN         0        1      0          0       0
MARTIN SALESMAN         0        1      0          0       0
TURNER SALESMAN         0        1      0          0       0
WARD   SALESMAN         0        1      0          0       0
```

# 12.16 Creating a Sparse Matrix

## Problem

[]()[]()You want to create a sparse matrix, such as the following one transposing the DEPTNO and JOB columns of table EMP:

```
D10        D20        D30        CLERKS MGRS  PREZ ANALS SALES
---------- ---------- ---------- ------ ----- ---- ----- ------
           SMITH                 SMITH
                      ALLEN                              ALLEN
                      WARD                               WARD
           JONES                        JONES
                      MARTIN                             MARTIN
                      BLAKE             BLAKE
CLARK                                   CLARK
           SCOTT                                   SCOTT
KING                                          KING
                      TURNER                             TURNER
           ADAMS                 ADAMS
                      JAMES      JAMES
           FORD                                    FORD
MILLER                           MILLER
```

## Solution

Use CASE expressions to create a sparse row-to-column transformation:

```
 1 select case deptno when 10 then ename end as d10,
 2        case deptno when 20 then ename end as d20,
 3        case deptno when 30 then ename end as d30,
 4        case job when 'CLERK' then ename end as clerks,
 5        case job when 'MANAGER' then ename end as mgrs,
 6        case job when 'PRESIDENT' then ename end as prez,
 7        case job when 'ANALYST' then ename end as anals,
 8        case job when 'SALESMAN' then ename end as sales
 9   from emp
```

## Discussion

To transform the DEPTNO and JOB rows to columns, simply use a CASE expression to evaluate the possible values returned by those rows. That’s all there is to it. As an aside, if you want to “densify” the report and get rid of some of those NULL rows, you would need to find something to group by. For example, use the window function ROW\_NUMBER OVER to assign a ranking for each employee per DEPTNO, and then use the aggregate function MAX to rub out some of the NULLs:

```
select max(case deptno when 10 then ename end) d10, 
        max(case deptno when 20 then ename end) d20, 
        max(case deptno when 30 then ename end) d30, 
        max(case job when 'CLERK' then ename end) clerks, 
        max(case job when 'MANAGER' then ename end) mgrs, 
        max(case job when 'PRESIDENT' then ename end) prez,  
        max(case job when 'ANALYST' then ename end) anals,  
        max(case job when 'SALESMAN' then ename end) sales 
   from (  
 select deptno, job, ename, 
        row_number()over(partition by deptno order by empno) rn  
   from emp  
        ) x  
  group by rn 

D10        D20        D30        CLERKS MGRS  PREZ ANALS SALES
---------- ---------- ---------- ------ ----- ---- ----- ------
CLARK      SMITH      ALLEN      SMITH  CLARK            ALLEN
KING       JONES      WARD              JONES KING       WARD
MILLER     SCOTT      MARTIN     MILLER       SCOTT      MARTIN
           ADAMS      BLAKE      ADAMS  BLAKE
           FORD       TURNER                  FORD       TURNER
                      JAMES      JAMES
```

# 12.17 Grouping Rows by Units of Time

## Problem

[]()[]()[]()You want to summarize data by some interval of time. For example, you have a transaction log and want to summarize transactions by five-second intervals. The rows in table TRX\_LOG are shown here:

```
select trx_id, 
        trx_date, 
        trx_cnt 
   from trx_log 
TRX_ID TRX_DATE                TRX_CNT
------ -------------------- ----------
     1 28-JUL-2020 19:03:07         44
     2 28-JUL-2020 19:03:08         18
     3 28-JUL-2020 19:03:09         23
     4 28-JUL-2020 19:03:10         29
     5 28-JUL-2020 19:03:11         27
     6 28-JUL-2020 19:03:12         45
     7 28-JUL-2020 19:03:13         45
     8 28-JUL-2020 19:03:14         32
     9 28-JUL-2020 19:03:15         41
    10 28-JUL-2020 19:03:16         15
    11 28-JUL-2020 19:03:17         24
    12 28-JUL-2020 19:03:18         47
    13 28-JUL-2020 19:03:19         37
    14 28-JUL-2020 19:03:20         48
    15 28-JUL-2020 19:03:21         46
    16 28-JUL-2020 19:03:22         44
    17 28-JUL-2020 19:03:23         36
    18 28-JUL-2020 19:03:24         41
    19 28-JUL-2020 19:03:25         33
    20 28-JUL-2020 19:03:26         19
```

You want to return the following result set:

```
GRP TRX_START            TRX_END                   TOTAL
--- -------------------- -------------------- ----------
  1 28-JUL-2020 19:03:07 28-JUL-2020 19:03:11        141
  2 28-JUL-2020 19:03:12 28-JUL-2020 19:03:16        178
  3 28-JUL-2020 19:03:17 28-JUL-2020 19:03:21        202
  4 28-JUL-2020 19:03:22 28-JUL-2020 19:03:26        173
```

## Solution

Group the entries into five row buckets. There are several ways to accomplish that logical grouping; this recipe does so by dividing the TRX\_ID values by five, using a technique shown earlier in [Recipe 12.7](#sqlckbk-CHP-12-SECT-7).

Once you’ve created the “groups,” use the aggregate functions MIN, MAX, and SUM to find the start time, end time, and total number of transactions for each “group” (SQL []()Server users should use CEILING instead of CEIL):

```
 1 select ceil(trx_id/5.0) as grp,
 2        min(trx_date)    as trx_start,
 3        max(trx_date)    as trx_end,
 4        sum(trx_cnt)     as total
 5   from trx_log
 6 group by ceil(trx_id/5.0)
```

## Discussion

The first step, and the key to the whole solution, is to logically group the rows together. By dividing by five and taking the smallest whole number greater than the quotient, you can create logical groups. For example:

```
select trx_id, 
        trx_date, 
        trx_cnt, 
        trx_id/5.0 as val, 
        ceil(trx_id/5.0) as grp 
   from trx_log 
TRX_ID TRX_DATE             TRX_CNT     VAL GRP
------ -------------------- -------  ------ ---
     1 28-JUL-2020 19:03:07      44     .20   1
     2 28-JUL-2020 19:03:08      18     .40   1
     3 28-JUL-2020 19:03:09      23     .60   1
     4 28-JUL-2020 19:03:10      29     .80   1
     5 28-JUL-2020 19:03:11      27    1.00   1
     6 28-JUL-2020 19:03:12      45    1.20   2
     7 28-JUL-2020 19:03:13      45    1.40   2
     8 28-JUL-2020 19:03:14      32    1.60   2
     9 28-JUL-2020 19:03:15      41    1.80   2
    10 28-JUL-2020 19:03:16      15    2.00   2
    11 28-JUL-2020 19:03:17      24    2.20   3
    12 28-JUL-2020 19:03:18      47    2.40   3
    13 28-JUL-2020 19:03:19      37    2.60   3
    14 28-JUL-2020 19:03:20      48    2.80   3
    15 28-JUL-2020 19:03:21      46    3.00   3
    16 28-JUL-2020 19:03:22      44    3.20   4
    17 28-JUL-2020 19:03:23      36    3.40   4
    18 28-JUL-2020 19:03:24      41    3.60   4
    19 28-JUL-2020 19:03:25      33    3.80   4
    20 28-JUL-2020 19:03:26      19    4.00   4
```

The last step is to apply the appropriate aggregate functions to find the total number of transactions per five seconds, along with the start and end times for each transaction:

```
select ceil(trx_id/5.0) as grp, 
        min(trx_date) as trx_start, 
        max(trx_date) as trx_end, 
        sum(trx_cnt) as total 
   from trx_log 
  group by ceil(trx_id/5.0) 
GRP TRX_START            TRX_END                   TOTAL
--- -------------------- -------------------- ----------
  1 28-JUL-2020 19:03:07 28-JUL-2005 19:03:11        141
  2 28-JUL-2020 19:03:12 28-JUL-2005 19:03:16        178
  3 28-JUL-2020 19:03:17 28-JUL-2005 19:03:21        202
  4 28-JUL-2020 19:03:22 28-JUL-2005 19:03:26        173
```

If your data is slightly different (perhaps you don’t have an ID for each row), you can always “group” by dividing the seconds of each TRX\_DATE row by five to create a similar grouping. Then you can include the hour for each TRX\_DATE and group by the actual hour and logical “grouping,” GRP. The following is an example of this technique (using Oracle’s TO\_CHAR and TO\_NUMBER functions, you would use the appropriate date and character formatting functions for your platform):

```
select trx_date,trx_cnt, 
        to_number(to_char(trx_date,'hh24')) hr, 
        ceil(to_number(to_char(trx_date-1/24/60/60,'miss'))/5.0) grp 
   from trx_log 

TRX_DATE             20   TRX_CNT         HR         GRP
-------------------- ---------- ----------  ----------
28-JUL-2020 19:03:07         44         19          62
28-JUL-2020 19:03:08         18         19          62
28-JUL-2020 19:03:09         23         19          62
28-JUL-2020 19:03:10         29         19          62
28-JUL-2020 19:03:11         27         19          62
28-JUL-2020 19:03:12         45         19          63
28-JUL-2020 19:03:13         45         19          63
28-JUL-2020 19:03:14         32         19          63
28-JUL-2020 19:03:15         41         19          63
28-JUL-2020 19:03:16         15         19          63
28-JUL-2020 19:03:17         24         19          64
28-JUL-2020 19:03:18         47         19          64
28-JUL-2020 19:03:19         37         19          64
28-JUL-2020 19:03:20         48         19          64
28-JUL-2020 19:03:21         46         19          64
28-JUL-2020 19:03:22         44         19          65
28-JUL-2020 19:03:23         36         19          65
28-JUL-2020 19:03:24         41         19          65
28-JUL-2020 19:03:25         33         19          65
28-JUL-2020 19:03:26         19         19          65
```

Regardless of the actual values for GRP, the key here is that you are grouping for every five seconds. From there you can apply the aggregate functions in the same way as in the original solution:

```
select hr,grp,sum(trx_cnt) total 
   from ( 
 select trx_date,trx_cnt, 
        to_number(to_char(trx_date,'hh24')) hr, 
        ceil(to_number(to_char(trx_date-1/24/60/60,'miss'))/5.0) grp 
   from trx_log 
        ) x 
  group by hr,grp 
HR        GRP       TOTAL
-- ----------  ----------
19         62         141
19         63         178
19         64         202
19         65         173
```

Including the hour in the grouping is useful if your transaction log spans hours. In DB2 and Oracle, you can also use the window function SUM OVER to produce the same result. The following query returns all rows from TRX\_LOG along with a running total for TRX\_CNT by logical “group,” and the TOTAL for TRX\_CNT for each row in the “group”:[]()[]()[]()

```
select trx_id, trx_date, trx_cnt, 
        sum(trx_cnt)over(partition by ceil(trx_id/5.0) 
                         order by trx_date 
                         range between unbounded preceding 
                           and current row) runing_total, 
        sum(trx_cnt)over(partition by ceil(trx_id/5.0)) total, 
        case when mod(trx_id,5.0) = 0 then 'X' end grp_end 
   from trx_log 

TRX_ID TRX_DATE                TRX_CNT RUNING_TOTAL      TOTAL GRP_END
------ -------------------- ---------- ------------ ---------- -------
     1 28-JUL-2020 19:03:07         44           44       141
     2 28-JUL-2020 19:03:08         18           62       141
     3 28-JUL-2020 19:03:09         23           85       141
     4 28-JUL-2020 19:03:10         29          114       141
     5 28-JUL-2020 19:03:11         27          141       141  X
     6 28-JUL-2020 19:03:12         45           45       178
     7 28-JUL-2020 19:03:13         45           90       178
     8 28-JUL-2020 19:03:14         32          122       178
     9 28-JUL-2020 19:03:15         41          163       178
    10 28-JUL-2020 19:03:16         15          178       178  X
    11 28-JUL-2020 19:03:17         24           24       202
    12 28-JUL-2020 19:03:18         47           71       202
    13 28-JUL-2020 19:03:19         37          108       202
    14 28-JUL-2020 19:03:20         48          156       202
    15 28-JUL-2020 19:03:21         46          202       202  X
    16 28-JUL-2020 19:03:22         44           44       173
    17 28-JUL-2020 19:03:23         36           80       173
    18 28-JUL-2020 19:03:24         41          121       173
    19 28-JUL-2020 19:03:25         33          154       173
    20 28-JUL-2020 19:03:26         19          173       173  X
```

# 12.18 Performing Aggregations over Different Groups/Partitions Simultaneously

## Problem

[]()You want to aggregate over different dimensions at the same time. For example, you want to return a result set that lists each employee’s name, their department, the number of employees in their department (themselves included), the number of employees that have the same job (themselves included in this count as well), and the total number of employees in the EMP table. The result set should look like the following:

```
ENAME  DEPTNO DEPTNO_CNT JOB       JOB_CNT   TOTAL
------ ------ ---------- --------- -------- ------
MILLER     10          3 CLERK            4     14
CLARK      10          3 MANAGER          3     14
KING       10          3 PRESIDENT        1     14
SCOTT      20          5 ANALYST          2     14
FORD       20          5 ANALYST          2     14
SMITH      20          5 CLERK            4     14
JONES      20          5 MANAGER          3     14
ADAMS      20          5 CLERK            4     14
JAMES      30          6 CLERK            4     14
MARTIN     30          6 SALESMAN         4     14
TURNER     30          6 SALESMAN         4     14
WARD       30          6 SALESMAN         4     14
ALLEN      30          6 SALESMAN         4     14
BLAKE      30          6 MANAGER          3     14
```

## Solution

[]()Use the COUNT OVER window function while specifying different partitions, or groups of data, on which to perform aggregation:

```
select ename,
       deptno,
       count(*)over(partition by deptno) deptno_cnt,
       job,
       count(*)over(partition by job) job_cnt,
       count(*)over() total
  from emp
```

## Discussion

This example really shows off the power and convenience of window functions. By simply specifying different partitions or groups of data to aggregate, you can create immensely detailed reports without having to self-join over and over, and without having to write cumbersome and perhaps poorly performing subqueries in your SELECT list. All the work is done by the window function COUNT OVER. To understand the output, focus on the OVER clause for a moment for each COUNT operation:

```
count(*)over(partition by deptno)

count(*)over(partition by job)

count(*)over()
```

Remember the main parts of the OVER clause: the PARTITION BY subclause, dividing the query into partitions; and the ORDER BY subclause, defining the logical order. Look at the first COUNT, which partitions by DEPTNO. The rows in table EMP will be grouped by DEPTNO, and the COUNT operation will be performed on all the rows in each group. Since there is no frame or window clause specified (no ORDER BY), all the rows in the group are counted. The PARTITION BY clause finds all the unique DEPTNO values, and then the COUNT function counts the number of rows having each value. In the specific example of COUNT(\*)OVER(PARTITION BY DEPTNO), the PARTITION BY clause identifies the partitions or groups to be values 10, 20, and 30.

The same processing is applied to the second COUNT, which partitions by JOB. The last count does not partition by anything and simply has an empty parentheses. An empty parentheses implies “the whole table.” So, whereas the two prior COUNTs aggregate values based on the defined groups or partitions, the final COUNT counts all rows in table EMP.

###### Warning

[]()Keep in mind that window functions are applied after the WHERE clause. If you were to filter the result set in some way, for example, excluding all employees in DEPTNO 10, the value for TOTAL would not be 14—it would be 11. To filter results after window functions have been evaluated, you must make your windowing query into an inline view and then filter on the results from that view.[]()

# 12.19 Performing Aggregations over a Moving Range of Values

## Problem

[]()You want to compute a moving aggregation, such as a moving sum on the salaries in table EMP. You want to compute a sum for every 90 days, starting with the HIREDATE of the first employee. You want to see how spending has fluctuated for every 90-day period between the first and last employee hired. You want to return the following result set:

```
HIREDATE        SAL SPENDING_PATTERN
----------- ------- ----------------
17-DEC-200     800              800
20-FEB-2011    1600             2400
22-FEB-2011    1250             3650
02-APR-2011    2975             5825
01-MAY-2011    2850             8675
09-JUN-2011    2450             8275
08-SEP-2011    1500             1500
28-SEP-2011    1250             2750
17-NOV-2011    5000             7750
03-DEC-2011     950            11700
03-DEC-2011    3000            11700
23-JAN-2012    1300            10250
09-DEC-2012    3000             3000
12-JAN-2013    1100             4100
```

## Solution

[]()Being able to specify a moving window in the framing or windowing clause of window functions makes this problem easy to solve, if your RDBMS supports such functions. The key is to order by HIREDATE in your window function and then specify a window of 90 days starting from the earliest employee hired. The sum will be computed using the salaries of employees hired up to 90 days prior to the current employee’s HIREDATE (the current employee is included in the sum). If you do not have window functions available, you can use scalar subqueries, but the solution will be more complex.

### DB2 and Oracle

For DB2 and Oracle, use the window function SUM OVER and order by HIREDATE. Specify a range of 90 days in the window or “framing” clause to allow the sum to be computed for each employee’s salary and to include the salaries of all employees hired up to 90 days earlier. []()[]()Because DB2 does not allow you to specify HIREDATE in the ORDER BY clause of a window function (line 3 in the following code), you can order by DAYS(HIREDATE) instead:

```
   1 select hiredate,
   2        sal,
   3        sum(sal)over(order by days(hiredate)
   4                        range between 90 preceding
   5                          and current row) spending_pattern
   6   from emp e
```

The Oracle solution is more straightforward than DB2’s, because Oracle allows window functions to order by datetime types:

```
   1 select hiredate,
   2        sal,
   3        sum(sal)over(order by hiredate
   4                        range between 90 preceding
   5                          and current row) spending_pattern
   6   from emp e
```

### MySQL

Use the window function with slightly altered syntax:

```
1  select hiredate,
2          sal,
3          sum(sal)over(order by hiredate
4              range interval 90 day preceding ) spending_pattern
5  from emp e
```

### PostgreSQL and SQL Server

Use a scalar subquery to sum the salaries of all employees hired up to 90 days prior to the day each employee was hired:

```
 1 select e.hiredate,
 2        e.sal,
 3        (select sum(sal) from emp d
 4          whered.hiredate between e.hiredate-90
 5                              and e.hiredate) as spending_pattern
 6   from emp e
 7  order by 1
```

## Discussion

### DB2, MySQL, and Oracle

DB2, MySQL, and Oracle share the same logical solution. The only minor differences between the solutions are in how you specify HIREDATE in the ORDER BY clause of the window function and the syntax of specifying the time interval in MySQL. At the time of this book’s writing, DB2 doesn’t allow a DATE value in such an ORDER BY clause if you are using a numeric value to set the window’s range. (For example, RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW allows you to order by a date, but RANGE BETWEEN 90 PRECEDING AND CURRENT ROW does not.)

To understand what the solution query is doing, you simply need to understand what the window clause is doing. The window you are defining orders the salaries for all employees by HIREDATE. Then the function computes a sum. The sum is not computed for all salaries. Instead, the processing is as follows:

1. The salary of the first employee hired is evaluated. Since no employees were hired before the first employee, the sum at this point is simply the first employee’s salary.
2. The salary of the next employee (by HIREDATE) is evaluated. This employee’s salary is included in the moving sum along with any other employees who were hired up to 90 days prior.

The HIREDATE of the first employee is December 17, 2010, and the HIREDATE of the next hired employee is February 20, 2011. The second employee was hired less than 90 days after the first employee, and thus the moving sum for the second employee is 2400 (1600 + 800). If you are having trouble understanding where the values in SPENDING\_PATTERN come from, examine the following query and result set:

```
select distinct 
        dense_rank()over(order by e.hiredate) window, 
        e.hiredate current_hiredate, 
        d.hiredate hiredate_within_90_days, 
        d.sal sals_used_for_sum 
   from emp e, 
        emp d 
 where d.hiredate between e.hiredate-90 and e.hiredate 

WINDOW CURRENT_HIREDATE HIREDATE_WITHIN_90_DAYS SALS_USED_FOR_SUM
------ ---------------- ----------------------- -----------------
     1 17-DEC-2010      17-DEC-2010                           800
     2 20-FEB-2011      17-DEC-2010                           800
     2 20-FEB-2011      20-FEB-2011                          1600
     3 22-FEB-2011      17-DEC-2010                           800
     3 22-FEB-2011      20-FEB-2011                          1600
     3 22-FEB-2011      22-FEB-2011                          1250
     4 02-APR-2011      20-FEB-2011                          1600
     4 02-APR-2011      22-FEB-2011                          1250
     4 02-APR-2011      02-APR-2011                          2975
     5 01-MAY-2011      20-FEB-2011                          1600
     5 01-MAY-2011      22-FEB-2011                          1250
     5 01-MAY-2011      02-APR-2011                          2975
     5 01-MAY-2011      01-MAY-2011                          2850
     6 09-JUN-2011      02-APR-2011                          2975
     6 09-JUN-2011      01-MAY-2011                          2850
     6 09-JUN-2011      09-JUN-2011                          2450
     7 08-SEP-2011      08-SEP-2011                          1500
     8 28-SEP-2011      08-SEP-2011                          1500
     8 28-SEP-2011      28-SEP-2011                          1250
     9 17-NOV-2011      08-SEP-2011                          1500
     9 17-NOV-2011      28-SEP-2011                          1250
     9 17-NOV-2011      17-NOV-2011                          5000
    10 03-DEC-2011      08-SEP-2011                          1500
    10 03-DEC-2011      28-SEP-2011                          1250
    10 03-DEC-2011      17-NOV-2011                          5000
    10 03-DEC-2011      03-DEC-2011                           950
    10 03-DEC-2011      03-DEC-2011                          3000
    11 23-JAN-2012      17-NOV-2011                          5000
    11 23-JAN-2012      03-DEC-2011                           950
    11 23-JAN-2012      03-DEC-2011                          3000
    11 23-JAN-2012      23-JAN-2012                          1300
    12 09-DEC-2012      09-DEC-2012                          3000
    13 12-JAN-2013      09-DEC-2012                          3000
    13 12-JAN-2013      12-JAN-2013                          1100
```

If you look at the WINDOW column, only those rows with the same WINDOW value will be considered for each sum. Take, for example, WINDOW 3. The salaries used for the sum for that window are 800, 1600, and 1250, which total 3650. If you look at the final result set in the “Problem” section, you’ll see the SPENDING\_PATTERN for February 22, 2011 (WINDOW 3) is 3650. As proof, to verify that the previous self-join includes the correct salaries for the windows defined, simply sum the values in SALS\_USED\_FOR\_SUM and group by CURRENT\_DATE. The result should be the same as the result set shown in the “Problem” section (with the duplicate row for December 3, 2011, filtered out):

```
select current_hiredate, 
        sum(sals_used_for_sum) spending_pattern 
   from ( 
 select distinct 
        dense_rank()over(order by e.hiredate) window, 
        e.hiredate current_hiredate, 
        d.hiredate hiredate_within_90_days, 
        d.sal sals_used_for_sum 
   from emp e, 
        emp d 
   where d.hiredate between e.hiredate-90 and e.hiredate 
         ) x 
   group by current_hiredate 

CURRENT_HIREDATE SPENDING_PATTERN
---------------- ----------------
17-DEC-2010                   800
20-FEB-2011                  2400
22-FEB-2011                  3650
02-APR-2011                  5825
01-MAY-2011                  8675
09-JUN-2011                  8275
08-SEP-2011                  1500
28-SEP-2011                  2750
17-NOV-2011                  7750
03-DEC-2011                 11700
23-JAN-2012                 10250
09-DEC-2012                  3000
12-JAN-2013                  4100
```

### PostgreSQL and SQL Server

The key to this solution is to use a scalar subquery (a self-join will work as well) while using the aggregate function SUM to compute a sum for every 90 days based on HIREDATE. If you are having trouble seeing how this works, simply convert the solution to a self-join and examine which rows are included in the computations. Consider the following result set, which returns the same result set as that in the solution:

```
select e.hiredate, 
        e.sal, 
        sum(d.sal) as spending_pattern 
   from emp e, emp d 
  where d.hiredate 
        between e.hiredate-90 and e.hiredate 
  group by e.hiredate,e.sal 
  order by 1 \

HIREDATE      SAL   SPENDING_PATTERN
----------- -----   ----------------
17-DEC-2010   800                800
20-FEB-2011  1600               2400
22-FEB-2011  1250               3650
02-APR-2011  2975               5825
01-MAY-2011  2850               8675
09-JUN-2011  2450               8275
08-SEP-2011  1500               1500
28-SEP-2011  1250               2750
17-NOV-2011  5000               7750
03-DEC-2011   950              11700
03-DEC-2011  3000              11700
23-JAN-2012  1300              10250
09-DEC-2012  3000               3000
12-JAN-2013  1100               4100
```

If it is still unclear, simply remove the aggregation and start with the Cartesian product. The first step is to generate a Cartesian product using table EMP so that each HIREDATE can be compared with all the other HIREDATEs. (Only a snippet of the result set is shown here because there are 196 rows (14 × 14) returned by a Cartesian of EMP):

```
select e.hiredate, 
        e.sal, 
        d.sal, 
        d.hiredate 
   from emp e, emp d 

HIREDATE      SAL      SAL HIREDATE
----------- -----    ----- -----------
17-DEC-2010   800      800 17-DEC-2010
17-DEC-2010   800     1600 20-FEB-2011
17-DEC-2010   800     1250 22-FEB-2011
17-DEC-2010   800     2975 02-APR-2011
17-DEC-2010   800     1250 28-SEP-2011
17-DEC-2010   800     2850 01-MAY-2011
17-DEC-2010   800     2450 09-JUN-2011
17-DEC-2010   800     3000 09-DEC-2012
17-DEC-2010   800     5000 17-NOV-2011
17-DEC-2010   800     1500 08-SEP-2011
17-DEC-2010   800     1100 12-JAN-2013
17-DEC-2010   800      950 03-DEC-2011
17-DEC-2010   800     3000 03-DEC-2011
17-DEC-2010   800     1300 23-JAN-2012
20-FEB-2011  1600      800 17-DEC-2010
20-FEB-2011  1600     1600 20-FEB-2011
20-FEB-2011  1600     1250 22-FEB-2011
20-FEB-2011  1600     2975 02-APR-2011
20-FEB-2011  1600     1250 28-SEP-2011
20-FEB-2011  1600     2850 01-MAY-2011
20-FEB-2011  1600     2450 09-JUN-2011
20-FEB-2011  1600     3000 09-DEC-2012
20-FEB-2011  1600     5000 17-NOV-2011
20-FEB-2011  1600     1500 08-SEP-2011
20-FEB-2011  1600     1100 12-JAN-2013
20-FEB-2011  1600      950 03-DEC-2011
20-FEB-2011  1600     3000 03-DEC-2011
20-FEB-2011  1600     1300 23-JAN-2012
```

If you examine the previous result set, you’ll notice that there is no HIREDATE 90 days earlier or equal to December 17, except for December 17. So, the sum for that row should be only 800. If you examine the next HIREDATE, February 20, you’ll notice that there is one HIREDATE that falls within the 90-day window (within 90 days prior), and that is December 17. If you sum the SAL from December 17 with the SAL from February 20 (because we are looking for HIREDATEs equal to each HIREDATE or within 90 days earlier), you get 2400, which happens to be the final result for that HIREDATE.

Now that you know how it works, use a filter in the WHERE clause to return for each HIREDATE and HIREDATE that is equal to it or is no more than 90 days earlier:

```
select e.hiredate, 
        e.sal, 
        d.sal sal_to_sum, 
        d.hiredate within_90_days 
   from emp e, emp d 
  where d.hiredate 
        between e.hiredate-90 and e.hiredate 
  order by 1 
HIREDATE      SAL SAL_TO_SUM WITHIN_90_DAYS
----------- ----- ---------- --------------
17-DEC-2010   800        800    17-DEC-2010
20-FEB-2011  1600        800    17-DEC-2010
20-FEB-2011  1600       1600    20-FEB-2011
22-FEB-2011  1250        800    17-DEC-2010
22-FEB-2011  1250       1600    20-FEB-2011
22-FEB-2011  1250       1250    22-FEB-2011
02-APR-2011  2975       1600    20-FEB-2011
02-APR-2011  2975       1250    22-FEB-2011
02-APR-2011  2975       2975    02-APR-2011
01-MAY-2011  2850       1600    20-FEB-2011
01-MAY-2011  2850       1250    22-FEB-2011
01-MAY-2011  2850       2975    02-APR-2011
01-MAY-2011  2850       2850    01-MAY-2011
09-JUN-2011  2450       2975    02-APR-2011
09-JUN-2011  2450       2850    01-MAY-2011
09-JUN-2011  2450       2450    09-JUN-2011
08-SEP-2011  1500       1500    08-SEP-2011
28-SEP-2011  1250       1500    08-SEP-2011
28-SEP-2011  1250       1250    28-SEP-2011
17-NOV-2011  5000       1500    08-SEP-2011
17-NOV-2011  5000       1250    28-SEP-2011
17-NOV-2011  5000       5000    17-NOV-2011
03-DEC-2011   950       1500    08-SEP-2011
03-DEC-2011   950       1250    28-SEP-2011
03-DEC-2011   950       5000    17-NOV-2011
03-DEC-2011   950        950    03-DEC-2011
03-DEC-2011   950       3000    03-DEC-2011
03-DEC-2011  3000       1500    08-SEP-2011
03-DEC-2011  3000       1250    28-SEP-2011
03-DEC-2011  3000       5000    17-NOV-2011
03-DEC-2011  3000        950    03-DEC-2011
03-DEC-2011  3000       3000    03-DEC-2011
23-JAN-2012  1300       5000    17-NOV-2011
23-JAN-2012  1300        950    03-DEC-2011
23-JAN-2012  1300       3000    03-DEC-2011
23-JAN-2012  1300       1300    23-JAN-2012
09-DEC-2012  3000       3000    09-DEC-2012
12-JAN-2013  1100       3000    09-DEC-2012
12-JAN-2013  1100       1100    12-JAN-2013
```

Now that you know which SALs are to be included in the moving window of summation, simply use the aggregate function SUM to produce a more expressive result set:

```
select e.hiredate,
       e.sal,
       sum(d.sal) as spending_pattern
  from emp e, emp d
 where d.hiredate
       between e.hiredate-90 and e.hiredate
 group by e.hiredate,e.sal
 order by 1
```

If you compare the result set for the previous query and the result set for the query shown here (which is the original solution presented), you will see they are the same:[]()

```
select e.hiredate,
       e.sal,
       (select sum(sal) from emp d
        where d.hiredate between e.hiredate-90
                             and e.hiredate) as spending_pattern
  from emp e
 order by 1

HIREDATE      SAL SPENDING_PATTERN
----------- ----- ----------------
17-DEC-2010  800               800
20-FEB-2011 1600              2400
22-FEB-2011 1250              3650
02-APR-2011 2975              5825
01-MAY-2011 2850              8675
09-JUN-2011 2450              8275
08-SEP-2011 1500              1500
28-SEP-2011 1250              2750
17-NOV-2011 5000              7750
03-DEC-2011  950             11700
03-DEC-2011 3000             11700
23-JAN-2012 1300             10250
09-DEC-2012 3000              3000
12-JAN-2013 1100              4100
```

# 12.20 Pivoting a Result Set with Subtotals

## Problem

[]()[]()[]()You want to create a report containing subtotals and then transpose the results to provide a more readable report. For example, you’ve been asked to create a report that displays for each department, the managers in the department, and a sum of the salaries of the employees who work for those managers. Additionally, you want to return two subtotals: the sum of all salaries in each department for those employees who have managers, and a sum of all salaries in the result set (the sum of the department subtotals). You currently have the following report:

```
DEPTNO        MGR        SAL
------ ---------- ----------
    10       7782       1300
    10       7839       2450
    10                  3750
    20       7566       6000
    20       7788       1100
    20       7839       2975
    20       7902        800
    20                 10875
    30       7698       6550
    30       7839       2850
    30                  9400
                       24025
```

You want to provide a more readable report and want to transform the previous result set to the following, which makes the meaning of the report much clearer:

```
MGR     DEPT10     DEPT20     DEPT30      TOTAL
---- ---------- ---------- ---------- ----------
7566          0       6000          0
7698          0          0       6550
7782       1300          0          0
7788          0       1100          0
7839       2450       2975       2850
7902          0        800          0
           3750      10875       9400     24025
```

## Solution

[]()The first step is to generate subtotals using the ROLLUP extension to GROUP BY. The next step is to perform a classic pivot (aggregate and CASE expression) to create the desired columns for your report. The GROUPING function allows you to easily determine which values are subtotals (that is, exist because of ROLLUP and otherwise would not normally be there). Depending on how your RDBMS sorts NULL values, you may need to add an ORDER BY to the solution to allow it to look like the previous target result set.

### DB2 and Oracle

Use the ROLLUP extension to GROUP BY and then use a CASE expression to format the data into a more readable report:

```
 1 select mgr,
 2        sum(case deptno when 10 then sal else 0 end) dept10,
 3        sum(case deptno when 20 then sal else 0 end) dept20,
 4        sum(case deptno when 30 then sal else 0 end) dept30,
 5        sum(case flag when '11' then sal else null end) total
 6   from (
 7 select deptno,mgr,sum(sal) sal,
 8        cast(grouping(deptno) as char(1))||
 9        cast(grouping(mgr) as char(1)) flag
10   from emp
11  where mgr is not null
12  group by rollup(deptno,mgr)
13        ) x
14  group by mgr
```

### SQL Server

Use the ROLLUP extension to GROUP BY and then use a CASE expression to format the data into a more readable report:

```
 1 select mgr,
 2        sum(case deptno when 10 then sal else 0 end) dept10,
 3        sum(case deptno when 20 then sal else 0 end) dept20,
 4        sum(case deptno when 30 then sal else 0 end) dept30,
 5        sum(case flag when '11' then sal else null end) total
 6   from (
 7 select deptno,mgr,sum(sal) sal,
 8        cast(grouping(deptno) as char(1))+
 9        cast(grouping(mgr) as char(1)) flag
10   from emp
11  where mgr is not null
12  group by deptno,mgr with rollup
13        ) x
14  group by mgr
```

### PostgreSQL

Use the ROLLUP extension to GROUP BY and then use a CASE expression to format the data into a more readable report:

```
 1   select mgr,
 2          sum(case deptno when 10 then sal else 0 end) dept10,
 3          sum(case deptno when 20 then sal else 0 end) dept20,
 4          sum(case deptno when 30 then sal else 0 end) dept30,
 5          sum(case flag when '11' then sal else null end) total
 6     from (
 7   select deptno,mgr,sum(sal) sal,
 8          concat(cast (grouping(deptno) as char(1)),
 9          cast(grouping(mgr) as char(1))) flag
 10  from emp
 11  where mgr is not null
 12  group by rollup (deptno,mgr)
 13       ) x
 14  group by mgr
```

### MySQL

Use the ROLLUP extension to GROUP BY and then use a CASE expression to format the data into a more readable report:

```
1    select mgr,
2          sum(case deptno when 10 then sal else 0 end) dept10,
3          sum(case deptno when 20 then sal else 0 end) dept20,
4          sum(case deptno when 30 then sal else 0 end) dept30,
5          sum(case flag when '11' then sal else null end) total
6     from (
7    select  deptno,mgr,sum(sal) sal,
8            concat( cast(grouping(deptno) as char(1)) ,
9            cast(grouping(mgr) as char(1))) flag
10   from emp
11  where mgr is not null
12   group by deptno,mgr with rollup
13         ) x
14   group by mgr;
```

## Discussion

[]()The solutions provided here are identical except for the string concatenation and how GROUPING is specified. Because the solutions are so similar, the following discussion will refer to the SQL Server solution to highlight the intermediate result sets (the discussion is relevant to DB2 and Oracle as well).

The first step is to generate a result set that sums the SAL for the employees in each DEPTNO per MGR. The idea is to show how much the employees make under a particular manager in a particular department. For example, the following query will allow you to compare the salaries of employees who work for KING in DEPTNO 10 compared with those who work for KING in DEPTNO 30:

```
select deptno,mgr,sum(sal) sal
  from emp
 where mgr is not null
 group by mgr,deptno
 order by 1,2

DEPTNO        MGR        SAL
------ ---------- ----------
    10       7782       1300
    10       7839       2450
    20       7566       6000
    20       7788       1100
    20       7839       2975
    20       7902        800
    30       7698       6550
    30       7839       2850
```

The next step is to use the ROLLUP extension to GROUP BY to create subtotals for each DEPTNO and across all employees (who have a manager):

```
select deptno,mgr,sum(sal) sal
  from emp
 where mgr is not null
 group by deptno,mgr with rollup

DEPTNO        MGR        SAL
------ ---------- ----------
    10       7782       1300
    10       7839       2450
    10                  3750
    20       7566       6000
    20       7788       1100
    20       7839       2975
    20       7902        800
    20                 10875
    30       7698       6550
    30       7839       2850
    30                  9400
                       24025
```

With the subtotals created, you need a way to determine which values are in fact subtotals (created by ROLLUP) and which are results of the regular GROUP BY. Use the GROUPING function to create bitmaps to help identify the subtotal values from the regular aggregate values:

```
select deptno,mgr,sum(sal) sal,
       cast(grouping(deptno) as char(1))+
       cast(grouping(mgr) as char(1)) flag
  from emp
 where mgr is not null
 group by deptno,mgr with rollup

DEPTNO        MGR        SAL FLAG
------ ---------- ---------- ----
    10       7782       1300 00
    10       7839       2450 00
    10                  3750 01
    20       7566       6000 00
    20       7788       1100 00
    20       7839       2975 00
    20       7902        800 00
    20                 10875 01
    30       7698       6550 00
    30       7839       2850 00
    30                  9400 01
                       24025 11
```

If it isn’t immediately obvious, the rows with a value of 00 for FLAG are the results of regular aggregation. The rows with a value of 01 for FLAG are the results of ROLLUP aggregating SAL by DEPTNO (since DEPTNO is listed first in the ROLLUP; if you switch the order, for example, GROUP BY MGR, DEPTNO WITH ROLLUP, you’d see quite different results). The row with a value of 11 for FLAG is the result of ROLLUP aggregating SAL over all rows.

At this point you have everything you need to create a beautified report by simply using CASE expressions. The goal is to provide a report that shows employee salaries for each manager across departments. If a manager does not have any subordinates in a particular department, a zero should be returned; otherwise, you want to return the sum of all salaries for that manager’s subordinates in that department. Additionally, you want to add a final column, TOTAL, representing a sum of all the salaries in the report. The solution satisfying all these requirements is shown here:[]()[]()[]()

```
select mgr,
       sum(case deptno when 10 then sal else 0 end) dept10,
       sum(case deptno when 20 then sal else 0 end) dept20,
       sum(case deptno when 30 then sal else 0 end) dept30,
       sum(case flag when '11' then sal else null end) total
  from (
select deptno,mgr,sum(sal) sal,
       cast(grouping(deptno) as char(1))+
       cast(grouping(mgr) as char(1)) flag
  from emp
 where mgr is not null
 group by deptno,mgr with rollup
       ) x
 group by mgr
 order by coalesce(mgr,9999)

MGR     DEPT10     DEPT20     DEPT30     TOTAL
---- ---------- ---------- ---------- ----------
7566          0       6000          0
7698          0          0       6550
7782       1300          0          0
7788          0       1100          0
7839       2450       2975       2850
7902          0        800          0
           3750      10875       9400    24025
```

# 12.21 Summing Up

Databases are for storing data, but eventually someone needs to retrieve the data and present it somewhere. The recipes in this chapter show a variety of important ways that data can be re-shaped or formatted to meet the needs of users. Apart from their general usefulness in giving users data in the form they need, these techniques play an important role in giving a database owner the ability to create a datawarehouse.

As you gain more experience in supporting users in the business, you will become more adept and extend the ideas here into more elaborate presentations.[]()