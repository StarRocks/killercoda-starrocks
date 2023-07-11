
Get a shell into our toolbox

Run `docker exec -it toolbox /bin/bash`{{exec}}

Then run the mysql client

Run `mysql -P9030 -h172.17.0.2 -uroot --prompt="StarRocks > "`{{exec}}

Create a database

`create database demo;`{{exec}}

Create the table

```
drop table taxi_green;
create table taxi_green (
     tpep_pickup_datetime DATETIME     
  , VendorID int                          
  , tpep_dropoff_datetime DATETIME   
  , passenger_count int                   
  , trip_distance float                   
  , PULocationID varchar(5)              
  , DOLocationID varchar(5)                
  , RatecodeID int                        
  , store_and_fwd_flag varchar            
  , payment_type int                       
  , fare_amount float                      
  , extra float                           
  , mta_tax float                          
  , improvement_surcharge float                         
  , tip_amount float                      
  , tolls_amount float                   
  , total_amount float                     
  , congestion_surcharge float            
  , airport_fee float            
)
ENGINE=OLAP
DUPLICATE KEY(`tpep_pickup_datetime`)
DISTRIBUTED BY HASH(`tpep_pickup_datetime`) BUCKETS 9;
```{{exec}}

Load in the data

Run 
```
alter system add broker local_load "172.17.0.2:8000";
load label xxxx1 (data infile("file:///tmp/green_tripdata_2023-01.parquet") into table taxi_green format as "parquet"(VendorID, tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, RatecodeID, trip_distance, store_and_fwd_flag, PULocationID, DOLocationID, payment_type, fare_amount, extra, mta_tax, tip_amount, tolls_amount, improvement_surcharge, total_amount, congestion_surcharge, airport_fee ) ) with broker local_load properties("timeout"="3600");
```{{exec}}



