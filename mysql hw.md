# 1a)
use sakila;
SELECT first_name, last_name FROM actor;

# 1b)
SELECT CONCAT_WS(' ', first_name, last_name) AS 'Actor Name' FROM actor;

# 2a)
SELECT actor_id, first_name, last_name FROM actor
WHERE first_name = 'Joe';

# 2b)
SELECT first_name, last_name FROM actor
WHERE last_name LIKE '%GEN%';

# 2c)
SELECT last_name, first_name FROM actor
WHERE last_name LIKE '%LI%';

# 2d)
SELECT country_id, country FROM country
WHERE country IN ('Afghanistan', 'Bangladesh', 'China');

# 3a)
ALTER TABLE actor
ADD COLUMN middle_name VARCHAR(45) AFTER first_name;

# 3b)
ALTER TABLE actor MODIFY middle_name BLOB;

# 3c)
ALTER TABLE actor DROP COLUMN middle_name; 

# 4a)
SELECT last_name, COUNT(*) count FROM actor
GROUP BY last_name HAVING count >= 1;

# 4b)
SELECT last_name, COUNT(*) count FROM actor
GROUP BY last_name HAVING count >= 2;

# 4c)
UPDATE actor
SET first_name = 'HARPO'
WHERE first_name = 'GROUCHO' AND last_name = 'WILLIAMS';

# 4d)
UPDATE actor
SET first_name = 'GROUCHO'
WHERE first_name = 'HARPO' AND last_name = 'WILLIAMS';

UPDATE actor
SET first_name = 'MUCHO GROUCHO'
WHERE first_name = 'GROUCHO' AND last_name <> 'WILLIAMS';

# 5a)
SHOW CREATE TABLE address;

# 6a)
SELECT first_name, last_name, addr.address
FROM staff s
JOIN address addr
ON s.address_id = addr.address_id;

# 6b)
SELECT first_name, last_name, t.total_amount, t.date
FROM staff s
JOIN (
SELECT staff_id, SUM(amount) total_amount, DATE_FORMAT(payment_date, '%Y-%m') date FROM payment
WHERE MONTH(payment_date) = 8
GROUP BY staff_id HAVING total_amount
) t
ON s.staff_id = t.staff_id;

# 6c)
SELECT title, fa.actor_count
FROM film f
JOIN (
SELECT film_id, COUNT(*) actor_count FROM film_actor
GROUP BY film_id HAVING actor_count >= 1) fa
ON f.film_id = fa.film_id;

# 6d)
SELECT film_id, title
FROM film
WHERE title ='Hunchback Impossible'; #Query for film id

SELECT Count(*) inventory_count       #Query for inventory count given id
FROM inventory
WHERE film_id = 439
GROUP BY film_id HAVING inventory_count >= 1;

# 6e)
SELECT last_name, first_name, t.total_paid
FROM customer c
JOIN (
SELECT customer_id, SUM(amount) total_paid
FROM payment
GROUP BY customer_id HAVING total_paid
) t
ON c.customer_id = t.customer_id
ORDER BY last_name;

# 7a)
SELECT title FROM film
WHERE title LIKE ('K%') OR title LIKE ('Q%') AND language_id IN 
(
SELECT language_id FROM language
WHERE name = 'English'
);

# 7b)
SELECT first_name, last_name
FROM actor
WHERE actor_id IN 
(
SELECT actor_id FROM film_actor
WHERE film_id = (SELECT film_id FROM film
WHERE title = 'ALONE TRIP')
);

# 7c)
SELECT first_name, last_name, email
FROM customer cus
LEFT OUTER JOIN address addr
ON cus.address_id = addr.address_id
LEFT OUTER JOIN city cty
ON addr.city_id = cty.city_id
LEFT OUTER JOIN country cnty
ON cty.country_id = cnty.country_id
WHERE cnty.country = 'Canada';

# 7d)
SELECT title
FROM film
WHERE film_id IN 
(
SELECT film_id 
FROM film_category
WHERE category_id = (
	SELECT category_id
	FROM category
	WHERE name = 'Family')
);

# 7e)
SELECT title, COALESCE(frc.film_rental_count, 0) film_rental_count # Query for film title and number of times it was rented
FROM film f                                                        # in descending order
LEFT OUTER JOIN 
( 
 SELECT film_id, SUM(COALESCE(r.inv_rental_count, 0)) film_rental_count #  Query for film_id and how many times 
 FROM inventory inv                                                     #  a film is rented from all stores
 LEFT OUTER JOIN 
	(
	 SELECT inventory_id, COUNT(*) inv_rental_count   # Query for inventory_id and number of times an inventory item was rented
     FROM rental
     GROUP BY inventory_id HAVING inv_rental_count >= 1
     ) r	
  ON inv.inventory_id = r.inventory_id
  GROUP BY film_id HAVING film_rental_count
) frc
ON f.film_id = frc.film_id
ORDER BY frc.film_rental_count DESC;

# 7f)
SELECT staff_id store_id, SUM(amount) staff_earning # staff_id is the same as store_id
FROM payment
GROUP BY staff_id HAVING staff_earning;

# 7g)
SELECT store_id, c.city, cntry.country
FROM store s
JOIN address addr
ON s.address_id = addr.address_id
JOIN city c
ON c.city_id = addr.city_id
JOIN country cntry
ON c.country_id = cntry.country_id;

# 7h)
SELECT name, catesub.gross_revenue		
FROM category c
JOIN
(
	SELECT category_id, SUM(filmsub.filmrev) gross_revenue  #Query for category id and gross rev for each category
	FROM film_category fc
	JOIN
	(
		SELECT film_id, SUM(rev.invrev) filmrev      #Query for film id and gross rev for each film
		FROM inventory inv
		JOIN
		(	SELECT inventory_id, SUM(amount) invrev		#Query for inventory id and gross rev for each inventory item
			FROM payment p 
			JOIN rental r
			ON p.rental_id = r.rental_id
			GROUP BY r.inventory_id HAVING invrev
		) rev
		ON inv.inventory_id = rev.inventory_id
		GROUP BY film_id HAVING filmrev
	) filmsub
	ON fc.film_id = filmsub.film_id
	GROUP BY category_id HAVING gross_revenue
	) catesub
ON c.category_id = catesub.category_id
ORDER BY catesub.gross_revenue DESC
LIMIT 5;

# 8a)
CREATE VIEW top_five_genres
AS SELECT name, catesub.gross_revenue		
FROM category c
JOIN
(
	SELECT category_id, SUM(filmsub.filmrev) gross_revenue  #Query for category id and gross rev for each category
	FROM film_category fc
	JOIN
	(
		SELECT film_id, SUM(rev.invrev) filmrev      #Query for film id and gross rev for each film
		FROM inventory inv
		JOIN
		(	SELECT inventory_id, SUM(amount) invrev		#Query for inventory id and gross rev for each inventory item
			FROM payment p 
			JOIN rental r
			ON p.rental_id = r.rental_id
			GROUP BY r.inventory_id HAVING invrev
		) rev
		ON inv.inventory_id = rev.inventory_id
		GROUP BY film_id HAVING filmrev
	) filmsub
	ON fc.film_id = filmsub.film_id
	GROUP BY category_id HAVING gross_revenue
	) catesub
ON c.category_id = catesub.category_id
ORDER BY catesub.gross_revenue DESC
LIMIT 5;

# 8b)
SELECT * FROM top_five_genres ;

# 8c)
DROP VIEW top_five_genres;