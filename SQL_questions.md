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

SELECT from_user,
COUNT(_) AS total_emails,
ROW_NUMBER() OVER (ORDER BY COUNT(_) DESC, from_user ASC)
FROM google_gmail_emails
GROUP BY from_user
ORDER BY 2 DESC, 1;
