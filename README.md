# Marketing-Analyst


✨ ✨ Solution for Marketing Analyst task for Turing College✨ ✨ 

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
WITH last_e AS 
(
  SELECT user_pseudo_id,event_timestamp,campaign,
    LAG(re.event_timestamp,1) OVER (PARTITION BY re.user_pseudo_id ORDER BY re.event_timestamp) AS last_event
  FROM tc-da-1.turing_data_analytics.raw_events AS re
),

new_session AS 
(
  SELECT *,
    CASE WHEN 
      TIMESTAMP_DIFF(TIMESTAMP_MICROS(event_timestamp), TIMESTAMP_MICROS(last_event), SECOND) >= 60*30
      OR le.last_event IS NULL THEN 1 ELSE 0 END AS is_new_session
  FROM last_e AS le
),

main_timetable AS 
(
  SELECT ns.user_pseudo_id,
    TIMESTAMP_MICROS(ns.event_timestamp) AS event_timestamp,
    TIMESTAMP_MICROS(ns.last_event) AS last_event,
    SUM(ns.is_new_session) OVER (ORDER BY ns.user_pseudo_id, ns.event_timestamp) AS global_session_id,
    ns.Campaign,
  FROM new_session AS ns
),

campaign_sessions AS 
(
  SELECT main_timetable.user_pseudo_id,
    Campaign,
    global_session_id,
    FORMAT_DATE('%A', event_timestamp) AS day_of_week,
    MIN(event_timestamp) OVER (PARTITION BY global_session_id) AS start_time,
    MAX(event_timestamp) OVER (PARTITION BY global_session_id) AS end_time,
    MAX(Campaign) OVER (PARTITION BY global_session_id) AS campaign_name
  FROM main_timetable
),

campaign_sessions_duration AS 
(
  SELECT DISTINCT user_pseudo_id, 
    global_session_id,day_of_week, 
    campaign_name,
    TIMESTAMP_DIFF(end_time, start_time, SECOND)/60 AS session_duration
  FROM campaign_sessions
)

SELECT
  day_of_week, 
  campaign_name,
  AVG(csd.session_duration) AS avg_length
FROM campaign_sessions_duration AS csd
WHERE csd.session_duration IS NOT NULL 
AND csd.campaign_name IN ('BlackFriday_V1','BlackFriday_V2','Data Share Promo','Holiday_V1','Holiday_V2','NewYear_V1','NewYear_V2')
GROUP BY 1,2
ORDER BY 1,2
```
