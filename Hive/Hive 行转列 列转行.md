### Hive 行转列 列转行

#####1. 行转列

*语法：*

```sql
LATERAL VIEW explode(expression) tableAlias AS columnAlias (',' columnAlias)*
```

*示例：*

```sql
-- column is array
select t,c from table lateral view explode(column) ctable as c
-- column is map
select t,k,v from table lateral view explode(column) ctable as k,v
```



*lateral view?*

> Lateral view is used in conjunction with user-defined table generating functions such as `explode()`. As mentioned in [Built-in Table-Generating Functions](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-Built-inTable-GeneratingFunctions(UDTF)), a UDTF generates zero or more output rows for each input row. A lateral view first applies the UDTF to each row of base table and then joins resulting output rows to the input rows to form a virtual table having the supplied table alias.

从上面可以知道，lateral view是与udtf配合使用，udtf是将一行变成多行，而lateral view则将udtf的output rows与input rows结合（join）在一起，以满足更好的应用。



*explode?*

> Using the syntax "SELECT udtf(col) AS colAlias..." has a few limitations:
>
> - No other expressions are allowed in SELECT
>   - SELECT pageid, explode(adid_list) AS myCol... is not supported
> - UDTF's can't be nested
>   - SELECT explode(explode(adid_list)) AS myCol... is not supported
> - GROUP BY / CLUSTER BY / DISTRIBUTE BY / SORT BY is not supported
>   - SELECT explode(adid_list) AS myCol ... GROUP BY myCol is not supported

从这里可以看到explode本身有这些限制，因此通常需要与lateral view一起使用。



*常用用法：*

a) 对于字符串，可以先`split(str,',')`，得到`array<string>`，再结合`lateral view explode`使用



#####2. 列转行

*语法：*

| Return Type | Name(Signature)   | Description                                                  |
| :---------- | :---------------- | :----------------------------------------------------------- |
| array       | collect_set(col)  | Returns a set of objects with duplicate elements eliminated. |
| array       | collect_list(col) | Returns a list of objects with duplicates. (As of Hive [0.13.0](https://issues.apache.org/jira/browse/HIVE-5294).) |



*常用用法：*

a) 配合concat_ws(',',collect_set(column))，可以实现将列转成array，再转成string



##### 3. 参考

[Hive Operators and User-Defined Functions (UDFs)](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-CollectionFunctions)

[LanguageManual LateralView](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LateralView)



