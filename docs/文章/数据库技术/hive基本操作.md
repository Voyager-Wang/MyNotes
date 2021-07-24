修改hive table properties

alter table sbtest set tblproperties('arctic.enable'='false');





CREATE TABLE  `test_arctic_basic_dev_007` (
    id int,
    k int,
    c string,
    pad string,
    `time` timestamp,
    primary key (`id`) disable novalidate)
    PARTITIONED BY (
`dt` string)
stored as parquet;





