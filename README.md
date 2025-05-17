# Finance_And_Spply_Chain_Analytics-
# Finance_and_Supply_Chain_Analytics
To derive business insights from the finance and supply chain data by writing SQL queries and to evaluate and draw conclusions from the data using MYSQL  as the platform

###  Finance and supply chain analytics

<img src="https://srmtek.com/wp-content/uploads/2021/09/Supply-Chain-Analytics.jpg" width="45%" height="40%">   <img src="https://github.com/Karthikc22/Finance_and_Supply_Chain_Analytics/blob/main/Asia%20Pacific%20market%20share%20by%20customers.png" width="45%" height="40%">    <img src="https://github.com/Karthikc22/Finance_and_Supply_Chain_Analytics/blob/main/Latin%20America%20market%20share%20by%20customers_page-0001%20-%20Copy.png" width="45%" height="30%">   <img src="https://th.bing.com/th/id/OIP.KCaUdzrRCysYgtG3EdNDBQHaEK?rs=1&pid=ImgDetMain" width="45%" height="30%">  


checkot the project on linkedin:


### Project Objective

 The performance of Excel has decreased due to the growing size of Excel files, leaving it in a fragile and non-functional state. In order to solve this problem, AtliQ Hardware has employed data specialists who will evaluate and draw 
 conclusions
 from the data using MYSQL as their database.
 To solve queries related to sales, finance, Market, regional analysis, customer behavior and supply chain forecasting

### Project outline

1. Company details
2. Problem Statement
3. Objective of the Project
4. Understanding Database
5. Reports
6. Key Insights

### Company details

1. AtliQ Hardware is a leading electronics manufacturing company that sells hardware's like PCs, Storage 
   devices, Network peripherals, keyboards, mouse, printers etc... across the country
2. Atliq business model has several customers from different platforms like brick and motor (bestbuy, croma, 
   staples) E-commerce (flipkart, amazon)
3. It has customer channels like retailer, direct, indirect channels

### Objective of the project

  To solve queries related to sales, finance, Market, regional analysis, customer behavior and supply chain forecasting

### Understanding Databases

1. We have a database named gdboo41 and in this database we have fact tables and dimension tables.
2. Followed the star schema methodology to connect fact tables and dim tables.

links
links

### SQL Queries

### Views ------------------------------------------------------------------------------------------------
### Net sales view query

    CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `root`@`localhost` 
    SQL SECURITY DEFINER
    VIEW `net_sales` AS
    SELECT 
        `sales_postinv_discount`.`date` AS `date`,
        `sales_postinv_discount`.`fiscal_year` AS `fiscal_year`,
        `sales_postinv_discount`.`customer_code` AS `customer_code`,
        `sales_postinv_discount`.`market` AS `market`,
        `sales_postinv_discount`.`product_code` AS `product_code`,
        `sales_postinv_discount`.`product` AS `product`,
        `sales_postinv_discount`.`variant` AS `variant`,
        `sales_postinv_discount`.`sold_quantity` AS `sold_quantity`,
        `sales_postinv_discount`.`gross_price_total` AS `gross_price_total`,
        `sales_postinv_discount`.`pre_invoice_discount_pct` AS `pre_invoice_discount_pct`,
        `sales_postinv_discount`.`net_invoice_sales` AS `net_invoice_sales`,
        `sales_postinv_discount`.`post_invoice_discount_pct` AS `post_invoice_discount_pct`,
        ((1 - `sales_postinv_discount`.`post_invoice_discount_pct`) * `sales_postinv_discount`.`net_invoice_sales`) AS `net_sales`
    FROM
        `sales_postinv_discount`


### Sales pre invoice deductions view query

    CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `root`@`localhost` 
    SQL SECURITY DEFINER
    VIEW `sales_preinv_discount` AS
    SELECT 
        `s`.`date` AS `date`,
        `s`.`fiscal_year` AS `fiscal_year`,
        `s`.`customer_code` AS `customer_code`,
        `c`.`market` AS `market`,
        `s`.`product_code` AS `product_code`,
        `p`.`product` AS `product`,
        `p`.`variant` AS `variant`,
        `s`.`sold_quantity` AS `sold_quantity`,
        `g`.`gross_price` AS `gross_price_per_item`,
        ROUND((`s`.`sold_quantity` * `g`.`gross_price`),
                2) AS `gross_price_total`,
        `pre`.`pre_invoice_discount_pct` AS `pre_invoice_discount_pct`
    FROM
        ((((`fact_sales_monthly` `s`
        JOIN `dim_customer` `c` ON ((`s`.`customer_code` = `c`.`customer_code`)))
        JOIN `dim_product` `p` ON ((`s`.`product_code` = `p`.`product_code`)))
        JOIN `fact_gross_price` `g` ON (((`g`.`fiscal_year` = `s`.`fiscal_year`)
            AND (`g`.`product_code` = `s`.`product_code`))))
        JOIN `fact_pre_invoice_deductions` `pre` ON (((`pre`.`customer_code` = `s`.`customer_code`)
            AND (`pre`.`fiscal_year` = `s`.`fiscal_year`))))


### Sales post invoice deductions view query

    CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `root`@`localhost` 
    SQL SECURITY DEFINER
    VIEW `sales_postinv_discount` AS
    SELECT 
        `s`.`date` AS `date`,
        `s`.`fiscal_year` AS `fiscal_year`,
        `s`.`customer_code` AS `customer_code`,
        `s`.`market` AS `market`,
        `s`.`product_code` AS `product_code`,
        `s`.`product` AS `product`,
        `s`.`variant` AS `variant`,
        `s`.`sold_quantity` AS `sold_quantity`,
        `s`.`gross_price_total` AS `gross_price_total`,
        `s`.`pre_invoice_discount_pct` AS `pre_invoice_discount_pct`,
        (`s`.`gross_price_total` - (`s`.`pre_invoice_discount_pct` * `s`.`gross_price_total`)) AS `net_invoice_sales`,
        (`po`.`discounts_pct` + `po`.`other_deductions_pct`) AS `post_invoice_discount_pct`
    FROM
        (`sales_preinv_discount` `s`
        JOIN `fact_post_invoice_deductions` `po` ON (((`po`.`customer_code` = `s`.`customer_code`)
            AND (`po`.`product_code` = `s`.`product_code`)
            AND (`po`.`date` = `s`.`date`))))

###  Stored Procedures queries ------------------------------------------------------------------------- 

###  Top N prodcts by net sales stored procedre

    CREATE DEFINER=`root`@`localhost` PROCEDURE `top_n_products_by_net_sales`(
    in_fiscal_year int,
    in_top_n int
    )
    BEGIN
    select 
    product,
    round(sum(net_sales)/1000000,2) as net_sales_mln
    from net_sales
    where fiscal_year = in_fiscal_year
    group by product
    order by net_sales_mln desc
    limit in_top_n;
    END

###  Top N prodcts per division by qty stored procedure

     CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_products_per_division_by_qty_sold`(
      in_fiscal_year int,
      in_top_n int
     )
    BEGIN
    with cte1 as 
    (
     select
     p.division,
     p.product,
     sum(sold_quantity) as total_qty
     from fact_sales_monthly s
     join dim_product p 
      on p.product_code=s.product_code
    where fiscal_year=in_fiscal_year
    group by p.product,p.division),
    cte2 as 
    (
    select
      *,
      dense_rank() over (partition by division order by total_qty desc) as drnk
     from cte1)
    select * from cte2 where drnk<=in_top_n;
    END

###	Top N markets by net sales stored procedure

    CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_markets_by_net_sales`(
    in_fiscal_year int,
    in_top_n int
    )
    BEGIN
    SELECT 
      market,
      round(sum(net_sales)/1000000,2) as net_sales_mln
    FROM 
    gdb0041.net_sales
    where fiscal_year = in_fiscal_year
    group by market
    order by net_sales_mln desc
    limit in_top_n;
    END

### Top N cstomers by net sales stored procedure

    CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_customers_by_net_sales`(
    in_market varchar(45),
    in_fiscal_year int,
    in_top_n int
    )
    BEGIN
    SELECT
			customer, 
			round(sum(net_sales)/1000000,2) as net_sales_mln
	from net_sales s
	join dim_customer c
	on s.customer_code=c.customer_code
	where 
	  s.fiscal_year=in_fiscal_year 
	  and s.market=in_market
	  group by customer
	  order by net_sales_mln desc
	  limit in_top_n;
    END

###	 Monthly gross sales by customers stored procedure

    CREATE DEFINER=`root`@`localhost` PROCEDURE `get_monthly_gross_sales_for_customers`(
     in_customer_codes text
    )
    BEGIN
    select 
      s.date,
      sum(round(g.gross_price*s.sold_quantity,2)) as gross_price_total
    from fact_sales_monthly s
    join fact_gross_price g 
    on
     g.fiscal_year = get_fiscal_year(s.date)
      and g.product_code = s.product_code
     where 
     find_in_set(s.customer_code, in_customer_codes)>0 
    group by s.date;
    END

### Forecast Accuracy stored procedure

    CREATE DEFINER=`root`@`localhost` PROCEDURE `get_forecast_accuracy`(
	  in_fiscal_year INT
	  )
    BEGIN
    with forecast_err_table as 
    (
    select
		s.customer_code as customer_code,
		c.customer as customer_name,
		c.market as market,
		sum(s.sold_quantity) as total_sold_qty,
		sum(s.forecast_quantity) as total_forecast_qty,
		sum(s.forecast_quantity-s.sold_quantity) as net_error,
		round(sum(s.forecast_quantity-s.sold_quantity)*100/sum(s.forecast_quantity),1) as net_error_pct,
		sum(abs(s.forecast_quantity-s.sold_quantity)) as abs_error,
		round(sum(abs(s.forecast_quantity-sold_quantity))*100/sum(s.forecast_quantity),2) as abs_error_pct
    from fact_act_est s
    join dim_customer c
	  on s.customer_code = c.customer_code
    where s.fiscal_year=in_fiscal_year
    group by customer_code
     )
    select 
	       *,
    if (abs_error_pct > 100, 0, 100.0 - abs_error_pct) as forecast_accuracy
    from forecast_err_table
    order by forecast_accuracy desc;


### User derfined fnctions -----------------------------------------------------------------------------

### Fiscal Year

    CREATE DEFINER=`root`@`localhost` FUNCTION `get_fiscal_year`(
     calender_date date
    ) RETURNS int
    DETERMINISTIC
    BEGIN
    declare fiscal_year int;
    set fiscal_year = year(date_add(calender_date, interval 4 month));
    return fiscal_year;
    END

### Fiscal Year by Quatar

    CREATE DEFINER=`root`@`localhost` FUNCTION `get_fiscal_quarter`(
    calender_date date
    ) RETURNS char(2) CHARSET utf8mb4
    DETERMINISTIC
    BEGIN
     declare m tinyint;
     declare qtr char(2);
     set m=month(calender_date);
     case
         when m in (9,10,11) then
              set qtr = "Q1";
		 when m in (12,1,2) then
              set qtr = "Q2";
		 when m in (3,4,5) then
              set qtr = "Q3";
		 else
              set qtr = "Q4";
     end case;
    RETURN qtr;
    END



### Reports 

### APAC Market Share
<img src="https://github.com/Karthikc22/Finance_and_Supply_Chain_Analytics/blob/main/Asia%20Pacific%20market%20share%20by%20customers.png" width="100%" height="80%">

### North America Market Share
<img src="https://github.com/Karthikc22/Finance_and_Supply_Chain_Analytics/blob/main/North%20America%20market%20share%20by%20customers.png" width="100%" height="80%">

### Eropean Market share
<img src="https://github.com/Karthikc22/Finance_and_Supply_Chain_Analytics/blob/main/European%20market%20share%20by%20customers.png" width="100%" height="80%">

### Latin America Market Share
<img src="https://github.com/Karthikc22/Finance_and_Supply_Chain_Analytics/blob/main/Latin%20America%20market%20share%20by%20customers_page-0001%20-%20Copy.png" width="100%" height="80%">

### Global Market Share
<img src="https://github.com/Karthikc22/Finance_and_Supply_Chain_Analytics/blob/main/Global%20Market%20share%20by%20customers.png" width="100%" height="80%">



### Key Insights

1.  In FY-2022 AtliQ Hardware achieved the highest sales
2.  India was the top market with the highest net sales for FY-2021
3.  Amazon contributed 13.23% to the global market share in 2021 FY
4.  Net Sales % of Amazon was the highest in three out of four regions in 2021 FY






    
   
       

     

    




