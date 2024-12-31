# A-B-Test-Marketing-Campaign
Analysis of an A/B test of a marketing campaign to determine effectiveness of campaign. 

Observations: Spent slightly more on test campaign. This is an issue. Not true test. <br>
Missing/ null data on some lines <br>


A/B Testing DataSet

<b> Data cleaning in Excel: </b> At this point I needed to change the date type on my 2 csv files-- control_group .csv & test_group.csv. I did this by opening the files in excel, using the text to columns command, and I then recombined 3 cells into correct date format for SQL (YYY-MM-DD). After this, I saved and re-uploaded the .csv files as 2 tables in SQL. <br> 

<b> SQL Code: <br> </b>

--create table for control_group csv file <br>
create table control_group ( <br>
	campaign_name varchar (255) <br>
	,campaign_date date <br>
	,spend_usd int <br>
	,impressions int <br>
	,reach int <br>
	,website_clicks int <br>
	,searches int <br>
	,content_views int <br>
	,add_to_cart int <br>
	,num_of_purchases int <br>
); <br>

--make sure table uploaded correctly <br>
select * <br>
from control_group; <br>


--create table for test_group csv <br>
create table test_group ( <br>
	campaign_name varchar (255) <br>
	,campaign_date date <br>
	,spend_usd int <br>
	,impressions int <br>
	,reach int <br>
	,website_clicks int <br>
	,searches int <br>
	,content_views int <br>
	,add_to_cart int <br>
	,num_of_purchases int <br>
); <br>

--make sure table uploaded correctly <br>
select * <br>
from test_group; <br>

/*create a temp table combining data from both test_group table and control_group since they have the same column names. I will use this temp table to run queries */ <br>
create temp table control_test_together as ( <br>
	select <br>
		c.campaign_name <br>
		,c.campaign_date <br>
		,c.spend_usd <br>
		,c.impressions <br>
		,c.reach <br>
		,c.website_clicks <br>
		,c.searches <br>
		,c.content_views <br>
		,c.add_to_cart <br>
		,c.num_of_purchases <br>
	from  <br>
		control_group c <br>
	union all  <br>
		select <br>
			t.campaign_name <br>
			,t.campaign_date <br>
			,t.spend_usd <br>
			,t.impressions <br>
			,t.reach <br>
			,t.website_clicks <br>
			,t.searches <br>
			,t.content_views <br>
			,t.add_to_cart <br>
			,t.num_of_purchases <br>
				from  <br>
					test_group t <br>
) <br>

--view new temp table <br>
select * from control_test_together <br>
 
/*Look at summary of sum and average for each column. What is the total and average ad spend, impressions, reach, clicks, searches, views, add to cart, and purchases. After performing this query, I downloaded a .csv file & visualized in tableau */ <br>
	campaign_name <br>
	,sum(b.spend_usd) as total_spend <br>
	,cast(avg(b.spend_usd) as int) as avg_spend <br>
	,sum(b.impressions) as total_impressions <br>
	,cast(avg(b.impressions) as int) as avg_impressions <br>
	,sum(b.reach) as total_reach <br>
	,cast(avg(b.reach) as int) as avg_reach <br>
	,sum(b.website_clicks) as total_clicks <br>
	,cast(avg(b.website_clicks) as int) as avg_clicks <br>
	,sum(b.searches) as total_searches <br>
	,cast(avg(b.searches) as int) as avg_searches <br>
	,sum(b.content_views) as total_views <br>
	,cast(avg(b.content_views) as int) as avg_views <br>
	,sum(add_to_cart) as total_add_to_cart <br>
	,cast(avg(add_to_cart) as int) as avg_add_to_cart <br>
	,sum(num_of_purchases) as total_num_of_purchases <br>
	,cast(avg(num_of_purchases) as int) as avg_num_of_purchases <br>
from <br>
	control_test_together b <br>
group by  <br>
	campaign_name; <br>

/* Next I want to evaluate funnel performance for both campaigns to see which is more effective at converting impressions to purchases. I will calculate the conversion rate at each stage for both campaigns. I used the coalesce function to deal with a couple of null values in my dataset. Results show 8% CTR for test campaign and 5% for control campaign. */ <br>
SOME STAGES OUT OF ORDER. Causes- multiple add to carts, abandoned carts, browsing without intent to buy, browsing for prices (price sensitivity), delayed purchase intent, technical difficulties with payment,  etc. <br>
select <br>
	b.campaign_name <br>
	,round( <br>
        coalesce( <br>
            cast(sum(coalesce(b.website_clicks, 0)) as DECIMAL(10, 2)) /  <br>
            nullif(cast(sum(coalesce(b.impressions, 0)) as DECIMAL(10, 2)), 0), <br>
            0 <br>
        ), <br>
		2 <br>
		) as click_through_rate <br>
	,round( <br>
        coalesce( <br>
            cast(sum(coalesce(b.searches, 0)) as DECIMAL(10, 2)) /  <br>
            nullif(cast(sum(coalesce(b.website_clicks, 0)) as DECIMAL(10, 2)), 0),  <br>
            0 <br>
        ), <br>
		2 <br>
		) as search_conversion_rate <br>
	,round( <br>
        coalesce( <br>
            cast(sum(coalesce(b.content_views, 0)) as DECIMAL(10, 2)) /  <br>
            nullif(cast(sum(coalesce(b.website_clicks, 0)) as DECIMAL(10, 2)), 0),  <br>
            0 <br>
        ), <br>
		2 <br>
		) as view_content_conversion_rate <br>
	,round( <br>
        coalesce( <br>
            cast(sum(coalesce(b.add_to_cart, 0)) as DECIMAL(10, 2)) /  <br>
            nullif(cast(sum(coalesce(b.content_views, 0)) as DECIMAL(10, 2)), 0),  <br>
            0 <br>
        ), <br>
		2 <br>
		) as add_to_cart_conversion_rate <br>
	,round( <br>
        coalesce( <br>
            cast(sum(coalesce(b.num_of_purchases, 0)) as DECIMAL(10, 2)) /  <br>
            nullif(cast(sum(coalesce(b.add_to_cart, 0)) as DECIMAL(10, 2)), 0),  <br>
            0 <br>
        ), <br>
		2 <br>
		) as purchase_conversion_rate <br>
from <br>
	control_test_together b <br>
group by <br>
campaign_name; <br>

/*Which campaign has the highest Return on Investment (ROI)? Why this matters: ROI is one of the most critical metrics in marketing. It measures how effectively each campaign is turning its spending into actual revenue (or purchases, in this case). Results: Control has slightly higher ROI, which shows it’s a more cost effective campaign*/ <br>
select <br>
	b.campaign_name <br>
	,sum(b.spend_usd) as total_spend <br>
	,sum(b.num_of_purchases) as total_purchases <br>
	,round ( <br>
		coalesce( <br>
        sum(cast(b.num_of_purchases as DECIMAL(10, 2))) /  <br>
        nullif(sum(cast(b.spend_usd as DECIMAL(10, 2))), 0),  <br>
        0 <br>
    )* 100, 2 <br>
	) as roi <br>
from control_test_together b <br>
group by <br>
	campaign_name; <br>


/*How much did each campaign spend, and what was the relationship between spending and purchases? Why this matters: This question helps you understand how much money was invested in each campaign and whether the spending aligns with the number of purchases. Results show we spent about the same for both test & control campaign. */ <br>
select <br>
    b.campaign_name, <br>
    sum(b.spend_usd) as total_spend, <br>
    sum(b.num_of_purchases) as total_purchases, <br>
    cast( <br>
        sum(cast(b.spend_usd as decimal(10, 2))) /  <br>
        nullif(sum(cast(b.num_of_purchases as decimal(10, 2))), 0)  <br>
        as decimal(10, 2) <br>
    ) as cost_per_purchase <br>
from <br>
    control_test_together b <br>
group by <br>
    b.campaign_name; <br>

/*How does the performance (spend, impressions, clicks, purchases) vary over time for each campaign? Why this matters: Understanding the time trends of campaign performance helps you identify patterns (e.g., which days performed better, seasonality effects, or if performance increased over time). I downloaded the resulting table and used it to visualize the relationship between ad spend and customer purchases in Tableau */
Analysis Steps: <br>
•	Plot the performance metrics (spend, impressions, clicks, purchases, etc.) over time for both campaigns. <br>
•	Identify any patterns such as spikes in activity, steady growth, or any significant drops. <br>

select  <br>
	c.campaign_date <br>
	,c.campaign_name <br>
	,sum(c.spend_usd) as total_spend <br>
	,sum(c.impressions) as total_impressions <br>
	,sum(c.website_clicks) as total_clicks <br>
	,sum(c.num_of_purchases) as total_purchases <br>
from <br>
	control_group c <br>
group by campaign_date, campaign_name <br>
order by campaign_date; <br>

select  <br>
	t.campaign_date <br>
	,t.campaign_name <br>
	,sum(t.spend_usd) as total_spend <br>
	,sum(t.impressions) as total_impressions <br>
	,sum(t.website_clicks) as total_clicks  <br>
	,sum(t.num_of_purchases) as total_purchases <br>
from <br>
	test_group t <br>
group by campaign_date, campaign_name <br>
order by campaign_date; <br>
