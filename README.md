# SQL-for-cohort-analysis



# KP1 1: Number of transactions
# KPI 2: Total payment value or Gross merchandise volume (GOV ) in each month
# KPI 3: Average order value (AOV) in each product
# KPI 4: Churn rate
# KPI 5: On-time delivery rate

# KPI 1: Number of transaction

select count(distinct(customer_id)) as order_num, customer_state, to_char (Date_trunc('month', order_purchase_timestamp), 'yyyy-mm' ) as mth  
from delivery_period       
left join customer 
using (customer_id)
where customer_state ='SP' OR
 customer_state ='RJ' OR
 customer_state='MG' OR
 customer_state='RS'
group by customer_state,mth



# KPI 2:  Gross merchandise volume (GOV)

select round(sum(payment_value) )as total_payment_value, customer_state, to_char (Date_trunc('month', order_purchase_timestamp), 'yyyy-mm' ) as mth  
from delivery_period       
left join order_payments_2 
using (order_id) left join customer
using (customer_id) where customer_state ='SP' OR
 customer_state ='RJ' OR customer_state='MG' OR customer_state='RS'
group by customer_state,mth;


# KP1 3:  ‘High-value' Average order value  (AOV)

select *
from (select round((freight_value + price) / count(order_id)) as average_order_value, product_category_name_english, customer_state
from  order_items_2     
inner join delivery_period   using (order_id)
inner join customer using (customer_id)
inner join productsusing (product_id)
inner join product_category_name_translation
using (product_category_name)
where customer_state ='SP' OR customer_state ='RJ' OR
 customer_state='MG' OR
 customer_state='RS'
group by customer_state, product_category_name_english,order_items_2.freight_value,order_items_2.price ) a
where average_order_value>2000
group by customer_state, product_category_name_english, a.average_order_value

# KP1 4: Churn rate

with total_amount_costomers_per_month as 
(select count(distinct(customer_unique_id) )as total_amount, customer_state, to_char (Date_trunc('month', order_purchase_timestamp), 'yyyy-mm' ) as mth  
from delivery_period       left join customer  using (customer_id)
where customer_state ='SP' OR customer_state ='RJ' OR customer_state='MG' OR
 customer_state=‘RS' group by customer_state, mth),
lost_costomers as (select count(distinct(customer_unique_id)) as lost_amount,  customer_state,to_char (Date_trunc('month', order_purchase_timestamp), 'yyyy-mm' ) as mth_2 
from delivery_period_20414858       left join customer  using (customer_id)
where( customer_state ='SP' OR customer_state ='RJ' OR customer_state='MG' OR customer_state='RS')  and order_status ='canceled'		  
group by customer_state, mth_2)
select lost_amount::NUMERIC/total_amount::NUMERIC as churn_rate,  lost_costomers.customer_state, mth from total_amount_costomers_per_month 
RIGHT join lost_costomers on  total_amount_costomers_per_month .customer_state= lost_costomers.customer_state and total_amount_costomers_per_month .mth=lost_costomers.mth_2
group by mth, lost_costomers.customer_state,churn_rate




# KP1 5:  On-time delivery 

with total_order as 
(select count(distinct(order_id) )as total_amount_of_order, customer_state
from delivery_period       left join customer  using (customer_id)
where (customer_state ='SP' OR  customer_state ='RJ' OR customer_state='MG' OR
 customer_state=‘RS')  and order_status=‘delivered' group by customer_state ),
on_time_orders as (select count(distinct(customer_unique_id)) as on_time_amount,  customer_state
from delivery_period_20414858      left join customer using (customer_id)
where( customer_state ='SP' OR customer_state ='RJ' OR customer_state='MG' OR customer_state='RS')  and order_status=‘delivered' and order_delivered_customer_date<order_estimated_delivery_date 	 
group by customer_state)
select on_time_amount::NUMERIC/ total_amount_of_order::NUMERIC *100 as on_time_order_rate,on_time_orders.customer_state
from total_order RIGHT join on_time_orders 
on on_time_orders.customer_state=total_order .customer_state
group by on_time_orders.customer_state,on_time_order_rate




