# postgres 表#

CREATE TABLE weather(
city varchar(80),
temp_lo int,   
temp_hi int,   
prcp real,   
date date);

CREATE TABLE cities(
name  varchar(80),
location point
);

DROP TABLE  tablename;

INSERT INTO weather(city,temp_lo,temp_hi,prcp,date)
VALUES('San Francisco',43,57,0.0,'1994-11-29');

INSERT INTO weather(date,city,temp_hi,temp_lo)
VALUES('1994-11-29','Hayward',54,37);


[postgres@pg96 ~]$ cat /home/postgres/weather.txt
 

'San Francisc3',43,57,0.0,'1994-11-29'
 

'San Francisc1',43,57,0.0,'1994-11-29'
 

'San Francisc2',43,57,0.0,'1994-11-29'

COPY weather FROM '/home/postgres/weather.txt' delimiter ',';

 
SELECT * FROM weather;

 
SELECT city,temp_lo,temp_hi,prcp,date FROM weather;
  
  
SELECT city,(temp_hi+temp_lo)/2 AS temp_avg,date FROM weather;
 
as 别名 是可选的
 
 
SELECT city,(temp_hi+temp_lo)/2 temp_avg,date FROM weather;

SELECT * FROM weather WHERE city = 'San Francisco' AND prcp >0.0;

SELECT * FROM weather ORDER BY city;

SELECT * FROM weather ORDER BY city,temp_lo;

SELECT DISTINCT city FROM weather;

SELECT DISTINCT city FROM weather ORDER BY city;

INSERT INTO cities(name,location) VALUES('San Francisco','-194,53');

SELECT * FROM weather,cities WHERE city=name;

SELECT city,temp_lo,temp_hi,prcp,date,location FROM weather,cities WHERE city=name;
 
内连接

SELECT * FROM weather INNER JOIN cities ON (weather.city=cities.name);

左外连接

SELECT * FROM weather LEFT OUTER JOIN cities ON (weather.city=cities.name);

自连接

SELECT W1.*,W2.* FROM weather W1,weather W2 where W1.city=W2.city;

