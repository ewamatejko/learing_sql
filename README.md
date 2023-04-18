# learing_sql

-- SQL EXERCISES
--https://pgexercises.com/questions

-- JOINS AND SUBQUERIES

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
SELECT 	b.starttime, f.name,
		CONCAT(m.firstname, ' ', m.surname), 
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
/*
How can you produce a list of bookings on the day of 2012-09-14 which will cost the member (or guest) more than $30? Remember that guests have different costs to members (the listed costs are per half-hour 'slot'), and the guest user is always ID 0. Include in your output the name of the facility, the name of the member formatted as a single column, and the cost. Order by descending cost.
*/


