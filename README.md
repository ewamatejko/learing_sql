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
/*
Add multiple facilities in one command. Use the following values:
facid: 9, Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800. facid: 10, Name: 'Squash Court 2', membercost: 3.5, guestcost: 17.5, initialoutlay: 5000, monthlymaintenance: 80.
*/

-- ISSUE 3: Insert calculated data into a table

 /*
 We want to automatically generate the value for the next facid, rather than specifying it as a constant. Use the following values for everything else:
Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800.
*/

-- ISSUE 4: Update some existing data
/*
We made a mistake when entering the data for the second tennis court. The initial outlay was 10000 rather than 8000: you need to alter the data to fix the error.
*/

-- ISSUE 5: Update multiple rows and columns at the same time
/*
We want to increase the price of the tennis courts for both members and guests. Update the costs to be 6 for members, and 30 for guests.
*/

-- ISSUE 6: Update a row based on the contents of another row
/*
We want to alter the price of the second tennis court so that it costs 10% more than the first one. Try to do this without using constant values for the prices, so that we can reuse the statement if we want to.
*/

-- ISSUE 7: Delete all bookings
*/
As part of a clearout of our database, we want to delete all bookings from the cd.bookings table. How can we accomplish this?
*/

-- ISSUE 8: Delete a member from the cd.members table
*/
We want to remove member 37, who has never made a booking, from our database. How can we achieve that?
*/

-- ISSUE 9: Delete based on a subquery
*/
In our previous exercises, we deleted a specific member who had never made a booking. How can we make that more general, to delete all members who have never made a booking?
*/

-- AGGREGATION --

-- ISSUE 1:
*/

*/

-- ISSUE
*/

*/
