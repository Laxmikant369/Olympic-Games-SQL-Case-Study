# Olympic-Games-SQL-Case-Study
## Analyse Incredible 120 Years of Olympic Games
Introduction

The modern Olympics represents an evolution of all games from 1986 to 2016. Because the Olympics is an international sports event. It is affected by global issues such as the COVID-19 pandemic, and world wars. Nevertheless, there are positive movements such as women's empowerment and the development of the game itself.

In this project, I used a 120-year-old dataset containing a massive collection of data. I downloaded the data from Kaggle Website. https://www.kaggle.com/datasets/heesoo37/120-years-of-olympic-history-athletes-and-results There are total 2 Csv files "athlete_events.csv" and "noc_regions.csv" and I imported the table into MySQL and created a database in my local machine and saved these 2 tables in the following names.

olympics_history<br>
Olympics_history_noc_regions<br>

I performed intermediate to advanced SQL queries (Subqueries, CTE, Joins) for better analysis. First and foremost, I wanted to study the dataset to get answers to the following questions: How many columns and rows are in the dataset? Ans:- ~271K Rows and 15 Columns

How many total datasets are there? Ans:- 2 datasets <br>
How many rows are in the dataset? Ans:- ~271K Rows <br>
<pre>select count(*) as Total_rows from  olympics_history;</pre>
![Image](https://github.com/user-attachments/assets/04953199-132e-4ddf-b170-6bf9f1fce937)
<br>
How many columns are in the dataset? Ans:- 15 Columns<br>
![Image](https://github.com/user-attachments/assets/b293e378-90f6-4152-b9c2-1798c4e72111)
### Derived answers of some questions by writing sql queries.<br>
## Ques-1 How many olympics games have been held?<br>
Ans:- : For this query I used 2 commands ie.distinct Command to get rid of any duplicate values and then Count of That result to show the number of Total Olympics games.<br>
<pre> SELECT COUNT(*) AS total_columns</pre>
![Image](https://github.com/user-attachments/assets/5b5ffb86-8938-428b-91db-eb743f869849)
<br>
## Ques-2 Mention the total no of nations who participated in each olympics game<br>
Ans:- Use a JOIN to display a table showing Countries who participated olympics game and then Count All countries with respect to each Olympic games By Grouping .<br>
<pre>WITH ALL_COUNTRIES AS 
(
	SELECT		GAMES,NR.REGION AS COUNTRY
	FROM		OLYMPICS_HISTORY AS OH
	JOIN		OLYMPICS_HISTORY_NOC_REGIONS NR ON OH.NOC=NR.NOC
	GROUP BY	GAMES,NR.REGION
)
	SELECT		GAMES,COUNT(1) AS TOTAL_NATIONS_PARTICIPATED
	FROM		ALL_COUNTRIES
	GROUP BY	GAMES
	ORDER BY	GAMES	;</pre>
![Image](https://github.com/user-attachments/assets/15eb8c6c-e41e-4258-bea3-5821d674ab11)
![Image](https://github.com/user-attachments/assets/b6530301-85d9-44d9-9b97-e90761eeb0f0)
![Image](https://github.com/user-attachments/assets/ffef6cda-3c53-44ce-af6f-e684da6ba22c)
<br>
## Ques-3 Which year saw the highest and lowest no of countries participating in olympics?<br>
Ans:- For this query First I need to know the Total Countries Participated for each Olympics Games , here I used CTE to create a Temporary result sets. and then I used Window function "First_value" and Over() function to fetch both Lowest Participation Country name and Highest Participation Country name by using total_nations_participated Column and also fetch their respective Olympic games name and last I merge the Olympic games name with Number of Participation using "Concat" Command.<br>
<pre>with all_countries as 
(
	select 	Games,nr.region
	from 	olympics_history as oh
	join 	Olympics_history_noc_regions nr on oh.NOC=nr.NOC
	group by games,nr.region
),
tot_countries as(
select Games,count(1) as total_nations_participated
from all_countries
group by Games
)	
select distinct
      concat(first_value(games) over(order by total_nations_participated)
      , ' - '
      , first_value(total_nations_participated) over(order by total_nations_participated)) as Lowest_Countries,
      concat(first_value(games) over(order by total_nations_participated desc)
      , ' - '
      , first_value(total_nations_participated) over(order by total_nations_participated desc)) as Highest_Countries
      from tot_countries
      order by 1;</pre>
![Image](https://github.com/user-attachments/assets/8eeb2471-9ca8-4909-957a-863bc7780b99)
<br>
## Ques-4 Which country has participated in all of the olympic games?<br>
Ans:- For this query , I used a CTE to create multiple temporary result sets like 1st find a Total Olymics games held till last date . 2nd find the all Countries who participated olympics game and last use a JOIN to display only those countries name who have played all Olympics games till last date by matching the values of total_olympics_games column of Total Games Table and total_participated_games column of countries_participated.<br>
<pre>with tot_games as 
   (
   select count(distinct games) as total_olympics_games 
   from olympics_history),
all_countries as 
   (
   select Games,nr.region as country
   from olympics_history as oh
   join Olympics_history_noc_regions nr on oh.NOC=nr.NOC
   group by games,nr.region
   ),
countries_participated as
   (select country,count(1) as all_participated_games
   from all_countries
   group by country
   )
   				
select cp.* 
from countries_participated as cp 
join tot_games as tg
on cp.all_participated_games=tg.total_olympics_games;</pre>
![Image](https://github.com/user-attachments/assets/3ae0b7fb-ef5e-4bf2-b39e-6e74b2ad3c35)
<br>
## Ques-5 Identify the sports which were played in all summer olympics.
Ans:- With the help of CTE I find total no of distinct summer olympics games(Ans. Total 29 Games). then after count the total Summer olympics games played for each Sports by Grouping. and last Join both the result sets on matching of Total_summer_games column and no_of_games column to fetch only that Sports who has Played total 29 summer olympics games .<br>
<pre>with t1 as 
	(select count(distinct Games) as Total_summer_games 
	from olympics_history
	where Season = 'Summer'),
t2 as
	(select  Sport,count(distinct Games) as no_of_games
	from olympics_history
	where Season = 'Summer' 
	group by Sport
	)
select * 
	from t2 join t1
	on t2.no_of_games=t1.Total_summer_games;</pre>
 <br>
 Another approach to solve this query is by Subquery
 <br>
<pre> SELECT A.*
	FROM
	(
	select  Sport,count(distinct Games) as Total_summergames
	from olympics_history
	where Season = 'Summer' 
	group by Sport
) A
WHERE A.Total_summergames = (select count(distinct Games) as Total_summer_games 
	from olympics_history
	where Season = 'Summer');</pre>
 ![Image](https://github.com/user-attachments/assets/3d7e2e43-2b4b-4484-a84c-fa53a768db0e)
 
 ## Ques-6 Which Sports were just played only once in the entire olympics?
 <pre>with t2 as
	(select distinct Sport,Games
	from olympics_history
	),
t3 as 
	(select Sport,count(Games) as Games_Played_Once
	from t2
	group by Sport
	having count(Games) = 1)

	 select t3.Sport,t2.Games as Olympics_games ,Games_Played_Once
	 from t3 join t2 
	 on t3.Sport=t2.Sport;</pre>
  ![Image](https://github.com/user-attachments/assets/7355ff81-e7c5-490e-b238-9240ef67af84)

  ## Ques-7 Fetch the total no of sports played in each olympic games.<br>
Ans:- The goal is a table that shows how many Sports have played on each Olympic games. and sort the rows by latest Games played .<br>
<pre>SELECT Games,count(distinct Sport) as no_of_sports
FROM olympics_history
group by Games
order by 1 desc;</pre>
![Image](https://github.com/user-attachments/assets/cae56803-227c-48ac-b89f-c89e3f6f68e7)
   ![Image](https://github.com/user-attachments/assets/1fc63a88-9322-46f1-8d56-4038afe1973e)
   ![Image](https://github.com/user-attachments/assets/a89014b3-e90f-463b-ab44-da95374dbeb7)

## Ques-8 Find the Total Athletes participated in the olympic games?
Ans:- Using Distinct Command and Count function we can display the total athletes Participated.
<pre>select count(distinct Name) Total_Athletes
	from olympics_history;</pre>
 ![Image](https://github.com/user-attachments/assets/33eb1eb8-345e-4121-bc0a-d1493f9ffc6d)

 ## Ques-9 Find the Ratio of male and female athletes participated in all olympic games?<br>
 <pre>with t1 as
        	(select sex, count(1) as cnt
        	from olympics_history
        	group by sex),
        t2 as
        	(select *, row_number() over(order by cnt) as rn
        	 from t1),
        female_cnt as
        	(select cnt from t2	where rn = 1),
        male_count as
        	(select cnt from t2	where rn = 2)

	select format(round(male_count.cnt*1.0/female_cnt.cnt,2),'f2') as ratio
	from female_cnt cross join male_count;</pre>
 ![Image](https://github.com/user-attachments/assets/223654e6-45f2-4f40-8be2-ec533e818df8)

 ## Ques-10 Find the top 5 athletes who have won the most gold medals? (we also consider those athletes who have same number of Gold Medals ) <br>
<pre>WITH T1 AS 
	(SELECT	Name,COUNT(1) AS TOTALMEDALS
	FROM	olympics_history
	WHERE	Medal = 'GOLD'
	GROUP BY Name
	),
	T2 AS 
	(SELECT *,DENSE_RANK() OVER(ORDER BY TOTALMEDALS DESC) AS RNK
	FROM T1 )
	SELECT * FROM T2 WHERE RNK <=5;</pre>
 ![Image](https://github.com/user-attachments/assets/6c4da90b-c2f0-4cb3-ad65-5d5d93e5e8cb)

 ## Ques-11 Fetch the top 5 athletes who have won the most medals (gold/silver/bronze). <br>
 <pre>with medals as
(   SELECT	Name,COUNT(1) AS TOTALMEDALS
	FROM	olympics_history
	WHERE	Medal in ('GOLD','silver','bronze')
	group by Name),
ranking as 
(   select *,DENSE_RANK() over (order by TOTALMEDALS desc) as rnk
    from	medals)
select	Name,TOTALMEDALS
from	ranking
where	rnk<=5;</pre>
![Image](https://github.com/user-attachments/assets/02a75e0a-3b9e-4855-bb77-062767703aec)

## Ques-12 Fetch the top 5 most successful countries in olympics. (Success is defined by Highest no of medals won.)<br>
Ans:- Here in this Query , I Joined both tables using Common columns "NOC" and filter only those records where Medals are NA. Then group the final result set into Region Field ,count the total Medals and Fetch Only Top 5 Countries who has earned Highest Medals.
<pre>select nr.region as country
,count(Medal) as Total_Medals	
from	olympics_history as oh
inner join	Olympics_history_noc_regions as nr
on	oh.NOC=nr.NOC
where	Medal <> 'NA'
group by nr.region
order by Total_Medals desc Limit 5;</pre>
![Image](https://github.com/user-attachments/assets/bfea59b3-b6a8-4d13-965c-43c0e88876a1)

## Ques-13 In which Sport/event, India has won highest medals.<br>
<pre>SELECT	 A.Sport AS SPORT,COUNT(1) AS TOTAL_MEDALS
FROM	olympics_history AS A
INNER JOIN(SELECT NOC,region
FROM	Olympics_history_noc_regions
WHERE	NOC = 'IND') AS K
ON		K.NOC = A.NOC
WHERE	Medal <> 'NA'
GROUP BY A.Sport
ORDER BY 2 DESC;</pre>
![Image](https://github.com/user-attachments/assets/87e9439b-2d1a-4ac4-9e0d-8a36742c6aa9)

## Ques-14 Break down all olympic games where India won medal for Hockey and how many medals in each olympic games.<br>
<pre>SELECT TEAM,SPORT,Games,Event,COUNT(1) AS TOTAL
FROM olympics_history
WHERE Medal <> 'NA'
AND Team = 'INDIA'
AND Sport = 'HOCKEY'
GROUP BY TEAM,SPORT,Games,Event
ORDER BY Games ;</pre>
![Image](https://github.com/user-attachments/assets/54ada0e9-3ec7-447f-a9b2-529eb65a831e)
  
