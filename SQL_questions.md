1. Most Profitable Companies : Find the 3 most profitable companies in the entire world.
   Output the result along with the corresponding company name.
   Sort the result based on profits in descending order.

SELECT company, profits
FROM forbes_global_2010_2014
ORDER BY profits DESC
LIMIT 3;

2. Workers With The Highest Salaries : You have been asked to find the job titles of the highest-paid employees. Your output should include the highest-paid title or multiple titles with the same salary

SELECT t.worker_title
FROM title t
JOIN worker w
ON w.worker_id = t.worker_ref_id
WHERE salary in (
SELECT MAX(salary)
FROM worker
)

3. Users By Average Session Time : Calculate each user's average session time. A session is defined as the time difference between a page_load and page_exit. For simplicity, assume a user has only 1 session per day and if there are multiple of the same events on that day, consider only the latest page_load and earliest page_exit, with an obvious restriction that load time event should happen before exit time event . Output the user_id and their average session time.

NOTE: Don't forget to use an alias name for internal query (in this case labeled 't'). Notice DATE() is used to perform actions based on date but not using date per se.

SELECT user_id, AVG(TIMESTAMPDIFF(SECOND, load_time, exit_time)) AS session_time
FROM (
SELECT
DATE(timestamp),
user_id,
MAX(IF(action = 'page_load', timestamp, NULL)) as load_time,
MIN(IF(action = 'page_exit', timestamp, NULL)) as exit_time
FROM facebook_web_log
GROUP BY 1, 2
) t
GROUP BY user_id
HAVING session_time IS NOT NULL;

4. Activity Rank : Find the email activity rank for each user. Email activity rank is defined by the total number of emails sent. The user with the highest number of emails sent will have a rank of 1, and so on. Output the user, total emails, and their activity rank. Order records by the total emails in descending order. Sort users with the same number of emails in alphabetical order.
   In your rankings, return a unique value (i.e., a unique rank) even if multiple users have the same number of emails. For tie breaker use alphabetical order of the user usernames.

NOTE : Quite tricky. Learn to use row_number()

SELECT from*user,
COUNT(*) AS total*emails,
ROW_NUMBER() OVER (ORDER BY COUNT(*) DESC, from_user ASC)
FROM google_gmail_emails
GROUP BY from_user
ORDER BY 2 DESC, 1;

5. Finding user purchases : Write a query that'll identify returning active users. A returning active user is a user that has made a second purchase within 7 days of any other of their purchases. Output a list of user_ids of these returning active users.

SELECT DISTINCT a1.user_id
FROM amazon_transactions a1
JOIN amazon_transactions a2
ON a1.user_id = a2.user_id
AND a1.id <> a2.id
AND DATEDIFF(a2.created_at, a1.created_at) BETWEEN 0 AND 7;

6. Monthly percentage difference : Given a table of purchases by date, calculate the month-over-month percentage change in revenue. The output should include the year-month date (YYYY-MM) and percentage change, rounded to the 2nd decimal point, and sorted from the beginning of the year to the end of the year.
   The percentage change column will be populated from the 2nd month forward and can be calculated as ((this month's revenue - last month's revenue) / last month's revenue)\*100.

NOTE : LAG is used to access previous value defined by offset value. Similarly to access next value use LEAD. In this case, you are supposed to take sum of values

SELECT DATE_FORMAT(created_at, '%Y-%m') AS ym,
ROUND((SUM(value) - LAG(SUM(value)) OVER ())
/ LAG(SUM(value)) OVER () \* 100, 2) AS revenue_diff_pct
FROM sf_transactions
GROUP BY ym
ORDER BY ym;

7. New Products : You are given a table of product launches by company by year. Write a query to count the net difference between the number of products companies launched in 2020 with the number of products companies launched in the previous year. Output the name of the companies and a net difference of net products released for 2020 compared to the previous year.

NOTE : When you have to count the values that are in characters, use count and case. And try to write an answer when you cannot hard code like 2019.

SELECT company_name,
COUNT(CASE WHEN (year = 2020) THEN 1 END) - COUNT(CASE WHEN (year = 2019) THEN 1 END) AS total_launch
FROM car_launches
GROUP BY company_name;

8. Premium vs Freemium : Find the total number of downloads for paying and non-paying users by date. Include only records where non-paying customers have more downloads than paying customers. The output should be sorted by earliest date first and contain 3 columns date, non-paying downloads, paying downloads.

SELECT date, SUM(IF(paying_customer = 'no', downloads, 0)) AS non_paying,
SUM(IF(paying_customer = 'yes', downloads, 0)) AS paying
FROM ms_download_facts
JOIN ms_user_dimension AS mud USING (user_id)
JOIN ms_acc_dimension AS mad USING (acc_id)
GROUP BY date
HAVING non_paying > paying
ORDER BY date ASC;

9. Top percentile fraud : ABC Corp is a mid-sized insurer in the US and in the recent past their fraudulent claims have increased significantly for their personal auto insurance portfolio. They have developed a ML based predictive model to identify propensity of fraudulent claims. Now, they assign highly experienced claim adjusters for top 5 percentile of claims identified by the model.
   Your objective is to identify the top 5 percentile of claims from each state. Your output should be policy number, state, claim cost, and fraud score.

WITH pctls AS
(SELECT \*, PERCENT_RANK() OVER (PARTITION BY state ORDER BY fraud_score DESC) AS pctl FROM fraud_score)

SELECT policy_num, state, claim_cost, fraud_score
FROM pctls
WHERE pctl <= .05;
