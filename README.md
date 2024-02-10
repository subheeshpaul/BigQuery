# BigQuery

This file contains Week 3 home work from data-engineering-zoomcamp

-- Creating external table referring to gcs path
CREATE OR REPLACE EXTERNAL TABLE `taxi_rides_ny.external_green_tripdata`
OPTIONS (
  format = 'parquet',
  uris = ['gs://zoomcamp-homework3/nyc_green_taxi_data/lpep_pickup_date*']
);

Question 1: What is count of records for the 2022 Green Taxi Data??
Select count(1) from taxi_rides_ny.external_green_tripdata 
840,402

-- Create a non partitioned table from external table
CREATE OR REPLACE TABLE taxi_rides_ny.green_tripdata AS
SELECT *,TIMESTAMP_MICROS(CAST(lpep_pickup_datetime / 1000 AS INT64)) as lpep_pickup_datet  FROM taxi_rides_ny.external_green_tripdata;


Question 2:
Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.
What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?
-- this query will process 6.41 MB when run
Select distinct PULocationID from taxi_rides_ny.green_tripdata 

this query will process 0 MB when run as BQ won't be able to show the query estimate details  about an external table
Select distinct PULocationID from taxi_rides_ny.external_green_tripdata 

How many records have a fare_amount of 0?
Records having fair amount = 0 ans: 1622
Select count(1) from taxi_rides_ny.external_green_tripdata where fare_amount = 0

Question 4:
What is the best strategy to make an optimized table in Big Query if your query will always order the results by PUlocationID and filter based on lpep_pickup_datetime? (Create a new table with this strategy)

Creating a partition and cluster table(Partition by lpep_pickup_datetime Cluster on PUlocationID)
CREATE OR REPLACE TABLE taxi_rides_ny.green_tripdata_partitoned_clustered
PARTITION BY DATE(lpep_pickup_datet)
CLUSTER BY PULocationID AS
SELECT * FROM taxi_rides_ny.green_tripdata 

Question5:
Write a query to retrieve the distinct PULocationID between lpep_pickup_datetime 06/01/2022 and 06/30/2022 (inclusive)
non partitioned non clustered table
query estimation 12.82 MB 
SELECT DISTINCT(PULocationID)
FROM taxi_rides_ny.green_tripdata
WHERE DATE(lpep_pickup_datet) BETWEEN '2022-06-01' AND '2022-06-30';


Write a query to retrieve the distinct PULocationID between lpep_pickup_datetime 06/01/2022 and 06/30/2022 (inclusive)
partitioned and clustered table
query estimation 1,12 MB 
SELECT DISTINCT(PULocationID)
FROM taxi_rides_ny.green_tripdata_partitoned_clustered
WHERE DATE(lpep_pickup_datet) BETWEEN '2022-06-01' AND '2022-06-30';

Question6:
Where is the data stored in the External Table you created?
GCP Bucket

Question7:
It is best practice in Big Query to always cluster your data:
False (Clustering is advised only when the tables have more than 1 GB data.

Question 8:
Write a SELECT count(*) query FROM the materialized table you created. How many bytes does it estimate will be read? Why?

This query will process 0 B when run.because the query is cached 

