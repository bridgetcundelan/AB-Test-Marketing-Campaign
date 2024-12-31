# A-B-Test-Marketing-Campaign
Analysis of an A/B test of a marketing campaign to determine effectiveness of campaign. 



A/B Testing DataSet

<b> SQL Code: <br> </b>

--create table for control_group csv file
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


Changed date on control_group .csv
-opened in excel
-used text to columns
-recombined 3 cells into correct date format for SQL (YYY-MM-DD)
-Resaved & uploaded into table.

--make sure table uploaded correctly
select *
from control_group;

Changed date on test_group .csv
-opened in excel
-used text to columns
-recombined 3 cells into correct date format for SQL (YYY-MM-DD)
-Resaved & uploaded into table.

--create table for test_group csv
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

--make sure table uploaded correctly
select *
from test_group;

/*create a temp table combining data from both test_group table and control_group since they have the same column names. I will use this temp table to run queries */
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
)

--view new temp table
select * from control_test_together

/*Answer question: What is the total spend, total impressions, total reach, and website views for each campaign? */
select
	campaign_name
	,sum(b.spend_usd) as total_spend
	,sum(b.impressions) as total_impressions
	,sum(b.reach) as total_reach
	,sum(b.content_views) as total_views
from
	control_test_together b
group by 
	campaign_name;

RESULTS
 
Spent slightly more on test campaign. This is an issue. Not true test.

1)	/*Answer question: What is the total spend, total impressions, total reach, and website views for each campaign? */ -- I downloaded excel
select
	campaign_name
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
	,sum(add_to_cart) as total_add_to_cart
	,cast(avg(add_to_cart) as int) as avg_add_to_cart
	,sum(num_of_purchases) as total_num_of_purchases
	,cast(avg(num_of_purchases) as int) as avg_num_of_purchases
from
	control_test_together b
group by 
	campaign_name;

2. Effectiveness Comparison Between Campaigns
Question: How do the Control and Test campaigns compare in terms of conversion at each step (e.g., Clicks, Views, Add to Cart, Purchases)?
•	This can help you evaluate the funnel performance for both campaigns and see which one is more effective at converting impressions to purchases.
Analysis Steps:
•	Calculate conversion rates at each stage for both campaigns:
o	Click-through rate (CTR) = Website Clicks / Impressions
•	/*What is the Click-Through Rate (CTR) for each campaign? Why this matters: The CTR measures how well the campaign is engaging its audience. It tells you the proportion of users who clicked on the ad after seeing it (reaching the website) */
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
from
	control_test_together b
group by 
campaign_name;
RESULTS: 
 
8% and 5% click through rate. Higher for test. 
o	Search conversion rate = Searches / Website Clicks
select
	b.campaign_name
	,round(
        coalesce(
            cast(sum(coalesce(b.searches, 0)) as DECIMAL(10, 2)) / 
            nullif(cast(sum(coalesce(b.website_clicks, 0)) as DECIMAL(10, 2)), 0), 
            0
        ), 
		2 
		) as search_conversion_rate
from
	control_test_together b
group by 
	campaign_name;
RESULTS:  
o	View Content conversion rate = View Content / Website Clicks
select
	b.campaign_name
	,round(
        coalesce(
            cast(sum(coalesce(b.content_views, 0)) as DECIMAL(10, 2)) / 
            nullif(cast(sum(coalesce(b.website_clicks, 0)) as DECIMAL(10, 2)), 0), 
            0
        ), 
		2 
		) as view_content_conversion_rate
from
	control_test_together b
group by 
	campaign_name;
RESULTS:  

o	Add to Cart conversion rate = Add to Cart / View Content
select
	b.campaign_name
	,round(
        coalesce(
            cast(sum(coalesce(b.add_to_cart, 0)) as DECIMAL(10, 2)) / 
            nullif(cast(sum(coalesce(b.content_views, 0)) as DECIMAL(10, 2)), 0), 
            0
        ), 
		2 
		) as add_to_cart_conversion_rate
from
	control_test_together b
group by 
	campaign_name;

RESULTS:  
o	Purchase conversion rate = Purchases / Add to Cart
select
	b.campaign_name
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
	campaign_name;

RESULTS:  
•	Compare the rates of the Control Campaign versus the Test Campaign at each step to see which campaign performs better at driving conversions.
HERE IS THE CODE ALL TOGETHER---- USE THIS AND USE DOWNLOADED CSV FILE TO PLOT!!

---SOME STAGES OUT OF ORDER. Causes- multiple add to carts, abandoned carts, browsing without intent to buy, browsing for prices (price sensitivity), delayed purchase intent, technical difficulties with payment,  etc.

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
campaign_name;


3 ROI: /*Which campaign has the highest Return on Investment (ROI)? Why this matters: ROI is one of the most critical metrics in marketing. It measures how effectively each campaign is turning its spending into actual revenue (or purchases, in this case)*/
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
	campaign_name;
RESULTS:
 
Control has slightly higher ROI, which shows it’s a more cost effective campaign

4. 8. Campaign Spend Optimization
Question: How efficiently is each campaign spending its budget relative to its conversions?

This question could reveal if either campaign is overspending relative to its performance, and thus, which one might benefit from reallocation of resources.

/*How much did each campaign spend, and what was the relationship between spending and purchases? Why this matters: This question helps you understand how much money was invested in each campaign and whether the spending aligns with the number of purchases */
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


RESULTS:
 
Spent about the same for test & control campaign.




5) /*How does the performance (spend, impressions, clicks, purchases) vary over time for each campaign? Why this matters: Understanding the time trends of campaign performance helps you identify patterns (e.g., which days performed better, seasonality effects, or if performance increased over time) */ I downloaded 3 csv files to put in visual
Analysis Steps:
•	Plot the performance metrics (spend, impressions, clicks, purchases, etc.) over time for both campaigns.
•	Identify any patterns such as spikes in activity, steady growth, or any significant drops.

select 
	c.campaign_date
	,c.campaign_name
	,sum(c.spend_usd) as total_spend
	,sum(c.impressions) as total_impressions
	,sum(c.website_clicks) as total_clicks
	,sum(c.num_of_purchases) as total_purchases
from
	control_group c
group by campaign_date, campaign_name
order by campaign_date;

select 
	t.campaign_date
	,t.campaign_name
	,sum(t.spend_usd) as total_spend
	,sum(t.impressions) as total_impressions
	,sum(t.website_clicks) as total_clicks
	,sum(t.num_of_purchases) as total_purchases
from
	test_group t
group by campaign_date, campaign_name
order by campaign_date;

Given the dataset on the A/B test for a marketing campaign (which includes information about spending, impressions, reach, clicks, searches, views, add-to-carts, and purchases for both Control and Test Campaigns), here are 5 key questions you should try to answer using SQL. These questions will help you extract valuable insights about campaign performance, user engagement, and marketing effectiveness.
 
1. Which campaign has the highest Return on Investment (ROI)?
Why this matters: ROI is one of the most critical metrics in marketing. It measures how effectively each campaign is turning its spending into actual revenue (or purchases, in this case).
SQL Query:
sql
Copy code
SELECT 
    "Campaign Name",
    SUM("Spend") AS total_spend,
    SUM("of Purchase") AS total_purchases,
    (SUM("of Purchase") / NULLIF(SUM("Spend"), 0)) AS roi
FROM campaigns
WHERE "Campaign Name" IN ('Control Campaign', 'Test Campaign')
GROUP BY "Campaign Name";
What to look for:
•	Compare the ROI of the Control vs. Test Campaign. A higher ROI indicates a more cost-effective campaign.
 
2. What is the conversion rate (Impressions to Purchases) for each campaign?
Why this matters: The conversion rate from impressions to purchases tells you how well the campaign is able to convert people who saw the ad into actual customers.
SQL Query:
sql
Copy code
SELECT 
    "Campaign Name",
    SUM("of Purchase") / NULLIF(SUM("of Impressions"), 0) AS conversion_rate
FROM campaigns
WHERE "Campaign Name" IN ('Control Campaign', 'Test Campaign')
GROUP BY "Campaign Name";
What to look for:
•	A higher conversion rate suggests that the campaign is more effective at driving purchases. Compare both campaigns.
 
3. How much did each campaign spend, and what was the relationship between spending and purchases?
Why this matters: This question helps you understand how much money was invested in each campaign and whether the spending aligns with the number of purchases.
SQL Query:
sql
Copy code
SELECT 
    "Campaign Name",
    SUM("Spend") AS total_spend,
    SUM("of Purchase") AS total_purchases,
    SUM("Spend") / NULLIF(SUM("of Purchase"), 0) AS cost_per_purchase
FROM campaigns
WHERE "Campaign Name" IN ('Control Campaign', 'Test Campaign')
GROUP BY "Campaign Name";
What to look for:
•	Cost per purchase will help you see which campaign was more efficient in terms of spending relative to actual purchases. A lower cost per purchase indicates a more efficient campaign.
 
4. What is the Click-Through Rate (CTR) for each campaign?
Why this matters: The CTR measures how well the campaign is engaging its audience. It tells you the proportion of users who clicked on the ad after seeing it (reaching the website).
SQL Query:
sql
Copy code
SELECT 
    "Campaign Name",
    SUM("of Website Clicks") / NULLIF(SUM("Reach"), 0) AS click_through_rate
FROM campaigns
WHERE "Campaign Name" IN ('Control Campaign', 'Test Campaign')
GROUP BY "Campaign Name";
What to look for:
•	A higher CTR indicates that the campaign's messaging or targeting was compelling enough for users to click. Compare the two campaigns to determine which one was more engaging.
 
5. How does the performance (spend, impressions, clicks, purchases) vary over time for each campaign?
Why this matters: Understanding the time trends of campaign performance helps you identify patterns (e.g., which days performed better, seasonality effects, or if performance increased over time).
SQL Query:
sql
Copy code
SELECT 
    "Date",
    "Campaign Name",
    SUM("Spend") AS total_spend,
    SUM("of Impressions") AS total_impressions,
    SUM("of Website Clicks") AS total_clicks,
    SUM("of Purchase") AS total_purchases
FROM campaigns
GROUP BY "Date", "Campaign Name"
ORDER BY "Date";
What to look for:
•	By plotting the results over time, you can observe daily trends. For instance, you might notice that the Test Campaign performs better in certain periods, or that spending and purchases rise in sync.
 
Bonus Question: How did different stages of the funnel (Impressions → Clicks → Add to Cart → Purchases) perform across campaigns?
Why this matters: Analyzing the full funnel of the campaign can show you how users progressed through different stages of interaction. Are users viewing the content but not clicking? Are they clicking but not purchasing?
SQL Query:
sql
Copy code
SELECT 
    "Campaign Name",
    SUM("of Impressions") AS total_impressions,
    SUM("of Website Clicks") AS total_clicks,
    SUM("of Add to Cart") AS total_add_to_cart,
    SUM("of Purchase") AS total_purchases
FROM campaigns
WHERE "Campaign Name" IN ('Control Campaign', 'Test Campaign')
GROUP BY "Campaign Name";
What to look for:
•	You can analyze where users drop off in the funnel. For example, a low Add to Cart or Click number in relation to Impressions could suggest issues with the ad’s messaging or landing page.
 
Summary: Top 5 Key Questions to Explore
1.	Which campaign has the highest ROI?
2.	What is the conversion rate (Impressions → Purchases) for each campaign?
3.	How much did each campaign spend, and what was the relationship between spending and purchases?
4.	What is the Click-Through Rate (CTR) for each campaign?
5.	How does campaign performance vary over time (spend, impressions, clicks, purchases)?
By answering these questions, you'll gain a comprehensive understanding of the campaign's performance and be able to provide actionable insights.
![image](https://github.com/user-attachments/assets/904cd234-d768-4fb5-94d4-c73ca579ef22)
