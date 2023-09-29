
Get a shell into our toolbox

Run `docker exec -it toolbox /bin/bash`{{exec}}

Then run the mysql client

Run `mysql -P9030 -h172.17.0.2 -uroot --prompt="StarRocks > "`{{exec}}

Create a database

`create database demo;`{{exec}}

Create the table

```
drop table demo.taxi_green;
create table demo.taxi_green (
     lpep_pickup_datetime DATETIME     
  , VendorID int                          
  , lpep_dropoff_datetime DATETIME   
  , passenger_count int                   
  , trip_distance float                   
  , PULocationID string          
  , DOLocationID string             
  , RatecodeID int                        
  , store_and_fwd_flag string            
  , payment_type int                       
  , fare_amount float                      
  , extra float                           
  , mta_tax float                          
  , improvement_surcharge float                         
  , tip_amount float                      
  , tolls_amount float                   
  , total_amount float                     
  , congestion_surcharge float            
  , trip_type int         
)
ENGINE=OLAP
DUPLICATE KEY(lpep_pickup_datetime)
DISTRIBUTED BY HASH(lpep_pickup_datetime) BUCKETS 9;
```{{exec}}

Load in the data

Run

```
use demo;
load label taxiload1 (data infile("file:///tmp/green_tripdata_2023-01.parquet") into table taxi_green format as "parquet"(VendorID, tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, RatecodeID, trip_distance, store_and_fwd_flag, PULocationID, DOLocationID, payment_type, fare_amount, extra, mta_tax, tip_amount, tolls_amount, improvement_surcharge, total_amount, congestion_surcharge, airport_fee ) ) with broker allin1broker properties("timeout"="3600");
```{{exec}}

To check the status run

```
use demo;
show load;
```{{exec}}

Query the data

```
select * from taxi_green limit 10;
```{{exec}}

If you want to see the profile of the queries (eg. see how much time it takes to execute the stages of the query), read https://github.com/StarRocks/starrocks/discussions/29735 for more info. 
