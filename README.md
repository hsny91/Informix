# Informix
This is an easy install guide of the informix database on
raspberry-pi. It is mostly a 1:1 from [here](https://www.raspberrypi.org/forums/viewtopic.php?f=37&t=97199)

## Overview 
- [download informix](#Installation-of-Informix)
- [Configuration and Initialization of a New Informix Instance](#Configuration-and-Initialization-of-a-New-Informix-Instance)
- [Create an Informix demo database](#Create-an-Informix-Demo-Database)
- [How to Create a Sensor Database with IBM Informix](#How-to-Create-a-Sensor-Database-with-IBM-Informix)
- [Read out The Internal Temperature of an Raspberry Pi](#Read-out-The-Internal-Temperature-of-an-Raspberry-Pi)


## Installation of Informix
- Download `Informix Developer Edition for Linux ARM v7 32 (Raspberry PI)`. Must be
that version and must be ARM enabled.
You can use [this](https://www.ibm.com/products/informix/editions) link.

- Copy the tar file for installation to your `raspberry` temporary folder: 
> **scp examplefile yourusername@yourserver:/tmp** I.e. in my case:

```
scp ids.12.10.UC9DE.Linux-ARM7.tar pi@yipAdress:/tmp
```
- Then login into your raspberry and do the following:

- Create a temporary folder for the Informix install files

```
mkdir /tmp/ifxinstall
```
- Change directory to that folder:

```
cd /tmp/ifxinstall
```
-  Extract the Informix tar file - wait a bit until everything is
   unpacked and do not interrupt:

```
tar xvf /tmp/ids.12.10.UC9DE.Linux-ARM7.tar 
```
- create a new group informix:

```
sudo addgroup informix
```
-  (Ref:1) Create a new user informix (with the primary group
    informix). During this step you will be asked for a password for
    informix. Please take a note of that password. You will need it
    later.
```
 sudo adduser --ingroup informix informix
```
- Make sure to add the user informix to the /etc/sudoers file by adding the following line by using the ‘sudo visudo’ command:

```
informix ALL=(ALL) NOPASSWD: ALL
```
- Run the installation script

```
sudo ./ids_install
```
- Follow the UI and enter the following:

     + accept -> 1 RET

     + RET

     + specify directory where to install the products:
       `/opt/IBM/informix1210UC9DE`

     + accept -> 1 (wait a long bit and do not interrupt)

- Optional: As soon as the installation has successfully finished,
  you can delete the ifxinstall folder and the Informix install tar
  file if you want.

```
rm -rf /tmp/ifxinstall
rm /tmp/ds.12.10.UC9DE.Linux-ARM7.tar 
```
- Optional, but highly recommended: create the following symbolic link:

```
sudo ln -s /opt/IBM/informix1210UC9DE /opt/IBM/informix
```
- Create the folder which will later contain the Informix database files:

```
sudo mkdir /opt/IBM/ifxdata
```
- Set its ownership and permissions:

```
sudo chown informix:informix /opt/IBM/ifxdata
sudo chmod 770 /opt/IBM/ifxdata
```
## Configuration and Initialization of a New Informix Instance

- Login as the informix user
```
  su informix 
    
```
- And enter your password of (Ref:1).
Set the $INFORMIXDIR environment variable to point to the Informix install directory (actually to the symbolic link):

```
  export INFORMIXDIR=/opt/IBM/informix
    
```
- Extend the $PATH environment variable:

```
  export PATH=$PATH:$INFORMIXDIR/bin
    
```
- Set the $INFORMIXSERVER environment variable (you can choose any name here, but let’s use ol_informix1210 for now to keep it simple):

```
  export INFORMIXSERVER=ol_informix1210
    
```
- Create a new Informix configuration file:

```
  cp $INFORMIXDIR/etc/onconfig.std $INFORMIXDIR/etc/onconfig
    
```
- Create a new Informix hosts definition file:

```
  cp $INFORMIXDIR/etc/sqlhosts.demo $INFORMIXDIR/etc/sqlhosts
    
```
- Edit the file $INFORMIXDIR/etc/onconfig (with nano, vi or any other editor)
I.e. enter the file via the nano editor (or any other editor)

```
 nano $INFORMIXDIR/etc/onconfig

```
- And apply the following changes (use the search option ^W (control-W)):

```
  ROOTPATH /opt/IBM/ifxdata/rootdbs
  DBSERVERNAME	ol_informix1210
  LTAPEDEV	/dev/null
  TAPEDEV	  /dev/null
  LOGFILES	10

```
Save the file and exit the editor (Control-O RET; Control-X).

- Edit the file $INFORMIXDIR/etc/sqlhosts

```
  nano $INFORMIXDIR/etc/sqlhosts
    
```

- And add the following line:

```
  ol_informix1210	onsoctcp	localhost	9088
    
```
Note: 9088 is the port which will be used by Informix for the client/server communication. You can choose any available port you want. Save the file and exit the editor. 

- Create an empty database file and set the correct access mode:

```
  touch /opt/IBM/ifxdata/rootdbs
  chmod 660 /opt/IBM/ifxdata/rootdbs

```
- Now we are ready to initialize Informix for the first time:

```
  oninit -iv
    
```
- The first initialization will take a few minutes and it will create a few system databases automatically. You can monitor the pogress by doing the following:

```
  tail -f /opt/IBM/informix/tmp/online.log
    
```
Please wait until you see the following entry in the `online.log` file before you continue: 
>**‘sysadmin’ database built successfully**


## Create an Informix Demo Database

**As user informix:**
- Execute the following command to create the 'stores_demo' database:

```
 dbaccessdemo -log
    
```
>Depending on what kind of storage you might be using for your RPi that command might take a few minutes to complete.

**As user informix:**
- To stop an Informix instance:

```
 onmode -ky
    
```
- To start an Informix instance:

```
 oninit
    
```
- To check the status of Informix:
```
 onstat -
    
```
- Display the last message log entries:

```
onstat -m
    
```
- Display some basic performance stats:

```
onstat -p
    
```
**As any user who has the Informix environment variables (see above) set:**

 - Execute SQL scripts from the command line:

```
dbaccess <database_name> <sql_script_file>
    
```
- Using dbaccess interactively:

```
dbaccess <database_name>
    
```
- or simply

```
dbaccess 
    
```

## How to Create a Sensor Database with IBM Informix
In this example we will create a sensor database with with a sensor data table to hold the data for sensors which are producing measurements in 1-minute intervals. So we will be dealing with regular time series.

- In the very first step let's create an empty Informix database
```
echo "create database sensor_db with log" | dbaccess - -

```
- Access database
```
dbaccess

```
- Click `Select`. Then choose `sensor-db@ol_informix1210`

- Now we need to describe/define the actual payload for the timeseries column in our sensor data table. 

```
create row type sensor_t
(
        timestamp       datetime year to fraction(5),
        value           decimal(15,2)
);

```
- In the next step we need to create a 'container' which will store the actual time series data.

```
execute procedure TSContainerCreate (
        'sensor_cont',
        'rootdbs',
        'sensor_t',
        2048,
        2048);

```
- In the following step we then create the actual time series base table with the TIMESERIES column 'sensor_values', referencing the time series type 'sensor_t':

```
create table sensor_ts (
        sensor_id char(40),
        sensor_type char(20),
        sensor_unit char(6),
        sensor_values timeseries(sensor_t),
        primary key (sensor_id)
) lock mode row;

```
- We are creating a VTI table with the name 'sensor_data' (like the relational table in the beginning of my post) based on the time series base table 'sensor_ts'.

```
execute procedure TSCreateVirtualTab (
        'sensor_data',
        'sensor_ts',
        'origin(2015-01-26 00:00:00.00000),calendar(ts_1min),container(sensor_cont),threshold(0),regular',0,'sensor_values');

```

- After executing that stored procedure, you will have a virtual table called 'sensor_data' with the following schema:

```
create table sensor_data
  (
    sensor_id char(40) not null ,
    sensor_type char(20),
    sensor_unit char(6),
    timestamp datetime year to fraction(5),
    value decimal(15,2)
  );

```
- Now we test the 'sensor_data' table by inserting the following 6 demo records.

```
insert into sensor_data values ("Sensor01", "Temp", "C", "2015-01-26 08:00"::datetime year to minute, 21.5);
insert into sensor_data values ("Sensor01", "Temp", "C", "2015-01-26 08:01"::datetime year to minute, 21.6);
insert into sensor_data values ("Sensor02", "Temp", "C", "2015-01-26 15:45"::datetime year to minute, 35.9);
insert into sensor_data values ("Sensor01", "Temp", "C", "2015-01-26 08:02"::datetime year to minute, 22.1);
insert into sensor_data values ("Sensor02", "Temp", "C", "2015-01-26 15:46"::datetime year to minute, 35.2);
insert into sensor_data values ("Sensor02", "Temp", "C", "2015-01-26 15:47"::datetime year to minute, 33.5);

```

-  Do a 'SELECT *' on the sensor_data table first:
```
echo "select * from sensor_data" | dbaccess sensor_db -

```
-  As a result of the search, we expect the following results

> sensor_id    Sensor01 
>
> sensor_type  Temp
>
> sensor_unit  C
>
> timestamp    2015-01-26 08:00:00.00000
>
> value        21.50
><br/><br/>
> sensor_id    Sensor01
>
> sensor_type  Temp
>
> sensor_unit  C
>
> timestamp    2015-01-26 08:01:00.00000
> 
> value        21.60
><br/><br/>
> sensor_id    Sensor01
> 
> sensor_type  Temp
> 
> sensor_unit  C
> 
> timestamp    2015-01-26 08:02:00.00000
> 
> value        22.10
><br/><br/>
> sensor_id    Sensor02
>
> sensor_type  Temp
>
> sensor_unit  C
> 
> timestamp    2015-01-26 15:45:00.00000
> 
> value        35.90
><br/><br/>
> sensor_id    Sensor02
>
> sensor_type  Temp
> 
> sensor_unit  C
> 
> timestamp    2015-01-26 15:46:00.00000
> 
> value        35.20
><br/><br/>
> sensor_id    Sensor02
> 
> sensor_type  Temp
> 
> sensor_unit  C
> 
> timestamp    2015-01-26 15:47:00.00000
>
>value        33.50
><br/><br/>
> 6 row(s) retrieved.
>
> Database closed.


- Now let's try to run the following SQL query against the time series base table: 

```
echo "SELECT sensor_id, sensor_type, sensor_unit FROM sensor_ts" | dbaccess sensor_db -

```

-  As a result of the search, we expect the following results


> Database selected.
>
> sensor_id&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sensor_type&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sensor_unit
>
> Sensor01&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Temp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;C
>
> Sensor02&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Temp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;C
>
> 2 row(s) retrieved.

- Before we finish that first and very basic part on how to use the Informix time series data type, We look at glimpse on what you can do with the stored data. Before I do that let's add a few additional example rows through our 'sensor_data' VTI table:

```
insert into sensor_data values ("Sensor01", "Temp", "C", "2015-01-26 07:58"::datetime year to minute, 20.2);
insert into sensor_data values ("Sensor01", "Temp", "C", "2015-01-26 07:59"::datetime year to minute, 20.7);
insert into sensor_data values ("Sensor01", "Temp", "C", "2015-01-27 10:11"::datetime year to minute, 26.3);
insert into sensor_data values ("Sensor01", "Temp", "C", "2015-01-27 10:45"::datetime year to minute, 26.9);
```
- In the following SQL example we are aggregating the minute interval based data for 'sensor_id' 'Sensor01' to an hourly granularity:

```
SELECT AggregateBy(
        'SUM($value)',
        'ts_1hour',
        sensor_values,
        0,
        '2015-01-26 00:00'::datetime year to minute,
        '2015-01-31 23:59'::datetime year to minute)
FROM sensor_ts
WHERE sensor_id = "Sensor01";

```
- The 'AggregateBy()' function is returning a TIMESERIES object with the aggregated values based on the supplied 'ts_1hour' calendar for the date/time range from/to provided. The result looks like this:

> (expression)  origin(2015-01-26 00:00:00.00000), calendar(ts_1hour), container(sensor_cont), threshold(0), 
>
>regular, [NULL, NULL, NULL, NULL, NULL, NULL, NULL, (40.90), (65.20), NULL, NULL, NULL, NULL, NULL,
> NULL,NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL,
> NULL, NULL, NULL, NULL, (53.20)]
>
> 1 row(s) retrieved.

- In the following SQL example we are aggregating the minute interval based data for 'sensor_id' 'Sensor01' to an daily granularity:

```
SELECT AggregateBy(
        'SUM($value)',
        'ts_1day',
        sensor_values,
        0,
        '2015-01-26 00:00'::datetime year to minute,
        '2015-01-31 23:59'::datetime year to minute)
FROM sensor_ts
WHERE sensor_id = "Sensor01";

```
- We expect the following results:
>(expression)  origin(2015-01-26 00:00:00.00000), calendar(ts_1day), container(sensor_cont), threshold(0), regular, [(106.10), (53.20)]
>
>1 row(s) retrieved.

- Since your application probably doesn't know how to handle an Informix time series object directly, let's convert those objects on the fly into a tabular format for a more classic relational processing. We are using a combination of the 'Transpose()' function and the 'Table()' constructor to create the derived table 'sensor':

```
SELECT sensor.timestamp::datetime year to hour, sensor.value FROM TABLE (
TRANSPOSE ((
SELECT AggregateBy(
        'SUM($value)',
        'ts_1hour',
        sensor_values,
        0,
        '2015-01-26 00:00'::datetime year to minute,
        '2015-01-31 23:59'::datetime year to minute)
FROM sensor_ts
WHERE sensor_id = "Sensor01"
))::sensor_t
) AS TAB (sensor);

```
- We expect the following results:

> timestamp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value
> <br/><br/>
> 2015-01-26&nbsp;&nbsp;&nbsp;07&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;40.90
> 
> 2015-01-26&nbsp;&nbsp;&nbsp;08&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;65.20
> 
> 2015-01-27&nbsp;&nbsp;&nbsp;10&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;53.20
>
> 3 row(s) retrieved.

- Now we  adjust the format of the timestamp column to an daily format. Finally re-run the daily aggregation in a similar way:

```
SELECT sensor.timestamp::datetime year to day timestamp, sensor.value FROM TABLE (
TRANSPOSE ((
SELECT AggregateBy(
        'SUM($value)',
        'ts_1day',
        sensor_values,
        0,
        '2015-01-26 00:00'::datetime year to minute,
        '2015-01-31 23:59'::datetime year to minute)
FROM sensor_ts
WHERE sensor_id = "Sensor01"
))::sensor_t
) AS TAB (sensor);

```
- We expect the following results:

> timestamp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value
> <br/><br/>
> 2015-01-26&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;106.10
> 
> 2015-01-27&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;53.20
>
> 2 row(s) retrieved.

# Increase your application's flexibility with a JSON based sensor data table!
- Now that you have learned the basics on how to create an Informix sensor data table based on structured data types so far, let's have a brief look on how to create a JSON based sensor data table.

- Now we need to describe/define the actual payload for the timeseries column in our sensor data table:

```
create row type sensor_json_t
(
        timestamp       datetime year to fraction(5),
        values           bson
);
```

- In the following SQL script we are creating a new time series container for the JSON time series, a new base table with the TIMESERIES column and a new VTI table to easily access the JSON based sensor data:

```
execute procedure TSContainerCreate (
        'sensor_json_cont',
        'rootdbs',
        'sensor_json_t',
        2048,
        2048);
```

```
create table sensor_json_ts (
        sensor_id char(40),
        sensor_type char(20),
        sensor_unit char(6),
        sensor_values timeseries(sensor_json_t),
        primary key (sensor_id)
) lock mode row;

```

```
execute procedure TSCreateVirtualTab (
        'sensor_json_data',
        'sensor_json_ts',
        'origin(2015-01-26 00:00:00.00000),calendar(ts_1min),container(sensor_json_cont),threshold(0),regular',0,'sensor_values');

```
- As soon as you have the VTI table 'sensor_json_data' created, you can start to insert new rows with JSON formatted sensor data. Notice that 'Sensor03' and 'Sensor04' have different number of measurement values ('temp1' and 'temp' respectively):

```
insert into sensor_json_data values ("Sensor03", "Temp", "C", "2015-01-26 08:00"::datetime year to minute, '{ "temp1":21.5 }'::json);
insert into sensor_json_data values ("Sensor03", "Temp", "C", "2015-01-26 08:01"::datetime year to minute, '{ "temp1":23.1 }'::json);
insert into sensor_json_data values ("Sensor03", "Temp", "C", "2015-01-27 10:45"::datetime year to minute, '{ "temp1":22.8 }'::json);
insert into sensor_json_data values ("Sensor04", "Temp", "C", "2015-01-26 12:02"::datetime year to minute, '{ "temp1":30.2, "temp2":10.3}'::json);
insert into sensor_json_data values ("Sensor04", "Temp", "C", "2015-01-27 15:46"::datetime year to minute, '{ "temp1":34.0, "temp2":9.7}'::json);
insert into sensor_json_data values ("Sensor04", "Temp", "C", "2015-01-27 15:47"::datetime year to minute, '{ "temp1":33.9, "temp2":9.2}'::json);

```
- Now let's select the data which we just inserted:

```
select sensor_id, sensor_type, sensor_unit, timestamp, values::json values from sensor_json_data;

```

- We expect the following results:

> sensor_id    Sensor03
>
> sensor_type  Temp
>
> sensor_unit  C
> 
> timestamp    2015-01-26 08:00:00.00000
>
> values       {"temp1":21.500000}
><br/><br/>
> sensor_id    Sensor03
>
> sensor_type  Temp
>
> sensor_unit  C
>
> timestamp    2015-01-26 08:01:00.00000
>
> values       {"temp1":23.100000}
>
> sensor_id    Sensor03
>
> sensor_type  Temp
>
> sensor_unit  C
>
> timestamp    2015-01-27 10:45:00.00000
>
> values       {"temp1":22.800000}
><br/><br/>
> sensor_id    Sensor04
> 
> sensor_type  Temp
> 
> sensor_unit  C
> 
> timestamp    2015-01-26 12:02:00.00000
> 
> values       {"temp1":30.200000,"temp2":10.300000}
><br/><br/>
> sensor_id    Sensor04
> 
> sensor_type  Temp
> 
> sensor_unit  C
> 
> timestamp    2015-01-27 15:46:00.00000
> 
> values       {"temp1":34.000000,"temp2":9.700000}
><br/><br/>
> sensor_id    Sensor04
>
> sensor_type  Temp
> 
> sensor_unit  C
> 
> timestamp    2015-01-27 15:47:00.00000
> 
> values       {"temp1":33.900000,"temp2":9.200000}
><br/><br/>
>6 row(s) retrieved.

- We are now using JSON documents to store the sensor data, we can  use the same time series functions to do the aggregations:

```
SELECT sensor.timestamp::datetime year to hour timestamp, sensor.value temp1 FROM TABLE (
TRANSPOSE ((
SELECT AggregateBy(
        'SUM($temp1)',
        'ts_1hour',
        sensor_values,
        0,
        '2015-01-26 00:00'::datetime year to minute,
        '2015-01-31 23:59'::datetime year to minute)::TimeSeries(sensor_t)
FROM sensor_json_ts
WHERE sensor_id = "Sensor03"
))
) AS TAB (sensor);

```

- We expect the following results:

> timestamp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value
> <br/><br/>
> 2015-01-26&nbsp;&nbsp;&nbsp;08&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;44.60
> 
> 2015-01-27&nbsp;&nbsp;&nbsp;10&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;22.80
>
> 2 row(s) retrieved.

- If you would like to do an aggregation on two JSON fields, you need to a create a new row type with two numeric elements, e.g. value1 and value2:

```
create row type sensor_twovals_t
(
        timestamp       datetime year to fraction(5),
        value1           decimal(15,2),
        value2           decimal(15,2)
);

```
- The associated query looks like:

```
SELECT sensor.timestamp::datetime year to hour timestamp,
        sensor.value1 temp1,  sensor.value2 temp2 FROM TABLE (
TRANSPOSE ((
SELECT AggregateBy(
        'SUM($temp1),SUM($temp2)',
        'ts_1hour',
        sensor_values,
        0,
        '2015-01-26 00:00'::datetime year to minute,
        '2015-01-31 23:59'::datetime year to minute)::TimeSeries(sensor_twovals_t)
FROM sensor_json_ts
WHERE sensor_id = "Sensor04"
))
) AS TAB (sensor);

```
- We expect the following results:

> timestamp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;temp1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;temp2
> <br/><br/>
> 2015-01-26&nbsp;&nbsp;&nbsp;12&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;30.20&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;10.30
> 
> 2015-01-27&nbsp;&nbsp;&nbsp;15&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;67.90&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;18.90
>
> 2 row(s) retrieved.


## Read out The Internal Temperature of an Raspberry Pi

- Now we are going to use the Informix database to automatically read out the Raspberry Pi's internal temperature every 60 seconds and store it in the 'sensor_data' table.
- The RPi's internal temperature can be accessed through the following file: `/sys/class/thermal/thermal_zone0/temp`

- To access that value from within Informix we are going to create a so called EXTERNAL TABLE. EXTERNAL TABLES allow easy access from and to external files and hence are great e.g. for loading and/or exporting data. Let's create an EXTERNAL TABLE called 'rpi_temp':

```
create external table rpi_temp
(
        temp1 integer external char(6)
)
using
(
        FORMAT "FIXED",
        DATAFILES
        (
                "DISK:/sys/class/thermal/thermal_zone0/temp"
        )
);

```
- So each time we do e.g. a 'SELECT temp1/1000 from rpi_temp' we will get the current RPi temperature in centigrade

```
echo "select temp1/1000 from rpi_temp" | dbaccess sensor_db -
```

- We expect the following results:

> Database selected
>
>    (expression)
>
>     48.6920000000000
>
> 1 row(s) retrieved.

- Now we need to find a way to read out that table every 60 seconds and store it in the sensor_data table. 

```
insert into sensor_data
        select "SensorRPiTemp", "Temperature", "C",  current::datetime year to minute,
                (temp1/1000)::DECIMAL(15,2) from rpi_temp;

```

- Now create a new task by inserting a new row into the 'sysadmin:ph_task' table as user 'informix':
  Note: Here you have to switch to database sysadmin@ol_informix1210 .

```
iINSERT INTO ph_task
(
        tk_name,
        tk_description,
        tk_type,
        tk_group,
        tk_execute,
        tk_start_time,
        tk_stop_time,
        tk_frequency,
        tk_dbs
)
VALUES
(
        "Read Out RPi Temp",
        "Reads out the RPi internal temp every 60 secs",
        "TASK",
        "MISC",
        "insert into sensor_db:sensor_data select 'SensorRPiTemp', 'Temperature', 'C', current::datetime year to minute, (temp1/1000)::DECIMAL(15,2) from sensor_db:rpi_temp",
        NULL,
        NULL,
        interval(1) minute to minute,
        "sensor_db"
)
```
- Since the Informix scheduler is caching the content of the 'ph_task' table, you should stop and re-start the scheduler with the following SQL statements as user 'informix’:

```
echo "EXECUTE FUNCTION task('scheduler shutdown')" | dbaccess sysadmin -
echo "EXECUTE FUNCTION task('scheduler start')" | dbaccess sysadmin -
```

- With the following SELECT statement you can read out the last five entries (temperature entries) from the 'sensor_data' table in descending order: 

```
echo "select first 5 * from sensor_data where sensor_id = 'SensorRPiTemp' order by timestamp desc" | dbaccess sensor_db -
```

Congratulations! :clap:

