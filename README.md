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
After creating the tables, I imported the .csv files into their respective tables. <br> <br>

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
<img src="https://github.com/user-attachments/assets/c38afb8c-2bc4-468c-a536-71ed80265c8f" width="700">

`/* There were null values on one row on 8/5/2019 on my Control data set on every column except spend. In an ideal scenario, I would investigate why there are null values here. Unfortunately, I am unable to do so in this hypothetical scenario. I assume this may be a lag effect-- i.e. Google Ads were paid for late at night but the clicks and impressions were recorded on the next day. For this reason, I have decided to replace the null values for this date with "0".  Results show 8% CTR for test campaign and 5% for control campaign. */`
```

update control_test_together
set impressions = 0
	,reach = 0
	,website_clicks = 0
	,searches = 0
	,content_views = 0
	,add_to_cart = 0
	,num_of_purchases = 0
where impressions is null
	or reach is null
	or website_clicks is null
	or searches is null
	or content_views is null
	or add_to_cart is null
	or num_of_purchases is null
;

select * from control_test_together;
```

`/* Next I want to evaluate funnel performance for both campaigns to see which is more effective at converting impressions into purchases. I will calculate the conversion rate at each stage for both campaigns.  Results show 8% CTR for test campaign and 5% for control campaign. */`

```
SELECT 
    b.campaign_name,
    ROUND(
        CAST(SUM(b.website_clicks) AS DECIMAL(10, 2)) /  
        NULLIF(CAST(SUM(b.impressions) AS DECIMAL(10, 2)), 0), 
        2
    ) AS click_through_rate,
    
    ROUND(
        CAST(SUM(b.searches) AS DECIMAL(10, 2)) / 
        NULLIF(CAST(SUM(b.website_clicks) AS DECIMAL(10, 2)), 0),
        2
    ) AS search_conversion_rate,
    
    ROUND(
        CAST(SUM(b.content_views) AS DECIMAL(10, 2)) /  
        NULLIF(CAST(SUM(b.website_clicks) AS DECIMAL(10, 2)), 0), 
        2
    ) AS view_content_conversion_rate,
    
    ROUND(
        CAST(SUM(b.add_to_cart) AS DECIMAL(10, 2)) /  
        NULLIF(CAST(SUM(b.content_views) AS DECIMAL(10, 2)), 0),  
        2
    ) AS add_to_cart_conversion_rate,
    
    ROUND(
        CAST(SUM(b.num_of_purchases) AS DECIMAL(10, 2)) /  
        NULLIF(CAST(SUM(b.add_to_cart) AS DECIMAL(10, 2)), 0),  
        2
    ) AS purchase_conversion_rate
FROM 
    control_test_together b
GROUP BY 
    b.campaign_name;
```
`/* Which campaign has the highest Return on Investment (ROI)? Why this matters: ROI is one of the most critical metrics in marketing. It measures how effectively each campaign is turning its spending into actual revenue (or purchases, in this case).  */`
```
SELECT 
    b.campaign_name,
    SUM(b.spend_usd) AS total_spend,
    SUM(b.num_of_purchases) AS total_purchases,
    ROUND(
        SUM(CAST(b.num_of_purchases AS DECIMAL(10, 2))) /  
        NULLIF(SUM(CAST(b.spend_usd AS DECIMAL(10, 2))), 0),  
        2
    ) * 100 AS roi
FROM 
    control_test_together b
GROUP BY 
    b.campaign_name; 
```
<img width="700" alt="Screenshot 2025-03-12 at 11 07 04 PM" src="https://github.com/user-attachments/assets/f987043e-02ef-499d-b6a1-c7ed93e77220" />

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
* The company spent slightly more money on the Test Campaign vs the Control Campaign. It would be better if the company spent the same on both campaigns so we could see a true comparision. <br>
* As a result, the ROI of the Control Campaign was slightly higher than the Test Campaign, making it a more cost effective campaign. I believe this is due to the higher ad spend.
* There is some null data on a couple of lines. It is unclear why there is missing data. Finding this out would help improve accuracy of analysis. <br>
* Funnel Performance: For the Control Campaign, there is a higher "add to cart" conversion rate than a "purchase conversion rate" which is unexpected. Possible causes could be-- customers with multiple add to carts, abandoned carts, or technical difficulties with payment. I would recommend the marketing team have a look at this stage to uncover the reasons customers aren't converting to purchases & try and optimize. <br>
* Funnel Performance: For both campaigns, there is a higher "search conversion rate" than a "view content conversion rate." Possible causes could be- customers browsing for prices and waiting to buy (price sensitivity) or delayed purchase intent. I would recommend the marketing team look at this stage to optimize. Perhaps they could offer customers who reach their webiste a coupon to encourage them further down the funnel. <br>
* Ad spend vs customer purchase: customer purchases tended to correlated with ad spend and it went up and down during the month. <br>

Suggested next steps for marketing team: <br>
* The results are so similar between the test & control campaigns, so I would not recommend they go forward with the test campaign. <br>
* I would recommend they consider offering a coupon for customers who provide their email address when they visit the website to encourage purchases & also review the check out process to optimize and uncover why customers aren't making the jump from "add to cart" to "purchase"
* After reviewing these customer touchpoints, they can once again test out a new "Test" and "Control" campaign with equal ad spend. 
