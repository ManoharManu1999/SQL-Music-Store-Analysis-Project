-- Q1 Who is the senior most employee based on job title? 

SELECT 
* 
FROM employee e 
ORDER BY e.levels DESC
LIMIT 1;

-- Q2 Which countries have the most Invoices?

SELECT 
	i.billing_country,
	COUNT(billing_country) AS invoices
FROM invoice i 
GROUP BY billing_country
ORDER BY COUNT(billing_country) DESC;

-- Q3 What are top 3 values of total invoice? 

SELECT 
	invoice_id,
	total
FROM invoice
ORDER BY total DESC
LIMIT 3;

/* Q4 Which city has the best customers? We would like to throw a promotional Music 
Festival in the city we made the most money. Write a query that returns one city that 
has the highest sum of invoice totals. Return both the city name & sum of all invoice 
totals */

SELECT 
	billing_city,
	SUM(total) AS total_sales
FROM invoice
GROUP BY billing_city 
ORDER BY SUM(total) DESC
LIMIT 1;

/* Q5 Who is the best customer? The customer who has spent the most money will be 
declared the best customer. Write a query that returns the person who has spent the 
most money  */

SELECT 
	i.customer_id,
	c.first_name,
	c.last_name,
	SUM(total) AS total_sales
FROM invoice i 
JOIN customer c 
ON c.customer_id = i.customer_id 
GROUP BY i.customer_id,
	c.first_name ,
	c.last_name
ORDER BY SUM(total) DESC
LIMIT 1;

/* Q6 Write query to return the email, first name, last name, & Genre of all Rock Music 
listeners. Return your list ordered alphabetically by email starting with A */

SELECT 
	DISTINCT 
	c.email,
	c.first_name,
	c.last_name
FROM customer c
JOIN invoice i 
ON c.customer_id = i.customer_id 
LEFT JOIN invoice_line il 
ON i.invoice_id = il.invoice_id 
LEFT JOIN track t 
ON il.track_id = t.track_id  
LEFT JOIN genre g 
ON t.genre_id = g.genre_id 
WHERE g.name = 'Rock'
ORDER BY c.email;

/* Q7 Let's invite the artists who have written the most rock music in our dataset. Write a 
query that returns the Artist name and total track count of the top 10 rock bands */

SELECT 
	a.artist_id,
	a.name,
	COUNT(t.track_id )
FROM artist a 
JOIN album a2 
ON a.artist_id = a2.artist_id 
JOIN track t
ON a2.album_id = t.album_id 
JOIN Genre g
ON t.genre_id = g.genre_id 
WHERE g.name = 'Rock'
GROUP BY a.name, a.artist_id
ORDER BY COUNT(t.track_id ) DESC
LIMIT 10;

/* Q8 Return all the track names that have a song length longer than the average song length. 
Return the Name and Milliseconds for each track. Order by the song length with the 
longest songs listed first */

SELECT 	
	name,
	milliseconds 
FROM track
WHERE milliseconds > (SELECT AVG(milliseconds) FROM track)
ORDER BY milliseconds DESC;

/* Q9 Find how much amount spent by each customer on artists? Write a query to return 
customer name, artist name and total spent */

SELECT
	c.customer_id,
	c.first_name,
	c.last_name,
	a2.name AS artist_name,
	SUM(iL.unit_price * IL.quantity ) AS total_spent
FROM customer c
JOIN invoice i 
ON c.customer_id = i.customer_id 
JOIN invoice_line il 
ON i.invoice_id = il.invoice_id 
JOIN track t 
ON il.track_id = t.track_id 
JOIN album a 
ON t.album_id = a.album_id 
JOIN artist a2 
ON a.artist_id = a2.artist_id 
GROUP BY 
	c.customer_id,
	c.first_name,
	c.last_name,
	a2.name
ORDER BY SUM(iL.unit_price * IL.quantity ) DESC; 


/* Q10 How much did each customer spend on the best-selling artist only? */

WITH cte_top_artist AS(
SELECT 
	a.artist_id,
	a.name AS artist_name,
	SUM(il.quantity * il.unit_price)
FROM artist a 
JOIN album a2 
	ON a.artist_id = a2.artist_id 
JOIN track t 
	ON a2.album_id = t.album_id 
JOIN invoice_line il 
	ON t.track_id = il.track_id 
JOIN invoice i 
	ON il.invoice_id = i.invoice_id 
JOIN customer c 
	ON i.customer_id = c.customer_id 
GROUP BY a.artist_id,
		a.name
ORDER BY SUM(il.quantity * il.unit_price) DESC 
LIMIT 1
)
SELECT c.customer_id, 
	   c.first_name, 
	   c.last_name, 
	   bsa.artist_name, 
	   SUM(il.unit_price*il.quantity) AS amount_spent
FROM invoice i
JOIN customer c 
	ON c.customer_id = i.customer_id
JOIN invoice_line il 
	ON il.invoice_id = i.invoice_id
JOIN track t 
	ON t.track_id = il.track_id
JOIN album alb 
	ON alb.album_id = t.album_id
JOIN cte_top_artist bsa 
	ON bsa.artist_id = alb.artist_id
GROUP BY 1,2,3,4
ORDER BY 5 DESC;


/* Q11 We want to find out the most popular music Genre for each country. We determine the 
most popular genre as the genre with the highest amount of purchases. Write a query 
that returns each country along with the top Genre. For countries where the maximum 
number of purchases is shared return all Genres */

SELECT 
	t.country,
	t.genre_name,
	t.total_sales 
FROM(
SELECT 
	c.country AS country,
	g.name AS genre_name,
	SUM(i.total) AS total_sales,
	RANK() OVER(PARTITION BY country ORDER BY SUM(i.total) DESC) AS ranking
FROM genre g 
JOIN track t
ON g.genre_id = t.genre_id 
JOIN invoice_line il  
ON t.track_id = il.track_id 
JOIN invoice i 
ON il.invoice_id = i.invoice_id 
JOIN customer c
ON i.customer_id = c.customer_id 
GROUP BY g.name,c.country
ORDER BY country, 3 DESC)t
WHERE ranking = 1;


/* Q12 Write a query that determines the customer that has spent the most on music for each 
country. Write a query that returns the country along with the top customer and how 
much they spent. For countries where the top amount spent is shared, provide all 
customers who spent this amount*/

-- Using Subquery

SELECT t.customer_id ,
	t.first_name,
	t.last_name,
	t.country,
	t.total_spending 
FROM
(SELECT
	c.customer_id,
	c.first_name,
	c.last_name,
	c.country,
	SUM(i.total) AS total_spending,
	RANK() OVER(PARTiTION BY c.country ORDER BY SUM(i.total) DESC) AS ranking
FROM customer c
JOIN invoice i 
ON c.customer_id = i.customer_id 
GROUP BY 	c.customer_id,
	c.first_name,
	c.last_name,
	c.country)t
WHERE t.ranking =1;

--- Using CTE method

WITH cte_top_customer AS
(
	SELECT
	c.customer_id,
	c.first_name,
	c.last_name,
	c.country,
	SUM(i.total) AS total_spending,
	RANK() OVER(PARTiTION BY c.country ORDER BY SUM(i.total) DESC) AS ranking
FROM customer c
JOIN invoice i 
ON c.customer_id = i.customer_id 
GROUP BY 	c.customer_id,
	c.first_name,
	c.last_name,
	c.country
)
SELECT
	customer_id ,
	first_name,
	last_name,
	country,
	total_spending 
FROM cte_top_customer
WHERE ranking =1
