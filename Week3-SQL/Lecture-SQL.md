
# SQL Lecture 

### Getting setup on BigQuery
1. Getting setup on BigQuery.
	2. Go to BigQuery UI https://console.cloud.google.com/bigquery
2. Adding in the public data sets. 
	3. Click the Add Data icon
	4. Add any dataset
	5. `bigquery-public-data` should become visible and populate. 
3. How to effectively use the BigQuery UI
	4. Navigating tables within DataSets

# Diving into SQL
[Cheat Sheet](https://cdn.sqltutorial.org/wp-content/uploads/2016/04/SQL-cheat-sheet.pdf) 
1. General syntax
	2. Comments
	3. Tables use back-ticks ` and not quotes '.
	4. Limit 
	```
	# SELECTING ALL ROWS, AND LIMITING
	SELECT
	  *
	FROM
	  `bigquery-public-data.new_york_trees.tree_census_2015`
	LIMIT
	  1000
	```

2.  Selecting columns
	3. Auto complete with TAB
	```
	SELECT
	  tree_id,
	  status,
	  spc_common
	FROM
	  `bigquery-public-data.new_york_trees.tree_census_2015`
	```
3. Ordering 
	```
	SELECT
	  *
	FROM
	  `bigquery-public-data.new_york_trees.tree_census_2015`
	ORDER BY
	  block_id DESC,
	  created_at ASC
	```
4. Renaming columns Using 'as'
	```
	SELECT
	  tree_id AS id,
	  status AS tree_status
	FROM
	  `bigquery-public-data.new_york_trees.tree_census_2015`
	```
5. Renaming tables using 'as'
	```
	SELECT
	  T.tree_id,
	  T.status,
	  T.spc_common
	FROM
	  `bigquery-public-data.new_york_trees.tree_census_2015` AS T
	```
6. Filtering using WHERE
	7. And, Or, In, and Like
	```
	# USING 'AND'
	SELECT
	  *
	FROM
	  `bigquery-public-data.new_york_trees.tree_census_2015`
	WHERE
	  tree_id = "108086"
	  AND status = "Alive"
	```
	```
	# USING 'OR'
	SELECT
	  *
	FROM
	  `bigquery-public-data.new_york_trees.tree_census_2015`
	WHERE
	  tree_id = 108086
	  OR tree_id = 45875
	```
	```
	# USING 'IN'
	SELECT
	  *
	FROM
	  `bigquery-public-data.new_york_trees.tree_census_2015`
	WHERE
	  tree_id IN (108086, 45875)
	```
	```
	# USING LIKE
	SELECT
	  *
	FROM
	  `bigquery-public-data.usa_names.usa_1910_current`
	WHERE
	  name LIKE '%Zac%'
	```
	
8. Group By and functions
	```
	SELECT
	  status,
	  COUNT(status) AS counts,
	  COUNT(DISTINCT status) AS d_counts
	FROM
	  `bigquery-public-data.new_york_trees.tree_census_2015`
	GROUP BY
	  status
	```

	```
	# GROUP BY WITH AVERAGE
	SELECT
	  status,
	  COUNT(tree_id) as counts,
	  AVG(tree_dbh) AS avg_diameter
	FROM
	  `bigquery-public-data.new_york_trees.tree_census_2015`
	GROUP BY
	  status
	```
	```
	# DOUBLE GROUP BY 
	SELECT
	  status,
	  health
	  COUNT(tree_id) as counts,
	  AVG(tree_dbh) AS avg_diameter
	FROM
	  `bigquery-public-data.new_york_trees.tree_census_2015`
	GROUP BY
	  status, health
	```
9. Temporary Tables and how to use them.
	```
	WITH T AS (
	  SELECT
	    block_id,
	    COUNT(tree_id) AS counts
	  FROM
	    `bigquery-public-data.new_york_trees.tree_census_2015`
	  GROUP BY
	    block_id
	  )
	SELECT * FROM  T WHERE  counts > 50
	```
7. Changing column data types
	8. Int to String
	9. String to DateTime
	```
	SELECT
	  CAST(year as STRING) as year_string,
	  CAST(month as STRING) as month_string, 
	  CAST(day as STRING) as day_string
	FROM
	  `bigquery-public-data.samples.gsod`
	WHERE
	  station_number = 723758
	  AND year = 2009
	```
	```
	WITH
	# FIRST CAST EACH YEAR, MONTH, DATE TO STRINGS
	  T AS (
	  SELECT
	    *,
	    CAST(year AS STRING) AS year_string,
	    CAST(month AS STRING) AS month_string,
	    CAST(day AS STRING) AS day_string
	  FROM
	    `bigquery-public-data.samples.gsod`
	  WHERE
	    station_number = 723758
	    AND year = 2009 ),


	# SECOND, CONCAT ALL THE STRINGS TOGETHER INTO ONE COLUMN]
	  TT AS (
	  SELECT
	    *,
	    CONCAT( year_string, "-", month_string, "-", day_string ) AS date_string
	  FROM
	    T),
	    
	# THIRD, CAST THE DATE STRING INTO A DATE FROMAT
	  TTT AS (
	  SELECT
	    *,
	    CAST(date_string AS DATE) AS date_date
	  FROM
	    TT)
	SELECT
	  *
	FROM
	  TTT
	WHERE
	  date_date BETWEEN '2009-10-01' AND '2009-10-10'
	ORDER BY date_date DESC
	```

10. Joining tables
* Review / go through duplicate column name errors.   
	```
	# WILL THROW DUPLICATE COLUMN ERROR
	WITH TABLE_2011 AS (
	    SELECT descript, COUNT(unique_key) as count_2011 FROM `bigquery-public-data.austin_incidents.incidents_2011` GROUP BY descript
	)
	, TABLE_2010 AS (
	    SELECT descript, COUNT(unique_key) as count_2010 FROM `bigquery-public-data.austin_incidents.incidents_2010` GROUP BY descript
	)
	SELECT * FROM TABLE_2011 as A JOIN TABLE_2010 as B ON A.descript = B.descript
	```
 
	```
	WITH TABLE_2011 AS (
	    SELECT descript, COUNT(unique_key) as count_2011 FROM `bigquery-public-data.austin_incidents.incidents_2011` GROUP BY descript
	)
	, TABLE_2010 AS (
	    SELECT descript, COUNT(unique_key) as count_2010 FROM `bigquery-public-data.austin_incidents.incidents_2010` GROUP BY descript
	)
	SELECT A.descript, A.count_2011, B.count_2010 FROM TABLE_2011 as A JOIN TABLE_2010 as B ON A.descript = B.descript
	```

11. Assigning values using CASE WHEN
```
WITH
  T AS (
  SELECT
    block_id,
    COUNT(tree_id) AS counts
  FROM
    `bigquery-public-data.new_york_trees.tree_census_2015`
  GROUP BY
    block_id )
SELECT
  *,
  CASE
    WHEN counts > 40 THEN 'lots of trees'
    WHEN counts > 30 THEN 'medium amount of trees'
  ELSE
  'not many trees'
END 
  AS column_how_many_trees
FROM
  T
```
# Using BigQuery in Google's Colab
1. Using Google's CoLab with BigQuery
	1. Open up my [starter notebook](https://colab.research.google.com/drive/1kHdTtuHTPEaMH32GotVum41YVdeyzQ74?usp=sharing)  to get started.
	2. Save a copy of that notebook in your drive. 
	3. Setup sharing.
	4. Being editing that file.
	5. Authentication
	6. Adding your project-id.
	7. Running your first query. 
	8. Dont forget to paste the link your notebook file into the bottom of the exercise file. 

   
