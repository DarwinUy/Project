# FTP pull
 
## Create Key

ssh-keygen -t rsa -f ~/.ssh/gcpkey -C darwin -P""
cat gcpkey.pub
ssh darwin@35.193.228.156
exit

## In new Terminal
### Put and get file
cd /home/darwin/Documents/Data_sets/CSV
sftp darwin@35.193.228.156
put region.csv 
get region.csv region_again.csv      #Get "File" "Location"

hdfs dfs -mkdir -p /user/input/cloud
hdfs dfs -put ~/Documents/Data_sets/CSV/region_again.csv /user/input/cloud/region_again.csv

hdfs dfs -ls /user/input/cloud


# SQL Import

## MySQL

Source /home/darwin/Documents/Data_sets/MYSQL/sample_data_videogames_mysql.sql


## Postgress

psql -h localhost -d steam_games -U darwin -f ~/data_sources/Postgress/data.sql

## TSQL

### Import skillcraft1_dataset

CREATE TABLE Test.dbo.skillcraft1_dataset
(
gameid INT,
leagueindex INT,
age INT,
hoursperweek INT,
totalhours INT,
apm Float,
selectbyhotkeys Float,
assigntohotkeys Float,
uniquehotkeys INT,
minimapattacks Float,
Minimaprightclicks Float,
numberofpacs Float,
gapbetweenpacs Float,
actionlatency Float,
actionsinpac Float,
totalmapexplored INT,
workersmade Float,
uniqueunitsmade INT,
complexunitsmade Float,
complexabilitiesused Float
)
GO

BULK INSERT Test.dbo.skillcraft1_dataset
FROM '/home/darwin/Documents/Data_sets/CSV/skillcraft1_dataset.csv'
WITH
(
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n'
)
GO


### Import Test.dbo.vgsales

CREATE TABLE Test.dbo.vgsales
(
Rank INT,
Name VARCHAR(255),
Platform VARCHAR(255),
Year VARCHAR(10),
Genre VARCHAR(255),
Publisher VARCHAR(255),
NA_Sales Float,
EU_Sales Float,
JP_Sales Float,
Other_Sales Float,
Global_Sales Float
)
GO

BULK INSERT Test.dbo.vgsales
FROM '/home/darwin/Documents/Data_sets/CSV/vgsales.csv'
WITH
(
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n'
)
GO


### Import Test.dbo.Video_games_listed

CREATE TABLE Test.dbo.Video_games_listed
(
SSID VARCHAR(255),
VG1Title VARCHAR(255),
VG2Title VARCHAR(255),
VG3Title VARCHAR(255)
)
GO

BULK INSERT Test.dbo.Video_games_listed
FROM '/home/darwin/Documents/Data_sets/CSV/Video_games_listed.csv'
WITH
(
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n'
)
GO


# Import Databases using Sqoop

## MySQL Imports


### Make the folder in HDFS

hdfs dfs -mkdir -p /user/input/mysql
 
 
### Import game table

sqoop import \
--connect jdbc:mysql://localhost/video_games \
--username darwin \
--password zxc \
--columns "id, genre_id, game_name" \
--table game \
--target-dir /user/input/mysql/game 

### Import game_platform table

sqoop import \
--connect jdbc:mysql://localhost/video_games \
--username darwin \
--password zxc \
--columns "id, game_publisher_id, platform_id, release_year " \
--table game_platform \
--target-dir /user/input/mysql/game_platform

### Import game_publisher table

sqoop import \
--connect jdbc:mysql://localhost/video_games \
--username darwin \
--password zxc \
--columns "id, game_id, publisher_id " \
--table game_publisher \
--target-dir /user/input/mysql/game_publisher



## Postgress Import

### Make the folder in HDFS

hdfs dfs -mkdir -p /user/input/postgress

### Import _genre table

sqoop import \
--connect jdbc:postgresql://localhost/steam_games \
--username darwin \
--password zxc \
--columns "id, genre_name" \
--m 1 \
--table _genre \
--target-dir /user/input/postgress/genre

### Import _platform table

sqoop import \
--connect jdbc:postgresql://localhost/steam_games \
--username darwin \
--password zxc \
--columns "id, platform_name" \
--m 1 \
--table _platform \
--target-dir /user/input/postgress/platform

### Import _publisher table

sqoop import \
--connect jdbc:postgresql://localhost/steam_games \
--username darwin \
--password zxc \
--columns "id, publisher_name" \
--m 1 \
--table _publisher \
--target-dir /user/input/postgress/publisher


## SQL Server Import

hdfs dfs -mkdir -p /user/input/sql-server


### Import skillcraft1_dataset

sqoop import \
--connect 'jdbc:sqlserver://localhost:1433;databaseName=Test' \
--username 'SA' \
--password 'Zaq1zaq1' \
--columns " \
gameid, leagueindex, age, hoursperweek, totalhours, \
apm, selectbyhotkeys, assigntohotkeys, uniquehotkeys, \
minimapattacks, Minimaprightclicks, numberofpacs, \
gapbetweenpacs, actionlatency, actionsinpac, \
totalmapexplored, workersmade, uniqueunitsmade, \
complexunitsmade, complexabilitiesused  \
" \
--m 1 \
--table skillcraft1_dataset \
--target-dir /user/input/sql-server/skillcraft1_dataset


### Import vgsales

sqoop import \
--connect 'jdbc:sqlserver://localhost:1433;databaseName=Test' \
--username 'SA' \
--password 'Zaq1zaq1' \
--columns " \
Rank, Name, Platform, Year, Genre, Publisher, \
NA_Sales, EU_Sales, JP_Sales, Other_Sales, Global_Sales \
" \
--m 1 \
--table vgsales \
--target-dir /user/input/sql-server/vgsales


### Import Video_games_listed

sqoop import \
--connect 'jdbc:sqlserver://localhost:1433;databaseName=Test' \
--username 'SA' \
--password 'Zaq1zaq1' \
--columns "SSID, VG1Title, VG2Title, VG3Title"  \
--m 1 \
--table Video_games_listed \
--target-dir /user/input/sql-server/Video_games_listed


# Hive External table import

CREATE DATABASE Raw;
USE Raw;

## Import game 

create external table game
(
id int,
genre_id int,
game_name varchar(200)
)
row format delimited
fields terminated by ','
location '/user/input/mysql/game';

load data inpath '/user/input/mysql/game' into table game;


## Import game_platform 

create external table game_platform
(
id int, 
game_publisher_id int, 
platform_id int, 
release_year int 
)
row format delimited
fields terminated by ','
location '/user/input/mysql/game_platform';

load data inpath '/user/input/mysql/game_platform' into table game_platform;


## Import game_publisher 

create external table game_publisher 
(
id int,
game_id int,
publisher_id int
)
row format delimited
fields terminated by ','
location '/user/input/mysql/game_publisher';

load data inpath '/user/input/mysql/game_publisher' into table game_publisher;


## Import _genre 

create external table genre
(
id smallint,
genre_name VARCHAR (100)
)
row format delimited
fields terminated by ','
location '/user/input/postgress/genre';

load data inpath '/user/input/postgress/genre' into table genre;


## Import _platform 

create external table platform
(
id smallint,
platform_name VARCHAR (100)
)
row format delimited
fields terminated by ','
location '/user/input/postgress/platform';

load data inpath '/user/input/postgress/platform' into table platform;


## Import _publisher 

create external table publisher
(
id smallint,
publisher_name VARCHAR (100)
)
row format delimited
fields terminated by ','
location '/user/input/postgress/publisher';

load data inpath '/user/input/postgress/publisher' into table publisher;


## Import skillcraft1_dataset 

create external table skillcraft1_dataset
(
gameid INT,
leagueindex INT,
age INT,
hoursperweek INT,
totalhours INT,
apm FLOAT,
selectbyhotkeys FLOAT,
assigntohotkeys FLOAT,
uniquehotkeys INT,
minimapattacks FLOAT,
Minimaprightclicks FLOAT,
numberofpacs FLOAT,
gapbetweenpacs FLOAT,
actionlatency FLOAT,
actionsinpac FLOAT,
totalmapexplored INT,
workersmade FLOAT,
uniqueunitsmade INT,
complexunitsmade FLOAT,
complexabilitiesused Float
)
row format delimited
fields terminated by ','
location '/user/input/sql-server/skillcraft1_dataset';

load data inpath '/user/input/sql-server/skillcraft1_dataset' into table skillcraft1_dataset;


## Import vgsales

create external table vgsales
(
Rank INT,
Name VARCHAR(255),
Platform VARCHAR(255),
Year INT,
Genre VARCHAR(255),
Publisher VARCHAR(255),
NA_Sales Float,
EU_Sales Float,
JP_Sales Float,
Other_Sales Float,
lobal_Sales Float
)
row format delimited
fields terminated by ','
location '/user/input/sql-server/vgsales';

load data  inpath '/user/input/sql-server/vgsales' into table vgsales;


## Import Video_games_listed

create external table Video_games_listed
(
SSID VARCHAR(255),
VG1Title VARCHAR(255),
VG2Title VARCHAR(255),
VG3Title VARCHAR(255)
)
row format delimited
fields terminated by ','
location '/user/input/sql-server/Video_games_listed';

load data  inpath '/user/input/sql-server/Video_games_listed' into table Video_games_listed;


#Moving External to Internal files

CREATE DATABASE DSL;
USE DSL;

## Import game 

create table game
(
id int,
genre_id int,
game_name varchar(200)
)
row format delimited
fields terminated by ',';

INSERT OVERWRITE TABLE game SELECT * FROM raw.game;

## Import game_platform 

create table game_platform
(
id int, 
game_publisher_id int, 
platform_id int, 
release_year int 
)
row format delimited
fields terminated by ',';

INSERT OVERWRITE TABLE game_platform SELECT * FROM raw.game_platform;

## Import game_publisher 

create table game_publisher 
(
id int,
game_id int,
publisher_id int
)
row format delimited
fields terminated by ',';

INSERT OVERWRITE TABLE game_publisher SELECT * FROM raw.game_publisher ;

## Import _genre 

create table genre
(
id smallint,
genre_name VARCHAR (100)
)
row format delimited
fields terminated by ',';

INSERT OVERWRITE TABLE genre SELECT * FROM raw.genre;

## Import _platform 

create table platform
(
id smallint,
platform_name VARCHAR (100)
)
row format delimited
fields terminated by ',';

INSERT OVERWRITE TABLE platform SELECT * FROM raw.platform;

## Import _publisher 

create table publisher
(
id smallint,
publisher_name VARCHAR (100)
)
row format delimited
fields terminated by ',';

INSERT OVERWRITE TABLE publisher SELECT * FROM raw.publisher;

## Import skillcraft1_dataset 

create table skillcraft1_dataset
(
gameid INT,
leagueindex INT,
age INT,
hoursperweek INT,
totalhours INT,
apm FLOAT,
selectbyhotkeys FLOAT,
assigntohotkeys FLOAT,
uniquehotkeys INT,
minimapattacks FLOAT,
Minimaprightclicks FLOAT,
numberofpacs FLOAT,
gapbetweenpacs FLOAT,
actionlatency FLOAT,
actionsinpac FLOAT,
totalmapexplored INT,
workersmade FLOAT,
uniqueunitsmade INT,
complexunitsmade FLOAT,
complexabilitiesused Float
)
row format delimited
fields terminated by ',';

INSERT OVERWRITE TABLE skillcraft1_dataset SELECT * FROM raw.skillcraft1_dataset;

## Import vgsales

create table vgsales
(
Rank INT,
Name VARCHAR(255),
Platform VARCHAR(255),
Year INT,
Genre VARCHAR(255),
Publisher VARCHAR(255),
NA_Sales Float,
EU_Sales Float,
JP_Sales Float,
Other_Sales Float,
lobal_Sales Float
)
row format delimited
fields terminated by ',';

INSERT OVERWRITE TABLE vgsales SELECT * FROM raw.vgsales;

## Import Video_games_listed

create table Video_games_listed
(
SSID VARCHAR(255),
VG1Title VARCHAR(255),
VG2Title VARCHAR(255),
VG3Title VARCHAR(255)
)
row format delimited
fields terminated by ',';

INSERT OVERWRITE TABLE Video_games_listed SELECT * FROM raw.Video_games_listed;


# Hive-External-to-HBase

CREATE DATABASE dsl_hbase;
USE dsl_hbase;

## Import game

create table game
(
id int,
genre_id int,
game_name varchar(200)
)
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties("hbase.columns.mapping"=":key, F1:genre_id, F1:game_name")
TBLPROPERTIES ("hbase.table.name" = "hbase_game");

INSERT OVERWRITE TABLE game SELECT * FROM raw.game;


## Import game_platform

create table game_platform
(
id int, 
game_publisher_id int, 
platform_id int, 
release_year int 
)
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties("hbase.columns.mapping"=":key, F1:game_publisher_id, F1:platform_id, F2:release_year")
TBLPROPERTIES ("hbase.table.name" = "hbase_game_platform");

INSERT OVERWRITE TABLE game_platform SELECT * FROM raw.game_platform;


## Import game_publisher

create table game_publisher 
(
id int,
game_id int,
publisher_id int
)
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties("hbase.columns.mapping"=":key, F1:game_id, F1:publisher_id")
TBLPROPERTIES ("hbase.table.name" = "hbase_game_publisher");

INSERT OVERWRITE TABLE game_publisher SELECT * FROM raw.game_publisher ;


## Import _genre

create table genre
(
id smallint,
genre_name VARCHAR (100)
)
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties("hbase.columns.mapping"=":key, F1:genre_name")
TBLPROPERTIES ("hbase.table.name" = "hbase_genre");

INSERT OVERWRITE TABLE genre SELECT * FROM raw.genre;


## Import _platform

create table platform
(
id smallint,
platform_name VARCHAR (100)
)
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties("hbase.columns.mapping"=":key, F1:platform_name")
TBLPROPERTIES ("hbase.table.name" = "hbase_platform");

INSERT OVERWRITE TABLE platform SELECT * FROM raw.platform;


## Import _publisher

create table publisher
(
id smallint,
publisher_name VARCHAR (100)
)
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties("hbase.columns.mapping"=":key, F1:publisher_name")
TBLPROPERTIES ("hbase.table.name" = "hbase_publisher");

INSERT OVERWRITE TABLE publisher SELECT * FROM raw.publisher;


## Import skillcraft1_dataset

create table skillcraft1_dataset
(
gameid INT,
leagueindex INT,
age INT,
hoursperweek INT,
totalhours INT,
apm FLOAT,
selectbyhotkeys FLOAT,
assigntohotkeys FLOAT,
uniquehotkeys INT,
minimapattacks FLOAT,
Minimaprightclicks FLOAT,
numberofpacs FLOAT,
gapbetweenpacs FLOAT,
actionlatency FLOAT,
actionsinpac FLOAT,
totalmapexplored INT,
workersmade FLOAT,
uniqueunitsmade INT,
complexunitsmade FLOAT,
complexabilitiesused Float
)
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties("hbase.columns.mapping"=":key, F1:leagueindex, F2:age, F2:hoursperweek, F2:totalhours, F2:apm, F3:selectbyhotkeys, F3:assigntohotkeys, F3:uniquehotkeys, F4:minimapattacks, F4:Minimaprightclicks, F5:numberofpacs, F5:gapbetweenpacs, F6:actionlatency, F6:actionsinpac, F4:totalmapexplored, F7:workersmade, F7:uniqueunitsmade, F7:complexunitsmade, F7:complexabilitiesused")
TBLPROPERTIES ("hbase.table.name" = "hbase_skillcraft1_dataset");

INSERT OVERWRITE TABLE skillcraft1_dataset SELECT * FROM raw.skillcraft1_dataset;


## Import vgsales

create table vgsales
(
Rank INT,
Name VARCHAR(255),
Platform VARCHAR(255),
Year INT,
Genre VARCHAR(255),
Publisher VARCHAR(255),
NA_Sales Float,
EU_Sales Float,
JP_Sales Float,
Other_Sales Float,
Global_Sales Float
)
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties("hbase.columns.mapping"=":key, F1:Name, F1:Platform, F1:Year, F1:Genre, F1:Publisher, F2:NA_Sales, F2:EU_Sales, F2:JP_Sales, F2:Other_Sales, F2:Global_Sales")
TBLPROPERTIES ("hbase.table.name" = "hbase_vgsales");

INSERT OVERWRITE TABLE vgsales SELECT * FROM raw.vgsales;


## Import Video_games_listed

create table Video_games_listed
(
SSID VARCHAR(255),
VG1Title VARCHAR(255),
VG2Title VARCHAR(255),
VG3Title VARCHAR(255)
)
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties("hbase.columns.mapping"=":key, F1:VG1Title, F1:VG2Title, F1:VG3Title")
TBLPROPERTIES ("hbase.table.name" = "hbase_Video_games_listed");

INSERT OVERWRITE TABLE Video_games_listed SELECT * FROM raw.Video_games_listed;

