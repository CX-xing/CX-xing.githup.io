＃手册

# Manual

＃＃ 快速开始

## Quick start
下面的代码是类似的README.md文件，但去掉注释和行编号，以便更好的参考找到了一个。

The code below is similar to the one found on the README.md file but with comments removed and rows numbered for better reference.

```Java
final String serverURL = "http://127.0.0.1:8086", username = "root", password = "root";
final InfluxDB influxDB = InfluxDBFactory.connect(serverURL, username, password);  // (1)

String databaseName = "NOAA_water_database";
influxDB.query(new Query("CREATE DATABASE " + databaseName));
influxDB.setDatabase(databaseName);                                       // (2)

String retentionPolicyName = "one_day_only";
influxDB.query(new Query("CREATE RETENTION POLICY " + retentionPolicyName 
		+ " ON " + databaseName + " DURATION 1d REPLICATION 1 DEFAULT"));
influxDB.setRetentionPolicy(retentionPolicyName);                         // (3)

influxDB.enableBatch(BatchOptions.DEFAULTS);                              // (4)

influxDB.write(Point.measurement("h2o_feet")                              // (5)
	.time(System.currentTimeMillis(), TimeUnit.MILLISECONDS)
	.tag("location", "santa_monica")
	.addField("level description", "below 3 feet")
	.addField("water_level", 2.064d)
	.build());

influxDB.write(Point.measurement("h2o_feet")                              // (5)
	.tag("location", "coyote_creek")
	.addField("level description", "between 6 and 9 feet")
	.addField("water_level", 8.12d)
	.build());                                                            // (6)

Thread.sleep(5_000L);                                                     // (7)

QueryResult queryResult = influxDB.query(new Query("SELECT * FROM h2o_feet"));

System.out.println(queryResult);
// It will print something like:
// QueryResult [results=[Result [series=[Series [name=h2o_feet, tags=null, 
// 		columns=[time, level description, location, water_level],
//		values=[
// 			[2020-03-22T20:50:12.929Z, below 3 feet, santa_monica, 2.064], 
//			[2020-03-22T20:50:12.929Z, between 6 and 9 feet, coyote_creek, 8.12]
//		]]], error=null]], error=null]

influxDB.close();                                                         // (8)
```

###连接到InfluxDB

### Connecting to InfluxDB
（1）`InfluxDB`客户端是线程安全的，我们的建议是让每个应用程序的单一实例，并在可能重新使用它。每个`InfluxDB`实例保存多个数据结构，包括用于管理不同的存储池，如HTTP客户端的读取和写入。

(1) The `InfluxDB` client is thread-safe and our recommendation is to have a single instance per application and reuse it, when possible. Every `InfluxDB` instance keeps multiple data structures, including those used to manage different pools like HTTP clients for reads and writes.

这是可能的读取或写入多个InfluxDB数据库只是一个客户端，即使这将在以后这里显示有。

It's possible to have just one client even when reading or writing to multiple InfluxDB databases and this will be shown later here.


###设置一个默认的数据库（可选）

### Setting a default database (optional)
（2）如果您没有查询与单一`InfluxDB`客户不同的数据库，它可以设置一个默认的数据库名称，并从该`InfluxDB`客户端的所有查询（读取和写入）将会对默认数据库执行。

(2) If you are not querying different databases with a single `InfluxDB` client, it's possible to set a default database name and all queries (reads and writes) from this `InfluxDB` client will be executed against the default database.

如果我们只注释掉线（2），那么所有的读取和写入查询会失败。为了避免这种情况，我们需要通过数据库名作为参数传递给`BatchPoints`（写）和`Query`（读取）。例如：

If we only comment out the line (2) then all reads and writes queries would fail. To avoid this, we need to pass the database name as parameter to `BatchPoints` (writes) and to `Query` (reads). For example:

```Java
// ...
String databaseName = "NOAA_water_database";
// influxDB.setDatabase() won't be called...
String retentionPolicyName = "one_day_only";
// ...

BatchPoints batchPoints = BatchPoints.database(databaseName).retentionPolicy(retentionPolicyName).build();

batchPoints.point(Point.measurement("h2o_feet")
	.time(System.currentTimeMillis(), TimeUnit.MILLISECONDS)
	.tag("location", "santa_monica")
	.addField("level description", "below 3 feet")
	.addField("water_level", 2.064d)
	.build());

// ...
influxDB.write(batchPoints);
// ...
QueryResult queryResult = influxDB.query(new Query("SELECT * FROM h2o_feet"), databaseName);
// ...
influxDB.close();
```

这是可能的使用都在同一时间接近：使用`influxDB.setDatabase`设置一个默认的数据库和读/写传递`databaseName`作为参数。在这种情况下，`databaseName`传递为参数将被使用。

It's possible to use both approaches at the same time: set a default database using `influxDB.setDatabase` and read/write passing a `databaseName` as parameter. On this case, the `databaseName` passed as parameter will be used.


###设置一个默认保留策略（可选）

### Setting a default retention policy (optional)
（3）TODO：如设置一个默认的数据库，在这里解释它如何与RP工作。

(3) TODO: like setting a default database, explain here how it works with RP.


###启用批量写入

### Enabling batch writes
（4）TODO：解释关于BatchOption参数：

(4) TODO: explanation about BatchOption parameters:

```Java
  // default values here are consistent with Telegraf
  public static final int DEFAULT_BATCH_ACTIONS_LIMIT = 1000;
  public static final int DEFAULT_BATCH_INTERVAL_DURATION = 1000;
  public static final int DEFAULT_JITTER_INTERVAL_DURATION = 0;
  public static final int DEFAULT_BUFFER_LIMIT = 10000;
  public static final TimeUnit DEFAULT_PRECISION = TimeUnit.NANOSECONDS;
```

####配置批次写入的抖动间隔

#### Configuring the jitter interval for batch writes

当使用大量influxdb Java的客户端的针对单个服务器，可能会发生的所有客户端

When using large number of influxdb-java clients against a single server it may happen that all the clients
将在同一时间提交他们的缓冲点，并可能服务器过载。这通常发生

will submit their buffered points at the same time and possibly overloading the server. This is usually happening
例如作为云成员主持大型集群网络 - 当所有的客户端在启动一次。

when all the clients are started at once - for instance as members of cloud hosted large cluster networks.  
如果所有客户端具有相同的flushDuration一套这种情况会周期性地重复。

If all the clients have the same flushDuration set this situation will repeat periodically.

为了解决这种情况的influxdb-Java提供了一个选项，由随机时间间隔，以便抵消flushDuration

To solve this situation the influxdb-java offers an option to offset the flushDuration by a random interval so that
客户端将刷新它们在不同的时间间隔缓冲区：

the clients will flush their buffers in different intervals:

```Java
influxDB.enableBatch(BatchOptions.DEFAULTS.jitterDuration(500));
```

####错误与批量写入处理

#### Error handling with batch writes
随着配料启用客户端提供了两种策略如何应对由InfluxDB服务器引发的错误。

With batching enabled the client provides two strategies how to deal with errors thrown by the InfluxDB server.

1.“单脉冲”写 - 对失败的写入请求到服务器InfluxDB一个错误被报告给使用上述装置的客户端。

   1. 'One shot' write - on failed write request to InfluxDB server an error is reported to the client using the means mentioned above.
2.“重试出错”（默认情况下使用）写 - 上未能写入batchInterval经过反复后，通过客户端的请求（如果有机会的话写会成功 - 错误是由服务器过载引起的，网络错误等等。）

   2. 'Retry on error' write (used by default) - on failed write the request by the client is repeated after batchInterval elapses (if there is a chance the write will succeed - the error was caused by overloading the server, a network error etc.)
当成功写入之前（失败）点之前被写入新的数据点，这些都在客户端和等待中排队，直到点被成功写入旧数据。

       When new data points are written before the previous (failed) points are successfully written, those are queued inside the client and wait until older data points are successfully written.
这个队列的大小被限制，并通过`BatchOptions.bufferLimit`属性配置。当达到极限，在队列中最早的点被丢弃。 “重试上错误”策略用于当由`BatchOptions.actions`定义独立的写批量大小大于`BatchOptions.bufferLimit`低。

       Size of this queue is limited and configured by `BatchOptions.bufferLimit` property. When the limit is reached, the oldest points in the queue are dropped. 'Retry on error' strategy is used when individual write batch size defined by `BatchOptions.actions` is lower than `BatchOptions.bufferLimit`.

注意：

Note:

*批处理功能创建一个内部线程池，需要关闭明确作为一个优雅的应用程序关闭或应用程序将正确地终止的一部分。要做到这一点，调用`influxDB.close（）`。

* Batching functionality creates an internal thread pool that needs to be shutdown explicitly as part of a graceful application shutdown or the application will terminate properly. To do so, call `influxDB.close()`.


###写入InfluxDB

### Writing to InfluxDB
(5) ...

(5) ...

`---- 8 <---- BEGIN DRAFT ---- 8 <----`

`----8<----BEGIN DRAFT----8<----`

批量冲洗过程中发生的任何错误都不能漏入`write`方法的调用者。默认情况下，任何错误都将被记录只是用“严重”级别。

Any errors that happen during the batch flush won't leak into the caller of the `write` method. By default, any kind of errors will be just logged with "SEVERE" level.
如果你需要得到通知，并做当这种异步错误发生的一些自定义逻辑，你可以用`BiConsumer <可迭代<点>，的Throwable>添加一个错误处理程序`使用重载`enableBatch`方法：

If you need to be notified and do some custom logic when such asynchronous errors happen, you can add an error handler with a `BiConsumer<Iterable<Point>, Throwable>` using the overloaded `enableBatch` method:

```Java
influxDB.enableBatch(BatchOptions.DEFAULTS.exceptionHandler(
        (failedPoints, throwable) -> { /* custom error handling here */ })
);
```

`---- 8 <---- END DRAFT ---- 8 <----`

`----8<----END DRAFT----8<----`

同步####写入InfluxDB（不推荐）

#### Writing synchronously to InfluxDB (not recommended)

如果你想同步写入数据点InfluxDB和处理错误（因为它们可能会发生）与每一个写：

If you want to write the data points synchronously to InfluxDB and handle the errors (as they may happen) with every write:

`---- 8 <---- BEGIN DRAFT ---- 8 <----`

`----8<----BEGIN DRAFT----8<----`

```Java
InfluxDB influxDB = InfluxDBFactory.connect("http://172.17.0.2:8086", "root", "root");
String dbName = "aTimeSeries";
influxDB.query(new Query("CREATE DATABASE " + dbName));
String rpName = "aRetentionPolicy";
influxDB.query(new Query("CREATE RETENTION POLICY " + rpName + " ON " + dbName + " DURATION 30h REPLICATION 2 DEFAULT"));

BatchPoints batchPoints = BatchPoints
                .database(dbName)
                .tag("async", "true")
                .retentionPolicy(rpName)
                .consistency(ConsistencyLevel.ALL)
                .build();
Point point1 = Point.measurement("cpu")
                    .time(System.currentTimeMillis(), TimeUnit.MILLISECONDS)
                    .addField("idle", 90L)
                    .addField("user", 9L)
                    .addField("system", 1L)
                    .build();
Point point2 = Point.measurement("disk")
                    .time(System.currentTimeMillis(), TimeUnit.MILLISECONDS)
                    .addField("used", 80L)
                    .addField("free", 1L)
                    .build();
batchPoints.point(point1);
batchPoints.point(point2);
influxDB.write(batchPoints);
Query query = new Query("SELECT idle FROM cpu", dbName);
influxDB.query(query);
influxDB.query(new Query("DROP RETENTION POLICY " + rpName + " ON " + dbName));
influxDB.query(new Query("DROP DATABASE " + dbName));
```
`---- 8 <---- END DRAFT ---- 8 <----`

`----8<----END DRAFT----8<----`


###从InfluxDB阅读

### Reading from InfluxDB
(7) ...

(7) ...


####查询使用回调

#### Query using Callbacks

influxdb的Java现在支持通过回调返回查询结果。只有一个

influxdb-java now supports returning results of a query via callbacks. Only one
下面的消费者都将被调用一次：

of the following consumers are going to be called once :

```Java
this.influxDB.query(new Query("SELECT idle FROM cpu", dbName), queryResult -> {
    // Do something with the result...
}, throwable -> {
    // Do something with the error...
});
```

####查询使用参数绑定（又名“准备语句”）

#### Query using parameter binding (a.k.a. "prepared statements")

如果您的查询是基于用户的输入，这是很好的做法，使用参数绑定，以避免[注入攻击（https://en.wikipedia.org/wiki/SQL_injection）。

If your Query is based on user input, it is good practice to use parameter binding to avoid [injection attacks](https://en.wikipedia.org/wiki/SQL_injection).
您可以创建与QueryBuilder的的帮助下参数绑定查询：

You can create queries with parameter binding with the help of the QueryBuilder:

```Java
Query query = QueryBuilder.newQuery("SELECT * FROM cpu WHERE idle > $idle AND system > $system")
        .forDatabase(dbName)
        .bind("idle", 90)
        .bind("system", 5)
        .create();
QueryResult results = influxDB.query(query);
```

绑定的值（）调用绑定到查询中的占位符（$空闲，$系统）。

The values of the bind() calls are bound to the placeholders in the query ($idle, $system).


##高级用法

## Advanced Usage

### Gzip已的支持

### Gzip's support

influxdb Java客户端默认不启用gzip压缩的HTTP请求体。如果你想使用gzip，以减少传输数据的大小，您可以拨打：

influxdb-java client doesn't enable gzip compress for http request body by default. If you want to enable gzip to reduce transfer data's size , you can call:

```Java
influxDB.enableGzip()
```

### UDP的支持

### UDP's support

influxdb Java客户端支持UDP协议了。你可以打电话直接通过UDP下面的方法来写。

influxdb-java client support udp protocol now. you can call following methods directly to write through UDP.

```Java
public void write(final int udpPort, final String records);
public void write(final int udpPort, final List<String> records);
public void write(final int udpPort, final Point point);
```

注：请确保写入内容的总大小不应> UDP协议的限制（64K），或者你应该使用HTTP而不是UDP。

Note: make sure write content's total size should not > UDP protocol's limit(64K), or you should use http instead of udp.

###分块支持

### Chunking support

influxdb Java客户端现在支持influxdb分块。下面的示例使用20的CHUNKSIZE和调用指定的消费者（例如的System.out.println）对于每个接收QueryResult中

influxdb-java client now supports influxdb chunking. The following example uses a chunkSize of 20 and invokes the specified Consumer (e.g. System.out.println) for each received QueryResult

```Java
Query query = new Query("SELECT idle FROM cpu", dbName);
influxDB.query(query, 20, queryResult -> System.out.println(queryResult));
```

### QueryResult中文件夹2个POJO

### QueryResult mapper to POJO

处理QueryResult对象的另一种方式是现在可用。

An alternative way to handle the QueryResult object is now available.
假设你有一个测量_CPU_：

Supposing that you have a measurement _CPU_:

```sql
> INSERT cpu,host=serverA,region=us_west idle=0.64,happydevop=false,uptimesecs=123456789i
>
> select * from cpu
name: cpu
time                           happydevop host    idle region  uptimesecs
----                           ---------- ----    ---- ------  ----------
2017-06-20T15:32:46.202829088Z false      serverA 0.64 us_west 123456789
```

而下面的代码键：

And the following tag keys:

```sql
> show tag keys from cpu
name: cpu
tagKey
------
host
region
```

1.创建一个POJO来表示你的测量。例如：

1. Create a POJO to represent your measurement. For example:

```Java
public class Cpu {
    private Instant time;
    private String hostname;
    private String region;
    private Double idle;
    private Boolean happydevop;
    private Long uptimeSecs;
    // getters (and setters if you need)
}
```

2.新增@测量，@ TimeColumn和@Column注释：

2. Add @Measurement,@TimeColumn and @Column annotations:

```Java
@Measurement(name = "cpu")
public class Cpu {
    @TimeColumn
    @Column(name = "time")
    private Instant time;
    @Column(name = "host", tag = true)
    private String hostname;
    @Column(name = "region", tag = true)
    private String region;
    @Column(name = "idle")
    private Double idle;
    @Column(name = "happydevop")
    private Boolean happydevop;
    @Column(name = "uptimesecs")
    private Long uptimeSecs;
    // getters (and setters if you need)
}
```

3.调用_InfluxDBResultMapper.toPOJO（...）_映射QueryResult中你的POJO：

3. Call _InfluxDBResultMapper.toPOJO(...)_ to map the QueryResult to your POJO:

```java
InfluxDB influxDB = InfluxDBFactory.connect("http://localhost:8086", "root", "root");
String dbName = "myTimeseries";
QueryResult queryResult = influxDB.query(new Query("SELECT * FROM cpu", dbName));

InfluxDBResultMapper resultMapper = new InfluxDBResultMapper(); // thread-safe - can be reused
List<Cpu> cpuList = resultMapper.toPOJO(queryResult, Cpu.class);
```

使用POJO ###写作

### Writing using POJO
我们使用的'annotations`将数据转换到POJO同样的方法，我们可以把数据作为POJO。

The same way we use `annotations` to transform data to POJO, we can write data as POJO.
具有相同POJO级CPU

Having the same POJO class Cpu

```java
String dbName = "myTimeseries";
String rpName = "aRetentionPolicy";
// Cpu has annotations @Measurement,@TimeColumn and @Column
Cpu cpu = new Cpu();
// ... setting data

Point point = Point.measurementByPOJO(cpu.getClass()).addFieldsFromPOJO(cpu).build();

influxDB.write(dbName, rpName, point);
```

#### QueryResult中映射限制

#### QueryResult mapper limitations

*如果您InfluxDB查询包含多个SELECT子句，你将不得不调用InfluxResultMapper＃toPOJO（）多次通过QueryResult中返回到各自的POJO每次测量图;

* If your InfluxDB query contains multiple SELECT clauses, you will have to call InfluxResultMapper#toPOJO() multiple times to map every measurement returned by QueryResult to the respective POJO;
*如果您InfluxDB查询包含多个SELECT子句**为相同的测量**，InfluxResultMapper将处理所有的结果，因为没有办法分清哪一个应该被映射到你的POJO。这可能导致返回一个无效的集合;

* If your InfluxDB query contains multiple SELECT clauses **for the same measurement**, InfluxResultMapper will process all results because there is no way to distinguish which one should be mapped to your POJO. It may result in an invalid collection being returned;
* A类字段与_ @柱（...，标签= TRUE）_（即，[InfluxDB标签]（https://docs.influxdata.com/influxdb/v1.2/concepts/glossary/#tag-注释值））必须声明为_String_。

* A Class field annotated with _@Column(..., tag = true)_ (i.e. a [InfluxDB Tag](https://docs.influxdata.com/influxdb/v1.2/concepts/glossary/#tag-value)) must be declared as _String_.

#### QueryBuilder的：

#### QueryBuilder:

创建InfluxDB查询的另一种方法是可用的。使用[QueryBuilder的（QUERY_BUILDER.md），您可以使用Java，而不是提供influxdb查询作为字符串创建查询。

An alternative way to create InfluxDB queries is available. By using the [QueryBuilder](QUERY_BUILDER.md) you can create queries using java instead of providing the influxdb queries as strings.

### InfluxDBMapper

### InfluxDBMapper

如果你想保存和使用模型加载数据，你可以使用[InfluxDBMapper（INFLUXDB_MAPPER.md）。

In case you want to save and load data using models you can use the [InfluxDBMapper](INFLUXDB_MAPPER.md).


###其他用途

### Other Usages

有关其他用法示例看看[InfluxDBTest.java]（https://github.com/influxdb/influxdb-java/blob/master/src/test/java/org/influxdb/InfluxDBTest.java“InfluxDBTest.java” ）

For additional usage examples have a look at [InfluxDBTest.java](https://github.com/influxdb/influxdb-java/blob/master/src/test/java/org/influxdb/InfluxDBTest.java "InfluxDBTest.java")



###发布

### Publishing

这是一个

This is a
[链接]（https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide）

[link](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide)
到Sonatype的OSS指南发布。我会更新这一节一次

to the sonatype oss guide to publishing. I'll update this section once
的[JIRA票]（https://issues.sonatype.org/browse/OSSRH-9728）是

the [jira ticket](https://issues.sonatype.org/browse/OSSRH-9728) is
关闭，我能够文物上传到Sonatype的存储库。

closed and I'm able to upload artifacts to the sonatype repositories.

＃＃＃ 经常问的问题

### Frequently Asked Questions

这是一个[常见问题]（FAQ.md）名单influxdb的Java。

This is a [FAQ](FAQ.md) list for influxdb-java.
