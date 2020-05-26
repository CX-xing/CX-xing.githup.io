#### QueryBuilder的：

#### QueryBuilder:

假设你有一个测量_h2o_feet_：

Supposing that you have a measurement _h2o_feet_:
```
> SELECT * FROM "h2o_feet"

name: h2o_feet
--------------
time                   level description      location       water_level
2015-08-18T00:00:00Z   below 3 feet           santa_monica   2.064
2015-08-18T00:00:00Z   between 6 and 9 feet   coyote_creek   8.12
[...]
2015-09-18T21:36:00Z   between 3 and 6 feet   santa_monica   5.066
2015-09-18T21:42:00Z   between 3 and 6 feet   santa_monica   4.938
```


#####的基本的SELECT语句

##### The basic SELECT statement


发出简单的SELECT语句

Issue simple select statements

```java
Query query = select().from(DATABASE,"h2o_feet");
```

```sqlite-psql
SELECT * FROM "h2o_feet"
```

从单次测量中选择特定的标签和领域

Select specific tags and fields from a single measurement

```java
Query query = select("level description","location","water_level").from(DATABASE,"h2o_feet");
```


```sqlite-psql
SELECT "level description",location,water_level FROM h2o_feet;
```

从一个单一的测量选择特定的标签和领域，并提供他们的标识符类型

Select specific tags and fields from a single measurement, and provide their identifier type


```java
Query query = select().column("\"level description\"::field").column("\"location\"::tag").column("\"water_level\"::field").from(DATABASE,"h2o_feet");
```


```sqlite-psql
SELECT "level description"::field,"location"::tag,"water_level"::field FROM h2o_feet;
```

选择从单一测量所有领域

Select all fields from a single measurement

```java
Query query = select().raw("*::field").from(DATABASE,"h2o_feet");
```


```sqlite-psql
SELECT *::field FROM h2o_feet;
```

从测量选择一个特定的领域，并执行基本的算术

Select a specific field from a measurement and perform basic arithmetic


```java
Query query = select().op(op(cop("water_level",MUL,2),"+",4)).from(DATABASE,"h2o_feet");
```


```sqlite-psql
SELECT (water_level * 2) + 4 FROM h2o_feet;
```

从以上的测量选择所有数据

Select all data from more than one measurement

```java
Query query = select().from(DATABASE,"\"h2o_feet\",\"h2o_pH\"");
```


```sqlite-psql
SELECT * FROM "h2o_feet","h2o_pH";
```

从一个完全合格的测量选择所有数据

Select all data from a fully qualified measurement

```java
Query query = select().from(DATABASE,"\"NOAA_water_database\".\"autogen\".\"h2o_feet\"");
```

```sqlite-psql
SELECT * FROM "NOAA_water_database"."autogen"."h2o_feet";
```

具有特定领域的键值选择数据

Select data that have specific field key-values

```java
Query query = select().from(DATABASE,"h2o_feet").where(gt("water_level",8));
```

```sqlite-psql
SELECT * FROM h2o_feet WHERE water_level > 8;
```

选择具有特定字符串字段键值数据

Select data that have a specific string field key-value

```java
Query query = select().from(DATABASE,"h2o_feet").where(eq("level description","below 3 feet"));
```

```sqlite-psql
SELECT * FROM h2o_feet WHERE "level description" = 'below 3 feet';
```

选择具有特定领域的键值，并执行基本的算术数据

Select data that have a specific field key-value and perform basic arithmetic

```java
Query query = select().from(DATABASE,"h2o_feet").where(gt(cop("water_level",ADD,2),11.9));
```

```sqlite-psql
SELECT * FROM h2o_feet WHERE (water_level + 2) > 11.9;
```

选择具有特定标签键值数据

Select data that have a specific tag key-value

```java
Query query = select().column("water_level").from(DATABASE,"h2o_feet").where(eq("location","santa_monica"));
```

```sqlite-psql
SELECT water_level FROM h2o_feet WHERE location = 'santa_monica';
```

具有特定领域关键值和标签键值选择数据

Select data that have specific field key-values and tag key-values

```java
Query query = select().column("water_level").from(DATABASE,"h2o_feet")
                .where(neq("location","santa_monica"))
                .andNested()
                .and(lt("water_level",-0.59))
                .or(gt("water_level",9.95))
                .close();
```

```sqlite-psql
SELECT water_level FROM h2o_feet WHERE location <> 'santa_monica' AND (water_level < -0.59 OR water_level > 9.95);
```

具有特定时间戳选择数据

Select data that have specific timestamps

```java
Query query = select().from(DATABASE,"h2o_feet")
                .where(gt("time",subTime(7,DAY)));
```

```sqlite-psql
SELECT * FROM h2o_feet WHERE time > now() - 7d;
```

##### GROUP BY子句

##### The GROUP BY clause

集团查询结果通过一个单一的标签

Group query results by a single tag

```java
Query query = select().mean("water_level").from(DATABASE,"h2o_feet") .groupBy("location");
```

```sqlite-psql
SELECT MEAN(water_level) FROM h2o_feet GROUP BY location;
```

集团查询结果由一个以上的标签

Group query results by more than one tag

```java
Query query = select().mean("index").from(DATABASE,"h2o_feet")
                .groupBy("location","randtag");
```

```sqlite-psql
SELECT MEAN(index) FROM h2o_feet GROUP BY location,randtag;
```

集团查询结果的所有标签

Group query results by all tags

```java
Query query = select().mean("index").from(DATABASE,"h2o_feet")
                .groupBy(raw("*"));
```

```sqlite-psql
SELECT MEAN(index) FROM h2o_feet GROUP BY *;
```

** GROUP BY的时间间隔**

**GROUP BY time intervals**

集团查询结果成12分钟一班

Group query results into 12 minute intervals

```java
Query query = select().count("water_level").from(DATABASE,"h2o_feet")
                .where(eq("location","coyote_creek"))
                .and(gte("time","2015-08-18T00:00:00Z"))
                .and(lte("time","2015-08-18T00:30:00Z'"))
                .groupBy(time(12l,MINUTE));
```

```sqlite-psql
SELECT COUNT(water_level) FROM h2o_feet WHERE location = 'coyote_creek' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z'' GROUP BY time(12m);
```

集团查询结果分为12米分钟的间隔，并通过代码键

Group query results into 12 minutes intervals and by a tag key

```java
        Query query = select().count("water_level").from(DATABASE,"h2o_feet")
                .where()
                .and(gte("time","2015-08-18T00:00:00Z"))
                .and(lte("time","2015-08-18T00:30:00Z'"))
                .groupBy(time(12l,MINUTE),"location");
```

```sqlite-psql
SELECT COUNT(water_level) FROM h2o_feet WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z'' GROUP BY time(12m),location;
```

#####高级GROUP BY时间（）语法

##### Advanced GROUP BY time() syntax

组查询结果分为18个分钟的时间间隔和移位预设时间边界向前

Group query results into 18 minute intervals and shift the preset time boundaries forward

```java
Query query = select().mean("water_level").from(DATABASE,"h2o_feet")
                .where(eq("location","coyote_creek"))
                .and(gte("time","2015-08-18T00:06:00Z"))
                .and(lte("time","2015-08-18T00:54:00Z"))
                .groupBy(time(18l,MINUTE,6l,MINUTE));
```

```sqlite-psql
SELECT MEAN(water_level) FROM h2o_feet WHERE location = 'coyote_creek' AND time >= '2015-08-18T00:06:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(18m,6m);
```

组查询结果分为12个分钟的时间间隔和移位预设时间界限回

Group query results into 12 minute intervals and shift the preset time boundaries back

```java
Query query = select().mean("water_level").from(DATABASE,"h2o_feet")
                .where(eq("location","coyote_creek"))
                .and(gte("time","2015-08-18T00:06:00Z"))
                .and(lte("time","2015-08-18T00:54:00Z"))
                .groupBy(time(18l,MINUTE,-12l,MINUTE));
```

```sqlite-psql
SELECT MEAN(water_level) FROM h2o_feet WHERE location = 'coyote_creek' AND time >= '2015-08-18T00:06:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(18m,-12m);
```

##### GROUP BY的时间间隔和填充（）

##### GROUP BY time intervals and fill()

```java
Query select = select()
                .column("water_level")
                .from(DATABASE, "h2o_feet")
                .where(gt("time", op(ti(24043524l, MINUTE), SUB, ti(6l, MINUTE))))
                .groupBy("water_level")
                .fill(100);
```

```sqlite-psql
SELECT water_level FROM h2o_feet WHERE time > 24043524m - 6m GROUP BY water_level fill(100);"
```

##### INTO子句

##### The INTO clause

重命名数据库

Rename a database

```java
Query select = select()
                .into("\"copy_NOAA_water_database\".\"autogen\".:MEASUREMENT")
                .from(DATABASE, "\"NOAA_water_database\".\"autogen\"./.*/")
                .groupBy(new RawText("*"));
```

```sqlite-psql
SELECT * INTO "copy_NOAA_water_database"."autogen".:MEASUREMENT FROM "NOAA_water_database"."autogen"./.*/ GROUP BY *;
```

编写一个查询的结果来测量

Write the results of a query to a measurement

```java
Query select = select().column("water_level").into("h2o_feet_copy_1").from(DATABASE,"h2o_feet").where(eq("location","coyote_creek"));
```

```sqlite-psql
SELECT water_level INTO h2o_feet_copy_1 FROM h2o_feet WHERE location = 'coyote_creek';
```

写汇总结果的测量

Write aggregated results to a measurement

```java
Query select = select()
                .mean("water_level")
                .into("all_my_averages")
                .from(DATABASE,"h2o_feet")
                .where(eq("location","coyote_creek"))
                .and(gte("time","2015-08-18T00:00:00Z"))
                .and(lte("time","2015-08-18T00:30:00Z"))
                .groupBy(time(12l,MINUTE));
```

```sqlite-psql
SELECT MEAN(water_level) INTO all_my_averages FROM h2o_feet WHERE location = 'coyote_creek' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z' GROUP BY time(12m);
```

对于多于一个的测量写入聚合结果，以不同的数据库（用逆向引用下采样）

Write aggregated results for more than one measurement to a different database (downsampling with backreferencing)

```java
Query select = select()
                .mean(raw("*"))
                .into("\"where_else\".\"autogen\".:MEASUREMENT")
                .fromRaw(DATABASE, "/.*/")
                .where(gte("time","2015-08-18T00:00:00Z"))
                .and(lte("time","2015-08-18T00:06:00Z"))
                    .groupBy(time(12l,MINUTE));
```

```sqlite-psql
SELECT MEAN(*) INTO "where_else"."autogen".:MEASUREMENT FROM /.*/ WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:06:00Z' GROUP BY time(12m);
```

##### ORDER BY时间DESC

##### ORDER BY time DESC

首先返回最新点

Return the newest points first

```java
Query select = select().from(DATABASE,"h2o_feet")
                .where(eq("location","santa_monica"))
                .orderBy(desc());
```

```sqlite-psql
SELECT * FROM h2o_feet WHERE location = 'santa_monica' ORDER BY time DESC;
```

首先返回最新的点，包括GROUP BY时间（）子句

Return the newest points first and include a GROUP BY time() clause

```java
Query select = select().mean("water_level")
                .from(DATABASE,"h2o_feet")
                .where(gte("time","2015-08-18T00:00:00Z"))
                .and(lte("time","2015-08-18T00:42:00Z"))
                .groupBy(time(12l,MINUTE))
                .orderBy(desc());
```

```sqlite-psql
SELECT MEAN(water_level) FROM h2o_feet WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:42:00Z' GROUP BY time(12m) ORDER BY time DESC;
```

##### LIMIT子句

##### The LIMIT clause

限制返回的点数

Limit the number of points returned

```java
Query select = select("water_level","location")
                .from(DATABASE,"h2o_feet").limit(3);
```

```sqlite-psql
SELECT water_level,location FROM h2o_feet LIMIT 3;
```

限制返回的点数，并包括一个GROUP BY子句

Limit the number points returned and include a GROUP BY clause

```java
Query select = select().mean("water_level")
                .from(DATABASE,"h2o_feet")
                .where()
                .and(gte("time","2015-08-18T00:00:00Z"))
                .and(lte("time","2015-08-18T00:42:00Z"))
                .groupBy(raw("*"),time(12l,MINUTE))
                .limit(2);
```

```sqlite-psql
SELECT MEAN(water_level) FROM h2o_feet WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:42:00Z' GROUP BY *,time(12m) LIMIT 2;
```

#####的SLIMIT条款

##### The SLIMIT clause

限制返回系列的数

Limit the number of series returned

```java
Query select = select().column("water_level")
                .from(DATABASE,"h2o_fleet")
                .groupBy(raw("*"))
                .sLimit(1);
```

```sqlite-psql
SELECT water_level FROM "h2o_feet" GROUP BY * SLIMIT 1

```
限制返回系列号，并包括一个GROUP BY时间（）子句

Limit the number of series returned and include a GROUP BY time() clause

```java
Query select = select().column("water_level")
                .from(DATABASE,"h2o_feet")
                .where()
                .and(gte("time","2015-08-18T00:00:00Z"))
                .and(lte("time","2015-08-18T00:42:00Z"))
                .groupBy(raw("*"),time(12l,MINUTE))
                .sLimit(1);
```

```sqlite-psql
SELECT water_level FROM h2o_feet WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:42:00Z' GROUP BY *,time(12m) SLIMIT 1;
```

#####的OFFSET子句

##### The OFFSET clause

分页点

Paginate points

```java
Query select = select("water_level","location").from(DATABASE,"h2o_feet").limit(3,3);
```

```sqlite-psql
SELECT water_level,location FROM h2o_feet LIMIT 3 OFFSET 3;
```

#####的S偏移条款

##### The SOFFSET clause

分页系列，并包括所有条款

Paginate series and include all clauses

```java
Query select = select().mean("water_level")
                .from(DATABASE,"h2o_feet")
                .where()
                .and(gte("time","2015-08-18T00:00:00Z"))
                .and(lte("time","2015-08-18T00:42:00Z"))
                .groupBy(raw("*"),time(12l,MINUTE))
                .orderBy(desc())
                .limit(2,2)
                .sLimit(1,1);
```

```sqlite-psql
SELECT MEAN(water_level) FROM h2o_feet WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:42:00Z' GROUP BY *,time(12m) ORDER BY time DESC LIMIT 2 OFFSET 2 SLIMIT 1 SOFFSET 1;
```

#####的时区条款

##### The Time Zone clause

返回UTC芝加哥的时区偏移

Return the UTC offset for Chicago’s time zone

```java
Query select = select()
                .column("test1")
                .from(DATABASE, "h2o_feet")
                .groupBy("test2", "test3")
                .sLimit(1)
                .tz("America/Chicago");
```

```sqlite-psql
SELECT test1 FROM foobar GROUP BY test2,test3 SLIMIT 1 tz('America/Chicago');
```

#####时语法

##### Time Syntax

指定与RFC3339的日期 - 时间字符串的时间范围

Specify a time range with RFC3339 date-time strings

```java
Query select = select().column("water_level")
                .from(DATABASE,"h2o_feet")
                .where(eq("location","santa_monica"))
                .and(gte("time","2015-08-18T00:00:00.000000000Z"))
                .and(lte("time","2015-08-18T00:12:00Z"));
```

```sqlite-psql
SELECT water_level FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-08-18T00:00:00.000000000Z' AND time <= '2015-08-18T00:12:00Z';
```

指定与第二精度划时代的时间戳的时间范围

Specify a time range with second-precision epoch timestamps

```java
Query select = select().column("water_level")
                .from(DATABASE,"h2o_feet")
                .where(eq("location","santa_monica"))
                .and(gte("time",ti(1439856000l,SECOND)))
                .and(lte("time",ti(1439856720l,SECOND)));
```

```sqlite-psql
SELECT water_level FROM h2o_feet WHERE location = 'santa_monica' AND time >= 1439856000s AND time <= 1439856720s;
```

上RFC3339样时间串执行基本的算术

Perform basic arithmetic on an RFC3339-like date-time string

```java
Query select = select().column("water_level")
                .from(DATABASE,"h2o_feet")
                .where(eq("location","santa_monica"))
                .and(gte("time",op("2015-09-18T21:24:00Z",SUB,ti(6l,MINUTE))));
```

```sqlite-psql
SELECT water_level FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-09-18T21:24:00Z' - 6m;
```

上一个划时代的时间戳进行基本算术运算

Perform basic arithmetic on an epoch timestamp

```java
Query select = select().column("water_level")
                .from(DATABASE,"h2o_feet")
                .where(eq("location","santa_monica"))
                .and(gte("time",op(ti(24043524l,MINUTE),SUB,ti(6l,MINUTE))));
```

```sqlite-psql
SELECT water_level FROM h2o_feet WHERE location = 'santa_monica' AND time >= 24043524m - 6m;
```

指定与相对于时间的时间范围内

Specify a time range with relative time

```java
Query select = select().column("water_level")
                .from(DATABASE,"h2o_feet")
                .where(eq("location","santa_monica"))
                .and(gte("time",subTime(1l,HOUR)));
```

```sqlite-psql
SELECT water_level FROM h2o_feet WHERE location = 'santa_monica' AND time >= now() - 1h;
```

＃＃＃＃＃ 常用表达

##### Regular expressions

使用正则表达式在SELECT子句中指定字段键和标签键

Use a regular expression to specify field keys and tag keys in the SELECT clause


```java
Query select = select().regex("l").from(DATABASE,"h2o_feet").limit(1);
```

```sqlite-psql
SELECT /l/ FROM h2o_feet LIMIT 1;
```

使用正则表达式与SELECT子句中的函数指定字段键

Use a regular expression to specify field keys with a function in the SELECT clause

```java
Query select = select().regex("l").distinct().from(DATABASE,"h2o_feet").limit(1);
```

```sqlite-psql
SELECT DISTINCT /l/ FROM h2o_feet LIMIT 1;
```
使用正则表达式在FROM子句指定的测量

Use a regular expression to specify measurements in the FROM clause

```java
Query select = select().mean("degrees").fromRaw(DATABASE,"/temperature/");
```

```sqlite-psql
SELECT MEAN(degrees) FROM /temperature/;
```

使用正则表达式来指定在WHERE子句中的字段值

Use a regular expression to specify a field value in the WHERE clause

```java
Query select = select().regex("/l/").from(DATABASE,"h2o_feet").where(regex("level description","/between/")).limit(1);
```

```sqlite-psql
SELECT /l/ FROM h2o_feet WHERE "level description" =~ /between/ LIMIT 1;
```

使用正则表达式来指定GROUP BY子句中的代码键

Use a regular expression to specify tag keys in the GROUP BY clause

```java
Query select = select().regex("/l/").from(DATABASE,"h2o_feet").where(regex("level description","/between/")).groupBy(raw("/l/")).limit(1);
```

```sqlite-psql
SELECT /l/ FROM h2o_feet WHERE "level description" =~ /between/ GROUP BY /l/ LIMIT 1;
```

没有直接执行功能可以通过原始表达式的支持

Function with no direct implementation can be supported by raw expressions

```java
Query select = select().raw("an expression on select").from(dbName, "cpu").where("an expression as condition");
```

```sqlite-psql
SELECT an expression on select FROM h2o_feet WHERE an expression as condition;
```

