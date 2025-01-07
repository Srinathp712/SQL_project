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

## OVERVIEW
### 1..Which country generated the highest profit for each roast type?
/*(with this query i learned, we can't use ranking column with where or having coditions. so we must use CTE, then we need to perform conditions)*/

''' SQL
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
'''
