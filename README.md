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

## Schema

Please refer to the schema in the VM as well to see a version where each statement is on a new line.

DROP TABLE IF EXISTS Students CASCADE;
DROP TABLE IF EXISTS Spells CASCADE; 
DROP TABLE IF EXISTS Quidditch_Matches CASCADE; 
DROP TABLE IF EXISTS Student_Can_Perform_Spells CASCADE;
DROP TABLE IF EXISTS Spells_Taught_In_Class CASCADE;
DROP TABLE IF EXISTS Professor_Discusses_Spells CASCADE;
DROP TABLE IF EXISTS Schools_Offer_Classes CASCADE;
DROP TABLE IF EXISTS Creatures_Native_To_School; 
DROP TABLE IF EXISTS Creatures CASCADE; 
DROP TABLE IF EXISTS Creatures_Involved_In_Classes CASCADE;
DROP TABLE IF EXISTS Houses_Play_In_Quidditch_Matches CASCADE;
DROP TABLE IF EXISTS Houses CASCADE;
DROP TABLE IF EXISTS Professors CASCADE;
DROP TABLE IF EXISTS Teaches_Classes CASCADE;
DROP TABLE IF EXISTS Professor_Supervises_House CASCADE;
DROP TABLE IF EXISTS Students_Take_Classes CASCADE;
DROP TABLE IF EXISTS Schools CASCADE; 

CREATE TABLE Schools (school_id INTEGER, school_name CHAR(50) NOT NULL, headmaster CHAR(50), located CHAR(50), PRIMARY KEY (school_id), UNIQUE(school_name));

CREATE TABLE Houses  (house_name CHAR(20), school_id INTEGER, std_count INTEGER NOT NULL CHECK(std_count >=0), founder CHAR(20), colors TEXT[2] NOT NULL, CHECK(colors[0] != colors[1]), mascot CHAR(20), curr_pts INTEGER NOT NULL CHECK(curr_pts >=0), PRIMARY KEY (house_name, school_id),FOREIGN KEY (school_id)REFERENCES Schools ON DELETE CASCADE);

CREATE TYPE mdi as (match_month CHAR(20), match_day INTEGER, match_year INTEGER);

CREATE TABLE Quidditch_Matches (
match_id INTEGER PRIMARY KEY, 
school_id_1 INTEGER NOT NULL,
school_id_2 INTEGER NOT NULL,
match_date mdi NOT NULL CHECK((match_date).match_month ='January' OR (match_date).match_month='February' OR (match_date).match_month='March' OR (match_date).match_month='April' OR (match_date).match_month='May' OR (match_date).match_month='June' OR (match_date).match_month='July' OR (match_date).match_month='August' OR (match_date).match_month='September' OR (match_date).match_month='October' OR (match_date).match_month='November' OR (match_date).match_month='December'), 
CHECK((((match_date).match_month='January' OR (match_date).match_month='March' OR (match_date).match_month='April' OR (match_date).match_month='May' OR (match_date).match_month='June' OR (match_date).match_month='July' OR (match_date).match_month='August' OR (match_date).match_month='September' OR (match_date).match_month='October' OR (match_date).match_month='November' OR (match_date).match_month='December')AND (match_date).match_day > 0 AND (match_date).match_day < 32)OR(((match_date).match_month='February')AND (match_date).match_day > 0 AND (match_date).match_day < 30)),
CHECK((match_date).match_year < 2022 AND (match_date).match_year >= 1),
team1 CHAR(20) NOT NULL, team2 CHAR(20) NOT NULL, CHECK(team1 != team2), team1_score INTEGER NOT NULL, CHECK(team1 != team2),  team2_score INTEGER NOT NULL, CHECK(team2_score >=0), CHECK(team1_score != team2_score),
FOREIGN KEY (team1, school_id_1)REFERENCES Houses(house_name, school_id) ON DELETE CASCADE,
FOREIGN KEY (team2, school_id_2)REFERENCES Houses(house_name, school_id) ON DELETE CASCADE);

CREATE TABLE Creatures (creature_id INTEGER PRIMARY KEY, creature_name CHAR(20) NOT NULL, dangerous BOOLEAN NOT NULL, class_use BOOLEAN NOT NULL, UNIQUE (creature_name));

CREATE TABLE Creatures_Native_To_School (creature_id INTEGER, school_id INTEGER,PRIMARY KEY (creature_id, school_id), FOREIGN KEY (creature_id) REFERENCES Creatures ON DELETE CASCADE,FOREIGN KEY (school_id) REFERENCES Schools ON DELETE CASCADE);

CREATE TABLE Spells (spell_id INTEGER, spell_name CHAR(20) NOT NULL, spell_descrip TEXT NOT NULL, PRIMARY KEY (spell_id), UNIQUE(spell_name));

CREATE TABLE Professors (prof_id INTEGER, prof_name CHAR(20) NOT NULL, house_supervised CHAR(20), is_alum BOOLEAN NOT NULL, school_id INTEGER,PRIMARY KEY (prof_id, school_id),FOREIGN KEY (school_id) REFERENCES Schools ON DELETE CASCADE);

CREATE TABLE Professor_Discusses_Spells (spell_id INTEGER,prof_id INTEGER, school_id INTEGER,PRIMARY KEY (prof_id, spell_id, school_id),FOREIGN KEY (spell_id)REFERENCES Spells(spell_id) ON DELETE CASCADE,FOREIGN KEY (prof_id, school_id) REFERENCES Professors(prof_id, school_id) ON DELETE CASCADE);

CREATE TABLE Teaches_Classes (class_id INTEGER,class_name CHAR(50), clevel CHAR(20), prof_id INTEGER, school_id INTEGER, PRIMARY KEY(class_id, school_id), FOREIGN KEY (prof_id, school_id) REFERENCES Professors(prof_id, school_id) ON DELETE CASCADE, FOREIGN KEY (school_id) REFERENCES Schools(school_id) ON DELETE CASCADE);

CREATE TABLE Schools_Offer_Classes (school_id INTEGER,class_id INTEGER,PRIMARY KEY (school_id, class_id),FOREIGN KEY (class_id, school_id) REFERENCES Teaches_Classes(class_id, school_id) ON DELETE CASCADE,FOREIGN KEY (school_id) REFERENCES Schools(school_id) ON DELETE CASCADE);

CREATE TABLE Creatures_Involved_In_Classes (creature_id INTEGER, class_id INTEGER, school_id INTEGER, PRIMARY KEY (creature_id, class_id, school_id),FOREIGN KEY (creature_id)REFERENCES Creatures(creature_id) ON DELETE CASCADE,FOREIGN KEY (class_id, school_id)REFERENCES Teaches_Classes(class_id, school_id) ON DELETE CASCADE);
 
CREATE TABLE Houses_Play_In_Quidditch_Matches (house_name CHAR(20),school_id INTEGER,match_id INTEGER,PRIMARY KEY(house_name, school_id, match_id), FOREIGN KEY (house_name, school_id) REFERENCES Houses(house_name, school_id) ON DELETE CASCADE,FOREIGN KEY (match_id)REFERENCES Quidditch_Matches(match_id) ON DELETE CASCADE);

CREATE TABLE Spells_Taught_In_Class (spell_id INTEGER, class_id INTEGER,school_id INTEGER, PRIMARY KEY (spell_id, class_id, school_id),FOREIGN KEY (spell_id)REFERENCES Spells ON DELETE CASCADE,FOREIGN KEY (class_id, school_id)REFERENCES Teaches_Classes ON DELETE CASCADE);

CREATE TABLE Professor_Supervises_House (prof_id INTEGER, house_name CHAR(20),school_id_1 INTEGER,school_id_2 INTEGER,PRIMARY KEY(prof_id, school_id_1, house_name, school_id_2),FOREIGN KEY (prof_id, school_id_1) REFERENCES Professors(prof_id, school_id) ON DELETE CASCADE,FOREIGN KEY (house_name, school_id_2)REFERENCES Houses(house_name, school_id), CHECK(school_id_1=school_id_2));

CREATE TABLE Students (std_id INTEGER, std_name CHAR(20) NOT NULL, year_entered INTEGER NOT NULL, graduation_year INTEGER not NULL, gpa REAL CHECK(gpa >=1 AND gpa <=4), blood_status CHAR(20) NOT NULL CHECK(blood_status= 'Muggle' OR blood_status= 'Pure' OR blood_status= 'Half'), patronus CHAR(20), house_name CHAR(20) NOT NULL, school_id INTEGER NOT NULL, PRIMARY KEY (std_id),FOREIGN KEY (school_id, house_name) REFERENCES Houses(school_id, house_name),CHECK(graduation_year > year_entered));

CREATE TABLE Student_Can_Perform_Spells (spell_id INTEGER, std_id INTEGER, PRIMARY KEY (spell_id, std_id), FOREIGN KEY (std_id) REFERENCES Students ON DELETE CASCADE,FOREIGN KEY (spell_id)REFERENCES Spells ON DELETE CASCADE);

CREATE TABLE Students_Take_Classes (class_id INTEGER,std_id INTEGER,school_id INTEGER,PRIMARY KEY(class_id, school_id, std_id), FOREIGN KEY(class_id, school_id) REFERENCES Teaches_Classes(class_id, school_id) ON DELETE CASCADE,FOREIGN KEY(std_id) REFERENCES Students(std_id) ON DELETE CASCADE);

INSERT INTO Schools (school_id, school_name, headmaster, located) values (11111, 'Hogwarts', 'Albus Dumbledore', 'Scotland');
INSERT INTO Schools (school_id, school_name, headmaster, located) values (11112, 'Beauxbatons Academy for Wizarding Arts', 'Madame Olympe Maxime', 'France');
INSERT INTO Schools (school_id, school_name, headmaster, located) values (11113, 'Durmstrang Institute', 'Igor Karkaroof', 'Norway');
INSERT INTO Schools (school_id, school_name, headmaster, located) values (11114, 'Castlelobruxo', 'Jose E. Pena', 'Brazil');
INSERT INTO Schools (school_id, school_name, headmaster, located) values (11115, 'Ilvermorny School of Witchcraft and Wizardry', 'Agilbert Fontaine', 'United States');
INSERT INTO Schools (school_id, school_name, headmaster, located) values (11116, 'Mahoutokoro School of Magic', 'Hirokazu Matsuno', 'Japan');
INSERT INTO Schools (school_id, school_name, headmaster, located) values (11117, 'Uagadou School of Magic', 'Joseph Okello', 'Uganda');
INSERT INTO Schools (school_id, school_name, headmaster, located) values (11118, 'Koldovstoretz of Magic', 'Dmitri Volstov', 'Russia');
INSERT INTO Schools (school_id, school_name, headmaster, located) values (11119, 'Salem Witches Institute', 'Joan Latimer', 'United States');
INSERT INTO Schools (school_id, school_name, headmaster, located) values (11120, 'Wizarding Academy of Dramatic Arts', 'Dame Helena Bonham Carter', 'Great Britain');

INSERT INTO Houses (house_name, school_id, std_count, founder, colors, mascot, curr_pts) VALUES ('Gryffindor', 11111, 100, 'Godric Gryffindor', '{Scarlett, Gold}', 'Lion', 100);
INSERT into Houses (house_name, school_id, std_count, founder, colors, mascot, curr_pts) VALUES ('Ravenclaw', 11111, 100, 'Rowena Ravenclaw', '{Blue, Bronze}', 'Eagle', 3250);
INSERT INTO Houses (house_name, school_id, std_count, founder, colors, mascot, curr_pts) VALUES ('Hufflepuff', 11111, 100, 'Helga Hufflepuff', '{Yellow, Black}', 'Badger', 5670);
INSERT INTO Houses (house_name, school_id, std_count, founder, colors, mascot, curr_pts) VALUES ('Slytherin', 11111, 95, 'Salazar Slytherin', '{Green, Silver}', 'Snake', 2575);
INSERT INTO Houses (house_name, school_id, std_count, founder, colors, mascot, curr_pts) VALUES ('Amour', 11112,  125, 'Amerie Amour', '{Pink, White}', 'Rabbit', 2000);
INSERT INTO Houses (house_name, school_id, std_count, founder, colors, mascot, curr_pts) VALUES ('LaMagie', 11112, 125, 'LaWanda LaMagie', '{Purple, Brown}', 'Squirrel', 1450);
INSERT INTO Houses (house_name, school_id, std_count, founder, colors, mascot, curr_pts) VALUES ('Munter', 11113, 59, 'Maximilian Munter', '{Green, Darker Green}', 'Alligator', 900);
INSERT INTO Houses (house_name, school_id, std_count, founder, colors, mascot, curr_pts) VALUES ('Vulchanova', 11113, 60, 'Valerie Vulchanova', '{Orange, Grey}', 'Vulcan', 1240);
INSERT INTO Houses (house_name, school_id, std_count, founder, colors, mascot, curr_pts) VALUES ('Apio', 11117, 100, 'Albert Apio', '{Red, Yellow}', 'Antelope', 400);
INSERT INTO Houses (house_name, school_id, std_count, founder, colors, mascot, curr_pts) VALUES ('Kiiza', 11117, 105, 'Kendrick Kiize', '{Black, White}', 'Kob', 750);
INSERT INTO Houses (house_name, school_id, std_count, founder, colors, mascot, curr_pts) VALUES ('Asimov', 11113, 60, 'Alexander Asimov', '{Aubergine, Violet}', 'Anteater', 2001);

INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (070121, 11111,11111,  ('July', 1, 2021), 'Slytherin', 'Gryffindor', 20, 45);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (070221, 11111,11111, ('July', 2, 2021), 'Ravenclaw', 'Hufflepuff', 14, 15);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (070321, 11111,11111,('July', 3, 2021), 'Gryffindor', 'Hufflepuff', 20, 47);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (070421, 11111,11111, ('July', 4, 2021), 'Slytherin', 'Ravenclaw', 15, 17);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (070521, 11111,11111, ('July', 5, 2021), 'Hufflepuff', 'Ravenclaw', 25, 27);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (101121,11111,11111, ( 'October', 11, 2021), 'Gryffindor', 'Ravenclaw', 15, 29);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (092521,11111,11111, ('September', 25, 2021), 'Slytherin', 'Gryffindor', 43, 20);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (092321, 11112, 11112, ('September', 23, 2021), 'Amour', 'LaMagie', 5, 35);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (081721, 11117, 11117, ('August', 17, 2021), 'Apio', 'Kiiza', 20, 25);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (100521, 11113, 11113, ('October', 5, 2021), 'Munter', 'Vulchanova', 90, 101);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (111721, 11117, 11111, ('November', 17, 2021), 'Apio', 'Slytherin', 80, 15);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (111821, 11111, 11112, ('November', 18, 2021), 'Ravenclaw', 'LaMagie', 10, 11);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (111921, 11117, 11111, ('November', 19, 2021), 'Kiiza', 'Gryffindor', 100, 101);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (112021, 11111, 11112, ('November', 20, 2021), 'Gryffindor', 'LaMagie', 20, 31);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (112121, 11117, 11113, ('November', 21, 2021), 'Apio', 'Vulchanova', 20, 31);
INSERT INTO Quidditch_Matches (match_id, school_id_1, school_id_2, match_date, team1, team2, team1_score, team2_score) values (112221, 11112, 11113, ('November', 22, 2021), 'LaMagie', 'Vulchanova', 35, 5);

INSERT INTO Creatures (creature_id, creature_name, dangerous, class_use) values (87, 'Hippogriff', TRUE, TRUE);
INSERT INTO Creatures (creature_id, creature_name, dangerous, class_use) values (90, 'Phoenix', FALSE, FALSE);
INSERT INTO Creatures (creature_id, creature_name, dangerous, class_use) values (85, 'Kelpie', TRUE, FALSE);
INSERT INTO Creatures (creature_id, creature_name, dangerous, class_use) values (60, 'Basilisk', TRUE, FALSE);
INSERT INTO Creatures (creature_id, creature_name, dangerous, class_use) values (54, 'Unicorn', FALSE, FALSE);
INSERT INTO Creatures (creature_id, creature_name, dangerous, class_use) values (06, 'Centaur', FALSE, TRUE);
INSERT INTO Creatures (creature_id, creature_name, dangerous, class_use) values (30, 'Fairy', FALSE, TRUE);
INSERT INTO Creatures (creature_id, creature_name, dangerous, class_use) values (32, 'Giant Squid', TRUE, FALSE);
INSERT INTO Creatures (creature_id, creature_name, dangerous, class_use) values (43, 'Jackalope', FALSE, TRUE);
INSERT INTO Creatures (creature_id, creature_name, dangerous, class_use) values (05, 'Fire Trail Snail', TRUE, TRUE);

INSERT INTO Creatures_Native_To_School (creature_id, school_id) values (87, 11111);
INSERT INTO Creatures_Native_To_School (creature_id, school_id) values (90, 11117);
INSERT INTO Creatures_Native_To_School (creature_id, school_id) values (60, 11116);
INSERT INTO Creatures_Native_To_School (creature_id, school_id) values (05, 11113);
INSERT INTO Creatures_Native_To_School (creature_id, school_id) values (06, 11118);
INSERT INTO Creatures_Native_To_School (creature_id, school_id) values (54, 11115);
INSERT INTO Creatures_Native_To_School (creature_id, school_id) values (30, 11112);
INSERT INTO Creatures_Native_To_School (creature_id, school_id) values (32, 11119);
INSERT INTO Creatures_Native_To_School (creature_id, school_id) values (43, 11114);
INSERT INTO Creatures_Native_To_School (creature_id, school_id) values (85, 11120);

INSERT INTO Spells (spell_id, spell_name, spell_descrip) VALUES (111, 'Expecto Patronum', 'Render the patronus. This is taught to students as they begin their educational journey at our institutions.');
INSERT INTO Spells (spell_id, spell_name, spell_descrip) VALUES (200, 'Reparo', 'Helpful for beginning students, this spell repairs the object the wand is facing.');
INSERT INTO Spells (spell_id, spell_name, spell_descrip) VALUES (202, 'Riddikulus', 'Turn something scary into something silly. This is one of our introductory spells, helping new students tackle their fears head on!');
INSERT INTO Spells (spell_id, spell_name, spell_descrip) VALUES (109, 'Accio', 'Taught in the second year, this spell helps the caster summon the desired item');
INSERT INTO Spells (spell_id, spell_name, spell_descrip) VALUES (243, 'Alohomora', 'Taught in second year, this spell helps the caster unlock the desired item');
INSERT INTO Spells (spell_id, spell_name, spell_descrip) VALUES (910, 'Petrificus Totalus', 'The caster can paralyze the being at the opposite end of the wand. This spell is not included in the core curriculum, but students in the Defensive Arts track will be taught this in preparation for their qualifying exam.');
INSERT INTO Spells (spell_id, spell_name, spell_descrip) VALUES (321, 'Wingardium Leviosa', 'This spell makes the object at the opposite end of the wand levitate; this is one of the first spells we teach to students at our institutions and will be helpful in all future courses.');
INSERT INTO Spells (spell_id, spell_name, spell_descrip) VALUES (900, 'Avada Kedavra', 'Kill the being at the opposite end of the wand. This spell is not included in the core curriculum of our schools, but students in Defensive Arts tracks will be taught this in preparation for their qualifying exam.');
INSERT INTO Spells (spell_id, spell_name, spell_descrip) VALUES (113, 'Lumos', 'Taught at the introductory spell camp, this spell illuminates from the end of the wand');
INSERT INTO Spells (spell_id, spell_name, spell_descrip) VALUES (928, 'Sectumsempra', 'Leave cuts on the intended that may never heal. This spell is not included in the core curriculum, but students in the Defensive Arts track will be taught this in preparation for their qualifying exam.');


INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (825, 'Severus Snape', 'Slytherin', TRUE, 11111);
INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (234, 'Fleur Delacour', 'LaMagie', TRUE, 11112);
INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (805, 'Rubeus Hagrid', null, TRUE, 11111);
INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (856, 'Dolores Umbridge', null, FALSE,  11111);
INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (756, 'Maximus Gold', 'Munter', TRUE, 11113);
INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (729, 'Nerida Karkaroff', 'Vulchanova', TRUE, 11113);
INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (800, 'Albus Dumbledore', null, TRUE, 11111);
INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (802, 'Filius Flitwick', 'Ravenclaw', TRUE, 11111);
INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (810, 'Pomona Sprout', 'Hufflepuff', TRUE, 11111);
INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (617, 'Santa Ukelele', 'Apio', TRUE, 11117);
INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (200, 'Gabrielle Delacour', 'Amour', TRUE, 11112);
INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (876, 'Minerva McGonagall', 'Gryffindor', TRUE, 11111);
INSERT INTO Professors (prof_id, prof_name, house_supervised, is_alum, school_id) VALUES (710, 'Gellert Grindelwald', 'Asimov', TRUE, 11113);

INSERT INTO Professor_Discusses_Spells (spell_id,prof_id,school_id) VALUES (111, 825, 11111); 
INSERT INTO Professor_Discusses_Spells (spell_id,prof_id,school_id) VALUES (200, 234, 11112);
INSERT INTO Professor_Discusses_Spells (spell_id,prof_id,school_id) VALUES (202, 802, 11111);
INSERT INTO Professor_Discusses_Spells (spell_id,prof_id,school_id) VALUES (109, 810, 11111);
INSERT INTO Professor_Discusses_Spells (spell_id,prof_id,school_id) VALUES (243, 856, 11111); 
INSERT INTO Professor_Discusses_Spells (spell_id,prof_id,school_id) VALUES (910, 756, 11113);
INSERT INTO Professor_Discusses_Spells (spell_id,prof_id,school_id) VALUES (321, 805, 11111);
INSERT INTO Professor_Discusses_Spells (spell_id,prof_id,school_id) VALUES (900, 617, 11117);
INSERT INTO Professor_Discusses_Spells (spell_id,prof_id,school_id) VALUES (113,729, 11113);
INSERT INTO Professor_Discusses_Spells (spell_id,prof_id,school_id) VALUES (928, 800, 11111);

INSERT INTO Teaches_Classes (class_id, school_id, class_name, clevel, prof_id) VALUES (2311, 11111, 'Introduction to Potions', 'First Year', 825);
INSERT INTO Teaches_Classes (class_id, school_id, class_name, clevel, prof_id) VALUES (2311, 11112, 'Introduction to Potions', 'First Year', 234);
INSERT INTO Teaches_Classes (class_id, school_id, class_name, clevel, prof_id) VALUES (2312, 11111, 'Care of Creatures for Beginners', 'First Year', 805);
INSERT INTO Teaches_Classes (class_id, school_id, class_name, clevel, prof_id) VALUES (2341, 11111, 'Defense Against the Dark Arts', 'Fourth Year', 856);
INSERT INTO Teaches_Classes (class_id, school_id, class_name, clevel, prof_id) VALUES (2341, 11113, 'Defense Against the Dark Arts', 'Fourth Year', 756);
INSERT INTO Teaches_Classes (class_id, school_id, class_name, clevel, prof_id) VALUES (2301, 11111, 'Musical Magic', 'First Year', 802);
INSERT INTO Teaches_Classes (class_id, school_id, class_name, clevel, prof_id) VALUES (2371, 11111, 'Advanced Defense Magic', 'Seventh Year', 802);
INSERT INTO Teaches_Classes (class_id, school_id, class_name, clevel, prof_id) VALUES (2371, 11113, 'Advanced Defense Magic', 'Seventh Year', 729);
INSERT INTO Teaches_Classes (class_id, school_id, class_name, clevel, prof_id) VALUES (2320, 11111, 'Herbology', 'Second Year', 810);
INSERT INTO Teaches_Classes (class_id, school_id, class_name, clevel, prof_id) VALUES (2350, 11117, 'Transfiguration', 'Fifth Year', 617);

INSERT INTO Schools_Offer_Classes (class_id, school_id) VALUES (2311, 11111);
INSERT INTO Schools_Offer_Classes (class_id, school_id) VALUES (2311, 11112);
INSERT INTO Schools_Offer_Classes (class_id, school_id) VALUES (2312, 11111);
INSERT INTO Schools_Offer_Classes (class_id, school_id) VALUES (2341, 11111);
INSERT INTO Schools_Offer_Classes (class_id, school_id) VALUES (2320, 11111);
INSERT INTO Schools_Offer_Classes (class_id, school_id) VALUES (2350, 11117);
INSERT INTO Schools_Offer_Classes (class_id, school_id) VALUES (2371, 11113);
INSERT INTO Schools_Offer_Classes (class_id, school_id) VALUES (2371, 11111);
INSERT INTO Schools_Offer_Classes (class_id, school_id) VALUES (2341, 11113);
INSERT INTO Schools_Offer_Classes (class_id, school_id) VALUES (2301, 11111);

INSERT INTO Creatures_Involved_In_Classes (creature_id, class_id, school_id) VALUES (87, 2312, 11111);
INSERT INTO Creatures_Involved_In_Classes (creature_id, class_id, school_id) VALUES (60, 2371, 11113);
INSERT INTO Creatures_Involved_In_Classes (creature_id, class_id, school_id) VALUES (85, 2312, 11111);
INSERT INTO Creatures_Involved_In_Classes (creature_id, class_id, school_id) VALUES (43, 2371, 11111);
INSERT INTO Creatures_Involved_In_Classes (creature_id, class_id, school_id) VALUES (60, 2350, 11117);
INSERT INTO Creatures_Involved_In_Classes (creature_id, class_id, school_id) VALUES (06, 2311, 11111);
INSERT INTO Creatures_Involved_In_Classes (creature_id, class_id, school_id) VALUES (05, 2371, 11111);
INSERT INTO Creatures_Involved_In_Classes (creature_id, class_id, school_id) VALUES (87, 2341, 11113);
INSERT INTO Creatures_Involved_In_Classes (creature_id, class_id, school_id) VALUES (32, 2320, 11111);
INSERT INTO Creatures_Involved_In_Classes (creature_id, class_id, school_id) VALUES (43, 2371, 11113);

INSERT INTO Houses_Play_In_Quidditch_Matches(house_name, match_id, school_id) VALUES ('Slytherin', 070121, 11111);
INSERT INTO Houses_Play_In_Quidditch_Matches(house_name, match_id, school_id) VALUES ('Gryffindor', 070121, 11111);
INSERT INTO Houses_Play_In_Quidditch_Matches(house_name, match_id, school_id) VALUES ('Hufflepuff', 070221, 11111);
INSERT INTO Houses_Play_In_Quidditch_Matches(house_name, match_id, school_id) VALUES ('Ravenclaw', 070221, 11111);
INSERT INTO Houses_Play_In_Quidditch_Matches(house_name, match_id, school_id) VALUES ('Gryffindor', 070321, 11111);
INSERT INTO Houses_Play_In_Quidditch_Matches(house_name, match_id, school_id) VALUES ('Hufflepuff', 070321, 11111);
INSERT INTO Houses_Play_In_Quidditch_Matches(house_name, match_id, school_id) VALUES ('Slytherin', 070421, 11111);
INSERT INTO Houses_Play_In_Quidditch_Matches(house_name, match_id, school_id) VALUES ('Ravenclaw', 070421, 11111);
INSERT INTO Houses_Play_In_Quidditch_Matches(house_name, match_id, school_id) VALUES ('Hufflepuff', 070521, 11111);
INSERT INTO Houses_Play_In_Quidditch_Matches(house_name, match_id, school_id) VALUES ('Ravenclaw', 070521, 11111);

INSERT INTO Spells_Taught_In_Class (spell_id, class_id, school_id) VALUES (111, 2311, 11111);
INSERT INTO Spells_Taught_In_Class (spell_id, class_id, school_id) VALUES (200, 2320, 11111);
INSERT INTO Spells_Taught_In_Class (spell_id, class_id, school_id) VALUES (910, 2341, 11113);
INSERT INTO Spells_Taught_In_Class (spell_id, class_id, school_id) VALUES (202, 2341, 11111);
INSERT INTO Spells_Taught_In_Class (spell_id, class_id, school_id) VALUES (109, 2311, 11112);
INSERT INTO Spells_Taught_In_Class (spell_id, class_id, school_id) VALUES (243, 2371, 11111);
INSERT INTO Spells_Taught_In_Class (spell_id, class_id, school_id) VALUES (910, 2371, 11113);
INSERT INTO Spells_Taught_In_Class (spell_id, class_id, school_id) VALUES (321, 2371, 11113);
INSERT INTO Spells_Taught_In_Class (spell_id, class_id, school_id) VALUES (113, 2311, 11112);
INSERT INTO Spells_Taught_In_Class (spell_id, class_id, school_id) VALUES (243, 2350, 11117);

INSERT INTO Professor_Supervises_House (prof_id, house_name, school_id_1, school_id_2) VALUES (825, 'Slytherin', 11111, 11111);
INSERT INTO Professor_Supervises_House (prof_id, house_name, school_id_1, school_id_2) VALUES (234, 'LaMagie', 11112, 11112);
INSERT INTO Professor_Supervises_House (prof_id, house_name, school_id_1, school_id_2) VALUES (756, 'Munter',11113, 11113);
INSERT INTO Professor_Supervises_House (prof_id, house_name, school_id_1, school_id_2) VALUES (729, 'Vulchanova', 11113, 11113);
INSERT INTO Professor_Supervises_House (prof_id, house_name, school_id_1, school_id_2) VALUES (802, 'Ravenclaw', 11111, 11111);
INSERT INTO Professor_Supervises_House (prof_id, house_name, school_id_1, school_id_2) VALUES (810, 'Hufflepuff', 11111, 11111);
INSERT INTO Professor_Supervises_House (prof_id, house_name, school_id_1, school_id_2) VALUES (617, 'Apio', 11117, 11117);
INSERT INTO Professor_Supervises_House (prof_id, house_name, school_id_1, school_id_2) VALUES (200, 'Amour', 11112, 11112);
INSERT INTO Professor_Supervises_House (prof_id, house_name, school_id_1, school_id_2) VALUES (876, 'Gryffindor', 11111, 11111);
INSERT INTO Professor_Supervises_House (prof_id, house_name, school_id_1, school_id_2) VALUES (710,'Asimov',11113, 11113);

INSERT INTO Students (std_id, std_name, year_entered, graduation_year, gpa, blood_status, patronus, house_name, school_id) VALUES (12222, 'Sam Potter', 2016, 2023, 3.76, 'Muggle', 'Ram', 'Hufflepuff', 11111);
INSERT INTO Students (std_id, std_name, year_entered, graduation_year, gpa, blood_status, patronus, house_name, school_id) VALUES (12225, 'Jasmine Weasley', 2017, 2024, 4.0, 'Pure', 'Mustang', 'Hufflepuff', 11111); 
INSERT INTO Students (std_id, std_name, year_entered, graduation_year, gpa, blood_status, patronus, house_name, school_id) VALUES (22226, 'Taylor Curie', 2014, 2021, 2.75, 'Half', 'Pig', 'Ravenclaw', 11111);
INSERT INTO Students (std_id, std_name, year_entered, graduation_year, gpa, blood_status, patronus, house_name, school_id) VALUES (22222, 'Hermione Granger', 2019, 2026, 4.0, 'Muggle', 'Phoenix', 'Gryffindor', 11111); 
INSERT INTO Students (std_id, std_name, year_entered, graduation_year, gpa, blood_status, patronus, house_name, school_id) VALUES (22223, 'Harry Potter', 2019, 2026, 1.0, 'Pure', 'Deer', 'Gryffindor', 11111);
INSERT INTO Students (std_id, std_name, year_entered, graduation_year, gpa, blood_status, patronus, house_name, school_id) VALUES (22224, 'Ron Weasley', 2019, 2026, 3.0, 'Pure', 'Jack Russell Terrier', 'Gryffindor', 11111); 
INSERT INTO Students (std_id, std_name, year_entered, graduation_year, gpa, blood_status, patronus, house_name, school_id) VALUES (22278, 'Parvati Patil', 2018, 2025, 3.42, 'Pure', 'Peacock', 'Ravenclaw', 11111);
INSERT INTO Students (std_id, std_name, year_entered, graduation_year, gpa, blood_status, patronus, house_name, school_id) VALUES (22350, 'Padma  Patil', 2018, 2025, 3.43, 'Pure', 'Ostrich', 'Ravenclaw', 11111);
INSERT INTO Students (std_id, std_name, year_entered, graduation_year, gpa, blood_status, patronus, house_name, school_id) VALUES (34688, 'Neville Longbottom', 2019, 2026, 2.89, 'Pure', 'Mouse', 'Gryffindor', 11111);
INSERT INTO Students (std_id, std_name, year_entered, graduation_year, gpa, blood_status, patronus, house_name, school_id) VALUES (34628, 'Seamus Finnigan', 2017, 2024, 3.89, 'Half', 'Ox', 'Gryffindor', 11111);

INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (111, 12222);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (109, 12222);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (200, 12225);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (202, 12225);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (243, 12225);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (910, 22226);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (202, 22222);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (900, 22222);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (900, 22278);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (109, 22278);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (243, 22223);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (900, 22350);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (321, 34688);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (109, 22224);
INSERT INTO Student_Can_Perform_Spells (spell_id, std_id) VALUES (928, 34628);

INSERT INTO Students_Take_Classes (std_id, school_id, class_id) VALUES (12225, 11111, 2320);
INSERT INTO Students_Take_Classes (std_id, school_id, class_id) VALUES (12222, 11111, 2311);
INSERT INTO Students_Take_Classes (std_id, school_id, class_id) VALUES (22226, 11111, 2371);
INSERT INTO Students_Take_Classes (std_id, school_id, class_id) VALUES (22222, 11111, 2311);
INSERT INTO Students_Take_Classes (std_id, school_id, class_id) VALUES (22278, 11111, 2312);
INSERT INTO Students_Take_Classes (std_id, school_id, class_id) VALUES (12222, 11111, 2312);
INSERT INTO Students_Take_Classes (std_id, school_id, class_id) VALUES (22222, 11111, 2312);
INSERT INTO Students_Take_Classes (std_id, school_id, class_id) VALUES (34688, 11111, 2311);
INSERT INTO Students_Take_Classes (std_id, school_id, class_id) VALUES (22350, 11111, 2312);
INSERT INTO Students_Take_Classes (std_id, school_id, class_id) VALUES (34628, 11111, 2371);
