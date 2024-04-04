# Schema Design

## Data setup

[Steps to Load the `dvdrental` database](https://www.postgresqltutorial.com/postgresql-getting-started/load-postgresql-sample-database/)

Note: Add postgres database `bin` path to your environmental variable `C:\Program Files\PostgreSQL\16\bin`

[Download ER Diagram](https://www.postgresqltutorial.com/wp-content/uploads/2018/03/printable-postgresql-sample-database-diagram.pdf)

## Q1- Find Total Revenue geneated by each flim?

```SQL
/*
steps:
1. join the tables based on the ER Diagram
2. select required columns for final result
3. group by flim and sum the amount
*/

SELECT film.title AS film_title, SUM(payment.amount) AS total_amount
FROM film
JOIN inventory ON film.film_id = inventory.film_id
JOIN rental ON inventory.inventory_id = rental.inventory_id
JOIN payment ON rental.rental_id = payment.rental_id
join customer ON customer.customer_id = payment.customer_id 
join address on address.address_id =customer.address_id 
join city on city.city_id =address.city_id 
GROUP BY film.title 
order by SUM(payment.amount) DESC;
```

## Q2- Select the top 3 city's generated high revenue?

```SQL
SELECT city.city AS city_name, SUM(payment.amount) AS total_amount
FROM film
JOIN inventory ON film.film_id = inventory.film_id
JOIN rental ON inventory.inventory_id = rental.inventory_id
JOIN payment ON rental.rental_id = payment.rental_id
JOIN customer ON customer.customer_id = payment.customer_id 
JOIN address ON address.address_id = customer.address_id 
JOIN city ON city.city_id = address.city_id 
GROUP BY city.city
ORDER BY total_amount DESC
LIMIT 3;
```
