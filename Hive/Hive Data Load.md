###Hive Data Load

##### 1. Excel篇

通常自己是不会弄出来个Excel需要导入Hive的，但是如果别人甩你一个Excel呢？

a) 打开excel文件，copy至Sublime Text里面，另存为txt文件；

b) 上传至hdfs指定目录;

c) 创建hive表，指定分隔符与location。

```sql
create external table tmp_ylh_excel_data(
	idea_id string,
	advertiser string,
	adplanid string,
	idea_name string,
	idea_segm string,
	url string,
	sindustry_code string,
	sindustry_name string,
	idea_show_cnt_day bigint,
	idea_show_cnt_total bigint,
	idea_update_date string,
	segm_update_date string,
	show_update_date string,
	ad_system string
)
row format delimited
fields terminated by '\t'
stored as textfile 
location '/user/dss/tmp/hivetables/yonglianghong/tmp_ylh_excel_data'
```



#####2. CSV篇

从Mysql里面导出数据，比如导成csv（这里使用Navicat），默认使用`,`分隔，且不可更改，如果字段值本身不含有`,`，就可以参考上面，只是需要更改指定的分隔符：

```sql
row format delimited
fields terminated by ','
```

如果字段本身含有`,`，则可以考虑修改字段间的分隔符比如为`\t`之类的，或者走上面Excel的思考。



关于CSV的DerSe：

```sql
CREATE TABLE csv_table(
a string,
b string ) 
ROW FORMAT SERDE 
'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES 
("separatorChar" = "\t",
"quoteChar"     = "'",
"escapeChar"    = "\\")  
STORED AS TEXTFILE;

separatorChar：分隔符
quoteChar：引号符
escapeChar：转义符
```

参考：https://blog.csdn.net/zengmingen/article/details/52636385



#####3. Mysql篇

可以用sqoop或者spark之类的工具：

```shell
sqoop import -D mapred.job.queue.name=root.dss.default --connect "jdbc:mysql://10.122.184.88:3335/dmp?zeroDateTimeBehavior=convertToNull&tinyInt1isBit=false" --table business_tag --username bus_data --password UOxqFDCZB --hive-import --hive-overwrite --hive-database dss --hive-table tmp_business_tag --m 1  --fields-terminated-by "\0001";
```



#####4. 其他

如果要跳过第1行，比如列头，可以在建表时设置如下参数：

```sql
tblproperties(
"skip.header.line.count"="n",  --跳过文件行首n行
"skip.footer.line.count"="n"   --跳过文件行尾n行 
)
```



示例：

```sql
create table test(
id int commet 'id',
name string commet '姓名'
) comment '测试表'
tblproperties(
"skip.header.line.count"="1",  --跳过文件行首1行
"skip.footer.line.count"="1"   --跳过文件行尾1行
);
```























