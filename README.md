# A-B-Test-Marketing-Campaign

<b> Background:</b> The marketing department of a ficticious business is performing an A/B test of a marketing campaign to determine effectiveness of its campaign. They are using a control and test version of the campaign. <br>

<b> Business Task: </b> Analyze performance of both campaigns to uncover trends & insights and determine if company should launch new campaign or if they need to make modifications before launch. <br>

<b> Tools: </b> I will be using Excel for data cleaning, SQL for analysis, and Tableau for visualization. <br>

<b> The Dataset: </b> <br> 
Public dataset found on Kaggle titled: [A/B Testing DataSet](https://www.kaggle.com/datasets/amirmotefaker/ab-testing-dataset/data)<br>

A/B testing helps in finding a better approach to finding customers, marketing products, getting a higher reach, or anything that helps a business convert most of its target customers into actual customers. <br>

Below are all the features in the dataset: <br>

Campaign Name: The name of the campaign <br>
Date: Date of the record <br>
Spend: Amount spent on the campaign in dollars <br>
of Impressions: Number of impressions the ad crossed through the campaign <br>
Reach: The number of unique impressions received in the ad <br>
of Website Clicks: Number of website clicks received through the ads  <br>
of Searches: Number of users who performed searches on the website <br>
of View Content: Number of users who viewed content and products on the website <br>
of Add to Cart: Number of users who added products to the cart <br>
of Purchase: Number of purchases <br> <br>
Two campaigns were performed by the company: <br>
Control Campaign <br>
Test Campaign <br> 

<b> Prepare the Data: </b> <br> 
This Kaggle data set contains metrics for 2 campaigns- a Control and Test Campaign over a one-month period in August, 2019.  
ROCCC Analysis:

Reliability: The data set appears to be from a ficticious business. <br>
Originality: Unclear <br> 
Comprehensiveness: There are 2 .csv files showing performance of control campaign and test campaign. The data set shows comprehensive metrics to review performance of campaign, however there are a few null values which may distort the data. <br>
Current: Data is from August 2019. It was collected over a short time period (1 month). We might have better insights if the company ran both campaigns for a longer time period. <br>
Cited: Data source not cited as it is from a ficticious business <br>

<b> Data cleaning in Excel: </b> I first need to change the date type on my 2 csv files before it can be uploaded to SQL-- control_group .csv & test_group.csv. I did this by opening the files in excel, using the text to columns command, and I then recombined 3 cells into correct date format for SQL (YYY-MM-DD). After this, I saved and re-uploaded the .csv files as 2 tables in SQL. <br> 

<b> Analyzing the data: </b> <br>
<b> SQL Code: <br> </b>

` --create table for control_group csv file `
```
create table control_group (
	campaign_name varchar (255)
	,campaign_date date
	,spend_usd int
	,impressions int
	,reach int
	,website_clicks int
	,searches int
	,content_views int
	,add_to_cart int
	,num_of_purchases int
);
```
` --make sure table uploaded correctly`
```
select * 
from control_group; 
```

`--create table for test_group csv`
```
create table test_group ( 
	campaign_name varchar (255)
	,campaign_date date
	,spend_usd int
	,impressions int
	,reach int
	,website_clicks int
	,searches int
	,content_views int
	,add_to_cart int
	,num_of_purchases int
);
```
` --make sure table uploaded correctly `
```
select * 
from test_group; 
```
After creating the tables, I imported the .csv files into their respective tables. <br>
`/*create a temp table combining data from both test_group table and control_group since they have the same column names. I will use this temp table to run queries */ ` 
```
create temp table control_test_together as ( 
	select 
		c.campaign_name 
		,c.campaign_date 
		,c.spend_usd 
		,c.impressions
		,c.reach
		,c.website_clicks
		,c.searches
		,c.content_views
		,c.add_to_cart
		,c.num_of_purchases
	from 
		control_group c 
	union all  
		select 
			t.campaign_name 
			,t.campaign_date 
			,t.spend_usd 
			,t.impressions
			,t.reach
			,t.website_clicks
			,t.searches
			,t.content_views
			,t.add_to_cart
			,t.num_of_purchases 
				from  
					test_group t 
);
``` 

`--view new temp table `
```
select * from control_test_together; <br>
 ```
`/*Look at summary of sum and average for each column. What is the total and average ad spend, impressions, reach, clicks, searches, views, add to cart, and purchases. After performing this query, I downloaded a .csv file & visualized in tableau */`
```
 	select
 	b.campaign_name 
	,sum(b.spend_usd) as total_spend 
	,cast(avg(b.spend_usd) as int) as avg_spend 
	,sum(b.impressions) as total_impressions 
	,cast(avg(b.impressions) as int) as avg_impressions 
	,sum(b.reach) as total_reach 
	,cast(avg(b.reach) as int) as avg_reach 
	,sum(b.website_clicks) as total_clicks 
	,cast(avg(b.website_clicks) as int) as avg_clicks 
	,sum(b.searches) as total_searches 
	,cast(avg(b.searches) as int) as avg_searches 
	,sum(b.content_views) as total_views 
	,cast(avg(b.content_views) as int) as avg_views 
	,sum(b.add_to_cart) as total_add_to_cart
	,cast(avg(b.add_to_cart) as int) as avg_add_to_cart 
	,sum(b.num_of_purchases) as total_num_of_purchases
	,cast(avg(b.num_of_purchases) as int) as avg_num_of_purchases 
from 
	control_test_together b
group by  
	b.campaign_name; 
```
<img src="https://github.com/user-attachments/assets/cd350011-0803-4b90-9bed-9652884e54b8" width="700">

`/* Next I want to evaluate funnel performance for both campaigns to see which is more effective at converting impressions into purchases. I will calculate the conversion rate at each stage for both campaigns. I used the coalesce function to deal with a couple of null values in my dataset. Results show 8% CTR for test campaign and 5% for control campaign. */`
```
select 
	b.campaign_name 
	,round( 
        coalesce( 
            cast(sum(coalesce(b.website_clicks, 0)) as DECIMAL(10, 2)) /  
            nullif(cast(sum(coalesce(b.impressions, 0)) as DECIMAL(10, 2)), 0), 
            0 
        ), 
		2 
		) as click_through_rate
	,round(
        coalesce( 
            cast(sum(coalesce(b.searches, 0)) as DECIMAL(10, 2)) / 
            nullif(cast(sum(coalesce(b.website_clicks, 0)) as DECIMAL(10, 2)), 0),
            0 
        ), 
		2 
		) as search_conversion_rate 
	,round( 
        coalesce( 
            cast(sum(coalesce(b.content_views, 0)) as DECIMAL(10, 2)) /  
            nullif(cast(sum(coalesce(b.website_clicks, 0)) as DECIMAL(10, 2)), 0), 
            0 
        ), 
		2 
		) as view_content_conversion_rate
	,round( 
        coalesce( 
            cast(sum(coalesce(b.add_to_cart, 0)) as DECIMAL(10, 2)) /  
            nullif(cast(sum(coalesce(b.content_views, 0)) as DECIMAL(10, 2)), 0),  
            0 
        ), 
		2
		) as add_to_cart_conversion_rate 
	,round( 
        coalesce(
            cast(sum(coalesce(b.num_of_purchases, 0)) as DECIMAL(10, 2)) /  
            nullif(cast(sum(coalesce(b.add_to_cart, 0)) as DECIMAL(10, 2)), 0),  
            0 
        ), 
		2 
		) as purchase_conversion_rate 
from 
	control_test_together b 
group by 
b.campaign_name; 
```
`/* Which campaign has the highest Return on Investment (ROI)? Why this matters: ROI is one of the most critical metrics in marketing. It measures how effectively each campaign is turning its spending into actual revenue (or purchases, in this case).  */`
```
select 
	b.campaign_name 
	,sum(b.spend_usd) as total_spend 
	,sum(b.num_of_purchases) as total_purchases 
	,round ( 
		coalesce( 
        sum(cast(b.num_of_purchases as DECIMAL(10, 2))) /  
        nullif(sum(cast(b.spend_usd as DECIMAL(10, 2))), 0),  
        0 
    )* 100, 2 
	) as roi 
from control_test_together b 
group by 
	b.campaign_name; 
```
<img src="https://github.com/user-attachments/assets/43441e63-1cf9-4cea-bf55-17e87ddb7c37" width="700">

`/*How much did each campaign spend, and what was the relationship between spending and purchases? Why this matters: This question helps you understand how much money was invested in each campaign and whether the spending aligns with the number of purchases. */`
```
select 
    b.campaign_name, 
    sum(b.spend_usd) as total_spend,
    sum(b.num_of_purchases) as total_purchases, 
    cast(
        sum(cast(b.spend_usd as decimal(10, 2))) / 
        nullif(sum(cast(b.num_of_purchases as decimal(10, 2))), 0) 
        as decimal(10, 2)
    ) as cost_per_purchase 
from 
    control_test_together b 
group by 
    b.campaign_name;
```
<img src="https://github.com/user-attachments/assets/3423e780-8f5e-43bf-bb4e-d6c596333fed" width="700">

`/*How does the performance (spend, impressions, clicks, purchases) vary over time for each campaign? Why this matters: Understanding the time trends of campaign performance helps you identify patterns (e.g., which days performed better, seasonality effects, or if performance increased over time). I downloaded the resulting table and used it to visualize the relationship between ad spend and customer purchases in Tableau */`

```
select  
	c.campaign_date 
	,c.campaign_name 
	,sum(c.spend_usd) as total_spend 
	,sum(c.impressions) as total_impressions 
	,sum(c.website_clicks) as total_clicks
	,sum(c.num_of_purchases) as total_purchases
from 
	control_group c 
group by c.campaign_date, c.campaign_name 
order by c.campaign_date; 
```
```
select  
	t.campaign_date 
	,t.campaign_name 
	,sum(t.spend_usd) as total_spend 
	,sum(t.impressions) as total_impressions 
	,sum(t.website_clicks) as total_clicks  
	,sum(t.num_of_purchases) as total_purchases 
from 
	test_group t 
group by t.campaign_date, t.campaign_name 
order by t.campaign_date; 
```
<b> Share: </b> Dashboards can be found on my Tableau Public profile. <br>

<b> Act: </b> <br>
Key Trends & Findings:<br>
-The company spent slightly more money on the Test Campaign vs the Control Campaign. It would be better if the company spent the same on both campaigns so we could see a true comparision. <br>
-There is some null data on a couple of lines. It is unclear why there is missing data. <br>
SOME STAGES OUT OF ORDER. Causes- multiple add to carts, abandoned carts, browsing without intent to buy, browsing for prices (price sensitivity), delayed purchase intent, technical difficulties with payment,  etc.
Results: Control has slightly higher ROI, which shows it’s a more cost effective campaign
 Results show we spent about the same for both test & control campaign.
-Control has slightly higher ROI, which shows it’s a more cost effective campaign![image](https://github.com/user-attachments/assets/6759cf01-52e8-48e5-af2c-b0f3cc91c59f)

 Analysis Steps: <br>
•	Plot the performance metrics (spend, impressions, clicks, purchases, etc.) over time for both campaigns. <br>
•	Identify any patterns such as spikes in activity, steady growth, or any significant drops. <br>

Suggested next steps for marketing team: <br>
-The results are so similar between the test & control campaigns, so I would not recommend they go forward with the test campaign. <br>
-Questions for the marketing team- 



