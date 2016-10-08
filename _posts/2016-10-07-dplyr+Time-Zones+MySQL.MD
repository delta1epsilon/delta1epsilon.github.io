---
title:  "dplyr + Time Zones + MySQL"
date:   2016-10-07 15:04:23
categories: [R, dplyr]
tags: [R, dplyr, MySQL]
---
Recently I encountered a need to convert time zones a lot of times after getting
data from MySQL database using dplyr. And in this post I would like to share
with you my solution which makes life a little bit easier.

Well, there is no big deal to convert time zones in R.
Suppose we have to convert UTC to AEST.
For instance, we could use `format` function:

``` r
date <- as.POSIXct('2016-10-01 09:12:34', tz = 'UTC')
format(date, tz = 'Australia/Sydney', usetz = TRUE)
# "2016-10-01 19:12:34 AEST"
```

We can convert time zones in date columns of a dataframe using dplyr `mutate`
function

``` r
library(dplyr)

df %>%
  mutate(DateColumn = as.POSIXct(DateColumn, tz = 'UTC'),
         DateColumn = format(DateColumn, tz = 'Australia/Sydney', usetz = TRUE)
         )
```

Or when you query your database (in my case MySQL) with dplyr and you want
your operations to be done on database side
and you have to convert time zones you can write

``` r
tbl(conection_to_db, 'TableName') %>%
  mutate(DateColumn_tz = CONVERT_TZ(DateColumn, 'UTC', 'Australia/Sydney')) %>%
  select(DateColumn = DateColumn_tz, OtherColumn1, OtherColumn2)
```

Note that `CONVERT_TZ` is MySQL function.
And since dplyr translates R code into SQL, there will be no errors.

However, imagine that all dates in your database are in one time zone and you
need to convert it to another.
Of course you can do that every time you work with data in the database.
However, it can be really annoying + it's extra lines of code + you may forget about it.

So, since I use dplyr for getting data from a database I decided to write my own
`select` function (on top of dplyr `select` and `mutate`) which would convert time
zones (on database side) in all date columns and select requested columns from
MySQL database.

Here is the function:

``` r
select <- function (.data, ...) {
  if ("tbl_mysql" %in% class(.data)) {  # check whether db is MySQL
    string_arguments <- deparse(substitute(args(...)), width.cutoff = 500)
    original_variables <- substring(string_arguments, 6, nchar(string_arguments) - 1)

    # get columns names
    original <- unlist(strsplit(original_variables, ","))

    # remove spaces from column names
    original <- gsub(' ', '', original)

    # all possible date columns in the database
    POSSIBLE_DATE_COLUMNS <- c('DateColumn1', 'DateColumn2', 'DateColumn3')

    # check whether there are date columns
    is_date_column <- original %in% POSSIBLE_DATE_COLUMNS

    if (any(is_date_column)) {
      DATE_COLUMNS <- original[is_date_column]

      for (date_column in DATE_COLUMNS) {  # loop for converting TZ
        # create an expression to be passed into "mutate"
        mutate_expression <-
            sprintf("%s = CONVERT_TZ(%s, 'UTC', 'Australia/Sydney')",
                    paste0(date_column, '_tz'),
                    date_column
                    )

        mutate_command <-
          parse(text=sprintf("dplyr::mutate(.data, %s)",
                             mutate_expression
                             )
               )
        .data <- eval(mutate_command)
      }

      # select columns
      original[original %in% DATE_COLUMNS] <-
          sprintf("%s = %s",
                  DATE_COLUMNS,
                  paste0(DATE_COLUMNS, '_tz')
                  )

      select_columns_command <-
          parse(text=sprintf("dplyr::select(.data, %s)",
                             paste(original, collapse = ', ')
                             )
                )
      result <- eval(select_columns_command)
    } else {
      result <- dplyr::select(.data, ...)        
    }

  } else {
    result <- dplyr::select(.data, ...)
  }

  return(result)
}
```

So, usage of the custom `select` function is the same as `dplyr::select` but now
we'll get all date columns right away in needed time zone.

``` r
tbl(conection_to_db, 'TableName') %>%
  select(DateColumn, OtherColumn)
```

#### Some notes regarding the custom `select`

There is an `if` clause for checking whether `.data` is of class `tbl_mysql`
because we use MySQL function `CONVERT_TZ`. And if `.data` is not of class `tbl_mysql`
`dplyr::select` is called on `.data`.

So, in variable `POSSIBLE_DATE_COLUMNS` you have to specify which columns in
the database are of type datetime.
To figure it out you can query your MySQL
[INFORMATION_SCHEMA](http://dev.mysql.com/doc/refman/5.7/en/columns-table.html).

``` sql
SELECT DISTINCT COLUMN_NAME
FROM information_schema.COLUMNS
WHERE TABLE_SCHEMA = 'YourDataBaseName'
      AND
      DATA_TYPE = 'datetime';
```

Also note that if you use the custom `select` function and after it
`dplyr::filter` with conditions on converted (in `select`) date columns,
then you will not get wanted result. Because `WHERE` clause of SQL query is
executed earlier than `SELECT` clause.
[Here](http://stackoverflow.com/questions/24127932/mysql-query-clause-execution-order)
is order of execution of SQL query. So, you have to use `CONVERT_TZ` MySQL function
in `dplyr::filter` as we did in custom `select` function.
