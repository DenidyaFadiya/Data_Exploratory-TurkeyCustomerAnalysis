# SQL Project - Data Cleaning Project Melbourne House Sales

<p align="center">
  <img width="550" height="400" src="https://user-images.githubusercontent.com/129844542/235352045-3fc19abe-f3eb-473d-bac3-5ea48ed23b41.jpeg">
</p>

In this Project I will be exploring the data to see the analysis on Turkey Customer Analysis by seeing the contribution of each Category, Payment Method, Age Group etc.

## Full Query for Data Exploratory Project
**Check the Full Query for Data Exploratory Project [here](https://github.com/DenidyaFadiya/Data_Exploratory-TurkeyCustomerAnalysis/blob/main/Full%20Query%20-%20Data%20Exploratory%20Turkey%20Customer%20Analysis).**

**Tableau Interactive Dashboard [here](https://public.tableau.com/app/profile/denidya.fadiya/viz/turkeynewvisualization/Dashboard1).**

In this Data Exploratory project I will be cleaning data that I took from [Kaggle Data Source](https://www.kaggle.com/datasets/mehmettahiraslan/customer-shopping-dataset).

## Exploratory Data Project

This Exploratory data I am going to be using the data I have cleaned in the previous Data Cleaning Project
I have 3 tables which are ; CustomerData, Category and Payment


### Select the data that I am going to use

**CustomerData**


![image](https://user-images.githubusercontent.com/129844542/235130053-f6eeb9a0-cc1d-4c9b-8260-dba5d7e331b6.png)

 

**Category**


![image](https://user-images.githubusercontent.com/129844542/235130080-8f1210ba-fec6-47b3-be16-8ba8c6368ade.png)


**Payment**

![image](https://user-images.githubusercontent.com/129844542/235130367-4be5096b-e997-402b-b977-59b783a1459b.png)

 

*optional = change the category to the data that represents it by using this query,but i will not do it since  
 ```
 select invoice_no, category_id,
case
	when Category_Id = 'a' then 'Clothing'
	when Category_Id = 'b' then 'Shoes'
	when Category_Id = 'c' then 'Books'
	when Category_Id = 'd' then 'Cosmetics'
	when Category_Id = 'e' then 'Food and Beverage'
	when Category_Id = 'f' then 'Toys'
	when Category_Id = 'g' then 'Technology'
	when Category_Id = 'h' then 'Shoes'
	else 'Souvernir'
end as CategoryFixed
from dbo.CustomerData
order by invoice_no
```
 ![image](https://user-images.githubusercontent.com/129844542/235130218-38b4f066-acd3-4e5f-9c21-056857cfd4f9.png)


![image](https://user-images.githubusercontent.com/129844542/235130425-afd8c5e5-1fd4-4c87-b9f9-e1129630c821.png)

Updating the Age Group
```
select age,
case
	when age < 26 then 'Young Adult'
	when age < 50 then 'Middle Age Adult'
	when age < 61 then 'Senior Adult'
	else 'Senior'
end as AgeGroup
from dbo.CustomerData
```
  ![image](https://user-images.githubusercontent.com/129844542/235130452-6c6ad518-6951-430b-808c-a6a716f76a18.png)
  
  
![image](https://user-images.githubusercontent.com/129844542/235130476-ef6b69ba-59bb-4463-aabb-1889326028d9.png)

Updating the query
```
ALTER TABLE dbo.CustomerData
Add Age_Group Nvarchar(255);


Update dbo.CustomerData
SET Age_Group = 
case
	when age < 26 then 'Young Adult'
	when age < 50 then 'Middle Age Adult'
	when age < 61 then 'Senior Adult'
	else 'Senior'
end
from dbo.CustomerData

select *
from dbo.CustomerData
```
![image](https://user-images.githubusercontent.com/129844542/235130527-02524ff7-ed39-4242-853f-9f258e45bc8c.png)

### Joining all of the table
I am gonna be using this table for the analysis later on
```
select invoice_no,
		Inv_date,
		gender, 
		age, 
		price, 
		category,
		quantity,
		payment_method, 
		Shopping_mall
from dbo.customerdata cus
join dbo.Category cat
	on Cat.category_id = Cus.category_id
join dbo.payment pay
	on pay.payment_id = cus.payment_id
```
![image](https://user-images.githubusercontent.com/129844542/235130549-9b9a5ce0-1749-41ad-b480-859a4e26ec80.png)


![image](https://user-images.githubusercontent.com/129844542/235130573-a96cbb42-8fac-4c96-8faa-8ff66b44f6ba.png)


### Total amount spent
This query is used to see the amount of money spent in Turkey
```
select SUM(price) as TotalSpent
from dbo.customerdata
```
 ![image](https://user-images.githubusercontent.com/129844542/235130604-7d5e8fc4-569c-4bad-80fb-600b0be3e6fb.png)
 
 
![image](https://user-images.githubusercontent.com/129844542/235130627-f225d85b-57cb-41ae-8a92-308d8289f72c.png)


### Calculating total order and amount spent by gender and calculate it by percent
Female Order and Spent more money than Male
```
select gender,
		COUNT(gender) as OrderByGender,
		SUM(price) as TotalSpentByGender,
		(SUM(price) * 100) / SUM(SUM(price)) over () as PercentSpentByGender
from dbo.customerdata
group by gender
order by TotalSpentByGender desc
```
![image](https://user-images.githubusercontent.com/129844542/235131735-bbdc69e0-72f2-4e7c-baf3-3c4afc826e5a.png)


![image](https://user-images.githubusercontent.com/129844542/235131740-079f761f-a031-40ab-8a6b-7736ac37e806.png)


### Calculating average amount spent by gender
Even though Female Spent the most money, but the average spending is led by Male
```
select gender,
		AVG(price) as AverageSpentByGender
from dbo.customerdata
group by gender
```
![image](https://user-images.githubusercontent.com/129844542/235131782-0a2209c1-fc85-4495-9ade-67b0f3cbf16c.png)


![image](https://user-images.githubusercontent.com/129844542/235131791-b1f08950-8d82-4b93-a99d-574d13507fa5.png)


### Total quantity  per category
Clothing is the most category bought with quantity of 68.163
Why im using the CTE so the calculation will be based on the category?
In this query the quantity will be separated in 5 criteria (1,2,3,4,5), meaning this query does not sum the specific criteria but calculating based on the number of quantity on Customer Data (sum all 1, sum all 2, sum all 3 etc).  CTE helps to compact the result based on the category
```
with CTE_QuantityPerCategory
as 
(
select category,
		sum(quantity) as TotalQuantity
from dbo.customerdata cus
join dbo.category cat
	on cat.category_id = cus.category_id
group by category 
)
select *
from CTE_QuantityPerCategory
order by TotalQuantity desc
```
![image](https://user-images.githubusercontent.com/129844542/235132024-4c74954c-9a54-474c-9c9c-2850496c40b6.png)


![image](https://user-images.githubusercontent.com/129844542/235132033-b9657861-7c40-4979-a9bd-b845218deb77.png)

 
### Counting the total transaction per each quantity
Clothing is the most bought category
```
with CTE_QuantityPerCategory
as 
(
select category,
		count(quantity) as TotalQuantity
from dbo.customerdata cus
join dbo.category cat
	on cat.category_id = cus.category_id
group by category 
)
select *
from CTE_QuantityPerCategory
order by TotalQuantity desc
```
![image](https://user-images.githubusercontent.com/129844542/235132061-14260e59-a9e3-4b35-87bb-9a0a72ea6154.png)


![image](https://user-images.githubusercontent.com/129844542/235132076-09d0cfb8-f413-4ebb-a7ba-00e03750112a.png)


### Which gender bought which category and how much is it?
We can see that first category that Female and Male bought the most is clothing even though Female bought more clothes than the Male do.
```
select gender, 
		cat.category,
		COUNT(category) as CategoryBought
from dbo.customerdata cus
join dbo.category cat
	on cat.category_id = Cus.category_id
group by gender, category
order by gender, CategoryBought desc
```
![image](https://user-images.githubusercontent.com/129844542/235132697-de1cf264-efa5-4e80-b8e2-39f1094a8e8d.png)


![image](https://user-images.githubusercontent.com/129844542/235132710-fe563acb-7f97-47ac-9f62-7d41c15e21f0.png)


### Most expensive transactions top 10
Technology is leading with the highest price and leading with $5.250
```
select 
	top 10 Max(price) as MaxPrice,
	Category
from dbo.customerdata cus
join dbo.Category cat
	on Cat.category_id = Cus.category_id
group by Price, category
order by MaxPrice desc
```
![image](https://user-images.githubusercontent.com/129844542/235132727-6e33635e-e802-4646-9cc6-1fd400b3a374.png)


![image](https://user-images.githubusercontent.com/129844542/235132748-51ef3333-e42c-46ac-9bd0-c032d7d0ac9a.png)


### Cheapest transactions top 10
From the query below, we can see that shoes is the cheapest transaction with $5.23 
```
select 
	top 10 Min(price) as MinPrice,
	Category
from dbo.customerdata cus
join dbo.Category cat
	on Cat.category_id = Cus.category_id
group by Price, category
order by MinPrice
```
![image](https://user-images.githubusercontent.com/129844542/235132761-5ddd496b-1dd9-4bbe-92c8-c3dc330a5534.png)


![image](https://user-images.githubusercontent.com/129844542/235132782-0ffb149a-bb8b-4868-8cd1-532a307517f6.png)
   

### Spreading of category contribution in percentage
This query below shows Clothing give the most contribution with 22.695 transaction giving 34% to the sales
```
select category,
		COUNT(Category) as OrderByCategory,
		COUNT(*) * 100.0 / SUM(COUNT(*)) over() as PercentPerCategory
from dbo.customerdata cus
join dbo.Category cat
	on Cat.category_id = Cus.category_id
group by category
```
![image](https://user-images.githubusercontent.com/129844542/235132809-fd9f8323-9979-49e4-bf23-b7742b67834d.png)


![image](https://user-images.githubusercontent.com/129844542/235132824-1211b15c-c9f0-4492-a88a-6bf10a175035.png)

 

### Count of transaction based on age group
Middle Age Adult is the age group who has the most transaction leading with 30.460 transaction and make 46% of the transactions
```
select age_group, 
		COUNT(age_group) as CountofAgeGroup,
		COUNT(*) * 100.0 / SUM(COUNT(*)) over() as PercentPerCategory
from dbo.customerdata cus
join dbo.Payment pay
	on pay.payment_id = cus.payment_id
group by age_group
order by CountofAgeGroup desc
```
![image](https://user-images.githubusercontent.com/129844542/235132851-a19ee716-48a1-4673-b21d-1adfed14a17f.png)


![image](https://user-images.githubusercontent.com/129844542/235132860-02c59ec0-2324-4835-b1c0-c5c4d1956cc7.png)


### Seeing the category bought based on age group
Each age group is showing the same patern on which category to be bought.
```
select age_group, 
		cat.category,
		COUNT(category) as CategoryBought
from dbo.customerdata cus
join dbo.category cat
	on cat.category_id = Cus.category_id
group by age_group, category
order by age_group, CategoryBought desc
```
![image](https://user-images.githubusercontent.com/129844542/235132877-d873ddc5-a4e4-45fb-ae2f-6b6868fb54d8.png)


![image](https://user-images.githubusercontent.com/129844542/235132898-6da50cf4-673b-471d-b33e-ed835f31e183.png)
   


### Average Money Spent by Age Group 
Senior Adult Spent the most money in average by $692
```
select  age_group,
		AVG(price) as AvgSpentbyAgeGroup
from dbo.customerdata 
group by age_group
order by AvgSpentbyAgeGroup desc
```
![image](https://user-images.githubusercontent.com/129844542/235132914-76098eab-7fbb-4c84-b835-4dd6e225edd6.png)


![image](https://user-images.githubusercontent.com/129844542/235132931-356037b9-e873-43e1-beac-686f6f7bbff8.png)
 
 

### Sum of transactions amount paid by each payment method method
Most of the transaction is still paid by cash leading with 44.5%
```
select  payment_method,
		SUM(price) as PaidByMethod,
		(SUM(price) * 100) / SUM(SUM(price)) over () as PercentPaidByMethod
from dbo.customerdata cus
join dbo.payment pay
	on pay.payment_id = cus.payment_id
group by payment_method
order by PercentPaidByMethod
```
![image](https://user-images.githubusercontent.com/129844542/235132961-e4cc8609-6301-400c-b09e-01cc85584126.png)


![image](https://user-images.githubusercontent.com/129844542/235133002-a1f3c1e6-7b77-4bf7-8d79-b3cb394271de.png)
