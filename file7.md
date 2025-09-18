# Chapter 7. Working with Numbers

[]()This chapter focuses on common operations involving numbers, including numeric computations. While SQL is not typically considered the first choice for complex computations, it is efficient for day-to-day numeric chores. More importantly, as databases and datawarehouses supporting SQL probably remain the most common place to find an organization’s data, using SQL to explore and evaluate that data is essential for anyone putting that data to work. The techniques in this section have also been chosen to help data scientists decide which data is the most promising for further analysis.

###### Tip

Some recipes in this chapter make use of aggregate functions and the GROUP BY clause. If you are not familiar with grouping, please read at least the first major section, called “Grouping,” in [Appendix A](https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/app01.html#sqlckbk-APP-A).

# 7.1 Computing an Average

## Problem

[]()[]()You want to compute the average value in a column, either for all rows in a table or for some subset of rows. For example, you might want to find the average salary for all employees as well as the average salary for each department.

## Solution

[]()When computing the average of all employee salaries, simply apply the AVG function to the column containing those salaries.

By excluding a WHERE clause, the average is computed against all non-NULL values:

```
1 select avg(sal) as avg_sal 
 2   from emp 

   AVG_SAL
----------
2073.21429
```

To compute the average salary for each department, use the GROUP BY clause to create a group corresponding to each department:

```
1 select deptno, avg(sal) as avg_sal 
 2   from emp 
 3  group by deptno 

    DEPTNO     AVG_SAL
----------  ----------
        10  2916.66667
        20        2175
        30  1566.66667
```

## Discussion

When finding an average where the whole table is the group or window, simply apply the AVG function to the column you are interested in without using the GROUP BY clause. []()It is important to realize that the function AVG ignores NULLs. The effect of NULL values being ignored can be seen here:

```
create table t2(sal integer)
insert into t2 values (10)
insert into t2 values (20)
insert into t2 values (null)
 select avg(sal)    select distinct 30/2 
   from t2            from t2 

  AVG(SAL)               30/2
----------         ----------
        15                 15


 select avg(coalesce(sal,0))    select distinct 30/3 
   from t2                        from t2 

AVG(COALESCE(SAL,0))                 30/3
--------------------           ----------
                  10                   10
```

[]()The COALESCE function will return the first non-NULL value found in the list of values that you pass. When NULL SAL values are converted to zero, the average changes. When invoking aggregate functions, always give thought to how you want NULLs handled.

The second part of the solution uses GROUP BY (line 3) to divide employee records into groups based on department affiliation. GROUP BY automatically causes aggregate functions such as AVG to execute and return a result for each group. In this example, AVG would execute once for each department-based group of employee records.

It is not necessary, by the way, to include GROUP BY columns in your select list. For example:

```
select avg(sal) 
   from emp 
  group by deptno 

  AVG(SAL)
----------
2916.66667
      2175
1566.66667
```

You are still grouping by DEPTNO even though it is not in the SELECT clause. Including the column you are grouping by in the SELECT clause often improves readability, but is not mandatory. It is mandatory, however, to avoid placing columns in your SELECT list that are not also in your GROUP BY clause.

## See Also

See [Appendix A](https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/app01.html#sqlckbk-APP-A) for a refresher on GROUP BY functionality.[]()[]()[]()

# 7.2 Finding the Min/Max Value in a Column

## Problem

[]()[]()[]()You want to find the highest and lowest values in a given column. For example, you want to find the highest and lowest salaries for all employees, as well as the highest and lowest salaries for each department.

## Solution

[]()[]()When searching for the lowest and highest salaries for all employees, simply use the functions MIN and MAX, respectively:

```
1 select min(sal) as min_sal, max(sal) as max_sal 
   2   from emp 

   MIN_SAL     MAX_SAL
----------  ----------
       800        5000
```

When searching for the lowest and highest salaries for each department, use the functions MIN and MAX with the GROUP BY clause:

```
1 select deptno, min(sal) as min_sal, max(sal) as max_sal 
   2   from emp 
   3  group by deptno 

        DEPTNO     MIN_SAL     MAX_SAL
    ----------  ----------  ----------
            10        1300        5000
            20         800        3000
            30         950        2850
```

## Discussion

When searching for the highest or lowest values, and in cases where the whole table is the group or window, simply apply the MIN or MAX function to the column you are interested in without using the GROUP BY clause.

[]()Remember that the MIN and MAX functions ignore NULLs, and that you can have NULL groups as well as NULL values for columns in a group. The following are examples that ultimately lead to a query using GROUP BY that returns NULL values for two groups (DEPTNO 10 and 20):

```
select deptno, comm 
   from emp 
  where deptno in (10,30) 
  order by 1 


     DEPTNO       COMM
 ---------- ----------
         10
         10
         10
         30        300
         30        500
         30
         30          0
         30       1300
         30


 select min(comm), max(comm) 
   from emp 

 MIN(COMM)   MAX(COMM)
----------  ----------
         0        1300


 select deptno, min(comm), max(comm) 
   from emp 
  group by deptno 

     DEPTNO  MIN(COMM)   MAX(COMM)
 ---------- ----------  ----------
         10
         20
         30          0        1300
```

[]()Remember, as [Appendix A](https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/app01.html#sqlckbk-APP-A) points out, even if nothing other than aggregate functions are listed in the SELECT clause, you can still group by other columns in the table; for example:

```
select min(comm), max(comm)
  from emp
 group by deptno

 MIN(COMM)   MAX(COMM)
----------  ----------
         0        1300
```

[]()Here you are still grouping by DEPTNO even though it is not in the SELECT clause. Including the column you are grouping by in the SELECT clause often improves readability, but is not mandatory. It is mandatory, however, that any column in the SELECT list of a GROUP BY query also be listed in the GROUP BY clause.

## See Also

See [Appendix A](https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/app01.html#sqlckbk-APP-A) for a refresher on GROUP BY functionality.[]()[]()[]()

# 7.3 Summing the Values in a Column

## Problem

[]()[]()You want to compute the sum of all values, such as all employee salaries, in a column.

## Solution

[]()When computing a sum where the whole table is the group or window, just apply the SUM function to the columns you are interested in without using the GROUP BY clause:

```
1 select sum(sal) 
 2  from emp 

  SUM(SAL)
----------
     29025
```

When creating multiple groups or windows of data, use the SUM function with the GROUP BY clause. The following example sums employee salaries by department:

```
1 select deptno, sum(sal) as total_for_dept 
 2   from emp 
 3  group by deptno 


    DEPTNO  TOTAL_FOR_DEPT
----------  --------------
        10            8750
        20           10875
        30            9400
```

## Discussion

When searching for the sum of all salaries for each department, you are creating groups or “windows” of data. Each employee’s salary is added together to produce a total for their respective department. This is an example of aggregation in SQL because detailed information, such as each individual employee’s salary, is not the focus; the focus is the end result for each department. []()[]()It is important to note that the SUM function will ignore NULLs, but you can have NULL groups, which can be seen here. DEPTNO 10 does not have any employees who earn a commission; thus, grouping by DEPTNO 10 while attempting to SUM the values in COMM will result in a group with a NULL value returned by SUM:

```
select deptno, comm 
   from emp 
  where deptno in (10,30) 
  order by 1 

    DEPTNO       COMM
---------- ----------
        10
        10
        10
        30        300
        30        500
        30
        30          0
        30       1300
        30


 select sum(comm) 
   from emp 

  SUM(COMM) 
 ---------- 
       2100 

 select deptno, sum(comm) 
   from emp 
  where deptno in (10,30) 
  group by deptno 

    DEPTNO   SUM(COMM)
----------  ----------
        10
        30        2100
```

## See Also

See [Appendix A](https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/app01.html#sqlckbk-APP-A) for a refresher on GROUP BY functionality.[]()[]()

# 7.4 Counting Rows in a Table

## Problem

[]()You want to count the number of rows in a table, or you want to count the number of values in a column. For example, you want to find the total number of employees as well as the number of employees in each department.

## Solution

[]()[]()When counting rows where the whole table is the group or window, simply use the COUNT function along with the * character:

```
1 select count(*) 
 2  from emp 

  COUNT(*)
----------
        14
```

[]()When creating multiple groups, or windows of data, use the COUNT function with the GROUP BY clause:

```
1 select deptno, count(*) 
 2   from emp 
 3  group by deptno 

    DEPTNO    COUNT(*)
----------  ----------
        10           3
        20           5
        30           6
```

## Discussion

When counting the number of employees for each department, you are creating groups or “windows” of data. Each employee found increments the count by one to produce a total for their respective department. This is an example of aggregation in SQL because detailed information, such as each individual employee’s salary or job, is not the focus; the focus is the end result for each department. []()[]()It is important to note that the COUNT function will ignore NULLs when passed a column name as an argument, but will include NULLs when passed the * character or any constant; consider the following:

```
select deptno, comm 
   from emp 

    DEPTNO        COMM
----------  ----------
        20
        30         300
        30         500
        20
        30        1300
        30
        10
        20
        10
        30           0
        20
        30
        20
        10


 select count(*), count(deptno), count(comm), count('hello') 
   from emp 

  COUNT(*)  COUNT(DEPTNO)   COUNT(COMM)  COUNT('HELLO')
----------  -------------   -----------  --------------
        14             14             4              14


 select deptno, count(*), count(comm), count('hello') 
   from emp 
  group by deptno 

     DEPTNO   COUNT(*)  COUNT(COMM)  COUNT('HELLO')
 ---------- ----------  -----------  --------------
         10          3            0               3
         20          5            0               5
         30          6            4               6
```

If all rows are null for the column passed to COUNT or if the table is empty, COUNT will return zero. It should also be noted that, even if nothing other than aggregate functions are specified in the SELECT clause, you can still group by other columns in the table, for example:

```
select count(*) 
   from emp 
  group by deptno 

   COUNT(*)
 ----------
          3
          5
          6
```

Notice that you are still grouping by DEPTNO even though it is not in the SELECT clause. Including the column you are grouping by in the SELECT clause often improves readability, but is not mandatory. If you do include it (in the SELECT list), it is mandatory that it is listed in the GROUP BY clause.

## See Also

See [Appendix A](https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/app01.html#sqlckbk-APP-A) for a refresher on GROUP BY functionality.[]()

# 7.5 Counting Values in a Column

## Problem

[]()You want to count the number of non-NULL values in a column. For example, you’d like to find out how many employees are on commission.

## Solution

Count the number of non-NULL values in the EMP table’s COMM column:

```
select count(comm) 
   from emp 

COUNT(COMM)
-----------
          4
```

## Discussion

[]()[]()[]()When you “count star,” as in COUNT(\*), what you are really counting is rows (regardless of actual value, which is why rows containing NULL and non-NULL values are counted). But when you COUNT a column, you are counting the number of non-NULL values in that column. The previous recipe’s discussion touches on this distinction. In this solution, COUNT(COMM) returns the number of non-NULL values in the COMM column. Since only commissioned employees have commissions, the result of COUNT(COMM) is the number of such employees.

# 7.6 Generating a Running Total

## Problem

[]()[]()You want to calculate a running total of values in a column.

## Solution

As an example, the following solutions show how to compute a running total of salaries for all employees. For readability, results are ordered by SAL whenever possible so that you can easily eyeball the progression of the running total.[]()

```
1 select ename, sal, 
 2        sum(sal) over (order by sal,empno) as running_total 
 3   from emp 
 4   order by 2 


ENAME              SAL  RUNNING_TOTAL
----------  ----------  -------------
SMITH              800            800
JAMES              950           1750
ADAMS             1100           2850
WARD              1250           4100
MARTIN            1250           5350
MILLER            1300           6650
TURNER            1500           8150
ALLEN             1600           9750
CLARK             2450          12200
BLAKE             2850          15050
JONES             2975          18025
SCOTT             3000          21025
FORD              3000          24025
KING              5000          29025
```

## Discussion

[]()The windowing function SUM OVER makes generating a running total a simple task. []()The ORDER BY clause in the solution includes not only the SAL column, but also the EMPNO column (which is the primary key) to avoid duplicate values in the running total. The column RUNNING\_TOTAL2 in the following example illustrates the problem that you might otherwise have with duplicates:

```
select empno, sal, 
        sum(sal)over(order by sal,empno) as running_total1, 
        sum(sal)over(order by sal) as running_total2 
   from emp 
  order by 2 

ENAME              SAL  RUNNING_TOTAL1  RUNNING_TOTAL2
----------  ----------  --------------  --------------
SMITH              800             800             800
JAMES              950            1750            1750
ADAMS             1100            2850            2850
WARD              1250            4100            5350
MARTIN            1250            5350            5350
MILLER            1300            6650            6650
TURNER            1500            8150            8150
ALLEN             1600            9750            9750
CLARK             2450           12200           12200
BLAKE             2850           15050           15050
JONES             2975           18025           18025
SCOTT             3000           21025           24025
FORD              3000           24025           24025
KING              5000           29025           29025
```

The values in RUNNING\_TOTAL2 for WARD, MARTIN, SCOTT, and FORD are incorrect. Their salaries occur more than once, and those duplicates are summed and added to the running total. This is why EMPNO (which is unique) is needed to produce the (correct) results that you see in RUNNING\_TOTAL1. Consider this: for ADAMS you see 2850 for RUNNING\_TOTAL1 and RUNNING\_TOTAL2. Add WARD’s salary of 1250 to 2850 and you get 4100, yet RUNNING\_TOTAL2 returns 5350. Why? Since WARD and MARTIN have the same SAL, their two 1250 salaries are added together to yield 2500, which is then added to 2850 to arrive at 5350 for both WARD and MARTIN. By specifying a combination of columns to order by that cannot result in duplicate values (e.g., any combination of SAL and EMPNO is unique), you ensure the correct progression of the running total.

# 7.7 Generating a Running Product

## Problem

[]()[]()You want to compute a running product on a numeric column. The operation is similar to [Recipe 7.6](#sqlckbk-CHP-7-SECT-6), but using multiplication instead of addition.

## Solution

By way of example, the solutions all compute running products of employee salaries. While a running product of salaries may not be all that useful, the technique can easily be applied to other, more useful domains.

[]()[]()Use the windowing function SUM OVER and take advantage of the fact that you can simulate multiplication by adding logarithms:

```
1 select empno,ename,sal, 
 2        exp(sum(ln(sal))over(order by sal,empno)) as running_prod 
 3   from emp 
 4  where deptno = 10 

EMPNO ENAME          SAL            RUNNING_PROD
----- ----------    ----    --------------------
 7934 MILLER        1300                    1300
 7782 CLARK         2450                 3185000
 7839 KING          5000             15925000000
```

It is not valid in SQL (or, formally speaking, in mathematics) to compute logarithms of values less than or equal to zero. If you have such values in your tables, you need to avoid passing those invalid values to SQL’s LN function. Precautions against invalid values and NULLs are not provided in this solution for the sake of readability, but you should consider whether to place such precautions in production code that you write. If you absolutely must work with negative and zero values, then this solution may not work for you. At the same time, if you have zeros (but no values below zero), a common workaround is to add 1 to all values, noting that the logarithm of 1 is always zero regardless of base.

SQL Server users use LOG instead of LN.

## Discussion

The solution takes advantage of the fact that you can multiply two numbers by:

1. Computing their respective natural logarithms
2. Summing those logarithms
3. Raising the result to the power of the mathematical constant *e* (using the EXP function)

The one caveat when using this approach is that it doesn’t work for summing zero or negative values, because any value less than or equal to zero is out of range for an SQL logarithm.

For an explanation of how the window function SUM OVER works, see [Recipe 7.6](#sqlckbk-CHP-7-SECT-6).

# 7.8 Smoothing a Series of Values

## Problem

[]()You have a series of values that appear over time, such as monthly sales figures. As is common, the data shows a lot of variation from point to point, but you are interested in the overall trend. Therefore, you want to implement a simple smoother, such as weighted running average to better identify the trend.

Imagine you have daily sales totals, in dollars, such as from a newsstand:

```
DATE1            SALES
2020-01-01         647
2020-01-02         561
2020-01-03         741
2020-01-04         978
2020-01-05        1062
2020-01-06        1072
...                ...
```

However, you know that there is volatility to the sales data that makes it difficult to discern an underlying trend. Possibly different days of the week or month are known to have especially high or low sales. Alternatively, maybe you are aware that due to the way the data is collected, sometimes sales for one day are moved into the next day, creating a trough followed by a peak, but there is no practical way to allocate the sales to their correct day. Therefore, you need to smooth the data over a number of days to achieve a proper view of what’s happening.

A moving average can be calculated by summing the current value and the preceding *n*-1 values and dividing by *n*. If you also display the previous values for reference, you expect something like this:

```
DATE1         sales    salesLagOne   SalesLagTwo    MovingAverage
-----        ------   -----------   ------------  --------------
2020-01-01    647       NULL           NULl                  NULL
2020-01-02    561       647            NULL                  NULL
2020-01-03    741       561            647                649.667
2020-01-04    978       741            561                    760
2020-01-05    1062      978            741                    927
2020-01-06    1072      1062           978               1037.333
2020-01-07    805       1072           1062               979.667
2020-01-08    662       805            1072               846.333
2020-01-09    1083      662            805                    850
2020-01-10    970       1083           662                    905
```

## Solution

The formula for the mean is well known. By applying a simple weighting to the formula, we can make it more relevant for this task by giving more weight to more recent values. []()Use the window function LAG to create a moving average:

```
select date1, sales,lag(sales,1) over(order by date1) as salesLagOne,
lag(sales,2) over(order by date1) as salesLagTwo,
(sales
+ (lag(sales,1) over(order by date1))
+ lag(sales,2) over(order by date1))/3 as MovingAverage
from sales
```

## Discussion

A weighted moving average is one of the simplest ways to analyze time-series data (data that appears at particular time intervals). This is just one way to calculate a simple moving average—you can also use a partition with average. Although we have selected a simple three-point moving average, there are different formulas with differing numbers of points according to the characteristics of the data you apply them \[.keep-together]#to—#that’s where this technique really comes into its own.

For example, a simple three-point weighted moving average that emphasizes the most recent data point could be implemented with the following variant on the solution, where coefficients and the denominator have been updated:

```
select date1, sales,lag(sales,1) over(order by date1),
lag(sales,2) over(order by date1),
((3*sales)
+ (2*(lag(sales,1) over(order by date1)))
+ (lag(sales,2) over(order by date1)))/6 as SalesMA
from sales
```

# 7.9 Calculating a Mode

## Problem

[]()[]()You want to find the mode (for those of you who don’t recall, the *mode* in mathematics is the element that appears most frequently for a given set of data) of the values in a column. For example, you want to find the mode of the salaries in DEPTNO 20.

Based on the following salaries:

```
select sal 
   from emp 
  where deptno = 20 
  order by sal 


       SAL
----------
       800
      1100
      2975
      3000
      3000
```

the mode is 3000.

## Solution

### DB2, MySQL, PostgreSQL, and SQL Server

[]()Use the window function DENSE\_RANK to rank the counts of the salaries to facilitate extracting the mode:

```
 1 select sal
 2   from (
 3 select sal,
 4        dense_rank()over( order by cnt desc) as rnk
 5   from (
 6 select sal, count(*) as cnt
 8   from emp
 9  where deptno = 20
10  group by sal
11        ) x
12        ) y
13  where rnk = 1
```

### Oracle

[]()[]()You can use the KEEP extension to the aggregate function MAX to find the mode SAL. One important note is that if there are ties, i.e., multiple rows that are the mode, the solution using KEEP will keep only one, and that is the one with the highest salary. If you want to see all modes (if more than one exists), you must modify this solution or simply use the DB2 solution presented earlier. In this case, since 3000 is the mode SAL in DEPTNO 20 and is also the highest SAL, this solution is sufficient:

```
1 select max(sal)
2        keep(dense_rank first order by cnt desc) sal
3   from (
4 select sal, count(*) cnt
5   from emp
6  where deptno=20
7  group by sal
8        )
```

## Discussion

### DB2 and SQL Server

The inline view X returns each SAL and the number of times it occurs. Inline view Y uses the window function DENSE\_RANK (which allows for ties) to sort the results.

The results are ranked based on the number of times each SAL occurs, as shown here:

```
1 select sal,
2        dense_rank()over(order by cnt desc) as rnk
3   from (
4 select sal,count(*) as cnt
5   from emp
6  where deptno = 20
7  group by sal
8        ) x

  SAL          RNK
-----   ----------
 3000            1
  800            2
 1100            2
 2975            2
```

The outermost portion of query simply keeps the row(s) where RNK is 1.

### Oracle

The inline view returns each SAL and the number of times it occurs and is shown here:

```
select sal, count(*) cnt 
     from emp 
    where deptno=20 
    group by sal 

      SAL          CNT
    -----   ----------
      800            1
     1100            1
     2975            1
     3000            2
```

[]()The next step is to use the KEEP extension of the aggregate function MAX to find the mode. If you analyze the KEEP clause shown here, you will notice three subclauses, DENSE\_RANK, FIRST, and ORDER BY CNT DESC:

```
keep(dense_rank first order by cnt desc)
```

This makes finding the mode extremely convenient. The KEEP clause determines which SAL will be returned by MAX by looking at the value of CNT returned by the inline view. Working from right to left, the values for CNT are ordered in descending order; then the first is kept of all the values for CNT returned in DENSE\_RANK order. Looking at the result set from the inline view, you can see that 3000 has the highest CNT of 2. The MAX(SAL) returned is the greatest SAL that has the greatest CNT, in this case 3000.

## See Also

See [Chapter 11](https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/ch11.html#sqlckbk-CHP-11), particularly the section on “Finding Knight Values,” for a deeper discussion of Oracle’s KEEP extension of aggregate functions.[]()[]()

# 7.10 Calculating a Median

## Problem

[]()[]()You want to calculate the median (for those of who do not recall, the *median* is the value of the middle member of a set of ordered elements) value for a column of numeric values. For example, you want to find the median of the salaries in DEPTNO 20. Based on the following salaries:

```
select sal 
   from emp 
  where deptno = 20 
  order by sal 

       SAL
----------
       800
      1100
      2975
      3000
      3000
```

the median is 2975.

## Solution

Other than the Oracle solution (which uses supplied functions to compute a median), the introduction of window functions allows for a more efficient solution compared to the traditional self-join.

### DB2 and PostgreSQL

[]()Use the window function PERCENTILE\_CONT to find the median:

```
1 select percentile_cont(0.5)
2         within group(order by sal)
3   from emp
4  where deptno=20
```

### SQL Server

Use the window function PERCENTILE\_CONT to find the median:

```
1 select percentile_cont(0.5)
    2         within group(order by sal)
    3         over()
    4   from emp
    5  where deptno=20
```

The SQL Server solution works on the same principle but requires an OVER clause.

### MySQL

MySQL doesn’t have the PERCENTILE\_CONT function, so a workaround is required. []()One way is to use the CUME\_DIST function in conjunction with a CTE, effectively re-creating the PERCENTILE\_CONT function:

```
with rank_tab (sal, rank_sal) as
(
select sal, cume_dist() over (order by sal)
            from emp
            where deptno=20
),

inter as
(
        select sal, rank_sal from rank_tab
        where rank_sal>=0.5
union
        select sal, rank_sal from rank_tab
        where rank_sal<=0.5
)

        select avg(sal) as MedianSal
               from inter
```

### Oracle

[]()Use the functions MEDIAN or PERCENTILE\_CONT:

```
1 select median(sal)
2   from emp
3  where deptno=20

1 select percentile_cont(0.5)
2         within group(order by sal)
3   from emp
4  where deptno=20
```

## Discussion

### Oracle, PostgreSQL, SQL Server, and DB2

Other than Oracle’s MEDIAN function, the structure of all the solutions is the same. The PERCENTILE\_CONT function allows you to directly apply the definition of a median, as the median is by definition the 50th percentile. Hence, applying this function with the appropriate syntax and using 0.5 as the argument finds the median.

Of course, other percentiles are also available from this function. For example, you can look for the 5th and/or 95th percentiles to find outliers (another method of finding outliers is outlined later in this chapter when we discuss the median absolute deviation).

### MySQL

MySQL doesn’t have a PERCENTILE\_CONT function, which makes things trickier. To find the median, the values for SAL must be ordered from lowest to highest. The CUME\_DIST function achieves this goal and labels each row with its percentile. Hence, it can be used to achieve the same outcome as the PERCENTILE\_CONT function used in the solution for the other databases.[]()

The only difficulty is that the CUME\_DIST function is not permitted in a WHERE clause. As a result, you need to apply it first in a CTE.

The only trap here is that if the number of rows is even, there won’t be a row exactly on the median. Hence, the solution is written to find the average of the highest value below or equal to the median, and the lowest value above or equal to the median. This method works for both odd and even numbers of rows, and if there is an odd number of rows giving an exact median, it will take average of two numbers that are equal.[]()[]()

# 7.11 Determining the Percentage of a Total

## Problem

[]()[]()You want to determine the percentage that values in a specific column represent against a total. For example, you want to determine what percentage of all salaries are the salaries in DEPTNO 10 (the percentage that DEPTNO 10 salaries contribute to the total).

## Solution

In general, computing a percentage against a total in SQL is no different than doing so on paper: simply divide, then multiply. In this example you want to find the percentage of total salaries in table EMP that come from DEPTNO 10. To do that, simply find the salaries for DEPTNO 10, and then divide by the total salary for the table. As the last step, multiply by 100 to return a value that represents a percent.

### MySQL and PostgreSQL

Divide the sum of the salaries in DEPTNO 10 by the sum of all salaries:

```
1 select (sum(
2          case when deptno = 10 then sal end)/sum(sal)
3         )*100 as pct
4   from emp
```

### DB2, Oracle, and SQL Server

[]()Use an inline view with the window function SUM OVER to find the sum of all salaries along with the sum of all salaries in DEPTNO 10. Then do the division and multiplication in the outer query:

```
1 select distinct (d10/total)*100 as pct
2   from (
3 select deptno,
4        sum(sal)over() total,
5        sum(sal)over(partition by deptno) d10
6   from emp
7        ) x
8  where deptno=10
```

## Discussion

### MySQL and PostgreSQL

[]()The CASE statement conveniently returns only the salaries from DEPTNO 10. They are then summed and divided by the sum of all the salaries. Because NULLs are ignored by aggregates, an ELSE clause is not needed in the CASE statement. To see exactly which values are divided, execute the query without the division:

```
select sum(case when deptno = 10 then sal end) as d10, 
        sum(sal) 
   from emp 

D10  SUM(SAL)
---- ---------
8750     29025
```

Depending on how you define SAL, you may need to explicitly use CAST when performing division to ensure the correct data type. For example, on DB2, SQL Server, and PostgreSQL, if SAL is stored as an integer, you can apply CAST to ensure a decimal value is returned, as shown here:

```
select (cast(
         sum(case when deptno = 10 then sal end)
            as decimal)/sum(sal)
        )*100 as pct
  from emp
```

### DB2, Oracle, and SQL Server

As an alternative to the traditional solution, this solution uses window functions to compute a percentage relative to the total. For DB2 and SQL Server, if you’ve stored SAL as an integer, you’ll need to use CAST before dividing:

```
select distinct
       cast(d10 as decimal)/total*100 as pct
  from (
select deptno,
       sum(sal)over() total,
       sum(sal)over(partition by deptno) d10
  from emp
       ) x
 where deptno=10
```

It is important to keep in mind that window functions are applied after the WHERE clause is evaluated. Thus, the filter on DEPTNO cannot be performed in inline view X. Consider the results of inline view X without and with the filter on DEPTNO. First without:

```
select deptno, 
        sum(sal)over() total, 
        sum(sal)over(partition by deptno) d10 
   from emp 

DEPTNO      TOTAL       D10
------- --------- ---------
     10     29025      8750
     10     29025      8750
     10     29025      8750
     20     29025     10875
     20     29025     10875
     20     29025     10875
     20     29025     10875
     20     29025     10875
     30     29025      9400
     30     29025      9400
     30     29025      9400
     30     29025      9400
     30     29025      9400
     30     29025      9400
```

and now with:

```
select deptno, 
        sum(sal)over() total, 
        sum(sal)over(partition by deptno) d10 
   from emp 
  where deptno=10 

DEPTNO     TOTAL       D10
------ --------- ---------
    10      8750      8750
    10      8750      8750
    10      8750      8750
```

Because window functions are applied after the WHERE clause, the value for TOTAL represents the sum of all salaries in DEPTNO 10 only. But to solve the problem you want the TOTAL to represent the sum of all salaries, period. That’s why the filter on DEPTNO must happen outside of inline view X.[]()[]()

# 7.12 Aggregating Nullable Columns

## Problem

[]()[]()[]()You want to perform an aggregation on a column, but the column is nullable. You want the accuracy of your aggregation to be preserved, but are concerned because aggregate functions ignore NULLs. For example, you want to determine the average commission for employees in DEPTNO 30, but there are some employees who do not earn a commission (COMM is NULL for those employees). Because NULLs are ignored by aggregates, the accuracy of the output is compromised. You would like to somehow include NULL values in your aggregation.

## Solution

[]()Use the COALESCE function to convert NULLs to zero so they will be included in the aggregation:

```
1 select avg(coalesce(comm,0)) as avg_comm
2   from emp
3  where deptno=30
```

## Discussion

When working with aggregate functions, keep in mind that NULLs are ignored. Consider the output of the solution without using the COALESCE function:

```
select avg(comm) 
   from emp 
  where deptno=30 

 AVG(COMM)
 ---------
       550
```

This query shows an average commission of 550 for DEPTNO 30, but a quick examination of those rows:

```
select ename, comm 
   from emp 
  where deptno=30 
 order by comm desc 

ENAME           COMM
---------- ---------
BLAKE
JAMES
MARTIN          1400
WARD             500
ALLEN            300
TURNER             0
```

shows that only four of the six employees can earn a commission. The sum of all commissions in DEPTNO 30 is 2200, and the average should be 2200/6, not 2200/4. By excluding the COALESCE function, you answer the question “What is the average commission of employees in DEPTNO 30 *who can earn a commission*?” rather than “What is the average commission of all employees in DEPTNO 30?” When working with aggregates, remember to treat NULLs accordingly.

# 7.13 Computing Averages Without High and Low Values

## Problem

[]()You want to compute an average, but you want to []()exclude the highest and lowest values to (hopefully) reduce the effect of skew. In statistical language, this is known as a *trimmed mean*. For example, you want to compute the average salary of all employees excluding the highest and lowest salaries.

## Solution

### MySQL and PostgreSQL

Use subqueries to exclude high and low values:

```
1 select avg(sal)
2   from emp
3  where sal not in (
4     (select min(sal) from emp),
5     (select max(sal) from emp)
6  )
```

### DB2, Oracle, and SQL Server

Use an inline view with the windowing functions MAX OVER and MIN OVER to generate a result set from which you can easily eliminate the high and low values:

```
1 select avg(sal)
2   from (
3 select sal, min(sal)over() min_sal, max(sal)over() max_sal
4   from emp
5        ) x
6  where sal not in (min_sal,max_sal)
```

## Discussion

### MySQL and PostgreSQL

The subqueries return the highest and lowest salaries in the table. By using NOT IN against the values returned, you exclude the highest and lowest salaries from the average. Keep in mind that if there are duplicates (if multiple employees have the highest or lowest salaries), they will all be excluded from the average. If your goal is to exclude only a single instance of the high and low values, simply subtract them from the SUM and then divide:

```
select (sum(sal)-min(sal)-max(sal))/(count(*)-2)
  from emp
```

### DB2, Oracle, and SQL Server

Inline view X returns each salary along with the highest and lowest salaries:

```
select sal, min(sal)over() min_sal, max(sal)over() max_sal 
   from emp 

      SAL   MIN_SAL   MAX_SAL
--------- --------- ---------
      800       800      5000
     1600       800      5000
     1250       800      5000
     2975       800      5000
     1250       800      5000
     2850       800      5000
     2450       800      5000
     3000       800      5000
     5000       800      5000
     1500       800      5000
     1100       800      5000
      950       800      5000
     3000       800      5000
     1300       800      5000
```

You can access the high and low salaries at every row, so finding which salaries are highest and/or lowest is trivial. The outer query filters the rows returned from inline view X such that any salary that matches either MIN\_SAL or MAX\_SAL is excluded from the average.

# Robust Statistics

[]()[]()In statistical parlance, a mean calculated with the largest and smallest values removed is called a *trimmed mean*. This can be considered a safer estimate of the average, and is an example of a *robust statistic*, so called because they are less sensitive to problems such as bias. [Recipe 7.16](#sqlckbk-CHP-7-SECT-16) is another example of a robust statistical tool. In both cases, these approaches are valuable to someone analyzing data within an RDBMS because they don’t require the analyst to make assumptions that are difficult to test with the relatively limited range of statistical tools available in SQL.[]()

# 7.14 Converting Alphanumeric Strings into Numbers

## Problem

[]()[]()[]()You have alphanumeric data and would like to return numbers only. You want to return the number 123321 from the string “paul123f321.”

## Solution

### DB2

Use the functions TRANSLATE and REPLACE to extract numeric characters from an alphanumeric string:

```
1 select cast(
2        replace(
3      translate( 'paul123f321',
4                 repeat('#',26),
5                 'abcdefghijklmnopqrstuvwxyz'),'#','')
6        as integer ) as num
7   from t1
```

### Oracle, SQL Server, and PostgreSQL

Use the functions TRANSLATE and REPLACE to extract numeric characters from an alphanumeric string:

```
1 select cast(
2        replace(
3      translate( 'paul123f321',
4                 'abcdefghijklmnopqrstuvwxyz',
5                 rpad('#',26,'#')),'#','')
6        as integer ) as num
7   from t1
```

### MySQL

As of the time of this writing, MySQL doesn’t support the TRANSLATE function; thus, a solution will not be provided.

## Discussion

The only difference between the two solutions is syntax; DB2 uses the function REPEAT rather than RPAD, and the parameter list for TRANSLATE is in a different order. The following explanation uses the Oracle/PostgreSQL solution but is relevant to DB2 as well. If you run query inside out (starting with TRANSLATE only), you’ll see this is simple. First, TRANSLATE converts any nonnumeric character to an instance of #:

```
select translate( 'paul123f321', 
                   'abcdefghijklmnopqrstuvwxyz', 
                   rpad('#',26,'#')) as num 
   from t1 

NUM
-----------
####123#321
```

Since all nonnumeric characters are now represented by #, simply use REPLACE to remove them, then use CAST the return the result as a number. This particular example is extremely simple because the data is alphanumeric. If additional characters can be stored, rather than fishing for those characters, it is easier to approach this problem differently: rather than finding nonnumeric characters and then removing them, find all numeric characters and remove anything that is not among them. The following example will help clarify this technique:

```
select replace( 
      translate('paul123f321', 
        replace(translate( 'paul123f321', 
                           '0123456789', 
                           rpad('#',10,'#')),'#',''), 
                rpad('#',length('paul123f321'),'#')),'#','') as num 
   from t1 

NUM
-----------
123321
```

This solution looks a bit more convoluted than the original but is not so bad once you break it down. Observe the innermost call to TRANSLATE:

```
select translate( 'paul123f321', 
                   '0123456789', 
                   rpad('#',10,'#')) 
   from t1 

TRANSLATE('
-----------
paul###f###
```

So, the initial approach is different; rather than replacing each nonnumeric character with an instance of #, you replace each numeric character with an instance of #. The next step removes all instances of #, thus leaving only nonnumeric characters:

```
select replace(translate( 'paul123f321', 
                           '0123456789', 
                           rpad('#',10,'#')),'#','') 
   from t1 

REPLA
-----
paulf
```

The next step is to call TRANSLATE again, this time to replace each of the nonnumeric characters (from the previous query) with an instance of # in the original string:

```
select translate('paul123f321', 
        replace(translate( 'paul123f321', 
                           '0123456789', 
                           rpad('#',10,'#')),'#',''), 
                rpad('#',length('paul123f321'),'#')) 
   from t1 

TRANSLATE('
-----------
####123#321
```

At this point, stop and examine the outermost call to TRANSLATE. The second parameter to RPAD (or the second parameter to REPEAT for DB2) is the length of the original string. This is convenient to use since no character can occur enough times to be greater than the string it is part of. Now that all nonnumeric characters are replaced by instances of #, the last step is to use REPLACE to remove all instances of #. Now you are left with a number.[]()[]()[]()

# 7.15 Changing Values in a Running Total

## Problem

[]()[]()You want to modify the values in a running total depending on the values in another column. Consider a scenario where you want to display the transaction history of a credit card account along with the current balance after each transaction. The following view, V, will be used in this example:

```
create view V (id,amt,trx) 
 as 
 select 1, 100, 'PR' from t1 union all 
 select 2, 100, 'PR' from t1 union all 
 select 3, 50,  'PY' from t1 union all 
 select 4, 100, 'PR' from t1 union all 
 select 5, 200, 'PY' from t1 union all 
 select 6, 50,  'PY' from t1 

   select * from V 

ID         AMT  TR
--  ----------  --
 1         100  PR
 2         100  PR
 3          50  PY
 4         100  PR
 5         200  PY
 6          50  PY
```

The ID column uniquely identifies each transaction. The AMT column represents the amount of money involved in each transaction (either a purchase or a payment). The TRX column defines the type of transaction; a payment is “PY” and a purchase is “PR.” If the value for TRX is PY, you want the current value for AMT subtracted from the running total; if the value for TRX is PR, you want the current value for AMT added to the running total. Ultimately you want to return the following result set:

```
TRX_TYPE        AMT    BALANCE
-------- ---------- ----------
PURCHASE        100        100
PURCHASE        100        200
PAYMENT          50        150
PURCHASE        100        250
PAYMENT         200         50
PAYMENT          50          0
```

## Solution

[]()[]()Use the window function SUM OVER to create the running total along with a CASE expression to determine the type of transaction:

```
 1 select case when trx = 'PY'
 2             then 'PAYMENT'
 3             else 'PURCHASE'
 4         end trx_type,
 5         amt,
 6         sum(
 7          case when trx = 'PY'
 8             then -amt else amt
 9          end
10        ) over (order by id,amt) as balance
11 from V
```

## Discussion

The CASE expression determines whether the current AMT is added or deducted from the running total. If the transaction is a payment, the AMT is changed to a negative value, thus reducing the amount of the running total. The result of the CASE expression is shown here:

```
select case when trx = 'PY' 
             then 'PAYMENT' 
             else 'PURCHASE' 
        end trx_type, 
        case when trx = 'PY' 
             then -amt else amt 
        end as amt 
   from V 

TRX_TYPE       AMT
-------- ---------
PURCHASE       100
PURCHASE       100
PAYMENT        -50
PURCHASE       100
PAYMENT       -200
PAYMENT        -50
```

After evaluating the transaction type, the values for AMT are then added to or subtracted from the running total. For an explanation on how the window function, SUM OVER, or the scalar subquery creates the running total, see recipe [Recipe 7.6](#sqlckbk-CHP-7-SECT-6).[]()[]()

# 7.16 Finding Outliers Using the Median Absolute Deviation

## Problem

[]()[]()[]()You want to identify values in your data that may be suspect. There are various reasons why values could be suspect—there could be a data collection issue, such as an error with the meter that records the value. There could be a data entry error such as a typo or similar. There could also be unusual circumstances when the data was generated that mean the data point is correct, but they still require you to use caution in any conclusion you make from the data. Therefore, you want to detect outliers.

A common way to detect outliers, taught in many statistics courses aimed at non-statisticians, is to calculate the standard deviation of the data and decide that data points more than three standard deviations (or some other similar distance) are outliers. However, this method can misidentify outliers if the data don’t follow a normal distribution, especially if the spread of data isn’t symmetrical or doesn’t thin out in the same way as a normal distribution as you move further from the mean.

## Solution

First find the median of the values using the recipe for finding the median from earlier in this chapter. You will need to put this query into a CTE to make it available for further querying. The deviation is the absolute difference between the median and each value; the median absolute deviation is the median of this value, so we need to calculate the median again.

### SQL Server

[]()SQL Server has the PERCENTILE\_CONT function, which simplifies finding the median. As we need to find two different medians and manipulate them, we need a series of CTEs:

```
with median (median)
as
(select distinct percentile_cont(0.5) within group(order by sal)
        over()
from emp),

Deviation (Deviation)
  as
(Select abs(sal-median)
from emp join median on 1=1),

MAD (MAD) as
(select DISTINCT PERCENTILE_CONT(0.5) within group(order by deviation) over()
from Deviation )

select abs(sal-median)/MAD, sal, ename, job
from MAD join emp on 1=1
```

### PostgreSQL and DB2

The overall pattern is the same, but there is different syntax for PERCENTILE\_CONT, as PostgreSQL and DB2 treat PERCENTILE\_CONT as an aggregate function rather than strictly a window function:

```
with median (median)
as
(select percentile_cont(0.5) within group(order by sal)
from emp),

devtab (deviation)
  as
(select abs(sal-median)
from emp join median),

MedAbsDeviation (MAD) as
(select percentile_cont (0.5) within group(order by deviation)
from devtab)

select abs(sal-median)/MAD, sal, ename, job
FROM MedAbsDeviation join emp
```

### Oracle

The recipe is simplified for Oracle users due to the existence of a median function. However, we still need to use a CTE to handle the scalar value of deviation:

```
with
Deviation (Deviation)
  as
(select abs(sal-median(sal))
from emp),

MAD (MAD) as
(select median(Deviation)
from Deviation )

select abs(sal-median)/MAD, sal, ename, job
FROM MAD join emp
```

### MySQL

As we saw in the earlier section on the median, there is unfortunately no MEDIAN or PERCENTILE\_CONT function in MySQL. This means that each of the medians we need to find to compute the median absolute deviation is two subqueries within a CTE. This makes the MySQL a little long-winded:

```
with rank_tab (sal, rank_sal) as (
select sal, cume_dist() over (order by sal)
from emp),
inter as
(
select sal, rank_sal from rank_tab
where rank_sal>=0.5
union
select sal, rank_sal from rank_tab
where rank_sal<=0.5
)
,

medianSal (medianSal) as

(
select (max(sal)+min(sal))/2
from inter),
deviationSal (Sal,deviationSal) as
(select Sal,abs(sal-medianSal)
from emp join medianSal
on 1=1
)
,

distDevSal (sal,deviationSal,distDeviationSal) as

(
select sal,deviationSal,cume_dist() over (order by deviationSal)
from deviationSal
),

DevInter (DevInter, sal) as
(
select min(deviationSal), sal
from distDevSal
where distDeviationSal >= 0.5

union

select max(DeviationSal), sal
from distDevSal
where distDeviationSal <= 0.5
),

MAD (MedianAbsoluteDeviance) as
(
select abs(emp.sal-(min(devInter)+max(devInter))/2)
from emp join DevInter on 1=1
)

select emp.sal,MedianAbsoluteDeviance,
(emp.sal-deviationSal)/MedianAbsoluteDeviance
from (emp join MAD on 1=1)
         join deviationSal on emp.sal=deviationSal.sal
```

## Discussion

In each case the recipe follows a similar strategy. First we need to calculate the median, and then we need to calculate the median of the difference between each value and the median, which is the actual median absolute deviation. Finally, we need to use a query to find the ratio of the deviation of each value to the median deviation. At that point, we can use the outcome in a similar way to the standard deviation. For example, if a value is three or more deviations from the median, it can be considered an outlier, to use a common interpretation.

As mentioned earlier, the benefit of this approach over the standard deviation is that the interpretation is still valid even if the data doesn’t display a normal distribution. For example, it can be lopsided, and the median absolute deviation will still give a sound answer.

In our salary data, there is one salary that is more than three absolute deviations from the median: the CEO’s.

Although there are differing opinions about the fairness of CEO salaries versus those of most other workers, given that the outlier salary belongs to the CEO, it fits with our understanding of the data. In other contexts, if there wasn’t a clear explanation of why the value differed so much, it could lead us to question whether that value was correct or whether the value made sense when taken with the rest of the values (e.g., if it not actually an error, it might make us think we need to analyze our data within more than one subgroup).

###### Note

Many of the common statistics, such as the mean and the standard deviation, assume that the shape of the data is a bell curve—a normal distribution. This is true for many data sets, and also not true for many data sets.

There are a number of methods for testing whether a data set follows a normal distribution, both by visualizing the data and through calculations. Statistical packages commonly contain functions for these tests, but they are nonexistent and hard to replicate in SQL. However, there are often alternative statistical tools that don’t assume the data takes a particular form—nonparametric statistics—and these are safer to use.[]()[]()[]()

# 7.17 Finding Anomalies Using Benford’s Law

## Problem

[]()[]()Although outliers, as shown in the previous recipe, are a readily identifiable form of anomalous data, some other data is less easy to identify as problematic. One way to detect situations where there are anomalous data but no obvious outliers is to look at the frequency of digits, which is usually expected to follow Benford’s law. Although using Benford’s law is most often associated with detecting fraud in situations where humans have added fake numbers to a data set, it can be used more generally to detect data that doesn’t follow expected patterns. For example, it can detect errors such as duplicated data points, which won’t necessarily stand out as outliers.

## Solution

To use Benford’s law, you need to calculate the expected distribution of digits and then the actual distribution to compare. Although the most sophisticated uses look at first, second, and combinations of digits, in this example we will stick to just the first digits.

You compare the frequency predicted by Benford’s law with the actual frequency of your data. Ultimately you want four columns—the first digit, the count of how many times each first digit appears, the frequency of first digits predicted by Benford’s law, and the actual frequency:

```
with
FirstDigits (FirstDigit)
as
(select left(cast(SAL as CHAR),1) as FirstDigit
        from emp),

TotalCount (Total)
as
 (select count(*)
  from emp),

ExpectedBenford (Digit,Expected)
as
  (select ID,(log10(ID + 1) - log10(ID)) as expected
   from t10
   where ID < 10)

select count(FirstDigit),Digit
,coalesce(count(*)/Total,0) as ActualProportion,Expected
From FirstDigits
     Join TotalCount
     Right Join ExpectedBenford
     on FirstDigits.FirstDigit=ExpectedBenford.Digit
group by Digit
order by Digit;
```

## Discussion

Because we need to make use of two different counts—one of the total rows, and another of the number of rows containing each different first digit—we need to use a CTE. Strictly speaking, we don’t need to put the expected Benford’s law results into a separate query within the CTE, but we have done so in this case as it allows us to identify the digits with a zero count and display them in the table via the right join.

It’s also possible to produce the FirstDigits count in the main query, but we have chosen not to improve readability through not needing to repeat the LEFT(CAST… expression in the GROUP BY clause.

The math behind Benford’s law is simple:

Expectedfrequency=log10(d+1d)

We can use the T10 pivot table to generate the appropriate values. From there we just need to calculate the actual frequencies for comparison, which first requires us to identify the first digit.

Benford’s law works best when there is a relatively large collection of values to apply it to, and when those values span more than one order of magnitude (10, 100, 1,000, etc.). Those conditions aren’t entirely met here. At the same time, the deviation from expected should still make us suspicious that these values are in some sense made-up values and worth investigating further.[]()[]()

# 7.18 Summing Up

An enterprise’s data is frequently found in a database supported by SQL, so it makes sense to use SQL to try to understand that data. SQL doesn’t have the full array of statistical tools you would expect in a purpose-built package such as SAS, the statistical programming language R, or Python’s statistical libraries. However, it does have a rich set of tools for calculation that as we have seen can provide a deep understanding of the statistical properties of your data.[]()