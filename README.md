# learing_sql

-- SQL EXERCISES
--https://pgexercises.com/questions

-- JOINS AND SUBQUERIES --

-- ISSUE 1: Retrieve the start times of members' bookings
-- JOIN, WHERE, ORDER BY
/*
How can you produce a list of the start times for bookings by members named 'David Farrell'?
*/
SELECT b.starttime, m.firstname, m.surname
FROM bookings b
JOIN members m
USING (memid)
WHERE firstname = 'David' AND surname = 'Farrell';

-- ISSUE 2: Work out the start times of bookings for tennis courts
-- JOIN, WHERE, ORDER BY 
/*
How can you produce a list of the start times for bookings for tennis courts, for the date '2012-09-21'? Return a list of start time and facility name pairings, ordered by the time.
*/
SELECT b.starttime, f.name
FROM facilities f
JOIN bookings b
USING (facid)
WHERE (b.starttime >= '2012-09-21' AND b.starttime < '2012-09-22') 
		AND f.name IN ('Tennis Court 1', 'Tennis Court 2')
ORDER BY b.starttime;

-- ISSUE 3: Produce a list of all members who have recommended another member
-- JOIN, DISTINCT
-- !! JOINING A TABLE TO ITSELF !! 
/*
How can you output a list of all members who have recommended another member? Ensure that there are no duplicates in the list, and that results are ordered by (surname, firstname).
*/
SELECT DISTINCT(temp.recommendedby) AS members_who_recommend, mem.firstname, mem.surname
FROM members mem
JOIN members temp
ON mem.memid = temp.recommendedby
ORDER BY mem.surname, mem.firstname;

-- ISSUE 4: Produce a list of all members, along with their recommender
-- LEFT JOIN
-- !! JOINING A TABLE TO ITSELF !!

/*
How can you output a list of all members, including the individual who recommended them (if any)? Ensure that results are ordered by (surname, firstname).
*/
SELECT 	mem.memid, mem.surname, mem.firstname, 
		mem.recommendedby, 
		temp.surname AS recommendator_surname, 
		temp.firstname AS recommendator_firstname
FROM members mem
LEFT JOIN members temp
ON mem.recommendedby = temp.memid
ORDER BY mem.surname, mem.firstname;

-- ISSUE 5: Produce a list of all members who have used a tennis court
-- CONCAT/||, JOIN, DISTINCT
/*
How can you produce a list of all members who have used a tennis court? Include in your output the name of the court, and the name of the member formatted as a single column. Ensure no duplicate data, and order by the member name followed by the facility name.
*/
SELECT 	DISTINCT m.firstname || ' ' || m.surname AS name, 
		f.name 
FROM members m
JOIN bookings b
USING (memid)
JOIN facilities f
USING (facid)
WHERE f.name IN ('Tennis Court 1', 'Tennis Court 2')
ORDER BY 1,2;

-- ISSUE 6: Produce a list of costly bookings
-- CASE WHEN, CONCAT/||, JOIN
/*
How can you produce a list of bookings on the day of 2012-09-14 which will cost the member (or guest) more than $30? Remember that guests have different costs to members (the listed costs are per half-hour 'slot'), and the guest user is always ID 0. Include in your output the name of the facility, the name of the member formatted as a single column, and the cost. Order by descending cost, and do not use any subqueries.
*/
SELECT 	b.starttime, f.name AS facility,
		CONCAT(m.firstname, ' ', m.surname) AS fullname, 
		CASE WHEN b.memid = 0 THEN (f.guestcost*b.slots)
		ELSE (f.membercost*b.slots)
		END AS totalcost
FROM bookings b
JOIN facilities f
USING (facid)
JOIN members m
USING (memid)
WHERE 	(b.starttime >= '2012-09-14' AND b.starttime < '2012-09-15')
		AND ((b.memid = 0 AND (f.guestcost*b.slots) > 30) 
		OR (b.memid !=0 AND (f.membercost*b.slots) > 30))
ORDER BY 4 DESC;

-- ISSUE 7: Produce a list of all members, along with their recommender, using no joins.
-- SELECT SUBQUERY (ONE RESULT)
-- !! CORRELATED SUBQUERY !!
/*
How can you output a list of all members, including the individual who recommended them (if any), without using any joins? Ensure that there are no duplicates in the list, and that each firstname + surname pairing is formatted as a column and ordered.
*/
SELECT DISTINCT (CONCAT(m.firstname, ' ', m.surname)) AS member,
	(SELECT (r.firstname || ' ' || r.surname) AS recommendator
	FROM members r
	WHERE r.memid = m.recommendedby)
FROM members m
ORDER BY member;

-- ISSUE 8: Produce a list of costly bookings, using a subquery
-- FROM SUBQUERY (temporary table with own names used in SELECT)
/*
How can you produce a list of bookings on the day of 2012-09-14 which will cost the member (or guest) more than $30? Remember that guests have different costs to members (the listed costs are per half-hour 'slot'), and the guest user is always ID 0. Include in your output the name of the facility, the name of the member formatted as a single column, and the cost. Order by descending cost.
*/
SELECT facility,
		fullname,
		totalcost
FROM (SELECT f.name AS facility, 
	  CONCAT(m.firstname, ' ', m.surname) AS fullname,
	  	CASE WHEN b.memid = 0 THEN (f.guestcost*b.slots)
		ELSE (f.membercost*b.slots)
		END AS totalcost
		FROM bookings b
		JOIN facilities f
		USING (facid)
		JOIN members m
		USING (memid)
		WHERE 	(b.starttime >= '2012-09-14' AND b.starttime < '2012-09-15')
		)sub1
WHERE totalcost > 30
ORDER BY totalcost DESC;


-- INSERT DATA INTO TABLE --

-- ISSUE 1:  Insert some data into a table
-- INSERT INTO () VALUES ()
/*
The club is adding a new facility - a spa. We need to add it into the facilities table. Use the following values: facid: 9, Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800.
*/
INSERT INTO facilities 
			(facid, 
			name,
			membercost, 
			guestcost, 
			initialoutlay, 
			monthlymaintenance)
VALUES 			(9, 
			 'Spa',
			20,
			30,
			100000,
			800);
			

-- ISSUE 2: Insert multiple rows of data into a table
-- INSERT INTO SELECT UNION (removes duplicates) SELECT
/*
Add multiple facilities in one command. Use the following values:
facid: 11, Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800. facid: 10, Name: 'Squash Court 2', membercost: 3.5, guestcost: 17.5, initialoutlay: 5000, monthlymaintenance: 80.
*/
INSERT INTO facilities (facid, 
			name,
			membercost, 
			guestcost, 
			initialoutlay, 
			monthlymaintenance)
SELECT 		10, 'Squash Court 2', 3.5, 17.5, 5000, 80
UNION
SELECT 		11, 'Spa', 20, 30, 100000, 800;


-- ISSUE 3: Insert calculated data into a table
-- INSERT INTO, SUBQUERY IN SELECT !!

 /*
 We want to automatically generate the value for the next facid, rather than specifying it as a constant. Use the following values for everything else:
Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800.
*/
INSERT INTO facilities (facid, 
			name,
			membercost, 
			guestcost, 
			initialoutlay, 
			monthlymaintenance)
SELECT (SELECT MAX(facid) FROM facilities)+1, 'Spa,', 20, 30, 100000, 800;


-- ISSUE 4: Update some existing data
-- UPDATE SET WHERE 

/*
We made a mistake when entering the data for the facid 12 (Spa, instead of Spa). You need to alter the data to fix the error.
*/
UPDATE facilities
SET name = 'Spa'
WHERE facid = 12;
			

-- ISSUE 5: Update multiple rows and columns at the same time
-- UPDATE SET WHERE
/*
We want to increase the price of the tennis courts for both members and guests. Update the costs to be 6 for members, and 30 for guests.
*/
UPDATE facilities
SET membercost = 6, guestcost = 30
WHERE name IN ('Tennis Court 1', 'Tennis Court 2');


-- ISSUE 6: Update a row based on the contents of another row
-- UPDATE SET WHERE
/*
We want to alter the price of the second tennis court so that it costs 10% more than the first one. Try to do this without using constant values for the prices, so that we can reuse the statement if we want to.
*/
UPDATE facilities
SET membercost = 1.1*(SELECT membercost
				 FROM facilities
				 WHERE name = 'Tennis Court 1'),
	guestcost = 1.1*(SELECT guestcost
				 FROM facilities
				 WHERE name = 'Tennis Court 1')
WHERE name = 'Tennis Court 2';


-- ISSUE 7: Delete all bookings
-- DELETE FROM
*/
As part of a clearout of our database, we want to delete all bookings from the cd.bookings table. How can we accomplish this?
*/
DELETE FROM bookings;


-- ISSUE 8: Delete a member from the cd.members table
-- DELETE FROM WHERE
*/
We want to remove member 37, who has never made a booking, from our database. How can we achieve that?
*/
DELETE FROM members 
WHERE memid = 37;

~NOTE: update or delete a record in the "members" table that has a foreign key relationship with the "bookings" table is not possible, because there is at least one record in the "bookings" table that still references the record in the "members" table that you are trying to delete or update.

-- ISSUE 9: Delete based on a subquery
*/
In our previous exercises, we deleted a specific member who had never made a booking. How can we make that more general, to delete all members who have never made a booking?
*/
DELETE FROM members
WHERE memid NOT IN (SELECT memid 
		FROM bookings);

-- DIFFERENCE BETWEEN 'TRUNCATE', 'DELETE' and 'DROP'

In SQL, TRUNCATE and DELETE are two different commands used to remove data from a table. However, there are some significant differences between them.

TRUNCATE is a DDL (Data Definition Language) command that deletes all the rows from a table, and it is faster than DELETE as it deallocates the data pages from the table. The TRUNCATE operation is not TRANSACTIONAL, and it cannot be rolled back. Additionally, it resets the identity column value to its seed value.

DELETE, on the other hand, is a DML (Data Manipulation Language) command that removes specific rows from a table based on a condition or all rows without a condition. The DELETE operation is transactional, and it can be rolled back. Additionally, it does not reset the identity column value.

DROP deletes an entire table from the database schema, including all data, indexes, and triggers associated with the table. DROP is a DDL command that is not transactional, and it cannot be rolled back. Once you drop a table, it is permanently deleted from the database schema.

In summary, if you want to remove all rows from a table and reset the identity column value, you can use TRUNCATE. If you want to remove specific rows or roll back the operation, you should use DELETE. If you want to remove an entire table and all data associated with it, you should use DROP.

TRANSACTIONAL
In the context of databases, a transaction is a sequence of one or more database operations that are treated as a single logical unit of work. These operations can include inserting, updating, or deleting data from one or more tables.

A transaction is considered transactional when it has the following properties, often referred to as ACID properties:

Atomicity: A transaction is an atomic unit of work, meaning that it must be executed as a single, indivisible operation. If any part of the transaction fails, the entire transaction is rolled back, and all changes made during the transaction are undone, so that the database is returned to its original state.

Consistency: A transaction must ensure that the database remains in a valid state after it has been completed, meaning that any integrity constraints, such as foreign key or check constraints, are satisfied.

Isolation: A transaction must be isolated from other transactions that are executing concurrently, so that each transaction appears to be executed in isolation from others, even though they are actually executing concurrently.

Durability: Once a transaction has been committed, its changes must be durable and must survive any subsequent system failures, such as power outages, hardware failures, or other types of crashes.

-- AGGREGATION --

-- ISSUE 1: Count the number of facilities
*/
For our first foray into aggregates, we're going to stick to something simple. We want to know how many facilities exist - simply produce a total count.
*/
SELECT COUNT(DISTINCT(facid))
FROM facilities;

~NOTE: 
COUNT(*) simply returns the number of rows
COUNT(address) counts the number of non-null addresses in the result set.
COUNT(DISTINCT address) counts the number of different addresses in the facilities table.


-- ISSUE 2: Count the number of expensive facilities
*/
Produce a count of the number of facilities that have a cost to guests of 10 or more.
*/
SELECT COUNT(facid)
FROM facilities
WHERE guestcost >=10;


-- ISSUE 3: Count the number of recommendations each member makes.
-- SUBQUERY IN FROM, CASE WHEN, SUBSTITUTING NULL BY INTEGER
*/
Produce a count of the number of recommendations each member has made. Order by member ID.
*/
SELECT sub1.recommendedby_v1, COUNT(sub1.recommendedby_v1) 
FROM 	(SELECT CASE WHEN recommendedby IS NULL
		THEN 0
		ELSE recommendedby
		END AS recommendedby_v1
		FROM members)sub1
GROUP BY sub1.recommendedby_v1
ORDER BY 1;


-- ISSUE 4: List the total slots booked per facility
*/
Produce a list of the total number of slots booked per facility. For now, just produce an output table consisting of facility id and slots, sorted by facility id.
*/
SELECT facid, SUM(slots) AS total_slots
FROM bookings
GROUP BY facid
ORDER BY 1;


-- ISSUE 5: List the total slots booked per facility in a given month
*/
Produce a list of the total number of slots booked per facility in the month of September 2012. Produce an output table consisting of facility id and slots, sorted by the number of slots.
*/
SELECT facid, SUM(slots) AS total_slots
FROM bookings
WHERE starttime >= '2012-09-01' AND starttime < '2012-10-01'
GROUP BY facid
ORDER BY 2;


-- ISSUE 6: List the total slots booked per facility per month
-- EXTRACT, DATE_TRUNC 
*/
Produce a list of the total number of slots booked per facility per month in the year of 2012. Produce an output table consisting of facility id and slots, sorted by the id and month.
*/
SELECT 	facid, DATE_TRUNC('month', starttime) AS month, 
		SUM(slots) AS total_slots
FROM bookings
GROUP BY 1,2
ORDER BY 1,2;

SELECT 	facid, EXTRACT(month FROM starttime) AS month, 
		SUM(slots) AS total_slots
FROM bookings
GROUP BY 1,2
ORDER BY 1,2;


-- ISSUE 7: Find the count of members who have made at least one booking
*/
Find the total number of members (including guests) who have made at least one booking.
*/
SELECT COUNT(DISTINCT(memid))
FROM bookings;


-- ISSUE 8: List facilities with more than 1000 slots booked
-- HAVING, GROUP BY
*/
Produce a list of facilities with more than 1000 slots booked. Produce an output table consisting of facility id and slots, sorted by facility id.
*/
SELECT facid, SUM(slots)
FROM bookings
GROUP BY 1
HAVING SUM(slots)>1000
ORDER BY facid;

-- ISSUE 9: Find the total revenue of each facility
*/
Produce a list of facilities along with their total revenue. The output table should consist of facility name and revenue, sorted by revenue. Remember that there's a different cost for guests and members!
*/
SELECT sub1.name, SUM(sub1.revenue)
FROM (SELECT f.name, f.facid, 
	  CASE WHEN b.memid != 0 THEN (f.membercost*b.slots)
		ELSE f.guestcost*b.slots
		END AS revenue
		FROM bookings b
		JOIN facilities f
		USING (facid))sub1
GROUP BY 1
ORDER BY 2;


-- ISSUE 10: Find facilities with a total revenue less than 1000
-- WITH SUBQUERY, CASE WHEN, GROUP BY...

*/
Produce a list of facilities with a total revenue less than 1000. Produce an output table consisting of facility name and revenue, sorted by revenue. Remember that there's a different cost for guests and members!
*/
SELECT sub1.name, SUM(sub1.revenue)
FROM (SELECT f.name, f.facid, 
	  CASE WHEN b.memid != 0 THEN (f.membercost*b.slots)
		ELSE f.guestcost*b.slots
		END AS revenue
		FROM bookings b
		JOIN facilities f
		USING (facid))sub1
GROUP BY 1
HAVING SUM(sub1.revenue) < 1000
ORDER BY 2;

-- ISSUE 11: Output the facility id that has the highest number of slots booked
/*
Output the facility id that has the highest number of slots booked. For bonus points, try a version without a LIMIT clause. This version will probably look messy!
*/
SELECT facid, SUM(slots)
FROM bookings
GROUP BY facid
ORDER BY 2 DESC
LIMIT 1;

!!! OPTION WITH 'WITH' !!!

WITH sum_slots AS (SELECT facid, SUM(slots) as total_slots
		FROM bookings
		GROUP BY facid)
SELECT facid, total_slots
FROM sum_slots
WHERE total_slots = (SELECT MAX(total_slots) 
			FROM sum_slots);


-- ISSUE 12: List the total slots booked per facility per month, part 2
/*
Produce a list of the total number of slots booked per facility per month in the year of 2012. In this version, include output rows containing totals for all months per facility, and a total for all months for all facilities. The output table should consist of facility id, month and slots, sorted by the id and month. When calculating the aggregated values for all months and all facids, return null values in the month and facid columns.
*/

-- ISSUE 13: List the total hours booked per named facility
/*
Produce a list of the total number of hours booked per facility, remembering that a slot lasts half an hour. The output table should consist of the facility id, name, and hours booked, sorted by facility id. Try formatting the hours to two decimal places.
*/

-- ISSUE 14: List each member's first booking after September 1st 2012
/*
Produce a list of each member name, id, and their first booking after September 1st 2012. Order by member ID.
*/

-- ISSUE 15: Produce a list of member names, with each row containing the total member count
/*
Produce a list of member names, with each row containing the total member count. Order by join date, and include guest members.
*/

-- ISSUE 16: Produce a numbered list of members
/*
Produce a monotonically increasing numbered list of members (including guests), ordered by their date of joining. Remember that member IDs are not guaranteed to be sequential.
*/

-- ISSUE 17: Output the facility id that has the highest number of slots booked, again
/*
Output the facility id that has the highest number of slots booked. Ensure that in the event of a tie, all tieing results get output.
*/

-- ISSUE 18: Rank members by (rounded) hours used
/*
Produce a list of members (including guests), along with the number of hours they've booked in facilities, rounded to the nearest ten hours. Rank them by this rounded figure, producing output of first name, surname, rounded hours, rank. Sort by rank, surname, and first name.
*/

-- ISSUE 19: Find the top three revenue generating facilities
/*
Produce a list of the top three revenue generating facilities (including ties). Output facility name and rank, sorted by rank and facility name.
*/

-- ISSUE 20: Classify facilities by value
/*
Classify facilities into equally sized groups of high, average, and low based on their revenue. Order by classification and facility name.
*/

-- ISSUE 21: Calculate the payback time for each facility
/*
Based on the 3 complete months of data so far, calculate the amount of time each facility will take to repay its cost of ownership. Remember to take into account ongoing monthly maintenance. Output facility name and payback time in months, order by facility name. Don't worry about differences in month lengths, we're only looking for a rough value here!
*/

-- ISSUE 22: Calculate a rolling average of total revenue
/*
For each day in August 2012, calculate a rolling average of total revenue over the previous 15 days. Output should contain date and revenue columns, sorted by the date. Remember to account for the possibility of a day having zero revenue. This one's a bit tough, so don't be afraid to check out the hint!
*/


-- DATE --

-- ISSUE 1: Produce a timestamp for 1 a.m. on the 31st of August 2012
-- TIMESTAMP (date+time), CAST
/*
Produce a timestamp for 1 a.m. on the 31st of August 2012.
*/
SELECT '2012-08-31 01:00:00' ::TIMESTAMP;

-- NOTE: The CAST() function converts a value (of any type) into the specified datatype.

-- ISSUE 2: Subtract timestamps from each other
-- INTERVAL (BETWEEN TIMESTAMPS)
/*
Find the result of subtracting the timestamp '2012-07-30 01:00:00' from the timestamp '2012-08-31 01:00:00'
*/
SELECT TIMESTAMP '2012-08-31 01:00:00' - TIMESTAMP '2012-07-30 01:00:00' AS interval;


-- ISSUE 3: Generate a list of all the dates in October 2012
-- GENERATE_SERIES
/*
Produce a list of all the dates in October 2012. They can be output as a timestamp (with time set to midnight) or a date.
*/
SELECT GENERATE_SERIES (timestamp '2012-10-01', timestamp '2012-10-31', interval '1 day') AS list_dates;

-- NOTE:  GENERATE_SERIES: This function allows you to generate a list of dates or numbers, specifying a start, an end, and an increment value


-- ISSUE 4: Get the day of the month from a timestamp
-- EXTRACT (TIMESTAMP TO INTEGER)
/*
Get the day of the month from the timestamp '2012-08-31' as an integer.
*/
SELECT EXTRACT (day FROM timestamp '2012-08-31');


-- ISSUE 5: Work out the number of seconds between timestamps
-- EXTRACT EPOCH
/*
Work out the number of seconds between the timestamps '2012-08-31 01:00:00' and '2012-09-02 00:00:00'
*/
SELECT EXTRACT (EPOCH FROM (TIMESTAMP '2012-09-02 00:00:00' - '2012-08-31 01:00:00'));

-- NOTE: EXTRACT EPOCH -Extracting the epoch converts an interval or timestamp into a number of seconds


-- ISSUE 6: Work out the number of days in each month of 2012
-- EXTRACT, GENERATE_SERIES, INTERVAL
/*
For each month of the year in 2012, output the number of days in that month. Format the output as an integer column containing the month of the year, and a second column containing an interval data type.
*/
SELECT EXTRACT (month FROM sub1.time_month) AS month_number,
		((sub1.time_month + interval '1 month') - sub1.time_month) AS days_number
FROM (SELECT GENERATE_SERIES (timestamp '2012-01-01', 
	timestamp '2012-12-31', 
	interval '1 month') AS time_month) sub1;

-- NOTE: subtracting two timestamps will always produce an interval in terms of days (or portions of a day). You won't just get an answer in terms of months or years, because the length of those time periods is variable.
-- NOTE: no need of using data from my database to operate on the calendar days!


-- ISSUE 7: Work out the number of days remaining in the month
-- DATE_TRUNC, INTERVAL
/*
For any given timestamp, work out the number of days remaining in the month. The current day should count as a whole day, regardless of the time. Use '2012-02-11 01:00:00' as an example timestamp for the purposes of making the answer. Format the output as a single interval value.
*/
SELECT ((DATE_TRUNC ('month', sub1.current_date) + interval '1 month')
		- DATE_TRUNC ('day', sub1.current_date)) AS days_remaining
FROM (SELECT timestamp '2012-02-11' AS current_date) sub1;


-- ISSUE 8: Work out the end time of bookings
-- INTERVAL
/*
Return a list of the start and end time of the last 10 bookings (ordered by the time at which they end, followed by the time at which they start) in the system.
*/
SELECT (starttime + interval '0.5 hour'*slots) AS endtime,
		starttime
FROM bookings
ORDER BY 1 DESC, 2 DESC
LIMIT 10;


-- ISSUE 9: Return a count of bookings for each month
-- DATE_TRUNC, COUNT, GROUP BY
/*
Return a count of bookings for each month, sorted by month
*/
SELECT DATE_TRUNC ('month', starttime) as time,
	COUNT(bookid)
FROM bookings
GROUP BY 1 
ORDER BY 1;

-- ISSUE 10: Work out the utilisation percentage for each facility by month
/*
Work out the utilisation percentage for each facility by month, sorted by name and month, rounded to 1 decimal place. Opening time is 8am, closing time is 8.30pm. You can treat every month as a full month, regardless of if there were some dates the club was not open.
*/
with errors!
SELECT sub1.facid, sub1.name, sub1.month_date, ROUND((sum_slots/monthlyslots*100),1) AS utilisation
FROM ((SELECT facid, f.name, sub1.month_date, sub1.sum_slots,	
		CAST(25*(CAST((month_date + interval '1 month') AS date)
		- CAST(month_date AS date)) AS numeric),1) AS monthlyslots  
		FROM (SELECT facid, DATE_TRUNC ('month', starttime) AS month_date,
		SUM(slots) AS sum_slots
		FROM bookings
		JOIN facilities f
	  	USING (facid)
		GROUP BY 1, 2, 3, 4
		ORDER BY 1, 2))sub1
ORDER BY 1;
with errors!!!

-- STRING OPERATIONS --

-- ISSUE 1: Format the names of members
/*
Output the names of all members, formatted as 'Surname, Firstname'
*/

-- NOTE: Subtracting date types gives us an integer number of days !!!


-- ISSUE 2: Find facilities by a name prefix
/*
Find all facilities whose name begins with 'Tennis'. Retrieve all columns.
*/

-- ISSUE 3: Perform a case-insensitive search
/*
Perform a case-insensitive search to find all facilities whose name begins with 'tennis'. Retrieve all columns.
*/

-- ISSUE 4: Find telephone numbers with parentheses
/*
You've noticed that the club's member table has telephone numbers with very inconsistent formatting. You'd like to find all the telephone numbers that contain parentheses, returning the member ID and telephone number sorted by member ID.
*/


-- ISSUE 5: Pad zip codes with leading zeroes
/*
The zip codes in our example dataset have had leading zeroes removed from them by virtue of being stored as a numeric type. Retrieve all zip codes from the members table, padding any zip codes less than 5 characters long with leading zeroes. Order by the new zip code.
*/

-- ISSUE 6: Count the number of members whose surname starts with each letter of the alphabet
/*
You'd like to produce a count of how many members you have whose surname starts with each letter of the alphabet. Sort by the letter, and don't worry about printing out a letter if the count is 0.
*/

-- ISSUE 7: Clean up telephone numbers
/*
The telephone numbers in the database are very inconsistently formatted. You'd like to print a list of member ids and numbers that have had '-','(',')', and ' ' characters removed. Order by member id.
*/

-- RECURSIVE QUERIES --

-- ISSUE 1: Find the upward recommendation chain for member ID 27
/*
Find the upward recommendation chain for member ID 27: that is, the member who recommended them, and the member who recommended that member, and so on. Return member ID, first name, and surname. Order by descending member id.
*/

-- ISSUE 2: Find the downward recommendation chain for member ID 1
/*
Find the downward recommendation chain for member ID 1: that is, the members they recommended, the members those members recommended, and so on. Return member ID and name, and order by ascending member id.
*/

-- ISSUE 3: Produce a CTE that can return the upward recommendation chain for any member
/*
Produce a CTE that can return the upward recommendation chain for any member. You should be able to select recommender from recommenders where member=x. Demonstrate it by getting the chains for members 12 and 22. Results table should have member and recommender, ordered by member ascending, recommender descending.
*/

-- ISSUE 
/*

*/

-- ISSUE 
/*

*/

-- ISSUE 
/*

*/

-- ISSUE 
/*

*/

-- ISSUE 
/*

*/

-- ISSUE 
/*

*/

-- ISSUE 
/*

*/

-- ISSUE 
/*

*/

-- ISSUE 
/*

*/

-- ISSUE 
/*

*/

-- ISSUE 
/*

*/

-- ISSUE 
/*

*/

-- ISSUE 
/*

*/
