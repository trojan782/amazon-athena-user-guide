# Avro SerDe<a name="avro-serde"></a>

## SerDe name<a name="serde-name"></a>

 [Avro SerDe](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe) 

## Library name<a name="library-name"></a>

 [org\.apache\.hadoop\.hive\.serde2\.avro\.AvroSerDe](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe) 

## Examples<a name="examples"></a>

Athena does not support using `avro.schema.url` to specify table schema for security reasons\. Use `avro.schema.literal`\. To extract schema from data in the Avro format, use the Apache `avro-tools-<version>.jar` with the `getschema` parameter\. This returns a schema that you can use in your `WITH SERDEPROPERTIES` statement\. For example:

 `java -jar avro-tools-1.8.2.jar getschema my_data.avro` 

The `avro-tools-<version>.jar` file is located in the `java` subdirectory of your installed Avro release\. To download Avro, see [Apache Avro releases](http://avro.apache.org/releases.html#Download)\. To download Apache Avro Tools directly, see the [Apache Avro tools Maven repository](https://mvnrepository.com/artifact/org.apache.avro/avro-tools)\.

After you obtain the schema, use a `CREATE TABLE` statement to create an Athena table based on underlying Avro data stored in Amazon S3\. To specify the Avro SerDe, use `ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'`\. As demonstrated in the following example, you must specify the schema using the `WITH SERDEPROPERTIES` clause in addition to specifying the column names and corresponding data types for the table\.

**Note**  
Replace *myregion* in `s3://athena-examples-myregion/path/to/data/` with the region identifier where you run Athena, for example, `s3://athena-examples-us-west-1/path/to/data/`\.

```
CREATE EXTERNAL TABLE flights_avro_example (
   yr INT,
   flightdate STRING,
   uniquecarrier STRING,
   airlineid INT,
   carrier STRING,
   flightnum STRING,
   origin STRING,
   dest STRING,
   depdelay INT,
   carrierdelay INT,
   weatherdelay INT
)
PARTITIONED BY (year STRING)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
WITH SERDEPROPERTIES ('avro.schema.literal'='
{
   "type" : "record",
   "name" : "flights_avro_subset",
   "namespace" : "default",
   "fields" : [ {
      "name" : "yr",
      "type" : [ "null", "int" ],
      "default" : null
   }, {
      "name" : "flightdate",
      "type" : [ "null", "string" ],
      "default" : null
   }, {
      "name" : "uniquecarrier",
      "type" : [ "null", "string" ],
      "default" : null
   }, {
      "name" : "airlineid",
      "type" : [ "null", "int" ],
      "default" : null
   }, {
      "name" : "carrier",
      "type" : [ "null", "string" ],
      "default" : null
   }, {
      "name" : "flightnum",
      "type" : [ "null", "string" ],
      "default" : null
   }, {
      "name" : "origin",
      "type" : [ "null", "string" ],
      "default" : null
   }, {
      "name" : "dest",
      "type" : [ "null", "string" ],
      "default" : null
   }, {
      "name" : "depdelay",
      "type" : [ "null", "int" ],
      "default" : null
   }, {
      "name" : "carrierdelay",
      "type" : [ "null", "int" ],
      "default" : null
   }, {
      "name" : "weatherdelay",
      "type" : [ "null", "int" ],
      "default" : null
    } ]
}
')
STORED AS AVRO
LOCATION 's3://athena-examples-myregion/flight/avro/';
```

Run the `MSCK REPAIR TABLE` statement on the table to refresh partition metadata\.

```
MSCK REPAIR TABLE flights_avro_example;
```

Query the top 10 departure cities by number of total departures\.

```
SELECT origin, count(*) AS total_departures
FROM flights_avro_example
WHERE year >= '2000'
GROUP BY origin
ORDER BY total_departures DESC
LIMIT 10;
```

**Note**  
The flight table data comes from [Flights](http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&amp;DB_Short_Name=On-Time) provided by US Department of Transportation, [Bureau of Transportation Statistics](http://www.transtats.bts.gov/)\. Desaturated from original\.