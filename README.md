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

~~NOTE: update or delete a record in the "members" table that has a foreign key relationship with the "bookings" table is not possible, because there is at least one record in the "bookings" table that still references the record in the "members" table that you are trying to delete or update.

-- ISSUE 9: Delete based on a subquery
*/
In our previous exercises, we deleted a specific member who had never made a booking. How can we make that more general, to delete all members who have never made a booking?
*/

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

-- ISSUE 1:
*/

*/

-- ISSUE
*/

*/
