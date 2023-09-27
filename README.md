# Murder-Mystery-Solution
My solution to the Murder Mystery SQL challenge: https://mystery.knightlab.com/#experienced

## Getting necessary details from the crime scene report table:
```
SELECT *
FROM crime_scene_report
WHERE city = 'SQL City'
	AND date = 20180115
	AND type = 'murder'
```
This results in: Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".

## Getting details of witnesses:
The following gives the first witness details:
```
SELECT *
FROM person
WHERE address_street_name = 'Northwestern Dr'
ORDER BY address_number DESC
LIMIT 1
```
Getting the details of the second witness just means changing the street name as well as incorporating a the LIKE function:
```
SELECT *
FROM person
WHERE address_street_name = 'Franklin Ave'
	AND name LIKE '%Annabel%'
ORDER BY address_number DESC
LIMIT 1
```

## Getting interview statements from the witnesses through the ID's found previously:
```
SELECT *
FROM interview
WHERE person_id = 14887
	OR person_id = 16371
```
Witness 1 Statement: I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".

Witness 2 Statement: I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.

## Finding details with membership ID starting with 48Z on January 9 2018:
```
SELECT *
FROM get_fit_now_check_in
WHERE membership_id LIKE '48Z%'
	AND check_in_date = 20180109
```
Membership IDS Starting with 48Z on that specific date were 48Z7A, and 48Z55.

## Finding which the killer by joining all tables together using the membership_id and plate number:
```
SELECT
	T1.membership_id,
	T3.id,
	T2.name,
	T4.plate_number
FROM get_fit_now_check_in AS T1
INNER JOIN get_fit_now_member AS T2 ON T1.membership_id = T2.id
INNER JOIN person AS T3 ON T2.person_id = T3.id
INNER JOIN drivers_license AS T4 ON T3.license_id = T4.id
WHERE membership_id = '48Z7A'
	OR membership_id = '48Z55'
	OR plate_number LIKE 'H42W'
```

## Finding the real villain:
Firstly plugging in the id of the killer in the interview table:
```
SELECT *
FROM interview
WHERE person_id = 67318
```
Transcript is: I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.

Joined the facebook_event_checkin, person, and drivers_license tables together using the WHERE clause to filter based on the killer's transcript:
```
SELECT
	T2.id,
	T1.event_name,
	T1.date,
	T2.name,
	T3.gender,
	T3.height,
	T3.hair_color,
	T3.car_make,
	T3.car_model
FROM facebook_event_checkin AS T1
INNER JOIN person AS T2 ON T1.person_id = T2.id
INNER JOIN drivers_license AS T3 ON T2.license_id = T3.id
WHERE event_name = 'SQL Symphony Concert'
	AND date BETWEEN 20171201 AND 20171231
	AND	gender = 'female'
	AND height BETWEEN 65 AND 67
	AND hair_color = 'red'
	AND car_make = 'Tesla'
	AND car_model = 'Model S'
```


