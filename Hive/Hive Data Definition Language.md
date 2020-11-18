###Hive Data Definition Language
#####1. Database

Hive中数据库的概念本质上仅仅是一个目录或者命名空间，创建一个数据库即意味着创建一个目录，数据库中的表将会以这个数据库目录的子目录形式存储。

```sql
-- 创建test库
create database if not exists test
        comment 'Test database'
        location '/user/hive/test'
        with dbproperties(
                'Date' = '2018-08-22',
                'Creator' = 'ylh'
                );

-- 显示test库
show databases like 'test*';

-- 查看test库信息
desc database test;
desc database extended test;

-- 使用test库
use test;

-- 删除test库
drop database if exists test restrict/cascade;
```



#####2. Hive Table Types

a. Managed Tables（管理表） - Default table type in hive
* 表的数据由hive管理，目录默认由hive.metastore.warehouse.dir配置（default /user/hive/warehouse）
* 如果表被删除，那么元数据和数据文件都会被删除
* 不便于被其它工具共享，比如Pig、Hbase等

b. External Tables（外部表）
* 数据文件不由Hive管理，不会被复制到hive的warehouse directory
* 如果表被删除，只是删除了Hive的元数据，数据文件不会被删除
* 便于被其他工具共享数据文件
* 创建外部表时，必须有”Location”子句

c. Temporary Tables （临时表）
* 只在当前session中有效
* 比如在复制数据时，需要一些中间表过渡，这很有效
* 数据目录由hive.exec.scratchdir配置
* 临时表不支持分区与创建索引
* 注意，如果临时表的名字与已经存在的表一样，那么当前session则不能访问被重名的表



#####3. Create Tables

a. Create table syntax（完整语法）
```sql
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name    -- (Note: TEMPORARY available in Hive 0.14.0 and later)
  [(col_name data_type [COMMENT col_comment], ... [constraint_specification])]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
  [SKEWED BY (col_name, col_name, ...)                  -- (Note: Available in Hive 0.10.0 and later)]
     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
     [STORED AS DIRECTORIES]
  [
   [ROW FORMAT row_format] 
   [STORED AS file_format]
     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]  -- (Note: Available in Hive 0.6.0 and later)
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]   -- (Note: Available in Hive 0.6.0 and later)
  [AS select_statement];   -- (Note: Available in Hive 0.5.0 and later; not supported for external tables)
```

b. Create external table（创建外部表）
```sql
CREATE EXTERNAL TABLE IF NOT EXISTS user (
  first_name STRING(64),
  last_name STRING(64),
  company_name STRING(64),
  address STRUCT<zip:INT, street:STRING>,
  country STRING(64),
  city STRING(32),
  state STRING(32),
  post INT,
  phone_nos ARRAY<STRING>,
  mail MAP<STRING, STRING>,
  web_address STRING(64)
  )
  ROW FORMAT DELIMITED 
    FIELDS TERMINATED BY ','
    COLLECTION ITEMS TERMINATED BY '\t'
    MAP KEYS TERMINATED BY ':'
    LINES TERMINATED BY '\n'
  STORED AS TEXTFILE;
 
LOAD DATA LOCAL INPATH '/home/user/User_Records.txt' OVERWRITE INTO TABLE user;
 
SELECT * FROM user;
```

c. Create Table As Select (CTAS)
CTAS has these restrictions（限制）:
* The target table cannot be a partitioned table（不能是分区表）
* The target table cannot be an external table（不能是外部表）
* The target table cannot be a list bucketing table（不能是分桶表）
```sql
CREATE TABLE new_key_value_store
   ROW FORMAT SERDE "org.apache.hadoop.hive.serde2.columnar.ColumnarSerDe"
   STORED AS RCFile
   AS
SELECT (key % 1024) new_key, concat(key, value) key_value_pair
FROM key_value_store
SORT BY new_key, key_value_pair;
```

d. Create Table Like
```sql
CREATE EXTERNAL TABLE IF NOT EXISTS test_db.user 
	LIKE default.user
        LOCATION '/user/hive/usertable'
        ;
 
INSERT OVERWRITE TABLE test_db.user SELECT * FROM default.user;
 
SELECT first_name, city, mail FROM test_db.user WHERE country='AU';
```

e.Bucketed Sorted Tables（分桶表）
```sql
CREATE TABLE page_view(viewTime INT, userid BIGINT,
     page_url STRING, referrer_url STRING,
     ip STRING COMMENT 'IP Address of the User')
 COMMENT 'This is the page view table'
 PARTITIONED BY(dt STRING, country STRING)
 CLUSTERED BY(userid) SORTED BY(viewTime) INTO 32 BUCKETS
 ROW FORMAT DELIMITED
   FIELDS TERMINATED BY '\001'
   COLLECTION ITEMS TERMINATED BY '\002'
   MAP KEYS TERMINATED BY '\003'
 STORED AS SEQUENCEFILE;
```
注意：
* 向分桶表插入数据时，需要配置hive.enforce.bucketing = true，类似于hive.exec.dynamic.partition=true
* 会自动设置reduce tasks的数量，和number of buckets（分桶数）相等

f. Skewed Tables（倾斜表）
```sql
CREATE TABLE list_bucket_single (
  key STRING, 
  value STRING
  )
SKEWED BY (key) ON (1,5,6) [STORED AS DIRECTORIES];

CREATE TABLE list_bucket_multiple (
  col1 STRING, 
  col2 int, 
  col3 STRING)
SKEWED BY (col1, col2) ON (('s1',1), ('s3',3), ('s13',13), ('s78',78)) [STORED AS DIRECTORIES];
```



#####4. Storage Formats

| **Storage Format**           | **Description**                                              |
| :--------------------------- | ------------------------------------------------------------ |
| STORED AS TEXTFILE           | Stored as plain text files. TEXTFILE is the default file format, unless the configuration parameter [hive.default.fileformat](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.default.fileformat) has a different setting.Use the DELIMITED clause to read delimited files.Enable escaping for the delimiter characters by using the 'ESCAPED BY' clause (such as ESCAPED BY '\') Escaping is needed if you want to work with data that can contain these delimiter characters. A custom NULL format can also be specified using the 'NULL DEFINED AS' clause (default is '\N'). |
| STORED AS SEQUENCEFILE       | Stored as compressed Sequence File.                          |
| STORED AS ORC                | Stored as [ORC file format](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-HiveQLSyntax). Supports ACID Transactions & Cost-based Optimizer (CBO). Stores column-level metadata. |
| STORED AS PARQUET            | Stored as Parquet format for the [Parquet](https://cwiki.apache.org/confluence/display/Hive/Parquet) columnar storage format in [Hive 0.13.0 and later](https://cwiki.apache.org/confluence/display/Hive/Parquet#Parquet-Hive0.13andlater); Use ROW FORMAT SERDE ... STORED AS INPUTFORMAT ... OUTPUTFORMAT syntax ... in [Hive 0.10, 0.11, or 0.12](https://cwiki.apache.org/confluence/display/Hive/Parquet#Parquet-Hive0.10-0.12). |
| STORED AS AVRO               | Stored as Avro format in [Hive 0.14.0 and later](https://issues.apache.org/jira/browse/HIVE-6806) (see [Avro SerDe](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe)). |
| STORED AS RCFILE             | Stored as [Record Columnar File](https://en.wikipedia.org/wiki/RCFile) format. |
| STORED AS JSONFILE           | Stored as Json file format in Hive 4.0.0 and later.          |
| STORED BY                    | Stored by a non-native table format. To create or link to a non-native table, for example a table backed by [HBase](https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration) or [Druid](https://cwiki.apache.org/confluence/display/Hive/Druid+Integration) or [Accumulo](https://cwiki.apache.org/confluence/display/Hive/AccumuloIntegration). See [StorageHandlers](https://cwiki.apache.org/confluence/display/Hive/StorageHandlers) for more information on this option. |
| INPUTFORMAT and OUTPUTFORMAT | in the file_format to specify the name of a corresponding InputFormat and OutputFormat class as a string literal.For example, 'org.apache.hadoop.hive.contrib.fileformat.base64.Base64TextInputFormat'. For LZO compression, the values to use are 'INPUTFORMAT "com.hadoop.mapred.DeprecatedLzoTextInputFormat" OUTPUTFORMAT "[org.apache.hadoop.hive.ql.io](http://org.apache.hadoop.hive.ql.io/).HiveIgnoreKeyTextOutputFormat"' (see [LZO Compression](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LZO)). |



#####5. Row Formats & SerDe

> Hive使用一个*inputformat*对象将输入流分割成记录，然后使用一个*outputformat*对象来将记录格式化为输出流（例如查询的输出结果），再使用一个SerDe在读数据是将记录解析成*列*，在写数据时将*列*编码成记录。
>

| **Storage Format** |                                                              |
| ------------------ | ------------------------------------------------------------ |
| STORED AS TEXTFILE | ROW FORMAT SERDE <br/>  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat' |
| STORED AS PARQUET  | ROW FORMAT SERDE <br/>  'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat' |
| STORED AS ORC      | ROW FORMAT SERDE <br/>  'org.apache.hadoop.hive.ql.io.orc.OrcSerde' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat' |



#####6. 参考

[Hive Database Commands](http://hadooptutorial.info/hive-database-commands/)
[Hive Table Creation Commands](http://hadooptutorial.info/hive-table-creation-commands/)
[Partitioning in Hive](http://hadooptutorial.info/partitioning-in-hive/)
[Bucketing In Hive](http://hadooptutorial.info/bucketing-in-hive/)
[Hive Data Definition Language](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)