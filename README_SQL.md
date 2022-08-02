# Notes on SQL
From [Microsoft documentation](https://docs.microsoft.com/en-us/sql/t-sql/functions/functions?view=sql-server-ver16)

## Types of Functions

### 1. Aggregate Functions

Aggregate functions perform a calculation on a set of values and return a single value. They are allowed in the select list or the HAVING clause of a SELECT statement. You can use an aggregation in combination with the GROUP BY clause to calculate the aggregation on categories of rows. Use the OVER clause to calculate the aggregation on a specific range of value. The OVER clause cannot follow the GROUPING or GROUPING_ID aggregations.

All aggregate functions are deterministic.

### 2. Analytic Functions
Analytic functions compute an aggregate value based on a group of rows. However, unlike aggregate functions, analytic functions can return multiple rows for each group. You can use analytic functions to compute moving averages, running totals, percentages, or top-N results within a group.

### 3. Bit manipulation functions
Bit manipulation functions allow you to process and store data more efficiently than with individual bits

### 4. Ranking Functions
Ranking functions return a ranking value for each row in a partition. Depending on the function that is used, some rows might receive the same value as other rows. Ranking functions are nondeterministic.

### 5. Rowset Functions
Rowset functions Return an object that can be used like table references in an SQL statement.

### 6. Scalar Functions
Operate on a single value and then return a single value. Scalar functions can be used wherever an expression is valid.


## Examples. Different clauses

1. Comments: -- for one line, /* */ for blocks

2. Nested calls

'''sql
-- Nested calls
SELECT some_columns
  FROM file_archive_info
  WHERE desfile_id IN (
    SELECT id
    FROM desfile
    WHERE pfw_attempt_id=11199
    AND filetype="processed_bias"
    );
'''

3. To broad view: set linesize 200

'''sql
-- SET
SET linesize 250
'''

4. Different schemas, databases, tables

'''sql
database_name.schema_name.table_name
'''


5. Sort by a column's values

'''sql
-- ORDER BY
SELECT id, archive_path
  FROM pfw_attempt
  WHERE reqnum=1122
  ORDER BY submitted_time
  -- note the submitted_time belongs to pfw_attempt table
'''

6. Use of LIKE and DISTINCT

'''sql
-- LIKE
SELECT id, name
  FROM pfw_attempt
  WHERE pfw_attempt_id=113344
  AND name LIKE ('%comparis%');

-- SELECT from different tables and compare
SELECT d.filename, d.filetype
  FROM opm_table o, desfile d
  WHERE d.id = o.desfile_id
  AND o.task=111888777

-- DISTINCT
SELECT DISTINCT d.filename
  FROM desfile d
  WHERE d.filetype="cal_detflat"
'''

7. To delete a table

'''sql
-- DELETE and PURGE (seems PURGE isn't needed on MS SQL)
-- Below doesn't raise an error if TABLE doesn't exists
DROP TABLE IF EXISTS database_name.schema.table_name;

DROP TABLE table_name;
'''

8. Various COUNT, SUBSTR, GROUP BY

'''sql
-- COUNT, SUBSTRING, GROUP BY
SELECT COUNT(a.id), SUBSTRING(a.name, 1, 10), SUBSTRING(a.field, 1, 8)
  FROM pfw_attempt a, request r
  WHERE r.pfw_attempt_id=a.id
  AND r.data_state!='BAD'
  AND r.pipeline='latest'
  GROUP BY a.name, a.field;
'''

9. To compare STR to INT

'''sql
-- CONCAT
SELECT a.id, e.exptime, e.filetype
  FROM pfw_attempt a, image_info e
  WHERE a.reqnum=2333
  AND CONCAT('ID00', a.id)=e.image_id;
'''

10. Use a subquery

'''sql
-- WITH works when you need a subquery
-- ROWNUM, WITH, !=
WITH sel_x as (
  SELECT pfw_attempt_id
    FROM proctag
    WHERE tag='Y5'
    AND rownum<4
  )
  SELECT arc.path, arc. filename
    FROM sel_x, archive arc
    WHERE sel_x.pfw_attempt_id=arc.pfw_attempt_id
    AND rownum<200
    AND arc.status!='OLD';
'''

11. Create temporary table and INSERT values. If using MS SQL, one could use GO.

'''sql
-- CREATE temporal table, BETWEEN, GROUP BY
CREATE TABLE fco_table AS
SELECT unitname,reqnum,max(attnum) AS max_reqnum
    FROM pfw_attempt
    WHERE reqnum BETWEEN 2863 AND 2875
    GROUP BY by unitname, reqnum;

--  using the columns and VALUES vector, to make sure the order is correct
INSERT INTO ops_proctag_def (tag, description)
    VALUES ('Y4A1_Y4E1_PRECAL', 'PRECAL (nightly) runs for Year 4, Epoch 1, for the Y4A1 Processing campaign');

INSERT INTO proctag (tag, created_date, created_by, pfw_attempt_id)
    SELECT 'Y4A1_Y4E2_SUPERCAL', SYSDATE, 'my_username', a.id
    FROM pfw_attempt a, SUPERCAL_Y4E2 x
    WHERE a.reqnum=x.reqnum
    AND a.unitname=x.unitname
    AND a.attnum=x.attnum;
'''

12. Select using wildcard underscore

'''sql
-- will use regions VA, CA, NY, NC
SELECT region
  FROM All_regions
  WHERE region LIKE '_A'
  OR region LIKE 'N_';
'''

13. Use JOIN

14. Save query result to table

'''sql
spool y5a1_precal.tab
select att.archive_path, att.reqnum, att.unitname, att.attnum
  from pfw_attempt att, proctag tag
  where tag.tag='Y5A1 PRECAL'
    and tag.pfw_attempt_id=att.id
    order by att.unitname, att.archive_path;
--or
select {...}; > y5a1_precal.tab
'''

15. Use TOP, AVERAGE, etc

16. Use string UPPER and alias

'''sql
-- Remember, the alias is the value to be displayed when printing the results
SELECT UPPER(last_name) + ', ' + first_name AS name
  FROM dball.Customers
  ORDER BY last_name;
'''

17. Aggregation

'''sql
/* AVG ( [ ALL | DISTINCT ] expression )  
   [ OVER ( [ partition_by_clause ] order_by_clause ) ]
- ALL is the default, but if using DISTINCT, it only goes through unique instance of each value
- OVER divides the result of FROM in partitions
*/
HERE ......
'''

18. Run SQL script

'''sql
-- in MS SQL  
sqlcmd script.sql
'''
