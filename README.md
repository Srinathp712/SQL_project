# COFFEE SALES ANALYSIS
![Coffee sales](https://github.com/Srinathp712/SQL_project/blob/main/coffee.pic)
## STAKEHOLDER REQUIRMENTS
### 1..Which country generated the highest profit for each roast type?
### 2..What is the total quantity sold and profit generated for each coffee type?
### 3..Which customer placed the highest number of orders, and how much profit did they generate?
### 4..Identify the top 3 most purchased coffee sizes for each country.
### 5..Calculate the average unit price for each product and roast type combination.
### 6..Which coffee type, roast type, and size combination contributes the highest profit, and how does this vary by country?
### 7..Identify the profitability trends over time by calculating the monthly profit growth percentage for each country.
### 8..Find the products that consistently fall within the bottom 10% of profit margins.
### 9..Analyze customer segmentation by determining which customers (grouped by quantity purchased and total profit) should be targeted for loyalty programs.

## QUERIES
### 1..Which country generated the highest profit for each roast type?
/*(with this query i learned, we can't use ranking column with where or having coditions. so we must use CTE, then we need to perform conditions)*/

```SQL
with temp as(
	select c.country, p.roast_type, round(sum(p.profit)) as profits,
	rank() over(partition by(p.roast_type) order by sum(p.profit) desc) as ranking
	from customers as c
	join product_orders as p on c.customer_id = p.customer_id
	group by 1,2
    order by profits desc
)
select country, roast_type, profits from temp
where ranking=1;
```

### 2..What is the total quantity sold and profit generated for each coffee type?

```sql
select coffee_type, count(quantity) as quantity, round(sum(profit),2) as profit from product_orders
group by 1;
```
### 3..Which customer placed the highest number of orders, and how much profit did they generate?
-- (with this query i learned that we must use group by to perform aggregate funtions)

```sql
select c.customer_name, sum(p.quantity) as orders, sum(p.profit) as profits
from customers as c
join product_orders as p on c.customer_id = p.customer_id
group by c.Customer_Name
order by orders desc
limit 5;
```

### 4..Identify the top 3 most purchased coffee sizes for each country.

```sql
with cte as(
	select c.country,p.size,sum(p.quantity) as total_quantity,
    row_number() over(partition by(c.country) order by(sum(p.quantity))desc) as ranking
    from customers as c
    join product_orders as p on c.customer_id = p.customer_id
	group by c.country,p.size
)
select country,size,total_quantity from cte
where ranking <=3;
```

### 5..Calculate the average unit price for each product and roast type combination.

```sql
select coffee_type, roast_type, round(avg(unit_price),2) as avg_price from product_orders
group by 1,2
order by 1;
```


### 6..Which coffee type, roast type, and size combination contributes the highest profit, and how does this vary by country? 
/*( with this query i learned, we will get incorrect aggregation if we use group by 1,2.. so we have to group all columns
to get desired output  for this quetion)*/

```sql
select cte.country,cte.coffee_type,cte.roast_type,cte.ranking
from (
	select c.country, p.coffee_type, p.roast_type, sum(p.profit) as highest_profit,
	rank() over(partition by (c.country) order by sum(p.profit) desc ) as ranking
	from customers as c
	join product_orders as p on c.customer_id = p.customer_id
	group by 1,2,3
) as cte
having cte.ranking <=3;
```

### 7..Identify the profitability trends over time by calculating the monthly profit growth percentage for each country.

```sql
select c.Country, year(p.order_date),sum(p.profit) as total_profit,
lag(sum(p.profit)) over(partition by country order by year(p.order_date) asc) as yearly_profit,
ROUND(
	100.0 * (SUM(p.profit) - LAG(SUM(p.profit)) OVER (PARTITION BY c.country ORDER BY YEAR(p.order_date)))
	/ LAG(SUM(p.profit)) OVER (PARTITION BY c.country ORDER BY YEAR(p.order_date)),2
    ) AS profit_growth_percentage
from customers as c
join product_orders as p on c.customer_id = p.customer_id
group by 1,2
order by 2;
```


### 8..Find the products that consistently fall within the bottom 10% of profit margins.
/* (with this query i learned, we cant use aggregatd column in having clause for usingg another function like max.
so we created another subquery.) */

```sql
select product_id,coffee_type,round(sum(profit),2) as total_profits from product_orders
group by 1,2
having total_profits <= (
	select max(temp.profits) * 0.1
    from (
		select product_id,coffee_type,
        sum(profit) as profits from product_orders
        group by 1,2
		) as temp
	)
order by total_profits desc;
```

### 9..Analyze customer segmentation by determining which country customers (grouped by quantity purchased and total profit) should be targeted for loyalty programs

```sql
select c.country,p.quantity as quantity,round(sum(p.profit),2) as profits,
count(case when c.loyalty_card = 'yes' then 1 else null end) as loyalty
from customers as c 
join product_orders as p on c.customer_id = p.customer_id
group by 1,2
order by 3 desc;
```
## DASHBOARD
![DASHBOARD]()

## CONCLUSION
### The coffee sales analysis revealed that the USA generated the highest profits for Dark Roast, making it the top-performing region for this type, while Canada excelled in Light Roast, indicating potential for growth in that segment. Espresso emerged as the most popular coffee type overall, driving the highest sales volumes across regions. Seasonal trends showed peak sales in Q4, emphasizing the importance of stocking inventory before the holiday season. "By leveraging regional preferences and implementing targeted marketing strategies, the client can enhance profits and improve customer satisfaction".
