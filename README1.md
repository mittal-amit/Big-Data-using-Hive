This project is on big data where we have to do analysis on a large dataset (More than 5 tables with millions of records) which is in Oracle Database.

The problem with running complex queries is:
- Queries take lot of time even if we do the optimizations.
- It impacts the performance of the source database like live transactions will get impacted.

So the solution is big data.

The above data set is on a monolithic architecture which can be expended with vertical scaling but this won't solve the problem, so we'll have to shift the data to distributed system where we can use  horizontal scaling.

We're going to use below mentioned systems
- MySQL (Transactional Database)
- Hive (Data warehouse, distributed database)
- Sqoop (To transfer the data from monolithic system to distributed system)
- HBase (NoSQL, on Distributed Database)
- HDFS (Hadoop Distributed File System for storage)

So we're going to follow the below mentioned steps to sort out this problem

Step 1: I've used two existing tables (Customers and Orders) for this project, anybody can use any table from their schema for this.

Step 2: We're going to transfer the data to HDFS i.e. from Oracle database to distributed systems and for this we've used Sqoop

#Table 1 - Import data to Hive
sqoop import \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username root \
--password cloudera \
--table orders \
--warehouse-dir /user/amitm/project\
   
#Table 2 - sqoop import \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username root \
--password cloudera \
--table customers \
--warehouse-dir /user/amitm/project

Step 3: Now, as data is transferred on HDFS, we need to create the scheme on Hive Warehouse. We can customize the schema or create exact same as table present in Oracle Database. I've create as same as present in transactional database.

We can create the table either
- Get the column names and login to Hive and create the table
- Or, create the same schema using Sqoop on Hive, in this case we don't need to mention the column names

#Create Hive table - 1

sqoop create-hive-table \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username root \
--password cloudera \
--table orders \
--hive-table ord_minip \
--fields-terminated-by ','

#Create Hive table - 2

sqoop create-hive-table \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username root \
--password cloudera \
--table customers \
--hive-table cust_minip \
--fields-terminated-by ','

Step 4: Now, we need to load the data into these tables

load data inpath '/user/amitm/project/orders' into table ord_minip;
load data inpath '/user/amitm/project/customers' into table cust_minip;

Step 5: Table has been created in Hive and we can access the tables, do the analysis but there is one more problem which is:

Hive is basically for Analytical system and it allows us to perform aggregations on the data but this takes a lot of time to do the processing though it's workable but we need to run multiple queries on the data and we need some alternate of SQL system which can process the data in less time and here comes the HBase in the picture. 

So, for this reason we create one more table on HBase (NoSQL database) which
- Helps in quick searching
- CRUD operations 

This table can be accessed from Hive as well as HBase. Hive helps in processing the data under MapReduce whereas HBase helps in quick searching and CRUD Operations.

Note: HBase isn't relational database and it doesn't follow ACID properties.
 
#Create Hbase-Hive Table: We have combined both the tables

CREATE TABLE ord_hive(customer_id int,customer_fname string,customer_lname string,order_id int, order_date string) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' with SERDEPROPERTIES ('hbase.columns.mapping'=':key, personal:customer_fname, personal:customer_lname, personal:order_id,personal:order_date');

Step 6: Load the data into Hive-HBase Table

insert overwrite table ord_hive select c.customer_id, c.customer_fname, c.customer_lname, o.order_id, o.order_date from cust_minip c join ord_minip o on c.customer_id = o.order_customer_id;

Data has been inserted and we'll be able to do the analysis or data can be given to Data Analyst/Data Scientist who can perform the statical analysis on the data.

Step 7: get 'ord_hive', 9996
