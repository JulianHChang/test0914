# Chapter 13. Hierarchical Queries

[]()This chapter introduces recipes for expressing hierarchical relationships that you may have in your data. It is typical when working with hierarchical data to have more difficulty retrieving and displaying the data (as a hierarchy) than storing it.

Although it’s only been a couple of years since MySQL added recursive CTEs, now that they are available it means that recursive CTEs are available in virtually every RDBMS. As a result, they are the gold standard for dealing with hierarchical queries, and this chapter will make liberal use of this capability to provide recipes to help you unravel the hierarchical structure of your data.

Before starting, examine table EMP and the hierarchical relationship between EMPNO and MGR:

```
select empno,mgr 
   from emp 
 order by 2 

     EMPNO        MGR
---------- ----------
      7788       7566
      7902       7566
      7499       7698
      7521       7698
      7900       7698
      7844       7698
      7654       7698
      7934       7782
      7876       7788
      7566       7839
      7782       7839
      7698       7839
      7369       7902
      7839
```

If you look carefully, you will see that each value for MGR is also an EMPNO, meaning the manager of each employee in table EMP is also an employee in table EMP and not stored somewhere else. The relationship between MGR and EMPNO is a parent-child relationship in that the value for MGR is the most immediate parent for a given EMPNO (it is also possible that the manager for a specific employee can have a manager as well, and those managers can in turn have managers, and so on, creating an *n*-tier hierarchy). If an employee has no manager, then MGR is NULL.

# 13.1 Expressing a Parent-Child Relationship

## Problem

[]()You want to include parent information along with data from child records. For example, you want to display each employee’s name along with the name of their manager. You want to return the following result set:

```
EMPS_AND_MGRS
------------------------------
FORD works for JONES
SCOTT works for JONES
JAMES works for BLAKE
TURNER works for BLAKE
MARTIN works for BLAKE
WARD works for BLAKE
ALLEN works for BLAKE
MILLER works for CLARK
ADAMS works for SCOTT
CLARK works for KING
BLAKE works for KING
JONES works for KING
SMITH works for FORD
```

## Solution

Self-join EMP on MGR and EMPNO to find the name of each employee’s manager. Then use your RDBMS’s supplied function(s) for string concatenation to generate the strings in the desired result set.

### DB2, Oracle, and PostgreSQL

Self-join on EMP. Then use the double vertical-bar (||) concatenation operator:

```
1 select a.ename || ' works for ' || b.ename as emps_and_mgrs
2   from emp a, emp b
3  where a.mgr = b.empno
```

### MySQL

Self-join on EMP. Then use the concatenation function CONCAT:

```
1 select concat(a.ename, ' works for ',b.ename) as emps_and_mgrs
2   from emp a, emp b
3  where a.mgr = b.empno
```

### SQL Server

Self-join on EMP. Then use the plus sign (+) as the concatenation operator:

```
1 select a.ename + ' works for ' + b.ename as emps_and_mgrs
2   from emp a, emp b
3  where a.mgr = b.empno
```

## Discussion

The implementation is essentially the same for all the solutions. The difference lies only in the method of string concatenation, and thus one discussion will cover all of the solutions.

The key is the join between MGR and EMPNO. The first step is to build a Cartesian product by joining EMP to itself (only a portion of the rows returned by the Cartesian product is shown here):

```
select a.empno, b.empno 
   from emp a, emp b 

EMPNO        MGR
----- ----------
 7369       7369
 7369       7499
 7369       7521
 7369       7566
 7369       7654
 7369       7698
 7369       7782
 7369       7788
 7369       7839
 7369       7844
 7369       7876
 7369       7900
 7369       7902
 7369       7934
 7499       7369
 7499       7499
 7499       7521
 7499       7566
 7499       7654
 7499       7698
 7499       7782
 7499       7788
 7499       7839
 7499       7844
 7499       7876
 7499       7900
 7499       7902
 7499       7934
```

As you can see, by using a Cartesian product you are returning every possible EMPNO/EMPNO combination (such that it looks like the manager for EMPNO 7369 is all the other employees in the table, including EMPNO 7369).

The next step is to filter the results such that you return only each employee and their manager’s EMPNO. Accomplish this by joining on MGR and EMPNO:

```
1 select a.empno, b.empno mgr 
 2   from emp a, emp b 
 3  where a.mgr = b.empno 

     EMPNO        MGR
---------- ----------
      7902       7566
      7788       7566
      7900       7698
      7844       7698
      7654       7698
      7521       7698
      7499       7698
      7934       7782
      7876       7788
      7782       7839
      7698       7839
      7566       7839
      7369       7902
```

Now that you have each employee and the EMPNO of their manager, you can return the name of each manager by simply selecting B.ENAME rather than B.EMPNO. If after some practice you have difficulty grasping how this works, you can use a scalar subquery rather than a self-join to get the answer:

```
select a.ename, 
        (select b.ename 
           from emp b 
          where b.empno = a.mgr) as mgr 
   from emp a 

ENAME      MGR
---------- ----------
SMITH      FORD
ALLEN      BLAKE
WARD       BLAKE
JONES      KING
MARTIN     BLAKE
BLAKE      KING
CLARK      KING
SCOTT      JONES
KING
TURNER     BLAKE
ADAMS      SCOTT
JAMES      BLAKE
FORD       JONES
MILLER     CLARK
```

The scalar subquery version is equivalent to the self-join, except for one row: employee KING is in the result set, but that is not the case with the self-join. “Why not?” you might ask. []()Remember, NULL is never equal to anything, not even itself. In the self-join solution, you use an equi-join between EMPNO and MGR, thus filtering out any employees who have NULL for MGR. To see employee KING when using the self-join method, you must outer join as shown in the following two queries. The first solution uses the ANSI outer join, while the second uses the Oracle outer-join syntax. The output is the same for both and is shown following the second query:[]()

```
/* ANSI */ 
 select a.ename, b.ename mgr 
   from emp a left join emp b 
     on (a.mgr = b.empno) 

 /* Oracle */ 
 select a.ename, b.ename mgr 
   from emp a, emp b 
  where a.mgr = b.empno (+) 

ENAME      MGR
---------- ----------
FORD       JONES
SCOTT      JONES
JAMES      BLAKE
TURNER     BLAKE
MARTIN     BLAKE
WARD       BLAKE
ALLEN      BLAKE
MILLER     CLARK
ADAMS      SCOTT
CLARK      KING
BLAKE      KING
JONES      KING
SMITH      FORD
KING
```

# 13.2 Expressing a Child-Parent-Grandparent Relationship

## Problem

[]()Employee CLARK works for KING, and to express that relationship you can use the first recipe in this chapter. What if employee CLARK was in turn a manager for another employee? Consider the following query:

```
select ename,empno,mgr 
   from emp 
  where ename in ('KING','CLARK','MILLER') 

ENAME        EMPNO     MGR
--------- -------- -------
CLARK         7782    7839
KING          7839
MILLER        7934    7782
```

As you can see, employee MILLER works for CLARK who in turn works for KING. You want to express the full hierarchy from MILLER to KING. You want to return the following result set:

```
LEAF___BRANCH___ROOT
---------------------
MILLER-->CLARK-->KING
```

However, the single self-join approach from the previous recipe will not suffice to show the entire relationship from top to bottom. You could write a query that does two self-joins, but what you really need is a general approach for traversing such hierarchies.

## Solution

This recipe differs from the first recipe because there is now a three-tier relationship, as the title suggests. If your RDBMS does not supply functionality for traversing tree-structured data, as is the case for Oracle, then you can solve this problem using the CTEs.

### DB2 and SQL Server

[]()Use the recursive WITH clause to find MILLER’s manager, CLARK, and then CLARK’s manager, KING. The SQL Server string concatenation operator + is used in this solution:

```
1    with  x (tree,mgr,depth)
2      as  (
3  select  cast(ename as varchar(100)),
4          mgr, 0
5    from  emp
6   where  ename = 'MILLER'
7   union  all
8  select  cast(x.tree+'-->'+e.ename as varchar(100)),
9          e.mgr, x.depth+1
10   from  emp e, x
11  where x.mgr = e.empno
12 )
13 select tree leaf___branch___root
14   from x
15  where depth = 2
```

This solution can work on other databases if the concatenation operator is changed. Hence, change to || for DB2 or CONCAT for PostgreSQL.

### MySQL and PostgreSQL

This is similar to the previous solution, but also needs the RECURSIVE keyword:

```
1    with recursive x (tree,mgr,depth)
2      as  (
3  select  cast(ename as varchar(100)),
4          mgr, 0
5    from  emp
6   where  ename = 'MILLER'
7   union  all
8  select  cast(concat(x.tree,'-->',emp.ename) as char(100)),
9          e.mgr, x.depth+1
10   from  emp e, x
11  where x.mgr = e.empno
12 )
13 select tree leaf___branch___root
14   from x
15  where depth = 2
```

### Oracle

[]()Use the function SYS\_CONNECT\_BY\_PATH to return MILLER; MILLER’s manager, CLARK; and then CLARK’s manager, KING. Use the CONNECT BY clause to walk the tree:

```
1  select ltrim(
2           sys_connect_by_path(ename,'-->'),
3         '-->') leaf___branch___root
4    from emp
5   where level = 3
6   start with ename = 'MILLER'
7 connect by prior mgr = empno
```

## Discussion

### DB2, SQL Server, PostgreSQL, and MySQL

The approach here is to start at the leaf node and walk your way up to the root (as useful practice, try walking in the other direction). The upper part of the UNION ALL simply finds the row for employee MILLER (the leaf node). The lower part of the UNION ALL finds the employee who is MILLER’s manager and then finds that person’s manager, and this process of finding the “manager’s manager” repeats until processing stops at the highest-level manager (the root node). The value for DEPTH starts at 0 and increments automatically by 1 each time a manager is found. DEPTH is a value that DB2 maintains for you when you execute a recursive query.

###### Tip

For an interesting and in-depth introduction to the WITH clause with a focus on its use recursively, see Jonathan Gennick’s article [“Understanding the WITH Clause”](http://gennick.com/with.htm).

Next, the second query of the UNION ALL joins the recursive view X to table EMP, to define the parent-child relationship. The query at this point, using SQL Server’s concatenation operator, is as follows:

```
  with x (tree,mgr,depth) 
     as ( 
 select cast(ename as varchar(100)), 
        mgr, 0 
   from emp 
  where ename = 'MILLER' 
  union all 
 select cast(x.tree+'-->'+e.ename as varchar(100)), 
        e.mgr, x.depth+1 
   from emp e, x 
  where x.mgr = e.empno 
 ) 
 select tree leaf___branch___root 
   from x 

TREE            DEPTH
---------- ----------
MILLER              0
CLARK               1
KING                2
```

At this point, the heart of the problem has been solved; starting from MILLER, return the full hierarchical relationship from bottom to top. What’s left then is merely formatting. Since the tree traversal is recursive, simply concatenate the current ENAME from EMP to the one before it, which gives you the following result set:

```
  with x (tree,mgr,depth) 
     as ( 
 select  cast(ename as varchar(100)), 
         mgr, 0 
   from emp 
  where ename = 'MILLER' 
  union all 
 select cast(x.tree+'-->'+e.ename as varchar(100)), 
        e.mgr, x.depth+1 
   from emp e, x 
  where x.mgr = e.empno 
 ) 
 select depth, tree 
   from x 

DEPTH TREE
----- ---------------------------
    0 MILLER
    1 MILLER-->CLARK
    2 MILLER-->CLARK-->KING
```

The final step is to keep only the last row in the hierarchy. There are several ways to do this, but the solution uses DEPTH to determine when the root is reached (obviously, if CLARK has a manager other than KING, the filter on DEPTH would have to change; for a more generic solution that requires no such filter, see the next recipe).

### Oracle

The CONNECT BY clause does all the work in the Oracle solution. Starting with MILLER, you walk all the way to KING without the need for any joins. The expression in the CONNECT BY clause defines the relationship of the data and how the tree will be walked:

```
 select ename 
    from emp 
   start with ename = 'MILLER' 
 connect by prior mgr = empno 

ENAME
--------
MILLER
CLARK
KING
```

[]()The keyword PRIOR lets you access values from the previous record in the hierarchy. Thus, for any given EMPNO, you can use PRIOR MGR to access that employee’s manager number. When you see a clause such as CONNECT BY PRIOR MGR = EMPNO, think of that clause as expressing a join between, in this case, parent and child.

###### Tip

For more on CONNECT BY and its use in hierarchical queries, [“Hierarchical Queries in Oracle”](https://oreil.ly/6yfha) is a good overview.

At this point, you have successfully displayed the full hierarchy starting from MILLER and ending at KING. The problem is for the most part solved. All that remains is the formatting. []()Use the function SYS\_CONNECT\_BY\_PATH to append each ENAME to the one before it:

```
 select sys_connect_by_path(ename,'-->') tree 
    from emp 
   start with ename = 'MILLER' 
 connect by prior mgr = empno 

TREE
---------------------------
-->MILLER
-->MILLER-->CLARK
-->MILLER-->CLARK-->KING
```

Because you are interested in only the complete hierarchy, you can filter on the pseudo-column LEVEL (a more generic approach is shown in the next recipe):

```
 select sys_connect_by_path(ename,'-->') tree 
    from emp 
   where level = 3 
   start with ename = 'MILLER' 
 connect by prior mgr = empno 

TREE
---------------------------
-->MILLER-->CLARK-->KING
```

[]()The final step is to use the LTRIM function to remove the leading --&gt; from the result set.[]()

# 13.3 Creating a Hierarchical View of a Table

## Problem

[]()You want to return a result set that describes the hierarchy of an entire table. In the case of the EMP table, employee KING has no manager, so KING is the root node. You want to display, starting from KING, all employees under KING and all employees (if any) under KING’s subordinates. Ultimately, you want to return the following result set:

```
EMP_TREE
------------------------------
KING
KING - BLAKE
KING - BLAKE - ALLEN
KING - BLAKE - JAMES
KING - BLAKE - MARTIN
KING - BLAKE - TURNER
KING - BLAKE - WARD
KING - CLARK
KING - CLARK - MILLER
KING - JONES
KING - JONES - FORD
KING - JONES - FORD - SMITH
KING - JONES - SCOTT
KING - JONES - SCOTT - ADAMS
```

## Solution

### DB2, PostgreSQL, and SQL Server

[]()Use the recursive WITH clause to start building the hierarchy at KING and then ultimately display all the employees. The solution following uses the DB2 concatenation operator (||). SQL Server users use the concatenation operator (+), and MySQL uses the CONCAT function. Other than the concatenation operators, the solution will work as-is on both RDBMSs:

```
 1   with x (ename,empno)
 2      as (
 3  select cast(ename as varchar(100)),empno
 4    from emp
 5   where mgr is null
 6   union all
 7  select cast(x.ename||' - '||e.ename as varchar(100)),
 8         e.empno
 9    from emp e, x
10   where e.mgr = x.empno
11  )
12  select ename as emp_tree
13    from x
14   order by 1
```

### MySQL

MySQL also needs the RECURSIVE keyword:

```
 1   with recursive x (ename,empno)
 2      as (
 3  select cast(ename as varchar(100)),empno
 4    from emp
 5   where mgr is null
 6   union all
 7  select cast(concat(x.ename,' - ',e.ename) as varchar(100)),
 8         e.empno
 9    from emp e, x
10   where e.mgr = x.empno
11  )
12  select ename as emp_tree
13    from x
14   order by 1
```

### Oracle

[]()Use the CONNECT BY function to define the hierarchy. Use the SYS\_CONNECT\_BY\_PATH function to format the output accordingly:

```
1  select ltrim(
2           sys_connect_by_path(ename,' - '),
3         ' - ') emp_tree
4    from emp
5    start with mgr is null
6  connect by prior empno=mgr
7    order by 1
```

This solution differs from the previous recipe in that it includes no filter on the LEVEL pseudo-column. Without the filter, all possible trees (where PRIOR EMPNO=MGR) are displayed.

## Discussion

### DB2, MySQL, PostgreSQL, and SQL Server

The first step is to identify the root row (employee KING) in the upper part of the UNION ALL in the recursive view X. The next step is to find KING’s subordinates, and their subordinates if there are any, by joining recursive view X to table EMP. Recursion will continue until you’ve returned all employees. Without the formatting you see in the final result set, the result set returned by the recursive view X is shown here:

```
with x (ename,empno) 
     as ( 
 select cast(ename as varchar(100)),empno 
   from emp 
  where mgr is null 
  union all 
 select cast(e.ename as varchar(100)),e.empno 
   from emp e, x 
  where e.mgr = x.empno 
  ) 
  select ename emp_tree 
    from x 

 EMP_TREE
 ----------------
 KING
 JONES
 SCOTT
 ADAMS
 FORD
 SMITH
 BLAKE
 ALLEN
 WARD
 MARTIN
 TURNER
 JAMES
 CLARK
 MILLER
```

All the rows in the hierarchy are returned (which can be useful), but without the formatting you cannot tell who the managers are. By concatenating each employee to her manager, you return more meaningful output. Produce the desired output simply by using the following:

```
cast(x.ename+','+e.ename as varchar(100))
```

in the SELECT clause of the lower portion of the UNION ALL in recursive view X.

The WITH clause is extremely useful in solving this type of problem, because the hierarchy can change (for example, leaf nodes become branch nodes) without any need to modify the query.

### Oracle

The CONNECT BY clause returns the rows in the hierarchy. []()The START WITH clause defines the root row. If you run the solution without SYS\_CONNECT\_BY\_PATH, you can see that the correct rows are returned (which can be useful), but not formatted to express the relationship of the rows:

```
select ename emp_tree 
   from emp 
  start with mgr is null 
 connect by prior empno = mgr 

EMP_TREE
-----------------
KING
JONES
SCOTT
ADAMS
FORD
SMITH
BLAKE
ALLEN
WARD
MARTIN
TURNER
JAMES
CLARK
MILLER
```

By using the pseudo-column LEVEL and the function LPAD, you can see the hierarchy more clearly, and you can ultimately see why SYS\_CONNECT\_BY\_PATH returns the results that you see in the desired output shown earlier:

```
select lpad('.',2*level,'.')||ename emp_tree
   from emp
  start with mgr is null
connect by prior empno = mgr

EMP_TREE
-----------------
..KING
....JONES
......SCOTT
........ADAMS
......FORD
........SMITH
....BLAKE
......ALLEN
......WARD
......MARTIN
......TURNER
......JAMES
....CLARK
......MILLER
```

The indentation in this output indicates who the managers are by nesting subordinates under their superiors. For example, KING works for no one. JONES works for KING. SCOTT works for JONES. ADAMS works for SCOTT.

If you look at the corresponding rows from the solution when using SYS\_CONNECT\_BY\_PATH, you will see that SYS\_CONNECT\_BY\_PATH rolls up the hierarchy for you. When you get to a new node, you see all the prior nodes as well:[]()

```
KING
KING - JONES
KING - JONES - SCOTT
KING - JONES - SCOTT - ADAMS
```

# 13.4 Finding All Child Rows for a Given Parent Row

## Problem

[]()You want to find all the employees who work for JONES, either directly or indirectly (i.e., they work for someone who works for JONES). The list of employees under JONES is shown here (JONES is included in the result set):

```
ENAME
----------
JONES
SCOTT
ADAMS
FORD
SMITH
```

## Solution

Being able to move to the absolute top or bottom of a tree is extremely useful. For this solution, there is no special formatting necessary. The goal is to simply return all employees who work under employee JONES, including JONES himself. This type of query really shows the usefulness of recursive SQL extensions like Oracle’s CONNECT BY and SQL Server’s/DB2’s WITH clause.

### DB2, PostgreSQL, and SQL Server

Use the recursive WITH clause to find all employees under JONES. Begin with JONES by specifying WHERE ENAME = JONES in the first of the two union queries:

```
 1   with x (ename,empno)
 2     as (
 3 select ename,empno
 4   from emp
 5  where ename = 'JONES'
 6  union all
 7 select e.ename, e.empno
 8   from emp e, x
 9  where x.empno = e.mgr
10 )
11 select ename
12   from x
```

### Oracle

[]()Use the CONNECT BY clause and specify START WITH ENAME = *JONES* to find all the employees under JONES:

```
1 select ename
2   from emp
3  start with ename = 'JONES'
4 connect by prior empno = mgr
```

## Discussion

### DB2, MySQL, PostgreSQL, and SQL Server

[]()The recursive WITH clause makes this a relatively easy problem to solve. The first part of the WITH clause, the upper part of the UNION ALL, returns the row for employee JONES. You need to return ENAME to see the name and EMPNO so you can use it to join on. The lower part of the UNION ALL recursively joins EMP.MGR to X.EMPNO. The join condition will be applied until the result set is exhausted.

### Oracle

[]()The START WTH clause tells the query to make JONES the root node. The condition in the CONNECT BY clause drives the tree walk and will run until the condition is no longer true.

# 13.5 Determining Which Rows Are Leaf, Branch, or Root Nodes

## Problem

[]()You want to determine what type of node a given row is: a leaf, branch, or root. For this example, a leaf node is an employee who is not a manager. A branch node is an employee who is both a manager and also has a manager. A root node is an employee without a manager. You want to return 1 (TRUE) or 0 (FALSE) to reflect the status of each row in the hierarchy. You want to return the following result set:

```
ENAME          IS_LEAF   IS_BRANCH     IS_ROOT
----------  ----------  ----------  ----------
KING                 0           0           1
JONES                0           1           0
SCOTT                0           1           0
FORD                 0           1           0
CLARK                0           1           0
BLAKE                0           1           0
ADAMS                1           0           0
MILLER               1           0           0
JAMES                1           0           0
TURNER               1           0           0
ALLEN                1           0           0
WARD                 1           0           0
MARTIN               1           0           0
SMITH                1           0           0
```

## Solution

It is important to realize that the EMP table is modeled in a tree hierarchy, not a recursive hierarchy, and the value for MGR for root nodes is NULL. If EMP were modeled to use a recursive hierarchy, root nodes would be self-referencing (i.e., the value for MGR for employee KING would be KING’s EMPNO). We find self-referencing to be counterintuitive and thus are using NULL values for root nodes’ MGR. For Oracle users using CONNECT BY and DB2/SQL Server users using WITH, you’ll find tree hierarchies easier to work with and potentially more efficient than recursive hierarchies. If you are in a situation where you have a recursive hierarchy and are using CONNECT BY or WITH, watch out: you can end up with a loop in your SQL. You need to code around such loops if you are stuck with recursive hierarchies.

### DB2, PostgreSQL, MySQL, and SQL Server

Use three scalar subqueries to determine the correct “Boolean” value (either a 1 or a 0) to return for each node type:

```
 1 select e.ename,
 2        (select sign(count(*)) from emp d
 3          where 0 =
 4            (select count(*) from emp f
 5              where f.mgr = e.empno)) as is_leaf,
 6        (select sign(count(*)) from emp d
 7          where d.mgr = e.empno
 8           and e.mgr is not null) as is_branch,
 9        (select sign(count(*)) from emp d
10          where d.empno = e.empno
11            and d.mgr is null) as is_root
12   from emp e
13 order by 4 desc,3 desc
```

### Oracle

The scalar subquery solution will work for Oracle as well and should be used if you are on a version of Oracle prior to Oracle Database 10*g*. The following solution highlights built-in functions provided by Oracle (that were introduced in Oracle Database 10*g*) to identify root and leaf rows. []()[]()The functions are CONNECT\_BY\_ROOT and CONNECT\_BY\_ISLEAF, respectively:

```
 1  select ename,
 2         connect_by_isleaf is_leaf,
 3         (select count(*) from emp e
 4           where e.mgr = emp.empno
 5             and emp.mgr is not null
 6             and rownum = 1) is_branch,
 7         decode(ename,connect_by_root(ename),1,0) is_root
 8    from emp
 9   start with mgr is null
10 connect by prior empno = mgr
11 order by 4 desc, 3 desc
```

## Discussion

### DB2, PostgreSQL, MySQL, and SQL Server

This solution simply applies the rules defined in the “Problem” section to determine leaves, branches, and roots. The first step is to determine whether an employee is a leaf node. If the employee is not a manager (no one works under them), then she is a leaf node. The first scalar subquery, IS\_LEAF, is shown here:

```
select e.ename, 
        (select sign(count(*)) from emp d 
          where 0 = 
            (select count(*) from emp f 
              where f.mgr = e.empno)) as is_leaf 
   from emp e 
 order by 2 desc 

ENAME        IS_LEAF
----------- --------
SMITH              1
ALLEN              1
WARD               1
ADAMS              1
TURNER             1
MARTIN             1
JAMES              1
MILLER             1
JONES              0
BLAKE              0
CLARK              0
FORD               0
SCOTT              0
KING               0
```

Because the output for IS\_LEAF should be a 0 or 1, it is necessary to take the SIGN of the COUNT(\*) operation. Otherwise, you would get 14 instead of 1 for leaf rows. As an alternative, you can use a table with only one row to count against, because you only want to return 0 or 1. For example:

```
select e.ename, 
        (select count(*) from t1 d 
          where not exists 
            (select null from emp f 
              where f.mgr = e.empno)) as is_leaf 
   from emp e 
 order by 2 desc 

ENAME         IS_LEAF
---------- ----------
SMITH               1
ALLEN               1
WARD                1
ADAMS               1
TURNER              1
MARTIN              1
JAMES               1
MILLER              1
JONES               0
BLAKE               0
CLARK               0
FORD                0
SCOTT               0
KING                0
```

The next step is to find branch nodes. If an employee is a manager (someone works for them) and they also happen to work for someone else, then the employee is a branch node. The results of the scalar subquery IS\_BRANCH are shown here:

```
select e.ename, 
        (select sign(count(*)) from emp d 
          where d.mgr = e.empno 
           and e.mgr is not null) as is_branch 
   from emp e 
 order by 2 desc 


ENAME       IS_BRANCH
----------- ---------
JONES               1
BLAKE               1
SCOTT               1
CLARK               1
FORD                1
SMITH               0
TURNER              0
MILLER              0
JAMES               0
ADAMS               0
KING                0
ALLEN               0
MARTIN              0
WARD                0
```

Again, it is necessary to take the SIGN of the COUNT(\*) operation. Otherwise, you will get (potentially) values greater than 1 when a node is a branch. Like scalar subquery IS\_LEAF, you can use a table with one row to avoid using SIGN. The following solution uses the T1 table:

```
select e.ename, 
       (select count(*) from t1 t 
         where exists ( 
          select null from emp f 
           where f.mgr = e.empno 
             and e.mgr is not null)) as is_branch 
   from emp e 
 order by 2 desc 


ENAME            IS_BRANCH
--------------- ----------
JONES                    1
BLAKE                    1
SCOTT                    1
CLARK                    1
FORD                     1
SMITH                    0
TURNER                   0
MILLER                   0
JAMES                    0
ADAMS                    0
KING                     0
ALLEN                    0
MARTIN                   0
WARD                     0
```

The last step is to find the root nodes. A root node is defined as an employee who is a manager but who does not work for anyone else. In table EMP, only KING is a root node. Scalar subquery IS\_ROOT is shown here:

```
select e.ename, 
        (select sign(count(*)) from emp d 
          where d.empno = e.empno 
            and d.mgr is null) as is_root 
   from emp e 
 order by 2 desc 


ENAME         IS_ROOT
----------  ---------
KING                1
SMITH               0
ALLEN               0
WARD                0
JONES               0
TURNER              0
JAMES               0
MILLER              0
FORD                0
ADAMS               0
MARTIN              0
BLAKE               0
CLARK               0
SCOTT               0
```

Because EMP is a small 14-row table, it is easy to see that employee KING is the only root node, so in this case taking the SIGN of the COUNT(\*) operation is not strictly necessary. If there can be multiple root nodes, then you can use SIGN, or you can use a one-row table in the scalar subquery as is shown earlier for IS\_BRANCH and IS\_LEAF.

### Oracle

For those of you on versions of Oracle prior to Oracle Database 10*g*, you can follow the discussion for the other RDBMSs, as that solution will work (without modifications) in Oracle. []()[]()If you are on Oracle Database 10*g* or later, you may want to take advantage of two functions to make identifying root and leaf nodes a simple task: they are CONNECT\_BY\_ROOT and CONNECT\_BY\_ISLEAF, respectively. As of the time of this writing, it is necessary to use CONNECT BY in your SQL statement in order for you to be able to use CONNECT\_BY\_ROOT and CONNECT\_BY\_ISLEAF. The first step is to find the leaf nodes by using CONNECT\_BY\_ISLEAF as follows:

```
select ename, 
         connect_by_isleaf is_leaf 
   from emp 
  start with mgr is null 
 connect by prior empno = mgr 
 order by 2 desc 


ENAME          IS_LEAF
----------  ----------
ADAMS                1
SMITH                1
ALLEN                1
TURNER               1
MARTIN               1
WARD                 1
JAMES                1
MILLER               1
KING                 0
JONES                0
BLAKE                0
CLARK                0
FORD                 0
SCOTT                0
```

The next step is to use a scalar subquery to find the branch nodes. Branch nodes are employees who are managers but who also work for someone else:

```
select ename, 
         (select count(*) from emp e 
           where e.mgr = emp.empno 
             and emp.mgr is not null 
             and rownum = 1) is_branch 
   from emp 
  start with mgr is null 
 connect by prior empno = mgr 
 order by 2 desc 

ENAME       IS_BRANCH
---------- ----------
JONES               1
SCOTT               1
BLAKE               1
FORD                1
CLARK               1
KING                0
MARTIN              0
MILLER              0
JAMES               0
TURNER              0
WARD                0
ADAMS               0
ALLEN               0
SMITH               0
```

The filter on ROWNUM is necessary to ensure that you return a count of 1 or 0, and nothing else.

The last step is to identify the root nodes by using the function CONNECT\_BY\_ROOT. The solution finds the ENAME for the root node and compares it with all the rows returned by the query. If there is a match, that row is the root node:

```
select ename, 
         decode(ename,connect_by_root(ename),1,0) is_root 
   from emp 
  start with mgr is null 
 connect by prior empno = mgr 
 order by 2 desc 

ENAME          IS_ROOT
----------  ----------
KING                 1
JONES                0
SCOTT                0
ADAMS                0
FORD                 0
SMITH                0
BLAKE                0
ALLEN                0
WARD                 0
MARTIN               0
TURNER               0
JAMES                0
CLARK                0
MILLER               0
```

[]()The SYS\_CONNECT\_BY\_PATH function rolls up a hierarchy starting from the root value, as shown here:

```
select ename, 
        ltrim(sys_connect_by_path(ename,','),',') path 
   from emp 
 start with mgr is null 
 connect by prior empno=mgr 

ENAME      PATH
---------- ----------------------------
KING       KING
JONES      KING,JONES
SCOTT      KING,JONES,SCOTT
ADAMS      KING,JONES,SCOTT,ADAMS
FORD       KING,JONES,FORD
SMITH      KING,JONES,FORD,SMITH
BLAKE      KING,BLAKE
ALLEN      KING,BLAKE,ALLEN
WARD       KING,BLAKE,WARD
MARTIN     KING,BLAKE,MARTIN
TURNER     KING,BLAKE,TURNER
JAMES      KING,BLAKE,JAMES
CLARK      KING,CLARK
MILLER     KING,CLARK,MILLER
```

To get the root row, simply substring out the first ENAME in PATH:

```
select ename, 
        substr(root,1,instr(root,',')-1) root 
   from ( 
 select ename, 
        ltrim(sys_connect_by_path(ename,','),',') root 
   from emp 
 start with mgr is null 
 connect by prior empno=mgr 
        ) 

ENAME      ROOT
---------- ----------
KING
JONES      KING
SCOTT      KING
ADAMS      KING
FORD       KING
SMITH      KING
BLAKE      KING
ALLEN      KING
WARD       KING
MARTIN     KING
TURNER     KING
JAMES      KING
CLARK      KING
MILLER     KING
```

The last step is to flag the result from the ROOT column; if it is NULL, that is your root row.[]()

# 13.6 Summing Up

The spread of CTEs across all vendors has made standardized approaches to hierarchical queries far more achievable. This a great step forward as hierarchical relationships appear in many kinds of data, even data where the relationship isn’t necessarily planned for, so queries need to account for it.[]()