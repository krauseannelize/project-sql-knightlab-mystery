# SQL Murder Mystery Solution

![Knight Lab SQL Murder Mystery Illustration](/knightlab-illustration.png)

## Where the Mystery Begins

The SQL game starts with this clue:

> A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​. Start by retrieving the corresponding crime scene report from the police department’s database.

A database schema diagram is also provided to assist in the investigation:

![SQL Murder Mystery Database Schema](/knightlab-database-schema.png)

## How I Approached the Mystery

### Step 1 | Retrieve the Crime Scene Report

#### Query

```sql
SELECT *
FROM
    crime_scene_report
WHERE
	date = '20180115'
	AND type = 'murder'
	AND city = 'SQL City'
;
```

#### Output

| date | type | description | city |
| --- | --- | --- | --- |
| 20180115 | murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". | SQL City |

---

### Step 2 | Identify the Witnesses 

#### Query

```sql
SELECT *
FROM
    person
WHERE
	(address_street_name LIKE '%Northwestern%'
  	AND address_number = (
        SELECT MAX(address_number)
    	FROM
		  	person
    	WHERE
			address_street_name LIKE '%Northwestern%'
	  	)
  	)
    OR
	(name LIKE '%Annabel%'
	AND address_street_name LIKE '%Franklin%'
    )
;
```

#### Output

| id | name | license_id | address_number | address_street_name | ssn |
| --- | --- | --- | --- | --- | --- |
| 14887 | Morty Schapiro | 118009 | 4919 | Northwestern Dr | 111564949 |
| 16371 | Annabel Miller | 490173 | 103	| Franklin Ave | 318771143 |

---

### Step 3 | Review Witness Interviews

#### Query

```sql
SELECT 
    CASE 
        WHEN person_id = 14887 THEN 'Witness 1'
        WHEN person_id = 16371 THEN 'Witness 2'
        ELSE 'Unknown'
    END AS witness_label
    , transcript
FROM
    interview
WHERE
    person_id IN (14887, 16371)
;
```

#### Output

| witness_label	| transcript |
| --- | --- |
| Witness 1 | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". |
| Witness 2 | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th. |

---

### Step 4 | Locate the Killer

#### Query

```sql
SELECT
    p.id
    , p.name
    , dl.gender
    , dl.plate_number
    , m.id AS gym_id
    , m.membership_status AS gym_status
FROM
    drivers_license AS dl
JOIN
    person AS p
ON
    dl.id = p.license_id
JOIN
    get_fit_now_member AS m
ON
    p.id = m.person_id
JOIN
    get_fit_now_check_in AS ci
ON
    m.id = ci.membership_id
WHERE
    dl.gender = 'male'
    AND m.id LIKE '48Z%'
    AND m.membership_status = 'gold'
    AND dl.plate_number LIKE '%H42W%'
    AND ci.check_in_date = 20180109
;
```

#### Output

| id | name | gender | plate_number | gym_id | gym_status |
| --- | --- | --- | --- | --- | --- |
| 67318 | Jeremy Bowers | male | 0H42W2 | 48Z55 | gold |

---

### Step 5 | Verify the Solution

#### Query

```sql
INSERT INTO solution VALUES (1, "Jeremy Bowers")
;
SELECT value FROM solution
;
```

#### Output

| value |
| --- |
| Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer. |

---

### Step 6 | Review the Killer's Interview

#### Query

```sql
SELECT *
FROM
    interview
WHERE
    person_id = 67318
;
```

#### Output

| person_id | transcript |
| --- | --- |
| 67318 | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. |

---

### Step 7 | Identify the Mastermind

#### Query

```sql
SELECT
	p.id
	, p.name
	, dl.gender
	, dl.height
	, dl.hair_color
	, dl.car_make || ' ' || dl.car_model AS car
	, COUNT(fc.event_id) AS symphony_attendances
FROM
	person AS p
JOIN
	drivers_license AS dl
ON
	p.license_id = dl.id
JOIN
	facebook_event_checkin AS fc
ON
	p.id = fc.person_id
WHERE
	dl.gender = 'female'
	AND dl.height >= 64
	AND dl.height <= 68
	AND dl.hair_color = 'red'
	AND dl.car_make = 'Tesla'
	AND dl.car_model = 'Model S'
	AND fc.event_name = 'SQL Symphony Concert'
	AND fc.date LIKE '201712%'
GROUP BY
	p.id
	, p.name
HAVING
	COUNT(fc.event_id) = 3
;
```

#### Output

| id | name | gender | height | hair_color | car | symphony_attendances |
| --- | --- | --- | --- | --- | --- | --- |
| i99716 | Miranda Priestly | female | 66 | red | Tesla Model S | 3 |

---

### Step 8 | Verify the Solution

#### Query

```sql
INSERT INTO solution VALUES (1, "Miranda Priestly")
;
SELECT value FROM solution
;
```

#### Output

| value |
| --- |
| Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne! |
