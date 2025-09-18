# Chapter 5. Metadata Queries

[]()This chapter presents recipes that allow you to find information about a given schema. For example, you may want to know what tables you’ve created or which foreign keys are not indexed. All of the RDBMSs in this book provide tables and views for obtaining such data. The recipes in this chapter will get you started on gleaning information from those tables and views.

Although at a high level the strategy of storing metadata in tables and views within the RDBMS is common, the ultimate implementation is not standardized to the same degree as most of the SQL language features covered in this book. Therefore, compared to other chapters, in this chapter having a different solution for each RDBMS is far more common.

The following is selection of the most common schema queries written for each of the RDMSs covered in the book. There is far more information available than the recipes in this chapter can show. Consult your RDBMS’s documentation for the complete list of catalog or data dictionary tables/views when you need to go beyond what’s presented here.

###### Tip

For the purposes of demonstration, all of the recipes in this chapter assume there is a schema named SMEAGOL.

# 5.1 Listing Tables in a Schema

## Problem

[]()[]()You want to see a list of all the tables you’ve created in a given schema.

## Solution

The solutions that follow all assume you are working with the SMEAGOL schema. The basic approach to a solution is the same for all RDBMSs: you query a system table (or view) containing a row for each table in the database.

### DB2

Query SYSCAT.TABLES:

```
1 select tabname
2   from syscat.tables
3  where tabschema = 'SMEAGOL'
```

### Oracle

Query SYS.ALL\_TABLES:

```
select table_name
  from all_tables
 where owner = 'SMEAGOL'
```

### PostgreSQL, MySQL, and SQL Server

Query INFORMATION\_SCHEMA.TABLES:

```
1 select table_name
2   from information_schema.tables
3  where table_schema = 'SMEAGOL'
```

## Discussion

In a delightfully circular manner, databases expose information about themselves through the very mechanisms that you create for your own applications: tables and views. Oracle, for example, maintains an extensive catalog of system views, such as ALL\_TABLES, that you can query for information about tables, indexes, grants, and any other database object.

###### Tip

Oracle’s catalog views are just that, views. They are based on an underlying set of tables that contain the information in a user-unfriendly form. The views put a usable face on Oracle’s catalog data.

Oracle’s system views and DB2’s system tables are each vendor-specific. PostgreSQL, MySQL, and SQL Server, on the other hand, support something called the *information* schema, which is a set of views defined by the ISO SQL standard. That’s why the same query can work for all three of those databases.

# 5.2 Listing a Table’s Columns

## Problem

[]()You want to list the columns in a table, along with their data types, and their position in the table they are in.

## Solution

The following solutions assume that you want to list columns, their data types, and their numeric position in the table named EMP in the schema SMEAGOL.

### DB2

Query SYSCAT.COLUMNS:

```
1 select colname, typename, colno
2   from syscat.columns
3  where tabname   = 'EMP'
4    and tabschema = 'SMEAGOL'
```

### Oracle

Query ALL\_TAB\_COLUMNS:

```
1 select column_name, data_type, column_id
2   from all_tab_columns
3  where owner      = 'SMEAGOL'
4    and table_name = 'EMP'
```

### PostgreSQL, MySQL, and SQL Server

Query INFORMATION\_SCHEMA.COLUMNS:

```
1 select column_name, data_type, ordinal_position
2   from information_schema.columns
3  where table_schema = 'SMEAGOL'
4    and table_name   = 'EMP'
```

## Discussion

Each vendor provides ways for you to get detailed information about your column data. In the previous examples, only the column name, data type, and position are returned. Additional useful items of information include length, nullability, and default values.

# 5.3 Listing Indexed Columns for a Table

## Problem

[]()[]()You want list indexes, their columns, and the column position (if available) in the index for a given table.

## Solution

The vendor-specific solutions that follow all assume that you are listing indexes for table EMP in the SMEAGOL schema.

### DB2

Query SYSCAT.INDEXES:

```
1  select a.tabname, b.indname, b.colname, b.colseq
2    from syscat.indexes a,
3         syscat.indexcoluse b
4   where a.tabname   = 'EMP'
5     and a.tabschema = 'SMEAGOL'
6     and a.indschema = b.indschema
7     and a.indname   = b.indname
```

### Oracle

Query SYS.ALL\_IND\_COLUMNS:

```
select table_name, index_name, column_name, column_position
  from sys.all_ind_columns
 where table_name  = 'EMP'
   and table_owner = 'SMEAGOL'
```

### PostgreSQL

Query PG\_CATALOG.PG\_INDEXES and INFORMATION\_SCHEMA.COLUMNS:

```
1  select a.tablename,a.indexname,b.column_name
2    from pg_catalog.pg_indexes a,
3         information_schema.columns b
4   where a.schemaname = 'SMEAGOL'
5     and a.tablename  = b.table_name
```

### MySQL

[]()Use the SHOW INDEX command:

```
show index from emp
```

### SQL Server

Query SYS.TABLES, SYS.INDEXES, SYS.INDEX\_COLUMNS, and SYS.COLUMNS:

```
 1  select a.name table_name,
 2         b.name index_name,
 3         d.name column_name,
 4         c.index_column_id
 5    from sys.tables a,
 6         sys.indexes b,
 7         sys.index_columns c,
 8         sys.columns d
 9  where a.object_id = b.object_id
10    and b.object_id = c.object_id
11    and b.index_id  = c.index_id
12    and c.object_id = d.object_id
13    and c.column_id = d.column_id
14    and a.name      = 'EMP'
```

## Discussion

When it comes to queries, it’s important to know what columns are/aren’t indexed. Indexes can provide good performance for queries against columns that are frequently used in filters and that are fairly selective. Indexes are also useful when joining between tables. By knowing what columns are indexed, you are already one step ahead of performance problems if they should occur. Additionally, you might want to find information about the indexes themselves: how many levels deep they are, how many distinct keys there are, how many leaf blocks there are, and so forth. Such information is also available from the views/tables queried in this recipe’s solutions.

# 5.4 Listing Constraints on a Table

## Problem

[]()[]()You want to list the constraints defined for a table in some schema and the columns they are defined on. For example, you want to find the constraints and the columns they are on for table EMP.

## Solution

### DB2

Query SYSCAT.TABCONST and SYSCAT.COLUMNS:

```
1  select a.tabname, a.constname, b.colname, a.type
2    from syscat.tabconst a,
3         syscat.columns b
4  where a.tabname   = 'EMP'
5    and a.tabschema = 'SMEAGOL'
6    and a.tabname   = b.tabname
7    and a.tabschema = b.tabschema
```

### Oracle

Query SYS.ALL\_CONSTRAINTS and SYS.ALL\_CONS\_COLUMNS:

```
 1  select a.table_name,
 2         a.constraint_name,
 3         b.column_name,
 4         a.constraint_type
 5    from all_constraints a,
 6         all_cons_columns b
 7  where a.table_name      = 'EMP'
 8    and a.owner           = 'SMEAGOL'
 9    and a.table_name      = b.table_name
10    and a.owner           = b.owner
11    and a.constraint_name = b.constraint_name
```

### PostgreSQL, MySQL, and SQL Server

Query INFORMATION\_SCHEMA.TABLE\_CONSTRAINTS and INFORMATION_ SCHEMA.KEY\_COLUMN\_USAGE:

```
 1  select a.table_name,
 2         a.constraint_name,
 3         b.column_name,
 4         a.constraint_type
 5    from information_schema.table_constraints a,
 6         information_schema.key_column_usage b
 7  where a.table_name      = 'EMP'
 8    and a.table_schema    = 'SMEAGOL'
 9    and a.table_name      = b.table_name
10    and a.table_schema    = b.table_schema
11    and a.constraint_name = b.constraint_name
```

## Discussion

Constraints are such a critical part of relational databases that it should go without saying why you need to know what constraints are on your tables. Listing the constraints on tables is useful for a variety of reasons: you may want to find tables missing a primary key, you may want to find which columns should be foreign keys but are not (i.e., child tables have data different from the parent tables and you want to know how that happened), or you may want to know about check constraints (Are columns nullable? Do they have to satisfy a specific condition? etc.).

# 5.5 Listing Foreign Keys Without Corresponding Indexes

## Problem

[]()[]()[]()You want to list tables that have foreign key columns that are not indexed. For example, you want to determine whether the foreign keys on table EMP are indexed.

## Solution

### DB2

Query SYSCAT.TABCONST, SYSCAT.KEYCOLUSE, SYSCAT.INDEXES, and SYSCAT.INDEXCOLUSE:

```
 1  select fkeys.tabname,
 2         fkeys.constname,
 3         fkeys.colname,
 4         ind_cols.indname
 5    from (
 6  select a.tabschema, a.tabname, a.constname, b.colname
 7    from syscat.tabconst a,
 8         syscat.keycoluse b
 9  where a.tabname    = 'EMP'
10    and a.tabschema  = 'SMEAGOL'
11    and a.type       = 'F'
12    and a.tabname    = b.tabname
13    and a.tabschema  = b.tabschema
14        ) fkeys
15        left join
16        (
17  select a.tabschema,
18         a.tabname,
19         a.indname,
20         b.colname
21    from syscat.indexes a,
22         syscat.indexcoluse b
23  where a.indschema  = b.indschema
24    and a.indname    = b.indname
25        ) ind_cols
26     on (fkeys.tabschema = ind_cols.tabschema
27          and fkeys.tabname   = ind_cols.tabname
28          and fkeys.colname   = ind_cols.colname )
29  where ind_cols.indname is null
```

### Oracle

Query SYS.ALL\_CONS\_COLUMNS, SYS.ALL\_CONSTRAINTS, and SYS.ALL\_IND\_COLUMNS:

```
 1  select a.table_name,
 2         a.constraint_name,
 3         a.column_name,
 4         c.index_name
 5    from all_cons_columns a,
 6         all_constraints b,
 7         all_ind_columns c
 8  where a.table_name      = 'EMP'
 9    and a.owner           = 'SMEAGOL'
10    and b.constraint_type = 'R'
11    and a.owner           = b.owner
12    and a.table_name      = b.table_name
13    and a.constraint_name = b.constraint_name
14    and a.owner           = c.table_owner (+)
15    and a.table_name      = c.table_name (+)
16    and a.column_name     = c.column_name (+)
17    and c.index_name      is null
```

### PostgreSQL

Query INFORMATION\_SCHEMA.KEY\_COLUMN\_USAGE, INFORMATION_ SCHEMA.REFERENTIAL\_CONSTRAINTS, INFORMATION\_SCHEMA.COLUMNS, and PG\_CATALOG.PG\_INDEXES:

```
 1  select fkeys.table_name,
 2         fkeys.constraint_name,
 3         fkeys.column_name,
 4         ind_cols.indexname
 5    from (
 6  select a.constraint_schema,
 7         a.table_name,
 8         a.constraint_name,
 9         a.column_name
10    from information_schema.key_column_usage a,
11         information_schema.referential_constraints b
12   where a.constraint_name   = b.constraint_name
13     and a.constraint_schema = b.constraint_schema
14     and a.constraint_schema = 'SMEAGOL'
15     and a.table_name        = 'EMP'
16         ) fkeys
17         left join
18         (
19  select a.schemaname, a.tablename, a.indexname, b.column_name
20    from pg_catalog.pg_indexes a,
21         information_schema.columns b
22   where a.tablename  = b.table_name
23     and a.schemaname = b.table_schema
24         ) ind_cols
25      on (  fkeys.constraint_schema = ind_cols.schemaname
26           and fkeys.table_name     = ind_cols.tablename
27           and fkeys.column_name    = ind_cols.column_name )
28   where ind_cols.indexname is null
```

### MySQL

[]()You can use the SHOW INDEX command to retrieve index information such as index name, columns in the index, and ordinal position of the columns in the index. Additionally, you can query INFORMATION\_SCHEMA.KEY\_COLUMN\_USAGE to list the foreign keys for a given table. In MySQL 5, foreign keys are said to be indexed automatically, but can in fact be dropped. To determine whether a foreign key column’s index has been dropped, you can execute SHOW INDEX for a particular table and compare the output with that of INFORMATION\_SCHEMA.KEY_ COLUMN\_USAGE.COLUMN\_NAME for the same table. If the COLUMN\_NAME is listed in KEY\_COLUMN\_USAGE but is not returned by SHOW INDEX, you know that column is not indexed.

### SQL Server

Query SYS.TABLES, SYS.FOREIGN\_KEYS, SYS.COLUMNS, SYS.INDEXES, and SYS.INDEX\_COLUMNS:

```
 1  select fkeys.table_name,
 2         fkeys.constraint_name,
 3         fkeys.column_name,
 4         ind_cols.index_name
 5    from (
 6  select a.object_id,
 7         d.column_id,
 8         a.name table_name,
 9         b.name constraint_name,
10         d.name column_name
11    from sys.tables a
12         join
13         sys.foreign_keys b
14      on ( a.name          = 'EMP'
15           and a.object_id = b.parent_object_id
16         )
17         join
18         sys.foreign_key_columns c
19     on (  b.object_id = c.constraint_object_id )
20        join
21        sys.columns d
22     on (    c.constraint_column_id = d.column_id
23         and a.object_id            = d.object_id
24        )
25        ) fkeys
26        left join
27        (
28 select a.name index_name,
29        b.object_id,
30        b.column_id
31   from sys.indexes a,
32        sys.index_columns b
33  where a.index_id = b.index_id
34        ) ind_cols
35     on (     fkeys.object_id = ind_cols.object_id
36          and fkeys.column_id = ind_cols.column_id )
37  where ind_cols.index_name is null
```

## Discussion

Each vendor uses its own locking mechanism when modifying rows. In cases where there is a parent-child relationship enforced via foreign key, having indexes on the child column(s) can reducing locking (see your specific RDBMS documentation for details). In other cases, it is common that a child table is joined to a parent table on the foreign key column, so an index may help improve performance in that scenario as well.[]()[]()[]()

# 5.6 Using SQL to Generate SQL

## Problem

[]()[]()[]()You want to create dynamic SQL statements, perhaps to automate maintenance tasks. You want to accomplish three tasks in particular: count the number of rows in your tables, disable foreign key constraints defined on your tables, and generate insert scripts from the data in your tables.

## Solution

The concept is to use strings to build SQL statements, and the values that need to be filled in (such as the object name the command acts upon) will be supplied by data from the tables you are selecting from. Keep in mind, the queries only generate the statements; you must then run these statements via script, manually, or however you execute your SQL statements. The following examples are queries that would work on an Oracle system. For other RDBMSs the technique is exactly the same, the only difference being things like the names of the data dictionary tables and date formatting. The output shown from the queries that follow are a portion of the rows returned from an instance of Oracle on my laptop. Your result sets will of course vary:

```
/* generate SQL to count all the rows in all your tables */

 select 'select count(*) from '||table_name||';' cnts 
   from user_tables; 

CNTS
----------------------------------------
select count(*) from ANT;
select count(*) from BONUS;
select count(*) from DEMO1;
select count(*) from DEMO2;
select count(*) from DEPT;
select count(*) from DUMMY;
select count(*) from EMP;
select count(*) from EMP_SALES;
select count(*) from EMP_SCORE;
select count(*) from PROFESSOR;
select count(*) from T;
select count(*) from T1;
select count(*) from T2;
select count(*) from T3;
select count(*) from TEACH;
select count(*) from TEST;
select count(*) from TRX_LOG;
select count(*) from X;

/* disable foreign keys from all tables */

 select 'alter table '||table_name|| 
        ' disable constraint '||constraint_name||';' cons 
   from user_constraints 
  where constraint_type = 'R'; 

CONS
------------------------------------------------
alter table ANT disable constraint ANT_FK;
alter table BONUS disable constraint BONUS_FK;
alter table DEMO1 disable constraint DEMO1_FK;
alter table DEMO2 disable constraint DEMO2_FK;
alter table DEPT disable constraint DEPT_FK;
alter table DUMMY disable constraint DUMMY_FK;
alter table EMP disable constraint EMP_FK;
alter table EMP_SALES disable constraint EMP_SALES_FK;
alter table EMP_SCORE disable constraint EMP_SCORE_FK;
alter table PROFESSOR disable constraint PROFESSOR_FK;

/* generate an insert script from some columns in table EMP */

 select 'insert into emp(empno,ename,hiredate) '||chr(10)|| 
        'values( '||empno||','||''''||ename 
        ||''',to_date('||''''||hiredate||''') );' inserts 
   from emp 
  where deptno = 10; 

INSERTS
--------------------------------------------------
insert into emp(empno,ename,hiredate)
values( 7782,'CLARK',to_date('09-JUN-2006 00:00:00') );

insert into emp(empno,ename,hiredate)
values( 7839,'KING',to_date('17-NOV-2006 00:00:00') );

insert into emp(empno,ename,hiredate)
values( 7934,'MILLER',to_date('23-JAN-2007 00:00:00') );
```

## Discussion

Using SQL to generate SQL is particularly useful for creating portable scripts such as you might use when testing on multiple environments. Additionally, as can be seen by the previous examples, using SQL to generate SQL is useful for performing batch maintenance, and for easily finding out information about multiple objects in one go. Generating SQL with SQL is an extremely simple operation, and the more you experiment with it, the easier it will become. The examples provided should give you a nice base on how to build your own “dynamic” SQL scripts because, quite frankly, there’s not much to it. Work on it and you’ll get it.[]()[]()[]()

# 5.7 Describing the Data Dictionary Views in an Oracle Database

## Problem

[]()[]()You are using Oracle. You can’t remember what data dictionary views are available to you, nor can you remember their column definitions. Worse yet, you do not have convenient access to vendor documentation.

## Solution

This is an Oracle-specific recipe. Not only does Oracle maintain a robust set of data dictionary views, but there are also data dictionary views to document the data dictionary views. It’s all so wonderfully circular.

[]()Query the view named DICTIONARY to list data dictionary views and their purposes:

```
select table_name, comments
  from dictionary
  order by table_name;

TABLE_NAME                     COMMENTS
------------------------------ --------------------------------------------
ALL_ALL_TABLES                 Description of all object and relational
                               tables accessible to the user

ALL_APPLY                      Details about each apply process that
                               dequeues from the queue visible to the
                               current user
…
```

Query DICT\_COLUMNS to describe the columns in a given data dictionary view:

```
select column_name, comments
     from dict_columns
 where table_name = 'ALL_TAB_COLUMNS';

COLUMN_NAME                     COMMENTS
------------------------------- --------------------------------------------
OWNER
TABLE_NAME                      Table, view or cluster name
COLUMN_NAME                     Column name
DATA_TYPE                       Datatype of the column
DATA_TYPE_MOD                   Datatype modifier of the column
DATA_TYPE_OWNER                 Owner of the datatype of the column
DATA_LENGTH                     Length of the column in bytes
DATA_PRECISION                  Length: decimal digits (NUMBER) or binary
                                digits (FLOAT)
```

## Discussion

Back in the day, when Oracle’s documentation set wasn’t so freely available on the web, it was incredibly convenient that Oracle made the DICTIONARY and DICT_ COLUMNS views available. Knowing just those two views, you could bootstrap to learning about all the other views and then shift to learning about your entire database.

Even today, it’s convenient to know about DICTIONARY and DICT\_COLUMNS. Often, if you aren’t quite certain which view describes a given object type, you can issue a wildcard query to find out. For example, to get a handle on what views might describe tables in your schema:

```
select table_name, comments
  from dictionary
 where table_name LIKE '%TABLE%'
 order by table_name;
```

This query returns all data dictionary view names that include the term TABLE. This approach takes advantage of Oracle’s fairly consistent data dictionary view naming conventions. Views describing tables are all likely to contain TABLE in their name. (Sometimes, as in the case of ALL\_TAB\_COLUMNS, TABLE is abbreviated TAB.)

# 5.8 Summing Up

Queries on metadata open up a range of possibilities for letting SQL do more of the work than you, and they relieve some of the need to *know* your database. This is especially useful as you deal with more complex databases with similarly complex structures.[]()