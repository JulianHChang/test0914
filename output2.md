**Chapter 14. Odds 'n' Ends**

This chapter contains queries that didn't fit in any other chapter,
either because the chapter they would belong to is already long enough,
or because the problems they solve are more fun than realistic. This
chapter is meant to be a "fun" chapter, in that the recipes here may or
may not be recipes that you would actually use; nevertheless, the
queries are interesting, and we wanted to include them in this book.

**14.1 Creating Cross-Tab Reports Using SQL Server's PIVOT Operator**

**Problem**

You want to create a cross-tab report to transform your result set's
rows into columns. You are aware of traditional methods of pivoting but
would like to try something different. In particular, you want to return
the following result set without using CASE expressions or joins:

DEPT_10 DEPT_20 DEPT_30 DEPT_40

\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\--

3 5 6 0

**Solution**

Use the PIVOT operator to create the required result set without CASE
expressions or additional joins:

1 select \[10\] as dept_10,

2 \[20\] as dept_20,

3 \[30\] as dept_30,

4 \[40\] as dept_40

5 from (select deptno, empno from emp) driver

6 pivot (

7 count(driver.empno)

8 for driver.deptno in ( \[10\],\[20\],\[30\],\[40\] )

9 ) as empPivot

**Discussion**

The PIVOT operator may seem strange at first, but the operation it
performs in the solution is technically the same as the more familiar
transposition query shown here:

**select sum(case deptno when 10 then 1 else 0 end) as dept_10,**

**sum(case deptno when 20 then 1 else 0 end) as dept_20,**

**sum(case deptno when 30 then 1 else 0 end) as dept_30,**

**sum(case deptno when 40 then 1 else 0 end) as dept_40**

**from emp**

DEPT_10 DEPT_20 DEPT_30 DEPT_40

\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\--

3 5 6 0

Now that you know what is essentially happening, let's break down what
the PIVOT operator is doing. Line 5 of the solution shows an inline view
named DRIVER:

from (select deptno, empno from emp) driver

We've used the alias DRIVER because the rows from this inline view (or
table expression) feed directly into the PIVOT operation. The PIVOT
operator rotates the rows to columns by evaluating the items listed on
line 8 in the FOR list (shown here):

for driver.deptno in ( \[10\],\[20\],\[30\],\[40\] )

The evaluation goes something like this:

1.  If there are any DEPTNOs with a value of 10, perform the aggregate
    operation defined (COUNT(DRIVER.EMPNO)) for those rows.

2.  Repeat for DEPTNOs 20, 30, and 40.

The items listed in the brackets on line 8 serve not only to define
values for which aggregation is performed; the items also become the
column names in the result set (without the square brackets). In the
SELECT clause of the solution, the items in the FOR list are referenced
and aliased. If you do not alias the items in the FOR list, the column
names become the items in the FOR list sans brackets.

Interestingly enough, since inline view DRIVER is just that---an inline
view---you may put more complex SQL in there. For example, consider the
situation where you want to modify the result set such that the actual
department name is the name of the column. Listed here are the rows in
table DEPT:

**select \* from dept**

DEPTNO DNAME LOC

\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\--

10 ACCOUNTING NEW YORK

20 RESEARCH DALLAS

30 SALES CHICAGO

40 OPERATIONS BOSTON

You want to use PIVOT to return the following result set:

ACCOUNTING RESEARCH SALES OPERATIONS

\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\--

3 5 6 0

Because inline view DRIVER can be practically any valid table
expression, you can perform the join from table EMP to table DEPT and
then have PIVOT evaluate those rows. The following query will return the
desired result set:

select \[ACCOUNTING\] as ACCOUNTING,

\[SALES\] as SALES,

\[RESEARCH\] as RESEARCH,

\[OPERATIONS\] as OPERATIONS

from (

select d.dname, e.empno

from emp e,dept d

where e.deptno=d.deptno

) driver

pivot (

count(driver.empno)

for driver.dname in
(\[ACCOUNTING\],\[SALES\],\[RESEARCH\],\[OPERATIONS\])

) as empPivot

As you can see, PIVOT provides an interesting spin on pivoting result
sets. Regardless of whether you prefer using it to the traditional
methods of pivoting, it's nice to have another tool in your toolbox.

**14.2 Unpivoting a Cross-Tab Report Using SQL Server's UNPIVOT
Operator**

**Problem**

You have a pivoted result set (or simply a fact table), and you want to
unpivot the result set. For example, instead of having a result set with
one row and four columns, you want to return a result set with two
columns and four rows. Using the result set from the previous recipe,
you want to convert it from this:

ACCOUNTING RESEARCH SALES OPERATIONS

\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\--

3 5 6 0

to this:

DNAME CNT

\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

ACCOUNTING 3

RESEARCH 5

SALES 6

OPERATIONS 0

**Solution**

You didn't think SQL Server would give you the ability to PIVOT without
being able to UNPIVOT, did you? To unpivot the result set, just use it
as the driver and let the UNPIVOT operator do all the work. All you need
to do is specify the column names:

1 select DNAME, CNT

2 from (

3 select \[ACCOUNTING\] as ACCOUNTING,

4 \[SALES\] as SALES,

5 \[RESEARCH\] as RESEARCH,

6 \[OPERATIONS\] as OPERATIONS

7 from (

8 select d.dname, e.empno

9 from emp e,dept d

10 where e.deptno=d.deptno

11

12 ) driver

13 pivot (

14 count(driver.empno)

15 for driver.dname in
(\[ACCOUNTING\],\[SALES\],\[RESEARCH\],\[OPERATIONS\])

16 ) as empPivot

17 ) new_driver

18 unpivot (cnt for dname in (ACCOUNTING,SALES,RESEARCH,OPERATIONS)

19 ) as un_pivot

Ideally, before reading this recipe you've read the one prior to it,
because the inline view NEW_DRIVER is simply the code from the previous
recipe (if you don't understand it, please refer to the previous recipe
before looking at this one). Since lines 3--16 consist of code you've
already seen, the only new syntax is on line 18, where you use UNPIVOT.

The UNPIVOT command simply looks at the result set from NEW_DRIVER and
evaluates each column and row. For example, the UNPIVOT operator
evaluates the column names from NEW_DRIVER. When it encounters
ACCOUNTING, it transforms the column name ACCOUNTING into a row value
(under the column DNAME). It also takes the value for ACCOUNTING from
NEW_DRIVER (which is 3) and returns that as part of the ACCOUNTING row
as well (under the column CNT). UNPIVOT does this for each of the items
specified in the FOR list and simply returns each one as a row.

The new result set is now skinny and has two columns, DNAME and CNT,
with four rows:

**select DNAME, CNT**

**from (**

**select \[ACCOUNTING\] as ACCOUNTING,**

**\[SALES\] as SALES,**

**\[RESEARCH\] as RESEARCH,**

**\[OPERATIONS\] as OPERATIONS**

**from (**

**select d.dname, e.empno**

**from emp e,dept d**

**where e.deptno=d.deptno**

**) driver**

**pivot (**

**count(driver.empno)**

**for driver.dname in (
\[ACCOUNTING\],\[SALES\],\[RESEARCH\],\[OPERATIONS\] )**

**) as empPivot**

**) new_driver**

**unpivot (cnt for dname in (ACCOUNTING,SALES,RESEARCH,OPERATIONS)**

**) as un_pivot**

DNAME CNT

\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

ACCOUNTING 3

RESEARCH 5

SALES 6

OPERATIONS 0

**14.3 Transposing a Result Set Using Oracle's MODEL Clause**

**Problem**

Like the first recipe in this chapter, you want to find an alternative
to the traditional pivoting techniques you've seen already. You want to
try your hand at Oracle's MODEL clause. Unlike SQL Server's PIVOT
operator, Oracle's MODEL clause does not exist to transpose result sets;
as a matter of fact, it would be quite accurate to say the application
of the MODEL clause for pivoting would be a misuse and clearly not what
the MODEL clause was intended for. Nevertheless, the MODEL clause
provides for an interesting approach to a common problem. For this
particular problem, you want to transform the following result set from
this:

**select deptno, count(\*) cnt**

**from emp**

**group by deptno**

DEPTNO CNT

\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

10 3

20 5

30 6

to this:

D10 D20 D30

\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

3 5 6

**Solution**

Use aggregation and CASE expressions in the MODEL clause just as you
would use them if pivoting with traditional techniques. The main
difference in this case is that you use arrays to store the values of
the aggregation and return the arrays in the result set:

select max(d10) d10,

max(d20) d20,

max(d30) d30

from (

select d10,d20,d30

from ( select deptno, count(\*) cnt from emp group by deptno )

model

dimension by(deptno d)

measures(deptno, cnt d10, cnt d20, cnt d30)

rules(

d10\[any\] = case when deptno\[cv()\]=10 then d10\[cv()\] else 0 end,

d20\[any\] = case when deptno\[cv()\]=20 then d20\[cv()\] else 0 end,

d30\[any\] = case when deptno\[cv()\]=30 then d30\[cv()\] else 0 end

)

)

**Discussion**

The MODEL clause is a powerful addition to the Oracle SQL toolbox. Once
you begin working with MODEL, you'll notice helpful features such as
iteration, array access to row values, the ability to "upsert" rows into
a result set, and the ability to build reference models. You'll quickly
see that this recipe doesn't take advantage of any of the cool features
the MODEL clause offers, but it's nice to be able to look at a problem
from multiple angles and use different features in unexpected ways (if
for no other reason than to learn where certain features are more useful
than others).

The first step to understanding the solution is to examine the inline
view in the FROM clause. The inline view simply counts the number of
employees in each DEPTNO in table EMP. The results are shown here:

**select deptno, count(\*) cnt**

**from emp**

**group by deptno**

DEPTNO CNT

\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

10 3

20 5

30 6

This result set is what is given to MODEL to work with. Examining the
MODEL clause, you see three subclauses that stand out: DIMENSION BY,
MEASURES, and RULES. Let's start with MEASURES.

The items in the MEASURES list are simply the arrays you are declaring
for this query. The query uses four arrays: DEPTNO, D10, D20, and D30.
Like columns in a SELECT list, arrays in the MEASURES list can have
aliases. As you can see, three of the four arrays are actually CNT from
the inline view.

If the MEASURES list contains our arrays, then the items in the
DIMENSION BY subclause are the array indices. Consider this: array D10
is simply an alias for CNT. If you look at the result set for the
previous inline view, you'll see that CNT has three values: 3, 5, and 6.
When you create an array of CNT, you are creating an array with three
elements, namely, the three integers: 3, 5, and 6. Now, how do you
access these values from the array individually? You use the array
index. The index, defined in the DIMENSION BY subclause, has the values
of 10, 20, and 30 (from the result set above). So, for example, the
following expression:

d10\[10\]

would evaluate to 3, as you are accessing the value for CNT in array D10
for DEPTNO 10 (which is 3).

Because all three arrays (D10, D20, D30) contain the values from CNT,
all three of them have the same results. How then do we get the proper
count into the correct array? Enter the RULES subclause. If you look at
the result set for the inline view shown earlier, you'll see that the
values for DEPTNO are 10, 20, and 30. The expressions involving CASE in
the RULES clause simply evaluate each value in the DEPTNO array:

- If the value is 10, store the CNT for DEPTNO 10 in D10\[10\] or else
  store 0.

- If the value is 20, store the CNT for DEPTNO 20 in D20\[20\] or else
  store 0.

- If the value is 30, store the CNT for DEPTNO 30 in D30\[30\] or else
  store 0.

If you find yourself feeling a bit like Alice tumbling down the rabbit
hole, don't worry; just stop and execute what's been discussed thus far.
The following result set represents what has been discussed. Sometimes
it's easier to read a bit, look at the code that actually performs what
you just read, and then go back and read it again. The following is
quite simple once you see it in action:

**select deptno, d10,d20,d30**

**from ( select deptno, count(\*) cnt from emp group by deptno )**

**model**

**dimension by(deptno d)**

**measures(deptno, cnt d10, cnt d20, cnt d30)**

**rules(**

**d10\[any\] = case when deptno\[cv()\]=10 then d10\[cv()\] else 0
end,**

**d20\[any\] = case when deptno\[cv()\]=20 then d20\[cv()\] else 0
end,**

**d30\[any\] = case when deptno\[cv()\]=30 then d30\[cv()\] else 0 end**

**)**

DEPTNO D10 D20 D30

\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

10 3 0 0

20 0 5 0

30 0 0 6

As you can see, the RULES subclause is what changed the values in each
array. If you are still not catching on, simply execute the same query
but comment out the expressions in the RULES subclass:

**select deptno, d10,d20,d30**

**from ( select deptno, count(\*) cnt from emp group by deptno )**

**model**

**dimension by(deptno d)**

**measures(deptno, cnt d10, cnt d20, cnt d30)**

**rules(**

**/\***

**d10\[any\] = case when deptno\[cv()\]=10 then d10\[cv()\] else 0
end,**

**d20\[any\] = case when deptno\[cv()\]=20 then d20\[cv()\] else 0
end,**

**d30\[any\] = case when deptno\[cv()\]=30 then d30\[cv()\] else 0 end**

**\*/**

**)**

DEPTNO D10 D20 D30

\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

10 3 3 3

20 5 5 5

30 6 6 6

It should be clear now that the result set from the MODEL clause is the
same as the inline view, except that the COUNT operation is aliased D10,
D20, and D30. The following query proves this:

**select deptno, count(\*) d10, count(\*) d20, count(\*) d30**

**from emp**

**group by deptno**

DEPTNO D10 D20 D30

\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

10 3 3 3

20 5 5 5

30 6 6 6

So, all the MODEL clause did was to take the values for DEPTNO and CNT,
put them into arrays, and then make sure that each array represents a
single DEPTNO. At this point, arrays D10, D20, and D30 each have a
single nonzero value representing the CNT for a given DEPTNO. The result
set is already transposed, and all that is left is to use the aggregate
function MAX (you could have used MIN or SUM; it would make no
difference in this case) to return only one row:

**select max(d10) d10,**

**max(d20) d20,**

**max(d30) d30**

**from (**

**select d10,d20,d30**

**from ( select deptno, count(\*) cnt from emp group by deptno )**

**model**

**dimension by(deptno d)**

**measures(deptno, cnt d10, cnt d20, cnt d30)**

**rules(**

**d10\[any\] = case when deptno\[cv()\]=10 then d10\[cv()\] else 0
end,**

**d20\[any\] = case when deptno\[cv()\]=20 then d20\[cv()\] else 0
end,**

**d30\[any\] = case when deptno\[cv()\]=30 then d30\[cv()\] else 0 end**

**)**

**)**

D10 D20 D30

\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

3 5 6

**14.4 Extracting Elements of a String from Unfixed Locations**

**Problem**

You have a string field that contains serialized log data. You want to
parse through the string and extract the relevant information.
Unfortunately, the relevant information is not at fixed points in the
string. Instead, you must use the fact that certain characters exist
around the information you need, to extract said information. For
example, consider the following strings:

xxxxxabc\[867\]xxx\[-\]xxxx\[5309\]xxxxx

xxxxxtime:\[11271978\]favnum:\[4\]id:\[Joe\]xxxxx

call:\[F_GET_ROWS()\]b1:\[ROSEWOOD...SIR\]b2:\[44400002\]77.90xxxxx

film:\[non_marked\]qq:\[unit\]tailpipe:\[withabanana?\]80sxxxxx

You want to extract the values between the square brackets, returning
the following result set:

FIRST_VAL SECOND_VAL LAST_VAL

\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\--

867 - 5309

11271978 4 Joe

F_GET_ROWS() ROSEWOOD...SIR 44400002

non_marked unit withabanana?

**Solution**

Despite not knowing the exact locations within the string of the
interesting values, you do know that they are located between square
brackets \[\], and you know there are three of them. Use Oracle's
built-in function INSTR to find the locations of the brackets. Use the
built-in function SUBSTR to extract the values from the string. View V
will contain the strings to parse and is defined as follows (its use is
strictly for readability):

create view V

as

select \'xxxxxabc\[867\]xxx\[-\]xxxx\[5309\]xxxxx\' msg

from dual

union all

select \'xxxxxtime:\[11271978\]favnum:\[4\]id:\[Joe\]xxxxx\' msg

from dual

union all

select
\'call:\[F_GET_ROWS()\]b1:\[ROSEWOOD...SIR\]b2:\[44400002\]77.90xxxxx\'
msg

from dual

union all

select
\'film:\[non_marked\]qq:\[unit\]tailpipe:\[withabanana?\]80sxxxxx\' msg

from dual

1 select substr(msg,

2 instr(msg,\'\[\',1,1)+1,

3 instr(msg,\'\]\',1,1)-instr(msg,\'\[\',1,1)-1) first_val,

4 substr(msg,

5 instr(msg,\'\[\',1,2)+1,

6 instr(msg,\'\]\',1,2)-instr(msg,\'\[\',1,2)-1) second_val,

7 substr(msg,

8 instr(msg,\'\[\',-1,1)+1,

9 instr(msg,\'\]\',-1,1)-instr(msg,\'\[\',-1,1)-1) last_val

10 from V

**Discussion**

Using Oracle's built-in function INSTR makes this problem fairly simple
to solve. Since you know the values you are after are enclosed in \[\],
and that there are three sets of \[\], the first step to this solution
is to simply use INSTR to find the numeric positions of \[\] in each
string. The following example returns the numeric position of the
opening and closing brackets in each row:

**select instr(msg,\'\[\',1,1) \"1st\_\[\",**

**instr(msg,\'\]\',1,1) \"\]\_1st\",**

**instr(msg,\'\[\',1,2) \"2nd\_\[\",**

**instr(msg,\'\]\',1,2) \"\]\_2nd\",**

**instr(msg,\'\[\',-1,1) \"3rd\_\[\",**

**instr(msg,\'\]\',-1,1) \"\]\_3rd\"**

**from V**

1st\_\[ \]\_1st 2nd\_\[ \]\_2nd 3rd\_\[ \]\_3rd

\-\-\-\-\-- \-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-- \-\-\-\-\-\-\-\-\--
\-\-\-\--

9 13 17 19 24 29

11 20 28 30 34 38

6 19 23 38 42 51

6 17 21 26 36 49

At this point, the hard work is done. All that is left is to plug the
numeric positions into SUBSTR to parse MSG at those locations. You'll
notice that in the complete solution there's some simple arithmetic on
the values returned by INSTR, particularly, +1 and --1; this is
necessary to ensure the opening square bracket, \[, is not returned in
the final result set. Listed here is the solution less addition and
subtraction of 1 on the return values from INSTR; notice how each value
has a leading square bracket:

**select substr(msg,**

**instr(msg,\'\[\',1,1),**

**instr(msg,\'\]\',1,1)-instr(msg,\'\[\',1,1)) first_val,**

**substr(msg,**

**instr(msg,\'\[\',1,2),**

**instr(msg,\'\]\',1,2)-instr(msg,\'\[\',1,2)) second_val,**

**substr(msg,**

**instr(msg,\'\[\',-1,1),**

**instr(msg,\'\]\',-1,1)-instr(msg,\'\[\',-1,1)) last_val**

**from V**

FIRST_VAL SECOND_VAL LAST_VAL

\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
\-\-\-\-\-\--

\[867 \[- \[5309

\[11271978 \[4 \[Joe

\[F_GET_ROWS() \[ROSEWOOD...SIR \[44400002

\[non_marked \[unit \[withabanana?

From the previous result set, you can see that the open bracket is
there. You may be thinking: "OK, put the addition of 1 to INSTR back and
the leading square bracket goes away. Why do we need to subtract 1?" The
reason is this: if you put the addition back but leave out the
subtraction, you end up including the closing square bracket, as shown
here:

**select substr(msg,**

**instr(msg,\'\[\',1,1)+1,**

**instr(msg,\'\]\',1,1)-instr(msg,\'\[\',1,1)) first_val,**

**substr(msg,**

**instr(msg,\'\[\',1,2)+1,**

**instr(msg,\'\]\',1,2)-instr(msg,\'\[\',1,2)) second_val,**

**substr(msg,**

**instr(msg,\'\[\',-1,1)+1,**

**instr(msg,\'\]\',-1,1)-instr(msg,\'\[\',-1,1)) last_val**

**from V**

FIRST_VAL SECOND_VAL LAST_VAL

\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-\-\--

867\] -\] 5309\]

11271978\] 4\] Joe\]

F_GET_ROWS()\] ROSEWOOD...SIR\] 44400002\]

non_marked\] unit\] withabanana?\]

At this point it should be clear: to ensure you include neither of the
square brackets, you must add one to the beginning index and subtract
one from the ending index.

**14.5 Finding the Number of Days in a Year (an Alternate Solution for
Oracle)**

**Problem**

You want to find the number of days in a year.

**Tip**

This recipe presents an alternative solution to "Determining the Number
of Days in a Year" from
[[Chapter 9]{.underline}](mhtml:file://C:\Users\Buff%20Panda\OneDrive\Documents\SQL%20code\experrt%20tsql-windows-2019\14.%20Odds%20’n’%20Ends%20_%20SQL%20Cookbook,%202nd%20Edition.mhtml!https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/ch09.html#sqlckbk-CHP-9).
This solution is specific to Oracle.

**Solution**

Use the TO_CHAR function to format the last date of the year into a
three-digit day-of-the-year number:

**1 select \'Days in 2021: \'\|\|**

**2 to_char(add_months(trunc(sysdate,\'y\'),12)-1,\'DDD\')**

**3 as report**

**4 from dual**

**5 union all**

**6 select \'Days in 2020: \'\|\|**

**7 to_char(add_months(trunc(**

**8 to_date(\'01-SEP-2020\'),\'y\'),12)-1,\'DDD\')**

**9 from dual**

REPORT

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

Days in 2021: 365

Days in 2020: 366

**Discussion**

Begin by using the TRUNC function to return the first day of the year
for the given date, as follows:

**select trunc(to_date(\'01-SEP-2020\'),\'y\')**

**from dual**

TRUNC(TO_DA

\-\-\-\-\-\-\-\-\-\--

01-JAN-2020

Next, use ADD_MONTHS to add one year (12 months) to the truncated date.
Then subtract one day, bringing you to the end of the year in which your
original date falls:

**select add_months(**

**trunc(to_date(\'01-SEP-2020\'),\'y\'),**

**12) before_subtraction,**

**add_months(**

**trunc(to_date(\'01-SEP-2020\'),\'y\'),**

**12)-1 after_subtraction**

**from dual**

BEFORE_SUBT AFTER_SUBTR

\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\--

01-JAN-2021 31-DEC-2020

Now that you have found the last day in the year you are working with,
simply use TO_CHAR to return a three-digit number representing on which
day (1st, 50th, etc.) of the year the last day is:

**select to_char(**

**add_months(**

**trunc(to_date(\'01-SEP-2020\'),\'y\'),**

**12)-1,\'DDD\') num_days_in_2020**

**from dual**

NUM

\-\--

366

**14.6 Searching for Mixed Alphanumeric Strings**

**Problem**

You have a column with mixed alphanumeric data. You want to return those
rows that have both alphabetical and numeric characters; in other words,
if a string has only number or only letters, do not return it. The
return values should have a mix of both letters and numbers. Consider
the following data:

STRINGS

\-\-\-\-\-\-\-\-\-\-\--

1010 switch

333

3453430278

ClassSummary

findRow 55

threes

The final result set should contain only those rows that have both
letters and numbers:

STRINGS

\-\-\-\-\-\-\-\-\-\-\--

1010 switch

findRow 55

**Solution**

Use the built-in function TRANSLATE to convert each occurrence of a
letter or digit into a specific character. Then keep only those strings
that have at least one occurrence of both. The solution uses Oracle
syntax, but both DB2 and PostgreSQL support TRANSLATE, so modifying the
solution to work on those platforms should be trivial:

with v as (

select \'ClassSummary\' strings from dual union

select \'3453430278\' from dual union

select \'findRow 55\' from dual union

select \'1010 switch\' from dual union

select \'333\' from dual union

select \'threes\' from dual

)

select strings

from (

select strings,

translate(

strings,

\'abcdefghijklmnopqrstuvwxyz0123456789\',

rpad(\'#\',26,\'#\')\|\|rpad(\'\*\',10,\'\*\')) translated

from v

) x

whereinstr(translated,\'#\') \> 0

and instr(translated,\'\*\') \> 0

**Tip**

As an alternative to the WITH clause, you may use an inline view or
simply create a view.

**Discussion**

The TRANSLATE function makes this problem extremely easy to solve. The
first step is to use TRANSLATE to identify all letters and all digits by
pound (#) and asterisk (\*) characters, respectively. The intermediate
results (from inline view X) are as follows:

**with v as (**

**select \'ClassSummary\' strings from dual union**

**select \'3453430278\' from dual union**

**select \'findRow 55\' from dual union**

**select \'1010 switch\' from dual union**

**select \'333\' from dual union**

**select \'threes\' from dual**

**)**

**select strings,**

**translate(**

**strings,**

**\'abcdefghijklmnopqrstuvwxyz0123456789\',**

**rpad(\'#\',26,\'#\')\|\|rpad(\'\*\',10,\'\*\')) translated**

**from v**

STRINGS TRANSLATED

\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\--

1010 switch \*\*\*\* \######

333 \*\*\*

3453430278 \*\*\*\*\*\*\*\*\*\*

ClassSummary C####S######

findRow 55 ####R## \*\*

threes \######

At this point, it is only a matter of keeping those rows that have at
least one instance each of \# and \*. Use the function INSTR to
determine whether \# and \* are in a string. If those two characters
are, in fact, present, then the value returned will be greater than
zero. The final strings to return, along with their translated values,
are shown next for clarity:

**with v as (**

**select \'ClassSummary\' strings from dual union**

**select \'3453430278\' from dual union**

**select \'findRow 55\' from dual union**

**select \'1010 switch\' from dual union**

**select \'333\' from dual union**

**select \'threes\' from dual**

**)**

**select strings, translated**

**from (**

**select strings,**

**translate(**

**strings,**

**\'abcdefghijklmnopqrstuvwxyz0123456789\',**

**rpad(\'#\',26,\'#\')\|\|rpad(\'\*\',10,\'\*\')) translated**

**from v**

**)**

**where instr(translated,\'#\') \> 0**

**and instr(translated,\'\*\') \> 0**

STRINGS TRANSLATED

\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\--

1010 switch \*\*\*\* \######

findRow 55 ####R## \*\*

**14.7 Converting Whole Numbers to Binary Using Oracle**

**Problem**

You want to convert a whole number to its binary representation on an
Oracle system. For example, you would like to return all the salaries in
table EMP in binary as part of the following result set:

ENAME SAL SAL_BINARY

\-\-\-\-\-\-\-\-\-- \-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

SMITH 800 1100100000

ALLEN 1600 11001000000

WARD 1250 10011100010

JONES 2975 101110011111

MARTIN 1250 10011100010

BLAKE 2850 101100100010

CLARK 2450 100110010010

SCOTT 3000 101110111000

KING 5000 1001110001000

TURNER 1500 10111011100

ADAMS 1100 10001001100

JAMES 950 1110110110

FORD 3000 101110111000

MILLER 1300 10100010100

**Solution**

Because of MODEL's ability to iterate and provide array access to row
values, it is a natural choice for this operation (assuming you are
forced to solve the problem in SQL, as a stored function is more
appropriate here). Like the rest of the solutions in this book, even if
you don't find a practical application for this code, focus on the
technique. It is useful to know that the MODEL clause can perform
procedural tasks while still keeping SQL's set-based nature and power.
So, even if you find yourself saying, "I'd never do this in SQL," that's
fine. We're in no way suggesting you should or shouldn't. We remind you
to focus on the technique, so you can apply it to whatever you consider
a more "practical" application.

The following solution returns all ENAME and SAL from table EMP, while
calling the MODEL clause in a scalar subquery (this way it serves as
sort of a standalone function from table EMP that simply receives an
input, processes it, and returns a value, much like a function would):

1 select ename,

2 sal,

3 (

4 select bin

5 from dual

6 model

7 dimension by ( 0 attr )

8 measures ( sal num,

9 cast(null as varchar2(30)) bin,

10 \'0123456789ABCDEF\' hex

11 )

12 rules iterate (10000) until (num\[0\] \<= 0) (

13 bin\[0\] = substr(hex\[cv()\],mod(num\[cv()\],2)+1,1)\|\|bin\[cv()\],

14 num\[0\] = trunc(num\[cv()\]/2)

15 )

16 ) sal_binary

17 from emp

**Discussion**

We mentioned in the "Solution" section that this problem is most likely
better solved via a stored function. Indeed, the idea for this recipe
came from a function. As a matter of fact, this recipe is an adaptation
of a function called TO_BASE, written by Tom Kyte of Oracle Corporation.
Like other recipes in this book that you may decide not to use, even if
you do not use this recipe, it does a nice job of showing of some of the
features of the MODEL clause such as iteration and array access of rows.

To make the explanation easier, we focus on a slight variation of the
subquery containing the MODEL clause. The code that follows is
essentially the subquery from the solution, except that it's been
hardwired to return the value 2 in binary:

**select bin**

**from dual**

**model**

**dimension by ( 0 attr )**

**measures ( 2 num,**

**cast(null as varchar2(30)) bin,**

**\'0123456789ABCDEF\' hex**

**)**

**rules iterate (10000) until (num\[0\] \<= 0) (**

**bin\[0\] = substr
(hex\[cv()\],mod(num\[cv()\],2)+1,1)\|\|bin\[cv()\],**

**num\[0\] = trunc(num\[cv()\]/2)**

**)**

BIN

\-\-\-\-\-\-\-\-\--

10

The following query outputs the values returned from one iteration of
the RULES defined in the previous query:

**select 2 start_val,**

**\'0123456789ABCDEF\' hex,**

**substr(\'0123456789ABCDEF\',mod(2,2)+1,1) \|\|**

**cast(null as varchar2(30)) bin,**

**trunc(2/2) num**

**from dual**

START_VAL HEX BIN NUM

\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--
\-\--

2 0123456789ABCDEF 0 1

START_VAL represents the number you want to convert to binary, which in
this case is 2. The value for BIN is the result of a substring operation
on *0123456789ABCDEF* (HEX, in the original solution). The value for NUM
is the test that will determine when you exit the loop.

As you can see from the preceding result set, the first time through the
loop BIN is 0 and NUM is 1. Because NUM is not less than or equal to 0,
another loop iteration occurs. The following SQL statement shows the
results of the next iteration:

**select num start_val,**

**substr(\'0123456789ABCDEF\',mod(1,2)+1,1) \|\| bin bin,**

**trunc(1/2) num**

**from (**

**select 2 start_val,**

**\'0123456789ABCDEF\' hex,**

**substr(\'0123456789ABCDEF\',mod(2,2)+1,1) \|\|**

**cast(null as varchar2(30)) bin,**

**trunc(2/2) num**

**from dual**

**)**

START_VAL BIN NUM

\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\--

1 10 0

The next time through the loop, the result of the substring operation on
HEX returns 1, and the prior value of BIN, 0, is appended to it. The
test, NUM, is now 0; thus, this is the last iteration, and the return
value "10" is the binary representation of the number 2. Once you're
comfortable with what's going on, you can remove the iteration from the
MODEL clause and step through it row by row to follow how the rules are
applied to come to the final result set, as is shown here:

**select 2 orig_val, num, bin**

**from dual**

**model**

**dimension by ( 0 attr )**

**measures ( 2 num,**

**cast(null as varchar2(30)) bin,**

**\'0123456789ABCDEF\' hex**

**)**

**rules (**

**bin\[0\] = substr
(hex\[cv()\],mod(num\[cv()\],2)+1,1)\|\|bin\[cv()\],**

**num\[0\] = trunc(num\[cv()\]/2),**

**bin\[1\] = substr (hex\[0\],mod(num\[0\],2)+1,1)\|\|bin\[0\],**

**num\[1\] = trunc(num\[0\]/2)**

**)**

ORIG_VAL NUM BIN

\-\-\-\-\-\-\-- \-\-- \-\-\-\-\-\-\-\--

2 1 0

2 0 10

**14.8 Pivoting a Ranked Result Set**

**Problem**

You want to rank the values in a table and then pivot the result set
into three columns. The idea is to show the top three, the next three,
and then all the rest. For example, you want to rank the employees in
table EMP by SAL and then pivot the results into three columns. The
desired result set is as follows:

TOP_3 NEXT_3 REST

\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-\-\-\--

KING (5000) BLAKE (2850) TURNER (1500)

FORD (3000) CLARK (2450) MILLER (1300)

SCOTT (3000) ALLEN (1600) MARTIN (1250)

JONES (2975) WARD (1250)

ADAMS (1100)

JAMES (950)

SMITH (800)

**Solution**

The key to this solution is to first use the window function DENSE_RANK
OVER to rank the employees by SAL while allowing for ties. By using
DENSE_RANK OVER, you can easily see the top three salaries, the next
three salaries, and then all the rest.

Next, use the window function ROW_NUMBER OVER to rank each employee
within their group (the top three, next three, or last group). From
there, simply perform a classic transpose, while using the built-in
string functions available on your platform to beautify the results. The
following solution uses Oracle syntax. Since all vendors now support
window functions, converting the solution to work for other platforms is
trivial:

1 select max(case grp when 1 then rpad(ename,6) \|\|

2 \' (\'\|\| sal \|\|\')\' end) top_3,

3 max(case grp when 2 then rpad(ename,6) \|\|

4 \' (\'\|\| sal \|\|\')\' end) next_3,

5 max(case grp when 3 then rpad(ename,6) \|\|

6 \' (\'\|\| sal \|\|\')\' end) rest

7 from (

8 select ename,

9 sal,

10 rnk,

11 case when rnk \<= 3 then 1

12 when rnk \<= 6 then 2

13 else 3

14 end grp,

15 row_number()over (

16 partition by case when rnk \<= 3 then 1

17 when rnk \<= 6 then 2

18 else 3

19 end

20 order by sal desc, ename

21 ) grp_rnk

22 from (

23 select ename,

24 sal,

25 dense_rank()over(order by sal desc) rnk

26 from emp

27 ) x

28 ) y

29 group by grp_rnk

**Discussion**

This recipe is a perfect example of how much you can accomplish with so
little, with the help of window functions. The solution may look
involved, but as you break it down from inside out, you will be
surprised how simple it is. Let's begin by executing inline view X
first:

**select ename,**

**sal,**

**dense_rank()over(order by sal desc) rnk**

**from emp**

ENAME SAL RNK

\-\-\-\-\-\-\-\-\-- \-\-\-\-- \-\-\-\-\-\-\-\-\--

KING 5000 1

SCOTT 3000 2

FORD 3000 2

JONES 2975 3

BLAKE 2850 4

CLARK 2450 5

ALLEN 1600 6

TURNER 1500 7

MILLER 1300 8

WARD 1250 9

MARTIN 1250 9

ADAMS 1100 10

JAMES 950 11

SMITH 800 12

As you can see from the previous result set, inline view X simply ranks
the employees by SAL, while allowing for ties (because the solution uses
DENSE_RANK instead of RANK, there are ties without gaps). The next step
is to take the rows from inline view X and create groups by using a CASE
expression to evaluate the ranking from DENSE_RANK. Additionally, use
the window function ROW_NUMBER OVER to rank the employees by SAL within
their group (within the group you are creating with the CASE
expression). All of this happens in inline view Y and is shown here:

**select ename,**

**sal,**

**rnk,**

**case when rnk \<= 3 then 1**

**when rnk \<= 6 then 2**

**else 3**

**end grp,**

**row_number()over (**

**partition by case when rnk \<= 3 then 1**

**when rnk \<= 6 then 2**

**else 3**

**end**

**order by sal desc, ename**

**) grp_rnk**

**from (**

**select ename,**

**sal,**

**dense_rank()over(order by sal desc) rnk**

**from emp**

**) x**

ENAME SAL RNK GRP GRP_RNK

\-\-\-\-\-\-\-\-\-- \-\-\-\-- \-\-\-- \-\-\-- \-\-\-\-\-\--

KING 5000 1 1 1

FORD 3000 2 1 2

SCOTT 3000 2 1 3

JONES 2975 3 1 4

BLAKE 2850 4 2 1

CLARK 2450 5 2 2

ALLEN 1600 6 2 3

TURNER 1500 7 3 1

MILLER 1300 8 3 2

MARTIN 1250 9 3 3

WARD 1250 9 3 4

ADAMS 1100 10 3 5

JAMES 950 11 3 6

SMITH 800 12 3 7

Now the query is starting to take shape, and if you followed it from the
beginning (from inline view X), you can see that it's not that
complicated. The query so far returns each employee; their SAL; their
RNK, which represents where their SAL ranks among all employees; their
GRP, which indicates the group each employee is in (based on SAL); and
finally GRP_RANK, which is a ranking (based on SAL) within their GRP.

At this point, perform a traditional pivot on ENAME while using the
Oracle concatenation operator \|\| to append the SAL. The function RPAD
ensures that the numeric values in parentheses line up nicely. Finally,
use GROUP BY on GRP_RNK to ensure you show each employee in the result
set. The final result set is shown here:

**select max(case grp when 1 then rpad(ename,6) \|\|**

**\' (\'\|\| sal \|\|\')\' end) top_3,**

**max(case grp when 2 then rpad(ename,6) \|\|**

**\' (\'\|\| sal \|\|\')\' end) next_3,**

**max(case grp when 3 then rpad(ename,6) \|\|**

**\' (\'\|\| sal \|\|\')\' end) rest**

**from (**

**select ename,**

**sal,**

**rnk,**

**case when rnk \<= 3 then 1**

**when rnk \<= 6 then 2**

**else 3**

**end grp,**

**row_number()over (**

**partition by case when rnk \<= 3 then 1**

**when rnk \<= 6 then 2**

**else 3**

**end**

**Order by sal desc, ename**

**) grp_rnk**

**from (**

**select ename,**

**sal,**

**dense_rank()over(order by sal desc) rnk**

**from emp**

**) x**

**) y**

**group by grp_rnk**

TOP_3 NEXT_3 REST

\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-\-\--

KING (5000) BLAKE (2850) TURNER (1500)

FORD (3000) CLARK (2450) MILLER (1300)

SCOTT (3000) ALLEN (1600) MARTIN (1250)

JONES (2975) WARD (1250)

ADAMS (1100)

JAMES (950)

SMITH (800)

If you examine the queries in all of the steps, you'll notice that table
EMP is accessed exactly once. One of the remarkable things about window
functions is how much work you can do in just one pass through your
data. There's no need for self-joins or temp tables; just get the rows
you need and then let the window functions do the rest. Only in inline
view X do you need to access EMP. From there, it's simply a matter of
massaging the result set to look the way you want. Consider what all
this means for performance if you can create this type of report with a
single table access. Pretty cool.

**14.9 Adding a Column Header into a Double Pivoted Result Set**

**Problem**

You want to stack two result sets and then pivot them into two columns.
Additionally, you want to add a "header" for each group of rows in each
column. For example, you have two tables containing information about
employees working in different areas of development in your company
(say, in research and applications):

**select \* from it_research**

DEPTNO ENAME

\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

100 HOPKINS

100 JONES

100 TONEY

200 MORALES

200 P.WHITAKER

200 MARCIANO

200 ROBINSON

300 LACY

300 WRIGHT

300 J.TAYLOR

**select \* from it_apps**

DEPTNO ENAME

\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

400 CORRALES

400 MAYWEATHER

400 CASTILLO

400 MARQUEZ

400 MOSLEY

500 GATTI

500 CALZAGHE

600 LAMOTTA

600 HAGLER

600 HEARNS

600 FRAZIER

700 GUINN

700 JUDAH

700 MARGARITO

You would like to create a report listing the employees from each table
in two columns. You want to return the DEPTNO followed by ENAME for
each. Ultimately, you want to return the following result set:

RESEARCH APPS

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\--

100 400

JONES MAYWEATHER

TONEY CASTILLO

HOPKINS MARQUEZ

200 MOSLEY

P.WHITAKER CORRALES

MARCIANO 500

ROBINSON CALZAGHE

MORALES GATTI

300 600

WRIGHT HAGLER

J.TAYLOR HEARNS

LACY FRAZIER

LAMOTTA

700

JUDAH

MARGARITO

GUINN

**Solution**

For the most part, this solution requires nothing more than a simple
stack 'n' pivot (union then pivot) with an added twist: the DEPTNO must
precede the ENAME for each employee returned. The technique here uses a
Cartesian product to generate an extra row for each DEPTNO, so you have
the required rows necessary to show all employees, plus room for the
DEPTNO. The solution uses Oracle syntax, but since DB2 supports window
functions that can compute moving windows (the framing clause),
converting this solution to work for DB2 is trivial. Because the IT\_
RESEARCH and IT_APPS tables exist only for this recipe, their table
creation statements are shown along with this solution:

create table IT_research (deptno number, ename varchar2(20))

insert into IT_research values (100,\'HOPKINS\')

insert into IT_research values (100,\'JONES\')

insert into IT_research values (100,\'TONEY\')

insert into IT_research values (200,\'MORALES\')

insert into IT_research values (200,\'P.WHITAKER\')

insert into IT_research values (200,\'MARCIANO\')

insert into IT_research values (200,\'ROBINSON\')

insert into IT_research values (300,\'LACY\')

insert into IT_research values (300,\'WRIGHT\')

insert into IT_research values (300,\'J.TAYLOR\')

create table IT_apps (deptno number, ename varchar2(20))

insert into IT_apps values (400,\'CORRALES\')

insert into IT_apps values (400,\'MAYWEATHER\')

insert into IT_apps values (400,\'CASTILLO\')

insert into IT_apps values (400,\'MARQUEZ\')

insert into IT_apps values (400,\'MOSLEY\')

insert into IT_apps values (500,\'GATTI\')

insert into IT_apps values (500,\'CALZAGHE\')

insert into IT_apps values (600,\'LAMOTTA\')

insert into IT_apps values (600,\'HAGLER\')

insert into IT_apps values (600,\'HEARNS\')

insert into IT_apps values (600,\'FRAZIER\')

insert into IT_apps values (700,\'GUINN\')

insert into IT_apps values (700,\'JUDAH\')

insert into IT_apps values (700,\'MARGARITO\')

1 select max(decode(flag2,0,it_dept)) research,

2 max(decode(flag2,1,it_dept)) apps

3 from (

4 select sum(flag1)over(partition by flag2

5 order by flag1,rownum) flag,

6 it_dept, flag2

7 from (

8 select 1 flag1, 0 flag2,

9 decode(rn,1,to_char(deptno),\' \'\|\|ename) it_dept

10 from (

11 select x.\*, y.id,

12 row_number()over(partition by x.deptno order by y.id) rn

13 from (

14 select deptno,

15 ename,

16 count(\*)over(partition by deptno) cnt

17 from it_research

18 ) x,

19 (select level id from dual connect by level \<= 2) y

20 )

21 where rn \<= cnt+1

22 union all

23 select 1 flag1, 1 flag2,

24 decode(rn,1,to_char(deptno),\' \'\|\|ename) it_dept

25 from (

26 select x.\*, y.id,

27 row_number()over(partition by x.deptno order by y.id) rn

28 from (

29 select deptno,

30 ename,

31 count(\*)over(partition by deptno) cnt

32 from it_apps

33 ) x,

34 (select level id from dual connect by level \<= 2) y

35 )

36 where rn \<= cnt+1

37 ) tmp1

38 ) tmp2

39 group by flag

**Discussion**

Like many of the other warehousing/report type queries, the solution
presented looks quite convoluted, but once broken down, you'll seen it's
nothing more than a stack 'n' pivot with a Cartesian twist (on the
rocks, with a little umbrella). The way to break down this query is to
work on each part of the UNION ALL first and then bring it together for
the pivot. Let's start with the lower portion of the UNION ALL:

**select 1 flag1, 1 flag2,**

**decode(rn,1,to_char(deptno),\' \'\|\|ename) it_dept**

**from (**

**select x.\*, y.id,**

**row_number()over(partition by x.deptno order by y.id) rn**

**from (**

**select deptno,**

**ename,**

**count(\*)over(partition by deptno) cnt**

**from it_apps**

**) x,**

**(select level id from dual connect by level \<= 2) y**

**) z**

**where rn \<= cnt+1**

FLAG1 FLAG2 IT_DEPT

\-\-\-\-- \-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

1 1 400

1 1 MAYWEATHER

1 1 CASTILLO

1 1 MARQUEZ

1 1 MOSLEY

1 1 CORRALES

1 1 500

1 1 CALZAGHE

1 1 GATTI

1 1 600

1 1 HAGLER

1 1 HEARNS

1 1 FRAZIER

1 1 LAMOTTA

1 1 700

1 1 JUDAH

1 1 MARGARITO

1 1 GUINN

Let's examine exactly how that result set is put together. Breaking down
the previous query to its simplest components, you have inline view X,
which simply returns each ENAME and DEPTNO and the number of employees
in each DEPTNO from table IT_APPS. The results are as follows:

**select deptno deptno,**

**ename,**

**count(\*)over(partition by deptno) cnt**

**from it_apps**

DEPTNO ENAME CNT

\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

400 CORRALES 5

400 MAYWEATHER 5

400 CASTILLO 5

400 MARQUEZ 5

400 MOSLEY 5

500 GATTI 2

500 CALZAGHE 2

600 LAMOTTA 4

600 HAGLER 4

600 HEARNS 4

600 FRAZIER 4

700 GUINN 3

700 JUDAH 3

700 MARGARITO 3

The next step is to create a Cartesian product between the rows returned
from inline view X and two rows generated from DUAL using CONNECT BY.
The results of this operation are as follows:

**select \***

**from (**

**select deptno deptno,**

**ename,**

**count(\*)over(partition by deptno) cnt**

**from it_apps**

**) x,**

**(select level id from dual connect by level \<= 2) y**

**order by 2**

DEPTNO ENAME CNT ID

\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-- \-\--

500 CALZAGHE 2 1

500 CALZAGHE 2 2

400 CASTILLO 5 1

400 CASTILLO 5 2

400 CORRALES 5 1

400 CORRALES 5 2

600 FRAZIER 4 1

600 FRAZIER 4 2

500 GATTI 2 1

500 GATTI 2 2

700 GUINN 3 1

700 GUINN 3 2

600 HAGLER 4 1

600 HAGLER 4 2

600 HEARNS 4 1

600 HEARNS 4 2

700 JUDAH 3 1

700 JUDAH 3 2

600 LAMOTTA 4 1

600 LAMOTTA 4 2

700 MARGARITO 3 1

700 MARGARITO 3 2

400 MARQUEZ 5 1

400 MARQUEZ 5 2

400 MAYWEATHER 5 1

400 MAYWEATHER 5 2

400 MOSLEY 5 1

400 MOSLEY 5 2

As you can see from these results, each row from inline view X is now
returned twice due to the Cartesian product with inline view Y. The
reason a Cartesian is needed will become clear shortly. The next step is
to take the current result set and rank each employee within his DEPTNO
by ID (ID has a value of 1 or 2 as was returned by the Cartesian
product). The result of this ranking is shown in the output from the
following query:

**select x.\*, y.id,**

**row_number()over(partition by x.deptno order by y.id) rn**

**from (**

**select deptno deptno,**

**ename,**

**count(\*)over(partition by deptno) cnt**

**from it_apps**

**) x,**

**(select level id from dual connect by level \<= 2) y**

DEPTNO ENAME CNT ID RN

\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-- \-\-- \-\-\-\-\-\-\-\-\--

400 CORRALES 5 1 1

400 MAYWEATHER 5 1 2

400 CASTILLO 5 1 3

400 MARQUEZ 5 1 4

400 MOSLEY 5 1 5

400 CORRALES 5 2 6

400 MOSLEY 5 2 7

400 MAYWEATHER 5 2 8

400 CASTILLO 5 2 9

400 MARQUEZ 5 2 10

500 GATTI 2 1 1

500 CALZAGHE 2 1 2

500 GATTI 2 2 3

500 CALZAGHE 2 2 4

600 LAMOTTA 4 1 1

600 HAGLER 4 1 2

600 HEARNS 4 1 3

600 FRAZIER 4 1 4

600 LAMOTTA 4 2 5

600 HAGLER 4 2 6

600 FRAZIER 4 2 7

600 HEARNS 4 2 8

700 GUINN 3 1 1

700 JUDAH 3 1 2

700 MARGARITO 3 1 3

700 GUINN 3 2 4

700 JUDAH 3 2 5

700 MARGARITO 3 2 6

Each employee is ranked; then his duplicate is ranked. The result set
contains duplicates for all employees in table IT_APP, along with their
ranking within their DEPTNO. The reason you need to generate these extra
rows is because you need a slot in the result set to slip in the DEPTNO
in the ENAME column. If you Cartesian-join IT_APPS with a one-row table,
you get no extra rows (because cardinality of any table × 1 =
cardinality of that table).

The next step is to take the results returned thus far and pivot the
result set such that all the ENAMEs are returned in one column but are
preceded by the DEPTNO they are in. The following query shows how this
happens:

**select 1 flag1, 1 flag2,**

**decode(rn,1,to_char(deptno),\' \'\|\|ename) it_dept**

**from (**

**select x.\*, y.id,**

**row_number()over(partition by x.deptno order by y.id) rn**

**from (**

**select deptno deptno,**

**ename,**

**count(\*)over(partition by deptno) cnt**

**from it_apps**

**) x,**

**(select level id from dual connect by level \<= 2) y**

**) z**

**where rn \<= cnt+1**

FLAG1 FLAG2 IT_DEPT

\-\-\-\-- \-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

1 1 400

1 1 MAYWEATHER

1 1 CASTILLO

1 1 MARQUEZ

1 1 MOSLEY

1 1 CORRALES

1 1 500

1 1 CALZAGHE

1 1 GATTI

1 1 600

1 1 HAGLER

1 1 HEARNS

1 1 FRAZIER

1 1 LAMOTTA

1 1 700

1 1 JUDAH

1 1 MARGARITO

1 1 GUINN

FLAG1 and FLAG2 come into play later and can be ignored for the moment.
Focus your attention on the rows in IT_DEPT. The number of rows returned
for each DEPTNO is CNT\*2, but all that is needed is CNT+1, which is the
filter in the WHERE clause. RN is the ranking for each employee. The
rows kept are all those ranked less than or equal to CNT+1; i.e., all
employees in each DEPTNO plus one more (this extra employee is the
employee who is ranked first in their DEPTNO). This extra row is where
the DEPTNO will slide in. By using DECODE (an older Oracle function that
gives more or less the equivalent of a CASE expression) to evaluate the
value of RN, you can slide the value of DEPTNO into the result set. The
employee who was at position one (based on the value of RN) is still
shown in the result set, but is now last in each DEPTNO (because the
order is irrelevant, this is not a problem). That pretty much covers the
lower part of the UNION ALL.

The upper part of the UNION ALL is processed in the same way as the
lower part, so there's no need to explain how that works. Instead, let's
examine the result set returned when stacking the queries:

**select 1 flag1, 0 flag2,**

**decode(rn,1,to_char(deptno),\' \'\|\|ename) it_dept**

**from (**

**select x.\*, y.id,**

**row_number()over(partition by x.deptno order by y.id) rn**

**from (**

**select deptno,**

**ename,**

**count(\*)over(partition by deptno) cnt**

**from it_research**

**) x,**

**(select level id from dual connect by level \<= 2) y**

**)**

**where rn \<= cnt+1**

**union all**

**select 1 flag1, 1 flag2,**

**decode(rn,1,to_char(deptno),\' \'\|\|ename) it_dept**

**from (**

**select x.\*, y.id,**

**row_number()over(partition by x.deptno order by y.id) rn**

**from (**

**select deptno deptno,**

**ename,**

**count(\*)over(partition by deptno) cnt**

**from it_apps**

**) x,**

**(select level id from dual connect by level \<= 2) y**

**)**

**where rn \<= cnt+1**

FLAG1 FLAG2 IT_DEPT

\-\-\-\-- \-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

1 0 100

1 0 JONES

1 0 TONEY

1 0 HOPKINS

1 0 200

1 0 P.WHITAKER

1 0 MARCIANO

1 0 ROBINSON

1 0 MORALES

1 0 300

1 0 WRIGHT

1 0 J.TAYLOR

1 0 LACY

1 1 400

1 1 MAYWEATHER

1 1 CASTILLO

1 1 MARQUEZ

1 1 MOSLEY

1 1 CORRALES

1 1 500

1 1 CALZAGHE

1 1 GATTI

1 1 600

1 1 HAGLER

1 1 HEARNS

1 1 FRAZIER

1 1 LAMOTTA

1 1 700

1 1 JUDAH

1 1 MARGARITO

1 1 GUINN

At this point, it isn't clear what FLAG1's purpose is, but you can see
that FLAG2 identifies which rows come from which part of the UNION ALL
(0 for the upper part, 1 for the lower part).

The next step is to wrap the stacked result set in an inline view and
create a running total on FLAG1 (finally, its purpose is revealed!),
which will act as a ranking for each row in each stack. The results of
the ranking (running total) are shown here:

**select sum(flag1)over(partition by flag2**

**order by flag1,rownum) flag,**

**it_dept, flag2**

**from (**

**select 1 flag1, 0 flag2,**

**decode(rn,1,to_char(deptno),\' \'\|\|ename) it_dept**

**from (**

**select x.\*, y.id,**

**row_number()over(partition by x.deptno order by y.id) rn**

**from (**

**select deptno,**

**ename,**

**count(\*)over(partition by deptno) cnt**

**from it_research**

**) x,**

**(select level id from dual connect by level \<= 2) y**

**)**

**where rn \<= cnt+1**

**union all**

**select 1 flag1, 1 flag2,**

**decode(rn,1,to_char(deptno),\' \'\|\|ename) it_dept**

**from (**

**select x.\*, y.id,**

**row_number()over(partition by x.deptno order by y.id) rn**

**from (**

**select deptno deptno,**

**ename,**

**count(\*)over(partition by deptno) cnt**

**from it_apps**

**) x,**

**(select level id from dual connect by level \<= 2) y**

**)**

**where rn \<= cnt+1**

**) tmp1**

FLAG IT_DEPT FLAG2

\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

1 100 0

2 JONES 0

3 TONEY 0

4 HOPKINS 0

5 200 0

6 P.WHITAKER 0

7 MARCIANO 0

8 ROBINSON 0

9 MORALES 0

10 300 0

11 WRIGHT 0

12 J.TAYLOR 0

13 LACY 0

1 400 1

2 MAYWEATHER 1

3 CASTILLO 1

4 MARQUEZ 1

5 MOSLEY 1

6 CORRALES 1

7 500 1

8 CALZAGHEe 1

9 GATTI 1

10 600 1

11 HAGLER 1

12 HEARNS 1

13 FRAZIER 1

14 LAMOTTA 1

15 700 1

16 JUDAH 1

17 MARGARITO 1

18 GUINN 1

The last step (finally!) is to pivot the value returned by TMP1 on FLAG2
while grouping by FLAG (the running total generated in TMP1). The
results from TMP1 are wrapped in an inline view and pivoted (wrapped in
a final inline view called TMP2). The ultimate solution and result set
are shown here:

**select max(decode(flag2,0,it_dept)) research,**

**max(decode(flag2,1,it_dept)) apps**

**from (**

**select sum(flag1)over(partition by flag2**

**order by flag1,rownum) flag,**

**it_dept, flag2**

**from (**

**select 1 flag1, 0 flag2,**

**decode(rn,1,to_char(deptno),\' \'\|\|ename) it_dept**

**from (**

**select x.\*, y.id,**

**row_number()over(partition by x.deptno order by y.id) rn**

**from (**

**select deptno,**

**ename,**

**count(\*)over(partition by deptno) cnt**

**from it_research**

**) x,**

**(select level id from dual connect by level \<= 2) y**

**)**

**where rn \<= cnt+1**

**union all**

**select 1 flag1, 1 flag2,**

**decode(rn,1,to_char(deptno),\' \'\|\|ename) it_dept**

**from (**

**select x.\*, y.id,**

**row_number()over(partition by x.deptno order by y.id) rn**

**from (**

**select deptno deptno,**

**ename,**

**count(\*)over(partition by deptno) cnt**

**from it_apps**

**) x,**

**(select level id from dual connect by level \<= 2) y**

**)**

**where rn \<= cnt+1**

**) tmp1**

**) tmp2**

**group by flag**

RESEARCH APPS

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\--

100 400

JONES MAYWEATHER

TONEY CASTILLO

HOPKINS MARQUEZ

200 MOSLEY

P.WHITAKER CORRALES

MARCIANO 500

ROBINSON CALZAGHE

MORALES GATTI

300 600

WRIGHT HAGLER

J.TAYLOR HEARNS

LACY FRAZIER

LAMOTTA

700

JUDAH

MARGARITO

GUINN

**14.10 Converting a Scalar Subquery to a Composite Subquery in Oracle**

**Problem**

You want to bypass the restriction of returning exactly one value from a
scalar subquery. For example, you attempt to execute the following
query:

select e.deptno,

e.ename,

e.sal,

(select d.dname,d.loc,sysdate today

from dept d

where e.deptno=d.deptno)

from emp e

but receive an error because subqueries in the SELECT list are allowed
to return only a single value.

**Solution**

Admittedly, this problem is quite unrealistic, because a simple join
between tables EMP and DEPT would allow you to return as many values you
want from DEPT. Nevertheless, the key is to focus on the technique and
understand how to apply it to a scenario that you find useful. The key
to bypassing the requirement to return a single value when placing a
SELECT within SELECT (scalar subquery) is to take advantage of Oracle's
object types. You can define an object to have several attributes, and
then you can work with it as a single entity or reference each element
individually. In effect, you don't really bypass the rule at all. You
simply return one value---an \[.keep-together\]#object---#that in turn
contains many attributes.

This solution makes use of the following object type:

create type generic_obj

as object (

val1 varchar2(10),

val2 varchar2(10),

val3 date

);

With this type in place, you can execute the following query:

**1 select x.deptno,**

**2 x.ename,**

**3 x.multival.val1 dname,**

**4 x.multival.val2 loc,**

**5 x.multival.val3 today**

**6 from (**

**7select e.deptno,**

**8 e.ename,**

**9 e.sal,**

**10 (select generic_obj(d.dname,d.loc,sysdate+1)**

**11 from dept d**

**12 where e.deptno=d.deptno) multival**

**13 from emp e**

**14 ) x**

DEPTNO ENAME DNAME LOC TODAY

\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\--

20 SMITH RESEARCH DALLAS 12-SEP-2020

30 ALLEN SALES CHICAGO 12-SEP-2020

30 WARD SALES CHICAGO 12-SEP-2020

20 JONES RESEARCH DALLAS 12-SEP-2020

30 MARTIN SALES CHICAGO 12-SEP-2020

30 BLAKE SALES CHICAGO 12-SEP-2020

10 CLARK ACCOUNTING NEW YORK 12-SEP-2020

20 SCOTT RESEARCH DALLAS 12-SEP-2020

10 KING ACCOUNTING NEW YORK 12-SEP-2020

30 TURNER SALES CHICAGO 12-SEP-2020

20 ADAMS RESEARCH DALLAS 12-SEP-2020

30 JAMES SALES CHICAGO 12-SEP-2020

20 FORD RESEARCH DALLAS 12-SEP-2020

10 MILLER ACCOUNTING NEW YORK 12-SEP-2020

**Discussion**

The key to the solution is to use the object's constructor function (by
default the constructor function has the same name as the object).
Because the object itself is a single scalar value, it does not violate
the scalar subquery rule, as you can see from the following:

**select e.deptno,**

**e.ename,**

**e.sal,**

**(select generic_obj(d.dname,d.loc,sysdate-1)**

**from dept d**

**where e.deptno=d.deptno) multival**

**from emp e**

DEPTNO ENAME SAL MULTIVAL(VAL1, VAL2, VAL3)

\-\-\-\-\-- \-\-\-\-\-- \-\-\-\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

20 SMITH 800 GENERIC_OBJ(\'RESEARCH\', \'DALLAS\', \'12-SEP-2020\')

30 ALLEN 1600 GENERIC_OBJ(\'SALES\', \'CHICAGO\', \'12-SEP-2020\')

30 WARD 1250 GENERIC_OBJ(\'SALES\', \'CHICAGO\', \'12-SEP-2020\')

20 JONES 2975 GENERIC_OBJ(\'RESEARCH\', \'DALLAS\', \'12-SEP-2020\')

30 MARTIN 1250 GENERIC_OBJ(\'SALES\', \'CHICAGO\', \'12-SEP-2020\')

30 BLAKE 2850 GENERIC_OBJ(\'SALES\', \'CHICAGO\', \'12-SEP-2020\')

10 CLARK 2450 GENERIC_OBJ(\'ACCOUNTING\', \'NEW YORK\', \'12-SEP-2020\')

20 SCOTT 3000 GENERIC_OBJ(\'RESEARCH\', \'DALLAS\', \'12-SEP-2020\')

10 KING 5000 GENERIC_OBJ(\'ACCOUNTING\', \'NEW YORK\', \'12-SEP-2020\')

30 TURNER 1500 GENERIC_OBJ(\'SALES\', \'CHICAGO\', \'12-SEP-2020\')

20 ADAMS 1100 GENERIC_OBJ(\'RESEARCH\', \'DALLAS\', \'12-SEP-2020\')

30 JAMES 950 GENERIC_OBJ(\'SALES\', \'CHICAGO\', \'12-SEP-2020\')

20 FORD 3000 GENERIC_OBJ(\'RESEARCH\', \'DALLAS\', \'12-SEP-2020\')

10 MILLER 1300 GENERIC_OBJ(\'ACCOUNTING\', \'NEW YORK\',
\'12-SEP-2020\')

The next step is to simply wrap the query in an inline view and extract
the attributes.

**Warning**

In Oracle, unlike the case with other vendors, you do not generally need
to name your inline views. In this particular case, however, you do need
to name your inline view. Otherwise, you will not be able to reference
the object's attributes.

**14.11 Parsing Serialized Data into Rows**

**Problem**

You have serialized data (stored in strings) that you want to parse and
return as rows. For example, you store the following data:

STRINGS

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

entry:stewiegriffin:lois:brian:

entry:moe::sizlack:

entry:petergriffin:meg:chris:

entry:willie:

entry:quagmire:mayorwest:cleveland:

entry:::flanders:

entry:robo:tchi:ken:

You want to convert these serialized strings into the following result
set:

VAL1 VAL2 VAL3

\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\--

moe sizlack

petergriffin meg chris

quagmire mayorwest cleveland

robo tchi ken

stewiegriffin lois brian

willie

flanders

**Solution**

Each serialized string in this example can store up to three values. The
values are delimited by colons, and a string may or may not have all
three entries. If a string does not have all three entries, you must be
careful to place the entries that are available into the correct column
in the result set. For example, consider the following row:

entry:::flanders:

This row represents an entry with the first two values missing and only
the third value available. Hence, if you examine the target result set
in the "Problem" section, you will notice that for the row FLANDERS is
in, both VAL1 and VAL2 are NULL.

The key to this solution is nothing more than a string walk with some
string parsing, following by a simple pivot. This solution uses rows
from view V, which is defined as follows. The example uses Oracle
syntax, but since nothing more than string parsing functions are needed
for this recipe, converting to other platforms is simple:

create view V

as

select \'entry:stewiegriffin:lois:brian:\' strings

from dual

union all

select \'entry:moe::sizlack:\'

from dual

union all

select \'entry:petergriffin:meg:chris:\'

from dual

union all

select \'entry:willie:\'

from dual

union all

select \'entry:quagmire:mayorwest:cleveland:\'

from dual

union all

select \'entry:::flanders:\'

from dual

union all

select \'entry:robo:tchi:ken:\'

from dual

Using view V to supply the example data to parse, the solution is as
follows:

1 with cartesian as (

2 select level id

3 from dual

4 connect by level \<= 100

5 )

6 select max(decode(id,1,substr(strings,p1+1,p2-1))) val1,

7 max(decode(id,2,substr(strings,p1+1,p2-1))) val2,

8 max(decode(id,3,substr(strings,p1+1,p2-1))) val3

9 from (

10 select v.strings,

11 c.id,

12 instr(v.strings,\':\',1,c.id) p1,

13 instr(v.strings,\':\',1,c.id+1)-instr(v.strings,\':\',1,c.id) p2

14 from v, cartesian c

15 where c.id \<= (length(v.strings)-length(replace(v.strings,\':\')))-1

16 )

17 group by strings

18 order by 1

**Discussion**

The first step is to walk the serialized strings:

**with cartesian as (**

**select level id**

**from dual**

**connect by level \<= 100**

**)**

**select v.strings,**

**c.id**

**from v,cartesian c**

**where c.id \<=
(length(v.strings)-length(replace(v.strings,\':\')))-1**

STRINGS ID

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
\-\--

entry:::flanders: 1

entry:::flanders: 2

entry:::flanders: 3

entry:moe::sizlack: 1

entry:moe::sizlack: 2

entry:moe::sizlack: 3

entry:petergriffin:meg:chris: 1

entry:petergriffin:meg:chris: 3

entry:petergriffin:meg:chris: 2

entry:quagmire:mayorwest:cleveland: 1

entry:quagmire:mayorwest:cleveland: 3

entry:quagmire:mayorwest:cleveland: 2

entry:robo:tchi:ken: 1

entry:robo:tchi:ken: 2

entry:robo:tchi:ken: 3

entry:stewiegriffin:lois:brian: 1

entry:stewiegriffin:lois:brian: 3

entry:stewiegriffin:lois:brian: 2

entry:willie: 1

The next step is to use the function INSTR to find the numeric position
of each colon in each string. Since each value you need to extract is
enclosed by two colons, the numeric values are aliased P1 and P2, for
"position one" and "position two":

**with cartesian as (**

**select level id**

**from dual**

**connect by level \<= 100**

**)**

**select v.strings,**

**c.id,**

**instr(v.strings,\':\',1,c.id) p1,**

**instr(v.strings,\':\',1,c.id+1)-instr(v.strings,\':\',1,c.id) p2**

**from v,cartesian c**

**where c.id \<=
(length(v.strings)-length(replace(v.strings,\':\')))-1**

**order by 1**

STRINGS ID P1 P2

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

entry:::flanders: 1 6 1

entry:::flanders: 2 7 1

entry:::flanders: 3 8 9

entry:moe::sizlack: 1 6 4

entry:moe::sizlack: 2 10 1

entry:moe::sizlack: 3 11 8

entry:petergriffin:meg:chris: 1 6 13

entry:petergriffin:meg:chris: 3 23 6

entry:petergriffin:meg:chris: 2 19 4

entry:quagmire:mayorwest:cleveland: 1 6 9

entry:quagmire:mayorwest:cleveland: 3 25 10

entry:quagmire:mayorwest:cleveland: 2 15 10

entry:robo:tchi:ken: 1 6 5

entry:robo:tchi:ken: 2 11 5

entry:robo:tchi:ken: 3 16 4

entry:stewiegriffin:lois:brian: 1 6 14

entry:stewiegriffin:lois:brian: 3 25 6

entry:stewiegriffin:lois:brian: 2 20 5

entry:willie: 1 6 7

Now that you know the numeric positions for each pair of colons in each
string, simply pass the information to the function SUBSTR to extract
values. Since you want to create a result set with three columns, use
DECODE to evaluate the ID from the Cartesian product:

**with cartesian as (**

**select level id**

**from dual**

**connect by level \<= 100**

**)**

**select decode(id,1,substr(strings,p1+1,p2-1)) val1,**

**decode(id,2,substr(strings,p1+1,p2-1)) val2,**

**decode(id,3,substr(strings,p1+1,p2-1)) val3**

**from (**

**select v.strings,**

**c.id,**

**instr(v.strings,\':\',1,c.id) p1,**

**instr(v.strings,\':\',1,c.id+1)-instr(v.strings,\':\',1,c.id) p2**

**from v,cartesian c**

**where c.id \<=
(length(v.strings)-length(replace(v.strings,\':\')))-1**

**)**

**order by 1**

VAL1 VAL2 VAL3

\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-\-\-\--

moe

petergriffin

quagmire

robo

stewiegriffin

willie

lois

meg

mayorwest

tchi

brian

sizlack

chris

cleveland

flanders

ken

The last step is to apply an aggregate function to the values returned
by SUBSTR while grouping by ID, to make a human-readable result set:

**with cartesian as (**

**select level id**

**from dual**

**connect by level \<= 100**

**)**

**select max(decode(id,1,substr(strings,p1+1,p2-1))) val1,**

**max(decode(id,2,substr(strings,p1+1,p2-1))) val2,**

**max(decode(id,3,substr(strings,p1+1,p2-1))) val3**

**from (**

**select v.strings,**

**c.id,**

**instr(v.strings,\':\',1,c.id) p1,**

**instr(v.strings,\':\',1,c.id+1)-instr(v.strings,\':\',1,c.id) p2**

**from v,cartesian c**

**where c.id \<=
(length(v.strings)-length(replace(v.strings,\':\')))-1**

**)**

**group by strings**

**order by 1**

VAL1 VAL2 VAL3

\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\--

moe sizlack

petergriffin meg chris

quagmire mayorwest cleveland

robo tchi ken

stewiegriffin lois brian

willie

flanders

**14.12 Calculating Percent Relative to Total**

**Problem**

You want to report a set of numeric values, and you want to show each
value as a percentage of the whole. For example, you are on an Oracle
system and you want to return a result set that shows the breakdown of
salaries by JOB so that you can determine which JOB position costs the
company the most money. You also want to include the number of employees
per JOB to prevent the results from being misleading. You want to
produce the following report:

JOB NUM_EMPS PCT_OF_ALL_SALARIES

\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

CLERK 4 14

ANALYST 2 20

MANAGER 3 28

SALESMAN 4 19

PRESIDENT 1 17

As you can see, if the number of employees is not included in the
report, it looks as if the president position takes very little of the
overall salary. Seeing that there is only one president helps put into
perspective what that 17% means.

**Solution**

Only Oracle enables a decent solution to this problem, which involves
using the built-in function RATIO_TO_REPORT. To calculate percentages of
the whole for other databases, you can use division as shown in [[Recipe
7.11]{.underline}](mhtml:file://C:\Users\Buff%20Panda\OneDrive\Documents\SQL%20code\experrt%20tsql-windows-2019\14.%20Odds%20’n’%20Ends%20_%20SQL%20Cookbook,%202nd%20Edition.mhtml!https://learning.oreilly.com/library/view/sql-cookbook-2nd/9781492077435/ch07.html#sqlckbk-CHP-7-SECT-11):

1 select job,num_emps,sum(round(pct)) pct_of_all_salaries

2 from (

3 select job,

4 count(\*)over(partition by job) num_emps,

5 ratio_to_report(sal)over()\*100 pct

6 from emp

7 )

8 group by job,num_emps

**Discussion**

The first step is to use the window function COUNT OVER to return the
number of employees per JOB. Then use RATIO_TO_REPORT to return the
percentage each salary counts against the total (the value is returned
in decimal):

**select job,**

**count(\*)over(partition by job) num_emps,**

**ratio_to_report(sal)over()\*100 pct**

**from emp**

JOB NUM_EMPS PCT

\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--

ANALYST 2 10.3359173

ANALYST 2 10.3359173

CLERK 4 2.75624462

CLERK 4 3.78983635

CLERK 4 4.4788975

CLERK 4 3.27304048

MANAGER 3 10.2497847

MANAGER 3 8.44099914

MANAGER 3 9.81912145

PRESIDENT 1 17.2265289

SALESMAN 4 5.51248923

SALESMAN 4 4.30663221

SALESMAN 4 5.16795866

SALESMAN 4 4.30663221

The last step is to use the aggregate function SUM to sum the values
returned by RATIO_TO_REPORT. Be sure to group by JOB and NUM_EMPS.
Multiply by 100 to return a whole number that represents a percentage
(e.g., to return 25 rather than 0.25 for 25%):

**select job,num_emps,sum(round(pct)) pct_of_all_salaries**

**from (**

**select job,**

**count(\*)over(partition by job) num_emps,**

**ratio_to_report(sal)over()\*100 pct**

**from emp**

**)**

**group by job,num_emps**

JOB NUM_EMPS PCT_OF_ALL_SALARIES

\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

CLERK 4 14

ANALYST 2 20

MANAGER 3 28

SALESMAN 4 19

PRESIDENT 1 17

**14.13 Testing for Existence of a Value Within a Group**

**Problem**

You want to create a Boolean flag for a row depending on whether any row
in its group contains a specific value. Consider an example of a student
who has taken a certain number of exams during a period of time. A
student will take three exams over three months. If a student passes one
of these exams, the requirement is satisfied and a flag should be
returned to express that fact. If a student did not pass any of the
three tests in the three-month period, then an additional flag should be
returned to express that fact as well. Consider the following example
(using Oracle syntax to make up rows for this example; minor
modifications are necessary for the other vendors, making user of window
functions):

create view V

as

select 1 student_id,

1 test_id,

2 grade_id,

1 period_id,

to_date(\'02/01/2020\',\'MM/DD/YYYY\') test_date,

0 pass_fail

from dual union all

select 1, 2, 2, 1, to_date(\'03/01/2020\',\'MM/DD/YYYY\'), 1 from dual
union all

select 1, 3, 2, 1, to_date(\'04/01/2020\',\'MM/DD/YYYY\'), 0 from dual
union all

select 1, 4, 2, 2, to_date(\'05/01/2020\',\'MM/DD/YYYY\'), 0 from dual
union all

select 1, 5, 2, 2, to_date(\'06/01/2020\',\'MM/DD/YYYY\'), 0 from dual
union all

select 1, 6, 2, 2, to_date(\'07/01/2020\',\'MM/DD/YYYY\'), 0 from dual

select \*

from V

STUDENT_ID TEST_ID GRADE_ID PERIOD_ID TEST_DATE PASS_FAIL

\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-- \-\-\-\-\-\-\-- \-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\--

1 1 2 1 01-FEB-2020 0

1 2 2 1 01-MAR-2020 1

1 3 2 1 01-APR-2020 0

1 4 2 2 01-MAY-2020 0

1 5 2 2 01-JUN-2020 0

1 6 2 2 01-JUL-2020 0

Examining the previous result set, you see that the student has taken
six tests over two, three-month periods. The student has passed one test
(1 means "pass"; 0 means "fail"); thus, the requirement is satisfied for
the entire first period. Because the student did not pass any exams
during the second period (the next three months), PASS_FAIL is 0 for all
three exams. You want to return a result set that highlights whether a
student has passed a test for a given period. Ultimately you want to
return the following result set:

STUDENT_ID TEST_ID GRADE_ID PERIOD_ID TEST_DATE METREQ IN_PROGRESS

\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-- \-\-\-\-\-\-\-- \-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-- \-\-\-\-\-\-\-\-\-\--

1 1 2 1 01-FEB-2020 + 0

1 2 2 1 01-MAR-2020 + 0

1 3 2 1 01-APR-2020 + 0

1 4 2 2 01-MAY-2020 - 0

1 5 2 2 01-JUN-2020 - 0

1 6 2 2 01-JUL-2020 - 1

The values for METREQ ("met requirement") are + and --, signifying the
student either has or has not satisfied the requirement of passing at
least one test in a period (three-month span), respectively. The value
for IN_PROGRESS should be 0 if a student has already passed a test in a
given period. If a student has not passed a test for a given period,
then the row that has the latest exam date for that student will have a
value of 1 for IN_PROGRESS.

**Solution**

This problem appears tricky because you have to treat rows in a group as
a group and not as individuals. Consider the values for PASS_FAIL in the
"Problem" section. If you evaluate row by row, it appears that the value
for METREQ for each row except TEST_ID 2 should be --, when it's not the
case. You must ensure you evaluate the rows as a group. By using the
window function MAX OVER, you can easily determine whether a student
passed at least one test during a particular period. Once you have that
information, the "Boolean" values are a simple matter of using CASE
expressions:

1 select student_id,

2 test_id,

3 grade_id,

4 period_id,

5 test_date,

6 decode( grp_p_f,1,lpad(\'+\',6),lpad(\'-\',6) ) metreq,

7 decode( grp_p_f,1,0,

8 decode( test_date,last_test,1,0 ) ) in_progress

9 from (

10 select V.\*,

11 max(pass_fail)over(partition by

12 student_id,grade_id,period_id) grp_p_f,

13 max(test_date)over(partition by

14 student_id,grade_id,period_id) last_test

15 from V

16 ) x

**Discussion**

The key to the solution is using the window function MAX OVER to return
the greatest value of PASS_FAIL for each group. Because the values for
PASS_FAIL are only 1 or 0, if a student passed at least one exam, then
MAX OVER would return 1 for the entire group. How this works is shown
here:

select V.\*,

max(pass_fail)over(partition by

student_id,grade_id,period_id) grp_pass_fail

from V

STUDENT_ID TEST_ID GRADE_ID PERIOD_ID TEST_DATE PASS_FAIL GRP_PASS_FAIL

\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-- \-\-\-\-\-\-\-- \-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\--

1 1 2 1 01-FEB-2020 0 1

1 2 2 1 01-MAR-2020 1 1

1 3 2 1 01-APR-2020 0 1

1 4 2 2 01-MAY-2020 0 0

1 5 2 2 01-JUN-2020 0 0

1 6 2 2 01-JUL-2020 0 0

The previous result set shows that the student passed at least one test
during the first period; thus, the entire group has a value of 1 or
"pass." The next requirement is that if the student has not passed any
tests in a period, return a value of 1 for the IN\_ PROGRESS flag for
the latest test date in that group. You can use the window function MAX
OVER to do this as well:

select V.\*,

max(pass_fail)over(partition by

student_id,grade_id,period_id) grp_p_f,

max(test_date)over(partition by

student_id,grade_id,period_id) last_test

from V

STUDENT_ID TEST_ID GRADE_ID PERIOD_ID TEST_DATE PASS_FAIL GRP_P_F
LAST_TEST

\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-- \-\-\-\-\-\-\-- \-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-- \-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\--

1 1 2 1 01-FEB-2020 0 1 01-APR-2020

1 2 2 1 01-MAR-2020 1 1 01-APR-2020

1 3 2 1 01-APR-2020 0 1 01-APR-2020

1 4 2 2 01-MAY-2020 0 0 01-JUL-2020

1 5 2 2 01-JUN-2020 0 0 01-JUL-2020

1 6 2 2 01-JUL-2020 0 0 01-JUL-2020

Now that you have determined for which period the student has passed a
test and what the latest test date for each period is, the last step is
simply a matter of applying some formatting magic to make the result set
look nice. The ultimate solution uses Oracle's DECODE function (CASE
supporters, eat your hearts out) to create the METREQ and IN_PROGRESS
columns. Use the LPAD function to right justify the values for METREQ:

select student_id,

test_id,

grade_id,

period_id,

test_date,

decode( grp_p_f,1,lpad(\'+\',6),lpad(\'-\',6) ) metreq,

decode( grp_p_f,1,0,

decode( test_date,last_test,1,0 ) ) in_progress

from (

select V.\*,

max(pass_fail)over(partition by

student_id,grade_id,period_id) grp_p_f,

max(test_date)over(partition by

student_id,grade_id,period_id) last_test

from V

) x

STUDENT_ID TEST_ID GRADE_ID PERIOD_ID TEST_DATE METREQ IN_PROGRESS

\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-- \-\-\-\-\-\-\-- \-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-- \-\-\-\-\-\-\-\-\-\--

1 1 2 1 01-FEB-2020 + 0

1 2 2 1 01-MAR-2020 + 0

1 3 2 1 01-APR-2020 + 0

1 4 2 2 01-MAY-2020 - 0

1 5 2 2 01-JUN-2020 - 0

1 6 2 2 01-JUL-2020 - 1

**14.14 Summing Up**

SQL is more powerful than many credit it. Throughout this book we have
tried to challenge you to see more applications than are typically
noted. In this chapter, we've headed straight for the edge cases and
tried to show just how you can push SQL, both with standard features and
with certain vendor-specific features.

table of contentssearch

Settings

Table of contents collapsed
