create database OlaCabs;

use olacabs;

create table Data
(pickup_date varchar(80),
pickup_time varchar(80),
pickup_datetime  varchar(80),
PickupArea varchar(80),
DropArea varchar(80),
Booking_id varchar(80),
Booking_type  varchar(80),
Booking_mode  varchar(80),
Driver_number  varchar(80),
Service_status  varchar(80),
Status varchar(80),
Fare varchar(80),
Distance Varchar(80),
Confirmed_at varchar(80)
);

create table Localities
(id INT,
Area Varchar(80),
city_id INT,
zone_id INT
);

select *
from localities;

select *
from data ;

SET SQL_SAFE_UPDATES=0;

/* 
1 	Find hour of 'pickup' and 'confirmed_at' time, and make a column of weekday as "Sun,Mon, etc"next to pickup_datetime			
2	Make a table with count of bookings with booking_type = p2p catgorized by booking mode as 'phone', 'online','app',etc			
3	Create columns for pickup and drop ZONES (using Localities data containing Zone IDs against each area) and fill corresponding values against pick-area and drop_area, using Sheet'Localities'			
4	Find top 5 drop zones in terms of  average revenue			
5	Find all unique driver numbers grouped by top 5 pickzones			
6	Make a list of top 10 driver by driver numbers in terms of fare collected where service_status is done, done-issue			
7	Make a hourwise table of bookings for week between Nov01-Nov-07 and highlight the hours with more than average no.of bookings day wise 
*/

# 1. Write the pseudo SQL queries for Q2,Q4,Q5 and Q7 

SELECT Booking_mode, COUNT(*) AS booking_count
FROM data
WHERE Booking_type = 'p2p'
GROUP BY Booking_mode
ORDER BY Booking_mode;



SELECT DropArea, AVG(Fare) AS AvgFare
FROM data
GROUP BY DropArea
ORDER BY AvgFare DESC
LIMIT 5;



WITH TopPickZones AS (
SELECT PickupArea
FROM data
GROUP BY PickupArea
ORDER BY COUNT(*) DESC
LIMIT 5
)
SELECT DISTINCT Driver_number, PickupArea
FROM data
WHERE PickupArea IN (SELECT PickupArea FROM TopPickZones);


WITH HourlyBookings AS (
SELECT EXTRACT(HOUR FROM pickup_time) AS booking_hour,
DATE(pickup_time) AS booking_date,
COUNT(*) AS booking_count
FROM data
WHERE Pickup_date BETWEEN '2023-11-01' AND '2023-11-07'
GROUP BY booking_hour, booking_date
),
AverageBookings AS (
SELECT booking_date, AVG(booking_count) AS avg_booking_count
FROM HourlyBookings
GROUP BY booking_date
)
SELECT hb.booking_date, hb.booking_hour, hb.booking_count,
CASE WHEN hb.booking_count > ab.avg_booking_count THEN 'highlight'
ELSE 'normal'
END AS highlight_status
FROM HourlyBookings hb JOIN AverageBookings ab ON hb.booking_date = ab.booking_date
ORDER BY hb.booking_date, hb.booking_hour;
