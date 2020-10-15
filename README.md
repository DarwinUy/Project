# Hive-External-to-Internal

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



