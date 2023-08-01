
Get a shell into our toolbox

Run `docker exec -it toolbox /bin/bash`{{exec}}

Then run the mysql client

Run `mysql -P9030 -h172.17.0.2 -uroot --prompt="StarRocks > "`{{exec}}

Create a database

`create database demo;`{{exec}}

Create the table

`use demo;`{{exec}}

`Drop table user_behavior;`{{exec}}

`CREATE TABLE `user_behavior` ( `UserID` int(11) NULL COMMENT "", `ItemID` int(11) NULL COMMENT "", `CategoryID` int(11) NULL COMMENT "",  `BehaviorType` varchar(65533) NULL COMMENT "", `Timestamp` datetime NULL COMMENT "" ) ENGINE=OLAP DUPLICATE KEY(`UserID`) COMMENT "OLAP" DISTRIBUTED BY HASH(`UserID`) BUCKETS 1 ;`{{exec}}

Load in the data

Run 
`use demo;`{{exec}}

`load label ecommload1 (data infile("file:///tmp/user_behavior_sample_data.parquet") into table user_behavior format as "parquet" ) with broker allin1broker properties("timeout"="3600");`{{exec}}

To check the status run

`use demo;`{{exec}}

`show load;`{{exec}}

Query the data

`select * from user_behavior limit 10;`{{exec}}

Scenario 1: Higher level view of users completing this conversion path within 1800s

```
with tmp1 as (
  with tmp as (
    select 
      t.level as level, 
      count(UserID) as res 
    from 
      (
        select 
          UserID, 
          window_funnel(
            18000, 
            `Timestamp`, 
            0, 
            [BehaviorType = 'pv' , 
            BehaviorType = 'cart', 
            BehaviorType = 'buy' ]
          ) as level 
        from 
          user_behavior 
        where `Timestamp` >= '2017-12-02 00:00:00' 
            and `Timestamp` <= '2017-12-02 23:59:59'
        group by 
          UserID
      ) as t 
    where 
      t.level > 0 
    group by 
      t.level 
  ) 
  select 
    tmp.level, 
    sum(tmp.res) over (
      order by 
        tmp.level rows between current row 
        and unbounded following
    ) as retention 
  from 
    tmp
) 
select 
  tmp1.level, 
  tmp1.retention, 
  last_value(tmp1.retention) over(
    order by 
      tmp1.level rows between current row 
      and 1 following
  )/ tmp1.retention as retention_ratio 
from 
  tmp1;
```

Scenario 2: Examine the item IDs of the top ten products with the worst conversion rate from PV (page views) to buy.

```
with tmp1 as (
  with tmp as (
    select 
      ItemID, 
      t.level as level, 
      count(UserID) as res 
    from 
      (
        select 
          ItemID, 
          UserID, 
          window_funnel(
            1800, 
            timestamp, 
            0, 
            [BehaviorType = 'pv', 
            BehaviorType ='buy' ]
          ) as level 
        from 
          user_behavior 
        where timestamp >= '2017-12-02 00:00:00' 
            and timestamp <= '2017-12-02 23:59:59'
        group by 
          ItemID, 
          UserID
      ) as t 
    where 
      t.level > 0 
    group by 
      t.ItemID, 
      t.level 
  ) 
  select 
    tmp.ItemID, 
    tmp.level, 
    sum(tmp.res) over (
      partition by tmp.ItemID 
      order by 
        tmp.level rows between current row 
        and unbounded following
    ) as retention 
  from 
    tmp
) 
select 
  tmp1.ItemID, 
  tmp1.level, 
  tmp1.retention / last_value(tmp1.retention) over(
    partition by tmp1.ItemID 
    order by 
      tmp1.level desc rows between current row 
      and 1 following
  ) as retention_ratio 
from 
  tmp1 
order by 
  level desc, 
  retention_ratio 
limit 
  10;
```

Scenario 3: Would like to see the user paths of those who dropped off?

```
select 
  log.BehaviorType, 
  count(log.BehaviorType) 
from 
  (
    select 
      ItemID, 
      UserID, 
      window_funnel(
        1800, 
        timestamp, 
        0, 
        [BehaviorType = 'pv' , 
        BehaviorType = 'buy' ]
      ) as level 
    from 
      user_behavior 
    where timestamp >= '2017-12-02 00:00:00' 
        and timestamp <= '2017-12-02 23:59:59'
    group by 
      ItemID, 
      UserID
  ) as list 
  left join (
    select 
      UserID, 
      array_agg(BehaviorType) as BehaviorType 
    from 
      user_behavior 
    where 
      ItemID = 3563468 
      and timestamp >= '2017-12-02 00:00:00' 
      and timestamp <= '2017-12-02 23:59:59' 
    group by 
      UserID
  ) as log on list.UserID = log.UserID 
where 
  list.ItemID = 3563468
  and list.level = 1 
group by 
  log.BehaviorType 
order by 
  count(BehaviorType) desc;
```

