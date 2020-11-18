#### dmp

1. dmp维度表



a. 标签市场管理表

| 字段英文名        | 字段中文名     | 字段类型     | 取值说明                  | 是否主键 |
| ----------------- | -------------- | ------------ | ------------------------- | -------- |
| tag_category_id   | 标签分类id     | int          |                           | Y        |
| tag_category_code | 标签分类标识   | varchar(30)  |                           |          |
| tag_category_name | 标签分类名称   | varchar(128) |                           |          |
| tag_metatable     | 标签分类元表   | varchar(60)  |                           |          |
| tag_hivetable     | 标签分类hive表 | varchar(60)  |                           |          |
| status            | 标签分类状态   | tinyint(4)   | 1为启用，0为测试，2为删除 |          |
| desc              | 描述           | varchar(255) |                           |          |
| create_time       | 创建时间       | timestamp    |                           |          |
| update_time       | 更新时间       | timestamp    |                           |          |

```sql
create table `dmp_dim_tag_market`(
	`tag_category_id` int NOT NULL PRIMARY KEY AUTO_INCREMENT COMMENT '标签分类自增id',
	`tag_category_code` varchar(30) NOT NULL COMMENT '标签分类标识',
	`tag_category_name` varchar(128) NOT NULL COMMENT '标签分类名称',
	`tag_metatable` varchar(60) NOT NULL COMMENT '标签分类元表，每个分类对应一个mysql表',
	`tag_hivetable` varchar(60) NOT NULL COMMENT '标签分类hive表',
	`status` tinyint(1) NOT NULL DEFAULT '0' COMMENT '标签分类启动状态，1为启用，0为测试，2为删除',
	`desc` varchar(255) NOT NULL DEFAULT '' COMMENT '描述',
	`create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
	`update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间'
	)ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT 'dmp标签市场管理表'
```



b. xxx标签表

| 字段英文名    | 字段中文名     | 字段类型     | 取值说明                            | 是否主键 |
| ------------- | -------------- | ------------ | ----------------------------------- | -------- |
| id            | 标签id         | int(11)      | 自增                                | Y        |
| name          | 标签名称       | varchar(128) |                                     |          |
| level         | 标签层级       | tinyint(4)   | 1为一级，2为二级，3为三级，以此类推 |          |
| parent_id     | 标签父级id     | int(11)      | 一级标签id为0                       |          |
| category_code | 标签分类标识   | varchar(12)  |                                     |          |
| status        | 标签状态       | tinyint(4)   | 1为启用，0为测试，2为删除           |          |
| uv            | 标签覆盖用户数 | bigint(21)   |                                     |          |
| score         | 标签质量分     | float(3,2)   |                                     |          |
| cover_degree  | 标签覆盖度     | float(3,2)   |                                     |          |
| use_degree    | 标签使用热度   | float(3,2)   |                                     |          |
| desc          | 描述           | varchar(255) |                                     |          |
| create_time   | 创建时间       | timestamp    |                                     |          |
| update_time   | 更新时间       | timestamp    |                                     |          |

```sql
CREATE TABLE `dmp_dim_xxx_tag` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '标签id',
  `name` varchar(128) NOT NULL COMMENT '标签名称',
  `level` tinyint(4) NOT NULL COMMENT '标签层级，1为一级，2位二级，3为三级，以此类推',
  `parent_id` int(11) NOT NULL COMMENT '父级id，一级标签id为0',
  `category_code` varchar(12) NOT NULL DEFAULT 'xxx' COMMENT '标签分类标识',
  `status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '标签状态,1为启用,0为测试,2为删除',
  `uv` bigint(21) DEFAULT '0' COMMENT '标签覆盖用户数',
  `score` float(3,2) DEFAULT NULL COMMENT '标签质量分',
  `cover_degree` float(3,2) DEFAULT NULL COMMENT '标签覆盖程度',
  `use_degree` float(3,2) DEFAULT NULL COMMENT '标签使用热度',
  `desc` varchar(255) NOT NULL COMMENT '标签描述',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)
  CONSTRAINT `category_code` FOREIGN KEY (`category_code`) REFERENCES `dmp_dim_tag_market` (`tag_category_code`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='xxx标签元表'
```



c. 标签维度总表(供前端和标签数据汇总用，也是将标签再次统一编码)

| 字段英文名    | 字段中文名   | 字段类型 | 取值说明                  | 是否主键 |
| ------------- | ------------ | -------- | ------------------------- | -------- |
| tag_code      | 标签编码     |          | tag_type+tag_id           | Y        |
| tag_id        | 标签id       |          | 与源表保持一致            |          |
| tag_name      | 标签名称     |          | 与源表保持一致            |          |
| tag_level     | 标签层级     |          | 与源表保持一致            |          |
| tag_type_code | 标签类型     |          | 与tag_category_code一致？ |          |
| tag_type_name | 标签类型名称 |          |                           |          |
| tag_uv        | 标签覆盖uv   |          |                           |          |
| create_time   | 创建时间     |          |                           |          |
| update_time   | 更新时间     |          |                           |          |



需要思考的问题：

> - province/city 可以分为1、2级，那city_level呢？
>
>   city_level可以理解为基于原始标签的新标签，与province/city本身已经没有关系了。
>
> - tag_type_code 不能带下划线？ 
>
>   如果tag_code采用下划线拼接，那么tag_type_code，就不能再包含下划线，有两种方法：
>
>   第一种是：将tag_type_code再次编码，比如hash，这样tag_type_code值本身就没有什么限制，但是在tag_code这里丧失了可读性；
>
>   第二种是：将tag_type_code里面的下划线去掉，tag_type_code也可以随意，丧失了一定可读性。
>
> - tag_code应该如何编码：
>
>   ~~接口的使用是tag_type_code+tag_id；(历史是这样)~~
>
>   画像计算的使用是tag_type_code+level+id；
>
>   其实只要统一就好，保留type+level+id这样三重信息；



延伸问题：

>应当如何给数据编码？兼保留可读性、易解析、保留一定的信息。



2. dmp画像计算

Hive：

a) 用户字典表：user_id <-> id(int)

b) crowd的bitmap表：<crowd, bitmap> 

c) portrait标签bitmap表：<tag_id, bitmap>

> 其实这里人群也是标签的概念，为何不将人群与portrait标签表放在一起？
>
> 因为crowd与portrait的更新频率可能不一致，在hive表不能更新的情况下，分开可能合适一点。



问题：似乎必须更新频率保持一致，如果crowd需要T+1更新，那么其他也得要T+1更新





