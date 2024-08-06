# SQL Murder Mystery - Solved

## Short summery about the case

A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​. Start by retrieving the corresponding crime scene report from the police department’s database.

## Retrieving Crime Scene Report for Murder on January 15, 2018, in SQL City

```SQL Query
SELECT type,date,description
  FROM crime_scene_report
 where type = 'murder' and date='20180115' and lower(city)='sql city'
```
| Type   | Date     | Description                                                                                                    |
|--------|----------|----------------------------------------------------------------------------------------------------------------|
| murder | 20180115 | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". |

## Finding witness 1:
From the crime_scene_report says the first witness lives at the last house on 'Northwestern Dr', Based on that information, Identifying the first witness and obtain the transcript of their interview.

```SQL Query 
SELECT person.id,person.name,interview.transcript
  FROM person
  join interview 
  on person.id= interview.person_id
  where person.address_street_name='Northwestern Dr'
  order by person.address_number Desc
  limit 1
```
| ID    | Transcript                                                                                                                                                                                               |
|-------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 14887 | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". |

##  Finding Witness 2:
The crime scene report indicates that the second witness, Annabel, resides somewhere on Franklin Avenue. Using this information, need to locate Annabel and obtain the transcript of her interview.

```SQL Query
SELECT person.id,person.name,interview.transcript
  FROM person
  join interview 
  on person.id= interview.person_id
  where person.name like'%Annabel%'
and person.address_street_name='Franklin Ave'
```
| ID    | Name          | Transcript                                                                                                                                          |
|-------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| 16371 | Annabel Miller | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th. |

## Next Steps in Investigation: Review Witness Statements and Examine Relevant Records
With the interviews from both witnesses in hand, our next objective is to review the details they provided. The subsequent step is to examine gym memberships, gym check-ins, and driver's licenses for any relevant information that could aid in advancing the investigation.

```SQL Query
with Gymsuspect as(select * from get_fit_now_member
LEFT join get_fit_now_check_in
on get_fit_now_member.id=get_fit_now_check_in.membership_id
where get_fit_now_check_in.check_in_date like "%0109"
and get_fit_now_member.id like '48z%' and get_fit_now_member.membership_status='gold'
),
Suspectdetail as(
  select Gymsuspect.person_id,Gymsuspect.name,drivers_license.plate_number,drivers_license.gender
from Gymsuspect			 
left join person
on person.id=Gymsuspect.person_id
left join drivers_license
on person.license_id=drivers_license.id
    where CHARINDEX('H42W',drivers_license.plate_number,0)>0 

  )
Select * from Suspectdetail
```
| Person ID | Name          | Plate Number | Gender |
|-----------|---------------|--------------|--------|
| 67318     | Jeremy Bowers | 0H42W2       | male   |

## Suspect Identification

```SQL Query
INSERT INTO solution VALUES (1, 'Jeremy Bowers');

SELECT value FROM solution;
```
| Value                                                                                                                                                                                                                                      |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer. |

## Advancing the Case: Identifying the Primary Suspect
Retrieving the transcript of the Murderer's Interview to proceed with further investigation
```SQL Query

Select * from interview
where person_id = '67318'
```
| Person ID | Transcript                                                                                                                                                                                                                  |
|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 67318     | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. |

# Potential Suspect Description
 Investigating the potential suspect with the following characteristics:
 - **Height:** Approximately 5'5" to 5'7" (65" to 67")
- **Hair Color:** Red
- **Vehicle:** Tesla Model S
- **Concert Attendance:** Attended the SQL Symphony Concert three times in December 2017
- **Financial Status:** Wealthy
This information is crucial for identifying and locating the prime suspect.

```SQL Query
SELECT *
FROM drivers_license
where gender ='female' and car_make='Tesla' and 
height Between '65' And '67'
```
| ID     | Age | Height | Eye Color | Hair Color | Gender | Plate Number | Car Make | Car Model |
|--------|-----|--------|-----------|------------|--------|--------------|----------|-----------|
| 202298 | 68  | 66     | green     | red        | female | 500123       | Tesla    | Model S   |
| 291182 | 65  | 66     | blue      | red        | female | 08CM64       | Tesla    | Model S   |
| 918773 | 48  | 65     | black     | red        | female | 917UU3       | Tesla    | Model S   |

## Collecting Extended Information on Suspects
 Identifying individuals who attended "SQL Symphony" exactly 3 times in December 2017, and includes their personal details and annual income.

``` SQL Query
SELECT person.id,person.name,person.license_id,person.ssn,facebook_event_checkin.event_name,facebook_event_checkin.date,income.annual_income
  FROM person
 join facebook_event_checkin
  on person.id=facebook_event_checkin.person_id
  join income
  on person.ssn=income.ssn
  where person.id in('99716','90700','78881')and
  facebook_event_checkin.event_name like '%SQL Symphony%' and facebook_event_checkin.date like '201712%'
  
  group by facebook_event_checkin.person_id
  having 
  count(*)=3
  ```
  | ID     | Name            | License ID | SSN       | Event Name            | Date     | Annual Income |
|--------|-----------------|------------|-----------|-----------------------|----------|---------------|
| 99716  | Miranda Priestly | 202298     | 987756388 | SQL Symphony Concert  | 2017-12-29 | 310000        |


## Primary Suspect

```SQL Query
INSERT INTO solution VALUES (1, 'Miranda Priestly');

SELECT value FROM solution;
```
| Value                                                                                                                                                                           |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne! |
