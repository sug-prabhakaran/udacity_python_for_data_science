 /* Set 1 - Question 1 - Family Friendly Movies By Quartile */

 SELECT film_title,
        c.name category_name,
        rental_count
   FROM (SELECT f.film_id,
                f.title film_title,
                COUNT(r.rental_id) AS rental_count
           FROM film f
                JOIN inventory i
                  ON f.film_id = i.film_id
                JOIN rental r
                  ON i.inventory_id = r.inventory_id
          GROUP BY 1, 2) sub
        JOIN film_category fc
     	 ON sub.film_id = fc.film_id
        JOIN category c
          ON fc.category_id = c.category_id
  WHERE c.name = 'Animation'
     OR c.name = 'Children'
     OR c.name = 'Classics'
     OR c.name = 'Comedy'
     OR c.name = 'Family'
     OR c.name = 'Music'
  ORDER BY 2, 1;

 /* Set 1 - Question 2 - Family Friendly Movies By Quartile */

 SELECT f.title,
        c.name category,
        f.rental_duration,
        NTILE(4) OVER (ORDER BY f.rental_duration) AS standard_quartile
   FROM film f
        JOIN film_category fc
          ON f.film_id = fc.film_id
        JOIN category c
          ON fc.category_id = c.category_id
  WHERE c.name = 'Animation'
     OR c.name = 'Children'
     OR c.name = 'Classics'
     OR c.name = 'Comedy'
     OR c.name = 'Family'
     OR c.name = 'Music';

 /* Set 1 - Question 3 - Category By Duration in Quartiles */

 SELECT category,
        standard_quartile,
        COUNT(*)
   FROM (SELECT f.title,
 	       c.name category,
       	       NTILE(4) OVER (ORDER BY f.rental_duration) AS standard_quartile
   	  FROM film f
   	       JOIN film_category fc
                ON f.film_id = fc.film_id
        	       JOIN category c
                ON fc.category_id = c.category_id
  	 WHERE c.name = 'Children'
             OR c.name = 'Music'
             OR c.name = 'Animation'
             OR c.name = 'Family'
             OR c.name = 'Classics'
             OR c.name = 'Comedy') sub
   GROUP BY 1, 2
   ORDER BY 1, 2;

 /* Set 2 - Question 1 - Store Comparison */

 SELECT DATE_TRUNC('month', r.rental_date) AS rental_month,
        s.store_id,
        COUNT(*) AS count_rentals
   FROM rental r
        JOIN staff s
          ON r.staff_id = s.staff_id
  GROUP BY 1, 2
  ORDER BY 2, 3 DESC;

 /*Set 2 - Question 2 - Top Paying Customer Payments by Month */

 SELECT DATE_TRUNC('month', payment_date) AS pay_month,
        customer_name,
        COUNT(p.amount) AS pay_countpermon,
        SUM(p.amount) AS pay_amount
   FROM (SELECT p.customer_id,
                CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
                SUM(p.amount)
           FROM payment p
                JOIN customer c
                  ON p.customer_id = c.customer_id
          GROUP BY 1, 2
          ORDER BY 3 DESC
          LIMIT 10) sub
   JOIN payment p
        ON sub.customer_id = p.customer_id
  GROUP BY 1, 2
  ORDER BY 2

 /*Set 2 - Question 3 - Top Paying Customer Payments by Month */

 WITH topten AS (
         SELECT p.customer_id,
                CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
                SUM(p.amount)
           FROM payment p
                JOIN customer c
                  ON p.customer_id = c.customer_id
          GROUP BY 1, 2
          ORDER BY 3 DESC
          LIMIT 10)

 SELECT DATE_TRUNC('month', p.payment_date) AS month,
 	    topten.customer_name,
        SUM(p.amount) mon_pay_amt,
        SUM(p.amount)-LAG(SUM(p.amount)) OVER (PARTITION BY topten.customer_name ORDER BY DATE_TRUNC('month', p.payment_date)) AS mon_pay_diff

   FROM topten
        JOIN payment p
          ON p.customer_id = topten.customer_id
  GROUP BY 1,2
  ORDER BY 4 DESC;
