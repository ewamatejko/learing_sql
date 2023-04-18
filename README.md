# learing_sql

-- SQL EXERCISES
--https://pgexercises.com/questions

-- JOINS AND SUBQUERIES

-- ISSUE 1: Retrieve the start times of members' bookings
/*
How can you produce a list of the start times for bookings by members named 'David Farrell'?
*/
SELECT b.starttime, m.firstname, m.surname
FROM bookings b
JOIN members m
USING (memid)
WHERE firstname = 'David' AND surname = 'Farrell';

-- ISSUE 2: Work out the start times of bookings for tennis courts
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
/*
How can you produce a list of all members who have used a tennis court? Include in your output the name of the court, and the name of the member formatted as a single column. Ensure no duplicate data, and order by the member name followed by the facility name.
*/

-- ISSUE 6: Produce a list of costly bookings
-- ISSUE 7: Produce a list of all members, along with their recommender, using no joins.
-- ISSUE 8: Produce a list of costly bookings, using a subquery

