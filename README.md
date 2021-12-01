# Project 2 

## Partner Information

Manasi Danke (md4022) and Jacqueline Gibson (jcg2216)

## Server Information

psql -U jcg2216 -h 35.196.73.133 -d proj1part2

## Modification Rationale: Composite Type, Array, and Document

For the composite type <code>mdi</code> we found this simplified the <code>Quidditch_Matches</code> entity significantly; rather than having separate attributes for month/day/year, combining them not only made it easier to understand the data, but it connected the date info in a meaningful way. Previously, the month/day/year did not have a direct relationship in each entry. Now, users can still pull the individual date info types, but they also have direct access to the others. Since a significant portion of our site data revolves around Quidditch matches, improving the entity to be more user friendly made the most sense for implementing our first composite type. For this portion of the project, we added additional tuples for <code>Quidditch_Matches</code>, creating additional "inter-school" data; this showed why having a centralized <code>mdi</code> type would be helpful, in this case making it easier to filter match data based on a given date range; you can see how we implemented this in Query 1.

We had several entity candidates for adding an array, but we chose to implement it for the <code>House</code> entity's colors. We did not do much with this information in Project 1. For Project 2, we thought it would be interesting to consider ways users would want to interact with house color information. Many sports fans are interested in more than just the name and track record of the team; there are aesthetic choices to consider as well. For our scenario, we imagined a subset of Quidditch Fans that want to identify which teams match their color constraints, and you can see how we implemented that in Query 2. 

For implementing a document, we wanted to tie it back to the usability of the application and considered the application of searching through text for important keywords. Since our <code>Spells</code> entity had the most robust text information, we implemented the document schema change here. In our scenario, it made sense that incoming students would want to filter information that was geared towards their lack of experience, so we chose to help them identify spells for beginners. The spell descriptions were updated to include keywords like "beginning", “begin”, or “introductory”, so that it would be easier to distinguish these from more advance spells. In Query 3, we built out a scenario where students were not only searching for beginner spells, but trying to identify potential mentors/tutors to help them master the magic. 

## Queries

<em>Query 1:</em>

<code>
SELECT (match_date).match_month, (match_date).match_day, (match_date).match_year, team1, team2, winning_house, school_name as winning_school from (SELECT \*, CASE WHEN team1_score > team2_score then team1 else team2 END as winning_house, CASE WHEN team1_score > team2_score THEN school_id_1 ELSE school_id_2 END as school_id FROM quidditch_matches WHERE (match_date).match_month = 'November' AND (match_date).match_day > 16 AND (match_date).match_day < 23) as WinTeams NATURAL JOIN schools
</code>

  <br></br>
 This query returns the results of this year’s Thanksgiving Quidditch Exhibition Matches; the matches took place from November 17th to November 22nd. In the year preceding the Triwizard Tournament, the top performing institutions in the Worldwide Web of Wizarding Schools are invited to play in this exhibition; any school that wins a game in this exhibition receives an extra set of aids for their participant in next year’s Triwizard Tournament! As you can imagine, this is a coveted prize, as the aids can be anything from extra time for a task to bonus points towards total performance. Triwizard Tournament fans want to know who proved victorious during the 2021 Exhibition Matches, so they’ll know which teams will have a leg up in the 2022 Tournament; those are the best teams to root for, after all!

  
What it returns:
  
![Alt text](https://github.com/JGibson2019/proj1-3/blob/master/query1project2.PNG "Query 1")
***
<em>Query 2:</em>
  
<code>
SELECT house_name, colors[1] as house_color_1, colors[2] as house_color_2 from houses where ARRAY['Green', 'Gold', 'Black'] && colors
</code>

  <br></br>

We have an influx of interested Quidditch fans in Jamaica, but they want to keep their new hobby a secret from Muggles. To fly under the radar, they only want to buy merchandise for houses who share colors with their national flag. As long as the team has at least one color in common with the flag, the fans have told us they are confident their gear will go by undetected. We’ve created this query so Jamaican fans can have a shortlist of teams whose gear will be the least suspicious.
  
What it returns:
  
![Alt text](https://github.com/JGibson2019/proj1-3/blob/master/query2project2.PNG "Query 2")
***
 <em>Query 3:</em>

<code>
SELECT std_name as mentor_option, graduation_year, gpa, spell_name, spell_descrip, is_first_year_spell FROM Students NATURAL JOIN Student_Can_Perform_Spells NATURAL JOIN (SELECT spell_id, spell_name, spell_descrip, to_tsvector(spell_descrip), to_tsvector(spell_descrip) @@ to_tsquery('begin') OR to_tsvector(spell_descrip) @@ to_tsquery('introductory') as is_first_year_spell from spells) as spell_info WHERE is_first_year_spell = true AND gpa > 3.75
</code>
  
<br></br>

Many first year students have not used spells before or may need extra help and guidance with performing introductory spells. The Worldwide Web of Wizarding Schools has selected Hogwarts as the first school to pilot our peer-to-peer mentoring program; this program offers spell support for those enrolled in first year courses during the 2021-2022 school year! Our goal is to pair first years with older students who have mastered these spells and proven this by receiving top marks on qualifying exams; to be eligible to mentor, students must be on track to graduate magna cum laude (aka maintain a 3.75 GPA or above). This query searches for spells taught in introductory and beginner classes and returns the names of students who can serve as mentors based on the qualifications mentioned above. This way, students know that their mentor is a Magic Maven in the making!

What it returns:
  
![Alt text](https://github.com/JGibson2019/proj1-3/blob/master/query_3_rerun.png "Query 3")
