# Marketing-Analyst


✨ ✨ Solution for Marketing Analyst task✨ ✨ 

<a href = 'https://docs.google.com/spreadsheets/d/1HFUrlI4NUbBio7uR0rBI3lxkxsFBP-BVWgelbZbob_A/edit?usp=sharing
'> The dashboard with visualisations.</a> It explains the main insights of the analysis and provides a solution of counting user sessions.

:rocket: SQL (BigQuery), Excel, Google Sheets

### The task:

- You have a follow up task from your marketing manager to identify overall trends of all marketing campaigns on your ecommerce site. She is particularly interested in finding out if users tend to spend more time on your website on certain weekdays and how that behavior differs across campaigns.
- Create a presentation centered around the dynamic weekday duration, focusing on differences between marketing campaigns.
- See whether you can apply 1-2 techniques learned in this or other modules throughout the course material to enhance your presentation on this subject.
- Explore the data. See whether there are interesting data points that can give more insights to your presentation.
- Provide analytical insights, what are the drawbacks of this analysis, what further analysis could you recommend?

You should use the turing_college.raw_events table to answer this question. Write a SQL query that would extract data from BigQuery, make a visualization using Google Sheets and briefly comment your findings. As we do not have session identifiers in the dataset, you will have to come up with your own logic for how you will model sessions. Have in mind that a single user can come to your website on multiple days and if you were to calculate time on the website this may have an impact on this metric.

### Main SQL Query:

``` SQL 
WITH t1 AS 
(
  SELECT user_pseudo_id,
  DATE_TRUNC(subscription_start, WEEK) as start_week,
  subscription_end
  FROM `tc-da-1.turing_data_analytics.subscriptions` AS s 
)

SELECT t1.start_week as sub_date,
  COUNT(start_week) as w0,
  SUM(CASE WHEN subscription_end IS NULL OR subscription_end > DATE_ADD(start_week, INTERVAL 1 WEEK) THEN 1 ELSE 0 END) AS w1,
  SUM(CASE WHEN subscription_end IS NULL OR subscription_end > DATE_ADD(start_week, INTERVAL 2 WEEK) THEN 1 ELSE 0 END) AS w2,
  SUM(CASE WHEN subscription_end IS NULL OR subscription_end > DATE_ADD(start_week, INTERVAL 3 WEEK) THEN 1 ELSE 0 END) AS w3,
  SUM(CASE WHEN subscription_end IS NULL OR subscription_end > DATE_ADD(start_week, INTERVAL 4 WEEK) THEN 1 ELSE 0 END) AS w4,
  SUM(CASE WHEN subscription_end IS NULL OR subscription_end > DATE_ADD(start_week, INTERVAL 5 WEEK) THEN 1 ELSE 0 END) AS w5,
  SUM(CASE WHEN subscription_end IS NULL OR subscription_end > DATE_ADD(start_week, INTERVAL 6 WEEK) THEN 1 ELSE 0 END) AS w6
FROM t1
GROUP BY 1
```
