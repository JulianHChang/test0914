# 41.Pandas Tutorial

Pandas is a Python library.

Pandas is used to analyze data.

**Learning by Reading**

We have created 14 tutorial pages for you to learn more about Pandas.

Starting with a basic introduction and ends up with cleaning and
plotting data:

## Basic

- Introduction

- Getting Started

- Pandas Series

- DataFrames

- Read CSV

- Read JSON

- Analyze Data

## Cleaning Data

- Clean Data

- Clean Empty Cells

- Clean Wrong Format

- Clean Wrong Data

- Remove Duplicates

## Advanced

- Correlations

- Plotting

# Pandas Introduction

**What is Pandas?**

Pandas is a Python library used for working with data sets.

It has functions for analyzing, cleaning, exploring, and manipulating
data.

The name \"Pandas\" has a reference to both \"Panel Data\", and \"Python
Data Analysis\" and was created by Wes McKinney in 2008.

**Why Use Pandas?**

Pandas allows us to analyze big data and make conclusions based on
statistical theories.

Pandas can clean messy data sets, and make them readable and relevant.

Relevant data is very important in data science.

**Data Science:** is a branch of computer science where we study how to
store, use and analyze data for deriving information from it.

**What Can Pandas Do?**

Pandas gives you answers about the data. Like:

- Is there a correlation between two or more columns?

- What is the average value?

- Max value?

- Min value?

Pandas are also able to delete rows that are not relevant, or contain
wrong values, like empty or NULL values. This is called *cleaning* the
data.

**Where is the Pandas Codebase?**

The source code for Pandas is located at this github repository
[[https://github.com/pandas-dev/pandas]{.underline}](https://github.com/pandas-dev/pandas)
**github:** enables many people to work on the same codebase.

# Pandas Getting Started

**Installation of Pandas**

If you have Python and PIP already installed on a system, then
installation of Pandas is very easy.

Install it using this command:

*C:\\Users\\Your Name\>pip install pandas*

If this command fails, then use a python distribution that already has
Pandas installed like Anaconda, Spyder etc.

**Import Pandas**

Once Pandas is installed, import it in your applications by adding the
import keyword:

**import pandas**

Now Pandas are imported and ready to use. **Example import pandas
mydataset = {**

> **\'cars\': \[\"BMW\", \"Volvo\", \"Ford\"\],**

## \'passings\': \[3, 7, 2\]

**} myvar = pandas.DataFrame(mydataset) print(myvar)**

**Pandas as pd**

Pandas is usually imported under the pd alias.

**alias:** In Python aliases are an alternate name for referring to the
same thing.

Create an alias with the as keyword while importing:

**import pandas as pd**

Now the Pandas package can be referred to as pd instead of pandas.
**Example import pandas as pd mydataset = {**

> **\'cars\': \[\"BMW\", \"Volvo\", \"Ford\"\],**

## \'passings\': \[3, 7, 2\]

**} myvar = pd.DataFrame(mydataset) print(myvar)**

**Checking Pandas Version**

The version string is stored under \_\_version\_\_ attribute.

**Example**

**import pandas as pd print(pd.\_\_version\_\_)**

# Pandas Series

**What is a Series?**

A Pandas Series is like a column in a table.

It is a one-dimensional array holding data of any type.

**Example**

Create a simple Pandas Series from a list:

**import pandas as pd**

**a = \[1, 7, 2\] myvar = pd.Series(a) print(myvar)**

**Labels**

If nothing else is specified, the values are labeled with their index
number. First value has index 0, second value has index 1 etc.

This label can be used to access a specified value.

**Example**

Return the first value of the Series:

**print(myvar\[0\])**

**Create Labels**

With the index argument, you can name your own labels.

**Example**

Create your own labels: **import pandas as pd**

**a = \[1, 7, 2\] myvar = pd.Series(a, index = \[\"x\", \"y\", \"z\"\])
print(myvar)**

When you have created labels, you can access an item by referring to the
label.

**Example**

Return the value of \"y\": **print(myvar\[\"y\"\])**

**Key/Value Objects as Series**

You can also use a key/value object, like a dictionary, when creating a
Series.

**Example**

Create a simple Pandas Series from a dictionary:

**import pandas as pd calories = {\"day1\": 420, \"day2\": 380,
\"day3\": 390} myvar = pd.Series(calories) print(myvar)**

**Note:** The keys of the dictionary become the labels.

To select only some of the items in the dictionary, use the index
argument and specify only the items you want to include in the Series.

**Example**

Create a Series using only data from \"day1\" and \"day2\":

**import pandas as pd calories = {\"day1\": 420, \"day2\": 380,
\"day3\": 390} myvar = pd.Series(calories, index = \[\"day1\",
\"day2\"\]) print(myvar)**

**DataFrames**

Datasets in Pandas are usually multi-dimensional tables, called
DataFrames.

Series is like a column, a DataFrame is the whole table.

**Example**

Create a DataFrame from two Series:

**import pandas as pd data = {**

> **\"calories\": \[420, 380, 390\],**

## \"duration\": \[50, 40, 45\]

**} myvar = pd.DataFrame(data) print(myvar)**

# Pandas DataFrames

**What is a DataFrame?**

A Pandas DataFrame is a 2 dimensional data structure, like a 2
dimensional array, or a table with rows and columns.

**Example**

Create a simple Pandas DataFrame:

**import pandas as pd data = {**

> **\"calories\": \[420, 380, 390\],**

## \"duration\": \[50, 40, 45\]

**}**

**#load data into a DataFrame object:**

**df = pd.DataFrame(data) print(df)**

**Result**

> *calories 420 duration 50*
>
> *Name: 0, dtype: int64*

**Locate Row**

As you can see from the result above, the DataFrame is like a table with
rows and columns.

Pandas use the loc attribute to return one or more specified row(s)

**Example**

Return row 0:

#refer to the row index:

print(df.loc\[0\])

**Result**

> *calories duration*

1.  *420 50*

2.  *380 40*

\\**Note:** This example returns a Pandas **Series**.

**Example**

Return row 0 and 1: **#use a list of indexes: print(df.loc\[\[0,
1\]\])**

**Result**

> *calories duration*
>
> *day1 420 50 day2 380 40 day3 390 45*

**Note:** When using \[\], the result is a Pandas **DataFrame**.

**Locate Named Indexes**

Use the named index in the loc attribute to return the specified row(s).

**Example**

Return \"day2\":

**#refer to the named index: print(df.loc\[\"day2\"\])**

**Result**

*calories 380*

> *duration 40*
>
> *Name: 0, dtype: int64*

**Load Files Into a DataFrame**

If your data sets are stored in a file, Pandas can load them into a
DataFrame.

**Example**

Load a comma separated file (CSV file) into a DataFrame:

**import pandas as pd df = pd.read_csv(\'data.csv\') print(df)**

# Pandas Read CSV

**Read CSV Files**

A simple way to store big data sets is to use CSV files (comma separated
files).

CSV files contain plain text and is a well known format that can be read
by everyone including Pandas.

In our examples we will be using a CSV file called \'data.csv\'.

**Download data.csv.** or **Open data.csv**

**Example**

Load the CSV into a DataFrame: **import pandas as pd**

**df = pd.read_csv(\'data.csv\') print(df.to_string())**

**Tip:** use to_string() to print the entire DataFrame.

By default, when you print a DataFrame, you will only get the first 5
rows, and the last 5 rows:

**Example**

Print a reduced sample: **import pandas as pd df =
pd.read_csv(\'data.csv\') print(df)**

# Pandas Read JSON

**Read JSON**

Big data sets are often stored, or extracted as JSON.

JSON is plain text, but has the format of an object, and is well known
in the world of programming, including Pandas.

In our examples we will be using a JSON file called \'data.json\'.

Open data.json.

**Example**

Load the JSON file into a DataFrame:

**import pandas as pd df = pd.read_json(\'data.json\')
print(df.to_string())**

**Tip:** use to_string() to print the entire DataFrame.

**Dictionary as JSON**

**JSON = Python Dictionary**

JSON objects have the same format as Python dictionaries.

If your JSON code is not in a file, but in a Python Dictionary, you can
load it into a DataFrame directly:

**Example**

Load a Python Dictionary into a DataFrame:

**import pandas as pd**

## data = { \"Duration\":{\"0\":60,\"1\":60,\"2\":60, \"3\":45,\"4\":45,\"5\":60 }, \"Pulse\":{\"0\":110,\"1\":117,\"2\":103,\"3\":109,\"4\":117,\"5\":102 }, \"Maxpulse\":{\"0\":130,\"1\":145,\"2\":135,\"3\":175,\"4\":148,\"5\":127 }, \"Calories\":{\"0\":409,\"1\":479,\"2\":340,\"3\":282,\"4\":406,\"5\":300}

**}**

**df = pd.DataFrame(data) print(df)**

# Pandas - Analyzing DataFrames

**Viewing the Data**

One of the most used methods for getting a quick overview of the
DataFrame, is the head() method.

The head() method returns the headers and a specified number of rows,
starting from the top.

**Example**

Get a quick overview by printing the first 10 rows of the DataFrame:

**import pandas as pd df = pd.read_csv(\'data.csv\')
print(df.head(10))**

In our examples we will be using a CSV file called \'data.csv\'.

Download data.csv, or open data.csv in your browser.

**Note:** if the number of rows is not specified, the head() method will
return the top 5 rows.

**Example**

Print the first 5 rows of the DataFrame:

**import pandas as pd df = pd.read_csv(\'data.csv\') print(df.head())**

There is also a tail() method for viewing the *last* rows of the
DataFrame.

The tail() method returns the headers and a specified number of rows,
starting from the bottom.

**Example**

Print the last 5 rows of the DataFrame: **print(df.tail())**

**Info About the Data**

The DataFrames object has a method called info(), that gives you more
information about the data set.

**Example**

Print information about the data: **print(df.info())**

**Result**

*\<class \'pandas.core.frame.DataFrame\'\> RangeIndex: 169 entries, 0 to
168 Data columns (total 4 columns):*

*\# Column Non-Null Count Dtype*

*\-\-- \-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\--*

1.  *Duration 169 non-nullint64*

2.  *Pulse 169 non-null int64*

3.  *Maxpulse 169 non-null int64*

4.  *Calories 164 non-null float64 dtypes: float64(1), int64(3) memory
    usage: 5.4 KB None*

**Result Explained**

The result tells us there are 169 rows and 4 columns:

> *RangeIndex: 169 entries, 0 to 168 Data columns (total 4 columns):*

*And the name of each column, with the data type:*

> *\# Column Non-Null Count Dtype*

*\-\-- \-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\--*

1.  *Duration 169 non-null int64*

2.  *Pulse 169 non-null int64*

3.  *Maxpulse 169 non-null int64*

4.  *Calories 164 non-null float64*

**Null Values**

The info() method also tells us how many Non-Null values there are
present in each column, and in our data set it seems like there are 164
of 169 NonNull values in the \"Calories\" column.

Which means that there are 5 rows with no value at all, in the
\"Calories\" column, for whatever reason.

Empty values, or Null values, can be bad when analyzing data, and you
should consider removing rows with empty values. This is a step towards
what is called *cleaning data.*

# Pandas - Cleaning Data

Data cleaning means fixing bad data in your data set.

Bad data could be:

- Empty cells

- Data in wrong format

- Wrong data

- Duplicates

In this tutorial you will learn how to deal with all of them.

**Our Data Set**

In the next chapters we will use this data set:

*Duration Date Pulse Max Pulse Calories*

1.  *60 \'2020/12/01\' 110 130 409.1*

2.  *60 \'2020/12/02\' 117 145 479.0*

3.  *60 \'2020/12/03\' 103 135 340.0*

4.  *45 \'2020/12/04\' 109 175 282.4*

5.  *45 \'2020/12/05\' 117 148 406.0*

6.  *60 \'2020/12/06\' 102 127 300.0*

7.  *60 \'2020/12/07\' 110 136 374.0*

8.  *450 \'2020/12/08\' 104 134 253.3*

9.  *30 \'2020/12/09\' 109 133 195.1*

10. *60 \'2020/12/10\' 98 124 269.0*

11. *60 \'2020/12/11\' 103 147 329.3*

12. *60 \'2020/12/12\' 100 120 250.7*

13. *60 \'2020/12/12\' 100 120 250.7*

14. *60 \'2020/12/13\' 106 128 345.3*

15. *60 \'2020/12/14\' 104 132 379.3*

16. *60 \'2020/12/15\' 98 123 275.0*

17. *60 \'2020/12/16\' 98 120 215.2*

18. *60 \'2020/12/17\' 100 120 300.0*

19. *45 \'2020/12/18\' 90 112 NaN*

20. *60 \'2020/12/19\' 103 123 323.0*

21. *45 \'2020/12/20\' 97 125 243.0*

22. *60 \'2020/12/21\' 108 131 364.2*

23. *45 NaN 100 119 282.0*

24. *60 \'2020/12/23\' 130 101 300.0*

25. *45 \'2020/12/24\' 105 132 246.0*

26. *60 \'2020/12/25\' 102 126 334.5*

27. *60 2020/12/26 100 120 250.0*

28. *60 \'2020/12/27\' 92 118 241.0*

29. *60 \'2020/12/28\' 103 132 NaN*

30. *60 \'2020/12/29\' 100 132 280.0*

31. *60 \'2020/12/30\' 102 129 380.3*

32. *60 \'2020/12/31\' 92 115*

The data set contains some empty cells (\"Date\" in row 22, and
\"Calories\" in row 18 and 28).

The data set contains the wrong format (\"Date\" in row 26).

The data set contains wrong data (\"Duration\" in row 7).

The data set contains duplicates (row 11 and 12).

# Pandas - Cleaning Empty Cells

**Empty Cells**

Empty cells can potentially give you a wrong result when you analyze
data.

**Remove Rows**

One way to deal with empty cells is to remove rows that contain empty
cells.

This is usually OK, since data sets can be very big, and removing a few
rows will not have a big impact on the result.

**Example**

Return a new Dataframe with no empty cells:

**import pandas as pd df = pd.read_csv(\'data.csv\') new_df =
df.dropna() print(new_df.to_string())**

In our cleaning examples we will be using a CSV file called
\'dirtydata.csv\'.

Download dirtydata.csv. or Open dirtydata.csv

**Note:** By default, the dropna() method returns a *new* DataFrame, and
will not change the original.

If you want to change the original DataFrame, use the inplace = True
argument:

**Example**

Remove all rows with NULL values:

**import pandas as pd**

**df = pd.read_csv(\'data.csv\') df.dropna(inplace = True)
print(df.to_string())**

**Note:** Now, the dropna(inplace = True) will NOT return a new
DataFrame, but it will remove all rows containing NULL values from the
original DataFrame.

**Replace Empty Values**

Another way of dealing with empty cells is to insert a *new* value
instead.

This way you do not have to delete entire rows just because of some
empty cells.

The fillna() method allows us to replace empty cells with a value:

**Example**

Replace NULL values with the number 130:

**import pandas as pd df = pd.read_csv(\'data.csv\') df.fillna(130,
inplace = True)**

**Replace Only For a Specified Columns**

The example above replaces all empty cells in the whole Data Frame.

To only replace empty values for one column, specify the *column name*
for the DataFrame:

**Example**

Replace NULL values in the \"Calories\" columns with the number 130:

**import pandas as pd df = pd.read_csv(\'data.csv\')
df\[\"Calories\"\].fillna(130, inplace = True)**

**Replace Using Mean, Median, or Mode**

A common way to replace empty cells is to calculate the mean, median or
mode value of the column.

Pandas uses the mean() median() and mode() methods to calculate the
respective values for a specified column:

**Example**

Calculate the MEAN, and replace any empty values with it:

**import pandas as pd df = pd.read_csv(\'data.csv\') x =
df\[\"Calories\"\].mean() df\[\"Calories\"\].fillna(x, inplace = True)**

**Mean** = the average value (the sum of all values divided by number of
values).

**Example**

Calculate the MEDIAN, and replace any empty values with it:

**import pandas as pd df = pd.read_csv(\'data.csv\') x =
df\[\"Calories\"\].median() df\[\"Calories\"\].fillna(x, inplace =
True)**

**Median** = the value in the middle, after you have sorted all values
ascending.

**Example**

Calculate the MODE, and replace any empty values with it:

**import pandas as pd df = pd.read_csv(\'data.csv\') x =
df\[\"Calories\"\].mode()\[0\] df\[\"Calories\"\].fillna(x, inplace =
True)**

**Mode** = the value that appears most frequently.

# Pandas - Cleaning Data of Wrong Format

**Data of Wrong Format**

Cells with data of wrong format can make it difficult, or even
impossible, to analyze data.

To fix it, you have two options: remove the rows, or convert all cells
in the columns into the same format.

**Convert Into a Correct Format**

In our Data Frame, we have two cells with the wrong format. Check out
row 22 and 26, the \'Date\' column should be a string that represents a
date:

*Duration Date Pulse Max Pulse Calories*

1.  *60 \'2020/12/01\' 110 130 409.1*

2.  *60 \'2020/12/02\' 117 145 479.0*

3.  *60 \'2020/12/03\' 103 135 340.0*

4.  *45 \'2020/12/04\' 109 175 282.4*

5.  *45 \'2020/12/05\' 117 148 406.0*

6.  *60 \'2020/12/06\' 102 127 300.0*

7.  *60 \'2020/12/07\' 110 136 374.0*

8.  *450 \'2020/12/08\' 104 134 253.3*

9.  *30 \'2020/12/09\' 109 133 195.1*

10. *60 \'2020/12/10\' 98 124 269.0*

11. *60 \'2020/12/11\' 103 147 329.3*

12. *60 \'2020/12/12\' 100 120 250.7*

13. *60 \'2020/12/12\' 100 120 250.7*

14. *60 \'2020/12/13\' 106 128 345.3*

15. *60 \'2020/12/14\' 104 132 379.3*

16. *60 \'2020/12/15\' 98 123 275.0*

17. *60 \'2020/12/16\' 98 120 215.2*

18. *60 \'2020/12/17\' 100 120 300.0*

19. *45 \'2020/12/18\' 90 112 NaN*

20. *60 \'2020/12/19\' 103 123 323.0*

21. *45 \'2020/12/20\' 97 125 243.0*

22. *60 \'2020/12/21\' 108 131 364.2*

23. *45 NaN 100 119 282.0*

24. *60 \'2020/12/23\' 130 101 300.0*

25. *45 \'2020/12/24\' 105 132 246.0*

26. *60 \'2020/12/25\' 102 126 334.5*

27. *60 2020/12/26 100 120 250.0*

28. *60 \'2020/12/27\' 92 118 241.0*

29. *60 \'2020/12/28\' 103 132 NaN*

30. *60 \'2020/12/29\' 100 132 280.0*

31. *60 \'2020/12/30\' 102 129 380.3*

32. *60 \'2020/12/31\' 92 115*

**Let\'s try to convert all cells in the \'Date\' column into dates.**

**Pandas has a to_datetime() method for this:**

**Example**

Convert to date: **import pandas as pd df = pd.read_csv(\'data.csv\')
df\[\'Date\'\] = pd.to_datetime(df\[\'Date\'\]) print(df.to_string())
Result:**

Duration Date Pulse Max Pulse Calories

1.  60 \'2020/12/01\' 110 130 409.1

2.  60 \'2020/12/02\' 117 145 479.0

3.  60 \'2020/12/03\' 103 135 340.0

4.  45 \'2020/12/04\' 109 175 282.4

5.  45 \'2020/12/05\' 117 148 406.0

6.  60 \'2020/12/06\' 102 127 300.0

7.  60 \'2020/12/07\' 110 136 374.0

8.  450 \'2020/12/08\' 104 134 253.3

9.  30 \'2020/12/09\' 109 133 195.1

10. 60 \'2020/12/10\' 98 124 269.0

11. 60 \'2020/12/11\' 103 147 329.3

12. 60 \'2020/12/12\' 100 120 250.7

13. 60 \'2020/12/12\' 100 120 250.7

14. 60 \'2020/12/13\' 106 128 345.3

15. 60 \'2020/12/14\' 104 132 379.3

16. 60 \'2020/12/15\' 98 123 275.0

17. 60 \'2020/12/16\' 98 120 215.2

18. 60 \'2020/12/17\' 100 120 300.0

19. 45 \'2020/12/18\' 90 112 NaN

20. 60 \'2020/12/19\' 103 123 323.0

21. 45 \'2020/12/20\' 97 125 243.0

22. 60 \'2020/12/21\' 108 131 364.2

23. 45 NaT 100 119 282.0

24. 60 \'2020/12/23\' 130 101 300.0

25. 45 \'2020/12/24\' 105 132 246.0

26. 60 \'2020/12/25\' 102 126 334.5

27. 60 \'2020/12/26\' 100 120 250.0

28. 60 \'2020/12/27\' 92 118 241.0

29. 60 \'2020/12/28\' 103 132 NaN

30. 60 \'2020/12/29\' 100 132 280.0

31. 60 \'2020/12/30\' 102 129 380.3

32. 60 \'2020/12/31\' 92 115

As you can see from the result, the date in row 26 was fixed, but the
empty date in row 22 got a NaT (Not a Time) value, in other words an
empty value. One way to deal with empty values is simply removing the
entire row.

**Removing Rows**

The result from the conversion in the example above gave us a NaT value,
which can be handled as a NULL value, and we can remove the row by using
the dropna() method.

**Example**

Remove rows with a NULL value in the \"Date\" column:
**df.dropna(subset=\[\'Date\'\], inplace = True)**

# Pandas - Fixing Wrong Data

**Wrong Data**

\"Wrong data\" does not have to be \"empty cells\" or \"wrong format\",
it can just be wrong, like if someone registered \"199\" instead of
\"1.99\".

Sometimes you can spot wrong data by looking at the data set, because
you have an expectation of what it should be.

If you take a look at our data set, you can see that in row 7, the
duration is 450, but for all the other rows the duration is between 30
and 60.

It doesn\'t have to be wrong, but taking in consideration that this is
the data set of someone\'s workout sessions, we conclude with the fact
that this person did not work out in 450 minutes.

*Duration Date Pulse Max Pulse Calories*

1.  *60 \'2020/12/01\' 110 130 409.1*

2.  *60 \'2020/12/02\' 117 145 479.0*

3.  *60 \'2020/12/03\' 103 135 340.0*

4.  *45 \'2020/12/04\' 109 175 282.4*

5.  *45 \'2020/12/05\' 117 148 406.0*

6.  *60 \'2020/12/06\' 102 127 300.0*

7.  *60 \'2020/12/07\' 110 136 374.0*

8.  *450 \'2020/12/08\' 104 134 253.3*

9.  *30 \'2020/12/09\' 109 133 195.1*

10. *60 \'2020/12/10\' 98 124 269.0*

11. *60 \'2020/12/11\' 103 147 329.3*

12. *60 \'2020/12/12\' 100 120 250.7*

13. *60 \'2020/12/12\' 100 120 250.7*

14. *60 \'2020/12/13\' 106 128 345.3*

15. *60 \'2020/12/14\' 104 132 379.3*

16. *60 \'2020/12/15\' 98 123 275.0*

17. *60 \'2020/12/16\' 98 120 215.2*

18. *60 \'2020/12/17\' 100 120 300.0*

19. *45 \'2020/12/18\' 90 112 NaN*

20. *60 \'2020/12/19\' 103 123 323.0*

21. *45 \'2020/12/20\' 97 125 243.0*

22. *60 \'2020/12/21\' 108 131 364.2*

23. *45 NaN 100 119 282.0*

24. *60 \'2020/12/23\' 130 101 300.0*

25. *45 \'2020/12/24\' 105 132 246.0*

26. *60 \'2020/12/25\' 102 126 334.5*

27. *60 2020/12/26 100 120 250.0*

28. *60 \'2020/12/27\' 92 118 241.0*

29. *60 \'2020/12/28\' 103 132 NaN*

30. *60 \'2020/12/29\' 100 132 280.0*

31. *60 \'2020/12/30\' 102 129 380.3*

32. *60 \'2020/12/31\' 92 115*

How can we fix wrong values, like the one for \"Duration\" in row 7?

**Replacing Values**

One way to fix wrong values is to replace them with something else.

In our example, it is most likely a typo, and the value should be \"45\"
instead of \"450\", and we could just insert \"45\" in row 7:

**Example**

**Set \"Duration\" = 45 in row 7:**

**df.loc\[7, \'Duration\'\] = 45**

For small data sets you might be able to replace the wrong data one by
one, but not for big data sets.

To replace wrong data for larger data sets you can create some rules,
e.g. set some boundaries for legal values, and replace any values that
are outside of the boundaries.

**Example**

Loop through all values in the \"Duration\" column.

If the value is higher than 120, set it to 120: **for x in df.index:**

> **if df.loc\[x, \"Duration\"\] \> 120:**
>
> **df.loc\[x, \"Duration\"\] = 120**

**Removing Rows**

Another way of handling wrong data is to remove the rows that contains
wrong data.

This way you do not have to find out what to replace them with, and
there is a good chance you do not need them to do your analyses.

**Example**

Delete rows where \"Duration\" is higher than 120: **for x in
df.index:**

> **if df.loc\[x, \"Duration\"\] \> 120:**
>
> **df.drop(x, inplace = True)**

# Pandas - Removing Duplicates

**Discovering Duplicates**

Duplicate rows are rows that have been registered more than one time.

*Duration Date Pulse Max Pulse Calories*

1.  *60 \'2020/12/01\' 110 130 409.1*

2.  *60 \'2020/12/02\' 117 145 479.0*

3.  *60 \'2020/12/03\' 103 135 340.0*

4.  *45 \'2020/12/04\' 109 175 282.4*

5.  *45 \'2020/12/05\' 117 148 406.0*

6.  *60 \'2020/12/06\' 102 127 300.0*

7.  *60 \'2020/12/07\' 110 136 374.0*

8.  *450 \'2020/12/08\' 104 134 253.3*

9.  *30 \'2020/12/09\' 109 133 195.1*

10. *60 \'2020/12/10\' 98 124 269.0*

11. *60 \'2020/12/11\' 103 147 329.3*

12. *60 \'2020/12/12\' 100 120 250.7*

13. *60 \'2020/12/12\' 100 120 250.7*

14. *60 \'2020/12/13\' 106 128 345.3*

15. *60 \'2020/12/14\' 104 132 379.3*

16. *60 \'2020/12/15\' 98 123 275.0*

17. *60 \'2020/12/16\' 98 120 215.2*

18. *60 \'2020/12/17\' 100 120 300.0*

19. *45 \'2020/12/18\' 90 112 NaN*

20. *60 \'2020/12/19\' 103 123 323.0*

21. *45 \'2020/12/20\' 97 125 243.0*

22. *60 \'2020/12/21\' 108 131 364.2*

23. *45 NaN 100 119 282.0*

24. *60 \'2020/12/23\' 130 101 300.0*

25. *45 \'2020/12/24\' 105 132 246.0*

26. *60 \'2020/12/25\' 102 126 334.5*

27. *60 2020/12/26 100 120 250.0*

28. *60 \'2020/12/27\' 92 118 241.0*

29. *60 \'2020/12/28\' 103 132 NaN*

30. *60 \'2020/12/29\' 100 132 280.0*

31. *60 \'2020/12/30\' 102 129 380.3*

32. *60 \'2020/12/31\' 92 115*

By taking a look at our test data set, we can assume that row 11 and 12
are duplicates.

To discover duplicates, we can use the duplicated() method.

The duplicated() method returns a Boolean values for each row:

**Example**

Returns True for every row that is a duplicate, otherwise False:
**print(df.duplicated())**

**Removing Duplicates**

To remove duplicates, use the drop_duplicates() method.

**Example**

Remove all duplicates:

**df.drop_duplicates(inplace = True)**

**Remember:** The (inplace = True) will make sure that the method does
NOT return a *new* DataFrame, but it will remove all duplicates from the
*original* DataFrame.

# Pandas - Data Correlations

**Finding Relationships**

A great aspect of the Pandas module is the corr() method.

The corr() method calculates the relationship between each column in
your data set.

The examples in this page uses a CSV file called: \'data.csv\'.

Download data.csv. or Open data.csv

**Example**

Show the relationship between the columns:

**df.corr()**

**Result**

> Duration Pulse Max Pulse Calories
>
> Duration 1.000000 -0.155408 0.009403 0.922721
>
> Pulse -0.155408 1.000000 0.786535 0.025120
>
> Maxpulse 0.009403 0.786535 1.000000 0.203814
>
> Calories 0.922721 0.025120 0.203814 1.000000

**Note:** The corr() method ignores \"not numeric\" columns.

**Result Explained**

The Result of the corr() method is a table with a lot of numbers that
represents how well the relationship is between two columns.

The number varies from -1 to 1.

1 means that there is a 1 to 1 relationship (a perfect correlation), and
for this data set, each time a value went up in the first column, the
other one went up as well.

0.9 is also a good relationship, and if you increase one value, the
other will probably increase as well.

-0.9 would be just as good a relationship as 0.9, but if you increase
one value, the other will probably go down.

0.2 means NOT a good relationship, meaning that if one value goes up
does not mean that the other will.

**What is a good correlation?** It depends on the use, but I think it is
safe to say you have to have at least 0.6 (or -0.6) to call it a good
correlation.

**Perfect Correlation:**

We can see that \"Duration\" and \"Duration\" got the number 1.000000,
which makes sense, each column always has a perfect relationship with
itself.

**Good Correlation:**

\"Duration\" and \"Calories\" have a 0.922721 correlation, which is a
very good correlation, and we can predict that the longer you work out,
the more calories you burn, and the other way around: if you burned a
lot of calories, you probably had a long workout.

**Bad Correlation:**

\"Duration\" and \"Maxpulse\" have a 0.009403 correlation, which is a
very bad correlation, meaning that we can not predict the max pulse by
just looking at the duration of the workout, and vice versa.

# Pandas - Plotting

![](media/image1.jpg){width="3.25in" height="2.4270833333333335in"}

**Plotting**

Pandas uses the plot() method to create diagrams.

We can use Pyplot, a submodule of the Matplotlib library to visualize
the diagram on the screen.

Read more about Matplotlib in our Matplotlib Tutorial.

**Example**

Import pyplot from Matplotlib and visualize our DataFrame:

**import pandas as pd import matplotlib.pyplot as plt df =
pd.read_csv(\'data.csv\') df.plot() plt.show()**

The examples in this page uses a CSV file called: \'data.csv\'.

Download data.csv or Open data.csv

**Scatter Plot**

Specify that you want a scatter plot with the kind argument:

kind = \'scatter\'

A scatter plot needs an x- and a y-axis.

In the example below we will use \"Duration\" for the x-axis and
\"Calories\" for the y-axis.

Include the x and y arguments like this:

x = \'Duration\', y = \'Calories\'

**Example import pandas as pd import matplotlib.pyplot as plt df =
pd.read_csv(\'data.csv\') df.plot(kind = \'scatter\', x = \'Duration\',
y = \'Calories\') plt.show()**

**Result**

**Remember:** In the previous example, we learned that the correlation
between \"Duration\" and \"Calories\" was 0.922721, and we concluded
that higher duration means more calories burned.

![](media/image2.jpg){width="3.6145833333333335in"
height="2.6979166666666665in"}

By looking at the scatterplot, I will agree.

Let\'s create another scatterplot, where there is a bad relationship
between the columns, like \"Duration\" and \"Maxpulse\", with the
correlation 0.009403:

**Example**

A scatterplot where there are no relationship between the columns:

**import pandas as pd import matplotlib.pyplot as plt df =
pd.read_csv(\'data.csv\') df.plot(kind = \'scatter\', x = \'Duration\',
y = \'Maxpulse\') plt.show()**

**Result**

![](media/image3.jpg){width="3.25in" height="2.4270833333333335in"}

**Histogram**

Use the kind argument to specify that you want a histogram:

**kind = \'hist\'**

A histogram needs only one column.

A histogram shows us the frequency of each interval, e.g. how many
workouts lasted between 50 and 60 minutes?

In the example below we will use the \"Duration\" column to create the
histogram:

**Example**

**df\[\"Duration\"\].plot(kind = \'hist\')**

**Result**![](media/image4.jpg){width="3.625in"
height="2.7083333333333335in"}

**Note:** The histogram tells us that there were over 100 workouts that
lasted between 50 and 60 minutes.

# 42.SciPy Tutorial

SciPy is a scientific computation library that uses NumPy underneath.

SciPy stands for Scientific Python.

**Learning by Reading**

We have created 10 tutorial for you to learn the fundamentals of SciPy:

- Getting Started

- Constants

- Optimizers

- Sparse Data

- Graphs

- Spatial Data

- Matlab Arrays

- Interpolation

- Significance Tests

# SciPy Introduction

**What is SciPy?**

SciPy is a scientific computation library that uses NumPy underneath.

SciPy stands for Scientific Python.

It provides more utility functions for optimization, stats and signal
processing.

Like NumPy, SciPy is open source so we can use it freely.

SciPy was created by NumPy\'s creator Travis Olliphant.

**Why Use SciPy?**

If SciPy uses NumPy underneath, why can we not just use NumPy?

SciPy has optimized and added functions that are frequently used in
NumPy and Data Science.

**Which Language is SciPy Written in?**

SciPy is predominantly written in Python, but a few segments are written
in C.

**Where is the SciPy Codebase?**

The source code for SciPy is located at this github repository
[[https://github.com/scipy/scipy]{.underline}](https://github.com/scipy/scipy)
**github:** enables many people to work on the same codebase.

# SciPy Getting Started

**Installation of SciPy**

If you have Python and PIP already installed on a system, then
installation of SciPy is very easy.

Install it using this command:

C:\\Users\\*Your Name*\>pip install scipy

If this command fails, then use a Python distribution that already has
SciPy installed like Anaconda, Spyder etc.

**Import SciPy**

Once SciPy is installed, import the SciPy module(s) you want to use in
your applications by adding the from scipy import *module* statement:

from scipy import constants

Now we have imported the *constants* module from SciPy, and the
application is ready to use it:

**Example**

How many cubic meters are in one liter:

**from scipy import constants print(constants.liter)**

**constants:** SciPy offers a set of mathematical constants, one of them
is liter which returns 1 liter as cubic meters.

You will learn more about constants in the next chapter.

**Checking SciPy Version**

The version string is stored under the \_\_version\_\_ attribute.

**Example**

**import scipy print(scipy.\_\_version\_\_)**

**Note:** two underscore characters are used in \_\_version\_\_.

# SciPy Constants

**Constants in SciPy**

As SciPy is more focused on scientific implementations, it provides many
built-in scientific constants.

These constants can be helpful when you are working with Data Science.

PI is an example of a scientific constant.

**Example**

Print the constant value of PI: **from scipy import constants
print(constants.pi)**

**Constant Units**

A list of all units under the constants module can be seen using the
dir() function.

**Example**

List all constants:

**from scipy import constants print(dir(constants))**

**Unit Categories**

The units are placed under these categories:

- Metric

- Binary

- Mass

- Angle

- Time

- Length

- Pressure

- Volume

- Speed

- Temperature

- Energy

- Power

- Force

**Metric (SI) Prefixes:**

Return the specified unit in **meter** (e.g. centi returns 0.01)

**Example**

**from scipy import constants**

+-------------------------------------+--------------------------------+
| **print(constants.yotta)**          | > **#1e+24**                   |
+=====================================+================================+
| **print(constants.zetta)**          | > **#1e+21**                   |
+-------------------------------------+--------------------------------+
| **print(constants.exa)**            | > **#1e+18**                   |
+-------------------------------------+--------------------------------+
| **print(constants.peta)**           | > **#1000000000000000.0**      |
+-------------------------------------+--------------------------------+
| **print(constants.tera)**           | > **#1000000000000.0**         |
+-------------------------------------+--------------------------------+
| **print(constants.giga)**           | **#1000000000.0**              |
+-------------------------------------+--------------------------------+
| **print(constants.mega)**           | **#1000000.0**                 |
+-------------------------------------+--------------------------------+
| **print(constants.kilo)**           | > **#1000.0**                  |
+-------------------------------------+--------------------------------+
| **print(constants.hecto)**          | **#100.0**                     |
+-------------------------------------+--------------------------------+
| **print(constants.deka)**           | > **#10.0**                    |
+-------------------------------------+--------------------------------+
| **print(constants.deci)**           | > **#0.1**                     |
+-------------------------------------+--------------------------------+
| **print(constants.centi)**          | > **#0.01**                    |
+-------------------------------------+--------------------------------+
| **print(constants.milli)**          | > **#0.001**                   |
+-------------------------------------+--------------------------------+
| **print(constants.micro)**          | > **#1e-06**                   |
+-------------------------------------+--------------------------------+
| **print(constants.nano)**           | > **#1e-09**                   |
+-------------------------------------+--------------------------------+
| **print(constants.pico)**           | > **#1e-12**                   |
+-------------------------------------+--------------------------------+
| **print(constants.femto)**          | > **#1e-15**                   |
+-------------------------------------+--------------------------------+
| **print(constants.atto)**           | > **#1e-18**                   |
+-------------------------------------+--------------------------------+
| **print(constants.zepto)**          | > **#1e-21**                   |
+-------------------------------------+--------------------------------+

**Binary Prefixes:**

Return the specified unit in **bytes** (e.g. kibi returns 1024)
**Example**

**from scipy import constants**

**print(constants.kibi) #1024 print(constants.mebi) #1048576
print(constants.gibi) #1073741824**

+----------------------------+-----------------------------------------+
| **print(constants.tebi)**  | > **#1099511627776**                    |
+============================+=========================================+
| **print(constants.pebi)**  | > **#1125899906842624**                 |
+----------------------------+-----------------------------------------+
| **print(constants.exbi)**  | > **#1152921504606846976**              |
+----------------------------+-----------------------------------------+
| **print(constants.zebi)**  | > **#1180591620717411303424**           |
+----------------------------+-----------------------------------------+
| **print(constants.yobi)**  | > **#1208925819614629174706176**        |
+----------------------------+-----------------------------------------+

**Mass:**

Return the specified unit in **kg** (e.g. gram returns 0.001)

**Example**

**from scipy import constants**

**print(constants.gram) #0.001 print(constants.metric_ton) #1000.0
print(constants.grain) #6.479891e-05 print(constants.lb)
#0.45359236999999997 print(constants.pound) #0.45359236999999997
print(constants.oz) #0.028349523124999998 print(constants.ounce)
#0.028349523124999998**

**(constants.stone) #6.3502931799999995 print(constants.long_ton)
#1016.0469088 print(constants.short_ton) #907.1847399999999
print(constants.troy_ounce) #0.031103476799999998
print(constants.troy_pound) #0.37324172159999996 print(constants.carat)
#0.0002 print(constants.atomic_mass) #1.66053904e-27
print(constants.m_u) #1.66053904e-27 print(constants.u)
#1.66053904e-27**

**Angle:**

Return the specified unit in **radians** (e.g. degree returns

0.017453292519943295)

**Example**

**from scipy import constants**

**print(constants.degree) #0.017453292519943295 print(constants.arcmin)
#0.0002908882086657216 print(constants.arcminute) #0.0002908882086657216
print(constants.arcsec) #4.84813681109536e-06 print(constants.arcsecond)
#4.84813681109536e-06**

**Time:**

Return the specified unit in **seconds** (e.g. hour returns 3600.0)

**Example**

**from scipy import constants**

**print(constants.minute) #60.0 print(constants.hour) #3600.0
print(constants.day) #86400.0 print(constants.week) #604800.0
print(constants.year) #31536000.0 print(constants.Julian_year)
#31557600.0**

**Length:**

Return the specified unit in **meters** (e.g. nautical_mile returns
1852.0)

**Example**

**from scipy import constants**

**print(constants.inch) #0.0254**

**print(constants.foot) #0.30479999999999996 print(constants.yard)
#0.9143999999999999 print(constants.mile) #1609.3439999999998
print(constants.mil) #2.5399999999999997e-05 print(constants.pt)
#0.00035277777777777776 print(constants.point) #0.00035277777777777776
print(constants.survey_foot) #0.3048006096012192
print(constants.survey_mile) #1609.3472186944373
print(constants.nautical_mile) #1852.0 (constants.fermi) #1e-15
print(constants.angstrom) #1e-10 print(constants.micron) #1e-06
print(constants.au) #149597870691.0 print(constants.astronomical_unit)
#149597870691.0 print(constants.light_year) #9460730472580800.0
print(constants.parsec) #3.0856775813057292e+16**

**Pressure:**

Return the specified unit in **pascals** (e.g. psi returns
6894.757293168361)

**Example**

**from scipy import constants**

**print(constants.atm) #101325.0**

**print(constants.atmosphere) #101325.0 print(constants.bar) #100000.0
print(constants.torr) #133.32236842105263 print(constants.mmHg)
#133.32236842105263 print(constants.psi) #6894.757293168361**

**Area:**

Return the specified unit in **square meters**(e.g. hectare returns
10000.0)

**Example**

**from scipy import constants**

**(constants.hectare) #10000.0 print(constants.acre)
#4046.8564223999992**

**Volume:**

Return the specified unit in **cubic meters** (e.g. liter returns 0.001)

**Example**

**from scipy import constants**

+---------------------------------------------+-------+----------------+
| **print(constants.liter)**                  |       | > **#0.001**   |
+=============================================+=======+:===============+
| **print(constants.litre)**                  |       | > **#0.001**   |
+---------------------------------------------+-------+----------------+

**print(constants.gallon) #0.0037854117839999997
print(constants.gallon_US) #0.0037854117839999997
print(constants.gallon_imp) #0.00454609 print(constants.fluid_ounce)
#2.9573529562499998e-05 print(constants.fluid_ounce_US)
#2.9573529562499998e-05 print(constants.fluid_ounce_imp) #2.84130625e-05
print(constants.barrel) #0.15898729492799998 print(constants.bbl)
#0.15898729492799998**

**Speed:**

Return the specified unit in **meters per second** (e.g. speed_of_sound
returns 340.5) **Example**

**from scipy import constants**

+-------------------------------+-----+--------------------------------+
| > **(constants.kmh)**         |     | > **#0.2777777777777778**      |
+===============================+=====+:===============================+
| **print(constants.mph)**      |     | > **#0.44703999999999994**     |
+-------------------------------+-----+--------------------------------+

**print(constants.mach) #340.5 print(constants.speed_of_sound) #340.5
print(constants.knot) #0.5144444444444445**

**Temperature:**

Return the specified unit in **Kelvin** (e.g. zero_Celsius returns
273.15)

**Example**

**from scipy import constants**

**print(constants.zero_Celsius) #273.15
print(constants.degree_Fahrenheit) #0.5555555555555556**

**Energy:**

Return the specified unit in **joules** (e.g. calorie returns 4.184)

**Example**

**from scipy import constants**

**print(constants.eV) #1.6021766208e-19**

**print(constants.electron_volt) #1.6021766208e-19
print(constants.calorie) #4.184 print(constants.calorie_th) #4.184
print(constants.calorie_IT) #4.1868 print(constants.erg) #1e-07**

+---------------------------------------+------------------------------+
| **(constants.Btu)**                   | **#1055.05585262**           |
+=======================================+==============================+
| **print(constants.Btu_IT)**           | > **#1055.05585262**         |
+---------------------------------------+------------------------------+
| **print(constants.Btu_th)**           | > **#1054.3502644888888**    |
+---------------------------------------+------------------------------+
| **print(constants.ton_TNT)**          | > **#4184000000.0**          |
+---------------------------------------+------------------------------+

**Power:**

Return the specified unit in **watts** (e.g. horsepower returns
745.6998715822701)

**Example**

**from scipy import constants**

**print(constants.hp) #745.6998715822701**

**print(constants.horsepower) #745.6998715822701**

**Force:**

Return the specified unit in **newton** (e.g. kilogram_force returns
9.80665)

**Example**

**from scipy import constants**

**print(constants.dyn) #1e-05 print(constants.dyne) #1e-05
print(constants.lbf) #4.4482216152605 print(constants.pound_force)
#4.4482216152605 print(constants.kgf) #9.80665
print(constants.kilogram_force) #9.80665**

# SciPy Optimizers

**Optimizers in SciPy**

Optimizers are a set of procedures defined in SciPy that either find the
minimum value of a function, or the root of an equation.

**Optimizing Functions**

Essentially, all of the algorithms in Machine Learning are nothing more
than a complex equation that needs to be minimized with the help of
given data.

**Roots of an Equation**

NumPy is capable of finding roots for polynomials and linear equations,
but it can not find roots for *nonlinear* equations, like this one:

x + cos(x)

For that you can use SciPy\'s optimize.root function.

This function takes two required arguments: ***fun*** - a function
representing an equation. ***x0*** - an initial guess for the root.

The function returns an object with information regarding the solution.

The actual solution is given under attribute x of the returned object:

**Example**

Find root of the equation x + cos(x): **from scipy.optimize import root
from math import cos def eqn(x):**

> **return x + cos(x)**

**myroot = root(eqn, 0) print(myroot.x)**

**Note:** The returned object has much more information about the
solution.

**Example**

Print all information about the solution (not just x which is the root)
**print(myroot)**

**Minimizing a Function**

A function, in this context, represents a curve, curves have *high
points* and *low points*.

High points are called *maxima*.

Low points are called *minima*.

The highest point in the whole curve is called *global maxima*, whereas
the rest of them are called *local maxima*.

The lowest point in the whole curve is called *global minima*, whereas
the rest of them are called *local minima*.

**Finding Minima**

We can use scipy.optimize.minimize() function to minimize the function.

The minimize() function takes the following arguments: ***fun*** - a
function representing an equation. ***x0*** - an initial guess for the
root.

***method*** - name of the method to use. Legal values:

> \'CG\'
>
> \'BFGS\'
>
> \'Newton-CG\'
>
> \'L-BFGS-B\'
>
> \'TNC\'
>
> \'COBYLA\'
>
> \'SLSQP\'

***callback*** - function called after each iteration of optimization.

***options*** - a dictionary defining extra params:

> {
>
> \"disp\": boolean - print detailed description
>
> \"gtol\": number - the tolerance of the error
>
> }

**Example**

Minimize the function x\^2 + x + 2 with BFGS:

**from scipy.optimize import minimize def eqn(x):**

**return x\*\*2 + x + 2 mymin = minimize(eqn, 0, method=\'BFGS\')
print(mymin)**

# SciPy Sparse Data

**What is Sparse Data**

Sparse data is data that has mostly unused elements (elements that
don\'t carry any information ).

It can be an array like this one:

\[1, 0, 2, 0, 0, 3, 0, 0, 0, 0, 0, 0\]

**Sparse Data:** is a data set where most of the item values are zero.

**Dense Array:** is the opposite of a sparse array: most of the values
are *not* zero.

In scientific computing, when we are dealing with partial derivatives in
linear algebra we will come across sparse data.

**How to Work With Sparse Data**

SciPy has a module, scipy.sparse that provides functions to deal with
sparse data.

There are primarily two types of sparse matrices that we use:

**CSC** - Compressed Sparse Column. For efficient arithmetic, fast
column slicing.

**CSR** - Compressed Sparse Row. For fast row slicing, faster matrix
vector products

We will use the **CSR** matrix in this tutorial.

**CSR Matrix**

We can create a CSR matrix by passing an array into the function
scipy.sparse.csr_matrix().

**Example**

Create a CSR matrix from an array: **import numpy as np**

**from scipy.sparse import csr_matrix arr = np.array(\[0, 0, 0, 0, 0, 1,
1, 0, 2\]) print(csr_matrix(arr))**

The example above returns:

> *(0, 5) 1*
>
> *(0, 6) 1*
>
> *(0, 8) 2*

From the result we can see that there are 3 items with value.

The 1. item is in row 0 position 5 and has the value 1.

The 2. item is in row 0 position 6 and has the value 1.

The 3. item is in row 0 position 8 and has the value 2.

**Sparse Matrix Methods**

Viewing stored data (not the zero items) with the data property:

**Example import numpy as np**

**from scipy.sparse import csr_matrix**

**arr = np.array(\[\[0, 0, 0\], \[0, 0, 1\], \[1, 0, 2\]\])
print(csr_matrix(arr).data)**

Counting nonzeros with the count_nonzero() method:

**Example import numpy as np**

**from scipy.sparse import csr_matrix**

**arr = np.array(\[\[0, 0, 0\], \[0, 0, 1\], \[1, 0, 2\]\])
print(csr_matrix(arr).count_nonzero())**

Removing zero-entries from the matrix with the eliminate_zeros() method:

**Example import numpy as np**

**from scipy.sparse import csr_matrix**

**arr = np.array(\[\[0, 0, 0\], \[0, 0, 1\], \[1, 0, 2\]\]) mat =
csr_matrix(arr) mat.eliminate_zeros() print(mat)**

Eliminating duplicate entries with the sum_duplicates() method:

**Example**

Eliminating duplicates by adding them: **import numpy as np**

**from scipy.sparse import csr_matrix**

**arr = np.array(\[\[0, 0, 0\], \[0, 0, 1\], \[1, 0, 2\]\]) mat =
csr_matrix(arr) mat.sum_duplicates() print(mat)**

Converting from csr to csc with the tocsc() method:

**Example import numpy as np**

**from scipy.sparse import csr_matrix**

**arr = np.array(\[\[0, 0, 0\], \[0, 0, 1\], \[1, 0, 2\]\]) newarr =
csr_matrix(arr).tocsc() print(newarr)**

**Note:** Apart from the mentioned sparse specific operations, sparse
matrices support all of the operations that normal matrices support e.g.
reshaping, summing, arithmetic, broadcasting etc.

# SciPy Graphs

**Working with Graphs**

Graphs are an essential data structure.

SciPy provides us with the module scipy.sparse.csgraph for working with
such data structures.

**Adjacency Matrix**

Adjacency matrix is a nxn matrix where n is the number of elements in a
graph.

And the values represent the connection between the elements.

For a graph like this, with elements A, B and C, the connections are:

A & B are connected with weight 1.

A & C are connected with weight 2.

C & B is not connected.

The Adjacency Matrix would look like this:

> *A B C*
>
> *A:\[0 1 2\]*
>
> *B:\[1 0 0\]*
>
> *C:\[2 0 0\]*

Below follows some of the most used methods for working with adjacency
matrices.

**Connected Components**

Find all of the connected components with the connected_components()
method.

**Example import numpy as np**

**from scipy.sparse.csgraph import connected_components**

**from scipy.sparse import csr_matrix arr = np.array(\[**

> **\[0, 1, 2\],**
>
> **\[1, 0, 0\],**
>
> **\[2, 0, 0\]**

**\])**

**newarr = csr_matrix(arr) print(connected_components(newarr))**

**Dijkstra**

Use the dijkstra method to find the shortest path in a graph from one
element to another.

It takes following arguments:

1.  **return_predecessors:** boolean (True to return whole path of
    traversal otherwise False).

2.  **indices:** index of the element to return all paths from that
    element only.

3.  **limit:** max weight of path.

**Example**

Find the shortest path from element 1 to 2: **import numpy as np**

**from scipy.sparse.csgraph import dijkstra from scipy.sparse import
csr_matrix arr = np.array(\[**

> **\[0, 1, 2\],**
>
> **\[1, 0, 0\],**
>
> **\[2, 0, 0\]**

**\])**

**newarr = csr_matrix(arr)**

**print(dijkstra(newarr, return_predecessors=True, indices=0))**

**Floyd Warshall**

Use the floyd_warshall() method to find the shortest path between all
pairs of elements.

**Example**

Find the shortest path between all pairs of elements: **import numpy as
np**

**from scipy.sparse.csgraph import floyd_warshall from scipy.sparse
import csr_matrix arr = np.array(\[**

> **\[0, 1, 2\],**
>
> **\[1, 0, 0\],**
>
> **\[2, 0, 0\]**

**\])**

**newarr = csr_matrix(arr)**

**print(floyd_warshall(newarr, return_predecessors=True))**

**Bellman Ford**

The bellman_ford() method can also find the shortest path between all
pairs of elements, but this method can handle negative weights as well.

**Example**

Find shortest path from element 1 to 2 with given graph with a negative
weight:

**import numpy as np**

**from scipy.sparse.csgraph import bellman_ford from scipy.sparse import
csr_matrix arr = np.array(\[**

> **\[0, -1, 2\],**
>
> **\[1, 0, 0\],**
>
> **\[2, 0, 0\]**

**\])**

**newarr = csr_matrix(arr)**

**print(bellman_ford(newarr, return_predecessors=True, indices=0))**

**Depth First Order**

The depth_first_order() method returns a depth first traversal from a
node.

This function takes following arguments:

1.  the graph.

2.  the starting element to traverse the graph from.

**Example**

Traverse the graph depth first for given adjacency matrix: **import
numpy as np**

**from scipy.sparse.csgraph import depth_first_order from scipy.sparse
import csr_matrix arr = np.array(\[**

> **\[0, 1, 0, 1\],**
>
> **\[1, 1, 1, 1\],**
>
> **\[2, 1, 1, 0\],**
>
> **\[0, 1, 0, 1\]**

**\])**

**newarr = csr_matrix(arr) print(depth_first_order(newarr, 1))**

**Breadth First Order**

The breadth_first_order() method returns a breadth first traversal from
a node.

This function takes following arguments:

1.  the graph.

2.  the starting element to traverse the graph from.

**Example**

Traverse the graph breadth first for given adjacency matrix: **import
numpy as np**

**from scipy.sparse.csgraph import breadth_first_order from scipy.sparse
import csr_matrix arr = np.array(\[**

> **\[0, 1, 0, 1\],**
>
> **\[1, 1, 1, 1\],**
>
> **\[2, 1, 1, 0\],**
>
> **\[0, 1, 0, 1\]**

**\])**

**newarr = csr_matrix(arr) print(breadth_first_order(newarr, 1))**

# SciPy Spatial Data

**Working with Spatial Data**

Spatial data refers to data that is represented in a geometric space.
E.g. points on a coordinate system.

We deal with spatial data problems on many tasks. E.g. finding if a
point is inside a boundary or not.

SciPy provides us with the module scipy.spatial, which has functions for
working with spatial data.

**Triangulation**

A Triangulation of a polygon is to divide the polygon into multiple
triangles with which we can compute an area of the polygon.

A Triangulation *with points* means creating surface composed triangles
in which all of the given points are on at least one vertex of any
triangle in the surface.

One method to generate these triangulations through points is the
Delaunay() Triangulation.

**Example**

Create a triangulation from following points:

**import numpy as np**

**from scipy.spatial import Delaunay import matplotlib.pyplot as plt**

**points = np.array(\[**

> **\[2, 4\],**
>
> **\[3, 4\],**
>
> **\[3, 0\],**
>
> **\[2, 2\],**
>
> **\[4, 1\]**

**\])**

**simplices = Delaunay(points).simplices**

**plt.triplot(points\[:, 0\], points\[:, 1\], simplices)
plt.scatter(points\[:, 0\], points\[:, 1\], color=\'r\') plt.show()**

**Result:**![](media/image5.jpg){width="3.6145833333333335in"
height="2.7083333333333335in"}

**Note:** The simplices property creates a generalization of the
triangle notation.

**Convex Hull**

A convex hull is the smallest polygon that covers all of the given
points.

Use the ConvexHull() method to create a Convex Hull.

**Example**

Create a convex hull for following points: **import numpy as np**

**from scipy.spatial import ConvexHull import matplotlib.pyplot as plt
points = np.array(\[**

> **\[2, 4\],**
>
> **\[3, 4\], \[3, 0\],**
>
> **\[2, 2\],**
>
> **\[4, 1\],**
>
> **\[1, 2\], \[5, 0\],**
>
> **\[3, 1\],**
>
> **\[1, 2\],**
>
> **\[0, 2\]**

**\])**

**hull = ConvexHull(points) hull_points = hull.simplices
plt.scatter(points\[:,0\], points\[:,1\]) for simplex in hull_points:**

> **plt.plot(points\[simplex,0\], points\[simplex,1\], \'k-\')**

**plt.show() Result:**

![](media/image6.jpg){width="3.25in" height="2.4270833333333335in"}

**KDTrees**

KDTrees are a datastructure optimized for nearest neighbor queries.

E.g. in a set of points using KDTrees we can efficiently ask which
points are nearest to a certain given point.

The KDTree() method returns a KDTree object.

The query() method returns the distance to the nearest neighbor *and*
the location of the neighbors.

**Example**

Find the nearest neighbor to point (1,1): **from scipy.spatial import
KDTree**

**points = \[(1, -1), (2, 3), (-2, 3), (2, -3)\] kdtree = KDTree(points)
res = kdtree.query((1, 1)) print(res)**

**Result: (2.0, 0)**

**Distance Matrix**

There are many Distance Metrics used to find various types of distances
between two points in data science, Euclidean distance, cosine distance
etc.

The distance between two vectors may not only be the length of straight
line between them, it can also be the angle between them from origin, or
number of unit steps required etc.

Many of the Machine Learning algorithm\'s performance depends greatly on
distance matrices. E.g. \"K Nearest Neighbors\'\', or \"K Means\'\' etc.

Let us look at some of the Distance Metrics:

**Euclidean Distance**

Find the euclidean distance between given points.

**Example**

**from scipy.spatial.distance import euclidean**

**p1 = (1, 0) p2 = (10, 2) res = euclidean(p1, p2) print(res)**

**Result:**

**9.21954445729**

**Cityblock Distance (Manhattan Distance)**

Is the distance computed using 4 degrees of movement.

E.g. we can only move: up, down, right, or left, not diagonally.

**Example**

Find the cityblock distance between given points: **from
scipy.spatial.distance import cityblock p1 = (1, 0) p2 = (10, 2) res =
cityblock(p1, p2) print(res)**

**Result: 11**

**Cosine Distance**

Is the value of cosine angle between the two points A and B.

**Example**

Find the cosine distance between given points: **from
scipy.spatial.distance import cosine p1 = (1, 0) p2 = (10, 2) res =
cosine(p1, p2) print(res)**

**Result:**

**0.019419324309079777**

**Hamming Distance**

Is the proportion of bits where two bits are different.

It\'s a way to measure distance for binary sequences.

**Example**

**Find the hamming distance between given points: from
scipy.spatial.distance import hamming p1 = (True, False, True) p2 =
(False, True, True) res = hamming(p1, p2) print(res)**

**Result:**

**0.666666666667**

# SciPy Matlab Arrays

**Working With Matlab Arrays**

We know that NumPy provides us with methods to persist the data in
readable formats for Python. But SciPy provides us with interoperability
with Matlab as well.

SciPy provides us with the module scipy.io, which has functions for
working with Matlab arrays.

**Exporting Data in Matlab Format**

The savemat() function allows us to export data in Matlab format.

The method takes the following parameters:

1.  **filename** - the file name for saving data.

2.  **mdict** - a dictionary containing the data.

3.  **do_compression** - a boolean value that specifies whether to
    compress the result or not. Default False.

**Example**

Export the following array as variable name \"vec\" to a mat file:

**from scipy import io import numpy as np arr = np.arange(10)**

**io.savemat(\'arr.mat\', {\"vec\": arr})**

**Note:** The example above saves a file name \"arr.mat\" on your
computer.

To open the file, check out the \"Import Data from Matlab Format\"
example below:

**Import Data from Matlab Format**

The loadmat() function allows us to import data from a Matlab file.

The function takes one required parameter:

**filename** - the file name of the saved data.

It will return a structured array whose keys are the variable names, and
the corresponding values are the variable values.

**Example**

Import the array from following mat file.:

**from scipy import io import numpy as np**

**arr = np.array(\[0, 1, 2, 3, 4, 5, 6, 7, 8, 9,\]) \# Export:**

**io.savemat(\'arr.mat\', {\"vec\": arr}) \# Import: mydata =
io.loadmat(\'arr.mat\') print(mydata)**

**Result:**

*{*

> *\'\_\_header\_\_\': b\'MATLAB 5.0 MAT-file Platform: nt, Created on:
> Tue Sep 22 13:12:32 2020\',*
>
> *\'\_\_version\_\_\': \'1.0\',*
>
> *\'\_\_globals\_\_\': \[\],*
>
> *\'vec\': array(\[\[0, 1, 2, 3, 4, 5, 6, 7, 8, 9\]\])*

*}*

Use the variable name \"vec\" to display only the array from the matlab
data:

**Example**

**\...**

**print(mydata\[\'vec\'\])**

**Result:**

*\[\[0 1 2 3 4 5 6 7 8 9\]\]*

**Note:** We can see that the array originally was 1D, but on extraction
it has increased one dimension.

In order to resolve this we can pass an additional argument
squeeze_me=True:

**Example \# Import: mydata = io.loadmat(\'arr.mat\', squeeze_me=True)
print(mydata\[\'vec\'\])**

***Result:***

*\[0 1 2 3 4 5 6 7 8 9\]*

# SciPy Interpolation

**What is Interpolation?**

Interpolation is a method for generating points between given points.

For example: for points 1 and 2, we may interpolate and find points 1.33
and 1.66.

Interpolation has many uses, in Machine Learning we often deal with
missing data in a dataset, interpolation is often used to substitute
those values.

This method of filling values is called *imputation*.

Apart from imputation, interpolation is often used where we need to
smooth the discrete points in a dataset.

**How to Implement it in SciPy?**

SciPy provides us with a module called scipy.interpolate which has many
functions to deal with interpolation:

**1D Interpolation**

The function interp1d() is used to interpolate a distribution with 1
variable.

It takes x and y points and returns a callable function that can be
called with new x and returns corresponding y.

**Example**

For given xs and ys interpolate values from 2.1, 2.2\... to 2.9:

**from scipy.interpolate import interp1d import numpy as np xs =
np.arange(10) ys = 2\*xs + 1**

**interp_func = interp1d(xs, ys)**

**newarr = interp_func(np.arange(2.1, 3, 0.1)) print(newarr)**

**Result:**

**\[5.2 5.4 5.6 5.8 6. 6.2 6.4 6.6 6.8\]**

**Note:** that new xs should be in the same range as of the old xs,
meaning that we can\'t call interp_func() with values higher than 10, or
less than 0.

**Spline Interpolation**

In 1D interpolation the points are fitted for a *single curve* whereas
in Spline interpolation the points are fitted against a *piecewise*
function defined with polynomials called splines.

The UnivariateSpline() function takes xs and ys and produces a callback
function that can be called with new xs.

**Piecewise function:** A function that has different definitions for
different ranges.

**Example**

Find univariate spline interpolation for 2.1, 2.2\... 2.9 for the
following non linear points:

**from scipy.interpolate import UnivariateSpline**

**import numpy as np xs = np.arange(10) ys = xs\*\*2 + np.sin(xs) + 1
interp_func = UnivariateSpline(xs, ys) newarr =
interp_func(np.arange(2.1, 3, 0.1)) print(newarr)**

**Result:**

*\[5.62826474 6.03987348 6.47131994 6.92265019 7.3939103 7.88514634*

> *8.39640439 8.92773053 9.47917082\]*

**Interpolation with Radial Basis Function**

Radial basis function is a function that is defined corresponding to a
fixed reference point.

The Rbf() function also takes xs and ys as arguments and produces a
callable function that can be called with new xs.

**Example**

Interpolate following xs and ys using rbf and find values for 2.1, 2.2
\... 2.9:

**from scipy.interpolate import Rbf import numpy as np xs =
np.arange(10) ys = xs\*\*2 + np.sin(xs) + 1 interp_func = Rbf(xs, ys)
newarr = interp_func(np.arange(2.1, 3, 0.1)) print(newarr)**

**Result:**

*\[6.25748981 6.62190817 7.00310702 7.40121814 7.8161443 8.24773402*

> *8.69590519 9.16070828 9.64233874\]*

# SciPy Statistical Significance Tests

**What is a Statistical Significance Test?**

In statistics, statistical significance means that the result that was
produced has a reason behind it, it was not produced randomly, or by
chance.

SciPy provides us with a module named scipy.stats, which has functions
for performing statistical significance tests.

Here are some techniques and keywords that are important when performing
such tests.

**Hypothesis in Statistics**

Hypothesis is an assumption about a parameter in population.

**Null Hypothesis**

It assumes that the observation is not statistically significant.

**Alternate Hypothesis**

It assumes that the observations are due to some reason.

It\'s an alternative to Null Hypothesis.

**Example:**

For an assessment of a student we would take:

*\"student is worse than average\"* - as a null hypothesis, and:

*\"student is better than average\"* - as an alternate hypothesis.

**One tailed test**

When our hypothesis is testing for one side of the value only, it is
called \"one tailed test\".

**Example:**

For the null hypothesis:

*\"the mean is equal to k\",* we can have alternate hypothesis:

*\"the mean is less than k\",* or:

*\"the mean is greater than k\"*

**Two tailed test**

When our hypothesis is tested for both sides of the values.

**Example:**

For the null hypothesis:

*\"the mean is equal to k\",* we can have alternate hypothesis:

*\"the mean is not equal to k\"*

In this case the mean is less than, or greater than k, and both sides
are to be checked.

**Alpha value**

Alpha value is the level of significance.

**Example:**

How close to extremes the data must be for null hypothesis to be
rejected.

It is usually taken as 0.01, 0.05, or 0.1.

**P value**

P value tells how close to extreme the data actually is.

P value and alpha values are compared to establish statistical
significance.

If p value \<= alpha we reject the null hypothesis and say that the data
is statistically significant. otherwise we accept the null hypothesis.

**T-Test**

T-tests are used to determine if there is significant difference between
means of two variables. and lets us know if they belong to the same
distribution.

It is a two tailed test.

The function ttest_ind() takes two samples of the same size and produces
a tuple of t-statistic and p-value.

**Example**

Find if the given values v1 and v2 are from same distribution:

**import numpy as np from scipy.stats import ttest_ind v1 =
np.random.normal(size=100) v2 = np.random.normal(size=100) res =
ttest_ind(v1, v2) print(res)**

**Result:**

0.68346891833752133

If you want to return only the p-value, use the p value property:

**Example**

**\...**

**res = ttest_ind(v1, v2).pvalue print(res)**

**Result:**

0.68346891833752133

**KS-Test**

KS test is used to check if given values follow a distribution.

The function takes the value to be tested, and the CDF as two
parameters.

A **CDF** can be either a string or a callable function that returns the
probability.

It can be used as a one tailed or two tailed test.

By default it is two tails. We can pass parameter alternatives as a
string of one of two-sided, less, or greater.

**Example**

Find if the given value follows the normal distribution:

**import numpy as np from scipy.stats import kstest v =
np.random.normal(size=100) res = kstest(v, \'norm\') print(res)**

**Result:**

*KstestResult(statistic=0.047798701221956841,
pvalue=0.97630967161777515)*

**Statistical Description of Data**

In order to see a summary of values in an array, we can use the
describe() function.

It returns the following description:

1.  number of observations (nobs)

2.  minimum and maximum values = minmax

3.  mean

4.  variance

5.  skewness

6.  kurtosis

**Example**

Show statistical description of the values in an array:

**import numpy as np from scipy.stats import describe**

**v = np.random.normal(size=100) res = describe(v) print(res)**

**Result:**

*DescribeResult( nobs=100,*

> *minmax=(-2.0991855456740121, 2.1304142707414964),
> mean=0.11503747689121079, variance=0.99418092655064605,
> skewness=0.013953400984243667, kurtosis=-0.671060517912661*
>
> *)*

**Normality Tests (Skewness and Kurtosis)**

Normality tests are based on the skewness and kurtosis.

The normaltest() function returns p value for the null hypothesis:

*\"x comes from a normal distribution\"*.

**Skewness:**

A measure of symmetry in data.

For normal distributions it is 0.

If it is negative, it means the data is skewed left.

If it is positive it means the data is skewed right.

**Kurtosis:**

A measure of whether the data is heavy or lightly tailed to a normal
distribution.

Positive kurtosis means heavy tailed.

Negative kurtosis means lightly tailed.

**Example**

Find skewness and kurtosis of values in an array:

**import numpy as np**

**from scipy.stats import skew, kurtosis v = np.random.normal(size=100)
print(skew(v)) print(kurtosis(v))**

**Result:**

0.11168446328610283

> -0.1879320563260931

**Example**

Find if the data comes from a normal distribution:

**import numpy as np from scipy.stats import normaltest v =
np.random.normal(size=100) print(normaltest(v))**

**Result:**

NormaltestResult(statistic=4.4783745697002848,
pvalue=0.10654505998635538)
