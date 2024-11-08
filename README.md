## Online sport Revenue

Performing various SQL queries on product data, including counts, aggregations, correlations, and common table expressions (CTEs), to analyze sales performance, pricing trends, brand popularity, and product categorization in order to derive insights.


### Count all columns as total_rows 
 1) Count the number of non-missing entries for description, listing_price, and last_visited
 2) Join info, finance, and traffic

```sql


        SELECT COUNT(*) AS total_rows,
             COUNT(description) AS count_description,
             COUNT(listing_price) AS count_listing_price,
           COUNT(last_visited) AS count_last_visited
        FROM info
        INNER JOIN finance 
          ON info.product_id = finance.product_id
        INNER JOIN traffic
        ON info.product_id = traffic.product_id


```
	
### Select the brand, listing_price as an integer, and a count of all products in finance 
  1) Join brands to finance on product_id
  2) Filter for products with a listing_price more than zero
 3) Aggregate results by brand and listing_price, and sort the results by listing_price in descending order

```sql


        SELECT b.brand, f.listing_price::INTEGER, COUNT (f.product_id)
          FROM finance AS f
          INNER JOIN brands AS b
            ON b.product_id = f.product_id
        WHERE listing_price > 0
        GROUP BY b.brand, f.listing_price
        ORDER BY listing_price DESC


```
### Select the brand, a count of all products in the finance table, and total revenue
  1) Create four labels for products based on their price range, aliasing as price_category
  2) Join brands to finance on product_id and filter out products missing a value for brand
  3) Group results by brand and price_category, sort by total_revenue

```sql


    SELECT b.brand, COUNT(f.*), SUM(revenue) AS total_revenue,
      CASE
        WHEN f.listing_price < 42 THEN 'Budget'
        WHEN f.listing_price >= 42 AND f.listing_price < 74 THEN 'Average'
        WHEN f.listing_price >= 74 AND f.listing_price < 129 THEN 'Expensive'
        ELSE 'Elite'
        END AS price_category

    FROM finance as f
        
    INNER JOIN brands AS b
      ON b.product_id = f.product_id
    WHERE b.brand IS NOT NULL
    GROUP  BY b.brand, price_category
    ORDER BY total_revenue DESC

```
### Select brand and average_discount as a percentage
  1) Join brands to finance on product_id
  2) Aggregate by brand
  3)  Filter for products without missing values for brand

 ```sql


    SELECT b.brand, AVG(f.discount * 100) AS average_discount
      FROM finance AS f
      INNER JOIN brands AS b
        ON b.product_id = f.product_id
    WHERE b.brand IS NOT NULL
    GROUP BY b.brand


```
###  Calculate the correlation between reviews and revenue as review_revenue_corr
  1) Join the reviews and finance tables on product_id

```sql
    SELECT CORR(r.reviews, f.revenue) review_revenue_corr
      FROM finance AS f
    INNER JOIN reviews AS r
      ON r.product_id = f.product_id
```



### Calculate description_length
  1) Convert rating to a numeric data type and calculate average_rating
  2) Join info to reviews on product_id and group the results by description_length
  3) Filter for products without missing values for description, and sort results by description_length

```sql
   
    SELECT TRUNC(LENGTH(i.description), -2) AS description_length, ROUND(AVG(r.rating::NUMERIC), 2) AS average_rating
    FROM info AS i
    INNER JOIN reviews AS r
     ON r.product_id = i.product_id
    WHERE i.description IS NOT NULL
    GROUP BY description_length
    ORDER BY description_length;
  
```

### Select brand, month from last_visited, and a count of all products in reviews aliased as num_reviews
  1) Join traffic with reviews and brands on product_id
  2) Group by brand and month, filtering out missing values for brand and month
  3) Order the results by brand and month

```sql
   
   SELECT b.brand, EXTRACT (MONTH FROM t.last_visited) AS month, COUNT(r.*) AS num_reviews
      FROM brands AS b 
    INNER JOIN reviews AS r 
      ON b.product_id = r.product_id
    INNER JOIN traffic AS t 
      ON t.product_id = b.product_id
    WHERE b.brand IS NOT NULL 
      AND EXTRACT(MONTH FROM t.last_visited) IS NOT NULL
    GROUP BY b.brand, month
    ORDER BY b.brand, month
   
```

### Create the footwear CTE, containing description and revenue
  1) Filter footwear for products with a description containing %shoe%, %trainer, or %foot%
  2) Also filter for products that are not missing values for description
  3) Calculate the number of products and median revenue for footwear products

```sql
    WITH footwear AS 
    (
      SELECT i.description, f.revenue
      FROM info AS i 
      INNER JOIN finance AS f 
        ON i.product_id = f.product_id
      WHERE (i.description ILIKE '%shoe%'
           OR i.description ILIKE '%trainer%'
           OR i.description ILIKE '%foot%')
          AND i.description IS NOT NULL
    )
    SELECT COUNT(*) AS num_footwear_products, 
       percentile_disc(0.5) WITHIN GROUP (ORDER BY revenue) AS median_footwear_revenue 
	   --percentile_disc(0.5) WITHIN GROUP  LO USO PARA CALCULAR LA MEDIANA
    FROM footwear;
  
```
### Copy the footwear CTE from the previous task
  
  1) Calculate the number of products in info and median revenue from finance
  2) Inner join info with finance on product_id
  3) Filter the selection for products with a description not in footwear
```sql
   
    WITH  footwear AS
      (SELECT i.description, f.revenue
       FROM info AS i
       INNER JOIN finance AS f
           ON i.product_id = f.product_id
       WHERE i.description ILIKE '%shoe%'
             OR i.description ILIKE '%trainer%'
             OR i.description ILIKE '%foot%'
             AND i.description IS NOT NULL
      )
    SELECT COUNT (i.*) AS num_clothing_products, percentile_disc(0.5) WITHIN GROUP (ORDER BY f.revenue) AS median_clothing_revenue
    FROM info AS i
    INNER JOIN finance AS f
      ON i.product_id = f.product_id
    WHERE i.description NOT IN(SELECT description FROM footwear )
   
```
